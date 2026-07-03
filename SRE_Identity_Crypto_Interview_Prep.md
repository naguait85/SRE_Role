# SRE – Identity & Cryptography Interview Prep

**Role:** Site Reliability Engineer III – Identity & Cryptography
**Client:** Bank of America (Core Technology Infrastructure / Mythos remediation program)
**Location:** Chandler, AZ (Hybrid)

> How to use this guide: Each section goes **Basics → Intermediate → Advanced**. Read the "Talking points" boxes to frame answers around *reliability, security, and compliance* — the three themes this role cares about most. Practice saying answers out loud in 60–90 seconds.

---

## Table of Contents
1. [How to Frame Yourself for This Role](#0)
2. [SRE Fundamentals (SLO / SLI / SLA / Error Budgets)](#1)
3. [Observability (Metrics, Logs, Traces, Alerting)](#2)
4. [Incident Response, On-Call & RCA](#3)
5. [Identity & Access Management (IAM)](#4)
6. [Cryptography Concepts](#5)
7. [Key Management Services (AWS/Azure/GCP) & HSMs](#6)
8. [Secrets Management](#7)
9. [Multi-Cloud & Hybrid](#8)
10. [Linux, Networking & Distributed Systems](#9)
11. [Infrastructure as Code (Terraform, CloudFormation, ARM)](#10)
12. [CI/CD & Automation](#11)
13. [Vulnerability Remediation & Compliance](#12)
14. [Policy-as-Code & Governance](#13)
15. [Scripting (Python, Go, Bash)](#14)
16. [Behavioral / Leadership (STAR)](#15)
17. [Questions to Ask the Interviewer](#16)
18. [Rapid-Fire Cheat Sheet](#17)

---

<a name="0"></a>
## 1. How to Frame Yourself for This Role

This is a **senior/SME SRE** role sitting at the intersection of **reliability engineering + security (identity & crypto) + cloud + compliance**, supporting a large cybersecurity remediation program ("Mythos"). Interviewers will probe three things:

1. **Depth of SRE practice** — Can you define SLOs/SLIs, run error budgets, and improve reliability with data, not vibes?
2. **Identity + cryptography expertise** — Do you truly understand IAM, encryption, key lifecycle, HSMs, and cloud KMS — not just buzzwords?
3. **Operating in a regulated bank** — Can you remediate vulnerabilities, satisfy auditors, and standardize/decommission non-compliant systems without breaking mission-critical platforms?

**Your positioning statement (elevator pitch):**
> "I'm an SRE who specializes in the reliability *and* security of identity and key-management platforms. I define and defend SLOs for authentication and crypto services, automate operations with Terraform and CI/CD, build observability so we catch identity/KMS issues before customers do, and I've led vulnerability remediation and audit-finding closure in regulated environments. My goal is always resilient, compliant systems that engineers can operate safely at scale."

**Golden threads to weave into every answer:**
- Reliability is a *feature* — measured with SLIs/SLOs and paid for with error budgets.
- Security is *not optional* in a bank — least privilege, encryption everywhere, key rotation, auditability.
- Automate toil away — IaC + policy-as-code + CI/CD.
- Blameless culture — learn from incidents, fix systemically.

---

<a name="1"></a>
## 2. SRE Fundamentals (SLO / SLI / SLA / Error Budgets)

### Basics

**Q: What is Site Reliability Engineering (SRE)?**
SRE is a discipline (originated at Google) that applies software engineering to operations problems. The goal is to build and run scalable, reliable systems by treating operations as a software problem — automating toil, measuring reliability quantitatively, and using error budgets to balance velocity vs. stability. Key principles: embrace risk, measure everything (SLIs/SLOs), eliminate toil, automate, and practice blameless postmortems.

**Q: Define SLI, SLO, and SLA. How do they relate?**
- **SLI (Service Level Indicator):** a *quantitative measure* of a service's behavior — e.g., request success rate, latency p99, availability. For identity: "% of authentication requests that succeed in < 300 ms."
- **SLO (Service Level Objective):** a *target* for an SLI over a window — e.g., "99.95% of auth requests succeed over 28 days." Internal goal that drives engineering decisions.
- **SLA (Service Level Agreement):** a *contract* with consequences (financial/credits) if targets are missed. SLAs are usually looser than SLOs (you want to breach the SLO and fix it *before* you breach the SLA).

Relationship: SLI is the *measurement*, SLO is the *goal* built on that measurement, SLA is the *promise* to a customer with penalties. SLO ⊂ (stricter than) SLA.

**Q: What is an error budget?**
Error budget = `100% − SLO`. If your SLO is 99.9% availability, your error budget is 0.1% (~43 minutes/month of allowed downtime). It quantifies acceptable unreliability. When the budget is healthy, teams ship features fast; when it's exhausted, focus shifts to reliability work (freeze risky launches). It turns "how reliable is reliable enough?" into a data-driven, shared agreement between dev and ops.

**Q: What is toil?**
Toil is manual, repetitive, automatable, tactical work that scales linearly with service growth and has no lasting engineering value — e.g., manually rotating certs, hand-applying config, clicking through consoles for access grants. SRE aims to cap toil (Google targets < 50% of time) and automate it away.

### Intermediate

**Q: How would you choose SLIs for an identity/authentication platform?**
Use the "user journey" lens. For an auth service I'd pick:
- **Availability SLI:** ratio of successful auth responses (non-5xx, valid token issued) to total valid requests.
- **Latency SLI:** proportion of token issuance / validation requests under a threshold (e.g., p99 < 300 ms).
- **Correctness SLI:** % of MFA challenges delivered and validated correctly.
- **Freshness/consistency SLI:** directory replication lag (e.g., 99% of group changes propagate < 60s).
For a KMS: **availability** of encrypt/decrypt/sign operations, **latency** of crypto ops, and **durability** of key material. I favor SLIs measured as close to the user as possible (e.g., at the API gateway/load balancer).

**Q: How do you set the *right* SLO target?**
Don't blindly chase 100% (infinitely expensive and unnecessary). Start from user expectations and business impact, look at historical performance, and set an achievable but meaningful target. For mission-critical bank identity, availability is often 99.95–99.99%, but I validate against real dependency chains — you can't promise more than your weakest critical dependency. I iterate: set, measure, review quarterly.

**Q: A team keeps blowing the error budget. What do you do?**
Follow the error-budget *policy* agreed in advance: (1) trigger a reliability freeze on feature launches, (2) redirect engineering to top reliability issues (identified via postmortems/error analysis), (3) root-cause the biggest budget burners, (4) improve testing/rollout safety (canary, progressive delivery). The point is it's a *pre-agreed policy*, not a negotiation during a crisis. If budget is chronically over, the SLO may be unrealistic or the architecture needs investment.

### Advanced

**Q: Explain the "burn rate" alerting approach and why it's better than static threshold alerts.**
Burn rate = how fast you're consuming the error budget relative to the SLO window. A burn rate of 1 means you'd exactly exhaust the budget by end of window; a burn rate of 14.4 means you'd exhaust a 30-day budget in ~2 days. **Multi-window, multi-burn-rate alerts** (from the Google SRE workbook) fire on both a fast-burn (e.g., 2% budget in 1 hour → page) and slow-burn (e.g., 10% in 3 days → ticket). Advantages: high precision + fast detection for real problems, fewer false pages for tiny blips, and alerts are tied to *user-visible impact* rather than arbitrary CPU thresholds.

**Q: How do error budgets change behavior in a regulated bank where you "can't" have downtime?**
Even in a bank, 100% is a myth — the honest question is *how much risk is acceptable and who owns it*. Error budgets make that explicit and shared between security, engineering, and the business. In regulated contexts I'd add: some SLOs are effectively floors mandated by compliance/SLA to regulators, so the budget also protects against *audit* risk, not just customer experience. I'd also track a separate "security SLO" style metric (e.g., time-to-remediate criticals) since a security regression can be worse than a brief outage.

> **Talking point:** Tie SLOs to *both* customer experience and compliance obligations. For identity/crypto, availability of authentication is a systemic dependency — if auth is down, *everything* is down — so its SLO is among the strictest in the enterprise.

---

<a name="2"></a>
## 3. Observability (Metrics, Logs, Traces, Alerting)

### Basics

**Q: What is observability and how is it different from monitoring?**
- **Monitoring** = watching *known* failure modes with pre-defined dashboards/alerts ("is CPU high?"). It answers questions you already knew to ask.
- **Observability** = the property of a system that lets you ask *arbitrary new questions* about its internal state from its external outputs, without shipping new code. It's about debugging *unknown-unknowns*.
Monitoring is a subset/output of an observable system.

**Q: What are the "three pillars" of observability?**
1. **Metrics** — numeric time-series (request rate, error rate, latency, saturation). Cheap, great for alerting and trends.
2. **Logs** — timestamped, discrete event records (structured JSON logs preferred). Great for detailed forensics.
3. **Traces** — the path of a single request across services (spans with timing). Great for latency and dependency analysis in distributed systems.
(Modern view adds **events** and **profiles**; and correlation via consistent IDs/labels ties all three together.)

**Q: What are the "Four Golden Signals"?**
From the Google SRE book — the four things to measure for any user-facing system:
1. **Latency** — time to serve a request (separate success vs error latency).
2. **Traffic** — demand on the system (requests/sec, auths/sec).
3. **Errors** — rate of failed requests.
4. **Saturation** — how "full" the service is (CPU, memory, connection pool, HSM throughput).
(Related: **USE** method — Utilization, Saturation, Errors — for resources; **RED** — Rate, Errors, Duration — for services.)

### Intermediate

**Q: What would an observability framework for an identity/KMS platform look like?**
- **Metrics:** auth success/failure rates by method (password, MFA, cert), token issuance latency, directory query latency and replication lag, KMS encrypt/decrypt/sign latency and throttle counts, HSM utilization and session counts, certificate expiry countdown.
- **Logs:** structured audit logs of every authN/authZ decision and every key operation (who, what key, when, from where, allow/deny) — critical for both debugging *and* compliance/audit.
- **Traces:** distributed tracing across the login flow (gateway → IdP → directory → MFA provider → token service) to pinpoint latency.
- **Alerting:** SLO burn-rate alerts, plus security-specific alerts (spike in auth failures = possible attack, unusual key-usage patterns, cert nearing expiry).
- **Dashboards + SLO reporting** for stakeholders; **correlation IDs** to pivot metrics↔logs↔traces.
Tools I've used/would use: Prometheus + Grafana, ELK/OpenSearch or Splunk, OpenTelemetry for traces, Datadog/Dynatrace, cloud-native (CloudWatch, Azure Monitor, GCP Cloud Operations).

**Q: What makes a *good* alert?**
Actionable, urgent, and tied to user/business impact. Every page should require human intervention *now*. Avoid alerting on causes (high CPU) when you can alert on symptoms (elevated error rate / SLO burn). Include runbook links, clear ownership, and appropriate severity/routing. Continuously prune noisy alerts — alert fatigue is a reliability risk.

**Q: What is cardinality and why does it matter?**
Cardinality = number of unique label/tag combinations in your metrics (e.g., per-user labels explode cardinality). High cardinality blows up storage and query cost in metrics systems like Prometheus. Rule of thumb: keep high-cardinality data (user IDs, request IDs) in logs/traces, not metric labels. This matters a lot for a large bank with millions of identities.

### Advanced

**Q: How do you observe cryptographic operations without leaking secrets?**
Never log key material, plaintext, or full secrets. Log *metadata*: key ID/alias, operation type, caller identity, result, latency — but redact/hash sensitive values. Use structured logging with explicit allow-lists of loggable fields. For HSMs, monitor throughput/session limits and error codes rather than payloads. Ensure log pipelines themselves are encrypted in transit and at rest, access-controlled, and tamper-evident (WORM/immutable storage) for audit integrity.

**Q: How does OpenTelemetry help in a multi-cloud shop?**
OTel is a vendor-neutral standard (APIs, SDKs, and the Collector) for generating and exporting metrics, logs, and traces. In multi-cloud it decouples instrumentation from backends — instrument once, export to any backend (Datadog, Prometheus, cloud-native) via the Collector. This avoids vendor lock-in and gives consistent telemetry across AWS/Azure/GCP/on-prem — essential for the hybrid environment in this JD.

> **Talking point:** For identity/crypto, observability *is* security. The same audit logs you use to debug are the ones auditors and threat hunters rely on. Design telemetry to serve reliability **and** compliance/ITDR simultaneously.

---

<a name="3"></a>
## 4. Incident Response, On-Call & RCA

### Basics

**Q: Walk me through the lifecycle of an incident.**
1. **Detection** — alert fires (SLO burn, error spike) or report comes in.
2. **Triage & severity** — assess impact/scope, assign a severity (Sev1–4).
3. **Response/coordination** — declare incident, assign **Incident Commander (IC)**, communications lead, ops lead; open a war room/bridge.
4. **Mitigation** — stop the bleeding (roll back, failover, disable feature) — restore service *before* full root cause.
5. **Resolution** — confirm recovery, monitor.
6. **Postmortem** — blameless RCA, action items, tracked to completion.

**Q: What is a blameless postmortem and why blameless?**
A written analysis after an incident focusing on *what* happened, timeline, impact, root cause(s), and *systemic* corrective actions — without blaming individuals. Blameless because people act reasonably given the info they had; blame drives hiding of mistakes and kills learning. Assume good intent, fix the system (guardrails, automation, better alerts), not the person.

**Q: What's the difference between mitigation and resolution/root cause?**
Mitigation = fastest action to restore service to users (failover, rollback). Root-cause resolution = fixing the underlying defect so it can't recur. In an incident, **mitigate first, root-cause later.** Reducing MTTR is about fast mitigation, not necessarily immediate deep fixes.

### Intermediate

**Q: Key incident metrics?**
- **MTTD** — Mean Time To Detect.
- **MTTA** — Mean Time To Acknowledge.
- **MTTR** — Mean Time To Recover/Resolve (or Repair/Respond — clarify which).
- **MTBF** — Mean Time Between Failures.
Track these to find where to invest (better detection vs faster mitigation vs preventing recurrence).

**Q: Give an RCA technique.**
**The 5 Whys** — iteratively ask "why" to move from symptom to root cause. Example: Auth outage → why? Token service crashed → why? Ran out of memory → why? Cert cache grew unbounded → why? A cert rotation bug re-fetched without eviction → why? No load test on rotation path. Root cause = missing eviction + test gap; fixes = bounded cache + rotation load test + memory alert. I complement 5 Whys with contributing-factors analysis (rarely a single root cause) and a timeline.

**Q: How do you run on-call sustainably?**
Reasonable rotation size (so no one burns out), clear escalation paths, up-to-date runbooks, actionable alerts only, follow-the-sun if global, comp/time-off for on-call load, and a hard rule that repeated pages generate a ticket to *fix the cause*. Track page volume as a health metric; excessive paging = reliability debt.

### Advanced

**Q: An identity outage is causing enterprise-wide login failures at a bank. Walk me through it.**
1. **Declare Sev1**, assign IC, open bridge, notify stakeholders (this is a systemic dependency — huge blast radius).
2. **Assess scope:** all users or subset? Which region/IdP/directory? Recent change? Check change calendar and deploy history first — most incidents follow a change.
3. **Mitigate:** if a recent config/cert/deploy change, roll it back or failover to a healthy region/replica. If it's a dependency (e.g., HSM/KMS unavailable), fail over to redundant HSM/region. Consider break-glass procedures.
4. **Communicate** at fixed intervals to leadership and downstream app teams.
5. **Restore & verify** via synthetic auth probes.
6. **Postmortem:** blameless RCA, e.g., expired signing cert, and systemic fixes: automated cert renewal + expiry alerts (multi-week lead), staged rollout, chaos test of failover.

**Q: How do you handle a security incident vs a reliability incident?**
They overlap but a security incident (e.g., suspected key compromise, credential-stuffing attack) adds: engage the **security incident response team/CSIRT**, preserve evidence/forensics, follow legal/regulatory breach-notification obligations, and prioritize containment (rotate/revoke keys, disable compromised accounts) over mere availability. In a bank, regulatory reporting timelines are strict. Coordinate SRE + SecOps under a unified command.

> **Talking point:** Emphasize *blameless* culture, *mitigate-first*, and that for identity/crypto every incident has a potential *security* dimension (compromise, key exposure) — so you loop in security by default.

---

<a name="4"></a>
## 5. Identity & Access Management (IAM)

### Basics

**Q: What is IAM?**
IAM = the framework of policies and technology to ensure the *right identities* (users, services, machines) have the *right access* to the *right resources* at the *right time*, for the *right reasons*. It covers **Authentication** (who are you?), **Authorization** (what can you do?), **Administration** (lifecycle of identities/entitlements), and **Audit** (proving it). Often summarized as **AAA**: Authentication, Authorization, Accounting.

**Q: Authentication vs Authorization?**
- **Authentication (AuthN):** verifying identity — proving *you are who you say*. Factors: something you know (password), have (token/phone), are (biometric).
- **Authorization (AuthZ):** determining *what an authenticated identity is allowed to do* — permissions/policies. AuthN always precedes AuthZ.

**Q: What are the common authentication factors and what is MFA?**
Factors: knowledge (password/PIN), possession (hardware token, phone/TOTP, push, FIDO2 key), inherence (biometrics). **MFA (Multi-Factor Authentication)** requires ≥2 factors from *different* categories, dramatically reducing account takeover. **Phishing-resistant MFA** (FIDO2/WebAuthn, PIV/smartcards) is the gold standard because it binds credentials to the origin.

**Q: What are directory services?**
Centralized databases of identities and attributes used for authN/authZ. Examples: **LDAP** (protocol/standard, e.g., OpenLDAP), **Microsoft Active Directory (AD)** (on-prem, uses LDAP + Kerberos), **Microsoft Entra ID / Azure AD** (cloud IdP). They store users, groups, service accounts, and enable single source of truth for identity.

### Intermediate

**Q: Explain OAuth 2.0 vs OIDC vs SAML.**
- **OAuth 2.0:** an **authorization** framework — delegated access via **access tokens** (e.g., "let this app call the API on my behalf"). It's about *access*, not proving identity.
- **OIDC (OpenID Connect):** an **authentication** layer *on top of* OAuth 2.0. Adds an **ID token (JWT)** that asserts *who the user is*. Modern web/mobile SSO standard.
- **SAML 2.0:** older **XML-based** standard for **SSO** and identity federation, common in enterprise/B2B (IdP issues signed SAML assertions to a Service Provider). Still heavily used in banks.
Rule of thumb: OAuth = authorization/API access; OIDC = modern authN/SSO; SAML = enterprise SSO (legacy but pervasive).

**Q: Explain the OAuth 2.0 Authorization Code flow (with PKCE).**
1. App redirects user to the Authorization Server to log in.
2. User authenticates & consents; AS returns a short-lived **authorization code** to the app's redirect URI.
3. App exchanges the code (+ client secret / PKCE verifier) at the token endpoint for an **access token** (and often a **refresh token** + OIDC ID token).
4. App calls the resource API with the access token.
**PKCE (Proof Key for Code Exchange)** protects public clients (SPAs/mobile) from code-interception: the client sends a hashed `code_challenge` up front and the plaintext `code_verifier` at exchange, so a stolen code is useless without the verifier.

**Q: What is a JWT and what are its security considerations?**
**JWT (JSON Web Token)** = compact, signed (JWS) — optionally encrypted (JWE) — token with three parts: header, payload (claims), signature. Claims include `iss`, `sub`, `aud`, `exp`, `iat`. Security: always **verify the signature** and **algorithm** (reject `alg: none`, pin expected algs to avoid alg-confusion attacks), validate `aud`/`iss`/`exp`, keep tokens short-lived, and don't store sensitive data in the (base64, *not encrypted*) payload. Use rotation of signing keys (via JWKS endpoint).

**Q: RBAC vs ABAC vs PBAC?**
- **RBAC (Role-Based):** permissions granted via roles; users get roles. Simple, but "role explosion" at scale.
- **ABAC (Attribute-Based):** decisions based on attributes (user dept, resource sensitivity, time, location, risk). Flexible, fine-grained, great for dynamic/context-aware access.
- **PBAC (Policy-Based):** centralized policies (often ABAC expressed as policy) evaluated by an engine (e.g., OPA). Enables **policy-as-code**.
Banks often layer RBAC for coarse access + ABAC for context-sensitive decisions.

**Q: What is the principle of least privilege and how do you enforce it?**
Grant the minimum access needed to do the job, for the minimum time. Enforce via: role/entitlement reviews (recertification/attestation), **JIT (just-in-time)** and **just-enough** access, **PAM** for privileged accounts, removing standing admin access, automated deprovisioning on role change/offboarding, and regular access analytics to find and prune unused permissions.

### Advanced

**Q: What is PAM and why is it critical in a bank?**
**Privileged Access Management** secures accounts with elevated rights (root, domain admin, service accounts, DB admins). Capabilities: credential vaulting, automatic rotation, session recording, **just-in-time elevation**, and approval workflows. Critical because privileged credentials are the #1 target — compromising one can mean total compromise. Tools: CyberArk, BeyondTrust, HashiCorp Vault (for secrets). Ties directly to reducing blast radius and satisfying auditors.

**Q: What is Zero Trust and how does identity fit?**
Zero Trust = "never trust, always verify" — no implicit trust based on network location. Every access request is authenticated, authorized, and continuously validated based on identity, device posture, and context/risk. **Identity is the new perimeter** — the primary control plane. Components: strong MFA, device trust, micro-segmentation, least-privilege, continuous evaluation (CAE), and per-request policy decisions. Highly relevant to Bank of America's security posture.

**Q: What is ITDR (Identity Threat Detection and Response)?** *(Preferred qualification)*
ITDR is a security discipline focused on detecting and responding to **identity-based threats** — credential theft, privilege escalation, lateral movement, MFA fatigue/bombing, golden/silver ticket attacks, risky OAuth grants, and directory (AD/Entra) attacks. It combines identity posture management (finding misconfigs/excess privilege) with real-time detection (anomalous logins, impossible travel, token replay) and automated response (step-up auth, session revocation, disabling accounts). It's a natural extension of SRE observability into the identity security domain.

**Q: How would you improve reliability of an enterprise directory (AD/LDAP)?**
Multi-master/redundant replication with monitored replication lag, geographically distributed read replicas near consumers, health-checked load balancing, capacity planning for peak auth storms (e.g., Monday-morning logins), caching of frequent lookups, strict change control on schema/GPO changes (a bad change here has enterprise blast radius), synthetic auth monitoring, and tested DR/failover. Treat directory availability as a top-tier SLO because nearly everything depends on it.

**Q: Machine/workload identity — how do services authenticate to each other securely?**
Prefer short-lived, automatically rotated credentials over static secrets: **mTLS** with a service mesh (SPIFFE/SPIRE identities), cloud workload identity (IAM roles for EC2/EKS, Azure Managed Identities, GCP Workload Identity Federation), and OIDC-federated CI/CD (e.g., GitHub Actions → cloud role, no long-lived keys). This eliminates hard-coded secrets and shrinks the attack surface.

> **Talking point:** Show you understand identity as *the* security control plane. Connect IAM reliability (directory uptime, auth latency SLOs) with IAM security (least privilege, PAM, Zero Trust, ITDR). That dual reliability+security framing is exactly what this role wants.

---

<a name="5"></a>
## 6. Cryptography Concepts

### Basics

**Q: Symmetric vs asymmetric encryption?**
- **Symmetric:** same key encrypts and decrypts (e.g., **AES**). Fast; good for bulk data. Challenge = securely sharing the key.
- **Asymmetric (public-key):** a key *pair* — public key encrypts / private key decrypts (and private signs / public verifies). E.g., **RSA, ECC**. Solves key distribution but is slower.
In practice we use **hybrid**: asymmetric to exchange a symmetric session key, then symmetric for the data (this is how TLS works).

**Q: Encryption vs hashing vs encoding?**
- **Encryption:** reversible with a key; provides confidentiality (AES, RSA).
- **Hashing:** one-way, fixed-length digest; provides integrity, not confidentiality (SHA-256). For passwords, use *slow, salted* hashes (bcrypt, scrypt, Argon2).
- **Encoding:** reversible transformation for representation, *no security* (Base64, hex). A common junior mistake is thinking Base64 = encryption.

**Q: What is a digital signature?**
Signer hashes the message and encrypts the hash with their **private key**. Anyone can verify with the **public key** by re-hashing and comparing. Provides **integrity** (not altered), **authenticity** (came from the signer), and **non-repudiation** (signer can't deny it). Basis of code signing, JWT signing, TLS certs.

**Q: What is TLS and how does the handshake work (high level)?**
TLS secures data in transit. Handshake (TLS 1.3 simplified): client & server agree on cipher suite, perform an (EC)DHE key exchange to derive a shared symmetric key with **forward secrecy**, the server proves identity via its **certificate** (chain to a trusted CA), then both switch to symmetric encryption for the session. TLS 1.3 removed legacy/weak options and is faster (1-RTT, optional 0-RTT).

**Q: Encryption at rest vs in transit vs in use?**
- **At rest:** data stored encrypted (disk/DB/object storage) — e.g., AES-256 with keys in a KMS.
- **In transit:** data encrypted while moving over a network (TLS/mTLS).
- **In use:** data encrypted even while being processed — **confidential computing** (secure enclaves like Intel SGX, AWS Nitro Enclaves), homomorphic encryption. Emerging but increasingly relevant.

### Intermediate

**Q: Explain envelope encryption.**
Instead of encrypting large data directly with a master key, you: (1) generate a **Data Encryption Key (DEK)**, (2) encrypt the data with the DEK (fast symmetric), (3) encrypt the DEK with a **Key Encryption Key (KEK)** held in the KMS/HSM, and (4) store the encrypted DEK alongside the ciphertext. To decrypt, the KMS decrypts the DEK, then you decrypt the data. Benefits: the master key never leaves the KMS/HSM, you limit KMS calls (scales for big data), and you can rotate the KEK without re-encrypting all data. This is how AWS KMS, Azure Key Vault, and GCP KMS work under the hood.

**Q: What is key lifecycle management?**
Managing a key through all stages: **generation** (with strong entropy, ideally in an HSM) → **distribution/provisioning** → **usage** (with access controls) → **rotation** (regularly and on suspected compromise) → **revocation** → **archival** (to decrypt old data) → **destruction** (crypto-shredding). Good lifecycle = defined crypto-periods, automated rotation, separation of duties, and full audit logging of every key operation.

**Q: What is key rotation and why rotate?**
Replacing a cryptographic key with a new one periodically or on-demand. Reasons: limit the amount of data exposed if a key is compromised, meet compliance mandates (e.g., PCI-DSS), and reduce cryptanalysis risk. With envelope encryption you often rotate the **KEK** (cheap — re-encrypt only DEKs) rather than re-encrypting all data. Automate it; manual rotation is toil and error-prone.

**Q: What is a certificate and what is PKI?**
A **digital certificate** (X.509) binds a public key to an identity, signed by a trusted **Certificate Authority (CA)**. **PKI (Public Key Infrastructure)** = the CAs, registration authorities, policies, and processes that issue, manage, and revoke certificates. Includes trust chains (root → intermediate → leaf), revocation (**CRL**, **OCSP**), and lifecycle management. Certificate *expiry* is a classic outage cause — automate renewal (ACME/Let's Encrypt, internal CA + cert-manager, Venafi).

**Q: What is a nonce / IV, and why does it matter?**
An **IV (Initialization Vector)** or **nonce** ensures the same plaintext encrypts to different ciphertext each time, preventing pattern leakage. It must be unique (and for some modes, unpredictable). **Never reuse a nonce with the same key** in AES-GCM — it catastrophically breaks confidentiality/integrity. Use authenticated modes like **AES-GCM** or **ChaCha20-Poly1305** (AEAD) that provide confidentiality *and* integrity.

### Advanced

**Q: What is an HSM and why use one?**
A **Hardware Security Module** is a tamper-resistant, hardened device that generates, stores, and uses cryptographic keys such that the **private key material never leaves the device in plaintext**. Crypto operations happen *inside* the HSM. Benefits: strong key protection, high-assurance random number generation, and certifications (**FIPS 140-2/140-3 Level 3**, Common Criteria) required by banking/PCI. Access via **PKCS#11**, KMIP, JCE, or CNG. Examples: **Thales Luna / payShield**, AWS CloudHSM, Azure Dedicated HSM, Entrust nShield.

**Q: HSM vs cloud KMS — when do you need a dedicated HSM?**
Cloud KMS (AWS KMS, Azure Key Vault standard, GCP KMS) is multi-tenant, managed, and backed by HSMs but you share infrastructure. Use a **dedicated HSM / CloudHSM / Key Vault Managed HSM** when you need: single-tenant isolation, **FIPS 140-2 Level 3** assurance, full control of key material, **BYOK/HYOK** (bring/hold your own key), or specific compliance (PCI-DSS PIN/EMV, regulatory mandates). Trade-off: more control and assurance vs more operational burden and cost. Banks frequently need dedicated HSMs for payment and root-key use cases.

**Q: What is Thales CipherTrust Manager (CTM)?** *(Preferred qualification)*
CipherTrust Manager is Thales's centralized platform for **enterprise key management and data protection** — it manages keys, policies, and access across databases, files, cloud, and applications, and supports **KMIP**, BYOK to cloud KMS, tokenization, and transparent data encryption. It's often paired with **Thales Luna HSMs** as the hardware root of trust. Value: one control plane for keys across a hybrid/multi-cloud estate, centralized policy and audit — exactly the standardization goal in this JD. If asked and you lack hands-on: be honest, then map it to concepts you *do* know (KMIP, centralized KMS, HSM-backed root of trust, BYOK).

**Q: What is KMIP?**
**Key Management Interoperability Protocol** — an OASIS standard for communication between key-management clients and servers (create, retrieve, rotate, destroy keys). It lets heterogeneous products (storage arrays, databases, HSMs, KMS) interoperate with a central key manager like CipherTrust — key to avoiding vendor lock-in in a multi-vendor bank.

**Q: What is crypto-agility and post-quantum cryptography (PQC)?**
- **Crypto-agility:** designing systems so algorithms/keys can be swapped without re-architecting — abstract crypto behind interfaces, inventory where crypto is used, avoid hard-coded algorithms. Vital for responding to newly-broken algorithms.
- **PQC:** algorithms resistant to quantum computers (which threaten RSA/ECC via Shor's algorithm). NIST standardized **ML-KEM (Kyber)** for key exchange and **ML-DSA (Dilithium)** / SLH-DSA for signatures (2024). Banks are starting **"harvest now, decrypt later"** risk assessments and crypto inventories to prepare for migration. Showing awareness here signals senior-level thinking.

**Q: What is separation of duties / dual control / M-of-N in key management?**
No single person should control a critical key end-to-end. **Dual control** requires two people to perform sensitive operations; **split knowledge / M-of-N (quorum)** requires k of n key-holders to reconstruct or authorize use of a master key (Shamir's Secret Sharing). Required by PCI-DSS and standard for HSM root/master keys. Reduces insider risk.

> **Talking point:** Demonstrate you understand *why* — HSMs and separation of duties exist to protect the *root of trust* and satisfy regulators. Envelope encryption + centralized KMS + HSM-backed root is the standard bank pattern; be ready to draw it.

---

<a name="6"></a>
## 7. Key Management Services (AWS / Azure / GCP) & HSMs

### Basics

**Q: What does a cloud KMS actually do?**
A managed service to create and control cryptographic keys and perform crypto operations (encrypt, decrypt, sign, verify, generate data keys) while keeping key material protected in HSMs. It integrates with other cloud services for encryption at rest, enforces access via IAM policies, rotates keys, and logs every use for audit. You call the KMS API; keys don't leave it.

**Q: Compare AWS KMS, Azure Key Vault, and GCP KMS at a high level.**

| Capability | AWS KMS | Azure Key Vault | GCP Cloud KMS |
|---|---|---|---|
| Keys | CMK/KMS keys (symmetric + asymmetric) | Keys (RSA/EC) | CryptoKeys + versions |
| Secrets | Secrets Manager (separate) | Secrets (built-in) | Secret Manager (separate) |
| Certs | ACM (separate) | Certificates (built-in) | Certificate Manager / CAS |
| HSM tier | CloudHSM / KMS (FIPS 140-2 L3 HSM-backed) | Managed HSM / Dedicated HSM | Cloud HSM / Cloud EKM |
| BYOK | Import key material | BYOK | Import + EKM (external) |
| Access control | IAM + key policies + grants | Azure RBAC + access policies | Cloud IAM |
| Audit | CloudTrail | Azure Monitor/Log Analytics | Cloud Audit Logs |

All three support envelope encryption, automatic rotation, and per-operation audit logging.

**Q: What is a CMK / customer-managed key vs a cloud-managed key?**
- **Cloud/AWS-managed key:** the provider creates/rotates it for a service; less control.
- **Customer-managed key (CMK):** you create, control policy, rotation, and can disable/delete it — needed for compliance, key separation, and the ability to *revoke access to data* by disabling the key. Banks almost always use customer-managed keys.

### Intermediate

**Q: What's the difference between BYOK, HYOK, and EKM?**
- **BYOK (Bring Your Own Key):** you generate key material (often in your on-prem HSM) and import it into the cloud KMS. You control generation; cloud stores/uses it.
- **HYOK (Hold Your Own Key):** the key stays in *your* control (on-prem HSM) and the cloud calls out to it — cloud never holds the key.
- **EKM (External Key Manager, GCP) / External Key Store (AWS XKS):** cloud KMS delegates crypto operations to an external key manager (e.g., Thales CipherTrust) you control. Maximizes control/compliance; adds latency and an availability dependency on your external HSM.

**Q: How does AWS KMS envelope encryption work with `GenerateDataKey`?**
`GenerateDataKey` returns both a **plaintext DEK** and an **encrypted DEK** (encrypted under your KMS key). You encrypt data locally with the plaintext DEK, then discard the plaintext DEK and store the encrypted DEK with the ciphertext. To decrypt, call `Decrypt` on the encrypted DEK to recover the plaintext DEK, then decrypt the data. This keeps the KMS master key in the HSM and scales to large data.

**Q: What is a KMS key policy / grant and how does access control work?**
In AWS, a **key policy** is the primary resource policy on the key; **grants** provide temporary, fine-grained delegation; and IAM policies also apply — access requires alignment across them. Azure uses **RBAC** (or legacy access policies) on the vault/key. GCP uses **Cloud IAM** roles on the key/keyring. Best practice: least privilege, separate *key administrators* (manage keys) from *key users* (use keys) — a form of separation of duties — and never let one role both administer and use a critical key.

**Q: How do you rotate keys in cloud KMS without breaking decryption of old data?**
Cloud KMS keeps **old key versions** for decryption while new versions handle new encryption. With envelope encryption you rotate the KEK and only DEKs may need re-wrapping; the ciphertext references the key version used. AWS KMS automatic rotation keeps prior backing material transparently. Plan re-encryption jobs for hard rotation (compromise) and always retain the ability to decrypt legacy ciphertext until it's migrated.

### Advanced

**Q: How do you make a KMS/HSM highly available and low-latency across regions?**
- **Multi-region keys** (AWS multi-Region keys, or replicate key material via BYOK to each region's KMS) so encrypt/decrypt works region-locally.
- **HSM clustering** with redundancy across AZs (CloudHSM cluster, Luna HA groups) and load balancing.
- Cache DEKs client-side (with bounded TTL) to reduce KMS calls and tolerate brief KMS blips (e.g., AWS Encryption SDK data key caching).
- Monitor KMS **throttling/quota** and request limit increases proactively — KMS throttling is a real outage cause.
- Define SLOs for crypto op latency/availability and test regional failover (game days).

**Q: What are the failure modes of relying on KMS for everything, and how do you mitigate?**
- **Availability dependency:** if KMS is down, apps can't decrypt → outage. Mitigate with data-key caching, multi-region keys, graceful degradation.
- **Throttling under load:** batch, cache, and request quota increases.
- **Accidental key deletion/disable:** enable deletion protection, mandatory waiting periods, and alerts; a disabled key instantly locks out data.
- **EKM/XKS external dependency:** external HSM outage breaks cloud crypto — need HA external HSMs and monitoring.
- **Cost/latency** of per-op calls at scale → envelope encryption + caching.

**Q: How would you approach standardizing key management across a multi-cloud bank (per this JD)?**
1. **Inventory** all keys, HSMs, and crypto usage across clouds/on-prem (find non-compliant/shadow crypto).
2. Establish a **central control plane** (e.g., CipherTrust Manager + KMIP) with a common policy, naming, tagging, rotation, and audit standard.
3. Define **enterprise standards**: approved algorithms/key lengths, rotation periods, separation of duties, FIPS-validated HSM roots of trust.
4. Enforce via **policy-as-code** and IaC modules so keys are provisioned compliantly by default.
5. **Migrate/decommission** non-compliant solutions (the JD's "eliminate non-compliant solutions"), with clear cutover and dual-run for safety.
6. Centralize **observability & audit** of all key operations for compliance.

> **Talking point:** For each cloud KMS, know the analogous concepts (key, key version, HSM tier, BYOK/EKM, IAM model, audit log). Interviewers love when you map concepts across providers and connect to *availability* (multi-region, caching) and *compliance* (FIPS, separation of duties, audit).

---

<a name="7"></a>
## 8. Secrets Management

### Basics

**Q: What is secrets management and why not just use env vars / config files?**
Secrets management is the secure storage, access control, rotation, and auditing of sensitive credentials (DB passwords, API keys, certs, tokens). Env vars / config files / code are bad because secrets get committed to git, logged, shared broadly, never rotated, and aren't audited. A dedicated secrets manager provides encryption at rest, fine-grained access, automatic rotation, dynamic secrets, and audit trails.

**Q: Name common secrets management tools.**
**HashiCorp Vault**, **AWS Secrets Manager**, **Azure Key Vault (secrets)**, **GCP Secret Manager**, **CyberArk** (privileged), CipherTrust. Kubernetes: External Secrets Operator, Sealed Secrets, or CSI Secrets Store driver.

### Intermediate

**Q: What are dynamic secrets (HashiCorp Vault)?**
Instead of a long-lived static credential, Vault generates **short-lived, on-demand credentials** for a backend (DB, cloud, SSH) when requested, with a lease/TTL, and automatically revokes them on expiry. Benefits: no standing credentials to steal, automatic rotation, unique per-consumer (great for auditing/traceability). This dramatically shrinks the blast radius of a leak.

**Q: How do you handle secret rotation without downtime?**
Support **two active versions** during rotation (current + previous) so consumers using the old secret keep working until they refresh; use versioned secrets and staged rollout. For DBs, Vault/Secrets Manager rotation Lambdas create the new credential, verify, then promote. Applications should fetch secrets at runtime (not bake at build) and handle refresh/retry on auth failure.

**Q: How do you keep secrets out of source control and CI/CD?**
- Pre-commit hooks + scanners (**gitleaks, truffleHog, detect-secrets**) and GitHub secret scanning.
- Never store secrets in the repo; reference them from a secrets manager at runtime.
- In CI/CD, use **OIDC federation** to get short-lived cloud credentials instead of storing long-lived keys; mask secrets in logs; scope them to specific jobs/environments.
- Rotate any leaked secret immediately (treat as compromised).

### Advanced

**Q: How do Vault and cloud KMS work together (the "trusted broker" pattern)?**
KMS/HSM protects the **root/unseal key** and provides the crypto root of trust; Vault provides the **secrets workflow** (dynamic secrets, leasing, policy, audit) on top. E.g., Vault **auto-unseal** using AWS KMS/Azure Key Vault, and Vault's **Transit** engine offers "encryption as a service" so apps never touch keys. This layering gives HSM-grade key protection plus rich secret lifecycle management — a common bank pattern.

**Q: How do workloads authenticate to a secrets manager securely (no chicken-and-egg secret-zero problem)?**
Use **platform identity** as the trust anchor: Vault auth methods like **Kubernetes** (service-account JWT), **AWS IAM/EC2**, **Azure Managed Identity**, or **JWT/OIDC** — the workload proves who it is via an identity the platform already vouches for, then receives a scoped token. This solves "secret zero" — no bootstrap secret to distribute.

> **Talking point:** Emphasize *short-lived/dynamic over static*, *fetch at runtime not build time*, *rotate on leak*, and *platform identity to avoid secret-zero*. This ties secrets management back to Zero Trust and least privilege.

---

<a name="8"></a>
## 9. Multi-Cloud & Hybrid

### Basics

**Q: Multi-cloud vs hybrid cloud?**
- **Multi-cloud:** using more than one public cloud provider (AWS + Azure + GCP), e.g., for resilience, avoiding lock-in, or best-of-breed services.
- **Hybrid cloud:** combining on-prem/private data center with public cloud, integrated so workloads/data can span both.
This JD explicitly involves both — enterprise identity/crypto spanning on-prem HSMs/AD and multiple public clouds.

**Q: Why do banks run multi-cloud/hybrid, and what's hard about it?**
Reasons: regulatory data residency, resilience/avoiding single-vendor risk, legacy on-prem systems (mainframe, HSMs), and gradual cloud migration. Hard parts: **identity federation** across clouds, **consistent security/policy**, **key management** across providers, networking/latency, observability across silos, and differing IAM models/APIs.

### Intermediate

**Q: How do you achieve consistent identity across clouds?**
Federate to a central IdP (Entra ID/Okta/Ping) via **SAML/OIDC**; use **workload identity federation** so workloads in one cloud assume roles in another without static keys; centralize directory as source of truth and sync/federate; enforce common RBAC/ABAC policies. Goal: one identity fabric, consistent MFA and conditional access everywhere.

**Q: How do you manage keys consistently across multi-cloud?**
Central control plane (CipherTrust/KMIP) or a consistent pattern per cloud with common standards; use **BYOK/EKM** to keep a single root of trust; abstract crypto behind an internal "encryption service" API so app teams get a uniform interface regardless of the underlying cloud KMS. Standardize rotation, tagging, and audit.

### Advanced

**Q: How do you design observability across a hybrid/multi-cloud estate?**
Standardize on **OpenTelemetry** for vendor-neutral instrumentation; aggregate into a central platform (Splunk/Datadog/Grafana) or federate per-cloud tools with a single pane; use consistent tagging/labels, correlation IDs, and a unified SLO framework so a login flow spanning on-prem AD → cloud IdP → cloud KMS can be traced end-to-end. Normalize logs to a common schema for cross-environment audit.

**Q: What are the reliability challenges unique to hybrid identity/crypto and how do you address them?**
- **Cross-site latency/partition:** on-prem HSM calls from cloud add latency and a failure dependency → cache, regionalize, and design for graceful degradation.
- **Network reliability:** VPN/Direct Connect/ExpressRoute links are SPOFs → redundant links, monitored, with failover.
- **Time/clock sync:** crypto and Kerberos are time-sensitive → reliable NTP.
- **Split-brain in directories:** careful replication topology and conflict handling.
Define SLOs that account for the whole hybrid path, and test failover between on-prem and cloud.

> **Talking point:** Stress *consistency* (identity fabric, common key standards, unified observability) and *managing cross-environment dependencies* (latency, network SPOFs, graceful degradation). That's the senior multi-cloud reliability mindset.

---

<a name="9"></a>
## 10. Linux, Networking & Distributed Systems

### Basics — Linux

**Q: How do you troubleshoot high CPU / memory / load on a Linux host?**
- **CPU:** `top`/`htop`, `mpstat`, `pidstat` — find the hot process; check `%us` vs `%sy` vs `%wa` (user vs kernel vs IO wait).
- **Memory:** `free -m`, `vmstat`, `/proc/meminfo`; check for OOM in `dmesg`/journald; look at swap usage.
- **Load average:** compare to core count; high load + high IO wait = disk bottleneck (`iostat`, `iotop`).
- **Disk:** `df -h` (space), `du`, `iostat -x`.
- **Systematically:** the **USE method** — for each resource check Utilization, Saturation, Errors.

**Q: How do you check what's listening on a port / debug connectivity?**
`ss -tulpn` (or `netstat -tulpn`) for listening sockets; `curl -v` / `telnet` / `nc -zv host port` for reachability; `ping` (ICMP), `traceroute`/`mtr` for path; `dig`/`nslookup` for DNS; `tcpdump`/`wireshark` for packet-level; check `iptables`/`nftables`/security groups for filtering.

**Q: How do you investigate a service that won't start (systemd)?**
`systemctl status svc`, `journalctl -u svc -e`, check config syntax, permissions, ports in use, dependencies (`systemctl list-dependencies`), and SELinux/AppArmor denials (`ausearch`/`dmesg`).

**Q: Key Linux security/permissions concepts?**
File permissions (rwx, `chmod`/`chown`), ownership, `sudo`/least privilege, SSH key-based auth (disable password/root login), SELinux/AppArmor MAC, and file integrity. For this role: securing keys/certs with strict perms (e.g., `600`, owned by service user), and no secrets world-readable.

### Basics — Networking

**Q: Walk through what happens when you type a URL and hit enter.**
DNS resolution → TCP handshake (SYN/SYN-ACK/ACK) → TLS handshake → HTTP request → server processes (possibly through load balancer, app, DB) → HTTP response → render. Great for showing end-to-end understanding; interviewers often drill into any layer.

**Q: TCP vs UDP?**
- **TCP:** connection-oriented, reliable, ordered, flow/congestion control (HTTP, TLS, databases).
- **UDP:** connectionless, no delivery guarantee, low overhead (DNS, DHCP, streaming, QUIC/HTTP3 built on UDP). Choose based on reliability vs latency needs.

**Q: Explain DNS and common DNS issues.**
DNS maps names → IPs via a hierarchy (root → TLD → authoritative). Record types: A/AAAA, CNAME, MX, TXT, SRV. Issues: TTL/caching causing stale records after cutover, propagation delays, misconfigured records, DNS as a SPOF/outage cause. For identity, DNS/SRV records locate directory/KDC servers — DNS failures break auth.

### Intermediate — Distributed Systems

**Q: Explain the CAP theorem.**
In a distributed system you can guarantee only two of **C**onsistency, **A**vailability, **P**artition tolerance — and since network partitions *will* happen, you really choose between **C and A** during a partition. **CP** systems (e.g., strongly-consistent stores) refuse requests to stay consistent; **AP** systems stay available but may serve stale data. Directories often favor availability with eventual consistency (replication lag), which matters for identity correctness.

**Q: Idempotency — what is it and why does it matter?**
An operation is idempotent if performing it multiple times yields the same result as once (e.g., PUT, DELETE). Critical for reliable distributed systems: retries (after timeouts/network blips) won't cause duplicate side effects. Use idempotency keys for non-idempotent operations. Important for automation and key operations (don't double-rotate).

**Q: What are retries, backoff, jitter, and circuit breakers?**
- **Retries** handle transient failures — but naive retries cause **retry storms**.
- **Exponential backoff** increases wait between retries; **jitter** randomizes it to avoid thundering-herd synchronization.
- **Circuit breaker** stops calling a failing dependency after a threshold, "opening" to fail fast and let it recover, then "half-open" to test. Prevents cascading failure — essential when depending on KMS/HSM/directory.

**Q: What is a cascading failure and how do you prevent it?**
One component's failure overloads others (e.g., a slow directory causes auth timeouts, retries pile up, thread pools exhaust, everything falls over). Prevent with: timeouts, circuit breakers, load shedding, bulkheads (isolate resource pools), backpressure, autoscaling, and graceful degradation. Bound blast radius so an identity slowdown doesn't take down the enterprise.

### Advanced

**Q: How do you design a highly available authentication service?**
- **Stateless** auth nodes behind health-checked load balancers, autoscaled.
- **Redundancy** across AZs/regions; active-active where possible.
- **Redundant dependencies:** replicated directories, multi-region KMS, HSM HA groups.
- **Timeouts + circuit breakers** on every dependency; **caching** of tokens/keys/directory lookups with bounded TTL.
- **Graceful degradation:** e.g., cached authZ decisions if the policy engine is briefly unavailable.
- **Rate limiting** to survive auth storms/attacks.
- **SLOs + burn-rate alerts + tested DR**. Remove SPOFs; test failure with chaos engineering.

**Q: What is chaos engineering?**
Deliberately injecting failure (kill instances, add latency, sever a region, expire a cert in staging) to validate resilience assumptions *before* real incidents. Run controlled "game days" with a hypothesis and blast-radius limits. For this role: test HSM failover, KMS throttling, directory replica loss, cert-expiry handling.

> **Talking point:** For distributed systems, always mention *timeouts + backoff/jitter + circuit breakers + graceful degradation*. For Linux, show a *systematic* method (USE) rather than random commands. Tie everything to protecting the identity/crypto critical path.

---

<a name="10"></a>
## 11. Infrastructure as Code (Terraform, CloudFormation, ARM)

### Basics

**Q: What is Infrastructure as Code and why use it?**
IaC = defining and provisioning infrastructure through machine-readable definition files instead of manual clicking. Benefits: **repeatability/consistency**, **version control** (git history, PR review, rollback), **automation** (CI/CD), **documentation-as-code**, drift detection, and disaster recovery (rebuild from code). Eliminates snowflake servers and configuration toil.

**Q: Declarative vs imperative IaC?**
- **Declarative** (Terraform, CloudFormation, ARM/Bicep): you describe the *desired end state*; the tool figures out how to get there. Idempotent.
- **Imperative** (scripts, some Ansible usage): you specify the *steps*. Terraform is declarative — you say "I want 3 instances," not "create an instance" three times.

**Q: Terraform vs CloudFormation vs ARM?**
- **Terraform** (HashiCorp): cloud-agnostic, HCL, huge provider ecosystem, works across AWS/Azure/GCP/on-prem — ideal for multi-cloud (this JD).
- **CloudFormation:** AWS-native (JSON/YAML), deep AWS integration, managed state.
- **ARM templates / Bicep:** Azure-native. Bicep is the friendlier DSL over ARM JSON.
For a multi-cloud bank, Terraform is usually the common denominator, sometimes with cloud-native tools for provider-specific features.

**Q: What is Terraform state and why does it matter?**
The **state file** maps your config to real-world resources and tracks metadata. It's how Terraform knows what exists and computes diffs. It can contain **secrets**, so store it in a secure **remote backend** (S3+DynamoDB lock, Terraform Cloud, Azure Storage) with encryption, access control, and **state locking** to prevent concurrent corruption. Never commit state to git.

### Intermediate

**Q: What is `terraform plan` vs `apply`, and how do you review changes safely?**
`plan` shows the diff (what will be created/changed/destroyed) without making changes; `apply` executes it. Safe workflow: PR contains code → CI runs `plan` and posts the diff for review → approval → `apply` (often gated/manual for prod). Watch for **destroy/replace** operations, especially on stateful resources (a KMS key or directory!) — those need extra scrutiny.

**Q: How do you keep Terraform DRY and reusable?**
Use **modules** (reusable, versioned components), input variables/outputs, workspaces or separate state per environment, and a module registry. For an enterprise: build **golden/compliant modules** (e.g., a "KMS key" module that bakes in rotation, tagging, policy, audit) so teams provision correctly *by default*.

**Q: How do you manage secrets in IaC?**
Never hard-code secrets in `.tf` files or commit them. Pull from a secrets manager/KMS at apply time (Vault provider, data sources), mark variables `sensitive`, secure the state backend (state can leak secrets), and use short-lived credentials (OIDC) for the pipeline rather than static cloud keys.

**Q: What is configuration drift and how do you detect/handle it?**
Drift = real infrastructure diverging from code (manual console changes, out-of-band fixes). Detect with scheduled `terraform plan`/drift detection or tools like driftctl. Handle by re-applying code (reconcile) and *preventing* future drift via guardrails (restrict console write access, require changes through pipeline). Drift is a compliance risk in a bank.

### Advanced

**Q: How do you structure Terraform for a large regulated enterprise?**
- **Layered/modular:** networking, identity, KMS, apps in separate state to limit blast radius.
- **Per-environment isolation** (dev/stage/prod) with separate state and credentials.
- **Golden modules** enforcing standards (encryption, tagging, rotation).
- **Policy-as-code gates** (Sentinel/OPA/Checkov) in CI to block non-compliant changes before apply.
- **CI/CD** with plan-review-approve, remote encrypted state + locking, OIDC creds.
- **Least-privilege pipeline roles** and full audit of applies.
This directly supports the JD's "automate operations with IaC" and "enforce enterprise standards."

**Q: How do you use IaC to remediate vulnerabilities at scale?**
Codify the fix once in a module/config (e.g., enforce TLS 1.2+, disable weak ciphers, enable key rotation, restrict security groups), then roll it across the estate via pipelines. Combine with drift detection so the fix stays applied, and policy-as-code so regressions are blocked. This turns one-off manual remediation into scalable, auditable, permanent fixes — exactly the Mythos remediation-at-scale goal.

> **Talking point:** Frame IaC as the vehicle for *standardization, compliance-by-default, and scalable remediation*. Golden modules + policy-as-code gates + secure remote state is the enterprise pattern. Call out extra caution on destroy/replace of stateful crypto/identity resources.

---

<a name="11"></a>
## 12. CI/CD & Automation

### Basics

**Q: What is CI/CD?**
- **CI (Continuous Integration):** developers merge frequently to a shared branch; each change triggers automated build + tests to catch issues early.
- **CD (Continuous Delivery/Deployment):** every validated change is automatically prepared for release (Delivery = ready, deploy is a manual gate) or automatically released to prod (Deployment). Goal: fast, safe, repeatable releases with less manual toil.

**Q: What are the typical stages of a pipeline?**
Source (trigger on commit/PR) → Build → Unit tests → Static analysis/**SAST** + dependency/secret scanning → Package/artifact → Deploy to staging → Integration/**DAST** tests → Approval gate → Deploy to prod → Post-deploy verification/smoke tests + monitoring. Rollback on failure.

**Q: Name CI/CD tools.**
Jenkins, GitLab CI, GitHub Actions, Azure DevOps, CircleCI, Argo CD / Flux (GitOps), Spinnaker, Tekton. For IaC: Terraform Cloud/Atlantis.

### Intermediate

**Q: Compare deployment strategies: blue-green, canary, rolling.**
- **Rolling:** replace instances gradually; simple, but mixed versions during rollout.
- **Blue-green:** stand up a full new (green) environment, switch traffic all at once, keep blue for instant rollback. Fast rollback, but double resources.
- **Canary:** route a small % of traffic to the new version, watch SLIs, then progressively increase. Best for limiting blast radius; needs good observability + automated rollback on SLO regression.
For critical identity services, canary with automated rollback tied to SLOs is safest.

**Q: What is GitOps?**
Git is the single source of truth for declarative infra/app config; an agent (Argo CD/Flux) continuously reconciles the cluster/infra to match git. Benefits: full audit trail (every change is a reviewed commit), easy rollback (git revert), and drift correction. Strong fit for regulated environments needing change traceability.

**Q: How do you secure a CI/CD pipeline (supply chain security)?**
- **Secrets:** OIDC federation for short-lived cloud creds; no static keys; secret scanning.
- **Least privilege** pipeline identities scoped per environment.
- **Scan** dependencies (SCA), code (SAST), containers, and IaC (Checkov/tfsec) as gates.
- **Sign artifacts** (Sigstore/cosign) and verify provenance (SLSA) to prevent tampering.
- **Protect the pipeline itself** (branch protection, required reviews, immutable build agents).
- **Audit** every deploy. Supply-chain attacks (SolarWinds-style) make this a top bank concern.

### Advanced

**Q: How would you build "policy-as-code" and compliance checks into pipelines?**
Add gates that run **OPA/Conftest, Sentinel, or Checkov/tfsec** against IaC and configs to enforce enterprise standards (encryption required, approved algorithms, tagging, no public exposure, rotation enabled). Non-compliant changes fail the pipeline *before* reaching prod ("shift left"). Generate compliance evidence/reports automatically from these checks for auditors. This automates the JD's "policy-as-code and compliance automation."

**Q: How do you automate operational tasks (reduce toil) as an SRE?**
Identify high-frequency toil (cert renewals, key rotation, access grants, patching), then codify it: scheduled automation (Lambda/Functions/cron), self-service tooling with guardrails, auto-remediation runbooks triggered by alerts, and IaC for provisioning. Measure toil reduction. For this role specifically: automated cert lifecycle, automated key rotation, automated compliance scanning, and auto-remediation of common findings.

> **Talking point:** Connect CI/CD to *safety* (canary + auto-rollback on SLO), *security* (OIDC, signing, scanning), and *compliance* (policy-as-code gates producing audit evidence). SRE = engineering away toil through automation.

---

<a name="12"></a>
## 13. Vulnerability Remediation & Compliance

### Basics

**Q: What is vulnerability management?**
A continuous process: **discover** assets → **scan/identify** vulnerabilities → **prioritize** by risk → **remediate** (patch/config/mitigate) → **verify** → **report**. Not just scanning — it's the full lifecycle of reducing exposure, with SLAs on time-to-remediate by severity.

**Q: What is CVE and CVSS?**
- **CVE (Common Vulnerabilities and Exposures):** a unique ID for a publicly known vulnerability (e.g., CVE-2021-44228 = Log4Shell).
- **CVSS (Common Vulnerability Scoring System):** a 0–10 severity score (Base/Temporal/Environmental metrics) → None/Low/Medium/High/Critical. Used to prioritize.
Modern prioritization also uses **EPSS** (probability of exploitation) and **CISA KEV** (known exploited) — patch what's actually being exploited first, not just the highest CVSS.

**Q: How do you prioritize which vulnerabilities to fix first?**
Risk = severity (CVSS) × exploitability (EPSS/KEV/exploit availability) × asset exposure/criticality (internet-facing? holds sensitive data? part of the identity/crypto critical path?). Fix actively-exploited, internet-facing, critical-asset vulns first. Align to regulatory SLAs (e.g., criticals within X days).

### Intermediate

**Q: Walk me through how you'd lead a large vulnerability remediation effort (à la Mythos).**
1. **Aggregate & normalize** findings from all scanners/the AI platform; dedupe.
2. **Prioritize** by risk (CVSS + EPSS/KEV + asset criticality/exposure).
3. **Assign ownership** to the right teams with clear SLAs.
4. **Remediate at scale** via automation/IaC (patch pipelines, config-as-code) rather than one-off manual fixes.
5. **Handle exceptions/compensating controls** where immediate patching isn't possible (risk acceptance with sign-off).
6. **Verify** with re-scans; track burn-down.
7. **Report** progress/metrics to leadership and auditors; feed lessons back to prevent recurrence (e.g., harden golden images).
Emphasize *reliability*: patching mission-critical identity/crypto systems needs staged rollout, testing, and rollback so remediation doesn't cause an outage.

**Q: How do you patch mission-critical systems without downtime?**
Redundancy + rolling/canary patching, test in non-prod first, maintenance windows for the riskiest changes, HA failover so nodes patch one at a time, automated rollback, and immutable infrastructure (replace rather than patch-in-place where possible). Balance urgency of the vuln against change risk using the error budget.

**Q: What are audit/compliance findings and how do you resolve them?**
Findings = gaps identified by internal/external auditors or automated compliance scans against a control framework. Resolution: understand the control intent, implement the fix (technical or process), gather **evidence** (logs, configs, screenshots, IaC), document remediation, and get auditor sign-off. Best practice: automate evidence collection and use policy-as-code to *prevent* the finding from recurring (continuous compliance).

### Advanced

**Q: What compliance frameworks are relevant to a bank, and how do they touch identity/crypto?**
- **PCI-DSS** (payment card data — strong crypto, key management with dual control/split knowledge, MFA, access control).
- **SOX** (financial reporting integrity — access controls, change management, segregation of duties).
- **GLBA / FFIEC** guidance (financial data protection).
- **NIST 800-53 / CSF**, **ISO 27001** (control frameworks).
- **SOC 2** (for service providers).
Identity/crypto controls map heavily to these: encryption standards, key lifecycle, least privilege, MFA, audit logging, separation of duties. Know that regulators expect *demonstrable, auditable* controls.

**Q: How do you "eliminate non-compliant solutions" and standardize (per the JD)?**
1. **Inventory** current tools/solutions and assess against enterprise standards (approved algorithms, FIPS HSMs, supported versions).
2. **Identify non-compliant** items (weak crypto, EOL software, unmanaged keys, shadow IT).
3. **Define target-state standard** and migration path.
4. **Migrate** consumers to the approved solution (often with dual-run for safety), then **decommission** the old one.
5. **Enforce** via policy-as-code so non-compliant solutions can't be reintroduced.
Do it with reliability discipline: communicate, stage, test, and provide rollback — decommissioning identity/crypto systems has enterprise blast radius.

> **Talking point:** Show risk-based prioritization (not just CVSS — EPSS/KEV/exposure), remediation *at scale via automation*, and that patching critical identity/crypto must be done with SRE safety (canary, rollback, error budget). Tie to continuous compliance and auditable evidence.

---

<a name="13"></a>
## 14. Policy-as-Code & Governance

### Basics

**Q: What is policy-as-code?**
Expressing governance/security/compliance rules as version-controlled, testable code that's automatically enforced — instead of PDFs and manual reviews. Enables consistent, automated, auditable enforcement. Tools: **Open Policy Agent (OPA)/Rego**, **HashiCorp Sentinel**, **Checkov/tfsec/KICS** (IaC), **Kyverno/OPA Gatekeeper** (Kubernetes), **AWS SCPs / Azure Policy / GCP Org Policy** (cloud guardrails).

**Q: What is OPA?**
Open Policy Agent — a general-purpose, open-source policy engine that decouples policy decisions from application logic. You write policies in **Rego**; services query OPA ("is this allowed?") and it returns allow/deny. Used for Kubernetes admission control, API authorization, IaC validation, and microservice authZ. Enables centralized, consistent policy across the stack.

### Intermediate & Advanced

**Q: Where would you apply policy-as-code in this identity/crypto SRE role?**
- **IaC gates:** block KMS keys without rotation, storage without encryption, weak TLS, public exposure.
- **Cloud guardrails:** SCPs/Azure Policy preventing creation of non-compliant crypto resources or disabling of logging.
- **Kubernetes admission:** enforce mTLS, no privileged pods, image signing.
- **AuthZ:** OPA for fine-grained ABAC decisions in services.
- **Compliance automation:** continuously evaluate resources against control frameworks and auto-generate audit evidence.
The point is **preventive** (block bad changes) + **detective** (continuously scan) controls, both as code.

**Q: Preventive vs detective controls?**
- **Preventive:** stop the bad thing from happening (policy gate blocks a non-compliant deploy, SCP denies an API call).
- **Detective:** identify it after the fact (config scanning, audit log alerts, drift detection).
Defense-in-depth uses both. Preventive is cheaper long-term; detective catches what slips through and provides audit assurance.

> **Talking point:** Policy-as-code = "compliance by default." It converts audit findings into permanent, automated guardrails so problems don't recur — a direct answer to "drive standardization" and "compliance automation."

---

<a name="14"></a>
## 15. Scripting (Python, Go, Bash)

### Basics

**Q: When do you reach for Bash vs Python vs Go?**
- **Bash:** quick glue, file/process manipulation, simple automation, chaining CLI tools. Keep it short; it gets unmaintainable fast.
- **Python:** most automation, API integrations, data processing, tooling — rich libraries (boto3, requests), readable, fast to write.
- **Go:** performance-sensitive tools, concurrency, single-binary CLI/daemons, cloud-native tooling (much of the SRE ecosystem — Kubernetes, Terraform — is Go). Compiled, statically typed, easy to distribute.

**Q: What makes a good automation script (production-grade)?**
Idempotency, error handling + meaningful exit codes, logging (not just prints), input validation, no hard-coded secrets (pull from vault/env), dry-run mode for destructive ops, timeouts/retries with backoff for network calls, and tests. Treat ops scripts as real software.

### Intermediate — Practical

**Q (Bash): Write a snippet to alert on certs expiring within 30 days.**
```bash
#!/usr/bin/env bash
set -euo pipefail
THRESHOLD_DAYS=30
for host in "$@"; do
  end_date=$(echo | openssl s_client -servername "$host" -connect "$host":443 2>/dev/null \
    | openssl x509 -noout -enddate | cut -d= -f2)
  end_epoch=$(date -d "$end_date" +%s)
  now_epoch=$(date +%s)
  days_left=$(( (end_epoch - now_epoch) / 86400 ))
  if (( days_left < THRESHOLD_DAYS )); then
    echo "WARN: $host cert expires in ${days_left} days ($end_date)"
  fi
done
```
Key points to mention: `set -euo pipefail` for safety, exit-code-driven alerting, and that in production you'd wire this to monitoring rather than eyeballing.

**Q (Python): Rotate a secret using AWS (conceptual).**
```python
import boto3, secrets, string

def rotate_secret(secret_id: str) -> None:
    client = boto3.client("secretsmanager")
    alphabet = string.ascii_letters + string.digits
    new_value = "".join(secrets.choice(alphabet) for _ in range(32))
    # create a new version (AWSPENDING), test it, then promote to AWSCURRENT
    client.put_secret_value(
        SecretId=secret_id,
        SecretString=new_value,
        VersionStages=["AWSPENDING"],
    )
    # ... verify the new secret works against the target service ...
    # ... then finalize so consumers pick it up, keep previous for rollback ...
```
Points: use the cryptographically-secure `secrets` module (not `random`), stage-then-promote so there's no downtime and rollback is possible, and never log the secret value.

**Q: How do you make an API call resilient in Python?**
Use retries with exponential backoff + jitter (e.g., `tenacity` or `urllib3 Retry`), set explicit timeouts, handle specific exceptions, and respect rate limits / `Retry-After`. Idempotency keys for non-idempotent calls.

### Advanced

**Q (Go): Why is Go popular in SRE, and what's a concurrency gotcha?**
Go compiles to a single static binary (easy deploy), has fast startup, and first-class concurrency via **goroutines + channels**. Common gotchas: data races on shared state (use `sync.Mutex` or channels, run `-race` detector), goroutine leaks (always have a way to cancel — `context.Context`), and unbounded goroutine creation. For SRE tooling that hits KMS/APIs concurrently, bound concurrency with worker pools/semaphores and propagate `context` for timeouts/cancellation.

> **Talking point:** Emphasize *production-grade* scripting — idempotent, safe, secret-aware, rets/timeouts, tested. Use the `secrets` module for crypto randomness, never log sensitive values, and stage-then-promote for rotation.

---

<a name="15"></a>
## 16. Behavioral / Leadership (STAR)

Use the **STAR** method: **S**ituation, **T**ask, **A**ction, **R**esult. Quantify results. Prepare 6–8 stories you can flex to different questions. Below are prompts + a model answer skeleton tuned to this role.

**Q: Tell me about a time you improved the reliability of a critical system.**
- *S:* Auth/identity platform had recurring latency spikes causing login failures.
- *T:* Own reliability; reduce incidents.
- *A:* Defined SLIs/SLOs, added burn-rate alerting and tracing, found a directory replication bottleneck, added read replicas + caching + circuit breakers, automated failover, ran a game day.
- *R:* Availability from 99.8% → 99.97%, P99 auth latency cut ~40%, on-call pages down 60%.

**Q: Tell me about leading a vulnerability remediation / audit-finding effort.**
- *S:* Large batch of criticals across many services (Mythos-style) with a tight deadline.
- *T:* Coordinate remediation without breaking prod.
- *A:* Aggregated/prioritized by risk (CVSS+KEV+exposure), assigned owners with SLAs, automated patching via IaC pipelines with canary + rollback, added policy-as-code to prevent recurrence, tracked burn-down, reported to leadership/auditors.
- *R:* X% of criticals closed in the window, zero remediation-caused outages, recurring findings eliminated by guardrails.

**Q: Tell me about a major incident you handled.**
Use section 4's identity-outage walkthrough as a real story: declared Sev1, IC role, mitigated via failover/rollback, blameless postmortem, systemic fixes (automated cert renewal + alerts). Emphasize calm coordination, mitigate-first, and follow-through on action items.

**Q: Tell me about a technical decision that had trade-offs / disagreement.**
Show you weigh options (e.g., dedicated HSM vs cloud KMS, or multi-region cost vs availability), used data, considered security/compliance/cost, communicated clearly, and drove consensus. Senior roles want judgment, not dogma.

**Q: Tell me about mentoring / technical leadership.** *(Preferred qualification)*
Describe raising the team's SRE maturity: introduced SLO practice, wrote runbooks, ran postmortem reviews, paired on Terraform/observability, set standards. Result: team autonomy up, incidents down, knowledge no longer siloed.

**Q: Tell me about standardizing / decommissioning a legacy system.**
Inventory → define standard → migrate consumers with dual-run → decommission → enforce with policy-as-code. Emphasize stakeholder communication and safe cutover given blast radius.

**Q: How do you handle competing priorities / high-pressure deadlines?** (Very relevant given "compressed remediation timelines.")
Prioritize by risk/impact, communicate trade-offs transparently, use error budgets/SLOs to make objective calls, break work down, and protect against burnout. Show you stay data-driven under pressure.

> **Tip:** For a bank, weave in *security consciousness, attention to compliance, and calm ownership under pressure*. Always end with a quantified result and what you learned.

---

<a name="16"></a>
## 17. Questions to Ask the Interviewer

Asking sharp questions signals seniority. Pick a few:

- What does the current SLO/error-budget practice look like for the identity and KMS platforms today — is it mature or something I'd help build?
- What are the biggest reliability or security pain points in the identity/crypto stack right now?
- How is the Mythos remediation work prioritized, and what does success look like in the first 6–12 months?
- What's the current split across AWS/Azure/GCP and on-prem for key management — and which HSM/KMS solutions are in play (Thales, CloudHSM, etc.)?
- How mature is the automation/IaC and policy-as-code footprint today? Where's the biggest toil?
- How do the SRE, security, and architecture teams collaborate day-to-day? Where does this role sit?
- What are the most pressing audit/compliance drivers shaping the roadmap?
- What does the on-call model look like for these critical platforms?
- How is the team measuring the success of the shift to a product-aligned model?
- What growth/leadership opportunities exist as these new teams are built out?

---

<a name="17"></a>
## 18. Rapid-Fire Cheat Sheet

**SRE**
- SLI = measure, SLO = target, SLA = contract w/ penalties. Error budget = 100% − SLO.
- 4 Golden Signals: Latency, Traffic, Errors, Saturation. USE (resources) / RED (services).
- Burn-rate alerts > static thresholds. Mitigate first, root-cause later. Blameless postmortems.
- Toil ≤ 50%; automate everything.

**Identity**
- AuthN = who you are; AuthZ = what you can do. AAA.
- OAuth2 = authorization; OIDC = authN on OAuth2 (ID token/JWT); SAML = enterprise SSO (XML).
- Auth Code + PKCE for public clients. Validate JWT sig/alg/aud/iss/exp; reject `alg:none`.
- RBAC/ABAC/PBAC; least privilege; PAM; JIT; Zero Trust ("never trust, always verify"); ITDR.
- Directory: LDAP, AD (LDAP+Kerberos), Entra ID.

**Crypto**
- Symmetric (AES, fast) vs asymmetric (RSA/ECC, key exchange) → hybrid (TLS).
- Hash = one-way integrity (SHA-256; passwords → Argon2/bcrypt). Encode ≠ encrypt.
- Envelope encryption: DEK encrypts data, KEK (in KMS/HSM) encrypts DEK.
- Key lifecycle: generate→distribute→use→rotate→revoke→archive→destroy.
- AEAD (AES-GCM); never reuse nonce+key. Forward secrecy (ECDHE).
- HSM = tamper-resistant, keys never leave, FIPS 140-2/3 L3. PKCS#11, KMIP.
- Separation of duties / dual control / M-of-N (Shamir). Crypto-agility & PQC (Kyber/Dilithium).

**KMS**
- AWS KMS / Azure Key Vault / GCP Cloud KMS — all HSM-backed, envelope encryption, audit logs.
- Customer-managed keys for control/revocation. BYOK / HYOK / EKM (external key store).
- HA: multi-region keys, HSM HA groups, data-key caching, watch throttling/quotas.
- Thales Luna (HSM) + CipherTrust Manager (central KMS/policy). KMIP for interop.

**Secrets**
- Vault/Secrets Manager/Key Vault. Dynamic short-lived secrets > static. Fetch at runtime.
- Solve secret-zero via platform identity (K8s SA, IAM, Managed Identity, OIDC).
- Scan for leaks (gitleaks); rotate on leak.

**Distributed/Linux**
- CAP: pick C or A under partition. Idempotency for safe retries.
- Timeouts + exponential backoff + jitter + circuit breakers → prevent cascading failure.
- USE method for host debugging; `ss`, `journalctl`, `iostat`, `dig`, `tcpdump`.

**IaC / CI-CD / Compliance**
- Terraform (multi-cloud) / CloudFormation (AWS) / ARM-Bicep (Azure). Secure remote state + locking.
- Golden modules + policy-as-code gates (OPA/Sentinel/Checkov) = compliance by default.
- Canary + auto-rollback on SLO regression. OIDC (no static keys). Sign artifacts (SLSA/cosign). GitOps (Argo/Flux).
- Prioritize vulns by CVSS + EPSS + KEV + exposure. PCI-DSS, SOX, FFIEC, NIST 800-53.
- Preventive + detective controls; automate audit evidence.

---

### Final prep checklist
- [ ] Can I define SLI/SLO/SLA/error budget and give an identity example in 60s?
- [ ] Can I explain envelope encryption and draw the KMS/HSM/DEK/KEK picture?
- [ ] Can I compare AWS/Azure/GCP KMS and BYOK vs HYOK vs EKM?
- [ ] Can I explain OAuth2 vs OIDC vs SAML and JWT validation pitfalls?
- [ ] Do I have 6–8 quantified STAR stories (reliability, remediation, incident, leadership)?
- [ ] Can I walk through an enterprise identity outage end-to-end?
- [ ] Am I honest about Thales/CipherTrust gaps but able to map to KMIP/HSM/central-KMS concepts?
- [ ] Do I connect everything back to *reliability + security + compliance*?

Good luck — you've got this.

