---
layout: page
title: Cover Letter
description: >
  My background, personality, motivation, hands-on experience, and how I approach incident response.
---

## 1. Background

Walking my two dogs every day is part of my routine. On a snowy day, I decided to buy them dog shoes for our walk. The first pair had velcro straps around the ankle, but they came off as soon as the dogs started running. The second pair, made of non-woven fabric for one-time use, came off just as easily. I was ready to give up, but as a last try, I wrapped bandage tape over the disposable shoes — and this time they stayed on, letting us walk safely.

What I took away from that small experience was the value of trying just one more time. In application development, too, a single attempt rarely gets things right. I've had to debug, measure performance, and rebuild repeatedly to land on the best approach. I've come to believe that persistence — not giving up even after failure and trying again — is what ultimately lets you overcome a problem and reach your goal.

## 2. Personality

### Strengths

I approach my work with an even-keeled, positive attitude, without strong likes or dislikes about what I'm assigned. I enjoy picking up new knowledge, and I make it a principle to do my best for the company's overall interest regardless of the task. When there's a gap that nobody has picked up, I tend to step in and fill it on my own initiative. For example, even though I was part of the development team, when a proposal needed to be written for a project, I judged that development couldn't start until the bid was won in the first place — so I took the lead on it, researching other companies' sites and drawing on my own experience to propose an open API UI improvement built around adopting Swagger UI, reaching out to vendors offering open-source tooling about commercializing their service, and building a prototype myself.

### Weaknesses

I tend to overthink and hesitate to state my opinion until I feel fully confident about it, which can mean relying on others' judgment or hesitating to assert my own view before a decision is made. To work on this, I try to dig harder for solid grounds to back up my position, then use that as a basis to engage confidently — taking others' opinions into account, but also speaking up and debating actively when needed.

## 3. Motivation

Building on my background in both statistics and IT, I have a strong interest in data analysis and systems development, and more recently I've been expanding into information security to help make IT services more stable and trustworthy.

During my undergraduate years, I worked on a statistical systems project involving data preprocessing, writing and analyzing data queries with SAS SQL, and contributed to a project replacing actual Economic Census survey data with National Tax Service administrative data — an experience that taught me to prioritize data accuracy and reliability.

As a developer, I've worked on a wide range of systems: real-time payment approval, merchant settlement, automated bookkeeping, ERP management accounting, and ITSM. In particular, while integrating systems with multiple client organizations over HTTP/TLS, I came to appreciate how important it is to continuously review and harden the system environment against security policy. That work involved tuning cipher suites, web server configuration, and Java security settings to keep services stable.

I also applied certificate-based mutual authentication (mTLS) while developing a MyData integration, which showed me that security isn't just a technical checkbox — it's directly tied to how much a service can be trusted. That hands-on experience naturally grew into a broader interest in information security.

Since then, I've earned the Information Security Engineer certification, and to build a more systematic foundation, I enrolled in a graduate program in information security, where I scored a 4.14 GPA in my first semester while continuing to deepen my understanding of security theory and practice.

Building on the data analysis and systems development experience I've accumulated, along with my growing understanding of information security, I want to contribute to building IT services that are stable and trustworthy.

## 4. Settlement, Data Integrity, and Large-Scale Data Processing

As a PG (payment gateway) developer, I built settlement batch jobs for payment methods including KakaoPay, Samsung Pay, and direct account transfers. I was directly responsible for reconciling data through transaction matching, settlement matching, and SFTP-based matching, and I operated a daily check batch that automatically verified whether amounts matched and files had been received. I learned firsthand that when numbers don't add up, the basic job is to trace the data flow backward across systems to find the cause.

At Bankware Global, I built an ERP management accounting system for an asset trust company. Management accounting allocates revenue and operating expenses across business units to calculate net profit per unit, so I designed a structure that used hierarchical queries to aggregate amounts by parent/child account code. This was hands-on experience designing a structure where accounting data is accurately allocated and aggregated by department.

For large-scale data processing, I chose the approach based on the situation at hand. When extracting hundreds of thousands of records to Excel, I used MyBatis's RowHandler to stream the data, avoiding OOM issues while keeping extraction stable. In batch environments with limited JVM memory, I used Spring Batch's chunk-based processing to split the work into manageable units and keep memory usage stable. Since large-scale processing is affected not just by application code but also by the DB and OS layer, on a project upgrading an Oracle access control solution — where I also took on a DBA role — I worked with Oracle engineers on the acceptance review. To handle large numbers of sessions stably and efficiently, I configured HugePages and reviewed and applied kernel-level parameters related to session limits. I believe it's important to first understand the constraints you're working within, and then choose the structure that fits them.

## 5. Incident Response

While responsible for a payment notification service, I ran into a recurring zombie process problem. I tried various tuning approaches — scaling out processes, switching thread models, adjusting the JVM heap — but none of them resolved it at the root, and the best I could do was an immediate workaround: pulling the affected process out of the L4 load balancer thanks to our triple-redundant setup. It never reproduced in the dev environment, so I moved on to other work without ever fully identifying the cause.

Looking back, even after I moved teams and time had passed, that unresolved problem stayed on my mind. When I revisited it, I concluded that the root cause was structural: at the time, the company had no framework for managing service daemons, so every time a process needed to be scaled out, a developer had to manually assign it a port. That realization led me to look into pod-based management with Kubernetes, and I pursued the CKAD certification while exploring what a better structural solution would look like on my own.

I had a similar experience while doing a DBA acceptance review on the Oracle access control upgrade project. I had created an index the same way on both the dev and production databases, but only production failed to use the new index. Using an ASH report to pin down the problematic session, I found that the old execution plan was still cached in the shared pool. I never fully identified the exact root cause, but purging that cursor to force a re-parse resolved the issue and the index was used correctly from then on.

I tend not to let go of a problem easily, even one I can't solve right away. Even if I stabilize the service with a workaround first, I believe the effort to find the root cause always has to continue in parallel. Root-causing an issue isn't always easy, but that's not a reason to stop trying.
