<script>
export default {
  data() {
    return {
      pokemonList: [],
      viewedAsDetail: false,
      selectedPokemon: '',
      search: '',
      pokemonAllList: [],
      suggestionList: [],
      currentPage: 1,
      perPage: 20,
      totalCount: 0,
      loading: false
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.totalCount / this.perPage) || 1;
    },
    paginationItems() {
      const total = this.totalPages;
      const cur = this.currentPage;
      if (total <= 7) {
        return Array.from({ length: total }, (_, i) => ({ type: 'page', num: i + 1 }));
      }
      const start = Math.max(1, Math.min(cur - 2, total - 4));
      const end = Math.min(total, Math.max(cur + 2, 5));
      const items = [];
      if (start > 1) {
        items.push({ type: 'page', num: 1 });
        items.push({ type: 'ellipsis' });
      }
      for (let i = start; i <= end; i++) {
        items.push({ type: 'page', num: i });
      }
      if (end < total) {
        items.push({ type: 'ellipsis' });
        items.push({ type: 'page', num: total });
      }
      return items;
    }
  },
  methods: {
    async getListPokemon() {
      this.loading = true;
      const offset = (this.currentPage - 1) * this.perPage;
      const url = `https://pokeapi.co/api/v2/pokemon?limit=${this.perPage}&offset=${offset}`;
      const response = await fetch(url);
      const data = await response.json();
      this.totalCount = data.count;
      const details = await Promise.all(
        data.results.map((r) =>
          fetch("https://pokeapi.co/api/v2/pokemon/" + r.name).then((res) => res.json())
        )
      );
      this.pokemonList = details;
      this.loading = false;
    },
    goToPage(page) {
      if (page < 1 || page > this.totalPages) return;
      this.currentPage = page;
      this.getListPokemon();
    },
    async viewedDetailPokemon(name){
       let response = await fetch("https://pokeapi.co/api/v2/pokemon/" + name);
       let data = await response.json();
       
       this.viewedAsDetail = true;
       this.selectedPokemon = data;
    },
    async getAllPokemon(){
        let response = await fetch("https://pokeapi.co/api/v2/pokemon/?limit=1350");
        let data = await response.json();
        let results = data.results;
       
        this.pokemonAllList = results;
    }
  },
  watch:{
    search(){
        let filteredPokemon = this.pokemonAllList.filter(pokemon => {
            return pokemon.name.includes(this.search);
        });

        this.suggestionList = filteredPokemon.slice(0, 3);
    }
  },
  mounted() {
    this.getListPokemon();
    this.getAllPokemon();
  }
};
</script>

