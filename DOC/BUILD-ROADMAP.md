# JOBEN ENTERPRISE — Build Roadmap

**Sumber kebenaran:** [`PRD-JOBEN-ENTERPRISE.md`](./PRD-JOBEN-ENTERPRISE.md)  
**Status:** roadmap engineering awal; belum ada capability aplikasi yang boleh
  diklaim `live_verified`.  
**Bahasa kerja:** Indonesia; istilah teknis mengikuti kontrak PRD.  
**Tujuan:** membangun produk yang dapat dipakai dari kondisi nyata sampai operasi
  produksi, tanpa melewati gate kebenaran data, keamanan, dan recovery.

> “Complete” di roadmap ini berarti setiap capability yang dirilis memiliki
> lifecycle utuh, source of truth, failure path, test, observability, runbook,
> dan proof record. Ini bukan janji bahwa sistem bebas bug selamanya; setiap
> perubahan provider, dependency, rule, dan threat harus masuk verification
> ulang.

---

## 1. Aturan prioritas

Urutan build ditentukan oleh risiko, bukan oleh halaman yang paling mudah
dibuat:

1. Kebenaran hasil scan dan isolasi tenant.
2. Keamanan credential dan least privilege.
3. Scan engine, evidence integrity, dan provenance.
4. Aggregation/scoring yang dapat direbuild dan diaudit.
5. Empty/error/degraded UX dan remediation.
6. Report, notification, auditor, dan governance.
7. Billing, AI, growth, dan ekspansi provider.
8. Hardening, compliance operasional, dan GA.

### Aturan yang berlaku di semua phase

- Tidak ada `pass`, `met`, score, `verified`, `paid`, atau `published` dari
  frontend, seed, fixture, AI, default value, atau optimistic response.
- Capability yang belum lulus gate harus tetap
  `planned`/`not_implemented`/`verification_required`/`degraded`.
- Setiap mutation server-side memiliki state machine, authorization,
  idempotency, audit, concurrency guard, dan rollback/recovery yang relevan.
- Setiap data customer memiliki organization scope yang diverifikasi dari
  session/actor context, bukan dipercaya dari request.
- Provider error, permission error, timeout, schema drift, stale, partial, dan
  missing evidence harus terlihat dan tidak boleh menjadi pass.
- Tidak ada credential plaintext di database, log, fixture, evidence, URL,
  screenshot, atau response.
- Satu phase tidak dianggap selesai karena jumlah endpoint, halaman, atau
  coverage test; exit gate dan proof record harus lengkap.

---

## 2. Peta urutan besar

| Urutan | Phase produk | Slice/dependency | Hasil yang boleh diklaim |
|---|---|---|---|
| 0 | Foundation & readiness | S0 / Gate A | Platform aman untuk menerima data customer |
| 1 | AWS evidence vertical slice | S1 / Gate B | Satu alur scan AWS nyata, end-to-end |
| 1B | Provider kedua & operational beta | S2–S3 / Gate C | Beta customer dengan operasi dan report |
| 2 | Trust, auditor, public surface | S4 | Publikasi evidence yang scoped dan dapat dicabut |
| 3 | ISO 27001 | Governance expansion | Framework dan mapping ISO yang terverifikasi |
| 4 | GDPR/CCPA | Privacy expansion | Data map dan workflow privacy yang dapat diaudit |
| 5 | Questionnaire AI | S5 / AI gate | AI retrieval/citation dengan human approval |
| 6 | PCI SAQ assistant | Product expansion | Wizard SAQ, bukan ASV scanning |
| 7 | White-label & auditor marketplace | Commercial expansion | Agency multi-tenant dengan batas tanggung jawab jelas |
| 8 | Hardening & GA | S6 / Gate D | Release produksi dengan SLO, recovery, legal, dan on-call |

Phase 0 sampai 1B adalah jalur MVP live. Phase 2–7 adalah ekspansi setelah
vertical slice terbukti. Phase 8 bukan “tambahan kosmetik”; GA tidak boleh
disamakan dengan MVP beta.

---

