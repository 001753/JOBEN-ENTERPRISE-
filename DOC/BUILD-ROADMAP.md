# JOBEN ENTERPRISE — Build Roadmap

**Sumber kebenaran:** [`PRD-JOBEN-ENTERPRISE.md`](./PRD-JOBEN-ENTERPRISE.md)  
**Versi roadmap:** 2.0 — PRD-complete phase contract  
**Status:** roadmap engineering; repository belum memiliki capability aplikasi yang
boleh diklaim `live_verified`.  
**Bahasa kerja:** Indonesia; istilah teknis mengikuti kontrak PRD.  
**Tujuan:** membangun produk yang dapat dipakai dari kondisi nyata sampai operasi
  produksi, tanpa melewati gate kebenaran data, keamanan, dan recovery.

**Aturan kelengkapan:** PRD §0–§25 adalah sumber kebenaran. Setiap requirement
harus memiliki phase pemilik, work package, dependency, acceptance evidence, dan
release gate pada dokumen ini. Requirement lintas-phase harus memiliki pekerjaan
atau kontrol berulang pada setiap phase yang menggunakannya; cukup menyebutnya
sebagai “cross-cutting” tanpa pemilik phase tidak dianggap coverage.

> “Complete” di roadmap ini berarti setiap capability yang dirilis memiliki
> lifecycle utuh, source of truth, failure path, test, observability, runbook,
> dan proof record. Ini bukan janji bahwa sistem bebas bug selamanya; setiap
> perubahan provider, dependency, rule, dan threat harus masuk verification
> ulang.

---

## 1. Aturan prioritas dan kontrol lintas-phase

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
- Setiap phase wajib memperbarui `CapabilityRecord`, decision/RFC log, provider
  matrix, data classification, threat model, test evidence, runbook, dan
  customer limitation yang terdampak.
- Setiap phase memiliki **phase owner**, **reviewer independen**, daftar capability
  yang boleh naik status, dan daftar capability yang wajib tetap
  `planned`/`verification_required`/`degraded`.
- Setiap release candidate harus diuji pada tiga kondisi: live customer namespace,
  provider sandbox/disposable account, dan demo namespace yang terisolasi. Data
  demo tidak boleh dipakai untuk membuktikan capability live.

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

### 2.1 Kontrak keputusan phase

| Phase | Input wajib | Output yang boleh diklaim | Tidak boleh diklaim sebelum exit |
|---|---|---|---|
| P0 | PRD baseline, hosting/provider facts, RFC open items | Foundation siap menerima sandbox/customer data sesuai scope | scan, score, finding, live connector, payment, AI |
| P1 | Gate A, AWS contract, disposable AWS sandbox | AWS evidence vertical slice dan checks yang proof-nya valid | provider kedua, score complete bila coverage incomplete |
| P1B | Gate B, provider matrix GitHub, operational owner | Beta customer dengan report, alert, operations, billing sandbox | GA, auditor/public evidence, AI normatif |
| P2 | Gate C, eligible evidence, legal/public copy review | Trust/auditor/public surface yang scoped dan revocable | claim certification, stale/demo publication |
| P3 | Gate C/S4, official ISO source, mapping owner | ISO framework/mapping/checks yang verified per capability | ISO certification/attestation |
| P4 | Gate C/S4, privacy source and residency facts | Privacy operations yang auditable dan policy-bound | legal conclusion atau unverified residency claim |
| P5 | Stable M-05/M-06, AI safety RFC and golden set | AI questionnaire/assistant dengan citation dan human approval | AI-written compliance truth |
| P6 | M-05/M-06, PCI scope source, human-review workflow | SAQ readiness assistant | ASV scanning, certification, normative auto-approval |
| P7 | M-01/M-12/M-13, delegated-access RFC | Agency and auditor marketplace boundaries | delegated write access to compliance truth |
| P8 | All claimed live capabilities, incident/restore/pentest evidence | GA release with SLO, recovery, legal, on-call | release with expired proof or unresolved critical truth/security risk |

