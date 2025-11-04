# Perbandingan Komprehensif Framework AI Multi-Agent (2025): AutoGPT, CrewAI, LangGraph, AutoGen, MetaGPT, Swarm/Agents SDK, Semantic Kernel, dan LlamaIndex Agents

## Ringkasan Eksekutif

Pengelolaan aplikasi berbasis model bahasa besar (LLM) yang semakin kompleks mendorong adopsi kerangka kerja multi-agent. Delapan kerangka kerja yang dibandingkan—AutoGPT, CrewAI, LangGraph, AutoGen, MetaGPT, Swarm/OpenAI Agents SDK, Semantic Kernel, dan LlamaIndex Agents—mewakili spektrum pendekatan: dari orkestrasi berbasis graf berstateful hingga percakapan multi-agent berbasis peristiwa, dari simulasi tim SOP hingga toolkit ringan untuk ergonomis pengembangan. Temuan kunci kami: tidak ada satu kerangka kerja yang unggul di semua dimensi. Pemilihan harus ditentukan oleh nature of workload (alur bercabang vs percakapan konkuren), kebutuhan statefulness dan observabilitas, serta konteks enterprise (keamanan, integrasi, dan governansi).[^1][^2][^3]

Rekomendasi singkat per use case:
- Riset-riset kompleks dan authoring multi-tahap: LangGraph untuk kontrol DAG, percabangan, dan human-in-the-loop (HITL); LlamaIndex untuk pipeline bertumpu pada Retrieval-Augmented Generation (RAG) yang kuat.[^5][^18][^19]
- Automasi prosedural dan tim berbasis peran: CrewAI untuk orchestration crews/flows dengan guardrails, memori, dan integrasi enterprise; MetaGPT bila alur SOP perangkat lunak perlu disimulasikan end‑to‑end.[^11][^13][^15]
- Kolaborasi percakapan konkuren dan integrasi event-driven: AutoGen (event-driven Core/AgentChat) untuk pola dialog multi-agent dinamis; Semantic Kernel untuk orkestrasi "skill" enterprise dengan dukungan multi-bahasa.[^8][^9][^21]
- Integrasi erat dengan ekosistem OpenAI serta readiness produksi: Agents SDK (evolusi produksi dari Swarm) dengan primitif agent/handoffs/guardrails/sessions/tracing; Swarm cocok untuk edukasi/eksperimen saja.[^3][^4]
- RAG-sentris dan knowledge-intensive: LlamaIndex Agents (AgentWorkflow, Orchestrator, Custom Planner) untuk menyeimbangkan kemudahan dan fleksibilitas serta mengefektifkan pengindeksan data.[^18][^19]

## Metodologi & Ruang Lingkup Perbandingan

Kerangka kerja dievaluasi menurut lima kriteria: (1) fitur dan capabilities inti (agent model, tool calling, memory/state, guardrails, HITL, planning/orchestration, observability), (2) kemudahan penggunaan dan kurva belajar, (3) performance dan skalabilitas (stateful vs stateless, streaming, persistensi, pola scale-out), (4) dukungan komunitas dan ekosistem (lisensi, dokumentasi, integrasi), serta (5) use case optimal. Sumber data berupa dokumentasi resmi, repositori GitHub, artikel teknis/review tepercaya, dan satu publikasi akademik tentang protokol agentik dan desain arsitektur.[^1][^2]

Limitasi: belum ada benchmark lintas framework yang seragam dan tepercaya untuk latensi/throughput/biaya; data metrik komunitas (mis. GitHub stars, frekuensi release) tidak diekstrak numerik dalam studi ini; beberapa fitur enterprise (mis. deployment/observability di luar ekosistem tertentu) tidak terdokumentasi uniformes; detail internal MetaGPT mengenai scaling/disaster recovery dan AutoGPT membutuhkan verifikasi lebih lanjut dari sumber primer.高齢

## Pemetaan Framework & Arsitektur Inti

Perbedaan paradigma orkestrasi menjadi pembeda utama. LangGraph meng落地kan alur berbasis graf berstateful dengan kontrol eksplisit dan persistensi; AutoGen menyajikan percakapan multi-agent berbasis peristiwa dan eksekusi terdistribusi; CrewAI menerapkan tim berbasis peran dengan orchestration crews/flows; LlamaIndex menyediakan tiga pola multi-agent (AgentWorkflow, Orchestrator sebagai tool, Custom Planner); Swarm/Agents SDK mengeksplorasi/menawarkan primitif ringan agent/handoffs/guardrails/sessions; Semantic Kernel menekankan orkestrasi skill dengan planner multi-langkah; MetaGPT menyimulasikan tim perangkat lunak dengan SOP.[^5][^8][^11][^18][^3][^21][^13]