# PHASE 0 — Foundation & readiness

**Target PRD:** minggu 1–3  
**Slice:** S0 Foundation truth  
**Prioritas:** P0 — wajib sebelum data customer  
**Exit:** Gate A lulus

## 0.1 Product truth, scope, dan capability registry

**Kerjakan**

- Bekukan MVP: SOC 2 Type II terbatas + AWS read-only cross-account.
- Tetapkan non-goals: provider lain, ISO penuh, privacy suite, AI
  questionnaire, PCI, marketplace, dan white-label belum live.
- Pecah PRD menjadi `CapabilityRecord` per modul, check, connector, report,
  dan AI feature.
- Definisikan status capability, limitation customer-facing, owner, reviewer,
  expiry/reverification, dan dependency.
- Buat `DOC/decision-log.md`, `DOC/capability-registry.md`, serta template
  dossier `DOC/completion/<moduleId>/`.

**Selesai jika**

- Tidak ada fitur yang ambigu antara demo, experimental, dan live.
- Setiap item backlog memiliki requirement, owner, source of truth, actor,
  tenant scope, limitation, SLO impact, cost ceiling, dan runbook reference.

## 0.2 Hosting, vendor, dan environment verification

**Kerjakan**

- Verifikasi SSH, Node 22 atau Node 20, slot Node app, PostgreSQL, disk,
  bandwidth, subdomain, SSL, dan outbound Redis TLS.
- Putuskan environment terpisah: development, staging, provider sandbox, dan
  production.
- Putuskan provider yang benar-benar dipakai melalui RFC: Clerk, PostgreSQL,
  Redis/BullMQ, S3/R2, Sentry, dan payment sandbox.
- Catat region/residency, retention, budget, dan threshold migrasi hosting.
- Buat `DOC/hosting-verification.md`, `DOC/provider-matrix.md`,
  `DOC/data-classification.md`, dan `DOC/threat-model.md`.

**Selesai jika**

- Semua blocker hosting berstatus pass atau memiliki keputusan tertulis,
  owner, risiko, mitigasi, dan rollback.
- Tidak ada secret yang diperlukan tetapi belum memiliki sumber provisioning.

## 0.3 Repository bootstrap dan CI

**Kerjakan**

- Bootstrap Next.js App Router + TypeScript sesuai stack PRD.
- Siapkan Tailwind/shadcn/design tokens, package config, locale
  `en/id/zh`, lint, format, typecheck, unit test, integration test, dan E2E.
- Siapkan route shell marketing/app/API; belum boleh ada compliance data palsu.
- Tambahkan dependency lockfile, Node version, migration command, dan
  environment contract.
- Buat redacted structured logging, correlation ID, error envelope,
  health/readiness check, dan rate-limit boundary.

**Selesai jika**

- Branch bersih dapat install, lint, typecheck, test, dan build di CI.
- Hello World marketing/app dapat dijalankan di staging.
- Semua route sensitif default-deny dan belum menampilkan status compliance.

## 0.4 Database, tenancy, dan audit foundation

**Kerjakan**

- Implementasikan Prisma/PostgreSQL migration baseline untuk `User`,
  `Organization`, membership/invitation, audit, capability, dan idempotency.
- Terapkan organization context resolver dan scoped repository.
- Tambahkan optimistic concurrency/version pada membership dan mutation penting.
- Implementasikan audit append-only dengan actor, purpose, scope, result,
  correlation ID, timestamp UTC, dan reason.
- Siapkan backup harian, retention awal, manifest, checksum, dan restore smoke
  test di environment terisolasi.

**Selesai jika**

- Migration maju dan rollback/backward-compatibility strategy terdokumentasi.
- Query tanpa tenant scope tidak dapat dikompilasi/dipakai pada service boundary
  yang customer-facing.
- Backup dapat direstore dan checksum, migration version, tenant count, serta
  referensi evidence dapat diverifikasi.

## 0.5 Identity, organization, dan RBAC (M-01)

**Kerjakan**

- Clerk sign-up/sign-in, identity sync, MFA policy untuk OWNER/ADMIN/internal,
  session revoke, sign-out, dan recovery.
