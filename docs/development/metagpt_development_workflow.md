# MetaGPT Development Workflow & Customization untuk Enterprise: End-to-End dari Setup hingga Deployment, CI/CD, dan Observabilitas

![Overview MetaGPT Development Workflow](../diagrams/workflow_overview.png)

*Diagram overview menunjukkan alur lengkap pengembangan MetaGPT dari setup hingga production deployment dengan monitoring.*

## 1. Pendahuluan & Ringkasan Eksekutif

Laporan ini menyajikan panduan end‑to‑end untuk membangun, mengelola, dan mengoperasikan sistem berbasis MetaGPT di lingkungan enterprise. Tujuannya adalah mempercepat adopsi tim inti—tech lead, machine learning (ML) engineer, platform engineer, Site Reliability Engineering (SRE), enterprise architect, dan product engineering manager—dengan rekomendasi yang actionable, pola desain yang teruji, serta kontrol kualitas dan observabilitas yang konsisten.

MetaGPT adalah kerangka kerja multi‑agen yang meniru operasi perusahaan perangkat lunak melalui Standard Operating Procedures (SOP) yang diorkestrasi; konsep intinya diringkas sebagai Code = SOP(Team), yang menekankan bahwa perilaku kolaboratif agen ditentukan oleh SOP yang eksplisit dan dapat dieksekusi. Framework ini menerima persyaratan satu baris dan menghasilkan artefak seperti user stories, analisis kompetitif, requirements, struktur data, API, dan dokumen, sembari mensimulasikan peran internal seperti Product Manager, Architect, Project Manager, dan Engineer[^1]. Filosofis ini relevan dalam skala enterprise karena menuntut disiplin proses (governance, quality gates, traceability) yang juga menjadi tulang punggung DevOps modern. MetaGPT tersedia sebagai proyek open source berlisensi MIT dan mendukung beberapa penyedia model API, sehingga memungkinkan adaptasi sesuai kebijakan dan infrastruktur perusahaan[^2].

Laporan ini berfokus pada enam domain yang saling berkaitan:

- Development workflow end‑to‑end: dari setup, konfigurasi provider Large Language Model (LLM), inisialisasi proyek, siklus iteratif (inkremental), hingga packaging dan deployment.
- Custom agent creation: rancangan Action dan Role, deklarasi states/transitions dalam SOP, serta pola orchestrasi single vs multi‑agent.
- Tool integration & extension: pendaftaran tool dengan decorator, tool registry, dan pola penggunaan tool berbasis fungsi maupun kelas.
- Project structure untuk enterprise: pemetaan artefak SOP ke direktori kerja, konfigurasi lintas lingkungan, dan manajemen secrets.
- CI/CD pipeline: automate build, test, quality gates, dan rilis; artefak yang dipromosikan; serta protection rules.
- Monitoring & logging: instrumentasi traces/metrics/logs, telemetri SOP, dan praktik alerting menuju runbook operasional.

Konteks industri menunjukkan bahwa menggabungkan kerangka kerja multi‑agen dengan praktik engineerings enterprise memerlukan harmonisasi antara kebebasan kreativitas agen dan kontrol proses yang ketat, misalnya melalui observabilitas end‑to‑end dan pipeline yang dapat diaudit[^3]. ICLR 2024 mempublikasikan paper MetaGPT sebagai rujukan akademik, menambah legitimasi bahwa pendekatan SOP+Multi‑Agent bukan sekadar pola praktis, melainkan objek penelitian yang diakui komunitas ilmiah[^4].

Catatan kesenjangan informasi: beberapa aspek—seperti parameter environment produksi spesifik (Docker image tags terbaru, resource requests/limits, readiness/liveness probe untuk deployment di Kubernetes), praktik testing terstandardisasi lintas‑repo MetaGPT (golden test suites, test data management), pedoman governance lengkap di luar开源 (DLP, PII redaction, approval workflows), dan contoh pipeline YAML production‑grade yang patuh compliance—belum terdokumentasi mendalam di sumber yang tersedia. Laporan ini menyebutkan batasan tersebut di bagian terkait dan menawarkan pola generik yang dapat disesuaikan oleh tim enterprise.

## 2. Arsitektur MetaGPT & SOP (Code = SOP(Team))

### 2.2 Arsitektur Inti

Arsitektur MetaGPT dibangun atas dua komponen inti: Action (tindakan) dan Role (peran). Action memodelkan unit kerja yang dapat dijalankan; Role Mengikat Action, memori, dan strategi berpikir/bertindak dari agen. Agen beroperasi dalam siklus: mengambil instruksi terbaru dari memori, memilih todo (Action), mengeksekusi todo, dan mengembalikan hasilnya sebagai Message. Siklus ini mendorong evolusi state proyek melalui messages yang terdokumentasi, sehingga keseluruhan eksekusi menjadi audit-friendly.

SOP formal menjaga agar langkah-langkah pengembangan tetap konsisten. Dalam analogi "assembly line", setiap peran memiliki tangan (Action) yang tugasnya jelas, dan pesan (Message) yang menghubungkan satu tahap ke tahap berikutnya. SOP berfungsi sebagai kontrak: menekan variabilitas yang tidak perlu, tetapi memberi ruang untuk variasi terarah—misalnya ketika engineer perlu mengeksplorasi opsi algoritmik berbeda dalam batas yang disepakati. Pada skala enterprise, SOP menjadi medium utama untuk governance: quality gates dapat ditempelkan di persimpangan antar‑tahap (misalnya, setelah requirements freeze vs setelah code complete), prerequisite antar‑role dapat dipantau, dan setiap output memiliki traceability. Pendekatan ini sangat sesuai dengan prinsip factory automation, tetapi—berbeda dengan lini perakitan—mempertahankanfleksibilitas melalui kemungkinan penyesuaian SOP berbasis data observabilitas[^1][^4].

Untuk memperjelas hubungan komponen dan siklus eksekusi, Tabel 1 memetakan komponen inti serta artefak SOP yang dihasilkan dan yang diperlukan sebagai input.

Tabel 1. Pemetaan komponen dan artefak SOP