Untuk mengilustrasikan perbedaan paradigma, Tabel 1 memetakan arsitektur inti dan implikasinya.

Tabel 1 — Matriks paradigma arsitektur dan implikasi desain
| Framework | Paradigma orkestrasi | Statefulness | Human-in-the-loop | Streaming | Implikasi utama |
|---|---|---|---|---|---|
| LangGraph | Graf (DAG) berstateful | Ya (persistensi native) | Kuat (interrupts, persetujuan) | Ya (token & step) | Kontrol alur eksplisit, percabangan, retry, time-travel |
| AutoGen | Percakapan/event-driven | Konteks dialog (AgentChat) | Didukung (via design patterns) | Tidak disebutkan (khusus) | Konkurensi asinkron, jaringan agen terdistribusi |
| CrewAI | Tim berbasis peran (crews/flows) | Ya (memory/knowledge) | Ya (guardrails, callbacks) | Tidak disebutkan (umum) | Orkestrasi prosedural dan enterprise tooling |
| LlamaIndex Agents | RAG + multi-agent patterns | Bervariasi (Workflow/Context) | via planner or orchestrator | Event streaming tersedia | Fleksibilitas naik-turun antara kemudahan dan kontrol |
| Swarm (eksperimen) | Handoffs ringan (stateless) | Tidak (stateless) | Tidak (fokus edukasi) | Ya | Kontrolabilitas tinggi untuk eksperimen, bukan produksi |
| Agents SDK (produksi) | Agent/handoffs/sessions | Ya (sessions) | via guardrails | Tidak disebutkan (umum) | Primitif modern produksi, integrasi ekosistem OpenAI |
| Semantic Kernel | Orkestrasi skill + planner | Ya (modular, memory) | via orchestration patterns | Tidak disebutkan (umum) | Enterprise readiness (.NET/Java/Python) |
| MetaGPT | SOP tim perangkat lunak | Tertentu (peran & output) | via human-in-the-loop opsional | Tidak disebutkan (umum) | End-to-end software SOP (PM/Architect/Eng/QA) |

Sumber: dokumentasi resmi dan review komparatif terverifikasi.[^5][^8][^11][^18][^3][^21][^13]

## Perbandingan Fitur & Capabilities

### Agent model & tool integration

Semua kerangka kerja membangun "agent" sebagai komponen inti, namun mekanisme eksekusi tool dan integrasi vary. LangGraph beroperasi sebagaiprimitif rendah dalam ekosistem LangChain, sehingga tool integration mengalir dari LCEL/LangChain; AutoGen menyediakan AgentChat (percakapan) dan Core (event-driven), dengan Extensions untuk eksekusi eksternal seperti Docker dan gRPC; CrewAI memiliki desain agen yang menambahkan tools, memori, pengetahuan, dan output terstruktur (Pydantic); LlamaIndex Agents mendukung tool calling via pola Orchestrator dan Custom Planner; Swarm mengekspresikan tool sebagai fungsi Python yang dipetakan otomatis ke skema JSON; Agents SDK menyediakan Function Tools dengan validasi Pydantic; Semantic Kernel berfokus pada "skill" sebagai unit composable, dipanggil function-calling style; MetaGPT memproduksi SOP yang dapat menimbulkan rencana, kode, dan dokumen, dengan dukungan tool/LLM configurable.[^6][^8][^12][^18][^4][^3][^21][^15]

Tabel 2 — Komponen dan integrasi tool
| Framework | Komponen inti | Tool integration | Validasi skema |
|---|---|---|---|
| LangGraph | Graph nodes/edges, state | via LangChain/LangSmith | via LangChain/Pydantic (umum) |
| AutoGen | Studio, AgentChat, Core, Extensions | Extensions: Docker, gRPC, MCP | via ekstensi/ECS |
| CrewAI | Agents, Crews, Flows | 700+ integrasi enterprise, Pydantic output | Pydantic |
| LlamaIndex Agents | AgentWorkflow, Orchestrator, Planner | Fungsi/sub-agen sebagai tool | via pola implementatif |
| Swarm | Agent, handoffs | Fungsi Python → tools JSON | otomatis via docstring/type hints |
| Agents SDK | Agent loop, function tools | Fungsi Python → tools (Pydantic) | Pydantic |
| Semantic Kernel | Skill, planner | Function calling & plugin | via SDK |
| MetaGPT | SOP tim (PM/Architect/Eng/QA) | Konfigurasi LLM & tools | sesuai desain SOP |

