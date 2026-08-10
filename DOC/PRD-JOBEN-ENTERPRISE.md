# JOBEN ENTERPRISE
## Product Requirements & System Delivery Plan

**Produk:** JOBEN ENTERPRISE  
**Kategori:** Compliance Automation & Trust Management Platform  
**Domain target:** `jobenapp.cloud`  
**Dokumen kanonik:** `/DOC/PRD-JOBEN-ENTERPRISE.md`  
**Versi:** 5.0 — Evidence-First Production Baseline
**Status:** Baseline engineering; implementasi hanya boleh mengklaim capability yang telah melewati gate pada §24
**Bahasa dokumen:** Bahasa Indonesia; istilah standar teknis dan compliance mengikuti istilah resmi  
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
| MVP mencakup terlalu banyak permukaan sekaligus | Scope besar menghambat validasi kebenaran | Satu vertical slice AWS read-only harus benar sebelum provider kedua |

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

JOBEN ENTERPRISE membantu startup, scale-up, dan enterprise di Indonesia/Asia
mempercepat kesiapan SOC 2, ISO 27001, dan privacy readiness melalui:

1. pengumpulan evidence dari sistem nyata;
2. pemantauan kontrol secara berkelanjutan;
3. Evidence Vault yang immutable dan dapat diaudit;
4. remediation guidance yang konkret;
5. Trust Page dan Auditor Portal;
6. Policy Center dan workflow review manusia;
7. AI assistant yang menjelaskan data organisasi, bukan menciptakan fakta compliance;
8. payment dan notifikasi yang sesuai kebutuhan pasar Indonesia.

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
- Menjadi produk B2B SaaS dengan paket Starter, Growth, dan Scale berbasis IDR.
- Membantu JOBEN sendiri melakukan dogfooding menuju SOC 2 Type I lalu Type II.

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
- Framework SOC 2 Type II dengan control seed yang hanya dipetakan ke check
  berstatus `live_verified`.
- Fase 1: satu vertical slice AWS cross-account IAM role, scan engine, evidence
  vault, control aggregator, dashboard, control detail, dan remediation.
- Fase 1B: provider GitHub App, PDF, notification, billing sandbox, dan AI
  gateway hanya setelah safety/contract gate lulus.
- Google Workspace, public trust page, auditor portal, dan fitur regulasi tidak
  dianggap MVP live sebelum gate provider dan privacy masing-masing lulus.
- i18n EN/ID/ZH dan cron terproteksi dikirim bertahap; string yang belum lengkap
  tidak boleh fallback diam-diam pada customer-facing release.

### 4.2 Out of scope MVP, tetapi disiapkan sebagai roadmap

- Azure, GCP, Okta, Vercel, Supabase, Firebase connector.
- ISO 27001 penuh, GDPR/CCPA data mapping, DSAR, DPA generator.
- Questionnaire AI penuh untuk PDF/Excel/CSV.
- PCI SAQ assistant.
- White-label agency dan marketplace auditor.
- Multi-currency non-IDR.
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

---

## 5. Keputusan Arsitektur

### 5.1 Arsitektur target

Satu Next.js App Router untuk marketing site, customer app, dan Route Handlers API.
Logic domain dipisahkan dalam service layer agar dapat diekstrak menjadi service
terpisah setelah migrasi hosting.

```text
jobenapp.cloud
  Marketing pages, public Trust Page, public chatbot

app.jobenapp.cloud
  Next.js UI + Route Handlers + domain services
  Auth, billing, integrations, scan, evidence, controls,
  regulations, AI gateway, notifications, i18n

External services
  PostgreSQL        persistent application data
  Redis managed     BullMQ on-demand queue
  S3/R2             immutable evidence and report storage
  cPanel Cron       scheduled job trigger
  Clerk             auth, MFA, SSO/SAML capability
  Xendit/Midtrans   payment
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
| Payment | Xendit primary, Midtrans secondary, Duitku/iPaymu non-recurring |
| AI | Gemini, Groq, DeepSeek melalui AI Gateway |
| i18n | next-intl, route `/en`, `/id`, `/zh` |
| Monitoring | Sentry dan status page eksternal |

Perubahan dari keputusan ini memerlukan RFC tertulis. Opsi NestJS terpisah tidak
digunakan sebelum hosting diverifikasi mendukung minimal dua Node.js app.

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
  id, name, slug, domain, region, preferredLocale, isDemoMode

User
  organizationId, clerkUserId, email, role, preferredLocale

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
  consecutiveFailures

CheckDefinitionMeta
  id, provider, severityDefault, status, requiredPermissions

CheckControlMapping
  checkDefinitionId, controlId

ScanRun
  integrationId, status, startedAt, finishedAt, triggeredBy, idempotencyKey

Evidence
  organizationId, sourceIntegrationId, scanRunId, findingId, controlStatusId,
  type, storageUrl, contentHash, canonicalizationVersion, schemaVersion,
  providerRequestId, sourceEndpoint, collectedAt, retentionUntil, immutable,
  integrityStatus, supersedesEvidenceId

ObservedFact
  evidenceId, provider, resourceKey, observedAt, payloadSchema, extractedFields

Finding
  organizationId, scanRunId, checkDefinitionId, checkVersion, observedFactId,
  resourceKey, result, errorCode, message, detectedAt, evaluatedAt

RemediationTemplate
  checkDefinitionId, summaryI18nKey, whyItMattersI18nKey,
  stepsI18nKey, estimatedEffort

DailyComplianceSnapshot
  organizationId, frameworkId, snapshotDate, score, metCount,
  partialCount, notMetCount, needsReviewCount

AuditLog / ProviderApiLog
  organizationId, actor, action, resource, endpoint, statusCode,
  durationMs, createdAt, metadata
```

