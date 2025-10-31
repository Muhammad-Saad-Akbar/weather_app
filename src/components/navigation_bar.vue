<template>
  <div class="sticky top-0 bg-weather-primary shadow-lg">
    <nav class="max-md:px-5 md:px-[2rem] flex items-center gap-4 text-white py-6 mx-auto">

      <RouterLink :to="{ name: 'home'}">
        <div class="flex items-center gap-3">
          <i class="ri-sun-fill max-md:text-lg md:text-xl"></i>
          <p class="max-md:text-base md:text-xl">The Local Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i class="ri-information-fill max-md:text-lg md:text-xl hover:text-weather-secondary duration-150 cursor-pointer" @click="toogleModal"></i>
        <i class="ri-add-line max-md:text-lg md:text-xl hover:text-weather-secondary duration-150 cursor-pointer" @click="addCity" v-if="route.query.preview"></i>
      </div>
    </nav>
  </div>
</template>

<script>
import {RouterLink} from "vue-router";
import Swal from 'sweetalert2'
import { useRouter } from "vue-router";
import { uid } from "uid"
import { useRoute } from "vue-router";

export default {
  name: "navigation_bar",
  components: {
    RouterLink
  },
  data() {
    return {
      savedCities: [],
      locationObj: {},
      router: useRouter(),
      route: useRoute(),
      updatedQuery: null
    }
  },
  methods: {
    toogleModal() {
      Swal.fire({
        width: 880,
        html: `
    <h1 class="text-left font-bold text-xl">About:</h1><p class="text-left pt-1">The Local Weather allows you to track the current and future weather of cities of your choosing.</p>
    <h1 class="text-left font-bold pt-4 text-xl">How it works:</h1>
    <p class="text-left pt-0.5">1, Search for your city by entering the name into the search bar.</p>
    <p class="text-left pt-0.5">2, Select a city within the results, this will take you to the current weather for your selection.</p>
    <p class="text-left pt-0.5">3, Track the city by clicking on the '+' icon in the top right. This will save the city to view at a later time on the home page.</p>
    <h1 class="text-left font-bold pt-4 text-xl">Removing a city</h1>
    <p class="text-left pt-0.5">If you no longer wish to track a city, simply select the city within the home page. At the bottom of the page, there will be an option to delete the city.</p>
  `,
        position: 'top',
        customClass: {
          popup: 'mt-32'
        },
        showCancelButton: true,
        showConfirmButton: false,
        cancelButtonText: `Close`,
        allowOutsideClick: false,
        cancelButtonColor: '#00668A'
      });
    },
    addCity() {
      if (localStorage.getItem("savedCities")) {
        this.savedCities = JSON.parse(localStorage.getItem("savedCities"));
      }
      this.locationObj = {
        id: uid(),
        state: this.route.params.state,
        city: this.route.params.city,
        coords: {
          lat: this.route.query.lat,
          lon: this.route.query.long
        }
      }
      this.savedCities.push(this.locationObj);
      localStorage.setItem("savedCities", JSON.stringify(this.savedCities));
      this.updatedQuery = Object.assign({}, this.route.query);
      delete this.updatedQuery.preview;
      this.updatedQuery.id = this.locationObj.id;
      this.router.replace( {
        query: this.updatedQuery
      });
    }
  }
}
</script>

<style scoped>

</style>