Jika input phase belum terpenuhi, implementation boleh hanya berupa contract,
fixture, threat model, atau `verification_required` surface. Capability tidak boleh
melompati phase melalui feature flag, seed, atau UI.

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
3. S3 public access;
4. S3 encryption;
5. CloudTrail;
6. security group exposure;
7. unused access key;
8. full admin inline policy.

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

## 9. Master traceability register — PRD §0–§25

Register ini adalah pemeriksaan kelengkapan, bukan pengganti kontrak detail pada
PRD. Setiap baris memiliki **phase owner**, phase yang melakukan verification
ulang, dan evidence minimum. Requirement baru boleh diberi status selesai hanya
jika proof record-nya direferensikan dari `CapabilityRecord`.

| PRD | Cakupan yang wajib masuk delivery | Phase owner | Verification ulang | Evidence minimum |
|---|---|---|---|---|
| §0 | Anti-false-data, definition of truth, demo isolation, perubahan reversible | P0 | P1–P8 untuk capability terdampak | decision log, threat model, live/demo negative test |
| §1 | Sasaran bisnis, sync ≤6 jam, alert <15 menit, single source of truth, batas klaim | P0 | P1–P8 copy dan metric review | product contract, SLO proof, disclaimer review |
| §2 | P-01 sampai P-08: truth, no fake data, deterministic, evidence, failure, tenant, human approval, least privilege | P0 | Semua phase dan module gate | invariant/security test matrix, capability diff |
| §3 | Persona dan OWNER/ADMIN/MEMBER/AUDITOR/GRC access, MFA | P0 | P1–P7 role regression | permission matrix, E2E role test, audit record |
| §4.1–4.3 | MVP scope, out-of-scope, priority, staged live claim | P0 | Setiap gate dan release | scope baseline, registry, release diff |
| §4.4–4.10 | Full-feature contract, M-01–M-15, dossier, S0–S6, Ready/Done | P0 | P1–P8 per module | module contract, dossier, independent sign-off |
| §5 | Target architecture, stack, hosting prerequisites, external services, RFC rule | P0 | P1B/P8 capacity dan migration review | hosting verification, provider matrix, RFC/rollback |
| §6 | Repository target, scan package, check/capability registry, ModuleContract, status/publication rules | P0 | P1–P8 registry consistency | schemas, registry diff, CI status validation |
| §7 | Scan, cron, idempotency, drift, AWS/GitHub/Google connector/check contracts | P1 | P1B/P3 provider re-verification | sandbox comparison, permission matrix, check dossiers |
| §8 | Finding/control statuses, freshness, coverage, data quality, score/rebuild | P1 | P3/P4 framework regression | deterministic fixtures, rebuild equality, score proof |
| §9 | Evidence lifecycle, redaction, hash/WORM/legal hold, remediation, PDF | P1 | P1B/P2/P5 eligibility review | hash/retention drill, PDF artifact, remediation E2E |
| §10 | Canonical entities, constraints, state machines, domain events, append-only rules | P0 | Every schema/event/migration phase | schema review, transition tests, replay proof |
| §11 | Customer APIs, internal jobs, HTTP envelope, idempotency, cursors, signed access, webhooks | P0 | Every endpoint introduced | OpenAPI/contract, auth, replay, tenant tests |
| §12 | Tokens, screens, truthful states, accessibility, EN/ID/ZH i18n | P0 | P1–P8 touched screens | visual states, a11y/localization test, locale inventory |
| §13 | AI gateway, routing, scoped retrieval, citation/refusal, approval, public FAQ | P5 | P8 disable/fallback drill | golden set, citation proof, cost log, injection test |
| §14 | IDR catalog, payment boundary, regulation monitor, channels, alert policy | P1B | P2–P4/P8 source/reconciliation review | webhook reconciliation, source hash, delivery evidence |
| §15 | Encryption, secrets, rate limit, audit, backup, residency, deletion, NFR, patch SLA | P0 | P1–P8 security/ops gate | secret/log scan, restore, SLO telemetry, residency decision |
| §16 | Phase 0–8 deliverables dan release gates | P0 | Gate A/B/C/D, S4/S5/S6 | signed gate packet and decision |
| §17 | Test pyramid, scan tests 1–10, check Definition of Done | P0 | Test pack tumbuh tiap capability | CI, sandbox expected/actual, two-org regression |
| §18 | Correlation, telemetry, ops dashboard, alerts | P0 | P1–P8 metric/alert review | redacted logs, dashboard, alert timestamps, drills |
| §19 | Ten open decisions dan format RFC | P0 | Sebelum phase dependent dimulai | RFC options/owner/impact/rollback/date |
| §20 | Risk-to-requirement traceability | P0 | Update setiap gate | risk register, linked test/proof IDs |
| §21 | Pre-coding dan merge/deploy checklist | P0 | Mandatory per work package | checklist dan release artifact |
| §22 | Decision/provider/threat/data/runbook/test/capability artifacts | P0 | Artifact delta setiap phase | artifact inventory dan review links |
| §23 | Incident runbooks, restore, quarantine, replay, AI disable, deletion, release discipline | P1B | P2–P8 drills | runbook drill, incident timeline, rollback proof |
| §24 | Definition of Ready/Done, Gate A/B/C/D, downgrade on failure | P0 | Applied setiap phase/capability | gate packet dan status transition audit |
| §25 | Document history dan change control | P0 | Setiap perubahan disetujui | version entry dan RFC/decision reference |

