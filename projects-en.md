---
layout: page
title: Project
description: >
  Key responsibilities and achievements by company, along with a detailed development history.
---

## Bankware Global — Financial Industry Division, Senior Professional
**Jan 2023 – Present**

Company: Builds client systems around digital banking solutions.

### Key Responsibilities & Achievements

**ITSM Rebuild**

Working on converting a BMC Remedy-based ITSM into a proprietary system aligned with the ITIL4 standard. I'm analyzing the client's IT service operations to redesign core processes — service requests, change management, incident management — and analyzing the SLAs of the existing system to migrate them into the new ITSM environment. I'm also redesigning interfaces with related systems such as e-approval and configuration management, and implementing new ITSM screens and user features with a React-based frontend.

**Oracle Access Management 12c Build**

Led an upgrade of the OAM (Oracle Access Manager) access control solution from 11g to 12c. Modified and migrated the OAM authentication integration system, an OAAM (Oracle Adaptive Access Manager)-based authorization/login management system, a CS agent installed on individual user PCs, and a simple-authentication business system to fit the new 12c server environment.

Also designed and built an automated deployment pipeline for the dev and staging servers using SVN and Jenkins, establishing an efficient and stable release process.

As the project DBA, I handled data and privilege migration, applied and verified backup/recovery processes, configured instance parameters, and set up statistics collection — all in service of DB performance optimization and stable operation. In particular, I worked with Oracle engineers to build an environment that met the client's DB standards, applying the optimal architecture and configuration.

**ERP HR/Payroll Statistics & Management Accounting Development**

Built the client's management accounting / payroll statistics service using our in-house solution. Used hierarchical queries to compute amount totals by parent/child account code, and improved the screen to let users dynamically select aggregation conditions. I also improved a UI component that previously didn't support removing individual employees once selected in a search filter, delivering a more convenient UI.

**Statistical Compilation System Preprocessing (SAS to Python)**

Analyzed and designed the process of transforming data collected from external agencies to fit the client's system and loading it into the DB. Analyzed the existing SAS-based implementation and migrated it to Python.

### Skill Set

Backend – BXM, Spring  
Frontend – varies by client (Angular.js, Realgrid, Nexacro, Vue.js, React vite)  
Data preprocessing – Python (Brightics)

### Development History

| Start | End | Details |
|---|---|---|
| 2025.07 | 2026.08 | ITSM development for a financial-sector public institution rebuilding its management information system (frontend: React vite) |
| 2024.05 | 2025.06 | Authentication/authorization system development for a life insurer's Oracle OAM integrated authentication solution upgrade |
| 2024.01 | 2024.03 | Proposal support for a central bank, plus an OpenAPI UI prototype built with Swagger |
| 2023.08 | 2023.12 | New management accounting features for an asset trust company's ERP (frontend: Realgrid, Angular.js) |
| 2023.06 | 2023.07 | HR/payroll statistics screens for a pension mutual fund's ERP (frontend: Nexacro) |
| 2023.03 | 2023.05 | Proposal support and prototype development for a commercial bank's FX system (frontend: SpiderGen) |
| 2023.01 | 2023.03 | SAS-to-Python migration of the business analysis preprocessing for a central bank's economic statistics system |


## Welcome Payments — Future Strategy Division, Assistant Manager
**Nov 2021 – Feb 2022 (4 months)**

Company: A Welcome Financial Group affiliate providing PG, prepaid electronic payment, and P2P services.

### Key Responsibilities & Achievements

**Prepaid Electronic Payment Service — MyData Integration**

Developed an integration with Koscom for a prepaid electronic payment service before it had launched publicly (prior to app store listing), taking it through to completed integration testing. Worked with the security team on Nginx reverse proxy and mTLS configuration and network setup during the integration. Used Spring Security to provide a token-based service, decrypting and validating JWT-encrypted user information in an Authorize Filter. Similar to pulling member info from a session on every call, I used the token to retrieve member info and process business logic with it.

### Skill Set

