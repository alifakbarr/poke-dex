# Dokumentasi Pembuatan PokeDex dengan Vue.js

Dokumentasi step-by-step pembuatan aplikasi **PokeDex** (katalog Pokemon) menggunakan **Vue 3**, **Vite**, dan **Tailwind CSS v4**. API yang digunakan: [PokeAPI](https://pokeapi.co/).

---

## Daftar Isi

1. [Persiapan & Setup Project](#1-persiapan--setup-project)
2. [Struktur Project](#2-struktur-project)
3. [Konfigurasi Tailwind CSS](#3-konfigurasi-tailwind-css)
4. [Halaman Utama & Desain Pokedex](#4-halaman-utama--desain-pokedex)
5. [Mengambil Data List Pokemon dari API](#5-mengambil-data-list-pokemon-dari-api)
6. [Tampilan List Pokemon (Grid)](#6-tampilan-list-pokemon-grid)
7. [Halaman Detail Pokemon](#7-halaman-detail-pokemon)
8. [Fitur Search & Suggestion](#8-fitur-search--suggestion)
9. [Ringkasan Fitur & Data](#9-ringkasan-fitur--data)

---

## 1. Persiapan & Setup Project

### Yang dibutuhkan

- **Node.js** (versi 20.x atau 22.x ke atas)
- **npm** (biasanya sudah ikut Node.js)

### Langkah

**1.1** Buat project Vue dengan Vite (jika dari awal):

```bash
npm create vite@latest poke-dex -- --template vue
cd poke-dex
```

**1.2** Install dependency (Vue & Vite saja; Tailwind dipakai via CDN, tidak perlu di-install):

```bash
npm install
```

**1.3** Jalankan development server:

```bash
npm run dev
```

Aplikasi berjalan di `http://localhost:5173` (atau port yang ditampilkan di terminal).

> **Catatan:** Tailwind CSS **tidak** di-install lewat npm. Styling memakai **Tailwind v4 via CDN** di `index.html` (lihat bagian 3).

---

## 2. Struktur Project

```
poke-dex/
├── index.html          # Entry HTML, load Tailwind CDN & mount Vue app
├── package.json        # Dependencies (Vue, Vite; Tailwind via CDN)
├── vite.config.js      # Konfigurasi Vite
├── src/
│   ├── main.js         # Entry JS: createApp(App).mount('#app')
│   ├── App.vue         # Root component, me-render Content.vue
│   └── Content.vue     # Semua UI PokeDex: search, list, detail
└── public/
    └── favicon.ico
```

- **App.vue**: Hanya meng-import dan me-render komponen `Content.vue`.
- **Content.vue**: Berisi seluruh logika dan tampilan (search, list Pokemon, detail Pokemon).

---

## 3. Konfigurasi Tailwind CSS (CDN)

Di project ini **Tailwind CSS v4 dipakai hanya via CDN**. Tidak ada instalasi `tailwindcss` lewat npm dan tidak ada file konfigurasi (tailwind.config.js, dll).

**3.1** Di **index.html**, tambahkan script Tailwind sebelum `</head>`:

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

**3.2** Setelah script CDN dimuat, semua class Tailwind (misalnya `flex`, `grid`, `bg-[#DC0A2D]`) bisa dipakai di komponen Vue. Tidak perlu menjalankan `npm install tailwindcss`.

**3.3** Font kustom (opsional) di **Content.vue**:

```html
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
.poke-title {
  font-family: 'Press Start 2P', cursive;
}
</style>
```

---

## 4. Halaman Utama & Desain Pokedex

### 4.1 Data & state di `Content.vue`

Di **Content.vue**, gunakan Options API dengan `data()`:

```js
data() {
  return {
    pokemonList: [],      // List Pokemon untuk tampilan grid (batch pertama)
    pokemonAllList: [],   // Semua nama Pokemon untuk search/suggestion
    suggestionList: [],   // Hasil filter suggestion (max 3)
    search: '',
    viewedAsDetail: false,
    selectedPokemon: null
  };
}
```

- **pokemonList**: Data Pokemon yang ditampilkan di grid (dari API list default).
- **pokemonAllList**: Daftar semua Pokemon (nama) dari API dengan `limit=1350` untuk fitur search.
- **suggestionList**: Suggestion search (misalnya 3 nama yang match).
- **viewedAsDetail**: Boolean untuk beralih antara tampilan list dan detail.
- **selectedPokemon**: Objek detail Pokemon yang sedang dilihat.

### 4.2 Layout halaman

- **Background**: Gradien merah khas Pokedex (`#DC0A2D` → `#B50721` → `#8B0000`).
- **Kartu tengah**: Kotak hitam dengan border abu (efek “layar Pokedex”) berisi:
  - Judul "POKÉDEX" + tagline.
  - Input search.
  - Teks suggestion (inline, tidak enter ke bawah).
- **Di bawah kartu**: Area untuk list Pokemon (grid) atau detail Pokemon (satu Pokemon).

Semua styling memakai class Tailwind di template (warna, border, rounded, shadow).

---

## 5. Mengambil Data List Pokemon dari API

### 5.1 Endpoint PokeAPI

- **List (batch pertama)**: `https://pokeapi.co/api/v2/pokemon/`  
  Mengembalikan `results` berisi array `{ name, url }` (default sekitar 20 Pokemon).
- **Detail per Pokemon**: `https://pokeapi.co/api/v2/pokemon/{name atau id}`  
  Mengembalikan objek lengkap (sprites, types, stats, height, weight, abilities, dll).
- **Semua nama (untuk search)**: `https://pokeapi.co/api/v2/pokemon/?limit=1350`  
  Hanya dipakai untuk mengambil daftar nama (untuk filter suggestion).

### 5.2 Method: `getListPokemon()`

- Fetch `https://pokeapi.co/api/v2/pokemon/`.
- Untuk setiap item di `data.results`, panggil `getEachPokemon(result)`.

### 5.3 Method: `getEachPokemon(result)`

- Fetch `https://pokeapi.co/api/v2/pokemon/{result.name}`.
- Parse JSON, lalu `this.pokemonList.push(data)`.
- Hasilnya: `pokemonList` berisi array objek Pokemon lengkap untuk ditampilkan di grid.

### 5.4 Method: `getAllPokemon()`

- Fetch `https://pokeapi.co/api/v2/pokemon/?limit=1350`.
- Simpan `data.results` ke `this.pokemonAllList` (berisi `{ name, url }`).
- Digunakan untuk filter suggestion berdasarkan input search.

### 5.5 Pemanggilan di `mounted()`

```js
mounted() {
  this.getListPokemon();
  this.getAllPokemon();
}
```

Saat halaman pertama kali load, list Pokemon tampil dan daftar untuk suggestion siap dipakai.

---

## 6. Tampilan List Pokemon (Grid)

### 6.1 Kondisi tampilan

- List ditampilkan hanya jika **tidak** dalam mode detail: `v-if="!viewedAsDetail"`.

### 6.2 Template grid

- Satu container dengan class `grid` (misalnya `grid-cols-2 sm:grid-cols-3 md:grid-cols-4`).
- Loop: `v-for="pokemon in pokemonList"` dengan `:key="pokemon.id"`.
- Setiap item:
  - Gambar: `pokemon.sprites.front_default`.
  - Nama: `pokemon.name` (bisa pakai `capitalize`).
  - Nomor: `pokemon.id`.
- Tambahkan `@click="viewedDetailPokemon(pokemon.name)"` agar saat diklik pindah ke halaman detail.

### 6.3 Method: `viewedDetailPokemon(name)`

- Fetch `https://pokeapi.co/api/v2/pokemon/{name}`.
- Set `this.selectedPokemon = data` dan `this.viewedAsDetail = true`.
- Tampilan otomatis beralih ke blok detail (karena `v-else` di template).

---

## 7. Halaman Detail Pokemon

### 7.1 Kondisi tampilan

- Detail ditampilkan jika `viewedAsDetail === true`, dengan `v-if="selectedPokemon"` di dalamnya.

### 7.2 Konten yang ditampilkan

- **Tombol kembali**: "← View All Pokemon" → `@click="viewedAsDetail = false"`.
- **Gambar**: `selectedPokemon.sprites?.front_default`.
- **Nama & nomor**: `selectedPokemon.name`, `selectedPokemon.id` (format #001, #025, dll).
- **Tipe**: Loop `selectedPokemon.types` → tampilkan `type.name`.
- **Tinggi**: `selectedPokemon.height / 10` (satuan meter).
- **Berat**: `selectedPokemon.weight / 10` (satuan kg).
- **Base stats**: Loop `selectedPokemon.stats` → nama stat + bar progress + `base_stat`.
- **Abilities**: Loop `selectedPokemon.abilities` → `ability.name`.

### 7.3 Layout

- Bisa satu baris: kiri (gambar + nama + tipe), kanan (tinggi, berat, stats, abilities).
- Di mobile bisa diubah jadi satu kolom (flex-col).

Semua data diambil dari objek `selectedPokemon` yang sudah di-fetch di `viewedDetailPokemon`.

---

## 8. Fitur Search & Suggestion

### 8.1 Binding input

- Input search: `v-model="search"` (two-way binding ke `data.search`).

### 8.2 Watcher `search`

- Saat `search` berubah, filter `pokemonAllList` berdasarkan nama yang mengandung teks search.
- Ambil maksimal 3 hasil: `filteredPokemon.slice(0, 3)` → simpan ke `suggestionList`.

Contoh:

```js
watch: {
  search() {
    let filteredPokemon = this.pokemonAllList.filter(pokemon => {
      return pokemon.name.includes(this.search);
    });
    this.suggestionList = filteredPokemon.slice(0, 3);
  }
}
```

### 8.3 Tampilan suggestion

- Tampilkan dalam satu baris (tidak enter ke bawah): gunakan `flex flex-wrap` dan elemen inline (misalnya `<span>`).
- Loop `v-for="(suggestion, index) in suggestionList"`.
- Tampilkan `suggestion.name`; antara item bisa diberi pemisah "|" (hanya di antara item, bukan setelah item terakhir).
- Klik suggestion: panggil `viewedDetailPokemon(suggestion.name)` agar langsung buka detail Pokemon tersebut.

---

## 9. Ringkasan Fitur & Data

| Fitur              | Cara kerja singkat |
|--------------------|--------------------|
| List Pokemon       | `getListPokemon()` + `getEachPokemon()` → `pokemonList`, ditampilkan dengan `v-for` di grid. |
| Klik card          | `@click="viewedDetailPokemon(pokemon.name)"` → fetch detail, set `selectedPokemon` dan `viewedAsDetail = true`. |
| Detail Pokemon     | Tampilkan `selectedPokemon` (gambar, nama, tipe, tinggi, berat, stats, abilities). |
| Tombol kembali     | Set `viewedAsDetail = false` → tampilan kembali ke grid. |
| Search suggestion  | `watch` pada `search` → filter `pokemonAllList` → `suggestionList` (max 3), tampil inline. |
| Klik suggestion    | `viewedDetailPokemon(suggestion.name)` → buka detail. |

### Data dari PokeAPI (per Pokemon)

- `id`, `name`
- `sprites.front_default`
- `types[]` → `type.name`
- `height`, `weight`
- `stats[]` → `stat.name`, `base_stat`
- `abilities[]` → `ability.name`

---

## Script yang Sering Dipakai

```bash
# Development
npm run dev

# Build production
npm run build

# Preview build
npm run preview
```

---

Dokumentasi ini menjelaskan alur pembuatan PokeDex dari setup project, integrasi API, sampai fitur list, detail, dan search suggestion. Jika ada langkah yang ingin diperdalam (misalnya filtering list by type atau pagination), bisa dikembangkan dari struktur yang sama.
