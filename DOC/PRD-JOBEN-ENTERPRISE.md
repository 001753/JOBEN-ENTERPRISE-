# JOBEN ENTERPRISE
## Product Requirements & System Delivery Plan

**Produk:** JOBEN ENTERPRISE  
**Kategori:** Compliance Automation & Trust Management Platform  
**Domain target:** `jobenapp.cloud`  
**Dokumen kanonik:** `/DOC/PRD-JOBEN-ENTERPRISE.md`  
**Versi:** 5.3 — Global-Ready Core, Indonesia/Asia MVP Contract
**Status:** Baseline engineering; implementasi hanya boleh mengklaim capability yang telah melewati gate pada §24
**Bahasa dokumen:** Bahasa Indonesia; istilah standar teknis dan compliance mengikuti istilah resmi  
**Strategi go-to-market:** Core system global-ready by design; MVP pertama diluncurkan untuk Indonesia/Asia
**Sumber konsolidasi:** PRD master v3.0 dan Addendum Scan & Continuous Control Monitoring v1.0

**Status repository saat ini:** dokumen PRD ini sudah direvisi sebagai baseline
engineering. Repository belum berisi aplikasi live, connector production, data
customer, atau hasil screening nyata. Sampai implementasi dan gate yang disebutkan
di sini lulus, jangan menampilkan screenshot, score, finding, atau capability
seolah-olah sudah tersedia.

> Dokumen ini adalah satu-satunya acuan produk dan engineering. Jika detail provider
> berubah atau belum didefinisikan di sini, tim wajib memeriksa dokumentasi resmi
> provider pada saat implementasi dan mencatat keputusan tersebut dalam RFC atau
> decision log. Jangan mengarang endpoint, response shape, permission, atau klaim
> compliance dari ingatan.

---

## 0. Kritik terhadap baseline sebelumnya dan kontrak revisi

PRD v4.0 sudah memiliki prinsip keamanan dan anti-data-palsu yang baik, tetapi
belum cukup ketat untuk menjadi kontrak produksi. Berikut adalah blocker
engineering yang harus ditutup:

| Gap | Risiko nyata | Keputusan v5.0 |
|---|---|---|
| `pass/fail` belum selalu memiliki sumber, versi rule, waktu observasi, dan cakupan | Hasil tidak dapat direproduksi saat API provider berubah | Finding wajib memiliki provenance lengkap, `checkVersion`, `observedAt`, dan coverage |
| Error dan data lama dapat bercampur dalam agregasi | Skor terlihat baik ketika sebagian provider gagal | Freshness policy dan `dataQuality` wajib; stale/error tidak pernah menjadi pass |
| Canonicalization dan WORM belum dikontrak | Hash dapat berbeda antar runtime atau evidence dapat dihapus | Canonical JSON versioned, content-addressed object, retention lock, dan verify job |
| Satu `executionError` tidak cukup untuk partial result | Resource yang gagal dapat hilang tanpa terlihat | Outcome per resource, error class, coverage, dan completeness wajib |
| Permission provider masih berupa daftar umum | Connector over-privileged atau check tidak dapat diverifikasi | Permission matrix per check/endpoint, diuji pada account sandbox |
| AI hanya diberi aturan disclaimer | Model masih dapat mengarang fakta dan sitasi | Retrieval wajib citation-backed, output schema, refusal, dan evaluator anti-hallucination |
| Tenant isolation hanya dijelaskan sebagai tanggung jawab service | Satu query lupa filter dapat menjadi IDOR | Authorization policy terpusat, scoped repository, negative tests, dan RLS bila tersedia |
| Backup, restore, migration, incident response belum menjadi release gate | Sistem bisa jalan tetapi tidak dapat dipulihkan | RTO/RPO, restore drill, migration rollback, SLO, dan runbook wajib |
| MVP mencakup terlalu banyak permukaan sekaligus | Scope besar menghambat validasi kebenaran | Satu vertical slice AWS read-only harus benar sebelum provider kedua; global-readiness dibangun sebagai fondasi, bukan perluasan fitur MVP |

### 0.1 Aturan build yang mengikat

1. **Tidak ada klaim capability tanpa bukti.** Setiap capability diberi status
   `planned`, `not_implemented`, `experimental`, `demo`, `live_verified`, atau
   `deprecated`. Hanya `live_verified` yang boleh diklaim ke customer.
2. **Tidak ada status compliance dari frontend, seed demo, LLM, atau default
   value.** Sumber status hanya evaluator versioned yang menerima response
   provider dan menulis evidence immutable.
3. **Tidak ada kegagalan diam-diam.** Partial coverage, stale data, permission
   error, provider error, dan unsupported API tampil dengan alasan dan timestamp.
4. **Kontrak provider wajib dipelihara.** Setiap check memiliki endpoint, permission
   minimal, versi API atau revision dokumentasi, fixture, tanggal verifikasi,
   account sandbox, dan owner. Perubahan kontrak memasukkan check ke
   `verification_required`, bukan tetap dianggap pass.
5. **Perubahan destructive harus reversible.** Migration, reconnect, retention,
   billing state, dan publish policy memiliki state machine, idempotency key,
   audit log, dan runbook rollback.
6. **Demo adalah produk terpisah.** Namespace `/api/demo/*`, organisasi/database
   terpisah, watermark permanen `Sample Data — Not Live`, dan tidak ada code path
   yang dapat memakai demo evidence pada dashboard live.

### 0.2 Definition of truth

- **Observed fact:** nilai mentah/terstruktur yang dikembalikan provider pada waktu
  tertentu; bukan opini dan bukan ringkasan AI.
- **Evaluation:** eksekusi rule deterministik terhadap observed fact dengan
  `checkId` dan `checkVersion` tertentu.
- **Evidence:** payload ter-redact yang disimpan immutable, hash, dan metadata sumber.
- **Finding:** hasil evaluation untuk resource/scope tertentu yang terhubung ke evidence.
- **Control status:** agregasi finding terbaru yang relevan dengan coverage dan
  data quality; bukan field yang boleh diubah dari UI.
- **Readiness score:** metrik internal berbasis scope dan freshness; bukan opini
  auditor, sertifikasi, atau jaminan lulus.

---

## 1. Ringkasan Eksekutif

JOBEN ENTERPRISE adalah platform global-ready untuk membantu startup, scale-up, dan
enterprise mempercepat kesiapan SOC 2, ISO 27001, dan privacy readiness. MVP
pertama memvalidasi produk pada customer Indonesia/Asia melalui:

1. pengumpulan evidence dari sistem nyata;
2. pemantauan kontrol secara berkelanjutan;
3. Evidence Vault yang immutable dan dapat diaudit;
4. remediation guidance yang konkret;
5. Trust Page dan Auditor Portal;
6. Policy Center dan workflow review manusia;
7. AI assistant yang menjelaskan data organisasi, bukan menciptakan fakta compliance;
8. payment dan notifikasi yang sesuai kebutuhan pasar Indonesia/Asia, melalui
   abstraction yang dapat diperluas ke provider dan region global.

Nilai produk bukan dashboard yang terlihat meyakinkan, melainkan **kebenaran,
ketertelusuran, dan integritas data**. Setiap status kontrol harus dapat ditelusuri
ke hasil scan deterministik dan evidence API provider yang nyata. Sistem tidak boleh
menampilkan status `met`, skor, atau finding yang tidak memiliki sumber data yang
dapat diverifikasi.

### 1.1 Sasaran bisnis

- Mengurangi waktu pengumpulan evidence dan persiapan audit.
- Memberi visibilitas drift kontrol maksimal setiap enam jam dan alert kritis
  dengan target pengiriman di bawah 15 menit sejak deteksi.
- Menyediakan satu sumber kebenaran untuk status kontrol, evidence, remediation,
  policy, dan readiness.
- Menjadi produk B2B SaaS dengan katalog plan, currency, tax, payment provider, dan
  entitlement yang dapat dilokalkan; MVP memakai IDR dan provider Indonesia/Asia.
- Membangun core system yang tidak terkunci pada satu region, currency, bahasa,
  payment provider, notification provider, data-residency policy, atau framework.
- Membantu JOBEN sendiri melakukan dogfooding menuju SOC 2 Type I lalu Type II.

### 1.1.1 Strategi global-ready, regional-first

Global-ready berarti capability inti dapat dipakai lintas negara tanpa migrasi data
atau rewrite domain logic ketika region baru dibuka. Regional-first berarti
availability provider, harga, bahasa, legal terms, regulatory mapping, support, dan
data residency untuk MVP sengaja dibatasi pada Indonesia/Asia.

Kontrak yang wajib berlaku sejak MVP:

- seluruh organisasi memiliki `homeRegion`, `dataResidencyPolicy`, `defaultLocale`,
  `timezone`, `currency`, dan `taxProfile` yang tersimpan di server;
- waktu disimpan dalam UTC dan ditampilkan menurut timezone organisasi/user;
- amount disimpan sebagai minor unit bersama ISO 4217 currency code; harga tidak
  boleh berupa angka IDR yang hardcode di frontend;
- provider, notification channel, AI model, object-storage region, dan framework
  dipilih melalui registry/configuration versioned, bukan percabangan negara di UI;
- data customer, evidence, export, log, dan AI retrieval tetap tenant-scoped serta
  mengikuti policy region; perpindahan region adalah workflow terkontrol dengan
  audit, consent/approval, dan rollback yang relevan;
- customer-facing copy menjelaskan region dan capability yang benar-benar tersedia;
  provider atau framework yang belum diverifikasi tetap `planned` atau
  `verification_required`.

### 1.2 Batasan klaim

JOBEN mengotomasi evidence collection, control monitoring, dan readiness workflow.
JOBEN **bukan** auditor, firma hukum, akuntan publik, penerbit sertifikasi, atau
pemberi jaminan lulus audit.

Disclaimer wajib untuk halaman, laporan, dan respons AI yang menyebut framework:

> “JOBEN Enterprise helps automate evidence collection and control monitoring.
> Certification/attestation is issued by independent accredited third parties.”

Dilarang menggunakan atau menerjemahkan secara ekuivalen klaim seperti:
“guarantee pass”, “automatically certified”, “instant SOC 2 report”, atau
“dijamin lulus audit”.

---

## 2. Prinsip Produk yang Tidak Dapat Ditawar

### P-01 — Ground truth di atas tampilan

Status berasal dari API provider yang dipanggil pada waktu tertentu dan disimpan
sebagai evidence. AI hanya boleh merangkum atau memberi konteks setelah fakta
tersimpan.

### P-02 — Tidak ada data palsu terselubung

- Mode live adalah default.
- Tanpa integrasi atau tanpa scan sukses, status adalah `not_started` atau
  `needs_review`, bukan `met`.
- Data sample hanya boleh digunakan dalam mode demo terpisah dan wajib diberi badge
  permanen `Sample Data — Not Live`.
- Endpoint demo menggunakan namespace `/api/demo/*`, terpisah dari data customer.

### P-03 — Keputusan pass/fail deterministik

`CheckDefinition.run()` tidak boleh memanggil LLM untuk menentukan hasil.
Perbandingan threshold dan kondisi dilakukan oleh kode yang versioned, dapat
direproduksi, dan dapat diuji.

### P-04 — Evidence auditor-grade

Evidence disimpan append-only di object storage eksternal, memiliki SHA-256,
timestamp, referensi sumber, dan metadata scan. Isi secret harus di-redact sebelum
disimpan.

### P-05 — Kegagalan harus terlihat

Permission kurang, provider down, rate limit, timeout, dan hasil yang belum dapat
diverifikasi ditampilkan sebagai error/unknown dengan alasan spesifik. Sistem tidak
boleh menyamarkan kegagalan menjadi pass atau memakai hasil lama tanpa label.

### P-06 — Isolasi tenant di server

Semua service dan query data organisasi wajib memvalidasi `organizationId` di
server/service layer. Filter di UI saja tidak cukup. Akses lintas organisasi harus
ditolak dan diuji.

### P-07 — Human-in-the-loop untuk output normatif

Draf AI untuk policy, questionnaire, regulasi, atau legal-adjacent content tidak
berlaku resmi sebelum review dan approval pengguna berwenang.

### P-08 — Least privilege

Setiap connector meminta permission minimum yang benar-benar dipakai oleh check
yang sudah diimplementasikan. Credential jangka panjang tidak disimpan mentah di
database.

---

## 3. Target Pengguna, Peran, dan Jobs-to-be-Done

| Persona | Kebutuhan utama | Hasil yang diharapkan |
|---|---|---|
| Founder/Owner startup | Mendapat readiness tanpa membangun GRC team besar | Status kontrol dan prioritas remediation yang bisa ditindaklanjuti |
| Security/Compliance Lead | Memantau drift dan menyiapkan audit | Evidence terverifikasi, histori, report, dan audit trail |
| Engineer/IT Admin | Menghubungkan cloud/SaaS secara aman | Setup connector read-only yang transparan dan mudah diverifikasi |
| Auditor eksternal | Meninjau evidence tanpa akses admin | Auditor Portal read-only dengan scope dan masa berlaku terbatas |
| GRC staff JOBEN | Memvalidasi perubahan regulasi dan mapping | Review queue, AI draft sebagai starting point, approval tercatat |
| Sales/prospek | Memahami nilai produk dan menemukan fit | Marketing site, demo sample yang jelas, chatbot lead qualification |
| Internal admin | Mengelola pricing, AI cost, dan operasional | Admin console terpisah dengan audit log |

### 3.1 Role access baseline

| Fitur | OWNER | ADMIN | MEMBER | AUDITOR_READONLY | Internal GRC |
|---|---:|---:|---:|---:|---:|
| Mengelola organisasi dan billing | Ya | Terbatas | Tidak | Tidak | Tidak |
| Connect/reconnect integration | Ya | Ya | Tidak | Tidak | Tidak |
| Menjalankan Scan Now | Ya | Ya | Ya | Tidak | Tidak |
| Melihat evidence | Ya | Ya | Sesuai scope | Scope read-only | Sesuai tugas |
| Approve policy/draft | Ya | Ya | Tidak | Tidak | Sesuai workflow |
| Invite anggota | Ya | Ya | Tidak | Tidak | Tidak |
| Review regulasi global | Tidak | Tidak | Tidak | Tidak | Ya |

MFA wajib untuk akun internal dan role `OWNER`/`ADMIN` melalui Clerk.

---

## 4. Ruang Lingkup dan Prioritas

### 4.1 In scope untuk MVP bertahap (Fase 0–1B)

- Fondasi Next.js App Router + TypeScript + PostgreSQL + Prisma.
- Auth Clerk dan multi-tenant organization.
- Global-ready tenant, locale, timezone, currency, tax, region, provider registry,
  dan feature/capability registry; MVP aktif pertama: market pack Indonesia/Asia
  yang provider serta legal review-nya tersedia.
- Framework SOC 2 Type II dengan control seed yang hanya dipetakan ke check
  berstatus `live_verified`.
- Fase 1: satu vertical slice AWS cross-account IAM role, scan engine, evidence
  vault, control aggregator, dashboard, control detail, dan remediation.