### Memory & state management

Statefulness mempengaruhi UX dan governance. Studi akademik menunjukkan variasi dukungan memori (short-term, long-term, semantic, episodic, procedural). LangGraph menyimpan state per node/transisi, cocok untuk konteks jangka panjang dan "time-travel"; AutoGen mempertahankan konteks dialog multi-agent; CrewAI menambahkan memori dan pengetahuan tingkat agen; LlamaIndex menggunakan konteks workflow dan pengambilan berbasis embedding; Swarm stateless; Agents SDK menambahkan sessions; Semantic Kernel memiliki modul memori yang extensible; MetaGPT bekerja dengan memori implisit berbasis peran.[^1][^5][^8][^11][^18][^3][^21][^15]

Tabel 3 — Dukungan memori (ringkasan)
| Framework | Short-term | Long-term | Semantic | Episodic | Procedural |
|---|---|---|---|---|---|
| LangGraph | Ya | Tidak | Tidak | Tidak | Tidak |
| AutoGen | Ya | Ya | Tidak | Ya | Tidak |
| CrewAI | Ya | Ya | Ya | Ya | Tidak |
| LlamaIndex | Ya | Ya | Ya | Tidak | Tidak |
| Swarm | Ya (per call) | Tidak | Tidak | Tidak | Tidak |
| Agents SDK | Ya (sessions) | Tidak | Tidak | Tidak | Tidak |
| Semantic Kernel | Ya | Ya | Ya | Tidak | Ya |
| MetaGPT | Ya | Ya | Ya | Tidak | Ya |

Sumber: ringkasan studi akademik tentang memori agentik.[^1]

### Guardrails, planning, observability

Guardrails krusial untuk produksi. Agents SDK menambahkan validasi input/output via guardrails; LangGraph mendukung pemeriksaan tingkat alur, serta moderasi/loop kualitas; CrewAI menyediakan guardrails, callbacks, dan Enterprise features (RBAC, triggers); AutoGen menawarkan validator dan logika percobaan ulang; observability/langsmith terintegrasi ke LangGraph; LlamaIndex dan Semantic Kernel memerlukan penerapan tambahan/eksternal untuk penegakan guardrails yang kuat; Swarm minim karena fokus edukasi.[^3][^5][^11][^8][^1][^21]

## Ease of Use & Learning Curve

Dalam praktik, выбор зависит от уровня abstraksi yang diinginkan. LlamaIndex AgentWorkflow meminimalkan kode untuk behavior multi-agent dengan heuristik hand-off default—ideal untuk prototipe cepat. Orchestraor (sub-agen sebagai tool) menambah kontrol tanpa menulis planner sendiri. Custom Planner memberi fleksibilitas maksimum, namun memerlukan prompting, parsing, dan eksekusi imperatif yang lebih matang.[^18] CrewAI menyederhanakan composable agents + crews/flows dengan modul memori/guardrails/observability dan Enterprise tooling, mempercepat adopsi tim produk.[^11] Swarm berbasis Agent/handoffs yang ringkas, stateless, dan mudah diuji, cocok untuk edukasi pola koordinasi; tetapi bukan untuk produksi dan perlu migrasi ke Agents SDK.[^4] LangGraph powerful namun memiliki kurva belajar lebih tinggi karena konsep graf, node/edge, dan state management; kompleksitas sebanding dengan kontrol yang didapat.[^5]

