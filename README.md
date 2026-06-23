# Graf Politikus Indonesia — Tier 4

> Knowledge Graph + LLM Pipeline: Text-to-Cypher | Graph Builder | GraphRAG

[![Python](https://img.shields.io/badge/Python-3.10-blue)](https://python.org)
[![Neo4j](https://img.shields.io/badge/Neo4j-AuraDB-green)](https://neo4j.com/cloud/platform/aura-graph-database/)
[![OpenRouter](https://img.shields.io/badge/LLM-OpenRouter-orange)](https://openrouter.ai)
[![Notebook](https://img.shields.io/badge/Platform-Google%20Colab-yellow)](https://colab.research.google.com)

---

## Deskripsi Proyek

Sistem knowledge graph berbasis Neo4j AuraDB yang memodelkan jaringan politik Indonesia secara komprehensif — mencakup **52.295 politikus**, **156 partai**, dan **881 institusi pendidikan**. Pipeline AI terintegrasi memungkinkan pengguna berinteraksi dengan graf melalui bahasa alami (Bahasa Indonesia), mengekstrak entitas dari teks berita, dan mendapatkan jawaban yang diperkaya konteks graf.

**Dataset:** ~52K+ nodes, multi-relasi (keanggotaan partai, alumni pendidikan, jaringan kerabat/dinasti)  
**Dataset:** Diambil dari github ,hasil dari query di wikipedia pada tugas ETS kemarin,anda bisa melihat dokumentasi dataset tersebut di repositori di bawah ini ,anda juga bisa import data langsung di neo4j menggunakan cypher tanpa menggunakan kode python dengan melihat dokumentasinya.
`https://github.com/ibrahimamar07/Graf-Data-Politikus-Indonesia`

---

## Arsitektur Sistem

![ImagePipeline](./imagepipeline.png)

### Stack Teknologi

| Layer           | Teknologi                                         |
| --------------- | ------------------------------------------------- |
| Graph Database  | Neo4j AuraDB (cloud, free tier)                   |
| Graph Analytics | Neo4j GDS Plugin (PageRank, Betweenness, Louvain) |
| LLM Gateway     | OpenRouter API (free models)                      |
| LLM Model       | `nvidia/nemotron-3-ultra-550b-a55b:free`          |
| Runtime         | Python 3.10, Google Colab                         |
| Libraries       | `neo4j`, `requests`, `pandas`, `tqdm`             |

---

## Skema Graf

### Node Labels & Properties

```
Politikus   { kodeId: string, nama: string, pagerank: float,
              betweenness: float, community: int }

Partai      { kodeId: string, nama: string,
              pagerank: float, betweenness: float }

Pendidikan  { kodeId: string, nama: string }

Person      { kodeId: string, nama: string }

```

### Relationship Types

```
(Politikus)-[:ANGGOTA_PARTAI]->(Partai)
(Politikus)-[:ALUMNI]->(Pendidikan)
(Politikus)-[:KERABAT {tipe: string}]->(Politikus)
(Politikus)-[:KERABAT {tipe: string}]->(Person)
```

### Statistik Dataset

| Label      | Jumlah Nodes            |
| ---------- | ----------------------- |
| Politikus  | ~52.295                 |
| Partai     | ~156                    |
| Pendidikan | ~881                    |
| Person     | (kerabat non-politikus) |

---

## Instalasi

### 1. Buka di Google Colab

Upload file `Graf_Politikus_Indonesia.ipynb` ke Google Colab atau buka langsung dari Google Drive.

### 2. Install Dependencies

Jalankan **Cell 0** — instalasi otomatis:

```bash
pip install neo4j requests pandas tqdm
```

### 3. Siapkan Neo4j AuraDB

1. Buat akun di [console.neo4j.io](https://console.neo4j.io/)
2. Buat instance AuraDB Free
3. Catat: **URI**, **Username** (`neo4j`), **Password**

### 4. Siapkan OpenRouter API Key

1. Daftar di [openrouter.ai](https://openrouter.ai)
2. Generate API key (gratis, tidak perlu kartu kredit untuk model free)
3. Pilih model free yang tersedia:
   - `nvidia/nemotron-3-ultra-550b-a55b:free`
   - `meta-llama/llama-3.1-8b-instruct:free`
   - `google/gemma-2-9b-it:free`
   - `mistralai/mistral-7b-instruct:free`
   - `qwen/qwen-2-7b-instruct:free`

---

## Konfigurasi

Edit **Cell 1** sebelum menjalankan notebook:

```python
# Neo4j AuraDB Credentials
NEO4J_URI      = "bolt://<your-auradb-uri>"
NEO4J_USER     = "neo4j"
NEO4J_PASSWORD = "<your-password>"

# OpenRouter API
OPENROUTER_API_KEY = "sk-or-v1-<your-key>"

# Model LLM gratis
FREE_MODEL = "nvidia/nemotron-3-ultra-550b-a55b:free"
```

> ⚠️ **Keamanan:** Jangan commit file notebook dengan kredensial nyata ke repository publik. Gunakan `os.environ` atau Colab Secrets untuk produksi.

---

## Cara Menjalankan

Jalankan cell secara berurutan:

| Cell        | Fungsi                     | Estimasi Waktu  |
| ----------- | -------------------------- | --------------- |
| **Cell 0**  | Install dependencies       | ~1 menit        |
| **Cell 1**  | Konfigurasi credentials    | —               |
| **Cell 2**  | Koneksi Neo4j + test LLM   | ~10 detik       |
| **Cell 3**  | Import dataset (52K nodes) | **10–20 menit** |
| **Cell 4**  | Graph Analytics (GDS)      | ~5 menit        |
| **Cell 5**  | Demo Text-to-Cypher        | ~1 menit        |
| **Cell 6**  | Demo Graph Builder         | ~1 menit        |
| **Cell 7**  | Demo RAG Pipeline          | ~2 menit        |
| **Cell 8**  | Interactive Chatbot        | on-demand       |
| **Cell 9**  | Evaluasi sistem            | ~2 menit        |
| **Cell 10** | Custom query (opsional)    | —               |

> **⚠️ Cell 3 harus dijalankan sekali saja.** Jalankan ulang hanya jika database direset. Gunakan constraint `MERGE` yang sudah ada untuk idempotency.

---

## Penggunaan

### Text-to-Cypher (Komponen 1)

```python
# Tanya langsung ke graf dengan bahasa natural
ask_graph("Siapa 5 politikus dengan pagerank tertinggi?")
ask_graph("Partai apa yang memiliki anggota terbanyak?")
ask_graph("Politikus mana yang punya kerabat politikus paling banyak?")
ask_graph("Universitas mana yang paling banyak menghasilkan politikus?")
```

### Graph Builder (Komponen 2)

```python
# Ekstrak entitas dari teks berita dan masukkan ke Neo4j
berita = """
Megawati Soekarnoputri menghadiri rapat pimpinan PDI-P di Jakarta.
Ganjar Pranowo yang merupakan kader PDIP juga hadir dalam rapat tersebut.
"""
extracted = extract_entities_from_text(berita)
stats = populate_graph_from_extraction(extracted, berita)
print(stats)
```

### RAG Pipeline (Komponen 3)

```python
# Tanya dengan jawaban diperkaya konteks graf
rag_query("Siapa politikus paling berpengaruh dalam jaringan politik Indonesia?")
rag_query("Bagaimana pola dinasti politik di Indonesia?")
rag_query("Universitas mana yang paling banyak mencetak politikus?")
```

### Political Chatbot (Gabungan)

```python
# Mode RAG (direkomendasikan) — graph context + LLM answer
political_chatbot("Berikan analisis distribusi keanggotaan partai.", mode="rag")

# Mode Cypher — query langsung + ringkasan LLM
political_chatbot("Partai apa yang paling dominan?", mode="cypher")

# Mode Direct — LLM murni tanpa graph context
political_chatbot("Apa itu dinasti politik?", mode="direct")
```

### Direct Cypher Query

```python
# Query langsung ke Neo4j tanpa LLM
results = run_query("""
    MATCH (p:Politikus)-[:ANGGOTA_PARTAI]->(par:Partai)
    WHERE toLower(par.nama) CONTAINS 'demokrat'
    RETURN p.nama, par.nama
    LIMIT 20
""")
```

---

## Penjelasan Logika Cypher & Pipeline AI

### 1. Import Dataset (`LOAD CSV + MERGE`)

```cypher
-- Pola dasar import node dengan idempotency
LOAD CSV WITH HEADERS FROM '<github-raw-url>/nodes_politikus.csv' AS row
WITH row WHERE row.`kodeId:ID` IS NOT NULL AND row.`kodeId:ID` <> ''
MERGE (p:Politikus {kodeId: row.`kodeId:ID`})
SET p.nama = row.nama
```

**Logika:** `MERGE` memastikan node tidak duplikat meski query dijalankan berkali-kali. Filter `WHERE IS NOT NULL` mencegah node kosong terbuat dari baris CSV yang rusak.

```cypher
-- Pola import edge dengan property
LOAD CSV WITH HEADERS FROM '<url>/edges_kerabat_politikus.csv' AS row
WITH row WHERE row.`:START_ID` IS NOT NULL AND row.`:END_ID` IS NOT NULL
MATCH (a:Politikus {kodeId: row.`:START_ID`})
MATCH (b:Politikus {kodeId: row.`:END_ID`})
MERGE (a)-[:KERABAT {tipe: row.tipe}]->(b)
```

**Logika:** Double `MATCH` sebelum `MERGE` memastikan kedua endpoint node sudah ada sebelum relasi dibuat — menghindari dangling edge.

### 2. Graph Analytics (GDS)

**Kenapa Graph Projection?**  
Neo4j GDS bekerja pada in-memory projection, bukan langsung di disk. Projection memungkinkan algoritma berjalan jauh lebih cepat.

```cypher
-- Buat projection multi-label undirected
CALL gds.graph.project(
  'graf_politik',
  ['Politikus', 'Partai', 'Pendidikan', 'Person'],
  {
    ANGGOTA_PARTAI: {orientation: 'UNDIRECTED'},
    ALUMNI:         {orientation: 'UNDIRECTED'},
    KERABAT:        {orientation: 'UNDIRECTED'}
  }
)

-- PageRank: siapa yang paling banyak "dirujuk" dalam jaringan
CALL gds.pageRank.write('graf_politik', {
  maxIterations: 20, dampingFactor: 0.85,
  writeProperty: 'pagerank'
})

-- Betweenness: siapa yang jadi "jembatan" antar komunitas
CALL gds.betweenness.write('graf_politik', {writeProperty: 'betweenness'})

-- Louvain: deteksi komunitas/kluster secara otomatis
CALL gds.louvain.write('graf_politik', {writeProperty: 'community'})
```

Hasil ditulis kembali ke properti node (`writeProperty`) sehingga bisa di-query dengan Cypher biasa setelahnya.

### 3. Text-to-Cypher Pipeline

````
[Pertanyaan NL]
      │
      ▼ call_llm(TEXT_TO_CYPHER_SYSTEM, question)
[Raw Cypher String]
      │
      ▼ strip markdown fences (```cypher ... ```)
[Clean Cypher]
      │
      ▼ run_query(cypher)
[Result List]
      │
      ├─ OK → tampilkan hasil
      └─ Error → retry: kirim (query + error) ke LLM untuk self-repair
````

**System Prompt Strategy:** LLM diberi schema lengkap (node labels, properties, relationship types) + contoh few-shot Q→A. Ini mengurangi hallucination query dan memastikan nama properti (`kodeId`, `pagerank`, dll.) digunakan secara tepat. `temperature=0.1` dipakai agar output deterministik.

**Self-Repair:** Jika query pertama gagal di Neo4j, error message dikembalikan ke LLM sebagai konteks untuk perbaikan — satu kali retry otomatis sebelum menyerah.

### 4. Graph Builder Pipeline

```
[Teks Berita]
      │
      ▼ call_llm(GRAPH_BUILDER_SYSTEM, text, temperature=0.0)
[Raw JSON String]
      │
      ▼ regex: re.search(r'\{.*\}', raw, re.DOTALL)
[Parsed JSON]  {"entities": [...], "relasi": [...]}
      │
      ▼ populate_graph_from_extraction()
      │
      ├─ Setiap entity → MERGE (:BeritaEntity:<Tipe> {extractedId})
      └─ Setiap relasi → MATCH kedua node → MERGE relasi
```

**Kenapa `temperature=0.0`?** Ekstraksi entitas adalah tugas struktural — kita ingin output JSON yang konsisten dan valid, bukan kreatif. `temperature=0` memaksimalkan determinisme.

**Label Dinamis:** `safe_label = re.sub(r'[^A-Za-z]', '', tipe)` membersihkan tipe entitas dari karakter non-alfabet sebelum digunakan sebagai Neo4j label — mencegah Cypher injection dari output LLM.

### 5. RAG (Graph-Augmented Retrieval) Pipeline

```
[Pertanyaan User]
      │
      ▼ retrieve_graph_context(question)        ← keyword routing
      │   ├─ "berpengaruh/pagerank" → MATCH PageRank top-10
      │   ├─ "dinasti/kerabat"      → MATCH KERABAT network
      │   ├─ "partai/anggota"       → MATCH ANGGOTA_PARTAI count
      │   └─ "universitas/alumni"   → MATCH ALUMNI count
      │
      ▼ format_context_for_llm(contexts)
[Structured Graph Context String]
      │
      ▼ call_llm(RAG_SYSTEM, context + question, temperature=0.3)
[Augmented Answer]
```

**Keyword Routing vs. Semantic Search:** Sistem ini menggunakan rule-based keyword matching (bukan embedding similarity) untuk memilih query graf yang relevan. Pendekatan ini lebih cepat dan deterministik untuk domain yang terdefinisi — tapi kurang fleksibel untuk pertanyaan out-of-domain.

**Augmented Prompt Pattern:**

```
=== KONTEKS DARI KNOWLEDGE GRAPH ===
[Top Influencer data]
[Dinasti data]
[Partai data]
...

PERTANYAAN USER: ...
Jawab berdasarkan konteks di atas:
```

LLM hanya "mereformulasi" fakta dari graf — mengurangi risiko halusinasi dibandingkan menjawab langsung dari parametric memory.

---

## Struktur Repository

```
.
├── Graf_Politikus_Indonesia_Tier4.ipynb   # Notebook utama
├── README.md                              # Dokumentasi ini
└── dataset_load_to_neo4j/                 # (di GitHub repo dataset)
    ├── nodes_politikus.csv
    ├── nodes_partai.csv
    ├── nodes_pendidikan.csv
    ├── nodes_person.csv
    ├── edges_anggota_partai.csv
    ├── edges_alumni.csv
    ├── edges_kerabat_politikus.csv
    └── edges_kerabat_person.csv
```

Dataset CSV diambil langsung dari hasil pengumpulan data di wikidata:  
`https://github.com/ibrahimamar07/Graf-Data-Politikus-Indonesia`
`https://raw.githubusercontent.com/ibrahimamar07/Graf-Data-Politikus-Indonesia/main/dataset_load_to_neo4j/`

---

## Troubleshooting

| Masalah                                 | Penyebab                         | Solusi                                              |
| --------------------------------------- | -------------------------------- | --------------------------------------------------- |
| `Koneksi gagal`                         | URI/password AuraDB salah        | Cek credential di Neo4j Console                     |
| `LLM error: KeyError 'choices'`         | API key invalid / rate limit     | Ganti API key atau model                            |
| `GDS tidak tersedia`                    | Plugin tidak aktif               | Notebook otomatis fallback ke Cypher degree         |
| Query error setelah retry               | LLM generate query tidak valid   | Coba pertanyaan yang lebih spesifik                 |
| Import lambat                           | AuraDB free tier bandwidth limit | Normal, tunggu 10–20 menit                          |
| `JSON Parsing Error` pada Graph Builder | LLM output tidak murni JSON      | Dicatch dan return `{"entities": [], "relasi": []}` |

---

## Tier 4 Checklist

- [x] **LLM Text-to-Cypher** — Natural language → Cypher query yang valid
- [x] **LLM Graph Builder** — Ekstraksi entitas dari teks berita → Neo4j nodes/edges
- [x] **Graph Analytics** — PageRank, Betweenness Centrality, Louvain Community (GDS)
- [x] **RAG Pipeline** — Graph-Augmented Retrieval untuk jawaban faktual
- [x] **Neo4j AuraDB** — Cloud graph database dengan 50K+ nodes
- [x] **OpenRouter API** — Free LLM model via API
- [x] **Multi-entitas** — Politikus + Partai + Pendidikan + Person
- [x] **>50 nodes** — 52.295+ node Politikus saja

---

## Dokumentasi Penggunaan AI dalam Pengembangan Kode

Sesuai ketentuan mata kuliah, seluruh penggunaan AI generatif untuk menghasilkan kode dalam proyek ini didokumentasikan di bawah — mencakup prompt yang dipakai, model yang digunakan, output yang diterima, dan modifikasi manual yang dilakukan setelahnya.

---

### Cell 3 — Import Dataset ke Neo4j AuraDB

**Model:** `claude-sonnet-4-6` (via claude.ai)

**Prompt yang dipakai:**

```
Buat fungsi Python untuk import CSV dari URL GitHub ke Neo4j menggunakan LOAD CSV.
Ada dua tipe: import_node dan import_edge.
Harus handle: constraint uniqueness, MERGE untuk idempotency,
filter row kosong/null, dan print progress dengan timing.
Dataset: Politikus, Partai, Pendidikan, Person (nodes) +
ANGGOTA_PARTAI, ALUMNI, KERABAT (edges).
```

**Output AI:** Skeleton fungsi `import_node()` dan `import_edge()` dengan `LOAD CSV WITH HEADERS`.

**Modifikasi manual:**

- Menambahkan filter `WHERE row.\`kodeId:ID\` IS NOT NULL AND row.\`kodeId:ID\` <> ''` — AI tidak menyertakan validasi baris kosong yang menyebabkan node phantom
- Menambahkan double `MATCH` sebelum `MERGE` pada `import_edge()` untuk mencegah dangling edge jika salah satu endpoint belum ada
- Menambahkan property `{tipe: row.tipe}` pada edge `KERABAT` — AI hanya generate `MERGE (a)-[:KERABAT]->(b)` tanpa property
- Menambahkan verifikasi akhir dengan query `count(n)` per label dan `count(r)` per relasi setelah semua import selesai

---

### Cell 4 — Graph Analytics (GDS)

**Model:** `claude-sonnet-4-6` (via claude.ai)

**Prompt yang dipakai:**

```
Buat kode Neo4j GDS untuk:
- Membuat graph projection multi-label (Politikus, Partai, Pendidikan, Person)
  dengan semua relasi sebagai UNDIRECTED
- Jalankan PageRank (20 iterasi, damping 0.85), Betweenness Centrality, Louvain
- Tulis hasil ke properti node (writeProperty)
- Tampilkan top 10 PageRank, top 10 dinasti (KERABAT count), top 10 universitas (ALUMNI count)
- Sertakan fallback Cypher jika GDS tidak tersedia
```

**Output AI:** Struktur `CALL gds.pageRank.write()`, `gds.betweenness.write()`, `gds.louvain.write()` yang sudah benar.

**Modifikasi manual:**

- Menambahkan blok `try: run_query("CALL gds.graph.drop('graf_politik', false)")` sebelum projection — AI tidak menyertakan cleanup projection lama, menyebabkan error `GraphAlreadyExists` saat re-run
- Mengubah query top PageRank agar filter `p.nama <> ''` — menghindari politikus dengan nama kosong muncul di ranking
- Menambahkan blok `GDS_AVAILABLE = True/False` sebagai flag agar fallback Cypher hanya aktif ketika GDS benar-benar gagal

---

### Cell 6 — Graph Builder (Komponen 2)

**Model:** `claude-sonnet-4-6` (via claude.ai)

**Prompt yang dipakai:**

```
Buat sistem ekstraksi entitas dari teks berita politik Indonesia ke Neo4j.
LLM harus output JSON dengan format:
{"entities": [{id, tipe, nama, atribut}], "relasi": [{dari, tipe_relasi, ke, atribut}]}
Fungsi extract_entities_from_text(text) parse JSON dari output LLM.
Fungsi populate_graph_from_extraction(extracted) masukkan ke Neo4j
dengan label dinamis BeritaEntity:<Tipe>.
Tangani JSON parsing error dengan fallback struktur kosong.
```

**Output AI:** Struktur `GRAPH_BUILDER_SYSTEM` prompt dan fungsi `extract_entities_from_text()` dengan regex JSON parsing.

**Modifikasi manual:**

- Menambahkan `safe_label = re.sub(r'[^A-Za-z]', '', tipe) or "Entity"` — AI tidak menyertakan sanitasi label, padahal tipe dari LLM bisa berisi spasi atau karakter spesial yang invalid sebagai Neo4j label
- Menambahkan `print(f"Raw LLM Response: {raw[:500]}...")` untuk debugging — krusial saat testing karena LLM free model kadang mengeluarkan teks prefiks sebelum JSON
- Mengubah parsing flow: coba regex `\{.*\}` dulu, lalu fallback ke `json.loads(raw)` langsung — AI hanya generate satu path parsing yang sering gagal untuk output LLM yang berisi preamble teks
- Menambahkan `try/except json.JSONDecodeError` dan `except Exception` secara terpisah dengan print debug pada setiap branch

---

### Cell 7 — RAG Pipeline (Komponen 3)

**Model:** `claude-sonnet-4-6` (via claude.ai)

**Prompt yang dipakai:**

```
Buat Graph-Augmented RAG pipeline:
- Fungsi retrieve_graph_context(question) dengan keyword routing:
  deteksi intent dari kata kunci (berpengaruh, dinasti, partai, universitas, dll)
  lalu jalankan query Cypher yang relevan
- Fungsi format_context_for_llm(contexts) format hasil query ke string terstruktur
- Fungsi rag_query(question) dengan 3 step: Retrieve → Augment → Generate
- RAG_SYSTEM prompt yang instruksikan LLM untuk hanya jawab dari konteks yang diberikan
```

**Output AI:** Skeleton `retrieve_graph_context()` dengan beberapa keyword dan `rag_query()` dengan step retrieve-augment-generate.

**Modifikasi manual:**

- Menambahkan 3 query konteks tambahan yang tidak ada di output AI: query PageRank top-10, query network kerabat, dan query komunitas Louvain — penting agar RAG punya konteks faktual yang cukup
- Menambahkan keyword bahasa Indonesia yang lebih luas pada setiap branch (contoh: `"berpengaruh"`, `"tokoh"`, `"dominan"` → query pagerank) karena AI hanya generate 1–2 keyword per branch
- Menambahkan penanganan `KeyError` dan `Exception` pada `rag_query()` dengan fallback message informatif — sama seperti komponen lain, AI tidak memisahkan jenis error
- Mengubah `temperature=0.3` pada `rag_query()` — AI default ke `0.1` yang terlalu rigid untuk jawaban analitik yang memerlukan sedikit paraphrase

---

### Cell 9 — Evaluasi Sistem

**Model:** `claude-sonnet-4-6` (via claude.ai)

**Prompt yang dipakai:**

```
Buat blok evaluasi untuk menguji keempat komponen secara otomatis:
- Text-to-Cypher: jalankan 3 test case, hitung pass rate
- Graph Builder: query count BeritaEntity dan link ke graph utama
- Graph Analytics: cek apakah pagerank, betweenness, community sudah tertulis ke node
- RAG: ukur waktu retrieve_graph_context dan jumlah konteks yang dikembalikan
- Database summary: count semua node dan relasi
- Tier 4 checklist
```

**Output AI:** Kerangka evaluasi dengan loop test case dan status PASS/FAIL/EMPTY.

**Modifikasi manual:**

- Menambahkan kolom `elapsed` (timing per test case) pada evaluasi Text-to-Cypher — penting untuk identifikasi query yang lambat
- Memperbaiki query evaluasi Graph Builder: `count(DISTINCT n.tipe)` untuk tipe unik — AI generate `count(n.tipe)` yang tidak deduplikat
- Menambahkan `try/except` per blok evaluasi (GDS props check) agar satu komponen yang gagal tidak menghentikan evaluasi komponen lain

---

---

## Author

**Ibrahim Amar** — `ibrahimamar07`
Institut Teknologi Sepuluh Nopember (ITS), Surabaya  
Program Studi Sistem Informasi — Semester 6  
Mata Kuliah: Knowledge Graphs