| Komponen/Role         | Deskripsi                                                                 | Artefak yang Dihasilkan                      | Input yang Diperlukan                            | Titik Quality Gate                          |
|-----------------------|---------------------------------------------------------------------------|----------------------------------------------|--------------------------------------------------|---------------------------------------------|
| Product Manager       | Mengumpulkan ide, menulis PRD/user stories                                | PRD, user stories, acceptance criteria       | Permintaan bisnis, constraints                   | PRD review/approve                           |
| Architect             | Mendesain struktur data/API/arsitektur tingkat tinggi                     | System design, API spec, data model          | PRD, non‑functional requirements                 | Architecture review/approve                  |
| Project Manager       | Membagi kerja menjadi backlog/tasks, mengatur prioritas                   | Task breakdown, plan                         | System design, estimates                         | Plan approval, dependency alignment          |
| Engineer              | Mengimplementasi fitur sesuai tasks                                       | Code, unit tests, refactor notes             | Tasks, design spec                               | Code complete gates, unit tests pass         |
| QA                    | Menjamin kualitas, membuat & menjalankan tests, laporan defect            | Test suites, bug reports                     | Tasks, acceptance criteria                       | Test execution pass, defect threshold        |
| Team (Orchestrator)   | Mengorkestrasi Role menggunakan SOP, menyimpan state via messages         | Traceable run log (messages), snapshots      | Semua artefak sebelumnya                         | SOP compliance checks, roll‑forward/backtrack|

Pemetaan ini membantu enterprise menempatkan kontrol kualitas tepat pada batas transisi yang rawan defect. Pada praktiknya, observabilitas dapat menghubungkan message‑level tracing ke artefak SOP yang konkret (misalnya, linkage antara task ID dan commit SHA), sehingga incident review tidak hanya bertanya "apa yang salah", tetapi juga "di tahap mana SOP menyimpang" dan "siapa peran yang memiliki konteks paling relevan"[^1][^4].

## 3. Setup & Konfigurasi: Lingkungan, Dependencies, dan Provider LLM

Setup production‑grade dimulai dari lingkungan eksekusi yang deterministic: gunakan Python 3.9–3.11, idealnya dalam environment yang diisolasi (misalnya conda atau virtualenv). Saat menulis panduan ini, repo resmi mencatat dukungan Python ≥ 3.9 dan < 3.12; pilih versi 3.9 atau 3.10/3.11 untuk menghindari risiko kompatibilitas. If‑needed, perkuat reproducibility melalui Docker atau devcontainer; setelah itu, lakukan instalasi via pip atau mode editable dari source, sesuai preferensi tim. Pastikan pip dan setuptools up‑to‑date untuk mencegah masalah build wheel di CI[^2].

Konfigurasi MetaGPT memanfaatkan file konfigurasi berurutan prioritas: ~/.metagpt/config2.yaml menanggalkan konfigurasi lokal di repo (config/config2.yaml). Gunakan CLI "metagpt --init-config" untuk membuat templat dan hindari menyalin secrets ke Version Control System (VCS). Praktik ini menjaga clean separation antara secrets dan kode. Elektivitas ini krusial untuk audit compliance dan pemisahan akses antar tim[^2][^5][^6][^7][^8].

Tabel 2 merangkum dukungan provider LLM dan parameter kunci yang umumnya diperlukan.

Tabel 2. Dukungan provider LLM dan parameter kunci

| Provider          | api_type          | Model Contoh                       | Kunci/Auth Utama                         | Base URL (umum)                   | Catatan Stabilitas/Referensi |
|-------------------|-------------------|------------------------------------|------------------------------------------|-----------------------------------|-------------------------------|
| OpenAI            | openai            | gpt‑4‑turbo / gpt‑4‑1106‑preview   | api_key                                  | Contoh tersedia di konfigurasi    | Stabil; gunakan latest sesuai kebijakan[^2] |
| Azure OpenAI      | azure             | model deployment Anda              | api_key                                   | Endpoint Azure                    | Sesuaikan deployment name & API version[^2] |
| Ollama            | ollama            | model lokal (variasi open source)  | (tidak selalu butuh key)                  | lokal (oleh Ollama)               | Berguna untuk on‑prem/test[^2]              |
| Groq              | groq              | model yang didukung Groq           | api_key                                  | endpoint Groq                     | Untuk throughput tinggi[^2]                  |

Catatan: tabel ini menampilkan kategori generik dan pola parameter yang lazim. Terapkan kebijakan internal untuk/model/versi apa yang diperbolehkan. Untuk secret management, gunakan pendekatan yang patuh (misalnya vault/cloud secret manager) dan pastikan config2.yaml tidak Contiene secrets—khususnya di fork atau CI job publik[^2][^5][^6][^7][^8].

### 3.1 Prerequisites & Instalasi

- Python 3.9–3.11 (ikuti pedoman repo tentang dukungan < 3.12).
- Buat env isolasi (misalnya conda), lakukan instalasi MetaGPT melalui:
  - pip install --upgrade metagpt (stabil),
  - atau mode editable: git clone + pip install -e . (berguna untuk pengembangan internal),
  - atau instal直接从 GitHub (mode developer). Pilih salah satu sesuai kontrol upgrade yang dibutuhkan[^2].

### 3.2 Konfigurasi LLM API

- Jalankan metagpt --init-config untuk membuat ~/.metagpt/config2.yaml.
- Prioritas konfigurasi: ~/.metagpt/config2.yaml > config/config2.yaml (repo).
- Isi parameter sesuai provider yang dipilih: api_type, model, api_key, base_url (bila diperlukan).
- Pertimbangkan enable_longterm_memory (default biasanya off) dan prompt_schema (json/markdown), karena memengaruhi determinisme prompt dan kebutuhan instrumentation downstream[^5][^6][^7][^8].

### 3.3 Konfigurasi Tools Opsional

