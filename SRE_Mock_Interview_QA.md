# SRE – Identity & Cryptography: MOCK INTERVIEW Q&A (Ready to Rehearse)

**Interview date: 7th** — this file is your drill sheet. Every answer is written to be **spoken in 45–90 seconds**. Practice each out loud twice.

**How to use:**
- ⭐ = very high probability, drill these first.
- Each answer has a **[Say this]** core answer and a **[If pushed]** follow-up for depth.
- The companion file `SRE_Identity_Crypto_Interview_Prep.md` has the deeper theory; this file is for *performance*.

**Universal answer formula (say it in this shape):**
1. One-sentence direct definition/answer.
2. One concrete example (ideally identity/crypto/bank).
3. Why it matters for reliability **and** security.

---

## ROUND 1 — Screening / "Tell me about yourself" (first 10 minutes)

### ⭐ Q1. Walk me through your background / tell me about yourself.
**[Say this]** "I'm a Site Reliability Engineer focused on the reliability *and* security of critical infrastructure — especially identity platforms and key-management systems. Day to day I define and defend SLOs for services like authentication and KMS, automate operations with Terraform and CI/CD, build observability so we catch identity and crypto issues before users do, and I lead vulnerability remediation and audit-finding closure in regulated environments. What draws me to this role is that it sits exactly at my sweet spot — SRE discipline applied to identity and cryptography in a highly regulated bank, which is some of the highest-stakes reliability work there is."

**[Tip]** Keep it 60 seconds, end with *why this role*. Don't recite your whole resume.

### ⭐ Q2. Why do you want to work at Bank of America / on this program?
**[Say this]** "Three reasons. First, scale and impact — identity and crypto are the systemic backbone of a bank; if auth or key management goes down, everything does, so the reliability work genuinely matters. Second, the Mythos remediation program is exactly the kind of high-visibility, security-driven work I do best — prioritizing and remediating vulnerabilities at scale without breaking mission-critical systems. Third, the chance to help build new product-aligned SRE teams and set enterprise standards for identity and cryptography is a great fit for where I want to grow."

### Q3. What does an SRE do, in your words?
**[Say this]** "SRE applies software engineering to operations. Instead of manually firefighting, we measure reliability with SLIs and SLOs, use error budgets to balance new features against stability, automate away toil, and run blameless postmortems so we fix systems, not blame people. The goal is systems that are reliable, secure, and operable at scale."

### Q4. What's your experience level with the cloud KMS solutions listed (AWS KMS, Azure Key Vault, GCP KMS, Thales)?
**[Say this — be honest, then bridge]** "I have hands-on experience with [name the ones you truly know — e.g., AWS KMS and Azure Key Vault] for envelope encryption, customer-managed keys, rotation, and access policies. The underlying concepts — HSM-backed roots of trust, DEK/KEK envelope encryption, KMIP, BYOK — carry across all of them, including Thales CipherTrust and Luna, so I ramp quickly on a specific vendor. I'd rather be upfront: [Thales/GCP] I've studied deeply but have less production time on, and I'm confident I close that gap fast because the fundamentals are the same."

**[Why this works]** Interviewers respect honesty + transferable fundamentals far more than bluffing that gets exposed later.

---

## ROUND 2 — SRE Deep-Dive ⭐ (the core of this interview)

### ⭐ Q5. Explain SLI, SLO, SLA, and error budget with an example.
**[Say this]** "An **SLI** is a measurement of service behavior — for authentication, say the percentage of login requests that succeed under 300 milliseconds. An **SLO** is the target for that SLI — for example, 99.95% over a rolling 28 days; that's our internal goal. An **SLA** is the external contract with penalties if we miss it, and it's always looser than the SLO so we catch problems before breaching the contract. The **error budget** is 100% minus the SLO — so at 99.95% we can be 'unreliable' for about 22 minutes a month. When the budget is healthy we ship fast; when it's burning, we freeze risky changes and focus on reliability."

**[If pushed — "how do you pick the SLO target?"]** "Start from user expectations and business impact, look at historical performance, and never chase 100% — it's infinitely expensive and you can't be more available than your weakest critical dependency. For bank auth it's typically 99.95–99.99%, validated against the real dependency chain, and reviewed quarterly."

### ⭐ Q6. What SLIs would you define for an identity/authentication platform?
**[Say this]** "I'd pick SLIs around the user journey. **Availability** — ratio of successful auth responses to valid requests. **Latency** — proportion of token issuance and validation under a threshold, like p99 under 300 ms. **Correctness** — MFA challenges delivered and validated successfully. And **freshness** — directory replication lag, like 99% of group changes propagate within 60 seconds. I measure as close to the user as possible, at the gateway or load balancer, so the SLI reflects real experience."