- Create/join organization, switch organization, invite accept/revoke/expiry/
  replay, membership suspend/revoke/delete.
- Role matrix dan ownership transfer dengan approval, version, serta audit.
- Negative test untuk seluruh route, job, export, webhook, signed URL,
  repository, dan queue.

**Selesai jika**

- Alur `signup → organization → invite → role → revoke session` dapat
  dijalankan tanpa SQL/manual override.
- Dua organization saling terisolasi pada read, write, export, job, dan audit.

## 0.6 Gate A — stop/go

**Wajib lulus**

- Auth/session, tenant authorization, migration, secret handling, logging
  redaction, CI, backup/restore, dan hosting verification.
- Secret scan, dependency audit awal, SAST awal, dan privilege review.
- Dossier M-01 dan foundation M-15 memiliki proof record.

**Keputusan**

- **GO:** boleh mulai S1 dan membuat environment/customer sandbox.
- **NO-GO:** capability tetap non-live; tidak boleh memasukkan customer data,
  seed demo ke jalur live, atau membuat optimistic dashboard.

---

# PHASE 1 — AWS evidence vertical slice

**Target PRD:** minggu 4–8  
**Slice:** S1 Evidence vertical slice  
**Prioritas:** P0 — real work pertama  
**Exit:** Gate B lulus

## 1.1 AWS contract dan credential safety

**Kerjakan**

- Cross-account IAM role read-only dengan External ID.
- Tulis endpoint/permission matrix per check, API/doc revision, rate-limit,
  timeout, pagination, fixture, sandbox account, redaction, dan expiry.
- Implementasikan connect, permission preview, verify identity/scope/
  permission, health, reconnect, revoke, dan degraded/error state.
- Simpan hanya encrypted credential reference; token jangka panjang tidak
  pernah masuk DB/log atau evidence.
- 401/403 tidak di-retry; transient error memiliki retry terbatas.

**Selesai jika**

- Revoke, permission kurang, expired credential, timeout, rate limit, dan
  schema drift menghasilkan status/error yang tepat.
- Histori tidak hilang saat reconnect/revoke.

## 1.2 Scan engine dan job orchestration (M-03/M-04)

**Kerjakan**

- Check registry sebagai single source of truth.
- `ScanRun` append-only dengan state
  `queued/running/cancelling/completed/partial/failed/cancelled/dead_letter`.
- Queue, lease/lock, idempotency, progress per check/resource, retry budget,
  cancellation boundary, circuit breaker, dan dead-letter.
- Adapter versioned, provider request correlation, schema validation, dan
  per-resource outcome.
- Cron/internal job dengan secret header, HMAC/timestamp/replay protection,
  timeout, batch, dan job ledger.

**Selesai jika**

- Duplicate Scan Now/cron menghasilkan satu run.
- Worker loss/stuck lease dapat direcover.
- Satu check gagal tidak menghilangkan hasil resource/check lain.

## 1.3 Delapan AWS checks prioritas

Implementasikan hanya check yang permission dan provider contract-nya sudah
terverifikasi. Urutan internal:

1. root MFA;
2. password policy;
3. CloudTrail aktif;
4. CloudTrail log validation;
5. S3 public access block;
6. S3 encryption;
7. security group exposure;
8. IAM access key age/rotation.

Untuk setiap check:

- evaluator deterministic pass/fail/error/not_applicable;
- stable resource key dan resource identifier;
- `checkId`, `checkVersion`, observedAt, coverage, message, errorCode;
- mapping SOC 2 yang valid, remediation template, fixture resmi, sandbox test;
- unit, contract, negative permission, timeout, schema drift, and manual test;
- dossier check + capability proof record.

## 1.4 Evidence vault dan integrity (M-05)

**Kerjakan**

- Pipeline `collect → redact → schema validate → canonicalize → hash →
  immutable write → link`.
- Canonical JSON versioned, SHA-256 content hash, content-addressed S3/R2,
  encryption, WORM/retention, legal hold, dan integrity status.