Tabel 4 — Indikator DX dan kebutuhan implementasi
| Framework | Kebutuhan setup | Kompleksitas implementasi | Abstraksi | Tooling DX | Kesesuaian pemula |
|---|---|---|---|---|---|
| LlamaIndex (AgentWorkflow) | Rendah | Rendah | Tinggi | Baik | Sangat cocok |
| CrewAI | Rendah–sedang | Sedang | Tinggi | Kuat (Studio/flows) | Baik |
| Swarm | Rendah | Rendah | Rendah | Minimal | Cocok untuk edukasi |
| Agents SDK | Rendah–sedang | Sedang | Sedang | Kuat (tracing/sessions) | Baik |
| LangGraph | Sedang | Sedang–tinggi | Rendah–sedang | Sangat baik (Studio/LangSmith) | Perlu effort |
| AutoGen | Sedang | Sedang | Sedang | Baik (Extensions) | Baik |
| Semantic Kernel | Sedang | Sedang | Sedang | Baik (multi-bahasa) | Baik |
| MetaGPT | Sedang | Sedang–tinggi | Tinggi (SOP) | Baik | Perlu konteks domain |

Sumber: dokumentasi resmi dan panduan komparatif DX.[^18][^11][^4][^5]

## Performance & Scalability

Dua aspek kunci: bagaimana runtime mengelola state (persistent vs session vs stateless) dan bagaimana aplikasi diskalakan (retry, persistensi, distribusi). LangGraph dirancang untuk stateful workflows dengan server yang skalabel horizontal, antrian tugas, persistensi bawaan, caching cerdas, dan retry otomatis—cocok untuk alur panjang dengan pengawasan kualitas; studi kasus menunjukkan waktu respons yang wajar untuk interaksi multi-agent kompleks (contoh rata-rata sekitar 5,95 detik, tergantung konteks tugas).[^5][^7] AutoGen Core mendukung sistem multi-agent berbasis peristiwa dan eksekusi terdistribusi; cocok untuk konkurensi asinkron dan beban dialog dinamis.[^8] Semantic Kernel menyediakan orkestrasi enterprise dan modularitas untuk skala lintas bahasa.[^20][^21] Agents SDK menyediakan agent loop, guardrails, dan sessions untuk kontrol produksi yang ergonomis; Swarm eksperimental dan stateless, membuat scaling produksi tidak direkomendasikan.[^3][^4]

Tabel 5 — Pola skalabilitas dan ketahanan
| Framework | State persistensi | Distribusi/konkurensi | Retry/caching | Streaming |
|---|---|---|---|---|
| LangGraph | Bawaan (post-step) | Server scale-out, antrian | Retry & caching | Ya (token & step) |
| AutoGen | Konteks dialog | Event-driven, gRPC/ECS | via logika aplikasi | Tidak disebutkan |
| CrewAI | Memory/knowledge | Orkestrasi crews/flows | Guardrails/callbacks | Tidak disebutkan |
| LlamaIndex | Workflow/Context | via orchestrator/planner | via implementasi | Event streaming |
| Swarm | Stateless | Tidak (eksperimen) | Tidak | Ya |
| Agents SDK | Sessions | Loop agen, handoffs | Guardrails (validasi) | Tidak disebutkan |
| Semantic Kernel | Modular/memory | Orkestrasi skill | via integrasi | Tidak disebutkan |
| MetaGPT | SOP-based | Tidak dioptimalkan runtime | via implementasi | Tidak disebutkan |

Sumber: dokumentasi resmi.[^5][^7][^8][^3][^4][^20][^21]

Catatan: belum ada dataset benchmark lintas framework yang seragam dan terverifikasi (latensi/throughput/biaya). Organisasi disarankan menjalankan pengukuran internal yang terkontrol (misalnya workload riset/analitik data) sesuai saran studi performa praktis.[^2]

## Dukungan Komunitas & Ekosistem

Dari sisi lisensi, beberapa framework bersifat open-source dengan lisensi permisif—misalnya LangGraph (MIT) dan Swarm (MIT). Dokumentasi dan ekosistem bervariasi: LangGraph terintegrasi dengan LangSmith untuk observability dan deployment (cloud, hybrid, self-hosted); CrewAI menyediakan Enterprise tooling (RBAC, triggers, integrasi Gmail/Drive/Outlook/Teams/HubSpot); Semantic Kernel memiliki dukungan lintas bahasa (.NET/Python/Java) dan orientasi enterprise; LlamaIndex kuat pada tooling RAG dan multi-agent patterns; Agents SDK mendapat dukungan resmi OpenAI dan ekosistem инструментов; AutoGen didukung Microsoft Research dan AWS Prescriptive Guidance; MetaGPT memiliki GitHub dan dokumentasi yang aktif.[^5][^11][^21][^18][^3][^8][^15][^16]