**[If pushed — KMS SLIs]** "For a KMS: availability of encrypt/decrypt/sign operations, latency of those ops, throttle/error rates, and durability of key material — plus HSM utilization and session counts as saturation signals."

### ⭐ Q7. What is an error budget policy and what happens when the budget is exhausted?
**[Say this]** "An error budget policy is a pre-agreed rule — not a negotiation during a crisis — for what happens when we burn the budget. Typically: freeze feature launches, redirect engineering to the top reliability issues identified from postmortems, root-cause the biggest budget-burners, and improve rollout safety with canaries and progressive delivery. The key is it's decided in advance and shared between dev, ops, and security, so it's objective. If we're *chronically* over budget, either the SLO is unrealistic or the architecture needs real investment."

### ⭐ Q8. How do you alert on SLOs? Why is burn-rate better than a static CPU threshold?
**[Say this]** "I use multi-window, multi-burn-rate alerts. Burn rate is how fast you're consuming the error budget — a burn rate of 1 exactly exhausts it by the window's end; 14 would exhaust a 30-day budget in about two days. I set a fast-burn alert that pages — say 2% of budget in an hour — and a slow-burn alert that just files a ticket — say 10% over three days. This is better than a static CPU threshold because it alerts on *user-visible impact*, not an arbitrary cause. High CPU might be fine; a spike in failed logins is not. It gives fast detection with far fewer false pages."

### Q9. What is toil and how do you reduce it?
**[Say this]** "Toil is manual, repetitive, automatable work that scales with the service and adds no lasting value — things like hand-rotating certs, manual access grants, or clicking through consoles. SRE tries to keep it under about half our time. I reduce it by codifying it: automated cert lifecycle, automated key rotation, self-service tooling with guardrails, IaC for provisioning, and auto-remediation runbooks triggered by alerts. Then I measure the toil reduction to prove it worked."

### Q10. What are the four golden signals?
**[Say this]** "Latency, traffic, errors, and saturation. Latency is how long requests take — and I separate success from error latency. Traffic is demand, like auths per second. Errors is the failure rate. Saturation is how full the system is — CPU, memory, connection pools, or HSM throughput. For resources I use the USE method — utilization, saturation, errors; for services, RED — rate, errors, duration."

### Q11. How do you drive reliability improvements with data rather than opinion?
**[Say this]** "I anchor everything on SLIs/SLOs and error-budget burn. I look at where the budget is actually being spent — which endpoints, which dependencies, which incidents — using metrics, logs, and traces. Postmortem action items get prioritized by budget impact. So instead of 'I think the directory is slow,' I can say 'directory latency caused 40% of our budget burn last month,' and that objectively justifies the fix. Data turns reliability from opinion into an engineering decision."

---

## ROUND 3 — Identity & Access Management (IAM) Deep-Dive ⭐

### ⭐ Q12. What's the difference between authentication and authorization?
**[Say this]** "Authentication is proving *who you are* — verifying identity with factors like a password, a token, or a biometric. Authorization is deciding *what you're allowed to do* once you're authenticated — the permissions and policies. Authentication always comes first; authorization builds on it. A clean way to say it: authentication is the ID check at the door, authorization is which rooms your badge opens."

### ⭐ Q13. Explain OAuth 2.0 vs OIDC vs SAML.
**[Say this]** "OAuth 2.0 is an *authorization* framework — it issues access tokens so an app can call an API on your behalf; it's about delegated access, not proving identity. OIDC, OpenID Connect, is an *authentication* layer on top of OAuth 2.0 — it adds an ID token, a JWT, that asserts who the user is; it's the modern SSO standard for web and mobile. SAML is the older XML-based standard for enterprise single sign-on and federation — an identity provider issues signed assertions to a service provider; it's still everywhere in banks. Shorthand: OAuth for API access, OIDC for modern login, SAML for enterprise SSO."

### ⭐ Q14. Walk me through the OAuth 2.0 Authorization Code flow with PKCE.
**[Say this]** "The app redirects the user to the authorization server to log in. After the user authenticates and consents, the server returns a short-lived authorization *code* to the app's redirect URI. The app then exchanges that code at the token endpoint for an access token — and usually a refresh token and, with OIDC, an ID token. PKCE protects public clients like SPAs and mobile: the app sends a hashed `code_challenge` up front and the plaintext `code_verifier` at exchange, so even if someone intercepts the authorization code, it's useless without the verifier. It closes the code-interception attack."

