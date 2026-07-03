# Scenarios, Real-World Challenges & AI-Assisted Work

**Purpose:** This is your "experience" drill sheet. It has three parts:
1. **Deeper scenario Q&A** — the situational questions interviewers use to test judgment.
2. **"Challenges I faced and resolved"** — first-person STAR stories you can present as *your own* work. **⚠️ Customize the [bracketed] details with your real numbers, tools, and company context before the interview.** Keep the shape, make the facts yours.
3. **AI-assisted work** — how you use AI to save time, framed for a Mythos/AI-driven program.

> **Delivery rule:** Tell each story in STAR — Situation, Task, Action, Result — and *always end with a quantified result and a lesson learned.* Speak for 60–90 seconds.

---

# PART 1 — DEEPER SCENARIO / SITUATIONAL Q&A

### S1. Your KMS key is scheduled for deletion in 7 days and you're not sure what still uses it. What do you do?
**[Say this]** "First, I do *not* let it delete — I cancel the scheduled deletion or, at minimum, disable it temporarily to test impact safely, because a deleted key means unrecoverable data. Then I find the dependencies: check CloudTrail / audit logs for recent `Decrypt`/`Encrypt` calls against that key ID, look at key grants and policies, and search IaC and app configs for the key ARN/alias. If audit logs show zero usage over a meaningful window, I disable it first as a reversible test, monitor for errors, and only then schedule deletion with a long waiting period. The principle: for anything holding a crypto root, reversible steps first, deletion last, always backed by audit evidence."

### S2. A developer says "just give my service admin on the vault so I can ship." How do you respond?
**[Say this]** "I'd say I want to unblock them fast, but admin on a vault violates least privilege and separation of duties — key *administration* and key *usage* must be different roles. So I'd grant a scoped role: use-only permissions on the specific keys or secrets their service needs, ideally via a golden IaC module so it's repeatable and auditable. If they need it now, I provision that scoped access in minutes. The outcome is they ship today *and* we pass the next audit — those aren't in conflict if the tooling is good."

### S3. Certificates keep expiring and causing outages. How do you fix it permanently?
**[Say this]** "One-off renewals are toil and they'll recur, so I fix the *system*. I build an inventory of every cert and its expiry — discovery is usually the missing piece. Then automated renewal: ACME with cert-manager for internal PKI, or a Venafi/CLM integration for enterprise certs, so renewal happens without humans. I add monitoring that counts down to expiry with multi-week lead alerts, not day-of. And I make issuance self-service through a pipeline so teams don't hand-craft certs. The result is expiry stops being an incident category at all."

### S4. During an incident, two engineers disagree on the fix while the bridge is live. What do you do as IC?
**[Say this]** "As Incident Commander my job is coordination, not being the smartest person. I keep the focus on *mitigation* first — the fastest safe action to restore service, usually rollback or failover — and defer the deeper root-cause debate to the postmortem. I'll make a timeboxed decision: 'We roll back now; if that doesn't work in 10 minutes, we try your approach.' Clear ownership, one decision-maker, and reduce the debate to a reversible experiment. Ego doesn't restore service; structure does."

### S5. You inherit a service with no SLOs and constant pages. Where do you start?
**[Say this]** "I start by listening to the data. I pull the last few months of incidents and alerts to see what actually pages and what actually hurts users. I define one or two meaningful SLIs from the user's perspective — availability and latency of the core operation — and set a realistic SLO from historical performance. Then I ruthlessly prune alerts: anything that isn't actionable or user-impacting gets deleted or downgraded to a ticket, and I switch to SLO burn-rate alerting. That usually cuts page volume dramatically in the first few weeks and buys the team room to fix root causes instead of firefighting."

### S6. How would you detect and respond to a suspected key compromise?
**[Say this]** "This is a security incident, so I engage the security/CSIRT team immediately and preserve forensic evidence. Detection signals: anomalous key-usage patterns in audit logs — unusual callers, geographies, volumes, or off-hours access. Containment first: rotate or revoke the affected key and disable the compromised credentials, then re-encrypt affected data under a new key using envelope encryption so I only re-wrap DEKs where possible. In parallel, assess blast radius — what data or sessions were exposed — and follow the bank's regulatory breach-notification timelines. Then a blameless postmortem with systemic fixes like tighter access, better anomaly detection, and shorter key crypto-periods."

