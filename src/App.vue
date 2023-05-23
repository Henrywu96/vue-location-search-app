<template>
  <div class="container">
    <h1>Location Search App</h1>
    <h4>By Min-Tzu Wu, 2023</h4>
    <Map @add-location="addLocation" @search="searchLocation" :location="currentLocation" />
    <h5>Left-click to add marker, right-click to delete marker</h5>
    <LocationTable :locations="locations" :currentLocation="currentLocation" @delete="deleteLocations" />
    <TimeZone v-if="currentLocation" :timeZone="currentLocation.timeZone" :localTime="currentLocation.localTime" :locations="locations" />
  </div>
</template>

<script>
import { ref } from 'vue';
import Map from './components/Map.vue';
import LocationTable from './components/LocationTable.vue';
import TimeZone from './components/TimeZone.vue';

export default {
  name: 'App',
  components: {
    Map,
    LocationTable,
    TimeZone,
  },
  setup() {
    const currentLocation = ref(null);
    const locations = ref([]);

    async function searchLocation(location) {
      const response = await fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(location)}.json?access_token=pk.eyJ1IjoiaGVucnl3dTk2IiwiYSI6ImNsaHdiZGU4MzBmaTQzc3A3eDBrN2l4aGYifQ.tzE4PT81QQ0iyj9kzAgbbg`);
      const data = await response.json();

      if (data.features && data.features.length > 0) {
        const { center } = data.features[0];
        const [longitude, latitude] = center;

        const newLocation = {
          id: Date.now(),
          name: location,
          latitude,
          longitude,
          timeZone: '',
          localTime: '',
        };

        locations.value.push(newLocation);
        currentLocation.value = newLocation;
      }
    }

    // add location
    async function addLocation(newLocation) {
      const response = await fetch(
        `https://api.timezonedb.com/v2.1/get-time-zone?key=DW3Q4NLXHUEW&format=json&by=position&lat=${newLocation.latitude}&lng=${newLocation.longitude}`
      );
      const data = await response.json();
      newLocation.timeZone = data.zoneName || 'Unknown';
      newLocation.localTime = data.datetime || 'Unknown';

      locations.value.push(newLocation);
      currentLocation.value = newLocation;
    }

    // delete locations
    function deleteLocations(updatedLocations) {
      locations.value.splice(0, locations.value.length, ...updatedLocations);
      currentLocation.value = null;
    }

    return {
      currentLocation,
      locations,
      searchLocation,
      addLocation,
      deleteLocations
    };
  },
};
</script>

<style scoped>

h1, h4 {
  display: flex;
  justify-content: center;
}

h5 {
  margin-top: 70px;
  margin-bottom: -70px;
  display: flex;
  justify-content: end;
}
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 16px;
}
</style>
