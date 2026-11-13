# 🎙️ YouTube Video Content Analysis using NLP

Proyek *end-to-end* **Natural Language Processing (NLP)** untuk mengekstrak, memproses, dan menganalisis dinamika wicara, polaritas sentimen, serta topik pembahasan dari video podcast YouTube: **"MAHFUD MD: MENYADAP PRESIDEN, MAKAR!"** (Durasi: 40 menit 23 detik).

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
1. Mengubah audio wicara bahasa Indonesia menjadi teks terstruktur berbasis *timestamp*.
2. Mengidentifikasi kata kunci dan frasa (N-gram) yang paling dominan dibahas.
3. Mengukur distribusi sentimen pembicara di setiap segmen percakapan.
4. Menemukan klaster tema tersembunyi (*latent topics*) serta memetakan pergeserannya dari menit ke menit.

---

## ⚙️ Arsitektur Pipeline

1. **Ekstraksi Audio (`yt-dlp` & `ffmpeg`)**: Mengunduh aliran audio berformat `.m4a` kualitas terbaik dari URL YouTube.
2. **Speech-to-Text (`OpenAI Whisper - small`)**: Transkripsi audio bahasa Indonesia menghasilkan **1.018 segmen** teks beserta *timestamp* (`start` & `end`).
3. **Text Preprocessing (`Regex`, `NLTK`, `Sastrawi`)**:
   - *Case folding* & pembersihan karakter non-alfabet.
   - Tokenisasi kata (`word_tokenize`).
   - *Stopword removal* bahasa Indonesia dengan **mempertahankan kata negasi** (`tidak`, `bukan`, `belum`, `jangan`, `tanpa`, `kurang`) agar konteks sentimen tetap akurat.
   - *Stemming* kata berimbuhan ke bentuk dasar menggunakan Sastrawi (mereduksi kosakata unik dari 1.010 menjadi 846 kata dasar).
4. **Analisis Frekuensi & N-Gram (`Counter` & `CountVectorizer`)**: Ekstraksi Top-20 Unigram, Bigram, dan Trigram per segmen (mencegah *false n-grams* lintas batas waktu).
5. **Visualisasi Word Cloud (`WordCloud`)**: Pembuatan awan kata resolusi tinggi berdasarkan bobot frekuensi kata dasar.
6. **Analisis Sentimen (`IndoBERT`)**: Klasifikasi polaritas tiap segmen menggunakan model *pretrained* `mdhugol/indonesia-bert-sentiment-classification` (Positif, Negatif, Netral).
7. **Topic Modeling (`Latent Dirichlet Allocation / LDA`)**: Evaluasi *Perplexity Score* ($K \in \{3,4,5,6\}$) dan pemodelan klaster topik optimal ($K=4$).
8. **Analisis Linimasa (`Temporal Binning`)**: Penggabungan data topik dan sentimen ke dalam interval waktu **2 menit** (*Stacked Area Chart* & *Line Chart*).
9. **Automated Report Generation (`python-docx` & `fpdf2`)**: Ekspor otomatis ringkasan eksekutif ke format `.docx`, `.pdf`, dan `.txt`.

---

## 📊 Ringkasan Temuan Utama

- **Kosakata & Frasa Dominan:**
  * **Unigram:** `tidak` (83x), `mundur` (83x), `presiden` (82x), `undang` (50x), `makar` (43x).
  * **Bigram:** `undang undang` (23x), `tidak mundur` (8x), `wakil presiden` (7x), `pemak zulan` (6x).
- **Distribusi Sentimen (IndoBERT):**
  * 🟢 **Positif:** 55.9% (569 segmen) — didominasi pernyataan normatif, edukasi tata negara, dan harapan penegakan hukum.
  * 🔴 **Negatif:** 38.2% (389 segmen) — muncul tajam pada pembahasan penyadapan kepala negara dan tuduhan makar (menit 18–22 & 30–38).
  * ⚪ **Netral:** 5.9% (60 segmen).
- **4 Pilar Topik Utama (LDA $K=4$):**
  1. *Diskusi Interaktif & Pendapat Umum* (29.3% / 298 segmen)
  2. *Isu Pengunduran Diri Wapres* (26.8% / 273 segmen)
  3. *Kontroversi Makar & Penyadapan* (22.2% / 226 segmen)
  4. *Regulasi Hukum & Lembaga Negara (MPR/UU)* (21.7% / 221 segmen)

---

## 📁 Struktur Direktori

```text
├── dataset/                        # Menyimpan dataset transkrip, CSV pembersihan, N-gram, & grafik plot
├── notebook/
│   └── YT_Content_Analyzer.ipynb     # Notebook utama (Tahap 1 - 11)
├── output/                         # Menyimpan hasil laporan akhir (.docx, .pdf, .txt)
├── .gitignore                      # Aturan file/folder yang diabaikan Git
├── requirements.txt                # Daftar dependensi library Python
└── README.md                       # Dokumentasi proyek
```

---

## 🚀 Instalasi & Penggunaan

### 1. Prasyarat Sistem
Pastikan **FFmpeg** sudah terinstal di sistem operasi Anda (wajib untuk `yt-dlp` dan `openai-whisper`):
* **Ubuntu/Debian / Google Colab:** `sudo apt-get update && sudo apt-get install -y ffmpeg`
* **macOS:** `brew install ffmpeg`
* **Windows:** `winget install Gyan.FFmpeg`

### 2. Instalasi Dependensi Python
```bash
git clone https://github.com/Fadhail-3596/YT-Content-Analyzer.git
cd YT-Content-Analyzer
pip install -r requirements.txt
```

### 3. Menjalankan Notebook
Buka file `.ipynb` di dalam folder `notebook/` menggunakan Jupyter Notebook, VS Code, atau unggah ke **Google Colab**. 
> 💡 **Tips:** Jika menggunakan Google Colab, aktifkan akselerasi GPU (**Runtime → Change runtime type → T4 GPU**) agar proses transkripsi Whisper dan inferensi IndoBERT berjalan jauh lebih cepat.

---

## 🛠️ Tech Stack
* **Audio & Speech-to-Text:** `yt-dlp`, `ffmpeg`, `openai-whisper`, `PyTorch`
* **NLP & Machine Learning:** `NLTK`, `Sastrawi`, `HuggingFace Transformers (IndoBERT)`, `Scikit-Learn (LDA & CountVectorizer)`
* **Data Manipulation & Visualization:** `Pandas`, `NumPy`, `Matplotlib`, `Seaborn`, `WordCloud`
* **Document Automation:** `python-docx`, `fpdf2`