### S7. Your team must migrate on-prem HSM-backed keys to a cloud KMS with zero data loss. How?
**[Say this]** "I'd use BYOK — generate or export key material securely from the on-prem HSM under dual control and import it into the cloud KMS so the same key can decrypt existing data, meaning no mass re-encryption up front. I run old and new in parallel: new writes use the cloud KMS, reads still work against legacy ciphertext. I migrate data lazily or with a controlled re-encryption job, tracking progress. Throughout, I keep the on-prem HSM as fallback until cutover is verified, monitor decrypt error rates as my safety signal, and only decommission the old path once everything's confirmed migrated. Reversibility and dual-run are what make it zero-loss."

### S8. A compliance audit finds you're using a deprecated TLS version and weak ciphers across many services. How do you remediate at scale?
**[Say this]** "I codify the fix once and roll it everywhere. I define the compliant standard — TLS 1.2 minimum, ideally 1.3, and an approved cipher suite list — as a configuration in code. I apply it across the estate through IaC and config management pipelines rather than touching each service by hand. I add a policy-as-code check so any new service that tries to use weak TLS fails the pipeline — that stops regression. I roll it out staged with canary and monitoring because a cipher change can break older clients, and I keep an exception process with compensating controls for anything that genuinely can't upgrade yet. Then I produce the re-scan evidence for the auditor."

### S9. Auth latency is fine at p50 but terrible at p99. How do you approach it?
**[Say this]** "Averages lie, so I go straight to the tail — p99 and p999 — and trace where the time goes. Tail latency usually comes from a specific cause: a slow dependency under contention, garbage-collection pauses, cache-miss stampedes, connection-pool exhaustion, or lock contention. Distributed tracing across the login flow tells me which hop spikes. Common fixes: bounded caches with jittered TTLs to avoid stampedes, connection pooling and timeouts, circuit breakers on the slow dependency, and right-sizing. I optimize the tail because in identity a slow login *is* a failed login to the user."

### S10. Leadership wants a new feature shipped, but you've blown the error budget. What do you say?
**[Say this]** "I'd bring the data, not an opinion. I'd show that we're over our error budget, which per our agreed policy means we pause risky launches and stabilize first — that policy exists precisely so this isn't an emotional argument during a crunch. I'd frame it as risk: shipping now, on an already-fragile service, likely means an outage that costs more than the delay. Then I'd offer a path: here are the top two reliability fixes, here's the time to get back in budget, and we can ship behind a feature flag with a canary once we're healthy. Reliability and delivery aren't enemies; the budget is how we balance them objectively."

---

# PART 2 — CHALLENGES I FACED AND RESOLVED (First-Person STAR Stories)

> **⚠️ MAKE THESE YOURS.** Replace [bracketed] items with your real tools, numbers, team, and timeframe. Each is designed to answer "Tell me about a challenge you faced and how you resolved it." Pick the 5–6 that best match your actual experience and rehearse those.

## Challenge 1 — Repeated authentication outages from an unreliable directory
**Situation:** "At [company], our enterprise authentication platform depended on [Active Directory / LDAP], and we were having recurring outages — roughly [once or twice a month] — where login latency spiked and users across multiple apps couldn't sign in. It was our top source of Sev1s."

**Task:** "I was asked to own the reliability of the auth path and stop the recurring outages."

**Action:** "I started by instrumenting it properly — I defined SLIs for auth success rate and p99 latency, added distributed tracing across the login flow, and set up SLO burn-rate alerting so we'd catch degradation early instead of after users complained. The tracing showed the root cause was [directory replication lag and a single overloaded read path]. I added [geographically distributed read replicas], introduced [caching of frequent group/attribute lookups with bounded, jittered TTLs], and put [timeouts and circuit breakers] on the directory calls so a slow directory couldn't cascade into total auth failure. I also automated failover and validated it with a game day."