### 9.1 Cross-cutting control pack wajib setiap phase

Kontrol berikut bukan pekerjaan sekali di P0. Phase owner wajib membuat delta
record untuk setiap phase yang mengubah data, provider, UI, capability, atau
operational behavior.

| Control pack | P0 baseline | P1–P4 expansion | P5–P7 expansion | P8 release proof |
|---|---|---|---|---|
| Truth/provenance | status enum, definition, demo isolation | provider evidence/evaluator/projection | governance, AI, public eligibility | release provenance dan expired-proof block |
| Tenant/security | authz, scoped repository, MFA | connector/object/queue/report isolation | delegated access, AI retrieval, billing webhook | pentest, SAST, IDOR/replay/SSRF/rate-limit |
| Lifecycle/recovery | state/event/idempotency | scan/evidence recovery | payment/publication/AI/privacy recovery | restore, rollback, incident exercise |
| Interface | schema/error/correlation | integration/scan/evidence API | report/notification/governance/billing/AI API | compatibility dan migration proof |
| UX/i18n | tokens, locale, truthful state taxonomy | onboarding dan scan states | public/auditor/AI/privacy/commercial copy | locale, accessibility, limitation review |
| Operations | logging, health, CI, backup plan | scan/evidence metrics/runbooks | payment/AI/provider/regulation metrics | SLO dashboard, on-call, cost, release controls |
| Evidence | classification, retention, proof template | hash/redaction/immutability | citation/publication/DSAR/legal hold | retention/deletion/residency/legal sign-off |

### 9.2 Module-to-phase ownership matrix

