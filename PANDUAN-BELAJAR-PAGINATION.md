# Panduan Belajar: Pagination untuk PokeDex

Panduan step-by-step untuk memahami dan nantinya menerapkan **pagination** (paginasi) di project PokeDex. Dokumen ini **hanya penjelasan konsep dan langkah**, tanpa implementasi langsung di kode.

---

## Daftar Isi

1. [Apa itu Pagination dan Kenapa Diperlukan?](#1-apa-itu-pagination-dan-kenapa-diperlukan)
2. [Cara PokeAPI Mendukung Pagination](#2-cara-pokeapi-mendukung-pagination)
3. [Konsep yang Perlu Kamu Kelola di Vue](#3-konsep-yang-perlu-kamu-kelola-di-vue)
4. [Langkah 1: State (Data) untuk Halaman](#4-langkah-1-state-data-untuk-halaman)
5. [Langkah 2: Fetch Data per Halaman](#5-langkah-2-fetch-data-per-halaman)
6. [Langkah 3: Tombol / UI untuk Ganti Halaman](#6-langkah-3-tombol--ui-untuk-ganti-halaman)
7. [Langkah 4: Hitung Total Halaman](#7-langkah-4-hitung-total-halaman)
8. [Langkah 5: Tampilkan Nomor Halaman (Opsional)](#8-langkah-5-tampilkan-nomor-halaman-opsional)
9. [Ringkasan Alur](#9-ringkasan-alur)
10. [Variasi: Infinite Scroll](#10-variasi-infinite-scroll)

---

## 1. Apa itu Pagination dan Kenapa Diperlukan?

**Pagination** = menampilkan data **per halaman** (misalnya 20 item per halaman), bukan sekaligus ratusan/ribuan item.

**Alasan untuk PokeDex:**

- Sekarang kamu mungkin fetch list pertama (20 Pokemon) lalu untuk setiap item fetch detail → itu sudah seperti “satu halaman”.
- Kalau ingin user bisa **lihat 20 Pokemon berikutnya**, **20 lagi**, dan seterusnya, kamu butuh konsep “halaman ke-1”, “halaman ke-2”, dll. Itu inti pagination.
- Manfaat: performa lebih baik (tidak load ratusan card sekaligus), UX lebih jelas (user pilih “halaman 2” atau “next”).

---

## 2. Cara PokeAPI Mendukung Pagination

PokeAPI memakai **offset** dan **limit** di URL:

- **limit**: berapa banyak item yang dikembalikan (default 20).
- **offset**: mulai dari item ke berapa (0 = dari awal).

**Contoh:**

- Halaman 1 (20 pertama):  
  `https://pokeapi.co/api/v2/pokemon?limit=20&offset=0`
- Halaman 2 (20 berikutnya):  
  `https://pokeapi.co/api/v2/pokemon?limit=20&offset=20`
- Halaman 3:  
  `https://pokeapi.co/api/v2/pokemon?limit=20&offset=40`

**Rumus umum:**

- `offset = (nomorHalaman - 1) * limit`  
  Contoh: halaman 2, limit 20 → offset = 20.

Response API juga memberi info:

- `count`: total banyaknya Pokemon (untuk hitung total halaman).
- `results`: array Pokemon untuk halaman tersebut (hanya `name`, `url`; detail tetap di-fetch terpisah kalau mau).

Jadi **step pertama** yang perlu kamu pahami: ubah cara fetch list dari “satu kali tanpa offset” menjadi “fetch dengan `limit` dan `offset` sesuai halaman yang dipilih”.

---

## 3. Konsep yang Perlu Kamu Kelola di Vue

Di sisi Vue, pagination biasanya mengelola:

1. **Halaman saat ini** (misalnya: 1, 2, 3 …).
2. **Jumlah item per halaman** (misalnya: 20).
3. **Data untuk halaman itu saja** (misalnya: `pokemonList` diisi ulang tiap ganti halaman).
4. **Total item** (dari API `count`) supaya bisa hitung total halaman.
5. **UI**: tombol Prev/Next dan (opsional) nomor halaman 1, 2, 3 …

Tidak perlu mengubah struktur PokeDex secara drastis; yang berubah terutama:

- Cara memanggil API (tambah `limit` & `offset`).
- Satu variabel “halaman saat ini”.
- Satu variabel “total count” (dari API).
- Tombol/angka untuk ganti halaman dan panggil ulang fetch.

---

## 4. Langkah 1: State (Data) untuk Halaman

Di `data()` (Options API) atau `ref()` (Composition API), tambahkan minimal:

- **currentPage** (atau `page`): number, halaman yang sedang ditampilkan. Mulai dari `1`.
- **perPage** (atau `limit`): number, berapa Pokemon per halaman, misalnya `20`.
- **totalCount** (atau `total`): number, total banyak Pokemon dari API (`response.count`). Dipakai untuk hitung “ada berapa halaman”.

`pokemonList` tetap dipakai untuk menampilkan list; isinya akan diisi ulang setiap kali user pindah halaman (langkah 2).

**Ringkas:**  
State pagination = “halaman ke berapa”, “berapa per halaman”, “total ada berapa”. Sisanya tetap list yang kamu tampilkan seperti sekarang.

---

## 5. Langkah 2: Fetch Data per Halaman

Ubah (atau buat) fungsi yang fetch list Pokemon sehingga:

1. Hitung **offset** dari halaman saat ini:  
   `offset = (currentPage - 1) * perPage`
2. Panggil API:  
   `https://pokeapi.co/api/v2/pokemon?limit=20&offset=${offset}`
3. Simpan **total count** dari response:  
   `this.totalCount = data.count` (atau variabel yang kamu pakai).
4. Untuk setiap item di `data.results`, fetch detail Pokemon (seperti `getEachPokemon` yang sekarang), lalu isi **hanya** `pokemonList` untuk halaman ini (biasanya `pokemonList = []` dulu, lalu push hasil fetch detail).

Jadi:

- **Saat pertama load**: panggil fetch dengan `currentPage = 1`, isi `pokemonList`.
- **Saat user klik “Next” / “Halaman 2”**: ubah `currentPage` jadi 2, panggil lagi fetch dengan offset baru, kosongkan lalu isi ulang `pokemonList`.

Ini inti pagination: **satu set list per halaman**, bukan satu list besar yang di-slice di front-end (meski slice juga bisa, tapi untuk PokeDex fetch per halaman lebih masuk akal).

---

## 6. Langkah 3: Tombol / UI untuk Ganti Halaman

Di template, tambahkan area untuk navigasi halaman, misalnya:

- Tombol **“Previous” / “Sebelumnya”**:  
  - Disabled jika `currentPage === 1`.  
  - Saat diklik: kurangi `currentPage` (misalnya `currentPage--`), lalu panggil fungsi fetch untuk halaman baru (langkah 2).
- Tombol **“Next” / “Selanjutnya”**:  
  - Disabled jika sudah di halaman terakhir (lihat langkah 4).  
  - Saat diklik: tambah `currentPage`, lalu panggil fetch lagi.

Event handler cukup ubah `currentPage` lalu panggil method yang sama yang dipakai untuk load list (method yang sudah kamu sesuaikan di langkah 2).

---

## 7. Langkah 4: Hitung Total Halaman

Total halaman = total item dibagi item per halaman, dibulatkan ke atas:

- **totalPages = Math.ceil(totalCount / perPage)**

Contoh: total 150 Pokemon, 20 per halaman → 150/20 = 7.5 → 8 halaman.

Kamu bisa simpan di **computed property** (misalnya `totalPages`) supaya reactive. Lalu:

- Tombol “Next” disabled jika `currentPage >= totalPages`.
- (Opsional) Tampilkan teks “Halaman 1 dari 8”.

---

## 8. Langkah 5: Tampilkan Nomor Halaman (Opsional)

Selain Prev/Next, kamu bisa tampilkan nomor halaman: 1, 2, 3, … 8.

- Loop dari 1 sampai `totalPages`.
- Tiap nomor bisa berupa tombol atau link.
- Saat diklik: set `currentPage = nomor itu`, lalu panggil fetch (sama seperti langkah 2).
- Beri style khusus untuk halaman aktif (misalnya `currentPage === nomor`).

Kalau total halaman banyak (misalnya 50), bisa batasi yang ditampilkan (misalnya hanya halaman sekitar `currentPage`, atau “1 … 4 5 6 … 50”). Itu penyempurnaan; untuk belajar, cukup 1, 2, 3 … sudah cukup.

---

## 9. Ringkasan Alur

1. **State**: `currentPage`, `perPage`, `totalCount`; `pokemonList` untuk list yang tampil.
2. **Fetch**: API dipanggil dengan `limit=perPage` dan `offset=(currentPage-1)*perPage`; simpan `count` ke `totalCount`, isi `pokemonList` dengan hasil (setelah fetch detail per Pokemon jika seperti sekarang).
3. **Total halaman**: `totalPages = Math.ceil(totalCount / perPage)`.
4. **UI**: Tombol Prev (disabled di halaman 1), tombol Next (disabled di halaman terakhir), optional nomor halaman; setiap klik ubah `currentPage` lalu panggil fetch lagi.
5. **Initial load**: `mounted()` panggil fetch dengan `currentPage = 1`.

Tidak perlu mengubah tampilan detail atau search; pagination hanya mengatur **cara list Pokemon di-load dan ditampilkan per halaman**.

---

## 10. Variasi: Infinite Scroll

Alternatif dari “tombol halaman” adalah **infinite scroll**: saat user scroll ke bawah, otomatis load halaman berikutnya dan **tambahkan** ke list (bukan ganti list).

Konsepnya:

- Tetap pakai `currentPage` (atau “page berikutnya yang akan di-load”).
- Saat user scroll mendekati bawah (deteksi dengan scroll position atau Intersection Observer), naikkan `currentPage`, fetch halaman itu, lalu **append** hasil ke `pokemonList` (bukan replace).
- Tidak perlu tombol Next/Prev; total halaman bisa tetap dihitung untuk “sudah load sampai halaman berapa”.

Ini bisa kamu coba setelah pagination tombol Prev/Next sudah nyaman.

---

Dengan mengikuti step-step di atas, kamu bisa mempraktikkan pagination di PokeDex sendiri: mulai dari state dan fetch, lalu tombol, lalu total halaman dan nomor halaman. Jika satu langkah belum jelas, fokuskan dulu pada langkah itu (misalnya hanya ubah fetch dan currentPage) baru lanjut ke UI.
