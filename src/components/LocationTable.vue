<template>
  <div class="location-table">
    <table class="table">
      <thead>
        <tr>
          <th>
            <input type="checkbox" v-model="selectAll" @change="selectAllRows" />
          </th>
          <th>Location</th>
          <th>Latitude</th>
          <th>Longitude</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="location in paginatedLocations" :key="location.id">
          <th>
            <input type="checkbox" v-model="selectedLocations" :value="location.id" />
          </th>
          <td>{{ location.name }}</td>
          <td>{{ location.latitude }}</td>
          <td>{{ location.longitude }}</td>
        </tr>
      </tbody>
    </table>

    <div>
      <button @click="deleteSelectedLocations" class="deleteButton">Delete Selected</button>
    </div>

    <div v-if="paginatedLocations.length > 0" class="pagination">
      <button v-for="page in pageCount" :key="page" @click="changePage(page)" class="pageButton">{{ page }}</button>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue';

export default {
  name: 'LocationTable',
  props: {
    locations: Array,
    currentLocation: Object
  },
  setup(props, { emit }) {
    const selectedLocations = ref([]);
    const currentPage = ref(1);
    const rowsPerPage = 10;
    const selectAll = ref(false);

    const pageCount = computed(() => {
      return Math.ceil(props.locations.length / rowsPerPage);
    });

    const paginatedLocations = computed(() => {
      const startIndex = (currentPage.value - 1) * rowsPerPage;
      const endIndex = startIndex + rowsPerPage;
      return props.locations.slice(startIndex, endIndex);
    });

    // select all rows
    function selectAllRows() {
      if (selectedLocations.value.length === paginatedLocations.value.length) {
        selectedLocations.value = [];
        selectAll.value = false;
      } else {
        selectedLocations.value = paginatedLocations.value.map((location) => location.id);
        selectAll.value = true;
      }
    }

    // change page
    function changePage(page) {
      currentPage.value = page;
    }

    // delete selected locations
    function deleteSelectedLocations() {
      const updatedLocations = props.locations.filter((location) => !selectedLocations.value.includes(location.id));
      emit('delete', updatedLocations);
      selectedLocations.value = [];
    }

    watch(
      () => props.locations.length,
      () => {
        if (currentPage.value > pageCount.value) {
          currentPage.value = pageCount.value;
        }
      }
    );

    // Event listener for 'addLocation' event
    // Listen to 'addLocation' event from Map component
    function addLocation(newLocation) {
      emit('add', newLocation);
    }

    return {
      selectedLocations,
      paginatedLocations,
      pageCount,
      selectAll,
      selectAllRows,
      changePage,
      deleteSelectedLocations,
      addLocation
    };
  },
};
</script>

<style scoped>
.location-table {
  margin-top: 80px;
  margin-bottom: 15px;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  padding: 8px;
  border: 1px solid #ddd;
}

.table th {
  background-color: #f2f2f2;
}

.deleteButton {
  margin-top: 8px;
  padding: 8px 16px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.deleteButton:hover {
  background-color: #c03329;
  transition: 0.3s;
}

.pageButton {
  padding: 8px 16px;
  background-color: #2dd0e6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pageButton:hover {
  background-color: #2aaabb;
  transition: 0.3s;
}

.pagination {
  margin-top: 16px;
}

.pagination button {
  margin-right: 8px;
}
</style>