Tools opsional menambah kapabilitas eksekusi agent (misalnya web search, browsing, text‑to‑speech, rendering diagram). Prinsipnya: treat tools sebagai dependensi eksternal yang membutuhkan credentials; segregasikan konfigurasinya dari secrets inti LLM, dan pastikan dokumentasi internal menjelaskan cara rotation keys dan scoping (dev/staging/prod). Pengalaman produksi menunjukkan bahwa ketika tool calling menjadi bagian SOP, observabilitas tool usage harus disetarakan dengan logs aplikasi biasa[^5][^2].

## 4. Development Workflow End‑to‑End: Dari Requirement ke Deployment

![Complete Development Workflow](../diagrams/metagpt_development_workflow.png)

*Workflow diagram menampilkan langkah-langkah lengkap dari setup environment hingga deployment dan monitoring.*

Workflow lengkap dapat dijalankan dari CLI atau melalui library API. CLI "metagpt" menyediakan opsi yang memadai untuk orkestrasi SOP dari satu baris requirement; library API memungkinkan integrasi dengan pipeline internal. Pada praktiknya, pilih jalur eksekusi berdasarkan "lokasi kontrol" (apakah SOP dijalankan dari pipeline terkelola atau dari layanan aplikasi internal).

Tabel 3 merangkum opsi CLI utama dan implikasinya terhadap alur kerja.

Tabel 3. Ringkasan opsi CLI MetaGPT dan implikasi

| Opsi              | Tipe       | Default (contoh) | Implikasi Workflow                                                |
|-------------------|------------|------------------|-------------------------------------------------------------------|
| idea              | TEXT       | None (wajib)     | Baris ide/requirement sebagai trigger eksekusi                    |
| --investment      | FLOAT      | 3.0              | Anggaran simulasi tim (dapat memengaruhi intensitas eksekusi)     |
| --n-round         | INTEGER    | 5                | Jumlah putaran simulasi; memengaruhi kedalaman iterasi            |
| --code-review     | BOOLEAN    | aktif            | Mengaktifkan review kode internal SOP                             |
| --run-tests       | BOOLEAN    | tidak aktif      | Mengaktifkan fase QA & eksekusi unit tests                        |
| --implement       | BOOLEAN    | aktif            | Mengaktifkan implementasi kode                                    |
| --project-name    | TEXT       | None             | Menamai proyek; memengaruhi direktori output                      |
| --inc             | BOOLEAN    | tidak aktif      | Mode inkremental untuk bekerja pada repoexisting                  |
| --project-path    | TEXT       | None             | Menentukan direktori proyek untuk iterasi                         |
| --reqa-file       | TEXT       | None             | Sumber untuk rewrite jaminan kualitas                             |
| --max-auto-summarize-code | INTEGER | 0           | Membatasi jumlah auto summarize; berguna untuk debugging          |
| --recover-path    | TEXT       | None             | Memulihkan proyek dari state tersimpan                            |
| --init-config     | BOOLEAN    | tidak aktif      | Membuat file konfigurasi ~/.metagpt/config2.yaml                  |

CLI ini biasanya memadai untuk iterasi cepat (ide → prototype). Namun, pada pipelines terkelola (misalnya GitHub Actions), library API menawarkan kontrol eksekusi yang lebih halus, termasuk penyisipan quality gates kustom dan pemulihan state dari artefak sebelumnya[^9].

Kualitas harus dijaga sejak awal. Tabel 4 mendaftar quality gates yang lazim pada SOP MetaGPT.

Tabel 4. Quality gates pada SOP dan kriteria kelulusan

| Tahap SOP                | Kriteria Kelulusan                                              | Output yang Diverifikasi                         |
|--------------------------|------------------------------------------------------------------|--------------------------------------------------|
| PRD freeze               | PRD lengkap, agree on scope & acceptance criteria               | Dokumen PRD, user stories                        |
| Architecture sign‑off    | Design sesuai NFR, integrasi API, data model konsisten          | System design, API spec                          |
| Code complete            | Implementasi sesuai tasks, tidak ada lint errors                | Codebase, lint reports                           |
| Unit tests pass          | ≥ ambang coverage yang disepakati, semua test hijau             | Test reports, coverage reports                   |
| QA verification          | Bug trend menurun, zero critical blockers                       | Bug reports, defect dashboard                    |
| SOP compliance           | Traceability steps terpenuhi, roles sesuai RACI                 | Message trace, SOP checklist                     |

Quality gates mencegah "drift" dari SOP, sehingga proses tidak hanya cepat, tetapi juga dapat diaudit dan diulang (repeatable). Kunci keberhasilannya adalah definisi ambang yang jelas (measurable) dan mekanisme enforcement yang konsisten di pipeline[^1][^9][^10].

### 4.1 Inisialisasi Proyek & Eksekusi Pertama

- Cara paling cepat: eksekusi "metagpt 'write a cli blackjack game'" untuk melihat tim agen (PM, Architect, PM, Engineer) bekerja menghasilkan artefak pengembangan perangkat lunak di direktori kerja lokal. Ini memvalidasi environment, konfigurasi LLM, serta wiring SOP end‑to‑end[^9][^11].
- Pola ini cocok untuk proof‑of‑concept internal dan sebagai baseline regress test: tim bisa menjalankan requirement serupa untuk memverifikasi bahwa update framework tidak merusak workflow inti[^9].

### 4.2 Pengembangan Inkremental & Iterasi Berkelanjutan

MetaGPT mendukung mode inkremental agar Anda dapat bekerja di atas repoexisting. Gunakan parameter --inc dan --project-path untuk menambahkan requirements atau memperbaiki bug tanpa memutarbalikkan basis kode. Perubahan yang berasal dari bugfix atau requirement baru akan masuk melalui dokumen iterasi (misalnya, docs/requirement.txt untuk batch requirement, dan docs/bugfix.txt untuk umpan balik bug). Fungsi --run-tests mengaktifkan fase unit tests, sementara --max-auto-summarize-code membatasi jumlah perbaikan otomatis saat ringkasan kode—berguna untuk debugging alur kerja tanpa infinite loop[^10].

Tabel 5 merangkum parameter CLI yang relevan untuk inkremental.

Tabel 5. CLI untuk inkremental: parameter utama

