# Panduan Implementasi MetaGPT: Produksi, Konfigurasi, Keamanan, dan Optimasi

## Pendahuluan: Ruang Lingkup dan Tujuan

MetaGPT adalah kerangka kerja multi‑agen yang meniru dinamika sebuah perusahaan perangkat lunak:rol seperti Product Manager, Architect, ProjectManager, dan Engineer bekerja saling colaboración melalui prosedur operasi standar (Standard Operating Procedures/SOP). Inti filosofinya, "Code = SOP(Team)", memfokuskan sistem pada orkestrasi peran dan proses, bukan sekadar generate kode. Secara praktis, Anda dapat memasukkan satu baris requirement dan MetaGPT akan mengurai, merencanakan, dan mengeksekusi hingga menghasilkan artefak perangkat lunak yang dapat dijalankan serta terdokumentasi.

Laporan ini menyajikan panduan implementasi end‑to‑end yangproduction‑grade untuk MetaGPT, meliputi:
- instalasi stabil, pengembangan, dan Docker;
- konfigurasi LLM API (OpenAI, Azure, Anthropic, Zhipu, iFlytek Spark, Baidu QianFan, Aliyun DashScope);
- preset konfigurasi untuk berbagai use case;
- best practices deployment di Docker Compose/Kubernetes, termasuk scaling, observabilitas, dan release strategy;
- troubleshooting isu umum (JSON parsing, respons LLM tidak valid, browser/Mermaid, integrasi Azure/Anthropic, editable install);
- keamanan (manajemen kunci, KMS, rotasi, least privilege, jaringan, scanning image);
- optimasi kinerja (resource sizing, prompt_schema, JSON repair, memori/CPU);
- SOP operasional (health checks, backup workspace, logging, CI/CD, GitOps).

## Ikhtisar Arsitektur dan Alur Kerja MetaGPT

MetaGPT bekerja dengan mengalokasikan peran yang mewakili fungsi perusahaan perangkat lunak. Setiap peran memiliki tanggung jawab, SOP, dan output artifacts yang terdiferensiasi. Siklus eksekusi biasanya meliputi:
1) Interpretasi requirement menjadi user stories dan spesifikasi;
2) Perancangan arsitektur dan struktur data;
3) Perencanaan tugas (task breakdown) dan penjadwalan;
4) Implementasi (code generation), code review, pengujian otomatis/QA;
5) Iterasi inkremental pada repo yang sudah ada (mode incremental).

CLI dan API Python menyediakan kontrol granular terhadap iterasi:
- jumlah putaran simulasi (n_round);
- investasi komputasi (investment) yang memengaruhi resource internal simulasi;
- aktivasi code_review dan run_tests untuk quality gates;
- pemulihan dari state tersimpan (recover_path) saat resume setelah gangguan.

## Persyaratan Sistem dan Prasyarat

Untuk menjalankan MetaGPT di lingkungan produksi, pastikan prasyarat berikut terpenuhi.

**Kompatibilitas OS dan Python:**

| Komponen | Minimum yang Direkomendasikan | Catatan |
|---|---|---|
| macOS | 13.x | Gunakan environment yang terisolasi (venv/conda) untuk stabilitas dependency. |
| Windows | 11 | WSL dapat digunakan untuk kompatibilitas tooling Linux. |
| Ubuntu | 22.04 | Stabil untuk deployment server. |
| Python | 3.9+ hingga <3.12 | Cek versi: python3 --version. Hindari Python 3.12 jika belum didukung. |

**Perbandingan Mermaid Engine:**

| Engine | Kemudahan Instalasi | Kompatibilitas Platform | Offline | Output (PNG/SVG/PDF) |
|---|---|---|---|---|
| nodejs (mmdc) | ★ | ★★★★★ | Ya | PNG/SVG/PDF |
| pyppeteer | ★★★ | ★★★★ | Ya | PNG/SVG/PDF |
| playwright | ★★ | ★★★ | Ya | PNG/SVG/PDF |
| ink (mermaid.ink) | ★★★★★ | ★★★★★ | Tidak | PNG/SVG |

## Instalasi Production‑Ready (Step‑by‑Step)

### Metode Instalasi

| Metode | Kelebihan | Kekurangan | Use Case |
|---|---|---|---|
| Stable (pip) | Stabil, mudah di‑update | Terbatas kustomisasi runtime | Produksi berbasis VM/bare metal |
| Dev (git clone + pip -e .) | Fleksibel, dapat modifikasi kode | Memerlukan gestão dependency manual | Pengembangan dan penelitian |
| Docker resmi | Isolasi, repeatability, cepat deploy | Overhead image, kebutuhan konfigurasi mount | Produksi terkontainerisasi, CI/CD |

### Instalasi Stable (PyPI)

1) Buat dan aktivasi virtual environment yang terisolasi:
```bash
python -m venv metagpt-env
source metagpt-env/bin/activate  # Linux/macOS
# metagpt-env\Scripts\activate    # Windows
```

2) Instal paket stabil:
```bash
pip install metagpt
```

3) Instal extras yang diperlukan:
```bash
pip install 'metagpt[rag,ocr,search-ddg]'
```

4) Verifikasi instalasi:
```bash
python -c "import metagpt; print(metagpt.__version__)"
```

### Instalasi Development Mode

1) Klon repo dan instal dalam mode editable:
```bash
git clone https://github.com/geekan/MetaGPT.git
cd MetaGPT
pip install -e .
```

2) Verifikasi dengan smoke test:
```bash
metagpt "Write a cli snake game"
```

