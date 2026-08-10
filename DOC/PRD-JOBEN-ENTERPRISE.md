# JOBEN ENTERPRISE
## Product Requirements & System Delivery Plan

**Produk:** JOBEN ENTERPRISE  
**Kategori:** Compliance Automation & Trust Management Platform  
**Domain target:** `jobenapp.cloud`  
**Dokumen kanonik:** `/DOC/PRD-JOBEN-ENTERPRISE.md`  
**Versi:** 4.0 — Consolidated Engineering Baseline  
**Status:** Draft siap dijadikan baseline pembangunan  
**Bahasa dokumen:** Bahasa Indonesia; istilah standar teknis dan compliance mengikuti istilah resmi  
**Sumber konsolidasi:** PRD master v3.0 dan Addendum Scan & Continuous Control Monitoring v1.0

> Dokumen ini adalah satu-satunya acuan produk dan engineering. Jika detail provider
> berubah atau belum didefinisikan di sini, tim wajib memeriksa dokumentasi resmi
> provider pada saat implementasi dan mencatat keputusan tersebut dalam RFC atau
> decision log. Jangan mengarang endpoint, response shape, permission, atau klaim
> compliance dari ingatan.

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

### 4.1 In scope untuk MVP (Fase 0–1)

- Fondasi Next.js App Router + TypeScript + PostgreSQL + Prisma.
- Auth Clerk dan multi-tenant organization.
- Framework SOC 2 Type II dengan seed control.
- Connector AWS cross-account IAM role.
- Connector GitHub App.
- Connector Google Workspace Admin SDK.
- Scan engine, finding, evidence vault, control aggregator, scoring.
- Dashboard, Integration page, Control detail, remediation.
- PDF report auditor-ready.
- Policy Center dengan lima template awal.
- Xendit checkout untuk Starter/Growth.
- AI Gateway dan chatbot in-app RAG dasar.
- i18n EN/ID/ZH untuk customer-facing MVP.
- Cron endpoint terproteksi dan retry/idempotency.

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
  provider: Provider;
  title: string;              // i18n key
  description: string;        // i18n key
  severity: Severity;
  mappedControls: string[];
  requiredPermissions: string[];
  run(ctx: ScanContext): Promise<CheckExecutionResult>;
}

interface CheckExecutionResult {
  findings: Array<{
    result: FindingResult;
    resourceIdentifier?: string;
    rawEvidence: object;      // API payload after secret redaction
    message: string;           // deterministic, non-AI
  }>;
  executionError?: string;    // unable to run, distinct from control failure
}
```

`registry.ts` adalah single source of truth check aktif per provider.
`runner.ts` membuat `ScanRun`, menjalankan check, menulis evidence/finding, dan
memperbarui control status. `aggregator.ts` menghitung status dan score.

---

## 7. Modul Scan & Continuous Control Monitoring

### 7.1 Entitas dan alur

```text
Integration
  → ScanRun
    → CheckDefinition
      → satu atau banyak ScanFinding
        → CheckControlMapping
          → OrgControlStatus
            → dashboard, alert, report, chatbot
```

- `CheckDefinition`: kode versioned, bukan data yang dapat diedit customer.
- `ScanRun`: satu eksekusi pada satu integration.
- `ScanFinding`: hasil per resource/check, termasuk payload evidence dan pesan
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
memuat action yang dipakai check yang telah implemented. Action baseline:

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

Aturan:

1. Check relevan belum pernah berhasil dijalankan → `not_started`/`needs_review`.
2. Semua finding relevan pass → `met`.
3. Ada finding fail → `not_met`.
4. Satu check menghasilkan resource campuran pass/fail → `partial` dengan
   proporsi eksplisit, misalnya `9/10`.
5. Error permission/provider tidak boleh dikonversi menjadi pass.
6. Status harus mengambil finding terbaru dari scan sukses pada setiap integration
   relevan dan seluruh proses wajib tenant-scoped.

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

Dashboard selalu menampilkan timestamp “Data per” untuk setiap integration dan
framework.

---

## 9. Evidence, Remediation, dan Report

### 9.1 Evidence lifecycle

1. Provider client menerima response.
2. Secret dan token di-redact.
3. Payload asli disimpan immutable di S3/R2.
4. SHA-256 dihitung dari payload canonical.
5. DB menyimpan `storageUrl`, hash, timestamp, dan relasi.
6. Evidence tidak dapat diedit atau ditimpa; koreksi membuat record baru.
7. Setiap akses Evidence Vault dicatat dalam audit log.

DB hanya menyimpan reference ke payload untuk menghindari evidence besar di
PostgreSQL. `rawEvidenceJson` dalam kontrak scan berarti payload yang tersimpan
verbatim di object storage setelah redaction, bukan JSON yang dibuat/diringkas AI.

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

ScanFinding
  scanRunId, checkDefinitionId, result, severity, resourceIdentifier,
  message, rawEvidenceRef, detectedAt

Evidence
  organizationId, sourceIntegrationId, controlStatusId, type, storageUrl,
  collectedAt, hash, immutable

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
- `Evidence.immutable` tidak dapat diubah setelah ditulis.
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

### 13.1 In-app chatbot

- RAG hanya mengambil `Control`, `Evidence`, `Policy`, dan `OrgControlStatus` milik
  organization yang sedang login.
- Retrieval dan embedding wajib tenant-scoped.
- Jawaban compliance harus menyertakan disclaimer dan tidak boleh menjanjikan
  sertifikasi/lulus audit.
- Chatbot menjelaskan status yang ada; ia tidak boleh mengubah status.

### 13.2 Public chatbot

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

NFR baseline:

| Area | Target |
|---|---|
| Uptime Fase 2 | 99.5% shared hosting |
| API CRUD p95 | <500 ms |
| Normal sync | maksimal 6 jam sejak perubahan sumber |
| Critical alert | target <15 menit sejak deteksi |
| UI locale | 100% EN/ID/ZH sebelum GA |
| AI cost | seluruhnya terukur via `AiUsageLog` |

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

### Fase 1 — MVP SOC 2 + scan, minggu 4–11

**Deliverables**

- SOC 2 controls dan weighted score.
- AWS + GitHub + Google Workspace integration flow.
- AWS minimal 8 checks prioritas, GitHub minimal 4 checks.
- ScanRun/Finding/Evidence/aggregator.
- Dashboard, integrations, control detail, remediation.
- PDF report.
- Policy Center lima template.
- Xendit Starter/Growth.
- AI Gateway + in-app chatbot RAG.
- Onboarding: signup → pay → connect → scan → inspect evidence.

**Release gate**

- Tidak ada status/skor dashboard tanpa ScanFinding nyata.
- Hasil AWS/GitHub dapat dibandingkan manual dan identik pada test account.
- Permission failure menjadi error/needs_review, bukan pass.
- Re-scan setelah remediation dapat mengubah status secara benar.
- PDF lolos dua reader.

### Fase 2 — Trust, auditor, public site, minggu 12–17

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

## 22. Riwayat Dokumen

| Versi | Perubahan |
|---|---|
| 4.0 | Konsolidasi PRD master v3.0 dan Addendum Scan v1.0 menjadi baseline engineering tunggal; konflik status/evidence/path diselesaikan eksplisit. |

Perubahan besar terhadap keputusan final memerlukan RFC baru dan pembaruan versi
dokumen ini. `/DOC/PRD-JOBEN-ENTERPRISE.md` tetap menjadi sumber kebenaran terbaru
setelah perubahan disetujui.