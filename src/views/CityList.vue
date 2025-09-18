<template>

  <div v-if="display">
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
  </div>
  <div v-else-if="savedCities.length == 0">
    <p>No locations added. To Start tracking a location, search in the field above.</p>
  </div>
  <div v-else>
    <CityCardSkeleton />
  </div>
</template>

<script>
import axios from "axios";
import CityCard from "@/views/CityCard.vue";
import {useRouter} from "vue-router";
import CityCardSkeleton from "@/components/CityCardSkeleton.vue";
export default {
  name: "CityList",
  components: {CityCardSkeleton, CityCard},
  data() {
    return {
      savedCities: [],
      weatherData: null,
      requests: [],
      display: false,
      router: useRouter()
    }
  },
  async created() {
     await this.getCities();
     console.log(this.savedCities);
  },
  methods: {
   async getCities() {
     if (localStorage.getItem("savedCities")) {
       const parsed = JSON.parse(localStorage.getItem("savedCities"));
       if (parsed.length > 0) {
         this.savedCities = parsed;

         this.savedCities.forEach((city) => {
           this.requests.push(axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&appid=f2594078fe0cccbee9e4edf1b08daae9&units=imperial`))
         })

         this.weatherData = await Promise.all(this.requests);

        await new Promise((res) => setTimeout(res, 1000));

         this.weatherData.forEach((value, index) => {
           this.savedCities[index].weather = value.data;
         })
         this.display = true;
         return;
       }
     }
     this.display = false;
   },
    goToCityView(city) {
     this.router.push({
       name: "cityView",
       params: {
         state: city.state,
         city: city.city
       },
       query: {
         id: city.id,
         lat: city.coords.lat,
         long: city.coords.lon,
       }
     })
    }
  }
}
</script>

