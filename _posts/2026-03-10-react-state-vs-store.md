---
layout: post
tags: [posts]
title: React 상태는 아무데나 두면 안 된다 — store를 쓸 때와 쓰지 말아야 할 때
description: >
  전역 store에 상태를 다 몰아넣었을 때 겪은 문제와, local state / 전역 store / 서버 상태를 구분해서 쓰게 된 기준.
---

ITSM 화면을 React로 새로 구축하면서, 초반에는 "일단 store에 넣어두면 어디서든 꺼내 쓸 수 있으니 편하다"는 생각으로 대부분의 상태를 전역 store에 몰아넣었다. 화면이 늘어날수록 그게 문제가 됐다.

## 1. 뭐가 문제였나

- **불필요한 리렌더링**: 검색창 입력값처럼 컴포넌트 하나 안에서만 쓰는 값까지 store에 두니, 글자를 한 자 칠 때마다 그 store를 구독하는 화면 전체가 다시 렌더링됐다. 화면이 커질수록 타이핑이 눈에 띄게 버벅였다.
- **데이터 흐름 추적이 어려워짐**: 어떤 값이 언제 왜 바뀌는지 store 액션을 다 뒤져야 알 수 있었다. 컴포넌트만 보고는 "이 값이 여기서 시작해서 저기까지 간다"는 흐름이 안 보였다.
- **서버에서 받아온 데이터의 신선도 문제**: 목록 조회 API 응답을 store에 그대로 저장해두고 여기저기서 꺼내 썼는데, 다른 화면에서 데이터를 수정한 뒤 store 갱신을 깜빡하면 화면마다 보여주는 값이 서로 달라지는 일이 생겼다. "지금 이 값이 최신인지"를 보장하는 책임이 전부 개발자 손에 맡겨져 있었다.

## 2. 상태를 세 종류로 나눠서 생각하기

| 종류 | 예시 | 둘 곳 |
|---|---|---|
| **Local UI state** | 입력창 값, 모달 열림/닫힘, 특정 행의 hover 상태 | `useState`/`useReducer` (해당 컴포넌트 안에서만) |
| **Shared UI state** | 로그인 사용자 정보, 다크모드 여부, 여러 화면이 같이 봐야 하는 필터 조건 | 전역 store (Redux/Zustand/Context) |
| **Server state** | 목록 조회 결과, 상세 조회 결과 등 서버가 원본을 갖고 있는 데이터 | React Query 같은 서버 상태 캐싱 라이브러리 |

이 세 개를 구분하지 않고 다 "전역 store"라는 한 바구니에 담으면, store가 UI 상태 관리자이면서 동시에 서버 데이터 캐시 역할까지 떠맡게 되고, 위에서 겪은 문제들이 자연스럽게 따라온다.

## 3. 판단 기준

새로운 상태를 어디에 둘지 정할 때 스스로 물어보는 순서:

1. **이 값을 쓰는 컴포넌트가 하나(혹은 그 하위 트리)뿐인가?** → `useState`로 충분하다. 굳이 store로 올릴 이유가 없다.
2. **이 값의 원본이 서버 응답인가?** → store에 복사해서 관리하지 말고, 서버 상태 캐싱 라이브러리에 맡긴다. store는 "서버에서 받아온 데이터"가 아니라 "그 데이터를 어떻게 보여줄지"에 대한 UI 상태만 갖는다.
3. **서로 멀리 떨어진 여러 화면이 정말로 같은 값을 실시간으로 공유해야 하는가?** → 그럴 때만 전역 store로 올린다.

3번 기준이 특히 중요했다. "혹시 나중에 다른 화면에서도 쓸지 모르니까" 미리 store에 넣어두는 습관이 문제의 시작이었다. 실제로 필요해지기 전까지는 컴포넌트 로컬에 두고, 진짜로 여러 화면이 공유해야 하는 시점이 왔을 때 그때 store로 끌어올리는 게 결과적으로 더 적은 코드로 끝났다.

## 4. 그리드 이벤트 콜백이 옛날 store 값을 참조하는 문제

store를 어디에 둘지 잘 정했다고 해서 끝이 아니었다. 저장 후 목록을 다시 불러오고, 방금 수정한 행을 다시 선택 상태로 복원하는 로직에서 한 번 더 발목을 잡혔다.