- Signed URL pendek, scoped, dapat dicabut; access purpose dan audit.
- Periodic verify, missing object/hash mismatch incident, quarantine, dan
  re-collection.
- Redaction canary untuk nested payload dan secret-like fields.

**Selesai jika**

- `ObservedFact` hanya dibuat dari evidence hash-valid.
- Record final tidak dapat di-update/delete melalui service maupun DB guard.
- Perubahan hash/object hilang terdeteksi dan tidak digunakan oleh evaluator.

## 1.5 Finding, control, score, dan SOC 2 seed (awal M-06)

**Kerjakan**

- Seed hanya control SOC 2 yang benar-benar dipetakan ke live-verified AWS
  checks.
- Evaluasi finding per resource dan mapping control.
- Projection control dan score snapshot versioned, rebuildable dari append-only
  finding.
- Freshness/coverage/dataQuality: no data bukan score nol; error/stale/
  incomplete/missing evidence tidak pernah pass.
- Explainability: control → finding → observed fact → evidence hash → rule
  version → scan/time.

**Selesai jika**

- Rebuild projection menghasilkan hasil identik.
- Formula, denominator, algorithm version, source projection version,
  freshness, coverage, dan limitation tersimpan.

## 1.6 Dashboard, onboarding, remediation awal (M-07/M-08)

**Kerjakan**

- Onboarding `signup → connect → verify → scan → progress → inspect`.
- Integrations, scan list/detail, control list/detail, finding, evidence
  metadata, dan remediation.
- Remediation lifecycle `open → acknowledged → in_progress → blocked →
  resolved → closed`, reopen, assignee, due date, comments, fix evidence,
  re-scan verification.
- UI state berbeda untuk loading, empty, queued, partial, stale, error,
  degraded, forbidden, dan capability belum tersedia.
- UI/API parity dan i18n string contract; tidak ada fallback diam-diam.

**Selesai jika**

- Close hanya menerima finding verifikasi terbaru, scope sama, evidence valid,
  dan closure reason.
- Perbaikan pada AWS sandbox lalu re-scan mengubah finding dan agregasi secara
  benar.

## 1.7 Gate B — stop/go

**Wajib lulus**

- AWS integration verify, minimal satu check end-to-end, evidence immutable,
  hash re-verification, deterministic finding, control projection,
  freshness/coverage, dan negative permission test.
- Delapan check yang diklaim memiliki proof record; check lain tetap
  `verification_required`.
- Test wajib scan PRD §17.2 nomor 1–9 lulus.
- Dua organization lulus cross-tenant regression pada UI/API/job/evidence.

**Keputusan**

- **GO:** boleh masuk beta terbatas dan menyelesaikan S3.
- **NO-GO:** jangan menambah provider/AI/billing; perbaiki truth/security lebih
  dahulu.

---

# PHASE 1B — Provider kedua & operational beta

**Target PRD:** minggu 9–14  
**Slice:** S2 Decision system + S3 Customer operating surface  
**Prioritas:** P0/P1 — beta customer

## 1B.1 GitHub App read-only (M-02/M-04)

- App installation, organization/repository scope, permission preview,
  verification, revoke/reconnect, health, rate-limit, pagination.
- Hanya check dengan endpoint/permission/sandbox yang sudah dibuktikan.
- Provider contract test, fixture, schema drift, revoked permission, outage,
  dan GitHub API deprecation behavior.
- Google Workspace **tidak** dimulai sampai API sharing/policy diverifikasi
  secara resmi dan RFC/provider matrix disetujui.

## 1B.2 Report & export (M-09)

- Snapshot dari projection version eksplisit; UI dan PDF memakai snapshot sama.
- Queue report state `requested/queued/generating/ready/expired/revoked/failed`.
- PDF/CSV deterministic, input hash, content hash, provenance appendix,
  disclaimer, citation, signed scoped access, expiry, revoke, access audit.
- Uji dua PDF reader dan mismatch snapshot.

## 1B.3 Notification & critical drift (M-10)

