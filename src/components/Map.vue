<template>
  <div class="map-container">
    <div class="search-container">
      <input type="text" v-model="searchQuery" @keydown.enter="searchLocation" placeholder="Search location" class="input">
      <button @click="searchLocation" class="button">Search</button>
    </div>
    
    <!-- get current location button -->
    <button @click="getCurrentLocation" class="currentButton">Get Current Location</button>
    <div class="map" ref="mapContainer"></div>
  </div>
</template>

<script>
import { onMounted, ref, watch, nextTick } from 'vue';
import mapboxgl from 'mapbox-gl';

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Map',
  props: {
    locations: Array,
    currentLocation: Object,
  },
  setup(props, { emit }) {
    const mapContainer = ref(null);
    const map = ref(null);
    const markers = ref([]);
    const searchQuery = ref('');

    onMounted(async () => {
      await nextTick();

      mapboxgl.accessToken = 'pk.eyJ1IjoiaGVucnl3dTk2IiwiYSI6ImNsaHdiZGU4MzBmaTQzc3A3eDBrN2l4aGYifQ.tzE4PT81QQ0iyj9kzAgbbg';
      map.value = new mapboxgl.Map({
        container: mapContainer.value,
        style: 'mapbox://styles/mapbox/streets-v12',
      });

      map.value.on('load', () => {
        // Create markers when the map is ready
        watch(() => props.locations, (newLocations) => {
          if (newLocations.length > 0) {
            map.value.flyTo({
              center: [newLocations[0].longitude, newLocations[0].latitude],
              zoom: 12,
            });

            // Clear previous markers
            clearMarkers();

            // Add new markers
            newLocations.forEach((location) => {
              const marker = new mapboxgl.Marker()
                .setLngLat([location.longitude, location.latitude])
                .addTo(map.value);

              markers.value.push(marker);
            });
          }
        });

        // Add marker on left-click
        map.value.on('click', (e) => {
          const { lng, lat } = e.lngLat;
          const marker = new mapboxgl.Marker()
            .setLngLat([lng, lat])
            .addTo(map.value);

          markers.value.push(marker);
        });

        // Clear markers on right-click
        map.value.on('contextmenu', () => {
          clearMarkers();
        });
        
      });
    });

    // clear markers
    function clearMarkers() {
      markers.value.forEach((marker) => marker.remove());
      markers.value = [];
    }

    // get current location
    function getCurrentLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const { longitude, latitude } = position.coords;
            const marker = new mapboxgl.Marker()
              .setLngLat([longitude, latitude])
              .addTo(map.value);
          
            markers.value.push(marker);
          
            map.value.flyTo({
              center: [longitude, latitude],
              zoom: 12,
            });
          },
          (error) => {
            console.error('Error getting current location:', error);
          }
        );
      } else {
        console.error('Geolocation is not supported by this browser.');
      }
    }

    async function searchLocation() {
      if (searchQuery.value) {
        const response = await fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(searchQuery.value)}.json?access_token=${mapboxgl.accessToken}`);
        const data = await response.json();

        if (data.features && data.features.length > 0) {
          const {center} = data.features[0];
          const [longitude, latitude] = center;
          const marker = new mapboxgl.Marker()
            .setLngLat([longitude, latitude])
            .addTo(map.value);
          
          markers.value.push(marker);
          
          map.value.flyTo({
            center: [longitude, latitude],
            zoom: 12,
          });

          const newLocation = {
            id: Date.now(),
            name: searchQuery.value,
            latitude,
            longitude,
            timeZone: '',
            localTime: '',
          };

          emit('addLocation', newLocation);
        } else {
          console.error('No location found for the search query:', searchQuery.value);
        }
      }
    }

    return {
      mapContainer,
      getCurrentLocation,
      searchLocation,
      searchQuery
    };
  },
};
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 400px;
  margin-top: 20px;
}

.input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.currentButton {
  background-color: #1bbe0d;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.currentButton:hover {
  background-color: #28911e;
  transition: 0.3s;
}

.button {
  margin-left: 8px;
  padding: 8px 16px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.button:hover {
  background-color: #0f75c9;
  transition: 0.3s;
}

.search-container {
  display: flex;
  margin-bottom: 16px;
}

.map {
  width: 100%;
  height: 100%;
}
</style>