**Result:** "Availability went from about [99.8%] to [99.97%], p99 auth latency dropped roughly [40%], and auth-related pages fell around [60%]. More importantly, it stopped being our top incident category."

**Lesson:** "You can't fix what you can't measure. Observability plus circuit breakers turned a recurring firefight into a stable, predictable service."

## Challenge 2 — Leading vulnerability remediation at scale under a tight deadline
**Situation:** "We received a large batch of vulnerability findings — [several hundred to a few thousand], including [criticals] — across [many services], with a compressed remediation deadline driven by [an audit / a security program]."

**Task:** "I had to coordinate remediation across teams without causing outages on mission-critical systems."

**Action:** "First I aggregated and deduplicated findings from [multiple scanners] into one view. Then I prioritized by *real* risk — not just CVSS, but combined with exploitability signals like EPSS and CISA's known-exploited list, plus asset exposure — so internet-facing and identity-critical systems went first. I assigned clear ownership with SLAs, and instead of manual one-off fixes, I remediated at scale through [IaC and patch pipelines] with canary rollout and automated rollback. For systems I couldn't patch immediately, I put compensating controls in place with documented risk acceptance. I tracked a burn-down chart and reported weekly to leadership, and I added [policy-as-code checks] so the same findings couldn't reappear."

**Result:** "We closed [X%] of criticals within the deadline with [zero] remediation-caused outages, and the guardrails eliminated the recurring findings entirely in the next scan cycle."

**Lesson:** "Remediation at scale is an automation and prioritization problem, and it has to be done with SRE safety so the fix doesn't become the outage."

## Challenge 3 — A KMS/HSM dependency became a reliability risk
**Situation:** "A set of our services called [AWS KMS / the HSM] on nearly every request for encryption and decryption. Under peak load we started hitting [KMS throttling], which caused request timeouts and a partial outage."

**Task:** "I needed to eliminate the crypto layer as a scaling bottleneck without weakening security."

**Action:** "I moved us to proper envelope encryption with client-side data-key caching — using [the AWS Encryption SDK's data key caching] with a bounded TTL and usage limits — so we generated a data key once and reused it for many operations instead of calling KMS per request. That cut KMS call volume by [~90%]. I added exponential backoff with jitter on the KMS client to avoid retry storms, put a circuit breaker around the crypto calls, requested a quota increase, and set up monitoring and alerting on KMS throttle metrics. I also moved toward [multi-region keys] so load and failure were spread."

**Result:** "KMS throttling errors dropped to effectively zero, crypto-operation latency at p99 improved [substantially], and we removed a tier-0 single point of failure — all while keeping the master key inside the HSM."

**Lesson:** "Cloud KMS is a tier-0 dependency; you design for its limits and failure the same way you would a database."

## Challenge 4 — Certificate expiry causing surprise outages
**Situation:** "We had a painful outage when a [signing / TLS] certificate expired unnoticed and took down [a critical service / authentication]. It turned out we had no reliable inventory of certs or their expiry dates."

**Task:** "I was asked to make certificate expiry a non-event going forward."

**Action:** "I built a discovery process to inventory every certificate and its expiry across the estate. Then I automated renewal — [cert-manager with ACME for internal PKI / a Venafi integration] — so renewal happened without human involvement. I added expiry monitoring with [multi-week lead-time alerts] instead of same-day, and made certificate issuance self-service through a pipeline so teams stopped hand-crafting certs. I also ran a blameless postmortem on the original outage and tracked the action items to closure."

**Result:** "Certificate expiry went from a recurring incident category to zero expiry-related outages since. Renewal became fully automated toil-free work."

**Lesson:** "Most 'surprise' outages aren't surprises — they're missing inventory and automation. Fix the system, not the symptom."

## Challenge 5 — Standardizing and decommissioning a non-compliant legacy solution
**Situation:** "We had a legacy [key management / secrets] solution that was non-compliant — [weak crypto / unmanaged keys / end-of-life] — and several teams still depended on it. Simply turning it off would have broken production."

