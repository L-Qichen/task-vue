<script setup lang="ts">
import { ref } from 'vue';

const latitude = ref(0);
const longitude = ref(0);
const isVisible = ref(false);

const getLocation = () => {
  navigator.geolocation.getCurrentPosition(
    (pos) => {
      latitude.value = pos.coords.latitude;
      longitude.value = pos.coords.longitude;
      isVisible.value = true;
    },
    (err) => {
      isVisible.value = false;
      console.warn("ERROR(" + err.code + "): " + err.message);
    }
  );
};
</script>

<template>
  <div class="flex">
    <div v-if="isVisible" class="flex absolute left-10 border border-gray-600 rounded">
      <span class="mx-2">Your Location:</span>
      <span class="mx-2">latitude: {{ latitude }}</span>
      <span class="mx-2">longitude: {{ longitude }}</span>
    </div>
    <button @click="getLocation" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none">
      Location
    </button>
  </div>
</template>

<style scoped></style>
