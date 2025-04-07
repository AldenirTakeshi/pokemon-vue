<script setup>
import { ref } from 'vue';
import { onMounted } from 'vue';

onMounted(() => {
  console.log('Componente Diretivas montado!');
});

const pokemons = ref(['Bulbasaur', 'Charmander', 'Squirtle', 'Pikachu']);

const pokemonShowList = ref(true);
const newPokemon = ref('');
</script>

<template>
  <div>
    <input type="text" v-model="newPokemon" placeholder="Novo Pokemon" />
    <button
      @click="
        pokemons.push(newPokemon);
        newPokemon = '';
      "
    >
      Adicionar
    </button>
    <button @click="pokemonShowList = !pokemonShowList">
      Ocultar/Mostrar Lista
    </button>

    <ul v-if="pokemonShowList">
      <li v-for="(poke, index) in pokemons" :key="index">
        {{ poke }}
      </li>
    </ul>
  </div>
</template>