### ⭐ Q15. What is a JWT and what are the security pitfalls?
**[Say this]** "A JWT is a compact, signed token with three parts — header, payload of claims, and signature. Claims include issuer, subject, audience, and expiry. The big pitfalls: always verify the signature *and* pin the expected algorithm — reject `alg: none` and guard against algorithm-confusion attacks where RS256 is downgraded to HS256. Validate audience, issuer, and expiry. Keep tokens short-lived, rotate signing keys via a JWKS endpoint, and remember the payload is only base64-encoded, *not* encrypted — so never put secrets in it."

### Q16. RBAC vs ABAC — when do you use each?
**[Say this]** "RBAC grants permissions through roles — simple and easy to audit, but you hit 'role explosion' at scale. ABAC makes decisions from attributes — user department, resource sensitivity, time, location, risk score — so it's far more fine-grained and context-aware. In a bank I'd typically layer them: RBAC for coarse access, ABAC for context-sensitive decisions like blocking access from an untrusted device or geography. Expressed as policy-as-code with something like OPA, that becomes PBAC."

### Q17. What is least privilege and how do you enforce it in practice?
**[Say this]** "Grant the minimum access needed, for the minimum time. In practice: regular access recertification and attestation, just-in-time and just-enough access instead of standing admin rights, privileged access management for admin accounts, automated deprovisioning when someone changes roles or leaves, and access analytics to find and strip unused permissions. The goal is to shrink the blast radius if any credential is compromised."

### ⭐ Q18. What is Zero Trust, and how does identity fit in?
**[Say this]** "Zero Trust means 'never trust, always verify' — no implicit trust just because a request comes from inside the network. Every request is authenticated, authorized, and continuously evaluated based on identity, device posture, and context. Identity becomes the new perimeter — it's the primary control plane. In practice that's strong phishing-resistant MFA, device trust, micro-segmentation, least privilege, and continuous access evaluation that can revoke a session mid-stream if risk changes. For a bank, identity is where Zero Trust lives or dies."

### Q19. What is PAM and why does it matter in a bank?
**[Say this]** "Privileged Access Management secures the high-power accounts — root, domain admin, service accounts, DBAs. It vaults credentials, rotates them automatically, records sessions, and grants just-in-time elevation with approvals instead of standing access. It matters because privileged credentials are attackers' number-one target — compromise one and you can own the environment. Tools like CyberArk or BeyondTrust. It's also core to satisfying auditors on separation of duties."

### Q20. (Preferred) What is ITDR — Identity Threat Detection and Response?
**[Say this]** "ITDR is the security discipline focused on identity-based threats — credential theft, privilege escalation, lateral movement, MFA fatigue attacks, token replay, and directory attacks like golden tickets. It combines identity posture management — finding excess privilege and misconfigurations — with real-time detection of things like impossible travel or anomalous logins, and automated response such as step-up auth or revoking a session. I think of it as extending SRE-style observability into the identity security domain."

### Q21. How would you improve the reliability of an enterprise directory like Active Directory?
**[Say this]** "Treat directory availability as a top-tier SLO because nearly everything depends on it. I'd use multi-master replication with monitored replication lag, geographically distributed read replicas near consumers, health-checked load balancing, and capacity planning for auth storms like Monday-morning logins. Strict change control on schema and GPO changes because the blast radius is enterprise-wide, synthetic auth monitoring to catch issues before users, and tested DR and failover. Caching frequent lookups helps latency too."

---

## ROUND 4 — Cryptography Deep-Dive ⭐

### ⭐ Q22. Symmetric vs asymmetric encryption — and what does TLS actually use?
**[Say this]** "Symmetric uses the same key to encrypt and decrypt — AES is the standard, it's fast, great for bulk data, but you have to share the key securely. Asymmetric uses a key pair — public key encrypts or verifies, private key decrypts or signs — RSA and ECC; it solves key distribution but is slower. TLS uses *both*: an asymmetric handshake, typically ECDHE, to authenticate the server and agree on a shared symmetric session key with forward secrecy, then fast symmetric encryption for the actual data. That's hybrid encryption."

### ⭐ Q23. Explain envelope encryption. (Very likely for a KMS role.)
**[Say this]** "Envelope encryption means you don't encrypt data directly with the master key. Instead you generate a data encryption key — a DEK — encrypt the data with the DEK using fast symmetric crypto, then encrypt the DEK itself with a key encryption key — the KEK — which lives inside the KMS or HSM and never leaves. You store the encrypted DEK alongside the ciphertext. To read it, the KMS decrypts the DEK, then you decrypt the data locally. The wins: the master key never leaves the HSM, you make far fewer KMS calls so it scales for big data, and you can rotate the KEK without re-encrypting everything. That's exactly how AWS KMS, Azure Key Vault, and GCP KMS work."

