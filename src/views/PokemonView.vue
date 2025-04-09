<script setup>
import { onMounted, ref, computed, onBeforeUnmount } from 'vue';
import axios from 'axios';
import PokemonCard from '../components/PokemonCard.vue';

const pokemons = ref([]);
const isLoading = ref(true);
const useOfficial = ref(false);
const darkMode = ref(false);
const searchQuery = ref('');
const showBackToTop = ref(false);
const movePage = ref(0);
const movePerPage = ref(4);

const offset = ref(0);
const limit = ref(50);
let loadingMore = false;

const selectedPokemon = ref(null);
const showModal = ref(false);

async function fetchPokemons() {
  if (loadingMore) return;
  loadingMore = true;
  isLoading.value = true;

  try {
    const response = await axios.get(
      `https://pokeapi.co/api/v2/pokemon?offset=${offset.value}&limit=${limit.value}`,
    );

    const newPokemons = response.data.results.map((poke) => {
      const id = poke.url.split('/').filter(Boolean).pop();
      return {
        name: poke.name,
        id,
        image: `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`,
      };
    });

    pokemons.value.push(...newPokemons);
    offset.value += limit.value;
  } catch (error) {
    console.error('Erro ao buscar mais pokémons:', error);
  } finally {
    loadingMore = false;
    isLoading.value = false;
  }
}

const filteredPokemons = computed(() => {
  const query = searchQuery.value.trim().toLowerCase();
  if (!query) return pokemons.value;

  return pokemons.value.filter(
    (poke) =>
      poke.name.toLowerCase().includes(query) || poke.id.toString() === query,
  );
});

async function openModal(pokemonId) {
  try {
    const response = await axios.get(
      `https://pokeapi.co/api/v2/pokemon/${pokemonId}`,
    );
    selectedPokemon.value = response.data;

    const speciesResponse = await axios.get(selectedPokemon.value.species.url);
    const evolutionChainUrl = speciesResponse.data.evolution_chain.url;
    const evolutiveResponse = await axios.get(evolutionChainUrl);

    const chain = evolutiveResponse.data.chain;
    selectedPokemon.value.evolutions = extractEvolutions(chain);

    const flavorText = speciesResponse.data.flavor_text_entries.find(
      (entry) => entry.language.name === 'pt' || entry.language.name === 'en',
    );
    selectedPokemon.value.description =
      flavorText?.flavor_text || 'Descrição não encontrada';

    showModal.value = true;
  } catch (error) {
    console.log('Error fetching pokemon details:', error);
  } finally {
    showModal.value = true;
  }
}

function extractEvolutions(chain) {
  const evolutions = [];

  function walkChain(node) {
    const id = node.species.url.split('/').filter(Boolean).pop();
    evolutions.push({
      name: node.species.name,
      id,
    });

    if (node.evolves_to.length > 0) {
      node.evolves_to.forEach((evolution) => {
        walkChain(evolution);
      });
    }
  }
  walkChain(chain);
  return evolutions;
}

function closeModal() {
  showModal.value = false;
  selectedPokemon.value = null;
}

function getStatColor(statName) {
  const colors = {
    hp: '#ef4444',
    attack: '#f59e0b',
    defense: '#3b82f6',
    'special-attack': '#8b5cf6',
    'special-defense': '#10b981',
    speed: '#f43f5e',
  };
  return colors[statName] || '#9ca3af';
}

function handleScroll() {
  const nearBottom =
    window.innerHeight + window.scrollY >= document.body.offsetHeight - 200;

  if (nearBottom) {
    fetchPokemons();
  }
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

function checkScrollPosition() {
  showBackToTop.value = window.scrollY > 300;
}

const currentMoves = computed(() => {
  if (!selectedPokemon.value?.moves) return [];
  const start = movePage.value * movePerPage.value;
  return selectedPokemon.value.moves.slice(start, start + movePerPage.value);
});

function nextMovePage() {
  if (
    movePage.value <
    Math.ceil(selectedPokemon.value.moves.length / movePerPage.value) - 1
  ) {
    movePage.value++;
  }
}

function prevMovePage() {
  if (movePage.value > 0) {
    movePage.value--;
  }
}

onMounted(() => {
  isLoading.value = true;
  fetchPokemons().finally(() => {
    isLoading.value = false;
  });
  window.addEventListener('scroll', handleScroll);
  window.addEventListener('scroll', checkScrollPosition);
});

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll);
  window.removeEventListener('scroll', checkScrollPosition);
});
</script>