Backend – SpringBoot, jdk11, Spring Security, Lombok, Swagger2  
Frontend – React  
CI/CD – Jenkins  
Web server – Nginx

### Development History

| Start | End | Details |
|---|---|---|
| 2021.01 | 2021.02 | Bank Pay bank account transfer integration for the prepaid electronic payment service |
| 2021.11 | 2021.12 | Koscom MyData information-provision API integration for the Welcome Pay prepaid electronic payment service |


## Korea Information & Communications — Systems Development Team, Staff
**Oct 2016 – Oct 2021 (5 years 1 month)**

Company: Korea's first VAN (value-added network) operator, providing terminal development, PG, and automated bookkeeping services.

### Key Responsibilities & Achievements

**PG Real-Time Approval Service**

Maintained mobile and PC EasyPay authentication services and developed simple-payment integrations with partners such as Payco, UnionPay, and Korea Financial Telecommunications & Clearings Institute account transfers, while maintaining the client-provided payment module and manual — contributing to acquiring both client and partner payment providers. Maintained a payment notification service built on the Apache Mina framework, contributing to service stability by onboarding new payment protocols, resolving HTTP/TCP communication issues, scaling out processes for high-volume traffic, and tuning the system through load testing and thread-model changes. Contributed to system quality management through routine PM work, DR incident-response drills, and service monitoring.

**Automated Bookkeeping Service**

Contributed to making tax filing easier for self-employed users by accommodating forms and business logic that the National Tax Service revises every year — comprehensive income tax, year-end settlement, tax-exempt business filings, and more. Ran a service partnership project with a commercial bank, building a hybrid-app integration API and a separate PC web service, contributing to B2B revenue. Improved service quality through regular penetration testing and source code vulnerability checks.

**PG Settlement Information Service**

Developed batch services for merchant/HQ transaction history, settlement reconciliation, and transaction reconciliation, making the work easier for merchant settlement staff. Ran daily check-batch monitoring and verification, checking amount matches and file receipt status to validate data integrity and improve service quality.

### 1) PG Team · Settlement Information (Nov 2020 – Oct 2021, 1 year)

Tech stack: Merchant/HQ manager transaction history – JAVA, Struts framework, JEUS7–WEBTOB, JDK1.7, ibatis/mybatis · Transaction/billing/refund/settlement SFTP/web reconciliation – ProC, JAVA–Spring, Struts

| Start | End | Details |
|---|---|---|
| 2021.11 | 2021.12 | Accommodated an API change for Cafe24's transaction reconciliation pagination (added offset/limit request params and a has_next_paging response param) |
| 2021.09 | 2021.10 | Built a VAN-transmission settlement file generation service for unattended kiosks at a beverage company |
| 2021.07 | 2021.09 | Onboarded KakaoPay/Kakao Money; owned building the transaction/sales-slip/reconciliation file generation |
| 2021.07 | 2021.08 | Accommodated SFTP/web reconciliation for simple account-transfer payments |
| 2021.05 | 2021.06 | Added generation of a National Tax Service cash-receipt SFTP reconciliation file, plus a type field based on receipt date |
| 2021.05 | 2021.05 | Remediated a penetration-test vulnerability; built and applied a function to reject transactions past their allowed window |
| 2021.05 | 2021.06 | Accommodated Samsung Pay authentication fees (owned the fee-calculation logic change in settlement/reconciliation file generation) |
| 2021.04 | 2021.05 | Accommodated a unified transaction history for merchant managers; unioned tables across payment methods |
| 2021.03 | 2021.04 | Accommodated partial cancellation for a gas station chain's fuel service; owned transaction history and receipts |
| 2021.01 | 2021.02 | Accommodated combined taxation for cash receipts; owned transaction history and sales slips |
| 2021.01 | 2021.02 | Accommodated sales-slip labeling requirements for a mail-order business; owned sales-slip development |
| 2020.11 | 2020.12 | Built a mobile sales-slip view using media queries |

### 2) Automated Bookkeeping Team · Fixed Assets & Tax Filing (Jul 2019 – Oct 2020, 1 year 4 months)

