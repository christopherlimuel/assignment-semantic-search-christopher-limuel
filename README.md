# Semantic Search & LLM Application

Prototipe sistem RAG (Retrieval-Augmented Generation) menggunakan similarity search untuk mengambil informasi relevan berdasarkan query, dan menggunakan model LLM Gemini 3.1 Flash-Lite untuk menyajikan jawaban.

---

## 1. Problem Statement & Dataset

### Problem Statement
Membuat LLM mampu menjawab pertanyaan berdasarkan daftar FAQ, mengatasi limitasi keyword-matching, memahami intensi pertanyaan, dan mengenali pertanyaan di luar konteks.

### Dataset Overview
* **Sumber Data:** ```ecommerce_faq_knowledge_base.csv```
* **Total Ukuran Data:** 59 data pasangan pertanyaan-jawaban
* **Struktur**:
  * ```faq_id```: Identifier unik (integer)
  * ```category```: Kategori pertanyaan (e.g., Pemesanan, Pengiriman, Pembayaran)
  * ```question```: Pertanyaan yang sering muncul (FAQ)
  * ```answer```: Jawaban dari pertanyaan

---

## 2. Metodologi

### Asumsi dan Batasan Masalah:

1. Semantic search mengambil 3 pasangan pertanyaan-jawaban terdekat dari query/pertanyaan, tanpa similarity threshold
2. Model LLM hanya menjawab berdasarkan 3 data pertanyaan-jawaban yang diambil oleh semantic search
3. Penilaian pertanyaan di luar konteks dilakukan oleh model LLM, dengan fallback response menyarankan untuk menghubungi customer service

### Arsitektur Sistem Pipeline
Indexing phase:

``` [Ingest] -> [Embed] -> [Store] ```
* Ingest: Memuat knowledge base FAQ dan menyiapkannya sebagai unit-unit retrievable
* Embed: Mengubah setiap unit retrievable menjadi vektor menggunakan model embedding ```gemini-embedding-2```
* Store: Menyimpan vektor embedding dalam vector database FAISS

Query phase:

``` [Retrieve] -> [Generate] ```

* Retrieve: Mengubah query menjadi vector embedding dan menjalankan similarity search untuk mengambil entry paling relevan dari query
* Generate: Menyisipkan hasil retrieval sebagai context ke dalam prompt, lalu menghasilkan jawaban melalui LLM

---

## 3. Hasil & Analisis

### Analisis Relevansi

Berdasarkan hasil query tes, ketiga jenis query dapat diretrieve dengan baik, dan LLM juga dapat menjawab sesuai dengan konteks yang diberikan. Untuk query yang di luar konteks masih menghasilkan retrieval, namun model LLM dapat mengenalinya dan merespon dengan baik. Untuk mencegah risiko halusinasi pada model LLM, dapat juga ditambahkan *similarity thresholding* pada proses semantic search agar pertanyaan yang di luar konteks dapat disaring terlebih dahulu sebelum mencapai model LLM.


### Hasil Query Tes

**Query persis dari database FAQ:**
>Apakah promo bisa digabung dengan diskon member?

>Halo! Senang sekali bisa membantu Anda. 
>
>Mengenai pertanyaan Anda, beberapa promo dapat digabung dengan diskon member, namun ada juga promo eksklusif yang tidak dapat ditumpuk. Untuk memastikannya, silakan cek ketentuan yang tertera pada halaman detail promo tersebut ya.

Sumber FAQ yang digunakan:
- **Apakah promo bisa digabung dengan diskon member?** -> Beberapa promo dapat digabung dengan diskon member, namun ada juga promo eksklusif yang tidak dapat ditumpuk. Ketentuan tertera pada halaman detail promo.
- **Apakah member baru mendapat diskon khusus?** -> Pengguna baru biasanya mendapat voucher selamat datang yang dapat digunakan pada transaksi pertama, cek halaman promo setelah pendaftaran.
- **Bagaimana cara mendapatkan info promo terbaru?** -> Info promo terbaru dapat dilihat pada halaman utama, email berlangganan, atau notifikasi aplikasi yang telah diaktifkan.
---
**Query yang diparafrase:**
>Biaya ongkir untuk pengembalian ditanggung oleh siapa?

>Halo! Saya dengan senang hati membantu menjawab pertanyaan Anda.
>
>Mengenai biaya ongkos kirim pengembalian, ketentuannya adalah sebagai berikut:
>* Jika barang tidak sesuai deskripsi atau terdapat cacat produksi, maka ongkos kirim pengembalian akan ditanggung oleh penjual.
>* Jika pengembalian dilakukan karena alasan pribadi (bukan karena kesalahan penjual), maka biaya ongkos kirim akan ditanggung oleh pembeli.
>
>Apakah ada hal lain yang bisa saya bantu?

Sumber FAQ yang digunakan:
- **Apakah biaya ongkos kirim pengembalian ditanggung pembeli?** -> Jika barang tidak sesuai deskripsi atau cacat produksi, ongkos kirim pengembalian ditanggung penjual. Jika alasan pribadi (bukan karena kesalahan penjual), biaya ditanggung pembeli.
- **Ke mana dana refund akan dikembalikan?** -> Dana refund akan dikembalikan ke metode pembayaran awal, atau ke saldo akun apabila memilih opsi tersebut saat pengajuan.
- **Bagaimana prosedur pengembalian barang yang tidak sesuai?** -> Ajukan pengembalian melalui menu 'Pesanan Saya' dengan memilih alasan dan menyertakan bukti foto, maksimal 3 hari sejak barang diterima.
---
**Query yang di luar konteks:**
>Mending RTX 4050 atau RTX 3060 untuk gaming?

>Jawaban: Mohon maaf, saya belum memiliki informasi mengenai perbandingan spesifikasi antara RTX 4050 dan RTX 3060 untuk kebutuhan gaming. Untuk informasi lebih lanjut, silakan hubungi customer service kami agar dapat dibantu lebih lanjut.

Sumber FAQ yang digunakan:
- **Bagaimana cara membandingkan beberapa produk sekaligus?** -> Gunakan fitur 'Bandingkan Produk' pada halaman kategori untuk melihat spesifikasi beberapa produk secara berdampingan.
- **Apakah bisa memesan dalam jumlah besar untuk kebutuhan reseller?** -> Untuk pembelian grosir, silakan hubungi penjual langsung melalui chat untuk menanyakan ketersediaan stok dan kemungkinan harga khusus.
- **Apakah tersedia opsi pengiriman same-day?** -> Pengiriman same-day tersedia untuk beberapa kota besar tertentu, tergantung ketersediaan kurir di lokasi penjual.
---

## 4. Cara Menjalankan Project

#### Prasyarat System
* Python 3.9+
* Google Gemini API Key (diakses dari system variable)

#### 1. Clone Repository & Install Dependencies
```bash
git clone https://github.com/christopherlimuel/assignment-semantic-search-christopher-limuel

pip install -r requirements.txt
```


#### 2. Menjalankan Notebook / Script Evaluasi

```bash
jupyter notebook notebook/starter_notebook_semantic_search.ipynb
```
