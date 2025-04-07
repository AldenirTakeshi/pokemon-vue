<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';

const pokemons = ref([]);
const isLoading = ref(true);
const useOfficial = ref(false);
const darkMode = ref(false);

const selectedPokemon = ref(null);
const showModal = ref(false);

async function fetchPokemons() {
  try {
    const response = await axios.get(
      'https://pokeapi.co/api/v2/pokemon?limit=50',
    );
    pokemons.value = response.data.results.map((pokemon) => {
      const id = pokemon.url.split('/').filter(Boolean).pop();

      return {
        name: pokemon.name,
        id,
      };
    });
  } catch (error) {
    console.log('Error fetching pokemons:', error);
  } finally {
    isLoading.value = false;
  }
}

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

onMounted(() => {
  fetchPokemons();
});
</script>

<template>
  <div>
    <h2>Pokemons API GET</h2>
    <button @click="useOfficial = !useOfficial">
      Mudar para: {{ useOfficial ? 'Sprite Padrão' : 'Imagem Oficial' }}
    </button>

    <p v-if="isLoading">Carregando pokemons...</p>

    <ol v-else class="poke-list">
      <li
        v-for="poke in pokemons"
        :key="poke.id"
        class="poke-card"
        @click="openModal(poke.id)"
      >
        <img
          :src="
            useOfficial
              ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${poke.id}.png`
              : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${poke.id}.png`
          "
          :alt="poke.name"
          width="80"
          height="80"
        />
        <p>{{ poke.name }}</p>
      </li>
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
  </div>
</template>

<style scoped>
/* Estilos Gerais */
div {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
  font-size: 2rem;
}

/* Botão de Alternância */
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

/* Lista de Pokémons */
.poke-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 20px;
  padding: 0;
  list-style: none;
  max-width: 900px;
  margin: 0 auto;
}

.poke-card {
  background: linear-gradient(135deg, #f3f4f6, #e5e7eb);
  border-radius: 12px;
  padding: 15px 10px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid #e5e7eb;
}

.poke-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
  background: linear-gradient(135deg, #e5e7eb, #d1d5db);
}

.poke-card p {
  margin-top: 10px;
  font-weight: 600;
  color: #1f2937;
  text-transform: capitalize;
}

.poke-card img {
  transition: all 0.3s ease;
}

.poke-card:hover img {
  transform: scale(1.1);
}

/* Modal */
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

/* Estatísticas */
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

/* Tipos e Informações */
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

/* Botão Fechar */
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

/* Animações */
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

/* Loading */
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
</style>
