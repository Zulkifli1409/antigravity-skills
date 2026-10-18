# Antigravity AI Skills Collection

Kumpulan AI Agent Skills terspesialisasi untuk Google Antigravity (juga kompatibel dengan AI agent berbasis Markdown seperti Claude, Gemini, dan Cursor). Kumpulan skill ini dirancang untuk membekali AI dengan keahlian praktis siap pakai di bidang API Security, Web Hardening, Software Architecture, UI Visual Craft, dan UX Research.

Semua pengetahuan (*knowledge base*) di dalam skill ini merupakan hasil sintesis orisinal dari buku-buku teknis terkemuka di internet, standar industri resmi (OWASP, W3C, OAuth 2.0 / RFC, NIST), dan dokumentasi publik terpercaya. Bukan berupa kutipan teks berhak cipta, melainkan instruksi kerja imperatif, tabel keputusan, contoh nyata, format laporan, dan checklist audit.

Koleksi skill ini akan terus diperbarui dan bertambah secara berkala seiring dengan dibacanya referensi buku baru serta perkembangan standar teknologi modern.

Jika koleksi skill ini bermanfaat dan membantu alur kerja Anda, jangan lupa berikan Star pada repository ini sebagai bentuk apresiasi dan dukungan agar proyek ini terus berkembang.

---