| Parameter               | Fungsi Utama                                                    | Contoh Penggunaan (disimulasikan)                      |
|-------------------------|------------------------------------------------------------------|--------------------------------------------------------|
| --inc                   | Mengaktifkan mode inkremental                                   | metagpt "ADD FEATURE X" --inc --project-path ./repo    |
| --project-path          | Menentukan direktori proyek existing                            | (lihat di atas)                                        |
| --project-name          | Menamai proyek (terutama untuk proyek baru)                     | metagpt "Create a Snake game" --project-name snake     |
| --run-tests             | Menjalankan fase unit tests                                     | metagpt "..." --run-tests                              |
| --max-auto-summarize-code | Membatasi jumlah ringkasan kode otomatis                        | metagpt "..." --max-auto-summarize-code 3              |
| --n-round               | Menentukan jumlah putaran eksekusi SOP                          | metagpt "..." --n-round 7                              |

Setelah iterasi, evaluasi artefak yang boleh diedit langsung versus yang tidak, termasuk lokasi dokumen yang menjadi sumber kebenaran (source of truth). Tabel 6 memetakan jalur file umum proyek MetaGPT dan是否可以 diedit.

Tabel 6. Struktur file proyek MetaGPT dan ketereditan

| Jalur                         | Editable? | Deskripsi                                                                 |
|------------------------------|-----------|---------------------------------------------------------------------------|
| .dependencies.json           | No        | Hubungan ketergantungan antar file; dikelola framework                   |
| docs/requirement.txt         | No        | Persyaratan baru untuk iterasi ini; akan digabung ke docs/prds           |
| docs/bugfix.txt              | No        | Umpan balik bug; dikonversi menjadi input untuk requirements              |
| docs/prds                    | Yes       | Rincian final requirements                                                 |
| docs/system_designs          | Yes       | Desain sistem utama                                                        |
| docs/tasks                   | Yes       | Tugas pengkodean untuk engineer                                           |
| docs/code_summaries          | Yes       | Ringkasan/tinjauan basis kode                                             |
| resources/competitive_analysis | No      | Analisis kompetitor                                                        |
| resources/data_api_design    | No        | Tampilan kelas/data                                                       |
| resources/seq_flow           | No        | File aliran sequence                                                       |
| resources/system_design      | No        | File desain sistem                                                        |
| resources/prd                | No        | File PRD                                                                   |
| resources/api_spec_and_tasks | No        | Spec API & tasks                                                           |
| tmp                          | No        | File perantara; tidak di‑git                                              |
| Direktori sumber proyek      | Yes       | Kode aplikasi                                                             |
| tests                        | Yes       | Kode pengujian unit                                                        |
| test_outputs                 | No        | Hasil eksekusi test                                                        |

Praktik ini menjaga kejelasan boundary: file "No" merupakan hasil turunan atau sumber eksternal yang tak boleh diubah sembarang; file "Yes" adalah artefak yang menjadi fokus kolaborasi tim. Dengan demikian, change management menjadi transparent dan dapat ditelusuri[^10].

### 4.3 Packaging & Deployment

Repo MetaGPT menyediakan Dockerfile dan panduan instalasi Docker; pilih containerization untuk reproducibility. Untuk deployment enterprise, Anda dapat menerapkan layanan stateless sebagai Pod di Kubernetes atau memilih serverless container (misalnya Cloud Run) bila workload bersifat event‑driven. Karena parameter environment produksi spesifik seperti resource requests/limits dan probe readiness/liveness tidak terdokumentasi pada sumber resmi saat ini, gunakan baseline default berikut[^2]:

- Requests: CPU 500m, Memory 1Gi; Limits: CPU 2, Memory 4Gi.
- Liveness probe: HTTP /healthz, initialDelay 30s, period 10s.
- Readiness probe: HTTP /ready, initialDelay 10s, period 5s.
- Autoscaling: HPA dengan target CPU 60%.

Rekomendasi ini adalah baseline yang konservatif dan perlu disesuaikan dengan profil beban aktual—misalnya untuk workloads yang lebih intensif pada LLM calls atau tool usage. Lebih lanjut, gunakan config2.yaml lokal untuk environment (dev/stage/prod) dan zewnętrzne secret manager; jangan menempatkan secrets di VCS. Jalankan build dan push registry privat, kemudian deploy dengan kontrol akses berbasis namespace.

## 5. Custom Agent Creation: Pola Desain & Implementasi

Custom agent diimplementasikan melalui dua kelas utama: Action dan Role. Action adalah unit yang mengeksekusi logika, baik yang memanggil LLM (melalui self._aask) maupun operasi tanpa LLM (misalnya eksekusi subprocess). Role adalah komposisi action, memori, dan strategi reaktivitas. Siklus eksekusiRole meliputi pengambilan instruksi terbaru, penjadwalan todo, eksekusi, dan emisi Message sebagai output. Pola single‑agent cocok untuk automation terarah dan bounded; multi‑agent diperlukan saat kolaborasi lintas peran menjadi kunci kualitas (misalnya PRD→Design→Code→QA)[^12].

Tabel 7 merangkum pola rancangan Action dan Role, serta reaktivitas.

Tabel 7. Pola rancangan Action dan Role

| Elemen                    | Tanggung Jawab Utama                                  | Fungsi/Metode Kunci            | Reaktivitas (react_mode)      |
|--------------------------|--------------------------------------------------------|--------------------------------|--------------------------------|
| Action                   | Menjalankan tugas, bisa LLM/non‑LLM                    | run(): …, self._aask(…)        | N/A (dieksekusi oleh Role)    |
| Role (single‑action)     | Mengikat 1 Action, memori, strategi                    | set_actions([…]), run()        | default atau sesuai kebutuhan |
| Role (multi‑action)      | Mengikat >1 Action, urutan eksekusi                    | set_actions([…]), _set_react_mode(react_mode="by_order") | by_order (urutan) atau free‑form |
| State (via messages)     | Menjaga context antar tahap                            | Message, self.get_memories(k=N) | Konsistensi state             |