- Event eligibility, preferences, quiet hours, emergency policy, dedupe key,
  enqueue, provider adapter, retry, escalation, acknowledge, dead-letter.
- Pass → fail critical drift menghasilkan alert dengan delivery timestamp.
- Target p95 critical delivery <15 menit; provider response selalu redacted.

## 1B.4 Queue, resilience, dan operator surface (M-15 partial)

- Pause/resume integration atau queue, retry/replay dead-letter, circuit
  breaker, stuck scan recovery, provider outage state.
- Internal dashboard: scan success/partial/error, duration, backlog,
  permission error, circuit breaker, evidence failure, alert latency.
- Sentry/error tracking tanpa token/raw evidence.
- Backup/restore drill dan runbook incident/provider outage/stuck scan/
  evidence integrity/credential revoke.

## 1B.5 Billing sandbox (M-13 partial)

- Catalog version, Xendit sandbox checkout, signed webhook inbox, subscription
  state, entitlement projection, renewal/failure/cancel/refund,
  reconciliation.
- Payment tidak pernah menulis finding/evidence/control status.
- Raw card data tidak disimpan; replay/out-of-order/signature failure diuji.
- Entitlement deny server-side dan limitation transparan.

## 1B.6 Gate C — customer beta

- Contract/sandbox comparison, negative permission, schema drift, restore drill,
  incident runbook, report auditability, notification delivery, dan
  reconciliation lulus.
- Semua check yang disebut live punya proof record dan expiry.
- Tidak ada critical security finding yang belum diterima tertulis.
- Support owner, customer limitation, beta feedback, dan rollback tersedia.

---

# PHASE 2 — Trust, auditor, dan public surface

**Target PRD:** setelah Phase 1B  
**Prioritas:** P1 — customer/auditor trust

## 2.1 Marketing site dan public information

- Marketing site penuh dengan copy yang tidak melebihkan capability.
- Public chatbot hanya jika AI gate sudah lulus; rate-limit dan abuse
  protection wajib.
- Halaman pricing menjelaskan scope, plan, retention, limitation, dan status
  provider secara jujur.

## 2.2 Trust Page (M-12)

- Draft → published → unpublishing → unpublished.
- Hanya capability `live_verified`, evidence eligible, scoped, tidak expired,
  dan bukan demo/stale yang boleh dipublish.
- Publication menyimpan eligibility snapshot, expiry, publisher, audit, dan
  dapat dicabut tanpa menghapus histori.

## 2.3 Auditor Portal

- Magic link read-only berbatas waktu, scope-bound, purpose-bound, revocable.
- Evidence citation/provenance dan report snapshot dapat diverifikasi.
- Cross-organization, scope escalation, expired/revoked link, demo leakage,
  dan unpublish history diuji.

## 2.4 SOC 2 operating loop

- Regulation monitor dari sumber resmi untuk SOC 2.
- JOBEN dogfooding SOC 2 Type I.
- WhatsApp critical alert hanya setelah provider, budget, legal, dan delivery
  policy diputuskan melalui RFC.

**Exit gate:** publikasi hanya menampilkan evidence yang eligible; auditor
dapat memverifikasi sumber, hash, waktu, scope, dan status capability.

---

# PHASE 3 — ISO 27001

**Target PRD:** minggu 18–23  
**Prioritas:** P1 — framework expansion

## 3.1 Framework and mapping

- Model framework versioned; import 93 Annex A controls dari sumber resmi yang
  diverifikasi.
- Mapping SOC 2 ↔ ISO dengan owner, rationale, coverage, dan version.
- Perubahan mapping menghasilkan impact report dan tidak mengubah histori
  finding lama.

## 3.2 Gap analysis and workflow

- Gap analysis scoped per organization, readiness snapshot, evidence reuse,
  control owner, remediation, review/approval, dan audit trail.
- Test denominator, not-applicable reason, stale evidence, dan control
  projection rebuild untuk ISO.

## 3.3 Additional connectors

- Azure/GCP/Okta/Vercel/Supabase/Firebase satu per satu, bukan sekaligus.
- Setiap provider harus melalui provider matrix, permission sandbox, fixture,
  schema drift, proof record, expiry, dan capability gate.