- Fase 1B: provider GitHub App, PDF, notification, billing sandbox, dan AI
  gateway hanya setelah safety/contract gate lulus.
- Google Workspace, public trust page, auditor portal, dan fitur regulasi tidak
  dianggap MVP live sebelum gate provider dan privacy masing-masing lulus.
- i18n EN/ID/ZH dan cron terproteksi dikirim bertahap; MVP hanya mempublikasikan
  locale yang telah lengkap untuk market pack aktif. String yang belum tersedia
  tidak boleh fallback diam-diam pada customer-facing release.

### 4.2 Out of scope MVP, tetapi disiapkan sebagai roadmap

- Azure, GCP, Okta, Vercel, Supabase, Firebase connector.
- ISO 27001 penuh, GDPR/CCPA data mapping, DSAR, DPA generator.
- Questionnaire AI penuh untuk PDF/Excel/CSV.
- PCI SAQ assistant.
- White-label agency dan marketplace auditor.
- Checkout global dan multi-currency live; currency abstraction dan catalog schema
  tetap wajib tersedia sejak MVP.
- Global data residency, multi-region active/active, dan seluruh payment provider
  internasional.
- ASV scanning PCI-DSS.
- Migrasi hosting ke VPS/cloud sebelum threshold traffic disepakati.

### 4.3 Prioritas delivery

Urutan prioritas ditentukan oleh risiko produk:

1. Kebenaran dan isolasi data scan.
2. Credential safety dan least privilege.
3. Scan engine dan evidence integrity.
4. Aggregation/scoring yang dapat diaudit.
5. UX empty/error state dan remediation.
6. Report dan auditor workflow.
7. Billing, AI, growth, dan fitur pendukung.

### 4.4 Kontrak “real work, completed, full feature”

Istilah `full feature` dalam PRD ini tidak berarti semua modul harus live pada
hari pertama. Artinya, setiap modul yang diaktifkan pada suatu release harus
memiliki lifecycle utuh dan dapat dipakai sampai hasil akhirnya tanpa
intervensi database manual, data contoh, atau status optimistis dari frontend.
Modul yang belum memenuhi kontrak wajib tetap `not_implemented`,
`verification_required`, atau `degraded` dan tidak boleh ditampilkan sebagai
fitur selesai.

Sebuah modul dinyatakan `completed` hanya jika:

1. boundary, owner, actor/role, tenant scope, dan source of truth sudah tertulis;
2. seluruh state transition validasi server-side dan memiliki state terminal yang
   jelas, termasuk retry, cancel, archive, dan recovery bila relevan;
3. alur utama dari create/configure sampai inspect/export/close dapat dijalankan
   melalui UI dan API tanpa bypass administratif tersembunyi;
4. input, output, error taxonomy, partial result, freshness, idempotency,
   permission, audit event, retention, dan rollback telah dikontrak;
5. tidak ada tombol, endpoint, job, atau provider adapter yang menghasilkan
   placeholder, silent fallback, default pass, atau data sintetis pada jalur live;
6. test happy path, empty state, validation, unauthorized, cross-tenant,
   duplicate/replay, timeout/provider failure, partial result, schema drift, dan
   recovery tersedia dan lulus;
7. operational runbook, metric, alert, cost ceiling, dan customer-facing
   limitation tersedia; dan
8. `CapabilityRecord` memiliki proof record, tanggal verifikasi, expiry, serta
   reviewer yang berbeda dari author untuk capability berisiko tinggi.

Kriteria ini berlaku untuk scan, evidence, control, policy, risk, vendor,
regulation, report, trust/auditor, notification, billing, AI, auth, dan
operations. CRUD generik bukan definisi kelengkapan; state machine dan hasil
bisnis yang dapat diverifikasi adalah definisinya.

### 4.5 Peta readiness modul utama

Tabel ini adalah baseline scope yang mengikat. `Full feature` berarti seluruh
kapabilitas yang tercantum pada kontrak modul, bukan sekadar halaman, tabel, atau
endpoint CRUD. Modul hanya boleh dipublikasikan bila status capability seluruh
kapabilitas wajibnya `live_verified` dan seluruh dependency pada kolom terakhir
juga `live_verified`.

| ID | Modul | Hasil bisnis yang wajib tersedia | Status awal | Dependency minimum |
|---|---|---|---|---|
| M-01 | Identity, Organization & RBAC | sign-up/sign-in, MFA, organisasi, invite, role, session, revoke, tenant isolation, audit | `planned` | Auth provider, DB, audit |
| M-02 | Integration & Credential | connect, verify, reconnect, revoke, least privilege, health, permission matrix, error recovery | `planned` | M-01, secret manager, provider contract |
| M-03 | Scan & Job Orchestrator | scheduled/on-demand scan, queue, lock, retry, cancel, progress, partial result, recovery | `planned` | M-01, M-02, queue |
| M-04 | Provider Connector | adapter versioned, API client, permission test, schema validation, deterministic checks, drift handling | `planned` | M-02, M-03, provider sandbox |
| M-05 | Evidence Vault & Integrity | redact, canonicalize, hash, immutable storage, retention, verify, authorized access, legal hold | `planned` | M-01, M-03, object storage |
| M-06 | Finding, Control & Scoring | evaluator, mapping, projection rebuild, freshness/coverage, score versioning, explainability | `planned` | M-03, M-04, M-05 |
| M-07 | Remediation & Human Review | guidance, owner, due date, status workflow, comment, evidence of fix, re-scan, audit | `planned` | M-06, M-01 |
| M-08 | Dashboard & Customer App | onboarding, integrations, score, control detail, evidence, empty/error/degraded state, i18n | `planned` | M-01, M-02, M-06, M-07 |
| M-09 | Report & Export | queued PDF, provenance appendix, signed access, expiry, revocation, reproducible snapshot | `planned` | M-05, M-06, M-07 |
| M-10 | Notification & Alert | rule, preference, channel, delivery, retry, dedupe, escalation, latency evidence | `planned` | M-03, M-06, notification provider |
| M-11 | Policy, Risk, Vendor & Regulation | versioned records, review/approval, mapping, reminders, impact propagation, audit | `planned` | M-01, M-06, M-10, official sources |
| M-12 | Trust Page & Auditor Portal | publish/unpublish, scope, expiry, read-only access, revocation, evidence citation, audit | `planned` | M-05, M-06, M-09 |
| M-13 | Billing & Entitlement | plan catalog, checkout, webhook inbox, subscription state, entitlement enforcement, reconciliation | `planned` | M-01, payment provider, audit |
| M-14 | AI Gateway & Assistants | scoped retrieval, citations, refusal, schema validation, human approval, usage/cost control | `planned` | M-01, M-05, M-06, AI provider |
| M-15 | Admin, Observability & Operations | capability registry, incidents, queues, SLO, cost, backup/restore, runbooks, release control | `planned` | all production dependencies |

Status `planned` pada tabel ini adalah status repository saat PRD ditulis, bukan
klaim bahwa modul sudah tersedia. Setiap implementasi wajib memecah modul menjadi
`CapabilityRecord` yang dapat dilacak ke requirement, commit, test evidence,
operational evidence, reviewer, dan expiry. Mengubah status modul menjadi
`completed` tanpa dossier completion tersebut adalah pelanggaran PRD.

### 4.6 Kontrak kelengkapan setiap modul

Setiap modul utama wajib memiliki dokumen kontrak tersendiri sebelum coding dan
dossier completion sebelum release. Kontrak tidak boleh hanya menjelaskan UI.
Minimum isi dan perilaku berikut berlaku untuk seluruh M-01 sampai M-15:

1. **Boundary dan ownership:** tujuan, non-goals, owner, aktor, role matrix,
   tenant scope, data classification, source of truth, dependency, dan public
   limitation.
2. **Lifecycle utuh:** command/query, state machine dengan transition guard
   server-side, state terminal, retry, idempotency, cancellation, timeout,
   archive/deletion, recovery, dan rollback yang relevan.
3. **Kontrak teknis:** request/response schema versioned, error envelope dan
   taxonomy, pagination, concurrency/lock, event/audit schema, retention,
   freshness, coverage, SLO, rate/cost limit, dan observability.
4. **User journey lengkap:** kondisi awal → konfigurasi → validasi → proses →
   hasil → inspect/export/close; tersedia melalui UI dan API untuk role yang
   diizinkan, tanpa SQL manual, feature flag tersembunyi, atau support override.
5. **Kebenaran hasil:** hasil memiliki provenance, timestamp, version, actor/
   provider, scope, dan bukti yang dapat diverifikasi. `loading`, `queued`,
   `unknown`, `error`, `stale`, `partial`, `not_available`, dan `degraded`
   tidak boleh dirender sebagai sukses.
6. **Resilience dan security:** unauthorized, cross-tenant, replay, duplicate,
   timeout, rate limit, provider outage, schema drift, secret leakage, privilege
   escalation, dan restore failure diuji sebagai failure yang terlihat.
7. **Completion evidence:** unit, integration, contract, E2E, security,
   accessibility, localization, performance, manual sandbox, runbook drill,
   metric/alert, screenshot atau export bila relevan, serta sign-off reviewer.

### 4.7 Kontrak khusus modul utama

#### M-01 — Identity, Organization & RBAC

- **Alur wajib:** create account → verify identity/MFA → create/join organization
  → invite → accept/revoke member → switch organization → sign out/revoke
  session. Organization context selalu berasal dari session yang tervalidasi.
- **State wajib:** `invited`, `active`, `suspended`, `revoked`, `deleted` untuk
  membership; perubahan role dan transfer ownership harus memiliki approval dan
  audit trail.
- **Acceptance:** semua route, job, export, webhook, signed URL, dan repository
  lulus negative test lintas tenant; MFA `OWNER`/`ADMIN` dan internal aktif;
  kehilangan session atau membership langsung menghentikan akses.

#### M-02/M-04 — Integration dan Provider Connector

- **Alur wajib:** pilih provider → tampilkan permission dan limitation → connect →
  verify identity/scope/permission → health check → scan → reconnect/revoke.
  Credential asli tidak pernah masuk database, log, fixture, evidence, atau UI.
- **Setiap adapter wajib memiliki:** provider API revision, endpoint matrix,
  permission minimal per check, rate-limit strategy, timeout, pagination,
  schema fixture, redaction rule, check registry, sandbox account, dan expiry
  verifikasi.
- **Acceptance:** revoked/insufficient permission menjadi `permission_error`;
  401/403 tidak di-retry; transient error punya retry terbatas; schema drift
  memblokir evaluasi terkait dan menjadi `verification_required`; reconnect
  tidak menghapus histori.

#### M-03 — Scan & Job Orchestrator

- **Alur wajib:** schedule/Scan Now → idempotency check → queue → progress per
  integration/check → evaluate → persist append-only result → aggregate →
  notify → complete/partial/error.
- **State wajib:** `queued`, `running`, `cancelling`, `completed`, `partial`,
  `failed`, `cancelled`, `dead_letter`; hanya worker yang berwenang dapat
  melakukan transition.
- **Acceptance:** duplicate invocation menghasilkan satu run; cancel tidak
  menghapus evidence; satu check/provider gagal tidak menghilangkan resource
  lain; stuck lease dapat direcover; retry tidak menggandakan finding atau
  notification; backlog, duration, dan failure reason terlihat operator.

#### M-05 — Evidence Vault & Integrity

- **Alur wajib:** collect → redact → schema validate → canonicalize → hash →
  write immutable object → link observed fact/finding → authorized view/download
  → periodic verify → quarantine/recapture jika integrity gagal.
- **Acceptance:** hash yang berubah terdeteksi; object hilang menjadi incident;
  record final tidak dapat di-update/delete; retention/legal hold ditegakkan;
  signed URL pendek, scoped, dapat dicabut, dan setiap akses dicatat; secret
  canary serta nested payload lulus redaction test.

#### M-06 — Finding, Control & Scoring

- **Alur wajib:** check version → observed fact → deterministic evaluation →
  finding per resource → mapping → control projection → score snapshot →
  explainable dashboard/report.
- **Acceptance:** projection dapat direbuild dari append-only finding dan
  menghasilkan hasil yang sama; error/stale/incomplete/missing evidence tidak
  pernah menjadi pass; formula/version/denominator tersimpan; `not_applicable`
  membutuhkan alasan dan permission; perubahan hasil dapat ditelusuri ke scan,
  evidence, rule, dan waktu observasi.

#### M-07 — Remediation & Human Review

- **Alur wajib:** finding fail → guidance versioned → assign owner/due date →
  acknowledge/in progress/blocked → attach fix evidence/comment → re-scan →
  verify resolved atau reopen → close/archive.
- **Acceptance:** hanya evidence dari scan terbaru yang dapat menutup finding;
  owner yang tidak berwenang tidak dapat mengubah status; due date dan SLA
  memicu reminder; AI note diberi label dan tidak menggantikan langkah manusia;
  close/reopen memiliki actor, alasan, dan audit.

#### M-08/M-09 — Dashboard, Customer App, Report & Export

- **Alur wajib:** onboarding → lihat source/freshness/coverage → drill-down ke
  finding/evidence/remediation → export snapshot yang sama dengan layar.
- **Acceptance:** tanpa data tampilkan empty state, bukan score nol; queued,
  error, partial, stale, dan degraded memiliki copy/CTA berbeda; semua count
  berasal dari query tenant-scoped; PDF memiliki hash/timestamp/disclaimer,
  hasil deterministik untuk input snapshot yang sama, dan akses kedaluwarsa.

#### M-10 — Notification & Alert

- **Alur wajib:** event eligible → preference/policy evaluation → dedupe key →
  enqueue → provider delivery → retry/escalate → delivered/failed/acknowledged.
- **Acceptance:** critical drift terukur <15 menit; duplikat webhook/job tidak
  mengirim dua alert; opt-out dan quiet hour dihormati kecuali policy emergency;
  delivery status dan provider response ter-redact; retry dan dead-letter dapat
  dioperasikan tanpa mengubah finding.

#### M-11/M-12 — Policy, Risk, Vendor, Regulation, Trust & Auditor

- **Alur wajib:** draft → review → approve/publish → version → periodic review →
  supersede/withdraw; trust/auditor publish hanya mengambil capability dan
  evidence yang `live_verified`, scoped, belum expired, dan diizinkan.
- **Acceptance:** AI tidak dapat publish; perubahan regulasi resmi memiliki
  source/hash/reviewer dan dampak mapping; auditor access read-only, scope- dan
  time-bound, dapat dicabut; unpublish tidak menghapus audit/history; public
  output tidak pernah menampilkan demo atau stale evidence sebagai current.

#### M-13 — Billing & Entitlement

- **Alur wajib:** catalog → checkout → signature-verified webhook inbox →
  subscription state → entitlement check → renewal/failure/cancel/refund →
  reconciliation.
- **Acceptance:** payment provider tidak pernah mengubah finding; duplicate/
  out-of-order/replayed event aman; entitlement ditolak server-side dan
  transparan; raw card data tidak disimpan; setiap transaksi dapat direkonsiliasi
  dengan provider dan audit log.