**[If pushed — "how does AWS do this?"]** "You call `GenerateDataKey`, which returns both a plaintext DEK and an encrypted DEK. You encrypt with the plaintext DEK, then discard it and store the encrypted one. To decrypt, you call `Decrypt` on the encrypted DEK to recover the plaintext, then decrypt your data."

### ⭐ Q24. What is key lifecycle management?
**[Say this]** "It's managing a key through every stage: generation with strong entropy, ideally inside an HSM; distribution and provisioning; usage under strict access control; rotation on a schedule and on suspected compromise; revocation; archival so you can still decrypt old data; and finally secure destruction, which for encrypted data is crypto-shredding — destroy the key and the data is unrecoverable. Good lifecycle management means defined crypto-periods, automated rotation, separation of duties, and full audit logging of every key operation."

### Q25. Hashing vs encryption vs encoding?
**[Say this]** "Encryption is reversible with a key and provides confidentiality — AES, RSA. Hashing is one-way, a fixed-length digest for integrity, not confidentiality — SHA-256; and for passwords you use slow, salted hashes like Argon2 or bcrypt. Encoding, like Base64, is just a reversible representation change with *no security at all*. A classic mistake is thinking Base64 is encryption — it isn't; anyone can decode it."

### ⭐ Q26. What is an HSM and why would a bank need one?
**[Say this]** "A Hardware Security Module is a tamper-resistant device that generates, stores, and uses cryptographic keys so the private key material *never leaves the device in plaintext* — the crypto happens inside it. Banks need them for strong key protection, high-quality random number generation, and certifications like FIPS 140-2 or 140-3 Level 3 that are required for payments and PCI. You access them through standards like PKCS#11 or KMIP. Examples are Thales Luna and payShield, AWS CloudHSM, Entrust nShield."

**[If pushed — "HSM vs cloud KMS?"]** "Cloud KMS is managed and multi-tenant but still HSM-backed. You go to a dedicated HSM or Managed HSM when you need single-tenant isolation, FIPS 140-2 Level 3 assurance, full control of key material, or bring-your-own-key for compliance — common for payment and root-key use cases. The trade-off is more control versus more operational burden."

### Q27. What is separation of duties / dual control / M-of-N in key management?
**[Say this]** "No single person should control a critical key end to end. Dual control means two people are required for a sensitive operation. Split knowledge or M-of-N — using something like Shamir's Secret Sharing — means you need k of n key-holders to reconstruct or authorize use of a master key. PCI-DSS requires this, and it's standard for HSM root and master keys. The point is reducing insider risk and satisfying auditors."

### Q28. (Senior signal) What are crypto-agility and post-quantum cryptography?
**[Say this]** "Crypto-agility is designing systems so you can swap algorithms or keys without re-architecting — abstract crypto behind interfaces, keep an inventory of where crypto is used, and avoid hard-coded algorithms. That matters because algorithms get broken. Post-quantum crypto is the next driver: quantum computers threaten RSA and ECC via Shor's algorithm, and NIST standardized replacements in 2024 — ML-KEM, Kyber, for key exchange and ML-DSA, Dilithium, for signatures. Banks are already doing 'harvest now, decrypt later' risk assessments and building crypto inventories to prepare to migrate."

---

## ROUND 5 — KMS / Cloud Key Management Deep-Dive ⭐

### ⭐ Q29. Compare AWS KMS, Azure Key Vault, and GCP Cloud KMS.
**[Say this]** "All three are managed, HSM-backed services that do envelope encryption, automatic rotation, IAM-controlled access, and per-operation audit logging. AWS KMS uses KMS keys with IAM plus key policies and grants, and logs to CloudTrail; secrets and certs are separate services, Secrets Manager and ACM. Azure Key Vault is the most consolidated — keys, secrets, *and* certificates in one vault, controlled by Azure RBAC. GCP Cloud KMS uses crypto keys and key versions on key rings, controlled by Cloud IAM. For higher assurance, each has an HSM tier — CloudHSM, Managed HSM, Cloud HSM — and external options like AWS XKS or GCP EKM."

### ⭐ Q30. What's the difference between BYOK, HYOK, and EKM/external key store?
**[Say this]** "BYOK, bring your own key, means you generate the key material — often in your on-prem HSM — and import it into the cloud KMS; you control generation, the cloud stores and uses it. HYOK, hold your own key, means the key never leaves your control; the cloud calls out to your HSM. EKM or external key store, like GCP EKM or AWS XKS, means the cloud KMS delegates the actual crypto operations to an external key manager you run, such as Thales CipherTrust — so you keep full control and meet strict compliance. The trade-off with the external options is added latency and a hard availability dependency on your external HSM."