**Exit gate:** framework, mapping, dan setiap check provider baru memiliki
provenance serta tidak memperluas status customer tanpa verification.

---

# PHASE 4 — GDPR/CCPA privacy operations

**Target PRD:** minggu 24–29  
**Prioritas:** P1 — privacy expansion

## 4.1 Data inventory

- PII data map per system/vendor/region/purpose/classification.
- Retention, deletion, legal hold, residency, export, and subject request
  policy versioned.
- Cookie scanner dengan sumber/observedAt dan hasil yang dapat diaudit.

## 4.2 DSAR and privacy workflows

- Request lifecycle, identity verification, scope, deadline, owner, review,
  export/delete, exception legal hold, completion evidence, dan audit.
- DPA generator hanya memakai template/version/source yang disetujui.
- Breach timer 72 jam sebagai workflow/alert, bukan klaim legal otomatis.

## 4.3 Vendor risk

- Vendor lifecycle, risk assessment, treatment, review due date, approval,
  source hash, impact propagation, dan notification.

**Exit gate:** privacy feature tidak menghapus audit/evidence yang harus
dipreservasi; setiap regional/residency claim diverifikasi terhadap vendor nyata.

---

# PHASE 5 — Questionnaire AI

**Target PRD:** minggu 30–35  
**Prioritas:** P1 — hanya setelah evidence system stabil

## 5.1 AI gateway safety

- Classify request → scoped retrieval → redact → model call → schema validate →
  citation verify → answer/refusal → usage log.
- Retrieval selalu organization/role/purpose scoped.
- AI tidak boleh menulis compliance status, evidence, subscription, atau
  published policy.

## 5.2 Questionnaire workflow

- Upload PDF/Excel/CSV, parsing, item extraction, evidence matching,
  confidence, source citations, draft answer, human review, approval, send/export.
- `humanApproved=true` wajib sebelum send/export.
- Retention/deletion untuk uploaded documents dan derived data.

## 5.3 Evaluator and cost controls

- Golden set answerable/unanswerable, insufficient evidence, invalid citation,
  prompt injection, cross-tenant, stale/integrity-failed evidence, legal claim,
  unsafe fallback, dan provider outage.
- Model routing, token/cost budget per org/feature/provider, AI disable switch,
  fallback refusal, dan `AiUsageLog`.

**Exit gate:** citation dapat ditemukan dan diverifikasi; refusal aman; tidak ada
AI output yang dapat mengubah truth system tanpa approval manusia.

---

# PHASE 6 — PCI SAQ assistant

**Target PRD:** minggu 36–39  
**Prioritas:** P2 — expansion

- Wizard pemilihan SAQ berbasis scope yang dijawab user dan evidence yang
  tersedia.
- Readiness checklist, source/citation, remediation, review, approval, dan
  export audit.
- Jelaskan secara permanen bahwa produk **tidak** membangun ASV scanning.
- Uji scope ambiguity, unsupported answer, stale evidence, dan no-evidence
  refusal.

**Exit gate:** assistant tidak menyajikan sertifikasi atau claim compliance;
output yang normatif wajib human review.

---

# PHASE 7 — White-label & auditor marketplace

**Target PRD:** minggu 40–43  
**Prioritas:** P2 — scale/commercial

## 7.1 Agency multi-tenant

- Agency → sub-organization boundary, delegated role, billing/entitlement,
  support access, data residency, dan audit.
- Custom branding/domain hanya mengubah presentation; tidak mengubah evidence
  truth, capability status, atau limitation.

## 7.2 Auditor marketplace

- Marketplace hanya sebagai penghubung, bukan penjamin kualitas auditor.
- Profile, invite, scope, contract/consent, access expiry/revoke, conflict
  boundary, dan audit.
- Uji delegated access, tenant escape, stale publication, dan offboarding.

**Exit gate:** semua delegated access time-bound/scoped/revocable dan tidak ada
agency/auditor yang dapat menulis compliance truth tanpa workflow resmi.

