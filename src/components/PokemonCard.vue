<template>
  <li class="poke-card" @click="$emit('click')">
    <img :src="image" :alt="name" />
    <p>{{ name }}</p>
  </li>
</template>

<script setup>
defineProps({
  id: [String, Number],
  name: String,
  image: String,
});
defineEmits(['click']);
</script>

<style scoped>
.poke-card {
  background: #f5f5f5;
  border-radius: 10px;
  padding: 10px;
  text-align: center;
  width: 100px;
  box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s;
}
.poke-card img {
  width: 96px;
  height: 96px;
  object-fit: contain;
  display: block;
  margin: 0 auto;
}

.poke-card:hover {
  transform: scale(1.05);
}
.types {
  margin-top: 6px;
}
.type-tag {
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 5px;
  margin: 2px;
  color: white;
  text-transform: capitalize;
}

.fire {
  background-color: #f08030;
}
.water {
  background-color: #6890f0;
}
.grass {
  background-color: #78c850;
}
.electric {
  background-color: #f8d030;
}
</style>