Tabel 6 — Ekosistem & dukungan
| Framework | Lisensi (indikatif) | Observability | Integrasi | Platform/ekstensi | Dukungan enterprise |
|---|---|---|---|---|---|
| LangGraph | MIT | LangSmith | LangChain | Cloud/hybrid/self-host | Ada (LangSmith Enterprise) |
| CrewAI | OSS | Built-in | 700+ apps | Enterprise console | Kuat (RBAC, triggers) |
| Semantic Kernel | OSS | via SDK/OTel | Azure/plugins | .NET/Python/Java | Kuat |
| LlamaIndex | OSS | via tooling | Data/RAG | Python | Umum |
| AutoGen | OSS | via Extensions | AWS/ECS | Python/.NET | Ada (docs AWS) |
| Agents SDK | OSS | Tracing built-in | OpenAI | Python | Ada (ekosistem OpenAI) |
| Swarm | MIT | Minimal | Eksperimen | Python | Tidak (eksperimental) |
| MetaGPT | OSS | via logging | Configurable LLM/tools | Python | Umum |

Sumber: dokumentasi resmi dan review ekosistem.[^5][^11][^21][^18][^3][^8][^15][^16]

## Use Cases Optimal per Framework

Riset & authoring multi-tahap. LangGraph unggul pada DAG bercabang dengan kontrol statefulness, HITL, dan evaluasi kualitas; cocok untuk pipeline yang mengharuskan review manusia, percabangan, dan retry. LlamaIndex AgentWorkflow/Orchestrator efektif untuk flow riset→penulisan→review dalam konteks knowledge-intensive berbasis RAG.[^5][^18][^19]

Automasi prosedural & tim peran. CrewAI mengorkestrasi "crews" dan "flows" berbasis peran dengan guardrails, memori, dan integrasi enterprise, ideal untuk proses bisnis terstruktur. MetaGPT menyimulasikan SOP perangkat lunak (product manager, architect, engineer, QA) untuk mengubah requirement satu baris menjadi artefak terstruktur.[^11][^13][^15]

Kolaborasi percakapan konkuren. AutoGen memungkinkan pola dialog multi-agent dinamis, pesan asinkron, dan penggunaan Extensions untuk eksekusi terdistribusi. Semantic Kernel memfasilitasi orkestrasi "skill" enterprise dengan planner dan integrasi multi-bahasa.[^8][^21][^20]

Integrasi OpenAI & produksi ringan. Agents SDK mengemas primitif produksi (agent/handoffs/guardrails/sessions/tracing) dan integrasi native dengan toolchain OpenAI; Swarm cocok untuk pembelajaran pola handoff dan uji ergonomis, tetapi bukan untuk produksi.[^3][^4]

RAG-sentris. LlamaIndex Agents menonjol dalam mengambil dan menggabungkan pengetahuan dari berbagai sumber data, dengan pola multi-agent yang fleksibel sesuai kebutuhan kontrol.[^18][^19]

Tabel 7 — Pemetaan framework ke use case
| Use case | Kebutuhan kunci | Rekomendasi framework | Catatan |
|---|---|---|---|
| Riset/penulisan multi-tahap | DAG, HITL, retry | LangGraph; LlamaIndex | Kontrol stateful vs RAG-fleksibel |
| Automasi prosedural | Roles, guardrails | CrewAI; MetaGPT | SOP tim & enterprise tooling |
| Percakapan konkuren | Event-driven, asinkron | AutoGen; Semantic Kernel | Jaringan agen vs orkestrasi skill |
| Integrasi OpenAI | Guardrails, sessions | Agents SDK | Migrasi dari Swarm eksperimental |
| RAG-sentris | Pengindeksan, retrieval | LlamaIndex Agents | Pola AgentWorkflow/Orchestrator |
| Edukasi/eksperimen | Ringan, handoffs | Swarm | Stateless, belajar pola koordinasi |

Sumber: dokumentasi resmi dan review komparatif.[^5][^11][^8][^3][^4][^18][^19][^13][^15]

## Rekomendasi Strategis & Decision Matrix

