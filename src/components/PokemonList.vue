<script setup>
import { ref } from 'vue';

const pokemons = ref(['Bulbasaur', 'Pikachu', 'Chimchar']);
const ShowList = ref(false);
const newPokemon = ref('');

function addPokemon() {
  const name = newPokemon.value.trim();
  if (name) {
    pokemons.value.push(name);
    newPokemon.value = '';
  }
}

function removePokemon(index) {
  pokemons.value.splice(index, 1);
}

function toggleList() {
  ShowList.value = !ShowList.value;
}
</script>

<template>
  <div>
    <h2>Lista de Pokemons</h2>
    <input type="text" v-model="newPokemon" />
    <button @click="addPokemon">Adicionar</button>
    <button @click="toggleList">Mostra/Ocultar</button>

    <ul v-if="ShowList">
      <li v-for="(poke, index) in pokemons" :key="index">
        {{ poke }} <button @click="removePokemon(index)">Remover</button>
      </li>
    </ul>
  </div>
</template>