### Q31. How do you make a KMS/HSM highly available and low-latency across regions?
**[Say this]** "Use multi-region keys — AWS multi-Region keys, or replicate key material via BYOK into each region — so encrypt and decrypt happen region-locally. Run HSMs in HA groups across availability zones with load balancing. Cache data keys client-side with a bounded TTL — the AWS Encryption SDK supports this — which cuts KMS calls and rides out brief blips. Proactively monitor KMS throttling and quotas, because throttling is a real outage cause, and request limit increases ahead of load. Then define latency and availability SLOs for crypto ops and test regional failover in game days."

### ⭐ Q32. What are the failure modes of depending on KMS for everything?
**[Say this]** "The big one is that KMS becomes a systemic dependency — if it's down, apps can't decrypt and you have an outage. I mitigate with data-key caching, multi-region keys, and graceful degradation. Throttling under load — mitigate with envelope encryption, caching, and quota increases. Accidental key disable or deletion instantly locks out all data — so enable deletion protection, mandatory waiting periods, and alerts. And with external key stores, an external HSM outage breaks cloud crypto, so you need HA there too. The theme is: treat the KMS as a tier-0 dependency and design for its failure."

### ⭐ Q33. (Directly from the JD) How would you standardize key management across a multi-cloud bank?
**[Say this]** "First, inventory every key, HSM, and crypto usage across clouds and on-prem to find non-compliant and shadow crypto. Then establish a central control plane — something like CipherTrust Manager with KMIP — with common policy, naming, tagging, rotation periods, and audit. Define enterprise standards: approved algorithms and key lengths, FIPS-validated HSM roots of trust, separation of duties. Enforce them with policy-as-code and IaC golden modules so keys are provisioned compliantly *by default*. Then migrate consumers off non-compliant solutions — usually dual-running for safety — and decommission the old ones. Finally, centralize observability and audit of all key operations. That's reliability discipline applied to a compliance goal."

### Q34. (Preferred) What do you know about Thales CipherTrust Manager and Luna HSMs?
**[Say this]** "Thales Luna is a FIPS-validated HSM that acts as the hardware root of trust — keys never leave it. CipherTrust Manager is the centralized platform on top: it manages keys, policies, and access across databases, files, cloud, and apps, and supports KMIP, bring-your-own-key to cloud KMS, tokenization, and transparent data encryption. The value is one control plane for keys across a hybrid, multi-cloud estate with centralized policy and audit — which is exactly the standardization this role is chasing. I'll be transparent about my hands-on depth, but the concepts — HSM root of trust, KMIP interop, centralized KMS, BYOK — I know well and map directly to what I've done on AWS and Azure."

### Q35. What is KMIP and why does it matter?
**[Say this]** "KMIP, the Key Management Interoperability Protocol, is an OASIS standard for how key-management clients and servers talk — create, retrieve, rotate, destroy keys. It matters because it lets heterogeneous products — storage arrays, databases, HSMs, cloud KMS — all interoperate with one central key manager like CipherTrust. In a multi-vendor bank, KMIP is how you avoid lock-in and keep a single, consistent key-management standard."

---

## ROUND 6 — Scenario / System-Design / Troubleshooting ⭐

> These are the questions that separate a "pass" from a "strong hire." Think out loud, state assumptions, and always cover reliability *and* security.

### ⭐ Q36. Scenario: Enterprise-wide login failures just started. Walk me through your response.
**[Say this]** "First I declare a Sev1 and take or assign the Incident Commander role, open a bridge, and notify downstream app teams — auth has enterprise blast radius. Then I assess scope: is it all users or a subset, one region, one IdP? My very first investigation is *what changed* — I check the deploy and change calendar, because most incidents follow a change. If it's a recent config, cert, or deploy change, I roll it back or fail over to a healthy region or replica; if a dependency like the KMS or HSM is unavailable, I fail over to the redundant one. I mitigate to restore service *before* doing full root cause. I communicate at fixed intervals to leadership. Once restored, I verify with synthetic auth probes, then run a blameless postmortem — say the root cause was an expired signing cert — and the systemic fixes are automated cert renewal with multi-week expiry alerts, staged rollout, and a chaos test of the failover path."

**[Interviewer wants]** Structure (declare→scope→check-change→mitigate→communicate→verify→postmortem), *mitigate-first*, and systemic prevention. Nail this one.