Jika workloads Anda didominasi alur bercabang dengan kebutuhan auditability yang tinggi, pilih LangGraph. Bila kebutuhan utama adalah integrasi data privat/umum dengan RAG, LlamaIndex Agents paling relevan. Untuk organisasi .NET/Java yang menekankan integrasi enterprise dan kepatuhan, Semantic Kernel adalah kandidat utama. Bila Anda membutuhkan percakapan multi-agent konkuren yang asinkron, AutoGen bernilai. Jika pipeline tim berbasis peran dengan guardrails enterprise adalah prioritas, gunakan CrewAI; sementara simulasi SOP perangkat lunak end‑to‑end dipertimbangkan untuk MetaGPT. Untuk tim yang intensif pada ekosistem OpenAI dan memerlukan readiness produksi, Agents SDK menjadi opsi utama—Swarm hanya untuk edukasi.[^1][^2][^22]

Tabel 8 — Decision matrix (kebutuhan → framework)
| Kebutuhan | Framework yang disarankan | Alasan ringkas |
|---|---|---|
| Kontrol alur DAG, HITL, auditability | LangGraph | Statefulness dan orkestrasi graf |
| RAG-sentris, knowledge fusion | LlamaIndex Agents | Tooling RAG + pola fleksibel |
| Enterprise lintas bahasa (.NET/Java) | Semantic Kernel | Orkestrasi skill & compliance |
| Percakapan konkuren asinkron | AutoGen | Event-driven, pesan asinkron |
| Tim peran + guardrails enterprise | CrewAI | Crews/flows + integrasi enterprise |
| SOP perangkat lunak end‑to‑end | MetaGPT | Simulasi tim SOP |
| Ekosistem OpenAI + produksi | Agents SDK | Guardrails, sessions, tracing |
| Edukasi handoff & uji ergonomis | Swarm | Stateless, eksperimental |

Sumber: telaah arsitektur dan kapabilitas relatif.[^1][^2][^22]

## Risiko, Keterbatasan, dan Praktik Terbaik

- Stateless vs stateful. Swarm stateless antar panggilan, cocok untuk edukasi tetapi kurang untuk produksi yang butuh kelangsungan state; Agents SDK menambahkan sessions untuk弥补.[^4][^3]
- Guardrails dan security. Validasi input/output, pemblokiran aksi berisiko, dan sandbox eksekusi kode (mis. Docker di AutoGen Extensions) wajib diterapkan; Risiko keamanan meningkat saat agen mengeksekusi kode atau mengakses sistem file—selalu sandbox dan batasi permission.[^8]
- Kompleksitas state & loop tak terbatas. Desain graf dan memory yang tidak tepat dapat menimbulkan loop tak berujung atau divergensi; gunakan monitor eksternal, validasi tingkat alur (LangGraph), dan guardrails (Agents SDK/CrewAI) untuk mencegah perilaku menyimpang.[^5][^3][^11]
- Observability & governance. Rekam jejak (tracing), metrik, dan audit trail adalah "muslihat" untuk produksi. Integrasi LangSmith/Langfuse dan logging agentik membantu diagnosis dan audit; tetapkan kebijakan akses berbasis peran (RBAC) dan kontrol jalur data sesuai kebutuhan enterprise.[^5][^1]
- Interoperabilitas dan protokol. Fragmentasi abstraksi lintas framework menghambat reuse alat dan portabilitas; adopsi prinsip SOA dan protokol agentik modern (mis. MCP, A2A, ANP) dapat mengurangi silo, namun standardisasi lintas vendor masih berkembang.[^1]

## Langkah Implementasi Cepat (Quick Starts) & Sumber Belajar

- CrewAI: Mulai dari Mendesain Agen, Orkestrasi Crews/Flows, dan mengaktifkan guardrails/memori; gunakan Studio/Enterprise console untuk deployment dan observability.[^11]
- LlamaIndex Agents: Gunakan AgentWorkflow untuk prototipe; naik ke Orchestrator untuk kontrol lebih; terapkan Custom Planner hanya saat dua pola pertama tak memadai.[^18]
- AutoGen: Jelajahi AgentChat untuk percakapan multi-agent dan Core untuk event-driven sistem terdistribusi; gunakan Extensions (mis. Docker) untuk eksekusi aman.[^8]
- LangGraph: Bangun graf dengan node/edges, definisikan state dan interrupts untuk HITL, aktifkan streaming untuk UX; gunakan Studio/LangSmith untuk debugging dan deployment.[^5]