Sebelum membuat custom agents, tim perlu menyepakati kontrak SOP: input/output setiap Action, prerequisite state, dan criteria untuk proceed/rollback. Tabel 8 menyediakan template kontrak untuk membantu governance.

Tabel 8. Template kontrak SOP untuk custom agents

| Action/Role          | Input State                               | Output State                                | Prerequisite                           | Quality Gate                         | Tool Dependensi        |
|----------------------|-------------------------------------------|---------------------------------------------|----------------------------------------|--------------------------------------|------------------------|
| Write PRD            | Business idea, constraints                | PRD draft (user stories, acceptance criteria) | Idea disetujui sponsor                 | PRD review & approve                 | N/A                    |
| Design API           | PRD, NFR                                  | System design, API spec                      | PRD freeze                              | Architecture sign‑off                | N/A                    |
| Implement Feature    | Tasks, design spec                        | Code, unit tests                             | Design sign‑off                         | Lint/format pass, tests pass         | (misal: code runner)   |
| QA Verify            | Code, acceptance criteria                 | Test reports, bug list                       | Implement complete                      | Zero critical blockers               | Test runner            |

Template ini mengurangi kebingungan "siapa menurunkan apa" dan "kapan boleh lintas tahap". Ia juga memperjelas dependency terhadap tools eksternal (misal test runner), yang penting untuk observabilitas dan kontrol keamanan[^12].

### 5.1 Single‑Action Agent

Untuk agent sederhana, buat kelas turunan Action dengan prompt template yang deterministik. Dalam metode run(), gunakan self._aask untuk memanggil LLM dan parse output untuk mendapatkan hasil yang diharapkan (misalnya fungsi Python beserta test cases). Pada Role, Lakukan set_actions([ActionKelas]) dan kemudian pada run() ambil instruksi terbaru dari memori (self.get_memories(k=1)[0]), jalankan self.rc.todo.run(msg.content), dan kembalikan Message yang berisi hasil. Pola ini ideal untuk tugas yang jelas batasnya (misalnya "tulis fungsi X dengan 2 test case"), tanpa perlu kolaborasi multi‑role[^12].

### 5.2 Multi‑Action Agent

Untuk sekuens tindakan, definisikan beberapa Action: salah satu mungkin write code via LLM; yang lain execute code melalui subprocess untuk memvalidasi. Pada Role, set_actions([ActionWrite, ActionRun]) dan set react_mode="by_order" untuk保证 urutan deterministik. Logika run() akan mengambil pesan terbaru, menjalankan Action pertama, menyimpan hasil ke memori, lalu melanjutkan ke Action kedua hingga sequence selesai. Pola ini mensimulasikan "assembly line" di mana kualitas bertambah di setiap tahap. Tambahkan quality gates di antara Actions untuk pemeriksaan intermediate (misalnya cek lint setelah write, cek test result setelah run)[^12].

## 6. Tool Integration & Extension Development

Tool adalah cara untuk memperluas kemampuan agent di luar LLM calls, misalnya menambahkan fungsi matematika, interaksi dengan sistem eksternal, atau penjelajahan web. Di MetaGPT, tool didaftarkan melalui decorator @register_tool, baik berbasis fungsi maupun kelas. Penting untuk menulis docstring (Google style) yang komprehensif agar agent dapat memilih tool yang tepat dan memahami kontrak input/output. Tool registry memungkinkan agent untuk mencari dan memanggil tool berdasarkan deskripsi tersebut[^13].

Tabel 9 merangkum atribut pendaftaran tool dan implikasinya.

Tabel 9. Atribut pendaftaran tool

| Atribut              | Deskripsi                                            | Implikasi pada Registry/Agent                                 |
|----------------------|------------------------------------------------------|----------------------------------------------------------------|
| tags                 | Kategori tool (misalnya "math", "web")              | Filter/tool discovery; cocok untuk tool grouping              |
| include_functions    | Daftar metode yang diekspor (khusus tool kelas)     | Meminimalkan "surface" dan menghindari metode non‑essential   |
| Nama tool            | Nama fungsi/kelas saat didaftarkan                  | Deterministik untuk pemanggilan agent                         |
| Docstring (Google)   | Deskripsi tujuan, parameter, return value           | "Executable documentation" untuk agent selection              |

Langkah integrasi tool kepada DataInterpreter (DI) cukup langsung: import tool dan передайте nama tool ke parameter tools saat inisialisasi DI. Pada praktiknya, buat daftar putih tool yang aman, terapkan timeouts dan rate limits sesuai kebijakan, daninstrumentasikan tool usage sebagai bagian dari traces (lihat bagian observabilitas). Untuk tool yang mengakses sistem eksternal (web search/browsing), gunakan credentials yang dibatasi (scoped) dan pastikan audit trail disediakan[^13][^5].

### 6.1 Tool Berbasis Fungsi

Tempatkan fungsi di direktori tools/libs, tulis docstring yang jelas, dan terapkan @register_tool. DataInterpreter akan memuat tool berdasarkan nama saat inisialisasi. Uji fungsi secara deterministik (contoh: calculate_factorial dengan input non‑negatif dan error handling untuk input negatif). Pola ini mudah dirawat dan sangat cocok untuk operasi yang stateless, pure, dan cepat[^13].

### 6.2 Tool Berbasis Kelas

Untuk tool yang menyatukan beberapa operasi terkait (misalnya Calculator), gunakan kelas dengan metode statis, deklarasi @register_tool dengan tags dan include_functions untuk memilih metode yang diekspor. Dengan cara ini, agent mengakses operasi matematika sebagai satu kesatuan tool ("Calculator"), dan docstring kelas/metode menjelaskan kontrak setiap operasi. Pola ini menjaga namespace tetap rapi dan mengurangi risiko kebingungan di agent saat memilih tool[^13].

## 7. Project Structure untuk Enterprise

![Enterprise Project Structure](../diagrams/project_structure.png)

*Diagram menampilkan struktur direktori yang enterprise-ready dengan pemisahan aplikasi, development, infrastructure, dan CI/CD layers.*