**Task:** "Migrate everyone to the approved, compliant standard and safely decommission the old system."

**Action:** "I inventoried every consumer and their dependencies. I defined the compliant target state — [centralized KMS / Vault with proper rotation and audit] — and built golden IaC modules so teams could adopt it correctly by default. I migrated consumers in waves, dual-running old and new in parallel so I could validate and roll back safely, with decrypt/auth error rates as my safety signal. Once each wave was fully cut over and verified, I decommissioned that slice. Finally I added policy-as-code so the deprecated solution couldn't be reintroduced. I communicated heavily with stakeholders throughout because the blast radius was enterprise-wide."

**Result:** "We fully retired the non-compliant system with [zero] production impact, closed the related audit findings, and standardized [N] teams onto one supported, auditable platform."

**Lesson:** "Decommissioning is a reliability exercise as much as a security one — dual-run and reversibility are what make an aggressive security change safe."

## Challenge 6 — Excessive alert noise and on-call burnout
**Situation:** "When I joined [team], on-call was drowning — [hundreds of alerts a week], most non-actionable. Engineers were burning out and real signals were getting lost in the noise."

**Task:** "Make on-call sustainable and ensure real incidents actually got caught."

**Action:** "I audited every alert against two questions: is it actionable, and is it tied to user impact? Anything that failed both got deleted or downgraded to a ticket. I replaced cause-based alerts like 'CPU high' with symptom-based SLO burn-rate alerts. I made sure every remaining page had a runbook and clear ownership, and I instituted a rule that any repeat page automatically generated a ticket to fix the root cause. I also tracked page volume as a health metric the team reviewed weekly."

**Result:** "Page volume dropped roughly [70%], mean time to acknowledge improved because signals were no longer buried, and on-call satisfaction went up measurably. We didn't miss real incidents — detection actually improved."

**Lesson:** "Alert fatigue is a reliability risk. Fewer, better alerts catch more real problems than a wall of noise."

## Challenge 7 — A high-severity incident I commanded end-to-end
**Situation:** "We had a Sev1 where [authentication failed enterprise-wide] — users across many downstream apps couldn't log in. Huge blast radius."

**Task:** "I took the Incident Commander role to coordinate the response and restore service fast."

**Action:** "I declared the Sev1, opened a bridge, and notified downstream teams and leadership. I focused the room on scope first — it was [all users, one region / one IdP] — and immediately checked recent changes, since most incidents follow a change. We found [a recent config change / an expired signing cert]. I made the call to [roll back / fail over] as the fastest safe mitigation and set a timebox: if it didn't recover in [10 minutes], we'd try the alternative. I kept communications flowing at fixed intervals. Once service was restored, I verified with synthetic auth probes."

**Result:** "We restored service in about [X minutes]. In the blameless postmortem we identified the systemic gap and fixed it — [automated cert renewal with multi-week alerts / staged rollout with canary] — so it couldn't recur, and I tracked every action item to closure."

**Lesson:** "In a major incident, mitigate first and root-cause later. Calm structure and clear ownership matter more than being the smartest person in the room."

---

# PART 3 — SENIOR-LEVEL IMPLEMENTATION WALKTHROUGHS

> These are "tell me about something you *built* / *designed* and how" answers. They show senior depth. Present them in first person; adjust tools to what you've actually used.

## Implementation 1 — Automated key rotation with zero downtime
**What I built:** "An automated key-rotation system for [our KMS-backed encryption] so we stopped doing risky manual rotations."

**How I implemented it:**
- "I used envelope encryption, so rotation meant rotating the KEK in [KMS] while old key versions stayed available to decrypt existing data — no mass re-encryption required."
- "I set up scheduled rotation via [KMS automatic rotation / a scheduled Lambda] and a stage-then-promote pattern: create the new key version, run automated validation that encrypt/decrypt works, then promote it as current while keeping the previous version for rollback."
- "Applications fetched keys at runtime, not at build, and handled version references in the ciphertext metadata, so a rotation never required a redeploy."
- "I added monitoring on rotation success, key age, and decrypt error rate as the safety signal, with alerts if a key exceeded its crypto-period."

