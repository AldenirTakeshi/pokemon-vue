<script setup>
import { ref } from 'vue';

const counter = ref(0);

function increment() {
  counter.value++;
}
</script>

<template>
  <button @click="increment">counter {{ counter }}</button>
</template>