Struktur direktori yang enterprise‑ready harus memisahkan konfigurasi (dev/stage/prod), kode aplikasi, skrip deployment, dan artefak SOP. Repo MetaGPT menyediakan baseline direktori dan konfigurasi (.devcontainer, .github, config, docs, examples, metagpt, tests) yang dapat dipetakan ke modul enterprise. Konfigurasi lintas lingkungan memanfaatkan config2.yaml dengan prioritas ~/.metagpt/config2.yaml > config/config2.yaml. Secrets tidak boleh masuk VCS; gunakan vault/secret manager dan rotasi kunci berkala[^2][^5][^7][^8].

Tabel 10 memetakan direktori repo ke konteks enterprise.

Tabel 10. Pemetaan direktori repo ke modul enterprise

| Direktori/Files           | Tujuan Enterprise                                   | Praktik Terbaik                                   |
|---------------------------|------------------------------------------------------|---------------------------------------------------|
| .devcontainer             | Dev environment deterministik                        | Versi image stabil, extend untuk corporate deps  |
| .github/                  | Workflow CI/CD, templates                            | Protection rules, code owners                     |
| config/                   | Konfigurasi defaults (non‑secret)                    | Environ overrides via ENV/Secret Manager          |
| docs/                     | Dokumentasi SOP/PRD/design                           | Source of truth untuk iterasi                     |
| examples/                 | Contoh reusable untuk pelatihan                      | Diminimalkan drift dari core SOP                  |
| metagpt/                  | Framework/internal libs                              | Isolasi vendor code                               |
| tests/                    | Test suites, regression                              | Golden test, coverage thresholds                  |
| config2.yaml.example      | Template konfigurasi                                 | Hindari secrets, dokumentasi param                |
| Dockerfile                | Build reproducibility                                 | Non‑root user, multi‑stage build                  |

Konfigurasi lintas lingkungan harus mengekspos parameter yang dapat diubah (model, timeouts, feature flags) dan menyimpan credentials terpisah. Tabel 11 merekomendasikan nilai default dev/staging/prod untuk beberapa parameter umum—ingat untuk mengganti default saat informasi environment produksi spesifik belum tersedia.

Tabel 11. Rekomendasi konfigurasi lintas lingkungan

| Parameter                  | Dev (default)      | Staging (rekomendasi) | Prod (rekomendasi)          |
|---------------------------|--------------------|------------------------|------------------------------|
| LLM model                 | gpt‑4‑turbo        | gpt‑4‑1106‑preview     | Sesuai kebijakan enterprise  |
| Timeout LLM (detik)       | 30                 | 60                     | 90                           |
| enable_longterm_memory    | false              | false                  | true (bila diperlukan)       |
| prompt_schema             | json               | json                   | json/markdown (sesuai SOP)   |
| Tool usage                | whitelist minimal  | whitelist ketat        | whitelist ketat + observabilitas |

Rancangan ini menjaga konsistensi SOP sambil memberi ruang penyesuaian sesuai profil risiko lingkungan. Pastikan config2.yaml.example terdokumentasi dan tidak berisi secrets[^5][^7][^8].

## 8. CI/CD Pipeline Setup (GitHub Actions)

![CI/CD Pipeline Architecture](../diagrams/cicd_pipeline.png)

*Diagram menunjukkan arsitektur CI/CD pipeline untuk MetaGPT dengan stages, quality checks, dan deployment targets.*

Pipeline CI/CD enterprise untuk MetaGPT harus mendukung multi‑job yang jelas: lint/format, unit tests, build container, security scan, dan deploy. Simpan secrets di GitHub Environments dengan protection rules dan approvals sesuai kebijakan. Quality gates dapat dijalankan sebagai bagian dari pipeline—misalnya validasi SOP checks, coverage thresholds, dan artefak compliance. Gunakan triggers yang aman (push, pull_request) dan centralize documentation tentang cara menjalankan pipeline secara manual untuk incident response[^14][^15][^16].

Tabel 12 menggambarkan stage pipeline dan artefak yang dipromosikan.

Tabel 12. Diagram pipeline (deskripsi tabular)

| Stage                 | Tujuan                                | Artefak yang Dihasilkan                      | Gate/Approval                        |
|-----------------------|----------------------------------------|----------------------------------------------|--------------------------------------|
| Lint & Format         | Konsistensi kode, early failure        | Lint reports, format diff                    | Must pass                            |
| Unit Tests            | Validasi fungsional                    | Test reports, coverage XML/HTML              | Coverage ≥ threshold                 |
| Build Container       | Reproducibility                        | Image, SBOM                                  | Must build                           |
| Security Scan         | Deteksi vulnerability                  | Scan reports (dependabot, container scan)    | No criticals                          |
| SOP Compliance Check  | Traceability SOP                       | Message trace, checklist report              | Must pass                            |
| Deploy Staging        | Validasi terintegrasi                  | Deployed service (staging)                   | Manual approval                      |
| QA Verification       | Pengujian pengguna / integrasi         | QA reports, bug trend                        | QA sign‑off                          |
| Deploy Prod           | Rilis ke produksi                      | Deployed service (prod)                      | Change approval ( CAB / Change Advisory Board ) |

Untuk memastikan pipeline berjalan stabil, pasang prat条件和 guardrails. Tabel 13 mendaftar pratconditions dan guardrails.

Tabel 13. Pratconditions & guardrails per job

| Job                  | Pratconditions                                      | Guardrails                                      |
|----------------------|-----------------------------------------------------|--------------------------------------------------|
| Lint & Format        | Python env siap, dependencies installed             | Fail‑fast, jelaskan output yang fail             |
| Unit Tests           | Lint pass                                           | Coverage thresholds, golden tests                |
| Build Container      | Lint+Tests pass                                     | Reproducible build, SBOM export                  |
| Security Scan        | Build pass                                          | No criticals; automática PR checks               |
| SOP Compliance       | Lint+Tests+Scan pass                                | Checklist must be green                          |
| Deploy Staging       | Compliance pass                                     | Manual approval, blue/green option               |
| QA Verification      | Deploy Staging pass                                 | Bug threshold; rollback if fail                  |
| Deploy Prod          | QA Verification pass                                | Change approval; audit logs                      |