#### M-14 — AI Gateway & Assistants

- **Alur wajib:** classify request → scoped retrieval → redact → model call →
  schema validate/citation verify → answer/refusal → usage log → human approval
  bila output normatif.
- **Acceptance:** cross-tenant retrieval, prompt injection, unsupported/legal
  claim, stale/integrity-failed evidence, dan insufficient evidence diuji;
  citation harus dapat ditemukan; refusal tidak boleh menjadi jawaban optimistis;
  fallback mempertahankan safety contract; AI tidak pernah menulis compliance
  status, evidence, subscription, atau published policy.

#### M-15 — Admin, Observability & Operations

- **Alur wajib:** capability registry → release diff → deploy gate → observe SLO/
  cost → incident → contain → recover → post-incident review.
- **Acceptance:** operator dapat pause/resume queue, revoke integration, quarantine
  evidence, replay notification, disable AI, restore backup, dan memverifikasi
  checksum/migration/tenant/evidence reference tanpa mengubah histori; critical
  alert, audit, backup/restore, dan runbook drill memiliki bukti timestamp.

### 4.8 Kontrak delivery core system

Bagian ini menerjemahkan “real work, completed, full feature” menjadi kontrak
delivery yang dapat diperiksa. Modul tidak dianggap selesai karena halaman UI,
route handler, tabel database, atau happy path telah dibuat. Modul selesai hanya
ketika alur bisnisnya dapat dimulai dari kondisi nyata, menghasilkan output yang
bersumber, menangani seluruh kegagalan yang diketahui, dan dapat dipulihkan oleh
role yang berwenang tanpa SQL manual atau perubahan data langsung.

#### 4.8.1 Definisi completion yang mengikat

Setiap modul M-01 sampai M-15 wajib memiliki satu atau lebih
`CapabilityRecord` dengan status terpisah. Capability yang belum siap tidak boleh
menahan modul lain secara diam-diam dan tidak boleh ikut menghasilkan status
customer-facing. Sebuah modul hanya boleh diberi label internal `completed` bila
semua capability wajibnya memenuhi seluruh kondisi berikut:

| Gate | Syarat lulus |
|---|---|
| Product outcome | Hasil bisnis, non-goals, aktor, role, tenant scope, dan limitation disetujui Product/Security |
| Source of truth | Satu owner data ditetapkan; tidak ada modul lain yang menulis tabel sumber secara langsung |
| Lifecycle | State, transition guard, terminal state, retry, timeout, cancel, archive/delete, recovery, dan rollback diuji |
| Interface | Command/query/event memiliki schema version, authorization, idempotency, pagination/limit, dan error envelope |
| Truth & provenance | Output memiliki actor/provider, scope, version, timestamp, correlation ID, dan referensi bukti |
| Failure visibility | Empty, stale, partial, timeout, permission, provider, schema, conflict, dan system failure terlihat dengan CTA pemulihan |
| Security | Tenant isolation, least privilege, replay, privilege escalation, secret leakage, signed access, dan audit diuji |
| Resilience | Queue/lock/retry/circuit breaker, backup/restore atau recovery yang relevan, serta failure drill lulus |
| User journey | UI dan API menyelesaikan alur dari create/configure sampai inspect/export/close untuk semua role yang diizinkan |
| Operations | SLO, metric, alert, cost ceiling, runbook, on-call owner, dan incident evidence tersedia |
| Proof | CI evidence, sandbox/manual evidence, capability diff, reviewer independen, expiry/reverification, dan customer limitation tersedia |

`completed` bukan status yang dapat diatur frontend. Untuk publikasi, status
operasionalnya tetap `live_verified` pada setiap capability. Jika salah satu gate
gagal, status harus `not_implemented`, `verification_required`, atau `degraded`
sesuai §6.4; hasil lama tidak boleh dipromosikan menjadi selesai.

#### 4.8.2 Matriks kontrak per modul utama

| Modul | Scope full-feature yang wajib selesai | Source of truth dan terminal sukses | Kegagalan yang wajib terlihat |
|---|---|---|---|
| M-01 Identity, Organization & RBAC | Auth event sync, membership multi-organisasi, invite, role, ownership transfer, MFA enforcement, session revoke, org switch, audit | Identity provider untuk autentikasi; `OrganizationMembership` untuk authorization; membership `active`, session revoked, audit tersimpan | invite expired/replayed, suspended/revoked member, MFA missing, session revoked, cross-tenant, ownership conflict |
| M-02 Integration & Credential | Connect, verify, permission preview, health, reconnect, revoke, credential reference rotation, provider-specific limitation | `Integration` dan encrypted credential reference; `verified` hanya setelah verification contract lulus | invalid credential, 401/403, insufficient permission, expired token, revoked connection, rate limit, verification timeout |
| M-03 Scan & Job Orchestrator | Schedule, Scan Now, queue, lease, progress, per-check outcome, cancel, retry, dead-letter, circuit breaker, recovery | `ScanRun`/job ledger append-only; run terminal `completed`, `partial`, `failed`, atau `cancelled` | duplicate/replay, stuck lease, backlog, timeout, partial coverage, dead-letter, worker loss, provider outage |
| M-04 Provider Connector | Versioned adapter, endpoint/permission matrix, pagination, schema validation, redaction handoff, deterministic check registry, sandbox verification | Provider response setelah redaction + `ProviderContract`/`CheckDefinition` version | schema drift, unsupported endpoint, pagination gap, provider error, permission error, deprecated API, unverified check |
| M-05 Evidence Vault & Integrity | Immutable write, canonicalization, hash, WORM/retention, legal hold, scoped read/download, verify, quarantine, re-collection | Object storage content-addressed evidence + DB reference/hash; evidence final immutable | redaction/schema failure, hash mismatch, missing object, expired URL, legal hold conflict, unauthorized access |
| M-06 Finding, Control & Scoring | Deterministic evaluation, finding per resource, mapping, projection rebuild, freshness/coverage, versioned score, explainable drill-down | Append-only `Finding`/`ObservedFact`; `OrgControlStatus` dan score adalah rebuildable projection | missing evidence, stale/partial/error, denominator change, rule version mismatch, invalid N/A, rebuild mismatch |
| M-07 Remediation & Human Review | Guidance version, assignment, SLA, comments, blocked state, fix evidence, re-scan verification, reopen, close/archive, approval | Remediation workflow + latest verified finding/evidence; close hanya dari verification | unauthorized owner, overdue, stale fix evidence, close without re-scan, reopen conflict, rejected approval |
| M-08 Dashboard & Customer App | Onboarding, integrations, score, control/finding/evidence/remediation drill-down, empty/error/degraded states, role-aware i18n | Read models tenant-scoped dari M-01–M-07; tidak ada local compliance truth | no data, loading/queued, stale, partial, provider error, missing capability, locale missing, forbidden scope |
| M-09 Report & Export | Snapshot request, queued generation, deterministic PDF/CSV, provenance appendix, signed access, expiry/revoke, export audit | Versioned `ReportSnapshot` dari snapshot data yang sama dengan UI | generation failure, snapshot mismatch, expired/revoked access, missing citation, reader incompatibility |
| M-10 Notification & Alert | Rule/policy, preferences, quiet hours, channel adapter, dedupe, retry, escalation, acknowledgement, delivery evidence | Notification ledger dan provider delivery receipt; finding tidak diubah oleh delivery | duplicate event, opt-out conflict, provider outage, rate limit, dead-letter, latency breach, redaction failure |
| M-11 Policy, Risk, Vendor & Regulation | Versioned records, review/approval, mapping, reminders, risk treatment, vendor lifecycle, official-source monitor, impact propagation | Versioned policy/risk/vendor/regulation records; approval/publish ledger | unreviewed draft, source unavailable, hash change, stale review, mapping conflict, unauthorized publish |
| M-12 Trust Page & Auditor Portal | Scoped publication, capability/evidence eligibility, time-bound read-only access, revoke/unpublish, citations, audit | `TrustPublication`/`AuditorAccess` policy dan eligible live evidence | expired/stale evidence, unverified capability, scope escalation, revoked link, demo leakage, unpublish history loss |
| M-13 Billing & Entitlement | Catalog version, checkout, signature-verified inbox, subscription state machine, entitlement enforcement, renewal/failure/cancel/refund, reconciliation | Provider webhook inbox + subscription ledger; entitlement projection server-side | replay/out-of-order webhook, signature failure, payment mismatch, grace-period ambiguity, unauthorized feature access |
| M-14 AI Gateway & Assistants | Scoped retrieval, redaction, model routing, schema/citation verification, refusal, human approval, usage/cost budget, deletion | Retrieval/evidence source dan `AiUsageLog`; AI tidak memiliki compliance write authority | prompt injection, insufficient evidence, invalid citation, cross-tenant retrieval, unsafe fallback, budget exceeded, provider outage |
| M-15 Admin, Observability & Operations | Capability registry, release diff, queue controls, incidents, SLO/cost, backup/restore, checksum, migration, runbook, emergency disable | Operational ledger, telemetry, backup manifest, incident record; tidak mengubah business history | alert loss, restore mismatch, checksum failure, migration failure, runaway cost, missing audit, unsafe operator action |

M-02 dan M-04, serta pasangan lain yang ditulis bersama di tabel readiness,
tetap wajib memiliki dossier dan `CapabilityRecord` terpisah. Pengelompokan hanya
untuk urutan pembahasan, bukan alasan untuk melewati gate salah satu modul.

#### 4.8.3 Dossier completion wajib

Sebelum modul atau capability berubah ke `live_verified`, repository harus
memiliki `DOC/completion/<moduleId>/` dengan isi minimal:

```text
README.md                  # scope, non-goals, owner, limitation, dependency
contract.md                # command/query/event/schema/error/state machine
permission-matrix.md       # role, endpoint/check, minimum permission, deny case
test-matrix.md             # expected result dan evidence untuk seluruh failure class
proof-record.md            # commit, CI runs, sandbox, reviewer, verifiedAt, expiresAt
runbook.md                 # operate, recover, rollback, customer communication
```

`proof-record.md` harus menghubungkan setiap acceptance criterion ke bukti yang
dapat dibuka ulang: test ID, log ter-redact, fixture/sandbox reference, screenshot
atau export bila relevan, dan reviewer. Bukti yang hanya berupa assertion developer,
data seed, screenshot dari sample data, atau klik manual tanpa expected result
tidak cukup. Dossier dapat ditunda selama fase perencanaan, tetapi wajib tersedia
sebelum coding capability dimulai pada Definition of Ready dan lengkap sebelum
release gate.

### 4.9 Urutan pembangunan dan dependency gate

Urutan berikut adalah dependency DAG, bukan daftar fitur yang boleh dilewati.
Setiap slice harus selesai end-to-end sebelum slice berikutnya menerima status
`live_verified`.

| Slice | Modul | Exit gate yang harus dibuktikan |
|---|---|---|
| S0 Foundation truth | M-01 + capability foundation M-15 | Multi-org membership, authz server-side, audit, migration, secret redaction, backup/restore smoke, CI, dan hosting verification |
| S1 Evidence vertical slice | M-02 + M-03 + M-04 + M-05 | AWS cross-account verify, minimal satu check nyata, queue/lock/retry, immutable evidence, hash re-verify, permission denial, schema drift |
| S2 Decision system | M-06 + M-07 | Rebuild projection identik, freshness/coverage, no-error-to-pass, remediation re-scan, close/reopen, tenant/security regression |
| S3 Customer operating surface | M-08 + M-09 + M-10 | UI/API parity, empty/error/degraded copy, deterministic report, signed access, dedupe/delivery latency, audit |
| S4 Governance surface | M-11 + M-12 | Human approval, official-source provenance, scoped/time-bound auditor access, stale/demo exclusion, publish/revoke |
| S5 Commercial and assistant surface | M-13 + M-14 | Webhook inbox/reconciliation, entitlement deny test, citation/refusal golden set, cost ceiling, provider fallback safety |
| S6 Operational readiness | M-15 full scope | Incident exercise, restore drill, queue controls, SLO telemetry, capability diff, on-call/runbooks, GA sign-off |

Dependency yang belum `live_verified` membuat capability dependent tetap
`verification_required`, bukan `experimental` yang diam-diam dipakai di
production. Sebuah slice tidak boleh dirilis hanya karena jumlah endpoint atau
persentase test coverage tercapai; seluruh exit gate pada tabel harus memiliki
proof record.

### 4.10 Definition of Ready per modul dan Definition of Done per release

Sebelum modul mulai dibangun, Product, Engineering, Security, dan Operations
harus menandatangani satu Module Contract yang menjawab:

1. Apa hasil bisnis yang dapat diverifikasi dan apa yang sengaja tidak dilakukan?
2. Siapa owner source of truth, actor, role, tenant scope, dan data classification?
3. Apa state machine lengkap serta transition guard untuk command UI/API/job?
4. Apa schema request/response/event, idempotency key, concurrency rule, dan
   error code yang stabil?
5. Bagaimana freshness, coverage, provenance, retention, audit, rollback, dan
   deletion/legal hold bekerja?
6. Bagaimana provider outage, partial result, schema drift, replay, restore
   failure, dan cross-tenant attempt diuji?
7. Apa SLO, cost ceiling, alert, runbook, customer limitation, dan expiry proof?

Release hanya boleh menyatakan core system `completed` jika seluruh slice yang
termasuk release mempunyai dossier lengkap, semua dependency berada pada status
yang diizinkan, seluruh capability customer-facing `live_verified`, dan tidak ada
critical/high finding terbuka yang memengaruhi kebenaran data, tenant isolation,
credential safety, evidence integrity, payment reconciliation, atau AI safety.

---

## 5. Keputusan Arsitektur

### 5.1 Arsitektur target

Satu Next.js App Router untuk marketing site, customer app, dan Route Handlers API.
Logic domain dipisahkan dalam service layer agar dapat diekstrak menjadi service
terpisah setelah migrasi hosting. Domain core tidak boleh mengetahui detail negara
atau vendor secara langsung; detail tersebut masuk melalui registry dan adapter
versioned.

```text
jobenapp.cloud
  Marketing pages, public Trust Page, public chatbot

app.jobenapp.cloud
  Next.js UI + Route Handlers + domain services
  Auth, billing, integrations, scan, evidence, controls,
  regulations, AI gateway, notifications, i18n

Global-ready control plane
  Organization/region directory, capability registry, plan/catalog,
  provider registry, policy/version registry, routing and audit

Regional execution boundary
  DB/object-storage/queue/provider adapters selected by homeRegion and policy
  MVP: Indonesia/Asia deployment boundary; future EU/US/APAC boundaries

External services
  PostgreSQL        persistent application data, region-aware tenant metadata
  Redis managed     BullMQ on-demand queue, region-scoped job routing
  S3/R2             immutable evidence and report storage, region policy enforced
  cPanel Cron       scheduled job trigger
  Clerk             auth, MFA, SSO/SAML capability
  Payment adapters  Xendit/Midtrans for MVP; global providers later
  Sentry            error tracking
```