---

# PHASE 8 — Hardening & General Availability

**Target PRD:** minggu 44–47  
**Prioritas:** P0 untuk GA
**Slice:** S6 Operational readiness  
**Exit:** Gate D lulus

## 8.1 Security and privacy hardening

- Independent pentest; remediate semua critical/high atau dokumentasikan
  acceptance formal dengan owner/expiry.
- Secret scan, SAST, dependency audit, log inspection, redaction canary,
  privilege review, IDOR/cross-tenant, replay, SSRF, rate-limit, and abuse
  regression.
- Review ToS, Privacy, DPA/retention/residency, consent, deletion/legal hold.

## 8.2 Reliability and performance

- Load test pada kapasitas hosting nyata: CRUD p95 <500 ms baseline,
  scan queue start p95 <5 menit normal backlog.
- SLO: API availability 99.5%/bulan, critical alert p95 <15 menit,
  RPO ≤1 jam bila provider mendukung, customer read RTO ≤2 jam.
- Backup restore berkala, checksum/reference verification, migration rollback,
  queue recovery, provider outage, object storage failure, and notification
  replay drills.

## 8.3 Release and operations

- Capability diff, commit SHA, schema/check/algorithm version, lockfile, and
  proof expiry pada setiap release.
- Feature flag default-deny untuk capability non-live.
- Internal operator dapat pause queue, revoke/reconnect, quarantine evidence,
  replay notification, disable AI, restore backup, dan memeriksa tenant/evidence
  tanpa mengubah business history.
- On-call owner, severity policy, status communication, P0/P1 timeline,
  post-incident review, and corrective actions.

## 8.4 GA gate

Gate D hanya lulus bila:

- Fase 1B operational gate, pentest, load test, restore drill, incident
  exercise, legal/privacy review, billing reconciliation, SLO dashboard, dan
  on-call owner disetujui.
- Semua customer-facing capability memiliki dossier lengkap dan proof record
  yang belum expired.
- Semua locale EN/ID/ZH yang diklaim live 100% lengkap.
- Tidak ada critical/high yang tidak memiliki keputusan risiko tertulis.

**Jika gagal:** release dihentikan dan capability diturunkan ke
`not_implemented`, `verification_required`, atau `degraded`. Jangan mengganti
hasil dengan sample data atau optimistic score.

---

## 3. Work breakdown prioritas

Backlog harus dikerjakan dalam urutan berikut. Item di bawah tidak boleh
dilewati dengan alasan UI sudah selesai.

| Prioritas | Work package | Dependency | Output utama |
|---|---|---|---|
| P0 | WP-00 Decision/readiness | none | scope, RFC, hosting/provider/security artifacts |
| P0 | WP-01 App/CI/config | WP-00 | runnable Next.js shell, CI, env contract |
| P0 | WP-02 Tenant/auth/audit | WP-01 | M-01, scoped repository, Gate A |
| P0 | WP-03 AWS credential/contract | WP-02 | M-02/M-04 verified integration foundation |
| P0 | WP-04 Queue/scan engine | WP-03 | M-03 with state, lock, retry, recovery |
| P0 | WP-05 Evidence integrity | WP-04 | M-05 immutable evidence |
| P0 | WP-06 First AWS check | WP-05 | one live-verified check and Gate B candidate |
| P0 | WP-07 Remaining AWS checks | WP-06 | eight checks with proof records |
| P0 | WP-08 Controls/remediation/dashboard | WP-06 | M-06/M-07/M-08 real customer flow |
| P1 | WP-09 GitHub App | Gate B | second provider |
| P1 | WP-10 Report/notification/ops | Gate B | M-09/M-10 and beta operations |
| P1 | WP-11 Billing sandbox | WP-02 + payment RFC | M-13 sandbox, isolated from scan truth |
| P1 | WP-12 Trust/auditor | Gate C | M-12 scoped public/auditor access |
| P1 | WP-13 ISO/GDPR | Gate C + source verification | governance expansion |
| P1 | WP-14 AI questionnaire | stable M-05/M-06 + AI RFC | M-14 safe assistant |
| P2 | WP-15 PCI/white-label/marketplace | relevant gates | expansion modules |
| P0 GA | WP-16 Hardening/GA | all live modules | Gate D |

