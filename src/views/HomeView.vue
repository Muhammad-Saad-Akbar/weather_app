<template>
  <main class="max-md:px-6 md:px-[2rem] text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a city or state" class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none
      focus:shadow-[0px_1px_0_0_#004E71]" v-model="searchQuery" @input="searchQueryResult"
      />
      <ul class="absolute bg-weather-primary w-full shadow-md py-2 px-1 top-[66px] text-white" v-if="mapBoxSearchResult">
          <p class="max-md:text-sm md:text-base py-2" v-if="searchError">Sorry, something went wrong, please try again.</p>
          <p class="max-md:text-sm md:text-base py-2" v-if="!serverError && mapBoxSearchResult.length == 0">No results match your query, try a different term.</p>
        <template v-else>
          <li class="py-2 cursor-pointer" v-for="searchResult in mapBoxSearchResult" :key="searchResult.id" @click="previewCity(searchResult)">
            {{ searchResult.properties.full_address }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <CityList />
    </div>
  </main>
</template>

<script>
import axios from 'axios';
import { useRouter } from 'vue-router';
import CityList from "@/views/CityList.vue";

export default {
  components: {CityList},
  data() {
    return {
      searchQuery: "",
      queryTimeout: null,
      mapboxApiKey: "pk.eyJ1Ijoic2FhZC1ha2JhciIsImEiOiJjbWJsZ3RtNHgwcGEwMmlxeThqNjRneDBxIn0.qCPd21uiPEXOpfOt3MQcLw",
      result: null,
      mapBoxSearchResult: null,
      searchError: null,
      city: '',
      state: '',
      router: useRouter()
    }
  },
  methods: {
    searchQueryResult() {
      clearTimeout(this.queryTimeout)
      this.queryTimeout = setTimeout(async () => {
      if (this.searchQuery != null) {
        try {
          this.result = await axios.get(`https://api.mapbox.com/search/geocode/v6/forward?q=${this.searchQuery}&types=place&access_token=${this.mapboxApiKey}`)
          this.mapBoxSearchResult = this.result.data.features
        }catch {
          this.searchError = true
        }
        return;
      }
        this.mapBoxSearchResult = null
      }, 300)
    },
    previewCity(searchResult) {
      console.log(searchResult)
      const parts = searchResult.properties.full_address.split(",");
      this.city = parts[0]?.trim();
      this.state = parts[1]?.trim();
      this.router.push({
        name: "cityView",
        params: {
          state: this.state,
          city: this.city
        },
        query: {
          lat: searchResult.geometry.coordinates[1],
          long: searchResult.geometry.coordinates[0],
          preview: true
        }
      })
    }
  }
}
</script>