### 5.2 Stack final

| Layer | Keputusan |
|---|---|
| Web/API | Next.js 14+ App Router, TypeScript |
| UI | TailwindCSS, shadcn/ui, design tokens JOBEN |
| Runtime | Node.js 22 LTS; fallback Node.js 20 LTS |
| Database | PostgreSQL versi hosting, Prisma |
| Queue | Redis managed eksternal + BullMQ untuk on-demand jobs |
| Scheduler | Native cPanel Cron memanggil endpoint internal |
| Realtime | SSE atau polling 5–15 detik; WebSocket bukan default |
| Object storage | AWS S3 atau Cloudflare R2, encrypted at rest |
| Auth | Clerk |
| Payment | Provider adapter contract; Xendit primary, Midtrans secondary, Duitku/iPaymu non-recurring for MVP; global providers added by region |
| AI | Gemini, Groq, DeepSeek melalui AI Gateway |
| i18n | next-intl, locale registry and route `/en`, `/id`, `/zh`; locale availability is capability-gated |
| Monitoring | Sentry dan status page eksternal |

Setiap adapter provider wajib mengimplementasikan kontrak domain yang sama untuk
connectivity, verification, error taxonomy, idempotency, webhook/event mapping,
rate-limit handling, dan audit. Pergantian atau penambahan provider tidak boleh
mengubah source of truth core system. Perubahan arsitektur dari keputusan ini
memerlukan RFC tertulis. Opsi NestJS terpisah tidak digunakan sebelum hosting
diverifikasi mendukung minimal dua Node.js app.

### 5.3 Gate hosting sebelum pembangunan yang bergantung pada hosting

Semua item berikut wajib dicatat pass/fail di `/DOC/hosting-verification.md` sebelum
deploy production:

1. SSH tersedia.
2. Node 22 atau fallback Node 20 tersedia.
3. Slot Node.js App tersedia.
4. Outbound TCP/TLS ke Redis managed tersedia.
5. Versi PostgreSQL dicatat.
6. Kuota disk/bandwidth cukup untuk app dan build.
7. Subdomain/addon domain dan SSL tersedia.

---

## 6. Struktur Repository yang Ditargetkan

```text
/
├── apps/web/
│   ├── app/
│   │   ├── [locale]/                 # marketing dan authenticated app
│   │   └── api/
│   │       ├── integrations/
│   │       ├── scans/
│   │       ├── controls/
│   │       ├── evidence/
│   │       ├── reports/
│   │       ├── webhooks/
│   │       └── internal/jobs/
│   ├── src/server/services/
│   │   ├── auth/
│   │   ├── billing/
│   │   ├── scan-engine/
│   │   ├── evidence/
│   │   ├── controls/
│   │   ├── regulations/
│   │   ├── ai-gateway/
│   │   ├── notifications/
│   │   └── audit/
│   └── prisma/
├── packages/ui/
├── packages/config/
├── locales/{en,id,zh}/
├── infra/aws/
├── DOC/
└── README.md
```

### 6.1 Scan engine package contract

```typescript
type Provider = "aws" | "github" | "google_workspace";
type FindingResult = "pass" | "fail" | "error" | "not_applicable";
type Severity = "critical" | "high" | "medium" | "low";

interface CheckDefinition {
  id: string;                 // immutable, e.g. aws.iam.mfa_root_enabled
  version: string;            // semver; bump for rule or interpretation changes
  provider: Provider;
  title: string;              // i18n key
  description: string;        // i18n key
  severity: Severity;
  mappedControls: string[];
  requiredPermissions: string[];
  providerContract: {
    apiVersionOrDocRevision: string;
    endpoints: string[];
    verifiedAt: string;
    fixtureRef: string;
    sandboxTestRef: string;
  };
  run(ctx: ScanContext): Promise<CheckExecutionResult>;
}

interface CheckExecutionResult {
  observedAt: string;
  coverage: {
    scope: string;
    attempted: number;
    evaluated: number;
    unavailable: number;
    complete: boolean;
  };
  findings: Array<{
    resourceKey: string;       // stable provider identifier, never display name only
    result: FindingResult;
    resourceIdentifier?: string;
    evidencePayload: object;   // redacted provider response, before AI
    evidenceHash: string;      // SHA-256 of canonical payload
    message: string;           // deterministic, non-AI
    errorCode?: string;        // provider/permission/timeout/schema/unknown
  }>;
  executionError?: {
    code: string;
    retryable: boolean;
    message: string;
  };                          // unable to run, distinct from control failure
}
```

`registry.ts` adalah single source of truth check aktif per provider.
`runner.ts` membuat `ScanRun`, menjalankan check, menulis evidence/finding, dan
memperbarui control status. `aggregator.ts` menghitung status dan score. Registry
yang berstatus `not_implemented` atau `verification_required` tidak boleh dipanggil
sebagai check live dan tidak boleh menghasilkan `pass`.

### 6.2 Capability registry dan proof record

Setiap check, connector, report, dan AI feature memiliki `CapabilityRecord`:

```text
capabilityId, status, owner, introducedAt, lastVerifiedAt,
providerDocsRef, codeRef, testRefs, limitations, expiresAt
```

`live_verified` memiliki masa berlaku maksimum 90 hari untuk provider checks, atau
lebih singkat bila provider mengumumkan deprecation. CI menolak release jika proof
record kedaluwarsa, fixture tidak tersedia, atau status capability bertentangan
dengan registry kode. UI/API harus mengekspos status capability untuk admin dan
menampilkan limitation yang relevan kepada customer.

### 6.3 Kontrak lintas modul

Semua modul domain harus mengikuti kontrak berikut sebelum diberi status
`implemented`:

```text
ModuleContract
  moduleId, version, owner, tenantScope, dataClassification
  sourceOfTruth, actors[], capabilities[], nonGoals[]
  states[], transitions[], terminalStates[]
  commands[]          # mutation yang idempotent
  queries[]           # read yang cursor-paginated dan tenant-scoped
  domainEvents[]      # immutable fact untuk modul lain
  failureClasses[]    # validation/auth/conflict/provider/timeout/schema/system
  auditEvents[]       # actor, purpose, scope, result, correlationId
  retentionPolicy, rollbackStrategy, runbookRefs
  slo, costCeiling, featureFlag, capabilityRecordId
```

Aturan yang mengikat:

- Modul hanya mengubah source of truth miliknya sendiri. Modul lain membaca
  domain event atau projection versioned, bukan menulis tabel internal secara
  langsung.
- Command yang dapat dipanggil ulang menggunakan `Idempotency-Key`; request hash
  berbeda pada key yang sama selalu ditolak sebagai conflict.
- Query selalu menerima resolved `organizationId` dari session/actor context.
  `organizationId` dari request hanya digunakan sebagai input yang divalidasi,
  bukan sebagai authorization.
- Setiap state transition menyimpan `fromState`, `toState`, actor, reason,
  timestamp, correlation ID, dan version. Transition tidak valid menghasilkan
  error stabil tanpa mengubah data.
- Error provider, timeout, dan partial result tidak boleh diubah menjadi
  `completed`, `met`, `paid`, `published`, atau `verified`.
- Event dan audit log tidak menjadi pengganti source of truth; keduanya harus
  mereferensikan entity/version yang dapat dibaca dan diverifikasi.
- Delete customer-facing hanya berupa soft delete/archive bila histori,
  evidence, legal hold, atau audit membutuhkan preservasi. Hard delete harus
  melalui policy deletion workflow yang terdokumentasi.
- UI tidak boleh menyembunyikan capability yang gagal. UI harus menampilkan
  status capability, umur data, alasan blokir, dan tindakan pemulihan yang
  tersedia.

### 6.4 Status capability dan aturan publikasi

| Status | Boleh dipanggil customer | Boleh memengaruhi compliance | Bukti minimum |
|---|---:|---:|---|
| `planned` | Tidak | Tidak | requirement dan owner |
| `not_implemented` | Tidak | Tidak | limitation dan roadmap |
| `experimental` | Hanya allowlist internal | Tidak | test terbatas + flag deny-by-default |
| `demo` | Hanya namespace demo terisolasi | Tidak | watermark + database demo |
| `verification_required` | Tidak | Tidak | daftar gap dan expiry |
| `degraded` | Terbatas, dengan banner | Tidak menaikkan status/score | incident, scope dampak, workaround |
| `live_verified` | Ya, sesuai role/plan | Ya, sesuai kontrak | proof record lengkap dan belum expired |
| `deprecated` | Tidak untuk operasi baru | Tidak | migration/rollback plan |

Perubahan dari status selain `live_verified` ke `live_verified` memerlukan
reviewer, capability diff, proof record baru, dan pemeriksaan bahwa semua
dependency module juga `live_verified`. Public Trust Page, report, chatbot,
billing entitlement, dan customer copy tidak boleh menyebut capability yang
belum `live_verified`.

---

## 7. Modul Scan & Continuous Control Monitoring

### 7.1 Entitas dan alur

```text
Integration
  → ScanRun
    → CheckDefinition
      → satu atau banyak Finding
        → CheckControlMapping
          → OrgControlStatus
            → dashboard, alert, report, chatbot
```

- `CheckDefinition`: kode versioned, bukan data yang dapat diedit customer.
- `ScanRun`: satu eksekusi pada satu integration.
- `Finding`: hasil per resource/check, termasuk referensi evidence dan pesan
  deterministik.
- Satu kontrol dapat dipetakan ke banyak check dan banyak provider.
- `executionError` berarti tidak dapat memverifikasi; bukan failure kontrol.

### 7.2 Scheduling dan idempotency

- Cron setiap enam jam memanggil `POST /api/internal/jobs/scan-runner`.
- Endpoint dilindungi secret header dan tidak menerima input organisasi bebas tanpa
  validasi server.
- Satu invocation memproses batch maksimal N integration, misalnya 20.
- Backlog drain dapat dipanggil setiap 15 menit.
- `Scan Now` dapat synchronous untuk check ringan atau enqueue + polling untuk scan
  berat.
- Lock/idempotency key mencegah dua scan aktif pada integration yang sama.
- Scan menyimpan durasi total dan durasi tiap check.
- Retry hanya untuk 429/500/503/transient dengan exponential backoff maksimal tiga
  kali; 401/403 langsung dicatat sebagai execution error.

### 7.3 Critical drift alert

Setelah scan selesai, sistem membandingkan hasil terbaru dengan scan sukses
sebelumnya pada integration yang sama. Perubahan `pass → fail` untuk severity
`critical` memicu in-app + email + WhatsApp sesuai konfigurasi organisasi.
Target: notifikasi terkirim <15 menit sejak deteksi, diukur melalui timestamp audit.

Circuit breaker: setelah lima `ScanRun` berturut-turut gagal total, integration
menjadi `error` dan auto-scan dihentikan sampai re-connect/re-verify.

### 7.4 Connector AWS

Metode wajib adalah cross-account IAM role dengan External ID, bukan access key
statis:

1. JOBEN menyediakan CloudFormation `/public/aws/joben-readonly-role.yaml`.
2. Customer membuat role di akun AWS sendiri.
3. Trust policy membatasi JOBEN account ID (`JOBEN_AWS_ACCOUNT_ID`) dan unique
   `sts:ExternalId` per integration.
4. Customer mengirim Role ARN.
5. JOBEN melakukan STS AssumeRole menggunakan credential service JOBEN.
6. Credential STS hanya hidup di memory selama scan dan tidak disimpan permanen.
7. `Integration.credentialsRef` hanya menyimpan reference/Role ARN terenkripsi.

Tidak boleh menggunakan managed `ReadOnlyAccess` generik. Policy custom hanya
memuat action yang dipakai check berstatus `live_verified`; action di bawah adalah
katalog kandidat untuk desain dan review, bukan permission yang otomatis diberikan.
Setiap release harus memiliki permission matrix yang menghubungkan action ke check,
endpoint, test evidence, dan owner. Action baseline:

```text
iam:GetAccountPasswordPolicy
iam:GetAccountSummary
iam:ListUsers
iam:ListAccessKeys
iam:GetAccessKeyLastUsed
iam:ListMFADevices
iam:GetLoginProfile
iam:ListPolicies
iam:GetPolicy
iam:GetPolicyVersion
iam:ListAttachedUserPolicies
iam:ListAttachedRolePolicies
iam:GenerateCredentialReport
iam:GetCredentialReport
s3:ListAllMyBuckets
s3:GetBucketAcl
s3:GetBucketPolicy
s3:GetBucketPolicyStatus
s3:GetBucketPublicAccessBlock
s3:GetEncryptionConfiguration
s3:GetBucketLogging
s3:GetBucketVersioning
cloudtrail:DescribeTrails
cloudtrail:GetTrailStatus
cloudtrail:GetEventSelectors
ec2:DescribeSecurityGroups
ec2:DescribeInstances
ec2:DescribeVolumes
ec2:DescribeSnapshots
kms:ListKeys
kms:DescribeKey
kms:GetKeyRotationStatus
config:DescribeConfigurationRecorders
config:DescribeConfigRules
config:GetComplianceDetailsByConfigRule
guardduty:ListDetectors
guardduty:GetDetector
rds:DescribeDBInstances
rds:DescribeDBSnapshots
sts:GetCallerIdentity
```

#### AWS checks fase 1

| ID | Severity | Control | Deterministic rule |
|---|---|---|---|
| `aws.iam.mfa_root_enabled` | critical | CC6.1 | `AccountMFAEnabled == 1` |
| `aws.iam.password_policy_strength` | high | CC6.1 | min 14; symbols; numbers; max age ≤90 |
| `aws.iam.mfa_all_users` | high | CC6.1 | user password aktif wajib `mfa_active=true` |
| `aws.iam.unused_access_keys` | medium | CC6.1, CC6.3 | key tidak dipakai >90 hari atau key tua tanpa last use |
| `aws.iam.no_full_admin_inline_policy` | critical | CC6.1 | Allow `Action:*`, `Resource:*`, tanpa condition |
| `aws.s3.no_public_buckets` | critical | CC6.1, CC6.6 | policy status/ACL/public block tidak publik |
| `aws.s3.encryption_at_rest` | high | CC6.3 | default encryption AES256 atau aws:kms |
| `aws.s3.versioning_enabled` | medium | CC7.2 | bucket penting berstatus Enabled |
| `aws.cloudtrail.enabled_multiregion` | critical | CC7.1, CC7.2 | multi-region, logging, log validation aktif |
| `aws.ec2.security_group_no_open_ssh_rdp` | critical | CC6.6 | port 22/3389 tidak terbuka ke `0.0.0.0/0` |
| `aws.ebs.encryption_at_rest` | high | CC6.3 | semua volume `Encrypted == true` |
| `aws.kms.key_rotation_enabled` | medium | CC6.3 | customer-managed key rotation aktif |
| `aws.guardduty.enabled` | high | CC7.1 | detector utama `ENABLED` |
| `aws.rds.encryption_at_rest` | high | CC6.3 | semua RDS `StorageEncrypted == true` |