### Format task implementasi per work package

Setiap task turunan harus memuat:

```text
Requirement/PRD reference
Actor, role, organization scope, data classification
Source of truth and non-goals
State machine and transition guards
Request/response/event schema version
Error, empty, stale, partial, timeout, permission, schema-drift behavior
Idempotency, concurrency, retention, rollback/recovery
Test cases and expected evidence
Metric, alert, SLO, cost ceiling
Runbook and customer limitation
CapabilityRecord and Definition of Ready/Done
```

---

## 4. Definition of Ready dan Definition of Done

### Ready sebelum coding capability

- Requirement, non-goal, actor/role, tenant scope, dan classification tertulis.
- Provider contract, endpoint, permission, rate limit, API/doc revision, dan
  deprecation behavior memiliki sumber resmi.
- State machine, error taxonomy, freshness/coverage, idempotency, dan rollback
  ditentukan.
- Evidence schema, redaction, retention, audit event, fixture, dan acceptance
  test tersedia.
- Owner, SLO impact, cost ceiling, reviewer, dan runbook ditentukan.

### Done sebelum `live_verified`

- Unit, integration, contract, E2E, security, regression, accessibility,
  localization, dan performance test relevan lulus.
- Sandbox/provider nyata cocok dengan expected manual untuk pass, fail, empty,
  partial, permission denied, timeout, dan schema change.
- Output non-pass memiliki resource key, deterministic message, error code,
  evidence ID/hash, observed time, dan rule version.
- Error/stale/incomplete/demo/missing evidence tidak menaikkan score.
- Dua organization lulus negative test di route, repository, queue, report,
  object storage, webhook, dan export.
- Secret scan, SAST, dependency audit, redaction, privilege review, restore,
  retry/lock/replay/circuit-breaker/runbook drill lulus.
- Docs, provider matrix, capability registry, limitation, proof record,
  reviewer berbeda, dan expiry/reverification terisi.

---

## 5. Larangan urutan build

Jangan melakukan hal berikut sebelum dependency dan gate-nya lulus:

- membuat dashboard score sebelum finding nyata dan evidence hash-valid;
- menambahkan provider kedua sebelum AWS vertical slice lulus;
- menambahkan AI sebelum scoped retrieval/citation/refusal evaluator siap;
- membuat Trust Page/auditor output dari data demo, stale, atau capability
  `verification_required`;
- menghubungkan billing sebagai syarat untuk menghasilkan scan truth;
- menganggap CRUD, halaman kosong, mocked API, atau seed sebagai full feature;
- melakukan migrasi hosting besar tanpa threshold, RFC, dan rollback;
- menyimpan credential provider, raw card, signed URL, atau PII sensitif dalam
  repository/fixture/log.

---

## 6. Milestone release

| Milestone | Artinya | Tidak boleh dilakukan |
|---|---|---|
| M0 Foundation ready | Gate A lulus | menerima customer data sebelum ini |
| M1 Evidence proof | Gate B lulus | mengklaim provider/check yang belum verified |
| M1B Customer beta | Gate C lulus | menyebut beta sebagai GA |
| M2 Trust ready | public/auditor eligibility teruji | mempublish stale/demo evidence |
| M3 Governance ready | ISO/privacy scope terverifikasi | menyatakan sertifikasi otomatis |
| M4 Assistant ready | AI golden set dan approval lulus | AI menulis compliance truth |
| M5 GA | Gate D lulus | deploy capability expired/degraded tanpa banner |

Roadmap ini menjadi rencana pembangunan di atas PRD. Ketika implementasi dimulai,
setiap phase harus diterjemahkan menjadi task kecil yang dapat direview dan
dibuktikan; perubahan scope atau keputusan provider yang memengaruhi dependency
wajib masuk decision log/RFC, bukan diputuskan diam-diam di kode.