<template>
  <div class="min-h-screen bg-gradient-to-b from-[#DC0A2D] via-[#B50721] to-[#8B0000] flex flex-col items-center justify-center px-4 py-8">
    <!-- Decorative top bar -->
    <div class="absolute top-0 left-0 right-0 h-3 bg-[#1a1a1a] flex items-center justify-center gap-2 py-1 animate-fade-down">
      <div class="dot-led dot-yellow w-2 h-2 rounded-full bg-yellow-400 shadow-[0_0_6px_#FECA1B]"></div>
      <div class="dot-led w-2 h-2 rounded-full bg-blue-500"></div>
      <div class="dot-led w-2 h-2 rounded-full bg-green-500"></div>
    </div>

    <!-- Main content card - Pokedex style -->
    <div class="pokedex-card relative w-full max-w-xl rounded-3xl border-4 border-[#2C2C2C] bg-[#1a1a1a] shadow-2xl shadow-black/50 overflow-hidden mt-6">
      <!-- Inner screen border -->
      <div class="rounded-2xl border-4 border-[#4a4a4a] m-2 bg-gradient-to-b from-[#2d2d2d] to-[#1a1a1a] p-8">
        <!-- Logo / Title -->
        <div class="text-center mb-10 animate-fade-in">
          <h1 class="text-3xl md:text-4xl font-black tracking-widest text-white drop-shadow-lg poke-title poke-title-glow">
            POKÉDEX
          </h1>
          <p class="text-yellow-400 text-sm mt-2 tracking-widest font-semibold animate-fade-in" style="animation-delay: 0.15s; animation-fill-mode: both;">Gotta Catch 'Em All!</p>
        </div>

        <!-- Search box - centered -->
        <div class="flex flex-col items-center gap-4">
          <div class="search-wrap w-full max-w-md bg-[#0d0d0d] rounded-2xl p-2 border-2 border-[#4a4a4a] shadow-inner transition-all duration-300 focus-within:border-yellow-500/60 focus-within:ring-2 focus-within:ring-yellow-500/20">
            <input
              v-model="search"
              type="text"
              placeholder="Cari Pokemon nama..."
              class="w-full bg-transparent text-white placeholder-gray-500 px-4 py-3 rounded-xl outline-none text-sm md:text-base"
            />
          </div>
          <p class="text-gray-500 text-xs">Contoh: Pikachu, Charmander, Charizard</p>
          <div class="flex flex-wrap items-center gap-x-1 gap-y-0 text-sm">
            <span class="text-gray-300 text-xs shrink-0">Suggestion:</span>
            <template v-for="(suggestion, index) in suggestionList" :key="suggestion.id">
              <span class="text-white hover:text-yellow-500 cursor-pointer transition-colors duration-200" @click="viewedDetailPokemon(suggestion.name)">{{ suggestion.name }}</span>
              <span v-if="index < suggestionList.length - 1" class="text-gray-500">|</span>
            </template>
          </div>
        </div>

        <!-- Decorative bottom -->
        <div class="mt-10 flex justify-center gap-4">
          <div class="h-1 w-16 rounded-full bg-[#DC0A2D]"></div>
          <div class="h-1 w-8 rounded-full bg-yellow-500"></div>
          <div class="h-1 w-16 rounded-full bg-[#DC0A2D]"></div>
        </div>
      </div>
    </div>

    <!-- List view - tampilkan list data di sini -->
    <div class="w-full max-w-4xl mt-8 px-4">
      <Transition name="view-switch" mode="out-in">
      <div v-if="!viewedAsDetail" key="list" class="rounded-2xl border-4 border-[#2C2C2C] bg-[#1a1a1a] shadow-2xl shadow-black/50 overflow-hidden">
        <div class="rounded-xl border-2 border-[#4a4a4a] m-2 bg-[#0d0d0d] p-4 min-h-[200px] relative">
          <div v-if="loading" class="absolute inset-0 bg-[#0d0d0d]/80 z-10 flex justify-center items-center rounded-xl transition-opacity duration-300" aria-hidden="true">
            <div class="flex flex-col items-center gap-3">
              <div class="poke-spinner"></div>
              <p class="text-yellow-400 text-sm">Memuat Pokemon...</p>
            </div>
          </div>
          <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-3">
            <div
              v-for="(pokemon, index) in pokemonList"
              :key="pokemon.id"
              class="poke-card rounded-xl border-2 border-[#4a4a4a] bg-[#2d2d2d] p-3 text-center cursor-pointer transition-all duration-300 hover:border-yellow-500/50 hover:scale-105 hover:shadow-lg hover:shadow-yellow-500/10 active:scale-[0.98]"
              :style="{ animationDelay: `${index * 40}ms` }"
              @click="viewedDetailPokemon(pokemon.name)"
            >
              <img :src="pokemon.sprites.front_default" :alt="pokemon.name" class="w-18 h-18 mx-auto object-contain transition-transform duration-300 group-hover:scale-110">
              <p class="text-white font-semibold mt-2 capitalize">{{ pokemon.name }}</p>
              <p class="text-gray-300 text-xs">#{{ pokemon.id }}</p>
            </div>
          </div>

          <!-- Pagination -->
          <div class="mt-6 flex flex-wrap items-center justify-center gap-3">
            <button
              :disabled="currentPage <= 1 || loading"
              @click="goToPage(currentPage - 1)"
              class="px-4 py-2 rounded-lg bg-[#2d2d2d] text-white font-semibold border-2 border-[#4a4a4a] disabled:opacity-50 disabled:cursor-not-allowed hover:bg-[#3d3d3d] hover:border-yellow-500/50 hover:scale-105 active:scale-95 transition-all duration-200"
            >
              ← Sebelumnya
            </button>
            <span class="text-gray-300 text-sm px-2">
              Halaman {{ currentPage }} dari {{ totalPages }}
            </span>
            <button
              :disabled="currentPage >= totalPages || loading"
              @click="goToPage(currentPage + 1)"
              class="px-4 py-2 rounded-lg bg-[#2d2d2d] text-white font-semibold border-2 border-[#4a4a4a] disabled:opacity-50 disabled:cursor-not-allowed hover:bg-[#3d3d3d] hover:border-yellow-500/50 hover:scale-105 active:scale-95 transition-all duration-200"
            >
              Selanjutnya →
            </button>
          </div>
          <div class="mt-2 flex justify-center gap-1 flex-wrap items-center">
            <template v-for="(item, index) in paginationItems" :key="item.type === 'ellipsis' ? `e-${index}` : item.num">
              <span v-if="item.type === 'ellipsis'" class="text-gray-500 px-1">...</span>
              <button
                v-else
                @click="goToPage(item.num)"
                :class="[
                  'w-9 h-9 rounded-lg font-semibold text-sm transition-all duration-200 hover:scale-110 active:scale-95',
                  currentPage === item.num
                    ? 'bg-yellow-500 text-[#1a1a1a] border-2 border-yellow-400 shadow-lg shadow-yellow-500/30'
                    : 'bg-[#2d2d2d] text-white border-2 border-[#4a4a4a] hover:border-yellow-500/50'
                ]"
              >
                {{ item.num }}
              </button>
            </template>
          </div>
        </div>
      </div>
      <div v-else key="detail" class="rounded-2xl border-4 border-[#2C2C2C] bg-[#1a1a1a] shadow-2xl shadow-black/50 overflow-hidden">
        <div class="rounded-xl border-2 border-[#4a4a4a] m-2 bg-[#0d0d0d] p-6" v-if="selectedPokemon">
        <div class="flex justify-end">
            <button @click="viewedAsDetail = false" class="mb-4 bg-yellow-500 hover:bg-yellow-400 hover:scale-105 active:scale-95 text-[#1a1a1a] font-bold px-4 py-2 rounded-lg transition-all duration-200">
              ← View All Pokemon
            </button>
        </div>

          <div class="flex flex-col md:flex-row gap-6">
            <!-- Gambar & info dasar -->
            <div class="flex-shrink-0 text-center md:w-1/3 animate-detail-in">
              <img
                :src="selectedPokemon.sprites?.front_default"
                :alt="selectedPokemon.name"
                class="w-56 h-56 mx-auto object-contain bg-[#2d2d2d] rounded-2xl p-4 border-2 border-[#4a4a4a] transition-transform duration-500 hover:scale-105"
              />
              <p class="text-white text-2xl font-bold mt-3 capitalize">{{ selectedPokemon.name }}</p>
              <p class="text-gray-500">#{{ String(selectedPokemon.id).padStart(3, '0') }}</p>
              <div class="flex flex-wrap justify-center gap-2 mt-2">
                <span
                  v-for="t in selectedPokemon.types"
                  :key="t.type.name"
                  class="px-3 py-1 rounded-full text-sm font-medium capitalize bg-[#2d2d2d] text-yellow-400 border border-[#4a4a4a]"
                >
                  {{ t.type.name }}
                </span>
              </div>
            </div>

            <!-- Stats & detail -->
            <div class="flex-1 space-y-4 animate-detail-in animate-delay-1">
              <div class="grid grid-cols-2 gap-4">
                <div class="bg-[#2d2d2d] rounded-xl p-3 border border-[#4a4a4a] transition-all duration-300 hover:border-yellow-500/30">
                  <p class="text-gray-500 text-xs uppercase">Tinggi</p>
                  <p class="text-white font-semibold">{{ selectedPokemon.height / 10 }} m</p>
                </div>
                <div class="bg-[#2d2d2d] rounded-xl p-3 border border-[#4a4a4a] transition-all duration-300 hover:border-yellow-500/30">
                  <p class="text-gray-500 text-xs uppercase">Berat</p>
                  <p class="text-white font-semibold">{{ selectedPokemon.weight / 10 }} kg</p>
                </div>
              </div>

              <div class="bg-[#2d2d2d] rounded-xl p-4 border border-[#4a4a4a] transition-all duration-300 hover:border-yellow-500/30">
                <p class="text-gray-500 text-xs uppercase mb-2">Base Stats</p>
                <div class="space-y-2">
                  <div v-for="s in selectedPokemon.stats" :key="s.stat.name" class="flex items-center gap-2">
                    <span class="text-gray-400 capitalize text-sm w-24">{{ s.stat.name.replace('-', ' ') }}</span>
                    <div class="flex-1 h-2 bg-[#1a1a1a] rounded-full overflow-hidden">
                      <div
                        class="h-full bg-[#DC0A2D] rounded-full transition-all duration-700 ease-out"
                        :style="{ width: Math.min(100, (s.base_stat / 255) * 100) + '%' }"
                      ></div>
                    </div>
                    <span class="text-white text-sm w-8">{{ s.base_stat }}</span>
                  </div>
                </div>
              </div>

              <div class="bg-[#2d2d2d] rounded-xl p-4 border border-[#4a4a4a]" v-if="selectedPokemon.abilities?.length">
                <p class="text-gray-500 text-xs uppercase mb-2">Abilities</p>
                <div class="flex flex-wrap gap-2">
                  <span
                    v-for="a in selectedPokemon.abilities"
                    :key="a.ability.name"
                    class="px-3 py-1 rounded-lg text-sm text-white bg-[#1a1a1a] capitalize"
                  >
                    {{ a.ability.name.replace('-', ' ') }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      </Transition>
    </div>

    <!-- Footer hint -->
    <p class="mt-8 text-white/70 text-sm animate-fade-in">Masukkan nama pokemon</p>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

.poke-title {
  font-family: 'Press Start 2P', cursive;
}

/* ========== Keyframes ========== */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
@keyframes cardIn {
  from {
    opacity: 0;
    transform: translateY(24px) scale(0.96);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
@keyframes glowDot {
  0%, 100% { opacity: 1; box-shadow: 0 0 6px #FECA1B; }
  50% { opacity: 0.85; box-shadow: 0 0 12px #FECA1B, 0 0 18px #FECA1B40; }
}
@keyframes pokeCardIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(10px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
@keyframes detailIn {
  from {
    opacity: 0;
    transform: translateX(-12px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* ========== Animasi masuk (page load) ========== */
.animate-fade-in {
  animation: fadeIn 0.5s ease-out forwards;
}
.animate-fade-down {
  animation: fadeInUp 0.5s ease-out forwards;
}
.pokedex-card {
  animation: cardIn 0.6s ease-out forwards;
}
.dot-yellow {
  animation: glowDot 2s ease-in-out infinite;
}

/* ========== Card Pokemon (stagger) ========== */
.poke-card {
  opacity: 0;
  animation: pokeCardIn 0.4s ease-out forwards;
}

/* ========== Loading spinner ========== */
.poke-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #4a4a4a;
  border-top-color: #FECA1B;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

/* ========== Transisi list ↔ detail ========== */
.view-switch-enter-active,
.view-switch-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}
.view-switch-enter-from {
  opacity: 0;
  transform: translateY(12px);
}
.view-switch-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ========== Halaman detail ========== */
.animate-detail-in {
  animation: detailIn 0.4s ease-out forwards;
}
.animate-delay-1 {
  opacity: 0;
  animation: detailIn 0.4s ease-out 0.15s forwards;
}
</style>