Fase 1 minimal mengimplementasikan delapan check prioritas: root MFA, password
policy, S3 public, S3 encryption, CloudTrail, security group, unused access key,
dan full admin policy. Check yang belum benar-benar jalan berstatus
`not_implemented`, bukan placeholder pass.

### 7.5 Connector GitHub

Gunakan GitHub App read-only, bukan PAT personal. Scope minimum:
Administration, Metadata, Contents, Secret scanning alerts, Dependabot alerts,
dan branch protection sesuai API resmi saat implementasi.

Simpan `installationId`, bukan installation token. Token dibuat dari App JWT dan
expire otomatis. Gunakan GraphQL untuk batch query bila sesuai; hormati rate limit
5000 request/jam dan gunakan pagination/ETag/incremental scan.

| ID | Severity | Control | Deterministic rule |
|---|---|---|---|
| `github.org.2fa_enforced` | critical | CC6.1 | `requiresTwoFactorAuthentication == true` |
| `github.org.no_outside_collaborators_admin` | high | CC6.1 | outside collaborator tidak boleh admin |
| `github.repo.branch_protection_default` | high | CC8.1 | default branch wajib PR review ≥1 dan status check |
| `github.repo.secret_scanning_enabled` | high | CC6.1, CC6.3 | private repo secret scanning enabled |
| `github.repo.dependabot_alerts_enabled` | medium | CC7.1 | vulnerability alerts API merespons enabled |
| `github.org.inactive_members_review` | low | CC6.1 | member tanpa aktivitas >180 hari ditinjau |
| `github.repo.no_secrets_in_workflow_files` | medium | CC6.3 | regex dasar pada workflow; label sebagai heuristic |

Heuristic workflow tidak menggantikan native secret scanning.

### 7.6 Connector Google Workspace

Gunakan service account JOBEN dengan Domain-Wide Delegation. Customer hanya
mendaftarkan Client ID dan scopes di Admin Console. Scope minimum:

```text
https://www.googleapis.com/auth/admin.directory.user.readonly
https://www.googleapis.com/auth/admin.directory.group.readonly
https://www.googleapis.com/auth/admin.reports.audit.readonly
https://www.googleapis.com/auth/admin.directory.domain.readonly
```

| ID | Severity | Control | Deterministic rule |
|---|---|---|---|
| `gws.org.2fa_enforced` | critical | CC6.1 | user domain wajib enforced 2SV |
| `gws.admin.super_admin_count_minimal` | medium | CC6.1 | count > konfigurasi atau ==0 menjadi fail |
| `gws.drive.external_sharing_restricted` | high | CC6.6 | domain sharing setting dibatasi/dimonitor |
| `gws.user.suspended_accounts_no_access` | low | CC6.1 | suspended user tidak punya residual session |

Endpoint Google untuk sharing dan policy harus diverifikasi dari dokumentasi resmi
saat implementasi karena API dapat deprecated/migrasi. Jika belum terverifikasi,
check tetap `not_implemented`.

---

## 8. Status, Aggregation, dan Scoring

### 8.1 Status finding

```text
pass | fail | error | not_applicable
```

`error` digunakan saat check gagal dijalankan. `fail` hanya untuk resource yang
berhasil diperiksa tetapi melanggar rule.

### 8.2 Status control

```text
not_started | needs_review | partial | met | not_met
```

Setiap status selalu disertai:

```text
status, evaluatedAt, sourceScanRunIds[], coverage,
freshness: fresh | stale | expired, dataQuality,
blockingReasons[], ruleVersions[]
```

`dataQuality`:

```text
verified_complete | verified_partial | stale | provider_error |
permission_error | schema_error | not_available
```

Aturan:

1. Check relevan belum pernah berhasil dijalankan → `not_started`/`needs_review`.
2. Semua finding relevan pass → `met`.
3. Ada finding fail → `not_met`.
4. Satu check menghasilkan resource campuran pass/fail → `partial` dengan
   proporsi eksplisit, misalnya `9/10`.
5. Error permission/provider tidak boleh dikonversi menjadi pass.
6. Status harus mengambil finding terbaru dari scan sukses pada setiap integration
   relevan dan seluruh proses wajib tenant-scoped.
7. Jika coverage tidak lengkap atau ada error pada scope yang diwajibkan, status
   maksimal `needs_review`/`partial`; tidak boleh `met`.
8. Jika evidence melewati freshness window check, status menjadi `needs_review`
   dengan `freshness=expired`; hasil lama tetap dapat dilihat sebagai histori dan
   tidak dipakai untuk menyatakan kondisi saat ini.
9. `not_applicable` hanya boleh berasal dari rule yang secara eksplisit menyatakan
   alasan dan scope; customer tidak boleh mengubah fail menjadi N/A untuk menaikkan
   score tanpa audit trail dan role yang berwenang.
10. Tidak ada fallback ke scan terakhir yang diam-diam. UI harus menyebut “last
    verified at”, umur data, scan yang gagal, dan apa yang belum tercakup.

### 8.3 Framework score

```text
score =
  Σ(weight control × contribution status)
  / Σ(weight seluruh control aktif)
  × 100
```

Kontribusi:

- `met` = 1
- `partial` = rasio resource pass
- `not_met`, `not_started`, `needs_review` = 0

Bobot control eksplisit dalam seed data. Default 1; bobot lebih tinggi harus
memiliki alasan GRC yang terdokumentasi. Control yang belum terhubung tetap masuk
penyebut agar skor tidak naik secara artifisial.

Score yang dapat dilihat customer wajib mengembalikan:

```text
score, denominator, includedControls, excludedControls,
coveragePercent, dataQuality, calculatedAt, algorithmVersion
```

Jika `dataQuality` bukan `verified_complete`, score diberi label `incomplete` dan
dashboard menampilkan alasan. Score tidak boleh dipakai untuk sort/filter sebagai
“passed” ketika incomplete. Perubahan formula menaikkan `algorithmVersion`,
menyimpan snapshot lama, dan tidak mengedit histori.

Dashboard selalu menampilkan timestamp “Data per” untuk setiap integration dan
framework, serta timezone.

---

## 9. Evidence, Remediation, dan Report

### 9.1 Evidence lifecycle

1. Provider client menerima response.
2. Secret dan token di-redact.
3. Payload setelah redaction divalidasi terhadap schema provider dan canonicalized
   dengan algoritma versioned (`canonicalizationVersion`).
4. SHA-256 dihitung dari bytes canonical, lalu object disimpan dengan key
   content-addressed yang mengandung hash. Hash dihitung ulang saat write dan
   saat read/verify.
5. Object storage memakai encryption, versioning, retention/WORM lock sesuai
   retention policy. Penghapusan hanya melalui documented legal/retention workflow,
   tidak melalui UI biasa.
6. DB menyimpan reference, hash, canonicalization version, timestamp, source
   endpoint, provider request ID, schema version, dan relasi tenant.
7. Evidence tidak dapat diedit atau ditimpa; koreksi membuat record baru dan
   menghubungkan `supersedesEvidenceId` tanpa menghapus record lama.
8. Job verifikasi berkala memeriksa object masih ada dan hash cocok. Kegagalan
   membuat incident dan evidence menjadi `integrity_failed`, bukan tetap valid.
9. Setiap akses/download/verify Evidence Vault dicatat dalam audit log dengan
   actor, purpose, scope, result, dan correlation ID.

DB hanya menyimpan reference ke payload untuk menghindari evidence besar di
PostgreSQL. `rawEvidenceJson` dalam kontrak scan berarti payload yang tersimpan
verbatim di object storage setelah redaction, bukan JSON yang dibuat/diringkas AI.

Redaction wajib diuji dengan secret canary dan mencakup token, authorization
header, private key, webhook secret, email/PII yang tidak diperlukan, serta nested
provider payload. Jika redaction atau schema validation gagal, evidence tidak
boleh dipakai untuk evaluation dan scan berstatus `schema_error`.

### 9.2 Remediation guidance

Setiap finding `fail` harus memiliki template versioned:

```typescript
interface RemediationGuidance {
  findingId: string;
  summary: string;              // manusia, deterministik
  whyItMatters: string;         // manusia, deterministik
  stepsToFix: RemediationStep[];
  estimatedEffort: "low" | "medium" | "high";
  aiContextualNote?: string;    // optional, jelas sebagai saran AI
}
```

`stepsToFix` ditulis manusia per check. AI hanya boleh menambahkan
`aiContextualNote` dan harus diberi label “Saran AI — verifikasi sebelum eksekusi”.

### 9.3 PDF report

Export server-side per organisasi/framework wajib memuat:

1. Cover: logo customer, logo JOBEN, framework, tanggal, rentang scan.
2. Executive summary: score, gauge, count status, integrations, tren 30/90 hari.
3. Control detail: kode, judul, status, finding ringkas, last checked.
4. Findings/remediation: critical sampai low.
5. Evidence appendix: hash SHA-256 dan timestamp.
6. Disclaimer attestation wajib.

Gunakan `DailyComplianceSnapshot` untuk tren tanpa query histori mentah yang berat.
PDF diuji pada minimal dua PDF reader.

---

## 10. Model Data Canonical

Model inti PostgreSQL/Prisma:

```text
Organization
  id, name, slug, domain, region, preferredLocale, mode, status,
  createdAt, archivedAt

User
  id, clerkUserId, email, preferredLocale, status, lastIdentitySyncAt

OrganizationMembership
  organizationId, userId, role, status, invitedBy, invitedAt, acceptedAt,
  suspendedAt, revokedAt, version

OrganizationInvitation
  organizationId, emailHash, role, tokenRef, status, expiresAt,
  createdBy, acceptedBy, acceptedAt, revokedAt

Framework
  code, name, version, isActive

OrgFramework
  organizationId, frameworkId, status, targetAuditDate

Control
  frameworkId, code, title, description, category, weight

OrgControlStatus
  organizationId, controlId, status, lastCheckedAt

Integration
  organizationId, provider, status, credentialsRef, lastSyncAt,
  consecutiveFailures, verificationVersion, verifiedAt, revokedAt

IntegrationVerification
  integrationId, status, requestedBy, providerIdentity, permissionResults,
  providerContractVersion, startedAt, finishedAt, errorCode, evidenceId

CheckDefinitionMeta
  id, provider, severityDefault, status, version, requiredPermissions,
  providerContractVersion, verifiedAt, expiresAt

CheckControlMapping
  checkDefinitionId, controlId

ScanRun
  organizationId, integrationId, status, trigger, requestedBy, idempotencyKey,
  leaseOwner, leaseExpiresAt, startedAt, finishedAt, cancellationRequestedAt,
  retryCount, coverage, failureClass, correlationId

ScanCheckAttempt
  scanRunId, checkDefinitionId, status, attemptNo, startedAt, finishedAt,
  resourceCounts, errorCode, providerRequestIds, correlationId

Evidence
  organizationId, sourceIntegrationId, scanRunId, findingId, controlStatusId,
  type, storageUrl, contentHash, canonicalizationVersion, schemaVersion,
  providerRequestId, sourceEndpoint, collectedAt, retentionUntil, immutable,
  integrityStatus, legalHoldStatus, supersedesEvidenceId, createdAt

ObservedFact
  evidenceId, provider, resourceKey, observedAt, payloadSchema, extractedFields

Finding
  organizationId, scanRunId, checkDefinitionId, checkVersion, observedFactId,
  resourceKey, result, errorCode, message, detectedAt, evaluatedAt,
  coverageStatus, supersedesFindingId

RemediationTemplate
  checkDefinitionId, summaryI18nKey, whyItMattersI18nKey,
  stepsI18nKey, estimatedEffort

Remediation
  organizationId, findingId, status, assigneeUserId, dueAt, acknowledgedAt,
  startedAt, blockedAt, resolvedAt, closedAt, closedBy, closureReason,
  reopenedAt, version

RemediationActivity
  remediationId, actor, type, commentRef, evidenceId, createdAt, correlationId

DailyComplianceSnapshot
  organizationId, frameworkId, snapshotDate, score, metCount,
  partialCount, notMetCount, needsReviewCount, coveragePercent,
  dataQuality, algorithmVersion, sourceProjectionVersion

AuditLog / ProviderApiLog
  organizationId, actor, action, resource, endpoint, statusCode,
  durationMs, createdAt, metadata

ReportSnapshot / ReportArtifact
  organizationId, frameworkId, snapshotVersion, status, requestedBy,
  inputHash, contentHash, storageRef, expiresAt, revokedAt, createdAt

NotificationRule / NotificationDelivery
  organizationId, eventType, preference, channel, dedupeKey, status,
  attemptNo, providerMessageRef, nextRetryAt, deliveredAt, acknowledgedAt,
  failureClass, createdAt

Policy / Risk / Vendor / RegulationUpdate
  organizationId, entityType, status, version, sourceRef, sourceHash,
  effectiveAt, reviewDueAt, approvedBy, approvedAt, publishedAt, supersededAt

TrustPublication / AuditorAccess
  organizationId, status, scope, capabilitySnapshot, evidenceSnapshot,
  tokenRef, expiresAt, revokedAt, publishedBy, createdAt

Plan / Subscription / PaymentEvent / Entitlement
  planVersion, organizationId, provider, externalRef, status, effectiveAt,
  idempotencyKey, eventHash, receivedAt, processedAt, reconciledAt

AiUsageLog / ChatSession / ChatMessage
  organizationId, actor, purpose, retrievalRefs, modelVersion, status,
  inputTokens, outputTokens, estimatedCost, citations, refusalCode, createdAt

Incident / BackupManifest / MigrationRecord
  scope, severity, status, owner, detectedAt, containedAt, resolvedAt,
  timelineRef, checksum, schemaVersion, verifiedAt, rollbackRef
```

Entitas tambahan yang tetap diperlukan di luar tabel canonical:

```text
Questionnaire, QuestionnaireItem,
Referral
```

Constraint wajib:

- Semua entity customer memiliki `organizationId` langsung atau relasi yang
  dapat divalidasi.
- `User` adalah identity global; role dan akses organisasi hanya berasal dari
  `OrganizationMembership` yang aktif. Tidak boleh ada kolom `User.role` atau
  `User.organizationId` yang dijadikan authorization source.
- Satu user boleh memiliki banyak membership, tetapi setiap request, job, signed
  URL, dan webhook wajib menyelesaikan tepat satu organization context sebelum
  repository dipanggil.
- Invite token disimpan sebagai reference/hash, bukan plaintext; token memiliki
  expiry, single-use, revoke, dan replay audit.
- Unique key pada kombinasi organisasi + entity yang sesuai.
- `Integration.credentialsRef` bukan credential plaintext.
- `Evidence.immutable` tidak dapat diubah setelah ditulis; database trigger/service
  guard menolak UPDATE/DELETE pada record final.
- `Finding` dan `Evidence` menyimpan organization scope yang dapat divalidasi
  langsung, atau repository wajib melakukan join-scoped authorization sebelum
  mengembalikannya.
