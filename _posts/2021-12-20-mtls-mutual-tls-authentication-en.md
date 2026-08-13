---
layout: post
tags: [posts-en]
title: Notes on mTLS (Mutual TLS)
description: >
  The difference between plain TLS and mTLS, the handshake flow, and how to configure it on Nginx and Java.
---

While integrating a prepaid electronic payment service with an external institution, I had to set up mTLS from scratch for the first time, so I wrote up the concepts and configuration here.

## 1. TLS vs mTLS

Plain TLS (one-way authentication) only has the client verify the server's certificate. When a browser connects to a website, the server proves "I really am this domain" with a certificate, and the client verifies it by following the CA chain.

mTLS (mutual TLS) adds one more step: **the server also requires and verifies the client's certificate.** In other words, the client also has to prove it's a trusted party with a certificate before the connection can be established.

| | TLS | mTLS |
|---|---|---|
| Server authentication | Yes | Yes |
| Client authentication | No | Yes |
| Typical use case | General web services | Inter-institution API integration, service-to-service comms on an internal network |

For cases like inter-institution API integration, where you need to "only allow access from an authenticated counterpart," mTLS gives you a stronger identity guarantee than an ID/password or an API key.

## 2. The mTLS Handshake Flow

1. Client requests a connection to the server (ClientHello)
2. Server sends its certificate (ServerHello, Certificate)
3. Server requests the client's certificate (CertificateRequest)
4. Client sends its certificate (Certificate)
5. Client proves ownership of the certificate by sending a value signed with its private key (CertificateVerify)
6. Both sides verify the other's certificate against their own truststore (list of trusted CAs)
7. If verification succeeds, they exchange a session key and start encrypted communication

Steps 4–5 are the crux of it. The client doesn't just send a certificate — it has to prove, via a signature, that it **actually holds the private key** corresponding to that certificate.

## 3. Configuring mTLS on Nginx

If you're verifying the client certificate at the reverse proxy layer, these are the key directives:

```nginx
server {
    listen 443 ssl;

    ssl_certificate     /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;

    # CA bundle used to verify the client certificate
    ssl_client_certificate /etc/nginx/certs/client-ca-bundle.crt;

    # Require a client certificate
    ssl_verify_client on;
    ssl_verify_depth 2;

    location / {
        proxy_pass http://backend;
        # Pass the verified client certificate info to the backend
        proxy_set_header X-SSL-Client-Verify $ssl_client_verify;
        proxy_set_header X-SSL-Client-DN     $ssl_client_s_dn;
    }
}
```

`ssl_verify_client on` is the key line — without it, this is no different from plain TLS where only the server certificate is exchanged. If you're integrating with multiple partners using different CAs, `ssl_client_certificate` needs to point to a bundle file that chains all the CA certificates together.

## 4. Keystore/Truststore Configuration in Java/Spring

In the Java world, it helps to keep two separate storage concepts straight so you don't get confused:

- **Keystore**: holds your own private key + your own certificate. Used to "prove who I am."
- **Truststore**: holds the list of CA certificates you use as the basis for deciding whether to trust the other party. Used to "judge whether the other party is trustworthy."

```bash
# Bundle a private key + certificate into a PKCS12 keystore
openssl pkcs12 -export -in client.crt -inkey client.key \
  -out client.p12 -name client-alias

# Import the partner's CA certificate into a truststore
keytool -importcert -alias partner-ca -file partner-ca.crt \
  -keystore truststore.jks
```

In Spring Boot, you configure this in `application.yml`:

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

`client-auth: need` is what makes the server require a client certificate (use `want` if it should be optional).

## 5. Common Issues

- **CN/SAN mismatch**: If the client certificate's CN or SAN doesn't match what was agreed with the integration partner beforehand, verification will fail. It's important to align the CN/SAN when requesting the certificate.
- **Missing chain**: If a certificate was issued through an intermediate CA and the intermediate certificate is missing from the chain file, the client's own verification may succeed while the server still fails to complete the chain.
- **Missing expiry/revocation checks**: mTLS can look like it "just keeps working" once it's set up, but if you forget to check certificate expiry and revocation status via CRL/OCSP, an expired certificate can keep getting allowed to connect.
- **Firewall/network configuration on both sides**: mTLS itself is a TLS concern above the application layer, but in practice the connection only succeeds end-to-end if the network configuration also lines up — IP allow-listing for the partner, open ports, and so on. The first step in troubleshooting is figuring out whether you're dealing with a certificate problem or a network problem.