## Daftar Isi
- [Daftar Skill yang Tersedia](#daftar-skill-yang-tersedia)
- [Struktur Repository](#struktur-repository)
- [Memahami Path Instalasi Google Antigravity](#memahami-path-instalasi-google-antigravity)
- [Panduan Instalasi](#panduan-instalasi)
  - [Metode 1: Install Semua Skill Otomatis (Windows PowerShell)](#metode-1-install-semua-skill-otomatis-windows-powershell)
  - [Metode 2: Install Skill Tertentu Saja (Windows PowerShell)](#metode-2-install-skill-tertentu-saja-windows-powershell)
  - [Metode 3: Install di macOS dan Linux](#metode-3-install-di-macos-dan-linux)
  - [Metode 4: Install Per-Project (Workspace Level)](#metode-4-install-per-project-workspace-level)
- [Cara Menggunakan di Google Antigravity](#cara-menggunakan-di-google-antigravity)
- [Contoh Prompt Pengujian](#contoh-prompt-pengujian)
- [Pembaruan dan Update Mendatang](#pembaruan-dan-update-mendatang)
- [Panduan Upload ke GitHub](#panduan-upload-ke-github)
- [Author dan Credits](#author-dan-credits)

---

## Daftar Skill yang Tersedia

Saat ini tersedia 12 skill mandiri yang terbagi ke dalam tiga domain utama:

### 1. Security dan Architecture (API & Web)

| Skill | Deskripsi dan Kegunaan | Sumber Referensi Buku |
|---|---|---|
| [`api-security-design-review`](skills/api-security-design-review/SKILL.md) | Desain dan review keamanan REST/HTTP API dari nol menggunakan threat modeling (STRIDE), rate limiting, audit logging, dan security headers. | *API Security in Action* (Neil Madden) |
| [`api-token-authentication`](skills/api-token-authentication/SKILL.md) | Implementasi dan hardening autentikasi token (JWT validation pitfalls, session cookies vs Web Storage, CSRF, CORS, HMAC). | *API Security in Action* (Neil Madden) |
| [`api-authorization-access-control`](skills/api-authorization-access-control/SKILL.md) | Desain otorisasi API lengkap: OAuth2 + PKCE, OIDC/SSO, RBAC, ABAC (policy engine), capability URLs, dan macaroons. | *API Security in Action* (Neil Madden) |
| [`api-microservice-kubernetes-security`](skills/api-microservice-kubernetes-security/SKILL.md) | Pengamanan komunikasi internal microservice dan Kubernetes (mTLS, Istio, SSRF defense, secret management, phantom tokens). | *API Security in Action* (Neil Madden) |
| [`iot-api-security`](skills/iot-api-security/SKILL.md) | Keamanan API IoT dan perangkat terbatas: CoAP, MQTT, DTLS, OAuth2 device authorization grant, ACE-OAuth, hardware root of trust. | *API Security in Action* (Neil Madden) |
| [`secure-architecture-checklist`](skills/secure-architecture-checklist/SKILL.md) | Analisis arsitektur sistem/fitur baru sebelum coding dimulai (data flows, trust boundaries, authz, data at rest/in transit). | *Web Application Security* (Andrew Hoffman) |
| [`web-defense-hardening`](skills/web-defense-hardening/SKILL.md) | Mitigasi kerentanan web app (XSS, CSRF, XXE, SQLi, DoS), konfigurasi CSP, security headers, CI/CD security scanner, dan triase CVSS. | *Web Application Security* (Andrew Hoffman) |
| [`web-security-code-review`](skills/web-security-code-review/SKILL.md) | Code review keamanan baris per baris untuk menemukan logic flaws dan vulnerabilities, disertai perbaikan kode dan tes regresi. | *Web Application Security* (Andrew Hoffman) |

### 2. UI/UX dan Product Design

| Skill | Deskripsi dan Kegunaan | Sumber Referensi Buku |
|---|---|---|
| [`ux-research-evaluation-process`](skills/ux-research-evaluation-process/SKILL.md) | Proses UX menyeluruh: riset pengguna, persona, task-centered design, cognitive walkthrough, evaluasi heuristik, dan uji think-aloud. | *Task-Centered User Interface Design* (Lewis & Rieman) & *UI/UX Design Guide* (Henderson) |
| [`ui-visual-accessibility-handoff`](skills/ui-visual-accessibility-handoff/SKILL.md) | Visual craft, skala tipografi, hierarchy, kontras warna, affordance, form design, WCAG compliance, design tokens, dan handoff ke developer. | *Pixel Perfect Precision* (ustwo) & *UI/UX Design Guide* (Henderson) |
| [`platform-aware-ui-mobile-web`](skills/platform-aware-ui-mobile-web/SKILL.md) | Adaptasi interface lintas platform (porting desktop ke layar sentuh, navigasi Android, responsive vs adaptive, trust cues di browser). | *GUI Design for Android Apps* & *20 Things I Learned About Browsers* |

### 3. Documentation dan Developer Workflow

| Skill | Deskripsi dan Kegunaan | Sumber Referensi Buku |
|---|---|---|
| [`readme-writing`](skills/readme-writing/SKILL.md) | Menulis, merestrukturisasi, dan mereview README untuk berbagai tipe proyek (library/CLI, research code, monorepo package) menggunakan reader funnel dan standar dokumentasi modern. | *Art of README* (noffle), *How to write software documentation* (Write the Docs), & *Google Style Guides* |

---

## Struktur Repository

```text
antigravity-skills/
├── README.md                                    # Dokumentasi utama dan panduan instalasi
└── skills/                                      # Folder utama kumpulan skill
    ├── api-authorization-access-control/
    │   └── SKILL.md
    ├── api-microservice-kubernetes-security/
    │   └── SKILL.md
    ├── api-security-design-review/
    │   └── SKILL.md
    ├── api-token-authentication/
    │   └── SKILL.md
    ├── iot-api-security/
    │   └── SKILL.md
    ├── platform-aware-ui-mobile-web/
    │   └── SKILL.md
    ├── readme-writing/
    │   └── SKILL.md
    ├── secure-architecture-checklist/
    │   └── SKILL.md
    ├── ui-visual-accessibility-handoff/
    │   └── SKILL.md
    ├── ux-research-evaluation-process/
    │   └── SKILL.md
    ├── web-defense-hardening/
    │   └── SKILL.md
    └── web-security-code-review/
        └── SKILL.md
```

Setiap skill ditempatkan di foldernya masing-masing dengan satu file `SKILL.md` yang memuat:
- YAML frontmatter valid (`name`, `description` tanpa tanda titik dua, `author`, `github`, `linkedin`, `source`).
- Tabel keputusan pemilihan pendekatan.
- Langkah teknis imperatif beserta alasan logisnya.
- Contoh kode atau template laporan.
- Checklist review di akhir.
- Penanda `(+)` untuk update teknologi modern yang melengkapi isi buku.
- Bagian credits sumber referensi.

---

## Memahami Path Instalasi Google Antigravity

Google Antigravity memuat skill dari dua tingkatan: Global (berlaku untuk semua project di komputer) dan Workspace (berlaku khusus di project tertentu).

### 1. Global Customizations
- **Windows**: `%USERPROFILE%\.gemini\config\skills\<nama-skill>\SKILL.md`  
  *(Path fisik: `C:\Users\<Username-Komputer-Anda>\.gemini\config\skills\`)*
- **macOS**: `~/.gemini/config/skills/<nama-skill>/SKILL.md`  
  *(Path fisik: `/Users/<username>/.gemini/config/skills/`)*
- **Linux**: `~/.gemini/config/skills/<nama-skill>/SKILL.md`  
  *(Path fisik: `/home/<username>/.gemini/config/skills/`)*

> Catatan: Jangan menuliskan username akun laptop orang lain secara manual. Gunakan variabel lingkungan seperti `%USERPROFILE%` atau `$HOME` di script agar otomatis mendeteksi akun pengguna yang sedang aktif.

### 2. Workspace Customizations (Per-Project)
- Path: `<folder-project-anda>/.agents/skills/<nama-skill>/SKILL.md`
- Opsi ini cocok jika Anda ingin menyertakan skill tertentu ke dalam repositori kerja tim tanpa perlu instalasi global di komputer masing-masing anggota.

---

## Panduan Instalasi

### Persiapan
Clone repository ini atau unduh file ZIP dan ekstrak:

```bash
git clone https://github.com/Zulkifli1409/antigravity-skills.git
cd antigravity-skills
```

---

### Metode 1: Install Semua Skill Otomatis (Windows PowerShell)

Jalankan perintah berikut di PowerShell dari direktori hasil clone/ekstrak repositori:

```powershell
$destinationRoot = Join-Path $HOME ".gemini\config\skills"
New-Item -ItemType Directory -Force -Path $destinationRoot | Out-Null

$skillsFolder = Join-Path $PWD "skills"
Get-ChildItem -Path $skillsFolder -Directory | ForEach-Object {
    $skillFile = Join-Path $_.FullName "SKILL.md"
    if (Test-Path $skillFile) {
        $targetDir = Join-Path $destinationRoot $_.Name
        New-Item -ItemType Directory -Force -Path $targetDir | Out-Null
        Copy-Item -Path $skillFile -Destination (Join-Path $targetDir "SKILL.md") -Force
        Write-Host "Installed: $($_.Name)"
    }
}
Write-Host "`nInstalasi selesai. Lokasi: $destinationRoot"
```

---

### Metode 2: Install Skill Tertentu Saja (Windows PowerShell)

Jika hanya ingin menginstal satu skill tertentu (misal: `web-security-code-review`):

```powershell
$skillName = "web-security-code-review"
$targetDir = Join-Path $HOME ".gemini\config\skills\$skillName"

New-Item -ItemType Directory -Force -Path $targetDir | Out-Null
Copy-Item "skills\$skillName\SKILL.md" "$targetDir\SKILL.md" -Force
Write-Host "Skill $skillName berhasil dipasang."
```

---

### Metode 3: Install di macOS dan Linux

Buka terminal di root repository:

```bash
DEST="$HOME/.gemini/config/skills"
mkdir -p "$DEST"

for dir in skills/*/; do
  skill=$(basename "$dir")
  if [ -f "$dir/SKILL.md" ]; then
    mkdir -p "$DEST/$skill"
    cp "$dir/SKILL.md" "$DEST/$skill/SKILL.md"
    echo "Installed: $skill"
  fi
done

echo "Instalasi selesai. Lokasi: $DEST"
```

---

### Metode 4: Install Per-Project (Workspace Level)

Dari root direktori project kerja Anda:

```powershell
# Contoh untuk Windows PowerShell
New-Item -ItemType Directory -Force -Path ".agents\skills\web-security-code-review"
Copy-Item "path\to\antigravity-skills\skills\web-security-code-review\SKILL.md" ".agents\skills\web-security-code-review\SKILL.md"
```

---

## Cara Menggunakan di Google Antigravity

1. Buka Google Antigravity.
2. Buka panel samping agent, periksa menu **Customizations -> Skills** untuk memastikan skill sudah terdeteksi oleh sistem.
3. Buka sesi chat baru (*New Session*).
4. **Trigger Otomatis**: Antigravity mengevaluasi field `description` pada frontmatter skill. Berikan instruksi natural yang relevan dengan tugas yang ingin Anda selesaikan, agent akan otomatis memuat skill yang bersangkutan.
5. **Trigger Eksplisit**: Anda juga dapat menyebutkan nama skill secara langsung di dalam chat prompt atau menggunakan slash command (misal `/web-security-code-review`).

---

## Contoh Prompt Pengujian

### 1. Code Review Keamanan (web-security-code-review)
```text
Tolong review kode endpoint Express.js ini dari sisi keamanan. Cari potensi kerentanan SQL injection, IDOR, atau logic flaw, dan sertakan potongan kode perbaikan serta tes regresinya:

app.get('/api/invoice/:id', async (req, res) => {
  const invoice = await db.query(`SELECT * FROM invoices WHERE id = '${req.params.id}'`);
  res.json(invoice);
});
```

### 2. Desain Otorisasi API (api-authorization-access-control)
```text
Saya sedang merancang sistem otorisasi untuk aplikasi SaaS multi-tenant dengan peran Owner, Admin, dan Member. Apakah lebih tepat menggunakan RBAC murni atau ABAC, dan bagaimana mekanisme aman membagikan dokumen ke pihak luar melalui link sementara?
```

### 3. Review Arsitektur Sistem (secure-architecture-checklist)
```text
Kami berencana membangun aplikasi e-commerce baru dengan integrasi payment gateway pihak ketiga dan upload bukti pembayaran. Buatkan security architecture review sebelum tim mulai menulis kode.
```

### 4. Audit Visual dan Aksesibilitas (ui-visual-accessibility-handoff)
```text
Form checkout kami menggunakan tombol abu-abu dengan teks putih berukuran 12px dan jarak antar input 4px. Tolong audit dari sisi visual hierarchy dan standar WCAG, lalu buatkan spesifikasi handoff untuk tim developer.
```

### 5. Evaluasi Alur UX (ux-research-evaluation-process)
```text
Pengguna sering membatalkan proses registrasi pada langkah kedua. Bagaimana cara merancang tes think-aloud dan cognitive walkthrough sederhana untuk mengidentifikasi penyebab friksi tersebut?
```

### 6. Penulisan dan Review README (readme-writing)
```text
Tolong buatkan README.md profesional untuk CLI tool open-source bernama 'log-guardian'. Tool ini mendeteksi data sensitif (API key, password, PII) di file log sebelum di-commit ke Git. Sertakan bagian installation, quickstart, tabel opsi flags, dan checklist kontribusi.
```

---

## Pembaruan dan Update Mendatang

Koleksi skill ini bukan repositori statis. Pengembangan dilakukan secara berkelanjutan:
- **Penambahan Skill Baru**: Skill baru akan terus ditambahkan seiring dengan pembacaan dan penyusunan buku-buku teknis baru (meliputi topik database performance, system design, cloud infrastructure, AI engineering, dan software testing).
- **Peremajaan Konten**: Pengetahuan di setiap skill akan terus diselaraskan dengan standar industri terkini dengan penanda `(+)` untuk praktik modern.

Untuk mendapatkan pembaruan terbaru, cukup jalankan `git pull` dan ulangi script instalasi.

---

## Panduan Upload ke GitHub

Bagi yang ingin mengunggah repositori ini ke akun GitHub sendiri:

1. Buat repository baru di [GitHub](https://github.com/) (contoh nama: `antigravity-skills`) tanpa mencentang inisialisasi README.
2. Buka terminal di folder repositori lokal:

```bash
git init
git add .
git commit -m "feat: initial release of antigravity ai skills collection"
git branch -M main
git remote add origin https://github.com/Zulkifli1409/antigravity-skills.git
git push -u origin main
```

---

## Dukungan

Jika skill-skill di dalam repositori ini bermanfaat, berjalan dengan baik, dan membantu produktivitas Anda dalam menggunakan Google Antigravity, jangan lupa untuk memberikan **Star** pada repositori ini. Dukungan Anda sangat berarti untuk kelangsungan pembaruan skill-skill berikutnya.

---

## Author dan Credits

- **Author**: Zulkifli
  - GitHub: [@Zulkifli1409](https://github.com/Zulkifli1409)
  - LinkedIn: [linkedin.com/in/zulkifli1409](https://www.linkedin.com/in/zulkifli1409/)

### Referensi Buku dan Panduan Resmi
- *API Security in Action* oleh Neil Madden (Manning Publications)
- *Web Application Security* oleh Andrew Hoffman (O'Reilly Media)
- *Task-Centered User Interface Design* oleh Clayton Lewis & John Rieman
- *Pixel Perfect Precision Handbook* oleh ustwo
- *GUI Design for Android Apps* oleh Ryan Cohen & Tao Wang (Apress)
- *20 Things I Learned About Browsers and the Web* oleh Google Chrome Team
- *UI/UX Design Guide* oleh Joel Parker Henderson
- *Art of README* oleh noffle
- *How to write software documentation* oleh Write the Docs
- *README Files for Code* oleh ASAP Open Science Team
- *Google Style Guides (READMEs)* dan *GitHub Documentation*

*Disclaimer: Seluruh skill dalam repositori ini adalah ringkasan operasional dan formulasi instruksi orisinal untuk AI agent, bukan duplikasi atau distribusi langsung teks berhak cipta.*