- `ObservedFact` tidak dapat dibuat tanpa evidence yang hash-valid.
- `ControlStatus` adalah projection yang dapat direbuild dari finding append-only;
  ia bukan source of truth.
- `publishedAt` policy AI tetap null sampai approval.
- `humanApproved` questionnaire wajib true sebelum export/send.
- State ledger untuk subscription, report, notification, auditor access, incident,
  dan job bersifat append-only atau memiliki transition audit; projection boleh
  direbuild dari ledger.
- Foreign key tenant tidak cukup sebagai authorization: service/repository wajib
  melakukan tenant check pada parent dan child, termasuk object storage key,
  queue message, export, dan webhook reconciliation.
- Semua timestamp disimpan UTC dengan timezone metadata untuk tampilan; clock
  skew provider dicatat pada verification/scan dan tidak boleh diabaikan.
- `mode=demo` hanya boleh merujuk ke database/object namespace demo dan tidak
  dapat diubah menjadi live melalui update biasa.

### 10.1 State machine canonical core system

Nama state berikut adalah kontrak domain, bukan label UI. Implementer boleh
menambah state internal yang tidak terlihat customer, tetapi tidak boleh mengubah
arti, melewati guard, atau menghapus state terminal tanpa RFC. Semua transition
menulis actor, reason, version, `correlationId`, dan timestamp UTC.

| Modul | State canonical | Terminal sukses/gagal | Guard minimum |
|---|---|---|---|
| M-01 | membership `invited → active → suspended/revoked → deleted`; invitation `pending → accepted/expired/revoked` | membership `active`, invitation terminal | verified identity, MFA policy, owner-transfer approval, single-use invite |
| M-02 | integration `draft → verifying → verified → degraded/error → revoked` | `verified` atau `revoked` | provider identity, scope, permission matrix, clock skew, credential reference |
| M-03 | run `queued → running → cancelling → completed/partial/failed/cancelled/dead_letter` | semua state setelah `queued` yang terminal | lease owner, idempotency, retry budget, cancellation boundary, dead-letter reason |
| M-04 | adapter `registered → verification_required → verified → deprecated/disabled` | `verified`, `disabled`, atau `deprecated` | provider contract, fixture, sandbox, permission matrix, expiry |
| M-05 | evidence `staged → redacted → validated → committed → integrity_failed/quarantined/expired` | `committed`, atau `quarantined` bila gagal | hash/canonical bytes, WORM/retention, legal hold, immutable write |
| M-06 | evaluation `queued → evaluating → evaluated → projected/rebuild_failed`; control `not_started/needs_review/partial/met/not_met` | evaluation `projected`; control status tetap projection | evidence valid, rule version, coverage, freshness, denominator |
| M-07 | remediation `open → acknowledged → in_progress → blocked → resolved → closed`; `closed → reopened` | `closed` atau `archived` | actor assignment, due-date policy, latest verification finding, closure reason |
| M-08 | view `loading → ready/empty/partial/stale/error/degraded` | tidak ada compliance state di UI | response provenance, capability status, locale completeness, tenant query |
| M-09 | report `requested → queued → generating → ready → expired/revoked/failed` | `ready`, `expired`, `revoked`, atau `failed` | snapshot version, input hash, access scope, artifact hash |
| M-10 | delivery `eligible → queued → sending → delivered/failed/dead_letter`; `delivered → acknowledged` | `delivered`, `acknowledged`, atau `dead_letter` | preference, quiet hour, emergency policy, dedupe, retry budget |
| M-11 | record `draft → in_review → approved → published → superseded/withdrawn` | `published`, `superseded`, atau `withdrawn` | reviewer separation, source provenance, mapping impact, effective date |
| M-12 | publication `draft → published → unpublishing → unpublished`; access `issued → active → expired/revoked` | publication `unpublished`; access `expired/revoked` | live capability, eligible evidence, scope, expiry, revoke audit |
| M-13 | subscription `pending → active → past_due/grace → cancelled/expired`; payment event `received → verified → applied/reconciled/rejected` | subscription `active/cancelled/expired`; event `reconciled/rejected` | signature, ordering policy, provider reference, entitlement projection |
| M-14 | request `received → retrieving → generating → validating → answered/refused/failed`; draft `draft → approved/rejected` | request answered/refused/failed; draft approved/rejected | scoped retrieval, citation verification, safety schema, budget, human approval |
| M-15 | incident `detected → acknowledged → contained → recovering → resolved → reviewed`; release `candidate → blocked/approved → deployed → rolled_back` | incident `reviewed`; release `deployed/rolled_back` | severity owner, evidence timeline, approval separation, rollback proof |

Tidak boleh ada transition langsung dari state input ke state sukses hanya karena
request HTTP berhasil. `202 Accepted` berarti command diterima/queued; state
terminal hanya boleh ditulis oleh service/worker pemilik source of truth setelah
guard dan side effect wajib berhasil. Jika side effect wajib gagal, entity masuk
state error/degraded yang sesuai dan retry/recovery dicatat.

### 10.2 Domain event minimum

Event berikut adalah immutable fact untuk komunikasi antar modul. Payload event
wajib memiliki `eventId`, `eventType`, `eventVersion`, `occurredAt`, `actor`,
`organizationId`, `entityId`, `entityVersion`, `correlationId`, dan `dataRef`.
Event tidak boleh membawa raw credential, signed URL, atau payload evidence
restricted; consumer mengambil data melalui scoped service.

| Event | Producer → consumer | Aturan |
|---|---|---|
| `MembershipActivated`, `MembershipRevoked` | M-01 → seluruh modul | cache/permission session harus invalidated; revoke berlaku pada request berikutnya |
| `IntegrationVerified`, `IntegrationRevoked`, `IntegrationDegraded` | M-02 → M-03/M-04/M-08/M-10 | scan baru diblokir bila tidak verified; histori tetap dapat dibaca |
| `ScanQueued`, `ScanStarted`, `ScanProgressed`, `ScanCompleted`, `ScanPartial`, `ScanFailed` | M-03 → M-06/M-08/M-10/M-15 | duplicate event aman; progress bukan evidence dan bukan compliance status |
| `EvidenceCommitted`, `EvidenceIntegrityFailed` | M-05 → M-06/M-08/M-09/M-12/M-14 | integrity failure menarik eligibility; consumer tidak boleh memakai evidence tersebut |
| `FindingEvaluated`, `ControlProjectionRebuilt` | M-06 → M-07/M-08/M-09/M-10/M-14 | finding immutable; projection version wajib cocok dengan report |
| `RemediationResolved`, `RemediationReopened` | M-07 → M-06/M-08/M-10 | resolved tidak sama dengan closed sebelum verification rule lulus |
| `ReportReady`, `ReportRevoked` | M-09 → M-08/M-12/M-15 | access harus scope/time-bound dan dapat dicabut |
| `NotificationDelivered`, `NotificationFailed` | M-10 → M-08/M-15 | delivery tidak mengubah finding atau control status |
| `RecordApproved`, `RecordPublished`, `RecordWithdrawn` | M-11 → M-08/M-12/M-14 | publish tanpa approval ditolak server-side |
| `TrustPublished`, `AuditorAccessRevoked` | M-12 → M-08/M-15 | publikasi harus menyimpan eligibility snapshot dan expiry |
| `SubscriptionChanged`, `EntitlementChanged` | M-13 → M-08/M-15 | entitlement hanya membatasi capability; tidak dapat menulis scan truth |
| `AiAnswerProduced`, `AiRefused`, `AiDraftApproved` | M-14 → M-08/M-11/M-15 | output AI tidak menulis compliance truth; citation/safety result tersimpan |
| `IncidentOpened`, `IncidentResolved`, `ReleaseApproved`, `ReleaseRolledBack` | M-15 → audit/operational consumers | incident closure membutuhkan corrective action dan timeline |

Consumer wajib idempotent berdasarkan `eventId`, menyimpan offset/processing
result, dan mengirim event ke dead-letter setelah retry policy habis. Replay event
harus menghasilkan state yang sama atau conflict yang dapat diaudit, bukan duplicate
finding, notification, charge, atau approval.

---

## 11. API dan Job Contract

Endpoint minimum:

| Method | Endpoint | Tujuan |
|---|---|---|
| `GET` | `/api/integrations` | daftar integration tenant |
| `POST` | `/api/integrations/{provider}/connect` | mulai flow connector |
| `POST` | `/api/integrations/{id}/verify` | test koneksi/permission |
| `POST` | `/api/integrations/{id}/scan` | Scan Now dengan lock |
| `GET` | `/api/scans/{id}` | status dan progress scan |
| `GET` | `/api/controls` | status kontrol teragregasi |
| `GET` | `/api/controls/{id}` | finding, evidence, remediation |
| `GET` | `/api/evidence/{id}` | akses signed/authorized evidence |
| `POST` | `/api/reports` | enqueue/generate PDF report |
| `GET` | `/api/reports/{id}` | status, provenance, dan signed report access |
| `POST` | `/api/reports/{id}/revoke` | cabut akses report |
| `GET` | `/api/remediations` | daftar remediation tenant dengan cursor |
| `POST` | `/api/remediations/{id}/transition` | acknowledge/start/block/resolve/close/reopen |
| `POST` | `/api/remediations/{id}/activities` | komentar atau attach fix evidence |
| `GET` | `/api/audit` | audit log sesuai role dan scope |
| `POST` | `/api/chat/sessions` | mulai sesi chatbot tenant |
| `POST` | `/api/chat/sessions/{id}/messages` | pertanyaan RAG |
| `POST` | `/api/webhooks/xendit` | callback payment terverifikasi |
| `POST` | `/api/webhooks/{provider}` | inbox webhook provider yang diaktifkan |
| `GET` | `/api/billing/subscription` | subscription dan entitlement efektif |
| `POST` | `/api/billing/checkout` | membuat checkout idempotent |
| `GET` | `/api/organizations/members` | daftar membership organisasi |
| `POST` | `/api/organizations/invitations` | membuat invitation |
| `POST` | `/api/organizations/invitations/{id}/revoke` | mencabut invitation |
| `POST` | `/api/organizations/members/{id}/transition` | suspend/revoke/role change |
| `GET` | `/api/capabilities` | capability status dan limitation yang boleh dilihat |
| `POST` | `/api/internal/jobs/scan-runner` | drain scan queue |
| `POST` | `/api/internal/jobs/evidence-verify` | verifikasi hash/object/retention |
| `POST` | `/api/internal/jobs/notification-retry` | drain notification retry/dead-letter |
| `POST` | `/api/internal/jobs/backup-verify` | verifikasi backup manifest dan restore sample |
| `POST` | `/api/internal/jobs/regulation-monitor` | fetch perubahan regulasi |

Internal job:

- wajib secret header;
- validasi method, timestamp/replay protection, dan payload;
- idempotent;
- tidak mengembalikan secret atau raw credential dalam response/log;
- mencatat `jobRunId`, batch, error, dan completion.

### 11.1 Contract HTTP yang wajib

- Semua endpoint customer memerlukan authenticated session, resolved
  `organizationId`, authorization decision, dan correlation ID. `organizationId`
  dari body/query/path tidak dipercaya sampai dibandingkan dengan session.
- Request body divalidasi dengan schema versioned; unknown field ditolak untuk
  operasi sensitif. Error response memakai envelope stabil:
  `code`, `message`, `details`, `correlationId`; tidak ada stack trace atau
  credential.
- Semua mutation menerima `Idempotency-Key`. Key diikat ke organization, actor,
  route, request hash, dan expiry; key sama dengan payload berbeda ditolak.
- State transition divalidasi server-side sebagai finite state machine. UI tidak
  dapat memaksa `verified`, `met`, `published`, `paid`, atau `completed`.
- Pagination memakai cursor; endpoint list memiliki limit maksimum dan query
  timeout. Filter tenant diterapkan sebelum pagination/count.
- Signed evidence/report URL berumur pendek, tidak mengungkap object key, dan
  setiap akses tetap diaudit.
- `verify` hanya mengubah integration menjadi `verified` setelah identity, scope,
  permission matrix, clock skew, dan provider request test lulus. Verify gagal
  tidak menghapus evidence historis.
- `scan` membuat satu `ScanRun` atau mengembalikan run aktif yang sama. HTTP 202
  berarti queued, bukan sukses; client harus membaca progress dan final status.
- Transition endpoint hanya menerima command yang diizinkan oleh state machine;
  client tidak boleh mengirim target state arbitrer. Response wajib mengembalikan
  state baru, transition version, audit reference, dan correlation ID.
- `POST /api/reports` membuat snapshot dari versi projection yang eksplisit.
  Report tidak boleh membaca data live secara terpisah untuk bagian yang sama;
  input hash yang sama menghasilkan artifact yang sama secara semantik.
- Remediation close wajib membawa `verificationFindingId` dari scan terbaru dan
  server harus memastikan finding tersebut scope-nya sama, evidence hash-valid,
  serta memenuhi rule penutupan.
- Membership transition, role change, ownership transfer, subscription change,
  trust publication, policy approval, dan AI approval wajib memakai optimistic
  concurrency (`version`/`If-Match`) agar update lama menjadi conflict, bukan
  menimpa keputusan terbaru.
- GET list yang berpotensi besar wajib memiliki `limit` maksimum, cursor expiry,
  sort yang allowlisted, dan query timeout; count mahal harus eksplisit/asinkron.
- Webhook payment menyimpan raw payload ter-redact, memverifikasi signature resmi,
  menolak replay, dan memproses event melalui inbox idempotent sebelum subscription
  berubah.
- Internal job memakai secret management, HMAC/timestamp/replay protection,
  allowlist route, timeout, lease/lock, dan dead-letter record. Secret tidak
  boleh diletakkan di URL.

---

## 12. UI/UX dan Design System

### 12.1 Token

```css
--joben-primary: #FF6A1A;
--joben-primary-dark: #D9530F;
--joben-primary-light: #FFA766;
--joben-bg-dark: #0B0D10;
--joben-bg-light: #FFFFFF;
--joben-surface: #14171C;
--joben-text-primary: #EDEFF2;
--joben-text-muted: #9AA1AC;
--joben-success: #2FBF71;
--joben-warning: #F5A524;
--joben-danger: #E5484D;
--joben-border: #23262B;
--joben-radius: 12px;
```

Gunakan Inter dan Noto Sans SC. Layout boleh mengambil inspirasi pola SaaS
compliance modern, tetapi visual, copy, icon, dan dashboard asset harus original.
Jangan menggunakan palet biru/ungu sebagai identitas utama.

### 12.2 Screen requirements

**Trust Dashboard**

- Score card per framework dengan progress ring.
- Control Monitoring breakdown kategori.
- Count met/partial/not_met/needs_review berasal dari query DB.
- Timestamp last scan selalu terlihat.
- Critical drift dan data freshness jelas.

**Integrations**

- Provider, connection status, last scan, pass/fail/error count.
- `Scan Now`, reconnect, verify permission.
- Pesan error actionable, misalnya trust policy atau permission spesifik.

**Control Detail**

