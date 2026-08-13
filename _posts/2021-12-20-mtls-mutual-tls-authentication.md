---
layout: post
tags: [posts]
title: mTLS(상호 인증 TLS) 구조 정리
description: >
  일반 TLS와 mTLS의 차이, 핸드셰이크 절차, Nginx·Java 환경에서의 설정 방식을 정리했다.
---

선불전자지급서비스를 외부 기관과 연계하면서 mTLS 설정을 처음부터 잡아볼 일이 생겨 개념과 설정 방식을 정리한다.

## 1. TLS vs mTLS

일반적인 TLS(편방향 인증)는 클라이언트가 서버의 인증서만 검증한다. 브라우저가 웹사이트에 접속할 때 서버가 "나는 이 도메인이 맞다"는 걸 인증서로 증명하고, 클라이언트는 그걸 CA 체인을 따라 검증하는 구조다.

mTLS(mutual TLS, 상호 인증)는 여기에 더해 **서버도 클라이언트의 인증서를 요구하고 검증**한다. 즉 클라이언트도 자신이 신뢰할 수 있는 주체임을 인증서로 증명해야 연결이 성립한다.

| 구분 | TLS | mTLS |
|---|---|---|
| 서버 인증 | O | O |
| 클라이언트 인증 | X | O |
| 주 사용처 | 일반 웹 서비스 | 기관 간 API 연계, 내부망 서비스 간 통신 |

기관 간 API 연계처럼 "인증된 상대방만 접근을 허용해야 하는" 경우, 아이디/패스워드나 API 키보다 mTLS가 더 강한 신원 보증을 준다.

## 2. mTLS 핸드셰이크 흐름

1. 클라이언트가 서버에 연결 요청 (ClientHello)
2. 서버가 자신의 인증서를 전달 (ServerHello, Certificate)
3. 서버가 클라이언트 인증서를 요구 (CertificateRequest)
4. 클라이언트가 자신의 인증서를 전달 (Certificate)
5. 클라이언트가 자신의 개인키로 서명한 값을 전달해 인증서의 소유권을 증명 (CertificateVerify)
6. 양쪽 모두 상대 인증서를 각자의 트러스트스토어(신뢰 CA 목록) 기준으로 검증
7. 검증에 성공하면 세션 키를 교환하고 암호화 통신 시작

핵심은 4~5번이다. 클라이언트가 인증서만 보내는 게 아니라, 그 인증서에 대응하는 **개인키를 실제로 가지고 있음을** 서명으로 증명해야 한다.

## 3. Nginx에서 mTLS 설정

리버스 프록시 단에서 클라이언트 인증서를 검증하도록 구성하는 경우, 핵심 지시어는 다음과 같다.

```nginx
server {
    listen 443 ssl;

    ssl_certificate     /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;

    # 클라이언트 인증서를 검증할 때 사용할 CA 번들
    ssl_client_certificate /etc/nginx/certs/client-ca-bundle.crt;

    # 클라이언트 인증서를 필수로 요구
    ssl_verify_client on;
    ssl_verify_depth 2;

    location / {
        proxy_pass http://backend;
        # 검증된 클라이언트 인증서 정보를 백엔드로 전달하고 싶을 때
        proxy_set_header X-SSL-Client-Verify $ssl_client_verify;
        proxy_set_header X-SSL-Client-DN     $ssl_client_s_dn;
    }
}
```

`ssl_verify_client on`이 핵심이다. 이게 없으면 서버 인증서만 오가는 일반 TLS와 다를 게 없다. 연계 상대방이 여럿이라 CA가 다양하면 `ssl_client_certificate`에 CA 인증서를 체인으로 이어붙인 번들 파일을 지정해야 한다.

## 4. Java/Spring에서 Keystore/Truststore 설정

Java 진영에서는 두 개의 저장소 개념을 구분해서 이해하는 게 헷갈리지 않는 요령이다.

- **Keystore**: 내 개인키 + 내 인증서를 담는 저장소. "내가 누구인지 증명할 때" 사용.
- **Truststore**: 상대방을 신뢰할지 판단할 때 기준이 되는 CA 인증서 목록을 담는 저장소. "상대방이 믿을 만한지 판단할 때" 사용.

```bash
# 개인키 + 인증서를 PKCS12 keystore로 묶기
openssl pkcs12 -export -in client.crt -inkey client.key \
  -out client.p12 -name client-alias

# 상대방 CA 인증서를 truststore로 import
keytool -importcert -alias partner-ca -file partner-ca.crt \
  -keystore truststore.jks
```

Spring Boot에서는 `application.yml`에서 이렇게 지정한다.

```yaml
server:
  ssl:
    key-store: classpath:client.p12
    key-store-password: ${KEYSTORE_PASSWORD}
    key-store-type: PKCS12
    trust-store: classpath:truststore.jks
    trust-store-password: ${TRUSTSTORE_PASSWORD}
    client-auth: need
```

`client-auth: need`가 서버 입장에서 클라이언트 인증서를 필수로 요구하는 옵션이다 (`want`로 하면 선택적 요구).

## 5. 자주 겪는 이슈

- **CN/SAN 불일치**: 클라이언트 인증서의 CN이나 SAN이 연계 상대방과 사전에 합의한 값과 다르면 검증 단계에서 거부된다. 인증서 발급 요청 시 CN/SAN을 미리 맞춰두는 게 중요하다.
- **체인 누락**: 중간 CA(intermediate CA)를 거쳐 발급된 인증서인데 체인 파일에 중간 인증서를 빠뜨리면, 클라이언트 자체 검증은 성공해도 서버 쪽에서 체인을 완성하지 못해 실패한다.
- **만료/폐기 확인 누락**: mTLS는 초기 설정만 맞으면 계속 동작하는 것처럼 보이지만, 인증서 만료일과 CRL/OCSP를 통한 폐기 여부 확인을 빠뜨리면 만료된 인증서로도 계속 연결이 허용되는 경우가 생긴다.
- **양방향 방화벽/네트워크 구성**: mTLS 자체는 애플리케이션 레이어 위 TLS 문제지만, 실제로는 상대방 IP 화이트리스트, 포트 개방 등 네트워크 구성이 같이 맞아야 최종적으로 연결이 성립한다. 인증서 문제인지 네트워크 문제인지 구분해서 접근하는 게 트러블슈팅의 첫걸음이다.