Gunakan repositori GitHub dan dokumentasi resmi sebagai rujukan praktis untuk contoh dan update terbaru.[^10][^12][^9][^6]

## Lampiran

Glosarium singkat:
- Agent: komponen yang menjalankan tindakan berdasarkan instruksi dan alat (tools).
- Handoff: mekanisme serah terima kendali/percakapan antar agen.
- Guardrails: validasi/pembatas untuk mencegah aksi di luar kebijakan.
- Memory: kemampuan menyimpan/memanggil konteks (short-/long-term, episodic, semantic, procedural).
- Planner: komponen yang menetapkan rencana tindakan multi-langkah.
- Orchestrator: agen yang mengatur eksekusi sub-agen sebagai alat.
- Workflow: kumpulan langkah/状态的 yang mengatur eksekusi aplikasi agentik.

Catatan tentang kesenjangan informasi:
- Belum tersedia benchmark lintas framework yang seragam dan terverifikasi (latensi/throughput/biaya).
- Metrik komunitas (stars/release cadence) tidak dirinci numerik dalam studi ini.
- Detail performa produksi untuk beberapa framework (mis. AutoGen, CrewAI) memerlukan studi kasus tambahan.
- Fitur enterprise non-tekstual (deployment/observability) membutuhkan verifikasi sumber primer.
- MetaGPT: detail scaling dan disaster recovery serta AutoGPT (dokumentasi/arsitektur) memerlukan rujukan primer tambahan.

---

## References

[^1]: Agentic AI Frameworks: Architectures, Protocols, and Design (arXiv, 2025). https://arxiv.org/html/2508.10146v1  
[^2]: Agentic AI Framework Benchmarks & Performance (AIMultiple). https://aimultiple.com/agentic-ai-frameworks/  
[^3]: OpenAI Agents SDK Documentation. https://openai.github.io/openai-agents-python/  
[^4]: openai/swarm: Educational framework exploring ergonomic multi-agent orchestration (GitHub). https://github.com/openai/swarm  
[^5]: LangGraph - LangChain. https://www.langchain.com/langgraph  
[^6]: LangGraph OSS Documentation (Python). https://langchain-ai.github.io/langgraph/  
[^7]: Evaluate and Improve LangGraph Multi-Agent System (Galileo AI). https://galileo.ai/blog/evaluate-langgraph-multi-agent-telecom  
[^8]: AutoGen Documentation (Stable). https://microsoft.github.io/autogen/stable//index.html  
[^9]: microsoft/autogen (GitHub). https://github.com/microsoft/autogen  
[^10]: AutoGen - AWS Prescriptive Guidance. https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-frameworks/autogen.html  
[^11]: CrewAI Documentation. https://docs.crewai.com/  
[^12]: crewAIInc/crewAI (GitHub). https://github.com/crewAIInc/crewAI  
[^13]: What is MetaGPT? (IBM). https://www.ibm.com/think/topics/metagpt  
[^14]: FoundationAgents/MetaGPT (GitHub). https://github.com/FoundationAgents/MetaGPT  
[^15]: MetaGPT Documentation: Introduction. https://docs.deepwisdom.ai/main/en/guide/get_started/introduction.html  
[^16]: Introducing llama-agents (LlamaIndex Blog). https://www.llamaindex.ai/blog/introducing-llama-agents-a-powerful-framework-for-building-production-multi-agent-ai-systems  
[^17]: Agents | LlamaIndex Python Documentation. https://developers.llamaindex.ai/python/framework/use_cases/agents/  
[^18]: Multi-agent patterns in LlamaIndex. https://developers.llamaindex.ai/python/framework/understanding/agent/multi_agent/  
[^19]: Semantic Kernel Agent Framework | Microsoft Learn. https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/  
[^20]: Semantic Kernel: Multi-agent Orchestration (DevBlogs). https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-multi-agent-orchestration/  
[^21]: microsoft/semantic-kernel (GitHub). https://github.com/microsoft/semantic-kernel  
[^22]: Comparing Open-Source AI Agent Frameworks (Langfuse Blog). https://langfuse.com/blog/2025-03-19-ai-agent-comparison  
[^23]: Best 5 Frameworks To Build Multi-Agent AI Applications (GetStream Blog). https://getstream.io/blog/multiagent-ai-frameworks/  
[^24]: New tools for building agents (OpenAI). https://openai.com/index/new-tools-for-building-agents/