```js
// 문제가 됐던 코드
const gridEvents = useMemo(
  () => ({
    onCurrentRowChanged(_grid, _oldRow, newRow) {
      if (newRow >= 0 && rowList[newRow]) {
        setSelectedRow(rowList[newRow]);
      } else {
        setSelectedRow(null); // 의도치 않게 여기로 자주 빠졌다
      }
    },
  }),
  [rowList, setSelectedRow]
);

// 저장 후 재조회, 방금 저장한 행을 다시 선택 상태로 복원
fetchList(() => {
  const idx = list.findIndex(row => row.id === savedId);
  if (idx >= 0) {
    setTimeout(() => {
      gridRef.current?.vw?.setCurrent({ dataRow: idx }); // 임시방편: 그냥 100ms 기다렸다 실행
    }, 100);
  }
});
```

`gridEvents`는 `rowList`가 바뀔 때만 새로 만들어지는 콜백이다. 그런데 재조회로 store의 목록이 갱신된 직후 곧바로 그리드에 `setCurrent`로 행 선택을 복원하면, 그리드가 그 순간 호출하는 `onCurrentRowChanged`가 **아직 새 `rowList`로 교체되지 않은, 오래된 클로저를 참조하는 콜백**일 수 있었다. 그 클로저 안의 `rowList`는 재조회 이전 상태라 인덱스가 어긋나거나 값이 없었고, 그러면 `else` 분기로 빠져서 방금 선택한 행의 상세 대신 `null`이나 엉뚱한 이전 값으로 덮어써버렸다. `setTimeout(..., 100)`으로 잠깐 기다렸다 실행하는 임시방편을 써봤지만, 이건 "그 사이에 리렌더링이 끝나길 바라는" 운에 맡기는 방식이라 느린 환경에서는 여전히 재현됐다.

근본 원인은 "이벤트가 발생한 시점"과 "React가 그 이벤트를 처리할 콜백을 최신 상태로 교체하는 시점" 사이에 시차가 있을 수 있다는 것이었다. 그래서 콜백 안에서 클로저로 캡처된 값 대신, 호출되는 바로 그 순간의 store 값을 직접 읽어오도록 바꿨다.

```js
// 수정한 코드 — 클로저 대신 store의 현재 값을 직접 읽는다
const gridEvents = {
  onCurrentRowChanged(_grid, _oldRow, newRow) {
    const rowList = useGridStore.getState().rowList; // 항상 최신 값
    if (newRow >= 0 && rowList[newRow]) {
      setSelectedRow(rowList[newRow]);
    } else if (newRow < 0) {
      setSelectedRow(null);
    }
  },
};

fetchList(() => {
  const { rowList } = useGridStore.getState();
  const idx = rowList.findIndex(row => row.id === savedId);
  if (idx >= 0) {
    gridRef.current?.vw?.setCurrent({ dataRow: idx }); // setTimeout 없이 바로 호출해도 안전
  }
});
```

Zustand의 `getState()`는 구독 없이 그 순간의 스냅샷을 즉시 읽어온다. 클로저에 뭐가 캡처돼 있었는지와 무관하게 항상 최신 값을 보게 되니, `setTimeout`으로 타이밍을 맞추려는 시도 자체가 필요 없어졌다. "store 값을 컴포넌트 렌더링 중에 구조분해해서 쓰는 것"과 "이벤트 콜백 안에서 그 순간의 값을 읽는 것"은 다른 문제라는 걸 이 케이스로 체감했다.

## 5. 정리

store는 "상태를 넣어두는 창고"가 아니라 "여러 화면이 실제로 공유해야 하는 상태만 신중하게 올리는 곳"으로 써야 한다는 걸 체감했다. 로컬로 충분한 값은 로컬에 두고, 서버가 원본인 데이터는 서버 상태 전용 도구에 맡기고, 정말 전역으로 공유돼야 하는 값만 store에 남기니 리렌더링 문제도 데이터 정합성 문제도 눈에 띄게 줄었다. 그리고 이벤트 콜백처럼 "React 렌더링 사이클 밖에서 비동기로 호출될 수 있는 지점"에서는, 클로저에 의존하지 말고 그 순간의 store 값을 직접 읽어오는 습관이 타이밍 버그를 근본적으로 줄여줬다.