**Result:** "Rotation became fully automated and downtime-free, we met the [PCI / internal] rotation-period requirement automatically, and it eliminated a whole class of manual toil and human error."

## Implementation 2 — Golden Terraform module for compliant-by-default KMS keys
**What I built:** "A reusable Terraform module so any team that needed a key got a compliant one by default."

**How I implemented it:**
- "The module provisioned a [KMS key] with rotation enabled, mandatory tagging, a key policy enforcing separation of duties — separate admin and usage roles — and audit logging wired in."
- "It refused insecure configurations, and I published it as a versioned module in our internal registry."
- "In CI I added policy-as-code checks — [Checkov / tfsec / OPA-Conftest] — that blocked any key created outside the module or with non-compliant settings before apply."
- "State lived in a secure remote backend with locking and encryption, and the pipeline used OIDC for short-lived credentials rather than static keys."

**Result:** "New keys became compliant by default, we stopped getting audit findings for misconfigured keys, and provisioning went from a manual ticket to self-service in minutes."

## Implementation 3 — End-to-end observability for the identity platform
**What I built:** "A unified observability framework across our identity and KMS platforms so we could see reliability and audit in one place."

**How I implemented it:**
- "Metrics for the golden signals plus identity-specific SLIs — auth success rate, token issuance latency, directory replication lag, KMS op latency and throttles — in [Prometheus/Grafana or Datadog]."
- "Structured audit logs of every authN/authZ decision and key operation — who, what, when, from where, allow or deny — carefully redacting any sensitive material, which served both debugging *and* compliance."
- "Distributed tracing with [OpenTelemetry] across the whole login flow so we could pinpoint which hop caused latency, with correlation IDs tying metrics, logs, and traces together."
- "SLO dashboards with multi-window burn-rate alerting, plus security-oriented alerts like spikes in auth failures signaling a possible attack."

**Result:** "Mean time to detect and diagnose dropped sharply, and the same telemetry doubled as audit evidence, so we served reliability and compliance with one system instead of two."

## Implementation 4 — Secrets management migration to short-lived dynamic secrets
**What I built:** "Migrated services off long-lived static credentials onto [HashiCorp Vault] dynamic secrets."

**How I implemented it:**
- "Stood up Vault with [KMS auto-unseal] so the HSM/KMS protected the root of trust, and enabled the [database/cloud] secrets engines to generate short-lived credentials on demand with a TTL and automatic revocation."
- "Solved the 'secret zero' bootstrap problem using platform identity — [Kubernetes service-account auth / cloud IAM] — so workloads authenticated to Vault without a pre-shared secret."
- "Added secret scanning in CI to catch hardcoded credentials and migrated apps to fetch secrets at runtime with refresh-on-failure handling."

**Result:** "We eliminated most standing static credentials, dramatically shrank the blast radius of any leak, and got per-consumer auditability. Rotation became automatic instead of a manual project."

---

# PART 4 — AI-ASSISTED WORK (How I Use AI to Save Time)

> This program (Mythos) is *AI-driven*, so showing you use AI thoughtfully — with security guardrails — is a strong signal. Frame it as: AI accelerates me, but I always keep a human in the loop and never feed it secrets. **Adjust to the tools you actually use.**

### Q: How do you use AI in your day-to-day SRE work?
**[Say this]** "I treat AI as a force multiplier for the toil-heavy parts of the job, with a human always reviewing before anything touches production. A few concrete ways: I use [GitHub Copilot / Cursor / an internal LLM] to scaffold Terraform modules, Python and Bash automation, and unit tests, which turns a half-day of boilerplate into an hour of review. I use it to draft and refactor runbooks and postmortems from incident notes. And I use it for faster root-cause analysis — summarizing large log dumps, explaining unfamiliar stack traces, or generating a regex or a query I'd otherwise fiddle with. The key discipline is I never paste secrets or sensitive data into a general model, and I always review AI output before it ships — it accelerates me, it doesn't replace judgment."

### Concrete AI use-cases to mention (pick what's true for you):

