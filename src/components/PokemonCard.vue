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
  background: linear-gradient(135deg, #f3f4f6, #e5e7eb);
  border-radius: 12px;
  padding: 15px 10px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid #e5e7eb;
}
.poke-card img {
  width: 96px;
  height: 96px;
  object-fit: contain;
  display: block;
  margin: 0 auto;
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
