# 🎙️ YouTube Video Content Analysis using NLP

Proyek *end-to-end* **Natural Language Processing (NLP)** untuk mengekstrak, memproses, dan menganalisis dinamika wicara, polaritas sentimen, serta topik pembahasan dari video podcast YouTube: **"MAHFUD MD: MENYADAP PRESIDEN, MAKAR!"** (Durasi: 40 menit 23 detik)[cite: 1, 3].

## 📋 Daftar Isi
- [Latar Belakang & Tujuan](#-latar-belakang--tujuan)
- [Arsitektur Pipeline](#-arsitektur-pipeline)
- [Ringkasan Temuan Utama](#-ringkasan-temuan-utama)
- [Struktur Direktori](#-struktur-direktori)
- [Instalasi & Penggunaan](#-instalasi--penggunaan)
- [Tech Stack](#-tech-stack)

---

## 🎯 Latar Belakang & Tujuan
Video diskusi politik dan hukum berdurasi panjang menyimpan banyak informasi penting yang sulit dipetakan secara manual. Proyek ini membangun *pipeline* komputasional otomatis mulai dari pengunduhan audio hingga generasi laporan eksekutif untuk:
1. Mengubah audio wicara bahasa Indonesia menjadi teks terstruktur berbasis *timestamp*[cite: 1, 4, 29].
2. Mengidentifikasi kata kunci dan frasa (N-gram) yang paling dominan dibahas[cite: 41, 45].
3. Mengukur distribusi sentimen pembicara di setiap segmen percakapan[cite: 55].
4. Menemukan klaster tema tersembunyi (*latent topics*) serta memetakan pergeserannya dari menit ke menit[cite: 61, 78].

---

## ⚙️ Arsitektur Pipeline

1. **Ekstraksi Audio (`yt-dlp` & `ffmpeg`)**: Mengunduh aliran audio berformat `.m4a` kualitas terbaik dari URL YouTube[cite: 2].
2. **Speech-to-Text (`OpenAI Whisper - small`)**[cite: 4]: Transkripsi audio bahasa Indonesia menghasilkan **1.018 segmen** teks beserta *timestamp* (`start` & `end`)[cite: 4, 5, 29].
3. **Text Preprocessing (`Regex`, `NLTK`, `Sastrawi`)**[cite: 31, 33, 35, 38]:
   - *Case folding* & pembersihan karakter non-alfabet[cite: 31, 32].
   - Tokenisasi kata (`word_tokenize`)[cite: 34].
   - *Stopword removal* bahasa Indonesia dengan **mempertahankan kata negasi** (`tidak`, `bukan`, `belum`, `jangan`, `tanpa`, `kurang`) agar konteks sentimen tetap akurat[cite: 35, 36].
   - *Stemming* kata berimbuhan ke bentuk dasar menggunakan Sastrawi (mereduksi kosakata unik dari 1.010 menjadi 846 kata dasar)[cite: 38, 41].
4. **Analisis Frekuensi & N-Gram (`Counter` & `CountVectorizer`)**[cite: 41, 46]: Ekstraksi Top-20 Unigram, Bigram, dan Trigram per segmen (mencegah *false n-grams* lintas batas waktu)[cite: 43, 45, 46].
5. **Visualisasi Word Cloud (`WordCloud`)**[cite: 51]: Pembuatan awan kata resolusi tinggi berdasarkan bobot frekuensi kata dasar[cite: 52, 53].
6. **Analisis Sentimen (`IndoBERT`)**[cite: 55]: Klasifikasi polaritas tiap segmen menggunakan model *pretrained* `mdhugol/indonesia-bert-sentiment-classification` (Positif, Negatif, Netral)[cite: 55, 56].
7. **Topic Modeling (`Latent Dirichlet Allocation / LDA`)**[cite: 61, 62]: Evaluasi *Perplexity Score* ($K \in \{3,4,5,6\}$) dan pemodelan klaster topik optimal ($K=4$)[cite: 63, 65].
8. **Analisis Linimasa (`Temporal Binning`)**[cite: 78, 79]: Penggabungan data topik dan sentimen ke dalam interval waktu **2 menit** (*Stacked Area Chart* & *Line Chart*)[cite: 78, 79, 80, 81].
9. **Automated Report Generation (`python-docx` & `fpdf2`)**: Ekspor otomatis ringkasan eksekutif ke format `.docx`, `.pdf`, dan `.txt`[cite: 72].

---

## 📊 Ringkasan Temuan Utama

* **Kosakata & Frasa Dominan:**
  * **Unigram:** `tidak` (83x), `mundur` (83x), `presiden` (82x), `undang` (50x), `makar` (43x)[cite: 43, 44].
  * **Bigram:** `undang undang` (23x), `tidak mundur` (8x), `wakil presiden` (7x), `pemak zulan` (6x)[cite: 49].
* **Distribusi Sentimen (IndoBERT):**
  * 🟢 **Positif:** 55.9% (569 segmen) — didominasi pernyataan normatif, edukasi tata negara, dan harapan penegakan hukum[cite: 60, 74, 76].
  * 🔴 **Negatif:** 38.2% (389 segmen) — muncul tajam pada pembahasan penyadapan kepala negara dan tuduhan makar (menit 18–22 & 30–38)[cite: 60, 74].
  * ⚪ **Netral:** 5.9% (60 segmen)[cite: 60].
* **4 Pilar Topik Utama (LDA $K=4$):**
  1. *Diskusi Interaktif & Pendapat Umum* (29.3% / 298 segmen)[cite: 68, 70]
  2. *Isu Pengunduran Diri Wapres* (26.8% / 273 segmen)[cite: 68, 70]
  3. *Kontroversi Makar & Penyadapan* (22.2% / 226 segmen)[cite: 68, 70]
  4. *Regulasi Hukum & Lembaga Negara (MPR/UU)* (21.7% / 221 segmen)[cite: 68, 70]

---

## 📁 Struktur Direktori

```text
├── dataset/                          # Menyimpan dataset transkrip, CSV pembersihan, N-gram, & grafik plot
├── notebook/
│   └── YT_Content_Analyzer.ipynb     # Notebook utama (Tahap 1 - 11)
├── output/                           # Menyimpan hasil laporan akhir (.docx, .pdf, .txt)
├── .gitignore                        # Aturan file/folder yang diabaikan Git
├── requirements.txt                  # Daftar dependensi library Python
└── README.md                         # Dokumentasi proyek