| Modul | P0 | P1 | P1B | P2 | P3 | P4 | P5 | P6 | P7 | P8 |
|---|---|---|---|---|---|---|---|---|---|---|
| M-01 Identity/RBAC | foundation | regression | beta roles | auditor roles | governance | privacy | AI purpose | approval | delegated | hardening |
| M-02 Integration/Credential | contract | AWS | GitHub/ops | eligibility | new providers | privacy vendors | document source | PCI scope | agency | rotation/pentest |
| M-03 Scan/Jobs | contract | AWS engine | resilience | drift/source | expansion | privacy jobs | questionnaire | SAQ jobs | delegated queues | SLO/recovery |
| M-04 Provider Connector | policy | AWS checks | GitHub | SOC2 source | six providers | privacy sources | parsers/models | PCI source | marketplace | deprecation |
| M-05 Evidence | data contract | AWS vault | report/restore | publication | ISO evidence | legal hold | citation | SAQ source | delegated | integrity drill |
| M-06 Finding/Scoring | projection | AWS/SOC2 | beta | eligibility | ISO | privacy impact | matching only | readiness only | no truth write | rebuild/load |
| M-07 Remediation/Review | lifecycle | AWS | SLA | policy review | ISO gap | DSAR/vendor | AI approval | SAQ approval | delegated review | audit/recovery |
| M-08 Dashboard/App | shell/tokens | onboarding | beta/i18n | public/auditor | frameworks | privacy | questionnaire | SAQ | branding | a11y/locale |
| M-09 Report/Export | contract | boundary | PDF/CSV | auditor | ISO | DSAR/DPA | questionnaire | SAQ | branded | artifact/recovery |
| M-10 Notification | event contract | drift | channels/ops | regulation | ISO | DSAR/breach | AI notices | reminders | marketplace | SLO/replay |
| M-11 Policy/Risk/Vendor/Reg | contract | SOC2 boundary | regulation ops | monitor/dogfood | ISO mapping | GDPR/vendor | AI draft | PCI policy | agency policy | governance |
| M-12 Trust/Auditor | eligibility | blocked | prep | full surface | ISO publish | privacy claims | citation boundary | disclaimer | marketplace | revoke/expiry |
| M-13 Billing/Entitlement | RFC/contract | blocked | sandbox | pricing copy | plan expansion | privacy data | AI cost | PCI plan | agency billing | reconcile |
| M-14 AI Gateway | safety/RFC | disabled | disabled | public FAQ gate | summary boundary | draft boundary | questionnaire | SAQ | marketplace copy | disable/fallback |
| M-15 Admin/Ops | foundation | scan/evidence | beta/restore | public/auditor | governance | privacy | AI cost/safety | SAQ | delegated | full GA |

### 9.3 Phase exit register

Setiap phase hanya boleh ditutup jika seluruh deliverable dan proof minimum
tersedia. Kegagalan satu item menahan phase dan menurunkan capability terdampak
ke `not_implemented`, `verification_required`, atau `degraded`.

| Phase | Requirement focus | Deliverable wajib | Exit proof minimum |
|---|---|---|---|
| P0 / S0 / Gate A | §0, §2–§6, §10–§12, §15–§24; M-01, M-15 foundation | app/CI/env/locale shell; hosting/provider/threat/data artifacts; Prisma/tenant/audit/idempotency; Clerk/MFA/session; demo namespace; health/cron; backup/restore smoke | Gate A packet; two-org authz regression; migration/restore checksum; secret/log scan; open-decision owner/RFC |
| P1 / S1 / Gate B | §7–§9, §10–§12, §17–§18; M-02–M-08 core | AWS External ID connector; permission matrix; queue/lease/retry/cancel; eight PRD checks; immutable evidence; deterministic finding/control; freshness/coverage; remediation/onboarding UI | AWS sandbox expected-vs-actual; PRD §17.2 tests 1–9; eight check dossiers; hash reverify; rebuild equality; two-org regression |
| P1B / S2–S3 / Gate C | §3, §7–§12, §14–§15, §17–§18, §23; M-02/M-04, M-09/M-10, M-13/M-15 partial | GitHub App; deterministic PDF/CSV snapshot; provenance/signed access; notification ledger and drift; queue operator controls; runbooks; billing sandbox/webhook reconciliation | Gate C packet; two-reader PDF; delivery timestamp p95; replay/out-of-order payment tests; restore/incident drill; beta limitation/rollback |
| P2 / S4 | §1, §3, §9, §11–§14, §18–§24; M-08–M-12 | truthful marketing/pricing; Trust Publication lifecycle; scoped auditor access; SOC 2 source monitor; dogfooding; WhatsApp only after RFC | publication eligibility/revoke tests; expired/stale/demo exclusion; auditor scope tests; source hash; copy/legal review |
| P3 | §4.2, §7–§12, §14–§15, §17–§20, §24; M-04/M-06/M-07/M-11/M-12 | versioned 93 Annex A; SOC 2↔ISO mapping; gap/readiness workflow; six additional connectors one at a time | official-source provenance; mapping impact/rebuild; per-provider sandbox/permission/schema-drift proof; no unverified status expansion |
| P4 | §4.2, §5, §10–§11, §14–§15, §17–§24; privacy M-05/M-07/M-08/M-09/M-10/M-11/M-15 | PII/data map; cookie scanner; retention/deletion/legal hold/residency; DSAR; approved DPA; breach timer workflow; vendor risk | scoped DSAR/export/delete evidence; legal-hold preservation; residency/vendor verification; source/version audit; breach alert |
| P5 / S5 | §1–§2, §4.2, §6, §9–§11, §13–§15, §17–§18, §23–§24; M-14 | gateway routing/retrieval/redaction; PDF/Excel/CSV questionnaire; citations/refusal; human approval; usage/cost; disable/fallback | golden set; citation quote verification; cross-tenant/injection/stale/legal refusal; `humanApproved`; cost log; provider outage drill |
| P6 | §4.2, §9, §11–§15, §17, §20–§24; PCI M-06–M-09/M-11/M-14/M-15 | SAQ scope wizard; readiness checklist; citations/remediation; review/approval/export; permanent no-ASV boundary | PCI source/version; ambiguity/no-evidence refusal; approval audit; disclaimer review; no certification claim |
| P7 | §3, §4.2, §6, §9–§12, §14–§15, §17–§18, §20, §23–§24; M-01/M-05/M-07–M-13/M-15 | agency/sub-org isolation; delegated roles/support; branding/domain; billing boundary; auditor marketplace profile/consent/scope/revoke | delegated permission matrix; tenant escape/offboarding/revoke tests; branding cannot alter truth; expiry audit; marketplace limitation |
| P8 / S6 / Gate D | all claimed-live PRD requirements, especially §15, §17–§24 | pentest; security/dependency/log review; load/SLO/RPO/RTO; restore/rollback/outage/replay drills; release manifest; on-call/legal/privacy review | Gate D; non-expired dossiers; 99.5%/p95 targets measured; 100% claimed locales; no unaccepted critical/high truth/security issue; signed release diff |