- Filter severity/status.
- Expand finding untuk raw evidence metadata dan remediation.
- Tampilkan `error`/`needs_review` berbeda visual dari fail.
- Evidence hash dan timestamp dapat dibuka auditor.

**Evidence Vault**

- Filter provider/control/date/type.
- Preview/download dengan authorization dan audit log.
- Tidak mengubah evidence.

**Empty/error state**

Setiap layar yang bergantung scan memiliki state eksplisit:

> “Belum ada integrasi tersambung — hubungkan AWS, GitHub, atau Google Workspace
> untuk mulai memantau kontrol Anda.”

CTA: “Hubungkan Integrasi Pertama Anda”. Tidak boleh ada grid kosong atau skor
default.

### 12.3 i18n

Locale fase 1: `en`, `id`, `zh`. Preferensi tersimpan user mengalahkan cookie,
geo-IP, `Accept-Language`, lalu fallback `en`. Semua halaman customer-facing,
email, invoice, dan chatbot harus diterjemahkan sebelum GA. Istilah standar seperti
SOC 2 Type II dan ISO 27001 tetap memakai nama resmi.

---

## 13. AI Gateway dan Automation

Semua AI melalui:

```text
src/server/services/ai-gateway/
  provider.ts
  router.ts
  providers/{gemini,groq,deepseek}.ts
  usage-logger.ts
```

Routing default:

| Fitur | Primary | Fallback |
|---|---|---|
| Questionnaire dokumen panjang | Gemini | Groq → DeepSeek |
| Chatbot | Groq | Gemini |
| Regulation summary | Gemini | DeepSeek |
| Policy draft | Gemini | Groq |
| Translation | Gemini | Groq |

Setiap call mencatat provider, feature, token input/output, estimasi biaya, dan
organisasi pada `AiUsageLog`.

AI bukan source of truth dan tidak boleh menulis langsung ke `Control`,
`Finding`, `Evidence`, `Subscription`, atau status compliance apa pun.

### 13.1 AI safety contract

Sebelum request AI:

1. Data access policy memilih dokumen berdasarkan `organizationId`, role, purpose,
   retention, dan classification. Retrieval lintas tenant harus mustahil pada
   repository layer, bukan hanya prompt.
2. Hanya evidence yang hash-valid, belum expired, dan memiliki source citation yang
   boleh masuk context. Evidence `integrity_failed`, `schema_error`, atau demo
   tidak boleh masuk context live.
3. Context diberi boundary antara instruksi sistem, data provider, dan teks user.
   Instruction dari evidence atau dokumen dianggap untrusted data untuk mencegah
   prompt injection.
4. PII/secret minimization dan redaction dilakukan sebelum request; provider AI
   tidak boleh menerima credential atau raw token.

Setiap response wajib divalidasi terhadap schema:

```text
answer, citations[{evidenceId, findingId?, quote, observedAt}],
uncertainties[], refusedClaims[], disclaimer
```

Rules:

- Citation harus merujuk ke evidence yang benar-benar dipakai dan quote harus dapat
  ditemukan pada payload/document chunk; citation yang tidak dapat diverifikasi
  membuat response ditolak.
- Jika evidence tidak cukup, model harus menjawab `insufficient_evidence` dan
  menyebut data yang dibutuhkan; model dilarang mengisi celah dengan asumsi.
- Model tidak boleh mengubah status, menyimpulkan “compliant”, menjanjikan audit
  outcome, atau memberikan legal advice.
- Policy/questionnaire/regulation draft memiliki `draft` state sampai human
  approval; approval menyimpan actor, timestamp, version, diff, dan evidence basis.
- Input/output, model version, retrieval IDs, latency, refusal, safety result, dan
  cost dicatat tanpa menyimpan secret; tenant dapat menghapus chat sesuai policy.
- Model fallback tidak boleh mengubah safety contract atau menghilangkan citation.

AI release gate mencakup golden set berisi pertanyaan answerable, unanswerable,
cross-tenant, prompt injection, stale evidence, dan legal-claim cases. Release
gagal bila citation salah, refusal tidak konsisten, atau data organisasi lain
terlihat pada output/log.

### 13.2 In-app chatbot

- RAG hanya mengambil `Control`, `Evidence`, `Policy`, dan `OrgControlStatus` milik
  organization yang sedang login.
- Retrieval dan embedding wajib tenant-scoped.
- Jawaban compliance harus menyertakan disclaimer dan tidak boleh menjanjikan
  sertifikasi/lulus audit.
- Chatbot menjelaskan status yang ada; ia tidak boleh mengubah status.

### 13.3 Public chatbot

Sesi anonim tidak memiliki `organizationId`, hanya menjawab FAQ dan
lead-qualification. Tidak boleh menerima atau menyimpan data customer privat.
Lead yang qualified dikirim ke sales sesuai consent dan policy privacy.

---

## 14. Billing, Regulation, dan Notification

### 14.1 Pricing

Harga berasal dari tabel versioned `Plan`/`Price`, bukan hardcode frontend. Setiap
price memiliki `currency` ISO 4217, minor-unit amount, billing interval, tax
behavior, region/market eligibility, effective period, dan provider mapping.
Seed MVP awal:

| Plan | Bulanan | Tahunan | Framework | Integrasi |
|---|---:|---:|---:|---:|
| Starter | sekitar Rp2,3 juta | sekitar Rp22 juta | 1 | 5 |
| Growth | sekitar Rp7,7 juta | sekitar Rp74 juta | 3 | unlimited |
| Scale | custom, mulai sekitar Rp14 juta | custom | unlimited | unlimited |

Angka adalah starting point dalam IDR dan wajib direview sebelum GA. Core system
tidak boleh mengasumsikan IDR sebagai currency universal; customer yang belum
memiliki market/currency price yang diverifikasi tidak boleh checkout.

### 14.2 Payment

Payment memakai provider adapter dan routing berdasarkan market, currency, dan
plan. Untuk MVP: Xendit primary untuk checkout Starter/Growth, Midtrans secondary,
dan Duitku/iPaymu untuk non-recurring. Provider global (misalnya kartu/invoice
internasional) hanya boleh diaktifkan setelah adapter contract, merchant/legal
review, tax, currency, webhook, refund, dan reconciliation gate lulus.
Callback wajib memverifikasi signature/token resmi provider. Jangan menyimpan data
kartu mentah. Status subscription dan transaction harus idempotent terhadap
webhook duplikat, out-of-order event, retry, refund, chargeback, dan cancellation.

### 14.3 Regulation monitor

Cron harian 06:00 UTC memeriksa sumber resmi yang terdaftar dalam
`RegulationSourceRegistry`, dengan policy pack per framework dan market. MVP
memulai dari AICPA, ISO, PCI SSC, EDPB, CPPA, dan sumber Indonesia/Asia yang
disetujui. Perubahan hash membuat `RegulationUpdate` berstatus `pending_review`.
Gemini membuat `aiDraftSummary`; staff GRC mengedit, memvalidasi, mengisi
`reviewedBy`, dan map ke controls. Setelah valid:

- control terdampak organisasi menjadi `needs_review`;
- notifikasi dikirim ke customer;
- summary resmi tidak pernah diisi otomatis oleh AI.

### 14.4 Notifications

Channel adalah adapter: in-app dan email untuk baseline; WhatsApp Business API
untuk MVP setelah vendor dan budget disetujui; channel regional/global lain hanya
diaktifkan setelah delivery, privacy, consent, retry, dan cost gate lulus. Semua
notification memiliki delivery status, retry policy, dedupe key, market/channel
policy, dan audit timestamp. Critical drift target <15 menit.

---

## 15. Security, Privacy, dan Reliability

- TLS in transit, encryption at rest untuk DB/object storage.
- Secret jangka panjang disimpan di secret manager; minimum shared hosting memakai
  environment panel yang tidak di-commit, dengan rencana migrasi.
- STS/GitHub installation token hanya di memory.
- Tidak ada `.env` atau credential di repository.
- Evidence access memakai authorization server-side dan signed access terbatas.
- Audit log append-only untuk evidence dan aksi administratif.
- Rate limit per user, organization, IP, dan public chatbot.
- Retry transient dengan backoff; tidak retry buta untuk auth/permission.
- Provider API call log mencatat endpoint, status, durasi, dan correlation ID.
- Backup database harian, retensi 30 hari, diuji restore.
- Data residency organization mengikuti `homeRegion` dan `dataResidencyPolicy`.
  MVP memakai region Indonesia/Asia yang telah diverifikasi. EU/US/APAC hanya
  boleh dipilih bila DB, object storage, queue, logs, AI retrieval, backup,
  support access, dan deletion flow di region tersebut sudah diverifikasi.
- Region boundary, cross-region transfer, subprocessors, legal basis, retention,
  export, deletion, dan customer-visible limitation wajib memiliki policy version
  serta audit record. Tidak ada cross-region fallback diam-diam untuk restricted
  data.
- Pentest independen sebelum GA.
- Sentry tidak boleh menangkap token, raw credential, atau evidence sensitif.
- Data diklasifikasikan minimal `public`, `internal`, `confidential`, dan
  `restricted`; evidence provider dan credential reference default-nya
  `restricted`. Akses export/download memerlukan purpose dan audit record.
- Retention, deletion, legal hold, export, dan subject request memiliki policy
  versioned. Penghapusan customer tidak boleh menghapus audit/evidence yang sedang
  berada dalam legal hold tanpa approval dan immutable record.
- Session invalidation, MFA untuk OWNER/ADMIN/internal, recovery flow, invite
  expiry, passwordless/auth provider events, dan privilege changes diaudit.
- Dependency patch SLA: critical security patch secepatnya dan maksimal 72 jam,
  high maksimal 14 hari, kecuali RFC risiko/mitigasi disetujui.

NFR baseline:

| Area | Target |
|---|---|
| Availability API customer (GA) | 99.5% per bulan; maintenance terjadwal dikecualikan |
| API CRUD p95 | <500 ms pada beban baseline yang didokumentasikan |
| Scan queue start p95 | <5 menit saat backlog normal |
| Normal sync | maksimal 6 jam sejak perubahan sumber, jika provider dan scheduler tersedia |
| Critical alert | p95 delivery <15 menit sejak deteksi; 100% dicatat sukses/gagal |
| UI locale | 100% dari setiap locale yang dipublikasikan; MVP EN/ID, ZH hanya bila market pack locale lulus |
| AI cost | seluruhnya terukur via `AiUsageLog` |
| RPO database | ≤24 jam pada Fase 1; ≤1 jam sebelum GA jika provider mendukung |
| RTO customer read path | ≤4 jam Fase 1; ≤2 jam sebelum GA |

Availability/SLO tidak mencakup provider outage, tetapi outage tersebut harus
terlihat sebagai `provider_error`, memiliki incident, dan tidak menghasilkan
status pass. SLO diukur dari telemetry yang dapat diaudit, bukan estimasi.

---

## 16. Roadmap dan Gate Delivery

### Fase 0 — Foundation, minggu 1–3

**Deliverables**

- Struktur app, UI package, config, CI.
- Hosting verification.
- PostgreSQL + Prisma migration baseline.
- Global-ready organization profile: home region, data residency policy, locale,
  timezone, currency, tax profile, provider/capability registry, dan versioning.
- Redis connectivity test.
- Clerk, Xendit sandbox, Sentry.
- Cron dummy endpoint.
- Locale folders, common strings, locale completeness check, dan UTC/timezone test.
- Domain/SSL plan.

**Gate**

- Hello World marketing/app dapat dideploy.
- Login/signup bekerja.
- Migration berhasil.
- Redis test berhasil.
- Cron memanggil endpoint dan tercatat.
- Tidak ada currency/region/provider hardcode pada domain core.
- Tenant region policy dan locale/timezone tersimpan serta diuji.
- Semua blocker hosting terdokumentasi.

### Fase 1 — AWS evidence vertical slice, minggu 4–8

**Deliverables**

- SOC 2 control seed terbatas pada mapping yang benar-benar didukung AWS.
- AWS cross-account IAM role dengan External ID dan permission matrix.
- Delapan AWS checks prioritas, masing-masing memiliki proof record.
- ScanRun/Finding/ObservedFact/Evidence/aggregator dengan freshness dan coverage.
- Dashboard, integration, control detail, evidence metadata, remediation.
- Onboarding: signup → connect → verify → scan → inspect evidence.

**Release gate**

- Tidak ada status/skor dashboard tanpa `Finding` nyata yang terhubung ke evidence
  hash-valid.
- Hasil AWS dapat dibandingkan manual dan identik pada test account yang dikontrol.
- Permission failure menjadi error/needs_review, bukan pass.
- Re-scan setelah remediation dapat mengubah status secara benar.
- Evidence hash dapat diverifikasi ulang dan object retention aktif.
- Dua organisasi tidak dapat saling membaca data melalui seluruh endpoint.

### Fase 1B — Provider kedua dan operasional, minggu 9–14

- GitHub App read-only, hanya check yang permission dan endpoint-nya sudah
  diverifikasi; Google Workspace menyusul hanya setelah API sharing/policy
  diverifikasi secara resmi.
- PDF report dengan appendix provenance.
- Queue retry, circuit breaker, notification delivery, backup/restore drill,
  internal operations dashboard, dan SLO alert.
- Billing sandbox terpisah dari scan truth; payment tidak menjadi syarat untuk
  memalsukan atau membuka status scan.
- MVP market pack Indonesia/Asia: price catalog IDR, payment routing, invoice/tax
  policy, notification channel, locale, data-residency limitation, dan
  customer-facing capability copy.

**Gate:** contract test, sandbox comparison, negative permission test, provider
schema drift test, restore drill, dan incident runbook lulus. Fitur yang belum
lulus tetap `not_implemented`/`verification_required`.

### Fase 2 — Trust, auditor, public site, setelah Fase 1B

- Marketing site penuh dan public chatbot.
- Trust Page per organization.
- Auditor Portal magic link read-only berbatas waktu.
- Regulation monitor untuk SOC 2.
- WhatsApp critical alerts.
- JOBEN dogfooding SOC 2 Type I.

### Fase 3 — Global expansion readiness, setelah MVP regional

- Market-pack framework untuk target region berikutnya dengan owner, legal basis,
  provider matrix, language, pricing/currency/tax, support, dan data-residency
  proof.
- Global payment adapter dan reconciliation untuk market yang disetujui.
- EU/US/APAC region verification, termasuk backup, logs, AI, subprocessors, and
  deletion/transfer controls.
- SSO/SAML dan enterprise identity/provider contracts sesuai target market.
- Global support/SLO, status communication, and incident coverage.

**Gate:** satu market/region baru tidak boleh live hanya karena UI translated.
Region tersebut wajib memiliki provider sandbox comparison, privacy/legal review,
residency evidence, billing reconciliation, locale completeness, support runbook,
cross-region isolation test, dan independent reviewer sign-off.

### Fase 4 — ISO 27001, setelah region expansion gate

- 93 Annex A controls.
- Reuse mapping SOC 2/ISO.
- Gap analysis.
- Connector Azure/GCP/Okta/Vercel/Supabase/Firebase.