### 8.1 Desain Pipeline & Quality Gates

Tetapkan coverage minimum, dependencia checks (misalnya konfirmasi secrets terpasang, model accessible), dan kebijakan signed artifact/SBOM. Saya juga menyarankan integrasi SOP compliance checks pada pipeline (cek apakah message trace memuat bukti transisi yang diperlukan). Pada produksi, gunakan deployment strategy yang aman (blue/green atau canary) dan maintain rollback plan yang teruji—misalnya mengembalikan ke image versi sebelumnya jika sinyal kualitas turun di bawah ambang[^14][^15][^16].

Catatan kesenjangan informasi: contoh pipeline YAML production‑grade yang spesifik untuk MetaGPT belum tersedia di sumber resmi saat ini. Tim perlu mengadopsi pola umum GitHub Actions dan menambah guardrails sesuai compliance internal.

## 9. Monitoring & Logging Strategies (Observability)

![Monitoring & Observability Architecture](../diagrams/monitoring_architecture.png)

*Diagram menampilkan arsitektur monitoring dan observability untuk MetaGPT dengan logging layer, metrics collection, observability stack, dan alerting system.*

Observabilitas multi‑agen menuntut instrumentasi yang menangkap traces, metrics, dan logs lintas komponen—agent actions, tool usage, LLM calls. OpenTelemetry (OTel) untuk AI agents emerging sebagai standar untuk melakukan instrumentation dengan span untuk "agent run", "tool call", "LLM request", dan "SOP step". Dengan relasi parent‑child span, tim dapat melihat waktu eksekusi per tahap SOP, latensi tool, dan error rate per komponen[^17]. Microsoft menekankan bahwa praktik observabilitas agentik harus mencakup monitoring, control, reliability, dan akurasi, serta kontrol jalur untuk memastikan AI behaves sesuai nilai organisasi[^18]. Praktik industri lain menambahkan bahwa logs perlu menjadi "accountability record" dari aksi agent (tool usage, hasil, error), bukan sekadar debug statements; gunakan structured logging dan korelasi ke run IDs untuk audit pasca insiden[^19].

Tabel 14 merangkum sinyal utama yang perlu dimonitor.

Tabel 14. Matriks sinyal observabilitas

| Sinyal          | Sumber (Agent/Tool/LLM)         | Wajib/Optional | Tujuan                                   |
|-----------------|----------------------------------|----------------|-------------------------------------------|
| Trace (run)     | Agent run, SOP steps, tool calls | Wajib          | lineage & latensi                         |
| Metric: success/error | LLM requests, tool calls    | Wajib          | reliabilitas                              |
| Metric: latency | LLM requests, tool calls         | Wajib          | performance                                |
| Count: token usage | LLM calls                     | Opsional (wajib di sebagian besar enterprise) | cost tracking & budgeting           |
| Log: tool usage | Tool calls                      | Wajib          | audit aksi eksternal                      |
| Log: SOP message | Role/Action messages            | Wajib          | traceability SOP                          |

Untuk alert, buat kebijakan threshold dan escalation yang jelas. Tabel 15 menggambarkan policy generik.

Tabel 15. Policy alert

| Sinyal                | Threshold (contoh)                     | Severity | Runbook                                    |
|-----------------------|----------------------------------------|----------|--------------------------------------------|
| Error rate LLM        | > 2% per 15 menit                      | High     | Cek provider status, retry/backoff, fallbacks |
| Latency p95 LLM       | > target 2x selama 10 menit            | Medium   | Profil beban, scale/out atau alihkan traffic |
| Token over‑run        | > anggaran harian 80%                  | Medium   | Kurangi concurrency, audit prompts         |
| Tool failure          | > 1% pada tool kritis                  | High     | Rotasi key, switch provider/tool, investigate |
| SOP step stuck        | Trace terputus > X menit               | High     | Cancel run, isolate state, manual proceed  |

### 9.1 Instrumentation Plan

Rencanakan span naming conventions untuk OTel. Contoh:
- agent.run (root) → sop.step (PRD, Design, Code, QA) → llm.request → tool.call.
- Tambahkan attributes: model, token_in/out, latency, status_code, tool_name, sop_step_id, run_id.
- Di aplikasi, gunakan hooks/patch untuk menutup span di akhir eksekusi action/tool/LLM. Integrasikan dengan pipeline sehingga run artifacts (SBOM, coverage) dan observability data ditautkan ke release notes dan change records. Tujuan akhirnya adalah setiap run memiliki "dossier" lengkap: apa yang dilakukan agent, apa tool yang dipanggil, apa hasil LLM, serta bukti quality gates dilampaui[^17].

Catatan kesenjangan informasi: detail implementasi instrumentasi MetaGPT spesifik belum terdokumentasi pada sumber resmi; gunakan pola OTel generik dan sesuaikan dengan kontrak Action/Role Anda.

## 10. Keamanan, Governance, dan Compliance

Konfigurasi API dan secrets harus disegregasikan dari kode, tidak masuk VCS, dan di‑rotate secara berkala. Terapkan principle of least privilege untuk credentials tool eksternal (web search, browsing, TTS) serta scoping per environment. Governance SOP perlu mendefinisikan approval workflow untuk perubahan Playbook/agents/tools; gunakan review dari Architecture/QA/Product sesuai kebijakan perubahan. Logging aman termasuk masking PII dan enkripsi data sensitive; pastikan audit trail untuk eksekusi agent dan akses tool—terutama pada proyek yang bersentuhan dengan data pribadi[^5][^2].

Tabel 16 menyajikan kontrol governance yang disarankan.

Tabel 16. Kontrol governance

| Domain              | Kontrol                               | Penerapan                                           |
|---------------------|----------------------------------------|-----------------------------------------------------|
| Secrets             | No secrets in VCS, vault/secret manager | ENV injection, rotation schedule                    |
| Approval Workflow   | PR approvals, change advisory board    | Branch protection, code owners                      |
| Audit               | Structured logging, trace IDs          |centralized log platform, tamper‑evident storage    |
| Data Masking        | PII redaction, tokenization            | Pre‑processors, log filters                         |