### ⭐ Q37. Scenario: Design a highly available authentication service.
**[Say this]** "I'd make the auth nodes stateless behind health-checked load balancers, autoscaled, and redundant across availability zones and ideally regions, active-active. Every dependency — directory, KMS, MFA provider — gets a timeout and a circuit breaker so a slow dependency can't cascade. I'd add caching with bounded TTLs for tokens, keys, and directory lookups, and graceful degradation, like honoring recently-cached authZ decisions if the policy engine blips. Rate limiting protects against auth storms and credential-stuffing attacks. Behind it: replicated directories, multi-region KMS, HSM HA groups. Then SLOs with burn-rate alerts, and I'd validate the whole thing with chaos game days — kill a replica, expire a cert in staging, throttle the KMS — to prove the failure modes are handled. And security throughout: MFA, short-lived tokens, full audit logging."

### ⭐ Q38. Scenario: You're handed thousands of vulnerability findings (Mythos-style) with a tight deadline. How do you run it?
**[Say this]** "I aggregate and normalize findings from all sources and dedupe them. Then I prioritize by *real* risk — not just CVSS, but CVSS combined with exploitability signals like EPSS and CISA's known-exploited list, and asset exposure and criticality; internet-facing systems on the identity or crypto critical path go first. I assign clear ownership with SLAs, and I remediate *at scale through automation* — patch pipelines and config-as-code — not one-off manual fixes. Where I can't patch immediately, I use compensating controls with documented risk acceptance and sign-off. I verify with re-scans, track a burn-down chart for leadership and auditors, and feed lessons back — hardening golden images and adding policy-as-code so the same finding can't recur. And critically, I patch mission-critical systems with SRE safety: staged rollout, canary, and rollback, so remediation never causes the outage we're trying to prevent."

### ⭐ Q39. Scenario: How do you patch a mission-critical identity system with no downtime?
**[Say this]** "Redundancy plus rolling or canary patching — patch one node at a time behind HA so the service stays up. Test in a non-prod environment first, use canary to expose a small slice of traffic and watch the SLIs, and wire automated rollback if the SLO regresses. Where possible I prefer immutable infrastructure — replace the node from a patched image rather than patching in place. For the riskiest changes I'll use a maintenance window, and I use the error budget to decide how aggressive to be — if the vuln is actively exploited, the urgency justifies spending budget."

### Q40. Scenario: A service depends on the KMS and KMS starts throttling under load. What happens and what do you do?
**[Say this]** "Under throttling, crypto operations start failing or slowing, which can cascade into request timeouts and retry storms that make it worse. Immediate mitigation: enable data-key caching so we're not calling KMS per operation, and make sure clients use exponential backoff with jitter rather than hammering. Then request a quota increase and check whether we can batch operations. Structurally, envelope encryption with client-side DEK caching is the fix — it dramatically cuts KMS call volume — plus circuit breakers so a throttled KMS fails fast instead of dragging everything down. Longer term, multi-region keys spread the load."

### Q41. Troubleshooting: A Linux host is at high load and the service is slow. How do you debug?
**[Say this]** "I use the USE method — for each resource check utilization, saturation, and errors. Start with `top` or `htop` and load average compared to core count. High user CPU points at the process; high I/O wait points at disk, so I check `iostat` and `iotop`. Memory with `free` and `vmstat`, and `dmesg` or journald for OOM kills. Disk space with `df`. Network with `ss -tulpn` for listeners and `ss`/`netstat` for connection states. If it's the service itself, `systemctl status` and `journalctl -u`. I move top-down from symptom to resource to root cause rather than randomly running commands."

### Q42. Troubleshooting: Intermittent auth latency spikes, but only sometimes. How do you find it?
**[Say this]** "Intermittent means I need tracing, not just averages. I'd look at p99 and p999 latency, not the mean, because averages hide spikes. Distributed tracing across the login flow — gateway, IdP, directory, MFA, token service — to see *which hop* spikes. I'd correlate the spikes in time with other signals: directory replication lag, KMS throttling, GC pauses, cache misses, or a cron job. High cardinality dimensions like per-region or per-IdP in logs and traces help isolate it. Often intermittent latency is a dependency under periodic load or a cache expiry stampede — so I check TTL alignment and add jitter to cache refresh."

### Q43. Scenario: How do you eliminate a non-compliant legacy crypto solution without breaking things?
**[Say this]** "Inventory who and what depends on it. Define the compliant target-state standard and a migration path. Migrate consumers over — usually dual-running old and new in parallel so I can validate and roll back safely. Once traffic is fully cut over and verified, decommission the legacy system. Then enforce with policy-as-code so it can't be reintroduced. Throughout, heavy stakeholder communication and staged cutover, because decommissioning identity or crypto systems has enterprise blast radius — the reliability discipline is what makes the security change safe."