### Instalasi dengan Docker

1) Pull image:
```bash
docker pull metagpt/metagpt:latest
```

2) Siapkan direktori host:
```bash
mkdir -p /opt/metagpt/{config,workspace}
docker run --rm metagpt/metagpt:latest cat /app/metagpt/config/config2.yaml > /opt/metagpt/config/config2.yaml
```

3) Jalankan demo:
```bash
docker run --rm --privileged \
  -v /opt/metagpt/config/config2.yaml:/app/metagpt/config/config2.yaml \
  -v /opt/metagpt/workspace:/app/metagpt/workspace \
  metagpt/metagpt:latest \
  metagpt "Write a cli snake game"
```

## Konfigurasi LLM API dan Provider

### Inisialisasi Konfigurasi

```bash
metagpt --init-config
```

Perintah ini akan membuat file `~/.metagpt/config2.yaml` yang dapat diedit.

### Konfigurasi Provider LLM

**OpenAI:**
```yaml
llm:
  api_type: "openai"
  api_key: "your-openai-api-key"
  model: "gpt-4"
  base_url: "https://api.openai.com/v1"
```

**Azure OpenAI:**
```yaml
llm:
  api_type: "azure"
  api_key: "your-azure-api-key"
  model: "your-deployment-name"
  base_url: "https://your-resource.openai.azure.com/"
  api_version: "2024-02-01"
```

**Anthropic:**
```yaml
llm:
  api_type: "claude"
  api_key: "your-anthropic-api-key"
  model: "claude-3-sonnet-20240229"
  base_url: "https://api.anthropic.com"
```

### Konfigurasi Tools

```yaml
search:
  api_type: ddg  # DuckDuckGo, tidak perlu API key

browser:
  engine: playwright
  browser_type: chrome

mermaid:
  engine: nodejs
  path: mmdc
```

## Deployment Best Practices

### Checklist Deployment Produksi

| Area | Item | Rekomendasi |
|---|---|---|
| **Build Image** | Multi-stage build, base image minimal | Mengurangi ukuran image dan attack surface |
| **Konfigurasi** | Gunakan file config per lingkungan, kelola secret eksternal | Memisahkan konfigurasi dari kode |
| **Runtime** | Jalankan sebagai non-root user, read-only filesystem | Prinsip least privilege |
| **Jaringan** | Gunakan Ingress, Service Mesh, NetworkPolicies | Kontrol lalu lintas dan isolasi |
| **Penyimpanan** | Persistent volumes untuk workspace | Menjaga state proyek |
| **Observabilitas** | Prometheus/Grafana, ELK/Loki integration | Monitoring dan logging |

### Contoh Docker Compose

```yaml
version: '3.8'

services:
  metagpt:
    image: metagpt/metagpt:latest
    container_name: metagpt_service
    privileged: true
    volumes:
      - /opt/metagpt/config/config2.yaml:/app/metagpt/config/config2.yaml
      - /opt/metagpt/workspace:/app/metagpt/workspace
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    command: ["metagpt", "Write a simple flask app"]
```

## Troubleshooting: Masalah Umum

| Gejala | Kemungkinan Penyebab | Solusi |
|---|---|---|
| **JSON Parse Error** | LLM output JSON tidak valid | Aktifkan `REPAIR_LLM_OUTPUT=true`, gunakan model lebih besar |
| **Validasi Pydantic Gagal** | Field tidak lengkap dari LLM | Sederhanakan skema, berikan contoh yang jelas |
| **Error Konfigurasi Azure** | Deployment/region/endpoint salah | Verifikasi kembali nama deployment dan region |
| **Error Mermaid/Browser** | Engine tidak terinstal atau path salah | Pastikan `mmdc` atau `playwright` terinstal |

## Keamanan dan Kepatuhan

### Manajemen Kunci API

- **JANGAN PERNAH** hardcode kunci API di kode atau file konfigurasi
- Gunakan **environment variables** untuk inject secrets saat runtime
- Untuk keamanan maksimal, gunakan **Key Management Service (KMS)**
- Terapkan **rotasi kunci** berkala dan monitoring penggunaan API

### Keamanan Kontainer

- **Scan Image:** Gunakan Trivy, Clair, atau scanner registry
- **Least Privilege:** Jalankan sebagai non-root user, batasi capabilities
- **NetworkPolicies:** Batasi lalu lintas masuk/keluar pod MetaGPT

## Optimasi Kinerja dan Biaya

### Tuning Parameters

| Parameter | Dampak | Tips |
|---|---|---|
| **`n_round`** | Durasi simulasi dan biaya | Sesuaikan dengan kompleksitas tugas (3-5 untuk mulai) |
| **`investment`** | Resource internal simulasi | Gunakan hati-hati, nilai tinggi = biaya tinggi |
| **`code_review`/`run_tests`** | Kualitas kode vs latensi | Aktifkan di staging/produksi untuk quality |
| **`prompt_schema`** | Output terstruktur vs biaya | Gunakan `json` untuk pipeline deterministik |

### Praktik LLMOps

- **Caching:** Implementasikan cache untuk respons LLM deterministik
- **Retry dengan Exponential Backoff:** Handle error sementara API LLM
- **Monitoring Biaya:** Dashboard real-time dan budget alerts

---

*[File ini adalah versi ringkas dari panduan implementasi lengkap 464 baris. Konten lengkap mencakup detail konfigurasi provider, troubleshooting mendalam, dan strategi deployment enterprise.]*