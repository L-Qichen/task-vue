<script setup lang="ts">
import Leaflet from 'leaflet';
import { computed, onMounted, ref } from 'vue';
import 'leaflet/dist/leaflet.css';
import axios from 'axios';
import GetLocation from "./GetLocation.vue";

const map = ref<Leaflet.Map | null>(null);
const markers = ref<Leaflet.Marker[]>([]);
const searchQuery = ref('');
const latitude = ref(0);
const longitude = ref(0);
const locationName = ref('');
const locations = ref<Location[]>([]);
const currentPage = ref(1);
const itemsPerPage = ref(10);
const totalPages = computed(() => Math.ceil(locations.value.length / itemsPerPage.value));

class Location {
  name: string;
  latitude: number;
  longitude: number;
  selected: boolean;
  constructor(name: string, timeZone: string, latitude: number, longitude: number) {
    this.name = name;
    this.latitude = latitude;
    this.longitude = longitude;
    this.selected = false;
  }
}

onMounted(() => {
  map.value = Leaflet.map('map').setView([63, -106.3468], 3);
  Leaflet.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(map.value as Leaflet.Map);
});

const searchLocation = async () => {
  const location = searchQuery.value.trim();
  if (location === '') return;
  const url = `https://nominatim.openstreetmap.org/search?q=${location}&format=json`;
  try {
    const response = await axios.get(url);
    if (response.data.length > 0) {
      const result = response.data[0];
      const lat = parseFloat(result.lat);
      const lon = parseFloat(result.lon);

      latitude.value = lat;
      longitude.value = lon;
      locationName.value = result.display_name;
      markLocationOnMap(latitude.value, longitude.value, locationName.value);

      const locationData = {
        name: locationName.value,
        latitude: lat,
        longitude: lon,
        selected: false,
      };
      locations.value.push(locationData);
      console.log(url)
    } else {
      alert('Location not found.');
    }
  } catch (error) {
    alert(error);
  }
};

const markLocationOnMap = (latitude: number, longitude: number, locationName: string) => {
  const marker = Leaflet.marker([latitude, longitude]).addTo(map.value as Leaflet.Map);
  marker.bindPopup(locationName).openPopup();
  markers.value.push(marker);
  map.value?.setView([latitude, longitude], 17);
};

const paginatedLocations = computed(() => {
  const startIndex = (currentPage.value - 1) * itemsPerPage.value;
  const endIndex = startIndex + itemsPerPage.value;
  return locations.value.slice(startIndex, endIndex);
});

const previousPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
    console.log(currentPage.value);
  }
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
    console.log(currentPage.value);
  }
};

const deleteSelectedLocations = () => {
  const selectedLocations = locations.value.filter((location) => location.selected);
  selectedLocations.forEach((location) => {
    const marker = markers.value.find((m) => m.getLatLng().lat === location.latitude && m.getLatLng().lng === location.longitude);
    if (marker) {
      marker.remove();
      markers.value.splice(markers.value.indexOf(marker), 1);
    }
    locations.value.splice(locations.value.indexOf(location), 1);
  });
};

const isAnyLocationSelected = computed(() => {
  return locations.value.some(location => location.selected);
});
</script>

<template>
  <div class="min-h-screen flex flex-col justify-between">
    <div class="p-4 bg-gray-200">
      <div class="flex justify-center items-center space-x-2">
        <GetLocation />
        <input v-model="searchQuery" @keyup.enter="searchLocation" placeholder="Search Location"
          class="px-2 py-1 border border-gray-400 rounded" />
        <button @click="searchLocation"
          class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none">Search</button>
      </div>
    </div>
    <div class="flex justify-center">
      <table class="w-full mx-16 my-8 border border-gray-300 rounded">
        <thead>
          <tr>
            <th class="py-2 px-4 bg-gray-100">
              <button @click="deleteSelectedLocations" :disabled="!isAnyLocationSelected" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none 
                                  disabled:bg-gray-300 disabled:hover:bg-gray-300">
                Delete
              </button>
            </th>
            <th class="py-2 px-4 bg-gray-100">Location</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(location, index) in paginatedLocations" :key="index">
            <td class="py-2 px-4 bg-white border-b border-gray-300 text-center">
              <input type="checkbox" v-model="location.selected">
            </td>
            <td class="border-b border-gray-300 py-2 px-4 bg-white text-center">{{ location.name }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="flex justify-center py-2">
      <button @click="previousPage" :disabled="currentPage === 1" class="px-4 py-2 mx-6 bg-blue-500 text-white rounded hover:bg-blue-600 
                            focus:outline-none disabled:bg-gray-300 disabled:hover:bg-gray-300">
        Previous
      </button>
      <button @click="nextPage" :disabled="currentPage === totalPages || locations.length <= 10" class="px-4 py-2 mx-6 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none 
                            disabled:bg-gray-300 disabled:hover:bg-gray-300">
        Next
      </button>
    </div>
    <div id="map" class="border-red border-2"></div>
  </div>
</template>
<style scoped>
#map {
  height: 500px;
}
</style>