**1. Writing automation and IaC faster**
"I use AI to scaffold Terraform, Ansible, Python, and Bash — for example generating a first draft of a key-rotation script or a golden module — then I review, harden, and test it. It easily cuts my scripting time by [half]."

**2. Vulnerability remediation acceleration (very relevant to Mythos)**
"For remediation work, AI helps me triage and summarize large volumes of findings, draft remediation steps for a given CVE, and generate the config or patch changes — which is exactly the kind of volume-plus-tight-timeline problem this program has. I still apply human risk-prioritization and test everything, but AI removes the grunt work of researching each finding."

**3. Incident response and RCA**
"During and after incidents I use AI to summarize noisy log streams, explain an unfamiliar error, and draft the postmortem timeline from the bridge notes — so the write-up is done while it's fresh instead of days later."

**4. Documentation and knowledge sharing**
"I use it to turn tribal knowledge into runbooks and to keep docs current, which directly reduces on-call toil and helps mentoring."

**5. Log/query/regex generation and analysis**
"I use it to quickly write Splunk/PromQL/KQL queries and regexes, and to spot anomalies in log samples — tasks that used to eat time."

**6. Code review and security review assist**
"I use AI as a first-pass reviewer to flag obvious issues before a human review — never as the final gate, but it catches easy things early."

### Q: What are the risks of using AI, and how do you manage them?
**[Say this]** "Three main risks, and I manage all three. First, data leakage — I never put secrets, keys, PII, or proprietary code into a public model; I use approved enterprise/private tooling and redact. Second, hallucination — AI can produce confident but wrong output, so everything gets human review and testing before production; I treat it as a draft, not truth. Third, over-reliance — I make sure I still understand the systems myself, because in an incident you can't outsource judgment. In a regulated bank, governance around AI use is essential, and I stay inside those guardrails."

### Q: How would AI-generated findings (like Mythos) change how an SRE works?
**[Say this]** "An AI platform like Mythos surfaces a huge volume of findings — often more than humans could find manually — which is powerful but shifts the bottleneck to *triage and safe remediation at scale*. That's squarely an SRE strength: I'd normalize and prioritize the findings by real risk, automate remediation through IaC and pipelines rather than manual fixes, and apply reliability discipline — canary, rollback, error budgets — so we remediate fast *without* causing outages. AI finds the problems; SRE fixes them safely and permanently with guardrails so they don't recur. The two are complementary."

### One-liner if they ask "are you comfortable with AI tooling?"
**[Say this]** "Very — I use it daily to move faster on automation, remediation, and documentation, always with a human in the loop and never with sensitive data in a public model. Given this program is AI-driven, I see that as a real advantage I bring."

---

# QUICK REFERENCE — Which story for which question

| If they ask… | Use… |
|---|---|
| "Improved reliability of a critical system" | Challenge 1 (directory) or Impl 3 (observability) |
| "Handled a major incident" | Challenge 7 (IC) + scenario S4 |
| "Vulnerability / audit remediation at scale" | Challenge 2 + scenario S8 |
| "Dealt with a scaling/performance problem" | Challenge 3 (KMS throttling) or S9 (tail latency) |
| "Fixed recurring toil / automated something" | Challenge 4 (certs) or Impl 1 (rotation) |
| "Standardized / decommissioned / drove change" | Challenge 5 + scenario S7 |
| "On-call / alert fatigue" | Challenge 6 + scenario S5 |
| "Built something from scratch (senior depth)" | Any Implementation 1–4 |
| "How do you use AI / save time" | Part 4 |
| "Security incident / key compromise" | Scenario S6 |

---

## FINAL REMINDERS
- **Customize the brackets.** Generic stories sound rehearsed; real numbers and tool names sound credible.
- **STAR every time**, end with a **quantified result + a lesson**.
- Every answer ties back to **reliability + security + compliance**.
- On AI: **human-in-the-loop, no secrets in public models, AI accelerates but doesn't replace judgment.**
- Be honest about tool gaps and pivot to transferable fundamentals.

You have the material — now rehearse the ⭐ stories out loud. Good luck on the 7th.