<template>
  <div>
    <h2>Pokedex-Vue</h2>
    <button @click="useOfficial = !useOfficial">
      Mudar para: {{ useOfficial ? 'Sprite Padrão' : 'Imagem Oficial' }}
    </button>

    <div class="search-bar">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Buscar por nome ou ID"
      />
    </div>

    <div v-if="loadingMore && !isLoading" class="loading-more">
      Carregando mais Pokémons...
    </div>

    <ol v-else class="poke-list">
      <PokemonCard
        v-for="poke in filteredPokemons"
        :key="poke.id"
        :id="poke.id"
        :name="poke.name"
        :image="
          useOfficial
            ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${poke.id}.png`
            : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${poke.id}.png`
        "
        @click="openModal(poke.id)"
      />
    </ol>

    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <h3>{{ selectedPokemon.name.toUpperCase() }}</h3>
        <img
          :src="
            useOfficial
              ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${selectedPokemon.id}.png`
              : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selectedPokemon.id}.png`
          "
          :alt="selectedPokemon.name"
          width="150"
        />
        <p>
          <strong>Habilidades: </strong>
          <span
            v-for="ability in selectedPokemon.abilities"
            :key="ability.ability.name"
          >
            {{ ability.ability.name }}
            <span
              v-if="
                ability !==
                selectedPokemon.abilities[selectedPokemon.abilities.length - 1]
              "
              >,
            </span>
          </span>
        </p>
        <p><strong>Descrição:</strong> {{ selectedPokemon.description }}</p>

        <div class="stats">
          <h4>Status Base</h4>
          <div
            v-for="stat in selectedPokemon.stats"
            :key="stat.stat.name"
            class="stat-row"
          >
            <span class="stat-name">{{ stat.stat.name.toUpperCase() }}</span>
            <div class="stat-bar">
              <div
                class="stat-fill"
                :style="{
                  width: stat.base_stat + '%',
                  backgroundColor: getStatColor(stat.stat.name),
                }"
              ></div>
            </div>
            <span class="stat-value">{{ stat.base_stat }}</span>
          </div>
        </div>
        <div v-if="selectedPokemon.game_indices?.length">
          <h4>Jogos em que aparece:</h4>
          <div class="game-list">
            <span
              v-for="game in selectedPokemon.game_indices"
              :key="game.version.name"
              class="game-tag"
            >
              {{ game.version.name }}
            </span>
          </div>
        </div>
        <div class="move-carousel">
          <h4>Ataques</h4>
          <div class="carousel-controls">
            <button @click="prevMovePage">◀</button>
            <div class="moves">
              <div
                class="move-card"
                v-for="move in currentMoves"
                :key="move.move.name"
              >
                {{ move.move.name }}
              </div>
            </div>
            <button @click="nextMovePage">▶</button>
          </div>
        </div>

        <div class="evolutions" v-if="selectedPokemon.evolutions?.length > 1">
          <h4>Evoluções</h4>
          <div class="evo-list">
            <div
              v-for="evo in selectedPokemon.evolutions"
              :key="evo.id"
              class="evo-card"
              @click="openModal(evo.id)"
            >
              <img
                :src="
                  useOfficial
                    ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${evo.id}.png`
                    : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${evo.id}.png`
                "
                :alt="evo.name"
                width="60"
              />
              <p>{{ evo.name }}</p>
            </div>
          </div>
        </div>

        <p><strong>Altura:</strong> {{ selectedPokemon.height / 10 }} m</p>
        <p><strong>Peso:</strong> {{ selectedPokemon.weight / 10 }} kg</p>
        <p>
          <strong>Tipos: </strong>
          <span v-for="t in selectedPokemon.types" :key="t.type.name">
            {{ t.type.name }}
            <span
              v-if="
                t !== selectedPokemon.types[selectedPokemon.types.length - 1]
              "
              >,
            </span>
          </span>
        </p>
        <button class="close-btn" @click="closeModal">Fechar</button>
      </div>
    </div>
    <button
      v-if="showBackToTop"
      @click="scrollToTop"
      class="back-to-top"
      title="Voltar ao topo"
    >
      ↑
    </button>
  </div>