### Fase 5 — GDPR/CCPA, setelah privacy/regional readiness

- PII data map, cookie scanner, DSAR, DPA, vendor risk, breach timer 72 jam.

### Fase 6 — Questionnaire AI, minggu 30–35

- Riset biaya provider terlebih dahulu.
- Upload PDF/Excel/CSV, parsing, evidence matching, confidence score.
- Human review dan `humanApproved` sebelum send/export.

### Fase 7 — PCI SAQ assistant, minggu 36–39

- Wizard pemilihan SAQ dan readiness checklist.
- Tidak membangun ASV scanning.

### Fase 8 — White-label dan auditor marketplace, setelah global expansion gate

- Agency multi-tenant, custom branding, sub-organization.
- Marketplace hanya sebagai penghubung, bukan penjamin kualitas auditor.

### Fase 9 — Hardening dan GA, setelah seluruh target release gate

- Pentest, remediasi critical/high.
- Load test sesuai kapasitas shared hosting nyata.
- Review threshold migrasi VPS/cloud.
- Legal review ToS/Privacy.
- Responsible disclosure/bug bounty.

---

## 17. Test Strategy dan Acceptance Criteria

### 17.1 Test pyramid

- Unit test untuk setiap evaluator check, aggregator, scorer, redactor, permission
  mapper, dan remediation template.
- Integration test untuk Prisma tenant scoping, evidence writer, provider client,
  queue idempotency, webhook signature.
- Contract test terhadap mock/provider sandbox dengan response fixture resmi.
- End-to-end test onboarding dan remediation.
- Security test untuk IDOR, cross-tenant access, secret leakage, replay webhook,
  privilege escalation.
- Manual verification terhadap cloud sandbox nyata.

### 17.2 Test wajib scan

1. AWS sandbox salah konfigurasi: public bucket, open security group, root MFA off;
   finding harus muncul tanpa false negative.
2. AWS sandbox benar: check pass tanpa false positive.
3. Cabut satu permission: hanya check terkait error; scan lain tetap berjalan.
4. Dua organization: organization A tidak dapat membaca finding/evidence B.
5. Cron dipanggil dua kali cepat: tidak menggandakan ScanRun atau charge API.
6. Critical drift: pass → fail menghasilkan notification dan delivery timestamp.
7. Connector token/credential tidak pernah muncul di DB, log, atau error response.
8. Re-scan setelah perbaikan mengubah finding dan agregasi secara konsisten.
9. Demo mode selalu menampilkan badge dan tidak bisa mengakses live endpoint.
10. PDF dapat dibuka pada dua reader dan evidence hash dapat diverifikasi.

### 17.3 Definition of Done per check

Sebuah check hanya boleh diberi `implemented` jika:

- API dan permission diverifikasi dari dokumentasi resmi/provider sandbox.
- Evaluator deterministik memiliki unit test pass/fail/error.
- `requiredPermissions` sesuai action yang benar-benar dipakai.
- Raw response redacted dan tersimpan immutable.
- Resource identifier tersedia pada finding non-pass.
- Error permission dibedakan dari fail.
- Control mapping dan remediation template tersedia.
- Manual test memiliki expected result.

---

## 18. Observability dan Operasional

Setiap scan memiliki correlation ID dan mencatat:

- organization/integration/scanRun;
- provider dan check ID;
- start/end/duration;
- API endpoint, status, retry count;
- jumlah finding per result;
- evidence write result/hash;
- notification trigger/delivery;
- error class dan redacted message.

Dashboard internal minimum:

- scan success/partial/error rate;
- rata-rata durasi per provider/check;
- permission error teratas;
- circuit breaker integrations;
- evidence write failure;
- notification latency;
- AI token/cost per organization/feature/provider;
- payment webhook failure/replay.

Alert operasional dibuat untuk:

- error rate meningkat;
- scan backlog;
- critical notification melewati 15 menit;
- object storage gagal;
- backup/restore gagal;
- provider rate limit atau API schema berubah.

---

## 19. Keputusan Terbuka dan RFC yang Dibutuhkan

Hal berikut tidak boleh diputuskan diam-diam:

1. Nama dan bentuk badan hukum untuk merchant Xendit/Midtrans.
2. CPA firm partner sebelum Fase 2 selesai.
3. Provider dan budget WhatsApp Business API.
4. Versi/kapabilitas hosting aktual, termasuk slot Node dan PostgreSQL.
5. Threshold migrasi dari shared hosting ke VPS/cloud.
6. Data residency dan object-storage region untuk setiap target pasar.
7. Endpoint Google Workspace yang menggantikan API deprecated.
8. Final pricing setelah validasi biaya provider dan willingness-to-pay.
9. Retention period evidence per plan dan kebutuhan legal customer.
10. Apakah organization boleh memiliki beberapa integration pada provider yang sama.

Setiap keputusan harus memiliki: konteks, opsi, keputusan, dampak, owner, tanggal,
dan rencana rollback/migrasi bila relevan.

---

## 20. Traceability Matrix

| Risiko/kebutuhan | Requirement utama | Bukti lulus |
|---|---|---|
| Dashboard terlihat sukses padahal data palsu | P-01, P-02, P-03 | Audit query + no-data E2E |
| Credential cloud bocor | P-08, §7.4, §15 | Secret scan + DB/log inspection |
| Auditor tidak bisa memverifikasi | P-04, §9.3 | Hash/evidence appendix/PDF review |
| Permission kurang disamarkan | P-05, §7.2 | Negative permission test |
| Data tenant bocor | P-06, §10 | Cross-tenant security test |
| AI membuat klaim legal | P-07, §13 | Prompt/output guardrail test |
| Scan double-run/charge | §7.2, §17.2 | Idempotency test |
| Drift terlambat diketahui | §7.3, §14.3, §18 | Notification latency metric |
| Hosting tidak mendukung desain | §5.3, Fase 0 | Hosting verification gate |

---

## 21. Checklist Implementer

Sebelum menulis fitur:

- [ ] Requirement dan status data sudah didefinisikan.
- [ ] Sumber API resmi dan permission sudah diverifikasi.
- [ ] Tenant boundary sudah dirancang di service layer.
- [ ] Error state dan empty state sudah dirancang.
- [ ] Tidak ada fallback sample/live yang ambigu.
- [ ] Data yang ditampilkan memiliki query source dan timestamp.
- [ ] Audit/security impact sudah dipertimbangkan.
- [ ] Test acceptance dan rollback sudah ditulis.

Sebelum merge/deploy:

- [ ] Lint, typecheck, unit, integration, dan E2E relevan lulus.
- [ ] Tidak ada secret atau credential di diff/log.
- [ ] Migration backward-compatible atau memiliki migration plan.
- [ ] Workflow dan environment variable terdokumentasi.
- [ ] Provider contract test masih sesuai dokumentasi resmi.
- [ ] Report dan status control dapat ditelusuri sampai evidence.
- [ ] Semua limitation dicatat, bukan diganti data simulasi.

---

## 22. Artefak wajib sebelum implementasi

PRD ini tidak dianggap siap dikerjakan hanya karena folder aplikasi berhasil
dibuat. Sebelum capability diberi status `live_verified`, repository wajib memiliki
artefak yang dapat direview:

| Artefak | Isi minimum | Pemilik |
|---|---|---|
| `DOC/decision-log.md` | keputusan, opsi, alasan, dampak, owner, tanggal, rollback | Tech lead |
| `DOC/provider-matrix.md` | endpoint resmi, API/doc revision, permission, rate limit, fixture, verification date | Connector owner |
| `DOC/threat-model.md` | assets, trust boundaries, abuse cases, mitigations, residual risk | Security owner |
| `DOC/data-classification.md` | field classification, retention, residency, deletion/legal hold | Privacy owner |
| `DOC/runbooks/*.md` | incident, provider outage, stuck scan, evidence integrity, restore, credential revoke | Operations |
| `DOC/test-evidence/` | test account config sans secret, expected/actual, logs redacted, screenshots/reports | QA owner |
| `DOC/capability-registry.md` | status dan proof reference setiap capability | Product/engineering |

Secret, token, private key, raw credential, dan signed URL dilarang masuk ke
artefak, fixture, screenshot, log, commit, atau issue. Test account harus
disposable atau memiliki documented cleanup.

---

## 23. Production readiness dan incident operations

### 23.1 Runbook minimum

Sebelum GA, operator harus dapat menjalankan tanpa improvisasi:

1. pause satu integration atau seluruh scan queue tanpa menghapus histori;
2. revoke/reconnect credential dan memastikan token lama tidak dapat dipakai;
3. menangani provider outage/rate limit/schema drift dengan status yang terlihat;
4. memulihkan database ke environment terisolasi dan memverifikasi checksum,
   tenant count, migration version, serta evidence references;
5. menangani `integrity_failed` dengan quarantine, incident, dan re-collection;
6. mengulang notification yang gagal tanpa duplikasi dan tanpa mengubah finding;
7. mematikan AI provider/fallback dengan aman tanpa mengganggu deterministic scan;
8. menghapus/export data sesuai policy, termasuk audit trail dan legal hold.

Setiap runbook memiliki trigger, severity, owner/on-call, containment, decision
tree, command/action yang aman, komunikasi customer, evidence collection, recovery,
dan post-incident review. Incident P0/P1 wajib memiliki timeline dan corrective
action; tidak boleh ditutup hanya karena dashboard kembali hijau.

### 23.2 Release discipline

- Development, staging, sandbox provider, dan production memakai project/credential
  terpisah. Data production tidak boleh dipakai sebagai fixture.
- Migration dijalankan expand → backfill terukur → switch → contract; rollback
  tested untuk setiap perubahan yang berisiko.
- Release artifact memiliki commit SHA, schema/migration version, check registry
  version, algorithm version, dependency lockfile, dan capability diff.
- Feature flag default deny untuk capability belum `live_verified`. Rollback flag
  tidak boleh merusak append-only evidence atau membuat status baru tampak valid.
- Production deploy memerlukan reviewer berbeda dari author untuk scan/evidence,
  auth/tenant, billing, dan AI safety changes.

---

## 24. Go/no-go gates dan Definition of Ready/Done

### 24.1 Definition of Ready

Sebuah feature/check boleh mulai diimplementasikan hanya jika:

- tujuan, non-goals, actor/role, tenant scope, dan data classification tertulis;
- provider contract, endpoint, permission, rate limit, API version/doc revision,
  dan unknown/deprecation behavior memiliki source resmi;
- state machine, error taxonomy, freshness/coverage rule, idempotency, dan rollback
  ditentukan;
- expected evidence schema, redaction rules, retention, audit events, dan test
  fixtures tersedia;
- acceptance test untuk happy path, partial result, permission failure, provider
  outage, schema drift, replay, dan cross-tenant access tertulis;
- owner, SLO impact, cost ceiling, dan operational runbook ditentukan.

### 24.2 Definition of Done untuk live capability

Capability baru hanya boleh menjadi `live_verified` jika semua ini lulus:

1. Unit, integration, contract, E2E, security, and regression tests lulus pada CI.
2. Test provider nyata/sandbox menghasilkan output yang sama dengan expected manual
   untuk pass, fail, empty scope, partial, permission denied, timeout, dan schema
   change.
3. Setiap output non-pass memiliki resource key, deterministic message, error
   code bila relevan, evidence ID/hash, observed time, dan rule version.
4. Aggregator membuktikan bahwa error, stale, incomplete, demo, dan missing
   evidence tidak menjadi pass atau menaikkan score secara diam-diam.
5. Dua organisasi diuji pada seluruh route/repository/queue/report/evidence path;
   cross-tenant read/write/export selalu ditolak dan diaudit.
6. Secret scan, log inspection, redaction canary, dependency audit, SAST, and
   privilege review lulus tanpa critical/high yang tidak diterima secara tertulis.
7. Migration, retry, lock, replay, circuit breaker, restore, and runbook drill
   lulus; RPO/RTO dan SLO memiliki telemetry nyata.
8. Dokumentasi provider matrix, capability registry, limitations, changelog, and
   customer-facing copy sudah diperbarui.
9. Reviewer menyetujui proof record; capability mendapat expiry/reverification date.

### 24.3 Release gates

**Gate A — Foundation:** auth/session, tenant authorization, schema/migration,
secret handling, logging redaction, CI, backup, restore, and hosting verification
lulus. Tidak ada customer data sebelum Gate A.

**Gate B — Evidence vertical slice:** AWS integration verify, satu check
end-to-end, immutable evidence, hash re-verification, deterministic finding,
control projection, freshness/coverage, dan negative permission test lulus.

**Gate C — Customer beta:** seluruh check yang diklaim live memiliki proof record;
onboarding dan remediation dapat diulang; provider outage terlihat; report/citation
dapat diaudit; support/runbooks aktif; tidak ada critical security finding.

**Gate D — GA:** Fase 1B operational gates, pentest, load test pada kapasitas
nyata, restore drill berkala, incident exercise, legal/privacy review, pricing/
billing reconciliation, SLO dashboard, dan owner on-call disetujui.

**Gate E — Region/market expansion:** sebelum market pack baru dipublikasikan,
provider dan framework yang diklaim live memiliki contract/sandbox evidence,
residency dan transfer policy diverifikasi, locale lengkap, pricing/currency/tax
reconciliation lulus, subprocessors/AI/log/backup/deletion tercakup, support dan
incident runbook aktif, cross-region isolation diuji, serta reviewer independen
menyetujui proof record dan expiry/reverification date.

Kegagalan gate menghentikan release. Tim wajib menurunkan capability ke
`not_implemented`, `verification_required`, atau `degraded`; tidak boleh mengganti
hasil dengan sample data atau menampilkan optimistic score.

---

## 25. Riwayat Dokumen

| Versi | Perubahan |
|---|---|
| 4.0 | Konsolidasi PRD master v3.0 dan Addendum Scan v1.0 menjadi baseline engineering tunggal; konflik status/evidence/path diselesaikan eksplisit. |
| 5.0 | Kritik dan hardening evidence-first: provenance, freshness/coverage, immutable evidence, AI safety contract, tenant/API security, vertical-slice roadmap, operational runbooks, dan release gates. |
| 5.1 | Penambahan peta readiness 15 modul utama, kontrak kelengkapan lintas modul, lifecycle/state machine, acceptance behavior, dan dependency gate agar `full feature` tidak disamakan dengan CRUD atau placeholder. |
| 5.2 | Penguatan core-system delivery: completion matrix M-01–M-15, dossier proof wajib, dependency slices, multi-organization canonical model, state machine/event contract, endpoint coverage, concurrency, dan ledger integrity. |
| 5.3 | Core system global-ready dengan MVP regional-first: tenant region policy, locale/timezone/currency/tax abstraction, provider adapter/registry, residency boundary, market-pack, dan Gate E ekspansi region. |

Perubahan besar terhadap keputusan final memerlukan RFC baru dan pembaruan versi
dokumen ini. `/DOC/PRD-JOBEN-ENTERPRISE.md` tetap menjadi sumber kebenaran terbaru
setelah perubahan disetujui.