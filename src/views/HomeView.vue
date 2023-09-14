<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        @input="getSearchResults"
        type="text"
        placeholder="Search a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus: border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResult"
      >
        <li v-if="searchError" class="py-2 px-2 cursor-pointer">
          Sorry, something went wrong, please try again.
        </li>
        <li v-if="!searchError && mapboxSearchResult.length === 0" class="py-2 px-2 cursor-pointer">
          No results match your query, try a different term.
        </li>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResult"
            :key="searchResult.id"
            class="py-2 px-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />

        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '../components/CityList.vue'
import CityCardSkeleton from '../components/CityCardSkeleton.vue'

const router = useRouter()
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(', ')
  router.push({
    name: 'cityView',
    params: {
      state: state,
      city: city
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true
    }
  })
}

const mapboxAPIKey =
  'pk.eyJ1IjoibWVobWV0c2FoaW5kZXYiLCJhIjoiY2xtaWQ1c2kzMDVlczNxbnpyNDV0NTNwMSJ9.QvigNiAES5EdK8AGwI2cRw'
const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResult = ref(null)
const searchError = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value != '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        )
        mapboxSearchResult.value = result.data.features
      } catch {
        searchError.value = true
      }
      return
    }
    mapboxSearchResult.value = null
  }, 300)
}
</script>

<style lang="scss" scoped></style>