Tech stack: Automated tax bookkeeping web/mobile – JAVA, Dexter framework, JEUS7–WEBTOB, JDK1.7, PL/SQL, ibatis · Automated tax bookkeeping batch service – JAVA, Spring–Quartz, JDK1.7, ibatis

| Start | End | Details |
|---|---|---|
| 2020.08 | 2020.10 | Built a feature to export large volumes of member data to Excel with user-selectable columns — users pick columns on screen, then the batch service generates and uploads the file (used RowHandler, LinkedHashMap, Poi.jar) |
| 2020.06 | 2020.07 | Applied a partner tax-accountant admin site plan and integrated login with the existing site |
| 2020.02 | 2020.05 | Applied comprehensive income tax revisions; updated book-closing and accounting-entry business logic |
| 2020.01 | 2020.02 | Applied revised requirements for tax-exempt business status filings |
| 2020.01 | 2020.02 | Applied 2019 year-end settlement revisions; built a simplified payment statement (used Klipsoft to generate the report format) |
| 2019.10 | 2020.01 | Accommodated a commercial bank's affiliated Alpha tax-filing service — customized and applied the existing service's publishing layer, adjusted permissions (used a symbolic link off an existing project to create the new instance), and built a CRM-related SMS batch service for member management and filing notices |
| 2019.09 | 2019.10 | Rebuilt the mobile hybrid app; developed HR/payroll (daily worker) app-integration APIs — fetch employee info, view/select/delete payment records, view payment details, calculate pay, load recent pay, save |
| 2019.07 | 2019.09 | Integrated homepage banner display for the automated bookkeeping web service |

### 3) PG Team · Real-Time Approval (Oct 2016 – Jun 2019, 2 years 9 months)