</template>

<style scoped>
div {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
  font-size: 2rem;
}

button {
  background-color: #3b82f6;
  color: white;
  border: none;
  padding: 10px 16px;
  border-radius: 8px;
  font-weight: bold;
  margin: 0 auto 20px;
  display: block;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

button:hover {
  background-color: #2563eb;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.poke-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 20px;
  padding: 0;
  list-style: none;
  max-width: 900px;
  margin: 0 auto;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

.modal {
  background: white;
  padding: 30px;
  border-radius: 16px;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  text-align: center;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  animation: slideUp 0.4s ease;
  position: relative;
}

.modal h3 {
  color: #1e40af;
  margin-bottom: 15px;
  font-size: 1.8rem;
}

.modal img {
  margin: 0 auto 15px;
  display: block;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.2));
}

.stats {
  background: #f9fafb;
  border-radius: 12px;
  padding: 15px;
  margin: 20px 0;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.stats h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #374151;
}

.stat-row {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.stat-name {
  width: 120px;
  font-weight: bold;
  font-size: 12px;
  text-align: left;
  color: #4b5563;
  text-transform: capitalize;
}

.stat-bar {
  flex: 1;
  height: 12px;
  background: #e5e7eb;
  border-radius: 6px;
  margin: 0 10px;
  overflow: hidden;
}

.stat-fill {
  height: 100%;
  border-radius: 6px;
  transition: width 0.6s ease;
}

.stat-value {
  width: 30px;
  font-size: 12px;
  text-align: right;
  font-weight: bold;
  color: #1f2937;
}

p {
  margin: 8px 0;
  color: #4b5563;
}

strong {
  color: #1f2937;
}

span {
  text-transform: capitalize;
}

.close-btn {
  background-color: #ef4444;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  font-weight: bold;
  margin-top: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.close-btn:hover {
  background-color: #dc2626;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

p[v-if='isLoading'] {
  text-align: center;
  font-size: 1.2rem;
  color: #6b7280;
  margin-top: 40px;
}

.evolutions {
  margin-top: 20px;
}
.evo-list {
  display: flex;
  gap: 10px;
  justify-content: center;
  flex-wrap: wrap;
}
.evo-card {
  cursor: pointer;
  background: #f3f4f6;
  border-radius: 10px;
  padding: 10px;
  width: 80px;
  text-align: center;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}
.evo-card:hover {
  transform: scale(1.05);
}

.search-bar {
  margin-bottom: 20px;
  display: grid;
  gap: 10px;
  justify-content: center;
}
.search-bar input {
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  width: 200px;
}
.search-bar button {
  padding: 6px 14px;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.search-bar button:hover {
  background-color: #2563eb;
}

.back-to-top {
  position: fixed;
  bottom: 30px;
  right: 30px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 50%;
  font-size: 24px;
  width: 48px;
  height: 48px;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  transition: background 0.3s;
  z-index: 999;
}

.back-to-top:hover {
  background: #2563eb;
}

.move-carousel {
  margin-top: 20px;
}
.carousel-controls {
  display: flex;
  align-items: center;
  justify-content: center;
}
.moves {
  display: grid;
  gap: 10px;
  overflow-x: auto;
}
.move-card {
  background: #f3f4f6;
  padding: 10px;
  border-radius: 8px;
  font-size: 14px;
  min-width: 100px;
  text-align: center;
}

.game-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  margin-top: 10px;
}

.game-tag {
  background: #e0e7ff;
  color: #1e40af;
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 12px;
  text-transform: uppercase;
  font-weight: 600;
  box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
}
</style>