---

## 10. Work breakdown prioritas

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
| P1 | WP-13 ISO framework/mapping | Gate C + official ISO source | versioned 93 Annex A, SOC 2↔ISO mapping, gap workflow |
| P1 | WP-14 ISO provider expansion | WP-13 + provider RFC per connector | Azure/GCP/Okta/Vercel/Supabase/Firebase, one-by-one proof records |
| P1 | WP-15 Privacy operations | Gate C + privacy source/residency decision | PII map, cookie scanner, DSAR, DPA, vendor risk, breach workflow |
| P1 | WP-16 AI gateway/questionnaire | stable M-05/M-06 + AI RFC + golden set | M-14 scoped retrieval, citation/refusal, human approval, usage/cost control |
| P2 | WP-17 PCI SAQ assistant | M-05/M-06 + PCI source + human-review workflow | SAQ scope wizard, readiness checklist, approved export; no ASV scanning |
| P2 | WP-18 White-label agency | M-01/M-12/M-13 + delegated-access RFC | agency/sub-organization, delegated roles, branding, billing/residency boundary |
| P2 | WP-19 Auditor marketplace | WP-18 + marketplace contract/consent policy | profile, invite, scope, expiry/revoke, conflict and offboarding controls |
| P0 GA | WP-20 Hardening/GA | all live modules + S6 | Gate D, pentest, load/SLO, restore/incident/legal/on-call proof |

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

## 11. Definition of Ready dan Definition of Done

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

## 12. Larangan urutan build

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

## 13. Milestone release

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

## 14. Riwayat roadmap

| Versi | Perubahan |
|---|---|
| 1.0 | Roadmap awal dari foundation sampai GA, dengan dependency dan gate utama. |
| 2.0 | Menambahkan kontrak kelengkapan PRD §0–§25, phase owner/verification/evidence, cross-cutting control pack, ownership M-01–M-15, phase exit register, dan work package terpisah untuk ISO, privacy, AI, PCI, white-label, marketplace, serta GA. Menyamakan delapan AWS checks dengan PRD §7.4. |