---

## ROUND 7 — IaC / CI/CD / Automation

### Q44. Why Terraform for a multi-cloud bank, and how do you manage state safely?
**[Say this]** "Terraform is cloud-agnostic — one tool and language across AWS, Azure, GCP, and on-prem — so it's the natural common denominator in a multi-cloud shop, sometimes alongside cloud-native tools for provider-specific features. State is critical: it maps config to real resources and can contain secrets, so I keep it in a secure remote backend — S3 with DynamoDB locking, or Terraform Cloud — with encryption, access control, and state locking to prevent concurrent corruption. Never commit state to git."

### Q45. How do you enforce enterprise standards and compliance through IaC?
**[Say this]** "Two things: golden modules and policy-as-code gates. I build reusable, versioned modules — like a 'KMS key' module that bakes in rotation, tagging, key policy, and audit — so teams provision compliantly by default. Then in the pipeline I run policy-as-code checks — Sentinel, OPA/Conftest, Checkov, or tfsec — that block non-compliant changes before apply: no unencrypted storage, no weak TLS, rotation required, nothing public. That shifts compliance left and auto-generates evidence for auditors."

### Q46. How do you deploy changes to critical services safely?
**[Say this]** "Canary with automated rollback tied to SLOs. Roll the change to a small slice of traffic, watch the golden signals and error-budget burn, and if it regresses, roll back automatically before it spreads. Blue-green when I need instant full rollback. For credentials I use OIDC federation so the pipeline gets short-lived cloud creds instead of static keys, I scan code, dependencies, containers, and IaC as gates, and I sign artifacts for supply-chain integrity. Every deploy is audited."

### Q47. How do you keep secrets out of code and pipelines?
**[Say this]** "Secrets never live in the repo — they're pulled from a secrets manager or KMS at runtime. Pre-commit hooks and scanners like gitleaks catch accidental commits, and any leaked secret is treated as compromised and rotated immediately. In CI/CD I use OIDC for short-lived cloud credentials rather than stored keys, mask secrets in logs, and scope them per environment. And I prefer dynamic, short-lived secrets — like Vault generating credentials on demand with a TTL — over long-lived static ones."

---

## ROUND 8 — Behavioral (STAR) ⭐

> Use **S**ituation → **T**ask → **A**ction → **R**esult. Quantify the result. Prepare these as *your own real stories* — the skeletons below show the shape and what to emphasize.

### ⭐ Q48. Tell me about a time you improved the reliability of a critical system.
**[Structure]** "A [identity/auth] platform had recurring latency spikes causing login failures *(S)*. I owned reducing incidents *(T)*. I defined SLIs and SLOs, added burn-rate alerting and distributed tracing, found a directory replication bottleneck, and added read replicas, caching, and circuit breakers with automated failover, then validated with a game day *(A)*. Availability went from 99.8% to 99.97%, p99 auth latency dropped about 40%, and on-call pages fell 60% *(R)*."
**[Emphasize]** data-driven, systemic fix, quantified result.

### ⭐ Q49. Tell me about leading a vulnerability remediation or audit-finding effort.
**[Structure]** "We had a large batch of critical findings across many services with a hard deadline *(S)*. I had to drive remediation without breaking production *(T)*. I aggregated and prioritized by risk — CVSS plus exploitability and exposure — assigned owners with SLAs, automated patching through IaC pipelines with canary and rollback, and added policy-as-code to stop recurrence *(A)*. We closed [X]% of criticals in the window with zero remediation-caused outages, and eliminated the recurring findings with guardrails *(R)*."

### Q50. Tell me about a major incident you handled.
**[Structure]** Use the Q36 outage as a real story: declared Sev1, ran the bridge as IC, mitigated via failover/rollback, then blameless postmortem with systemic fixes like automated cert renewal and expiry alerts. Emphasize *calm coordination, mitigate-first, and follow-through* on action items.

### Q51. Tell me about a tough technical decision with trade-offs or disagreement.
**[Structure]** Pick something like dedicated HSM vs cloud KMS, or multi-region cost vs availability. Show you gathered data, weighed security, compliance, cost, and reliability, communicated the trade-offs clearly, and drove consensus. **They're testing judgment, not dogma.**

### Q52. (Preferred: leadership) Tell me about mentoring or providing technical leadership.
**[Structure]** "I raised the team's SRE maturity — introduced the SLO practice, wrote runbooks, ran postmortem reviews, and paired with engineers on Terraform and observability, and set standards. As a result the team became more autonomous, incidents dropped, and knowledge stopped being siloed." Ties directly to the JD's mentorship expectation.

