<template>
  <div class="time-zone">
    <div v-if="latestLocation" class="location-info">
      <p><strong>Location:</strong> {{ latestLocation.name }}</p>
      <p><strong>Timezone:</strong> {{ latestLocation.timezone }}</p>
      <p><strong>Local Time:</strong> {{ latestLocation.localTime }}</p>
    </div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';

export default {
  name: 'TimeZone',
  props: {
    locations: {
      type: Array,
      default: () => [],
    },
  },
  setup(props) {
    const latestLocation = ref(null);

    watchEffect(() => {
      const latestLocationIndex = props.locations.length - 1;
      if (latestLocationIndex >= 0) {
        const latestLocationData = props.locations[latestLocationIndex];
        fetchTimezoneAndLocalTime(latestLocationData);
      }
    });

    // fetch time zone and local time of the latest searched location
    async function fetchTimezoneAndLocalTime(location) {
      const response = await fetch(
        `https://api.timezonedb.com/v2.1/get-time-zone?key=DW3Q4NLXHUEW&format=json&by=position&lat=${location.latitude}&lng=${location.longitude}`
      );
      const data = await response.json();
      latestLocation.value = {
        name: location.name,
        timezone: data.zoneName || 'Unknown',
        localTime: data.formatted || 'Unknown'
      };
    }

    return {
      latestLocation
    };
  },
};
</script>

<style scoped>
.time-zone {
  margin-bottom: 16px;
}

.location-info {
  background-color: #f2f2f2;
  padding: 16px;
}
</style>
