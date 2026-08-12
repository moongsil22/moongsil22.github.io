---
layout: post
title: Oracle 계층형 쿼리(CONNECT BY)로 계정코드 집계하기
description: >
  관리회계 시스템에서 상/하위 계정코드 단위 금액합계를 구할 때 쓴 Oracle 계층형 쿼리(START WITH, CONNECT BY, PRIOR, LEVEL) 정리.
---

관리회계 시스템에서 계정코드는 보통 트리 구조다. 최상위 계정 아래 중분류, 그 아래 세부 계정이 달리고, 화면에는 상위 계정의 금액이 하위 계정 금액을 다 합친 값으로 보여야 한다. 이런 상/하위 집계를 애플리케이션 코드에서 재귀로 풀 수도 있지만, Oracle에서는 계층형 쿼리로 한 번에 처리하는 게 훨씬 간결하다.

## 1. 계층형 쿼리 기본 문법

```sql
SELECT LEVEL,
       LPAD(' ', (LEVEL - 1) * 2) || ACNT_NM AS ACNT_NM,
       ACNT_CODE,
       PARENT_CODE
FROM   ACNT_MST
START WITH PARENT_CODE IS NULL     -- 최상위 계정에서 시작
CONNECT BY PRIOR ACNT_CODE = PARENT_CODE  -- 부모-자식 연결 조건
ORDER SIBLINGS BY ACNT_CODE;
```

- **START WITH**: 트리의 시작점(루트) 조건. 여기서는 부모가 없는 최상위 계정.
- **CONNECT BY PRIOR**: 부모-자식 관계를 정의. `PRIOR`가 붙은 쪽이 "이전 단계(부모) 행"을 가리킨다.
- **LEVEL**: 루트를 1로 시작해서 내려갈수록 증가하는 의사컬럼(pseudo column). 들여쓰기 표현에 자주 쓴다.
- **ORDER SIBLINGS BY**: 계층 구조를 유지한 채로 같은 레벨(형제 노드) 안에서만 정렬한다. 일반 `ORDER BY`를 쓰면 계층 순서 자체가 깨진다.

## 2. 상/하위 계정코드 단위 금액합계

핵심은 "하위 계정의 금액을 상위 계정에 다 더해서 보여줘야 한다"는 요건이다. 계층형 쿼리로 각 계정의 하위 계정 목록을 구한 다음, 그 목록에 속한 거래 금액을 합산하는 방식으로 풀었다.

```sql
SELECT M.ACNT_CODE,
       M.ACNT_NM,
       (SELECT NVL(SUM(T.AMOUNT), 0)
        FROM   TRAN_DTL T
        WHERE  T.ACNT_CODE IN (
                 SELECT ACNT_CODE
                 FROM   ACNT_MST
                 START WITH ACNT_CODE = M.ACNT_CODE
                 CONNECT BY PRIOR ACNT_CODE = PARENT_CODE
               )
       ) AS TOTAL_AMOUNT
FROM   ACNT_MST M
ORDER  BY M.ACNT_CODE;
```

각 계정(M.ACNT_CODE)마다 그 계정 자신과 모든 하위 계정을 `START WITH ACNT_CODE = M.ACNT_CODE ... CONNECT BY PRIOR`로 구한 뒤, 그 코드 목록에 속하는 거래 금액을 전부 더한다. 말단 계정(하위가 없는 계정)은 자기 자신만 포함되니 자기 금액이 그대로 나오고, 중분류·최상위 계정은 하위 계정 금액이 자동으로 다 합산돼서 나온다.

## 3. 실무에서 겪은 포인트

- **집계조건을 화면에서 동적으로 바꿔야 하는 경우**: 사용자가 "이번 달만", "특정 부서만" 같은 조건을 화면에서 선택하면 그 조건을 안쪽 `TRAN_DTL` 서브쿼리의 `WHERE`절에 바인드로 얹는 식으로 처리했다. 계층 구조(계정 트리)와 집계 조건(기간/부서)을 분리해두면, 조건이 바뀌어도 계층 쿼리 부분은 그대로 재사용할 수 있어서 유지보수가 편했다.
- **CONNECT BY LOOP 에러**: 데이터 정합성이 깨져서 계정 A의 부모가 계정 B, B의 부모가 다시 A로 순환 참조가 생기면 `ORA-01436: CONNECT BY LOOP`가 난다. 운영 데이터에서 이런 순환이 생기지 않도록 부모 코드 입력 시점에 검증 로직을 넣어두는 게 안전하다.
- **성능**: 계정 트리 자체는 대개 몇백~몇천 건 수준이라 계층 쿼리 자체는 가볍지만, 안쪽 서브쿼리가 거래 상세 테이블(대용량)을 매 계정마다 스캔하는 구조라 계정 수가 많아지면 느려질 수 있다. 계정코드에 인덱스가 잡혀있는지, 거래 기간 조건으로 파티션 프루닝이 되는지를 먼저 확인하는 게 순서다.

## 4. 계정×부서 행렬로 변환하기 (PIVOT)

관리회계의 핵심은 "사업부서별로 손익을 나눠서 본다"는 것이라, 화면에는 "계정코드는 행으로, 부서는 열로" 늘어놓은 매트릭스 형태로 보여줘야 하는 경우가 많다. 기간은 조회조건으로 필터링하고, 그 기간 내 데이터를 부서 단위로 펼치는 건 `PIVOT`으로 처리했다.

```sql
SELECT *
FROM (
    SELECT ACNT_CODE,
           DEPT_CODE,
           AMOUNT
    FROM   TRAN_DTL
    WHERE  TRAN_DATE BETWEEN :FROM_DATE AND :TO_DATE   -- 기간은 조회조건으로 필터링
)
PIVOT (
    SUM(AMOUNT)
    FOR DEPT_CODE IN ('D01' AS "영업1부", 'D02' AS "영업2부", 'D03' AS "관리부")
)
ORDER BY ACNT_CODE;
```

`FOR ... IN` 뒤에 나열한 값들이 각각 하나의 컬럼이 되면서, 원래 행으로 쌓여있던 부서별 금액이 계정코드당 한 행짜리 매트릭스로 펼쳐진다. 반대로 매트릭스 형태를 다시 행 단위로 풀어야 할 때는 `UNPIVOT`을 쓰면 된다.

여기서 실무적으로 걸리는 부분은, `IN` 절에 나열하는 부서 목록이 고정돼 있지 않고 **사용자가 화면에서 조회할 부서나 기간을 동적으로 바꾼다는 점**이다. `PIVOT`의 `IN` 절은 정적 SQL에서는 목록을 미리 써놔야 하기 때문에, 조회 조건에 맞춰 `IN` 절 문자열을 동적으로 조립해 실행하는 Dynamic SQL로 처리했다. 부서 개수 자체가 매번 달라지는 화면이라면, PIVOT을 애초에 애플리케이션단에서(예: 조회 결과를 받아 화면 그리드가 자체적으로 행↔열 전환) 처리하는 방법도 같이 고려해볼 만하다.

## 5. 정리

계층형 쿼리는 문법 자체는 짧지만 "부모-자식을 어떤 컬럼으로 연결할지"만 정확히 잡으면 애플리케이션 레이어에서 재귀 호출로 풀던 로직을 SQL 한 번으로 끝낼 수 있다. 관리회계처럼 계정 트리를 다루는 도메인에서는 기본기로 알아둘 만하다.