Modul lain yang diperlukan:

```text
Policy, Risk, Vendor, RegulationUpdate,
Questionnaire, QuestionnaireItem,
Plan, Subscription, PaymentTransaction,
AiUsageLog, ChatSession, ChatMessage, Referral
```

Constraint wajib:

- Semua entity customer memiliki `organizationId` langsung atau relasi yang
  dapat divalidasi.
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
| `POST` | `/api/chat/sessions` | mulai sesi chatbot tenant |
| `POST` | `/api/chat/sessions/{id}/messages` | pertanyaan RAG |
| `POST` | `/api/webhooks/xendit` | callback payment terverifikasi |
| `POST` | `/api/internal/jobs/scan-runner` | drain scan queue |
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

Harga berasal dari tabel `Plan`, bukan hardcode frontend. Seed awal:

| Plan | Bulanan | Tahunan | Framework | Integrasi |
|---|---:|---:|---:|---:|
| Starter | sekitar Rp2,3 juta | sekitar Rp22 juta | 1 | 5 |
| Growth | sekitar Rp7,7 juta | sekitar Rp74 juta | 3 | unlimited |
| Scale | custom, mulai sekitar Rp14 juta | custom | unlimited | unlimited |

Angka adalah starting point dan wajib direview sebelum GA.

### 14.2 Payment

Xendit primary untuk checkout Starter/Growth, Midtrans secondary, Duitku/iPaymu
untuk non-recurring. Callback wajib memverifikasi signature/token resmi provider.
Jangan menyimpan data kartu mentah. Status subscription dan transaction harus
idempotent terhadap webhook duplikat.

### 14.3 Regulation monitor

Cron harian 06:00 UTC memeriksa sumber resmi AICPA, ISO, PCI SSC, EDPB, dan CPPA.
Perubahan hash membuat `RegulationUpdate` berstatus `pending_review`. Gemini
membuat `aiDraftSummary`; staff GRC mengedit, memvalidasi, mengisi `reviewedBy`,
dan map ke controls. Setelah valid:

- control terdampak organisasi menjadi `needs_review`;
- notifikasi dikirim ke customer;
- summary resmi tidak pernah diisi otomatis oleh AI.

### 14.4 Notifications

Channel: in-app, email, dan WhatsApp Business API setelah vendor dan budget
disetujui. Semua notification memiliki delivery status, retry policy, dan audit
timestamp. Critical drift target <15 menit.

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
- Data residency organization mendukung region EU/US/ID/APAC sesuai kapabilitas
  vendor yang diverifikasi.
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
| UI locale | 100% EN/ID/ZH sebelum GA |
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
- Redis connectivity test.
- Clerk, Xendit sandbox, Sentry.
- Cron dummy endpoint.
- Locale folders dan common strings.
- Domain/SSL plan.

**Gate**

- Hello World marketing/app dapat dideploy.
- Login/signup bekerja.
- Migration berhasil.
- Redis test berhasil.
- Cron memanggil endpoint dan tercatat.
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

### Fase 3 — ISO 27001, minggu 18–23

- 93 Annex A controls.
- Reuse mapping SOC 2/ISO.
- Gap analysis.
- Connector Azure/GCP/Okta/Vercel/Supabase/Firebase.

### Fase 4 — GDPR/CCPA, minggu 24–29

- PII data map, cookie scanner, DSAR, DPA, vendor risk, breach timer 72 jam.

### Fase 5 — Questionnaire AI, minggu 30–35

- Riset biaya provider terlebih dahulu.
- Upload PDF/Excel/CSV, parsing, evidence matching, confidence score.
- Human review dan `humanApproved` sebelum send/export.

### Fase 6 — PCI SAQ assistant, minggu 36–39

- Wizard pemilihan SAQ dan readiness checklist.
- Tidak membangun ASV scanning.

### Fase 7 — White-label dan auditor marketplace, minggu 40–43

- Agency multi-tenant, custom branding, sub-organization.
- Marketplace hanya sebagai penghubung, bukan penjamin kualitas auditor.

### Fase 8 — Hardening dan GA, minggu 44–47

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
dibuat. Sebelum feature diberi status `implemented`, repository wajib memiliki
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

Kegagalan gate menghentikan release. Tim wajib menurunkan capability ke
`not_implemented`, `verification_required`, atau `degraded`; tidak boleh mengganti
hasil dengan sample data atau menampilkan optimistic score.

---

## 25. Riwayat Dokumen

| Versi | Perubahan |
|---|---|
| 4.0 | Konsolidasi PRD master v3.0 dan Addendum Scan v1.0 menjadi baseline engineering tunggal; konflik status/evidence/path diselesaikan eksplisit. |
| 5.0 | Kritik dan hardening evidence-first: provenance, freshness/coverage, immutable evidence, AI safety contract, tenant/API security, vertical-slice roadmap, operational runbooks, dan release gates. |

Perubahan besar terhadap keputusan final memerlukan RFC baru dan pembaruan versi
dokumen ini. `/DOC/PRD-JOBEN-ENTERPRISE.md` tetap menjadi sumber kebenaran terbaru
setelah perubahan disetujui.