Catatan kesenjangan informasi: pedoman governance lengkap di luar proyek开源 belum tersedia dalam sumber resmi; adopsi kebijakan internal yang kompatibel.

## 11. Lampiran & Referensi Cepat

Lampiran ini menyediakan beberapa quick reference untuk mempercepat eksekusi harian tim.

Tabel 17. CLI "metagpt" — opsi dan deskripsi ringkas

| Opsi                    | Deskripsi                                                       |
|-------------------------|-----------------------------------------------------------------|
| idea                    | Persyaratan satu baris (wajib)                                 |
| --investment            | Anggaran simulasi tim                                           |
| --n-round               | Jumlah putaran eksekusi SOP                                    |
| --code-review           | Aktifkan review kode                                            |
| --run-tests             | Aktifkan fase unit tests                                        |
| --implement             | Aktifkan implementasi kode                                      |
| --project-name          | Nama proyek unik                                                 |
| --inc                   | Mode inkremental untuk repo existing                            |
| --project-path          | Direktori proyek untuk iterasi                                  |
| --reqa-file             | Sumber untuk rewrite jaminan kualitas                           |
| --max-auto-summarize-code | Batas auto summarize                                            |
| --recover-path          | Pemulihan proyek dari state tersimpan                           |
| --init-config           | Inisialisasi file konfigurasi                                   |

Tabel 18. Konfigurasi provider LLM — ringkasan parameter

| Provider          | api_type   | Parameter Wajib                      | Catatan |
|-------------------|------------|--------------------------------------|---------|
| OpenAI            | openai     | api_key, model                       | Base URL mengikuti konfigurasi |
| Azure OpenAI      | azure      | api_key, model (deployment)          | API version diset di endpoint |
| Ollama            | ollama     | (biasanya tanpa key)                 | Lokal/on‑prem                 |
| Groq              | groq       | api_key, model                       | Throughput tinggi             |

Template kontrak SOP & agent — gunakan Tabel 8 sebagai dasar, pastikan setiap Action/Role memiliki deskripsi input/output dan quality gate yang jelas.

---

## Catatan Kesenjangan Informasi & Penyesuaian Enterprise

- Environment production: parameter spesifik Docker image tags, resource requests/limits, dan probe readiness/liveness belum tersedia di dokumentasi resmi. Gunakan baseline yang disarankan dan kalibrasi sesuai beban aktual.
- Testing lintas‑repo: belum ada standar resmi untuk golden test suites dan manajemen test data; tetapkan policy internal untuk menjaga konsistensi regress test MetaGPT.
- Governance di luar开源: kebijakan DLP/PII redaction dan approval workflows perlu disusun tim keamanan dancompliance perusahaan.
- Pipeline YAML production‑grade: rujuk GitHub Actions best practices dan sesuaikan dengan SOP MetaGPT serta kontrol compliance internal.
- Dokumentasi instrumentasi spesifik MetaGPT (span/attribute naming) belum tersedia; gunakan OpenTelemetry generik dan konvensi internal.
- Standar penulisan log structure belum tersedia; gunakan struktur JSON + TraceID/RunID dan putuskan kebijakan masking PII.

---

## Referensi

[^1]: MetaGPT: The Multi-Agent Framework - Introduction. https://docs.deepwisdom.ai/main/en/guide/get_started/introduction.html  
[^2]: FoundationAgents/MetaGPT: The Multi-Agent Framework - GitHub. https://github.com/FoundationAgents/MetaGPT  
[^3]: What is MetaGPT ? | IBM. https://www.ibm.com/think/topics/metagpt  
[^4]: MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework (ICLR 2024). https://proceedings.iclr.cc/paper_files/paper/2024/file/6507b115562bb0a305f1958ccc87355a-Paper-Conference.pdf  
[^5]: Configuration - MetaGPT. https://docs.deepwisdom.ai/main/en/guide/get_started/configuration.html  
[^6]: Setup - MetaGPT. https://docs.deepwisdom.ai/main/en/guide/get_started/setup.html  
[^7]: config2.example.yaml (MetaGPT). https://github.com/geekan/MetaGPT/blob/main/config/config2.example.yaml  
[^8]: config2.py (MetaGPT). https://github.com/geekan/MetaGPT/blob/main/metagpt/config2.py  
[^9]: Quickstart - MetaGPT. https://docs.deepwisdom.ai/main/en/guide/get_started/quickstart.html  
[^10]: Incremental Development - MetaGPT. https://docs.deepwisdom.ai/v0.6/en/guide/in_depth_guides/incremental_development.html  
[^11]: MetaGPT software_company.py (contoh eksekusi satu baris). https://github.com/geekan/MetaGPT/blob/main/metagpt/software_company.py  
[^12]: Agent 101 - MetaGPT. https://docs.deepwisdom.ai/main/en/guide/tutorials/agent_101.html  
[^13]: Create and Use Tools - MetaGPT. https://docs.deepwisdom.ai/main/en/guide/tutorials/create_and_use_tools.html  
[^14]: Automating and deploying workflows with GitHub Actions - GitHub Resources. https://resources.github.com/learn/pathways/automation/essentials/automating-deploying-workflows-with-github-actions/  
[^15]: Best practices for CI/CD monitoring - Datadog. https://www.datadoghq.com/blog/best-practices-for-ci-cd-monitoring/  
[^16]: The Complete Guide to CI/CD Pipeline Monitoring - Splunk. https://www.splunk.com/en_us/blog/learn/monitoring-ci-cd.html  
[^17]: AI Agent Observability - OpenTelemetry Blog (2025). https://opentelemetry.io/blog/2025/ai-agent-observability/  
[^18]: Agent Factory: Top 5 agent observability best practices for reliable AI - Microsoft Azure Blog. https://azure.microsoft.com/en-us/blog/agent-factory-top-5-agent-observability-best-practices-for-reliable-ai/  
[^19]: AI agent observability: Here's what you need to know - Merge.dev. https://www.merge.dev/blog/ai-agent-observability