Tech stack: Client integration samples per language for merchants – ASP, JSP, PHP, .NET, Python 3.5 (Django) · Client payment-message encryption/communication library – dll (C#), jar (Java), CLI (C) · External payment institution gateway & payment notification – JAVA, Apache Mina framework, WebT (TmaxClient), JDK1.6–1.8 · Integrated online payment authentication window – JAVA, Struts framework, JEUS5-WEBTOB, JDK1.5

| Start | End | Details |
|---|---|---|
| 2019.05 | 2019.07 | Technical support and coordination (meetings, conference calls, email) for onboarding new domestic/international merchants |
| 2019.03 | 2019.05 | Maintained the PC integrated payment authentication window (added VP authenticators Konai/Toss, fixed internal simple-payment business-logic bugs, handled logic changes from a securities firm's acquirer change, fixed a card-prefix lookup bug for overseas card authentication) |
| 2019.02 | 2019.03 | Improved the merchant payment-notification daemon — split logs and added forced-termination handling. In a triple-redundant setup with 3 processes per server, occasional abnormal response delays occurred without hitting a connect/read timeout or throwing an exception (specific processes over HTTPS would back up 20–30 transactions in a row, then clear naturally after ~10 minutes; happened twice a year; never reproducible in the dev environment) |
| 2019.01 | 2019.02 | Refreshed the Korean/English merchant integration manual |
| 2018.12 | 2019.01 | Extended the digit length for mobile Shinhan Simple Pay ARS identity verification (per a KISA policy update) |
| 2018.11 | 2018.12 | Accommodated an internal policy change to interest-free purchase lookup logic; updated ledger-lookup business logic in the integrated payment window |
| 2018.11 | 2018.12 | Accommodated a tiered fee policy for small merchants; updated ledger-lookup business logic in the integrated payment window |
| 2018.10 | 2018.12 | Improved the credit-card selection UI for instant discounts (coupons, pre-discounts) in the integrated authentication window |
| 2018.10 | 2018.12 | Improved the merchant payment-notification daemon; upgraded the httpclient/mina libraries and hardened the source |
| 2018.07 | 2018.12 | Onboarded Payco; owned the merchant integration sample and authentication API integration |
| 2018.07 | 2018.07 | Improved the interest-free event lookup API logic |
| 2018.05 | 2018.06 | Followed up on a CMS firm-banking system's Unix-to-Linux migration project |
| 2018.04 | 2018.06 | Improved the SMS URL payment service (added cancellation, slip printing, seller SMS notification) |
| 2018.03 | 2018.03 | Improved the merchant payment-notification daemon; changed the threading model and added JVM startup options |
| 2018.02 | 2018.02 | Started owning the Naver Checkout web API; changed the account-transfer integration method |
| 2018.01 | 2018.02 | Started owning the mobile integrated payment authentication window; onboarded Shinhan Card Simple Pay and identity-verification integration |
| 2018.01 | 2018.02 | Started owning the PC integrated payment authentication window; developed a card-event lookup API (JSON, XML) |
| 2017.12 | 2017.12 | Load-tested the WebT (TmaxClient) communication library |
| 2017.11 | 2017.11 | Fixed a communication error in the merchant payment-notification daemon for a specific merchant (a JAVA export/import extension patch) |
| 2017.08 | 2017.10 | Onboarded Samsung Pay, SSG Pay (account transfer), and KFTC BankPay; owned the merchant integration samples |
| 2017.06 | 2017.09 | Onboarded UnionPay UPOP 5.0; owned the external payment-gateway communication daemon and merchant integration samples |
| 2017.04 | 2017.06 | Followed up on a CMS firm-banking and delivery-agency system build (supporting outsourced staff) |
| 2017.04 | 2017.04 | Onboarded SSG Pay (credit card); updated the merchant integration sample for the new integration method |
| 2017.03 | 2017.03 | Fixed a UTF-8 Korean text corruption issue in the client payment-communication module (jar); changed the encryption library |
| 2016.12 | 2017.03 | Started owning the merchant payment-notification daemon; built a notification-receiving daemon for merchants |
| 2016.12 | 2016.12 | Improved (automated) JEUS log collection and analysis per WAS container |
| 2016.11 | 2016.12 | Built a Python 3.5 merchant integration sample using the Django framework |
| 2016.10 | 2016.11 | Set up dev environments and connected payment samples for each merchant-provided language (JSP/PHP/ASP/.NET) |


## C-Square Soft — Server Development Team, Staff
**Jul 2015 – Oct 2016 (1 year 4 months)**

Company: A PG provider specializing in the ticketing and lodging industries.

### Key Responsibilities & Achievements

**PG Real-Time Approval / Settlement Information Service**

Owned development of the payment service's storefront manager and merchant settlement/VAN acquiring batch services, making the work easier for merchant settlement staff. Contributed to acquiring partners by integrating new affiliate payment providers (PayAt, mobile-phone billing) and building VAN acquiring processes.

### Skill Set

Settlement/acquiring batch process – C, Pro C, in-house framework, AIX  
Back-office storefront manager – JSP, JAVA, in-house framework, TOMCAT5.0.28, APACHE2, JDK1.4

### Development History

| Start | End | Details |
|---|---|---|
| 2016.02 | 2016.10 | Supported development of the next-gen integrated payment window / merchant manager |
| 2016.07 | 2016.10 | Built acquiring/settlement batch processes for partial cancellations by payment method (credit card/account transfer); built the SPC acquiring batch process |
| 2016.06 | 2016.07 | Integrated the PayAt baseline (ledger/contract) information API |
| 2016.04 | 2016.05 | Built a merchant reconciliation API per payment method |
| 2016.03 | 2016.04 | Built an SKT points approval-management system |
| 2016.02 | 2016.08 | Operated a settlement-agency system for the Armed Forces Welfare Corps |
| 2016.01 | 2016.02 | Integrated SKP mobile-phone billing |
| 2015.11 | 2015.12 | Integrated the LG U+ account-transfer API |
| 2015.08 | 2015.09 | Built a risk-management/settlement-batch system for auto-debit (recurring payment) merchants |
| 2015.07 | 2016.08 | Maintained and operated the PG back office, client-provided payment module, and settlement/acquiring batch processes |