### ⭐ Q53. How do you handle competing priorities under a compressed deadline? (Very likely — this program has tight timelines.)
**[Say this]** "I prioritize by risk and impact, communicate trade-offs transparently rather than silently dropping things, and use SLOs and error budgets to make objective calls instead of arguing opinions. I break work into shippable increments so we make visible progress, escalate early when something's truly blocked, and I watch the team's load because burnout is itself a reliability risk. Under pressure I stay data-driven and calm — that steadiness is half the job in incident-heavy work."

### Q54. Tell me about a time you disagreed with a security or compliance requirement.
**[Say this — careful framing for a bank]** "I start from the assumption the control exists for a reason, so I dig into the *intent* first. If I think the implementation is causing reliability harm, I bring data and propose an alternative that still satisfies the control's intent — for example, a compensating control or a safer rollout — rather than just pushing back. In a regulated bank you don't bypass controls; you partner with security to meet the requirement in a more reliable way." **Never signal you'd cut a compliance corner.**

---

## ROUND 9 — Rapid-Fire Technical (be crisp, 1–2 sentences each)

- **CIA triad?** Confidentiality, Integrity, Availability — the three security goals.
- **AAA?** Authentication, Authorization, Accounting/Audit.
- **CAP theorem?** Under a network partition you choose Consistency *or* Availability; you can't have both.
- **Idempotency?** Same operation repeated gives the same result — essential so retries are safe.
- **Blue-green vs canary?** Blue-green switches all traffic to a new env for instant rollback; canary shifts a small % gradually while watching SLIs.
- **Symmetric algo?** AES. **Asymmetric?** RSA, ECC. **Hash?** SHA-256; passwords use Argon2/bcrypt.
- **Salt vs pepper?** Salt is a unique per-hash random value stored with the hash; pepper is a secret added to all, kept separate.
- **mTLS?** Both client and server present certs — mutual authentication; used for service-to-service.
- **What's `alg: none` and why reject it?** A JWT header claiming no signature — accepting it lets attackers forge tokens.
- **Forward secrecy?** Ephemeral keys (ECDHE) so a compromised long-term key can't decrypt past sessions.
- **DEK vs KEK?** Data Encryption Key encrypts data; Key Encryption Key (in KMS/HSM) encrypts the DEK.
- **FIPS 140-2 Level 3?** The HSM assurance level banks typically require — tamper-resistant with identity-based auth.
- **MTTR/MTTD/MTBF?** Mean Time To Recover / To Detect / Between Failures.
- **RTO vs RPO?** Recovery Time Objective = how fast you recover; Recovery Point Objective = how much data loss is acceptable.
- **SPOF?** Single Point of Failure — eliminate with redundancy.
- **Kerberos?** Ticket-based network auth using a KDC; underpins Active Directory. Very time-sensitive — needs NTP.

---

## FINAL PREP — the 3 days before the 7th

**Day-by-day:**
- **Now → Day 5:** Read every ⭐ answer out loud twice. Focus: SLO/SLI/error budget, envelope encryption, OAuth/OIDC/SAML, the outage scenario (Q36), the remediation scenario (Q38).
- **Day 6:** Write out your 6–8 STAR stories with *real* numbers. Rehearse Q1, Q2, Q48, Q49, Q53.
- **Day 7 (interview day):** Skim the Rapid-Fire section and the cheat sheet in the companion file. Re-read your STAR stories. Then stop and breathe.

**The 5 answers you MUST be able to give flawlessly:**
1. SLI / SLO / SLA / error budget with an identity example (Q5).
2. Envelope encryption + how cloud KMS uses it (Q23).
3. OAuth 2.0 vs OIDC vs SAML (Q13).
4. The enterprise login-outage incident walkthrough (Q36).
5. Leading vulnerability remediation at scale without breaking prod (Q38).

**Golden rule for every answer:** connect it back to **reliability + security + compliance**. That trio is the whole job.

**On honesty:** For any tool you haven't touched (Thales, GCP KMS, a specific tracer), say so plainly and immediately pivot to the transferable fundamentals. Interviewers trust honesty and transferable depth far more than they trust someone who bluffs and gets caught.

**Interview-day behaviors:**
- Think out loud on scenarios — they're scoring your *reasoning*, not just the final answer.
- State your assumptions before designing.
- If you don't know, say "I haven't hit that directly, but here's how I'd reason about it…"
- Have 3–4 questions ready to ask them (see the companion file's list).
- Be calm and structured. In SRE, composure under pressure *is* the competency.

You've got a real shot at this. Drill the ⭐ answers, own your STAR stories, and stay calm. Good luck on the 7th.



