<template>
    <div class="flex flex-col flex-1 items-center">
<!--      Banner-->
      <div v-if="route.query.preview" class="max-md:text-sm md:text-base text-white p-4 bg-weather-secondary w-full text-center">
        <p>You are currently previewing this city, click the "+" icon to start tracking this city.</p>
      </div>

<!--      Weather Overview-->
      <div v-if="cityWeatherData" class="flex flex-col items-center text-white py-12">
        <h1 class="max-md:text-3xl md:text-5xl mb-2 tracking-wider">{{route.params.city}}</h1>
        <p class="max-md:mb-8 md:mb-12 max-md:text-sm md:text-base tracking-wide">
            {{
                new Date(cityWeatherData.data.currentTime).toLocaleDateString("en-us",
                  {
                    weekday: "short",
                    day: "2-digit",
                    month: "long",
                  }
                )
          }}
          {{
            new Date(cityWeatherData.data.currentTime).toLocaleTimeString("en-us",
              {
                timeStyle: "short",
              }
            )
          }}
        </p>
        <p class="max-md:text-5xl md:text-8xl mb-8 tracking-wide">{{ Math.round(cityWeatherData.data.current.temp) }}&deg;</p>
        <p class="max-md:text-sm md:text-base tracking-wider">Feels like {{ Math.round(cityWeatherData.data.current.feels_like) }}&deg;</p>
        <p class="capitalize max-md:text-sm md:text-base tracking-wider">{{cityWeatherData.data.current.weather[0].description}}</p>
        <img class="max-md:w-[135px] md:w-[150px] h-auto" :src="`http://openweathermap.org/img/wn/${cityWeatherData.data.current.weather[0].icon}@2x.png`" alt="Weather Img">
      </div>
    </div>
  <hr class="border-gray-400" />
  <!-- Hourly Weather -->
  <div v-if="cityWeatherData" class="w-full py-12">
    <div class="mx-8 text-white">
      <h2 class="mb-7 max-md:text-base md:text-lg font-bold tracking-wide">Hourly Weather</h2>
      <div class="flex gap-20 overflow-x-scroll scrollbar-hide">
        <div v-for="cityHourData in cityWeatherData.data.hourly" class="flex flex-col gap-4 items-center">
          <p class="whitespace-nowrap text-md">{{
            new Date(cityHourData.currentTime).toLocaleTimeString("en-us",
              {
                hour: "numeric",
              }
            )
            }}</p>
          <img :src="`http://openweathermap.org/img/wn/${cityHourData.weather[0].icon}@2x.png`" class="w-auto h-[50px] object-cover" alt="">
          <p class="text-xl">{{ Math.round(cityHourData.temp) }}</p>
        </div>
      </div>
    </div>
  </div>

  <hr class="border-gray-400" />

<!--  Weekly weather-->
  <div v-if="cityWeatherData" class="w-full py-12">
    <div class="mx-8 text-white">
      <h2 class="mb-4 max-md:text-base md:text-lg font-bold tracking-wide">7 Day Forecast</h2>
      <div class="flex items-center" v-for="day in cityWeatherData.data.daily" :key="day.dt">
        <p class="flex-1">
          {{new Date(day.dt * 1000).toLocaleDateString("en-us", {
          weekday: "long",
        }
        )
          }}
        </p>
        <img :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`" class="w-[50px] h-[50px] object-cover" />
        <div class="flex gap-2 flex-1 justify-end">
          <p>H:  {{Math.round(day.temp.max)}}&deg; </p>
          <p>L:  {{Math.round(day.temp.min)}}&deg; </p>
        </div>
      </div>
    </div>
  </div>

<!--  Removing City-->
  <div v-if="cityWeatherData" class="flex justify-center py-12">
    <div class="max-md:text-sm md:text-base flex gap-2 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity">
      <i class="ri-delete-bin-line"></i>
      <p class="tracking-wide">Remove City</p>
    </div>
  </div>

  <div v-else>
    <CityViewSkeleton />
  </div>



</template>

<script>
import axios from "axios";
import { useRoute } from 'vue-router';
import { useRouter } from "vue-router";
import CityViewSkeleton from "@/components/CityViewSkeleton.vue";

export default {
  name: "AsyncComponent",
  components: {CityViewSkeleton},
  data() {
    return {
      route: useRoute(),
      weatherData: null,
      localOffset: null,
      utc: null,
      utcForHourly: null,
      cityWeatherData: null,
      router: useRouter(),
      cities: null,
      updatedCities: null
    }
  },
  async created() {
      this.cityWeatherData =  await this.getWeatherData();
      console.log(this.cityWeatherData)
  },
  methods: {
    async getWeatherData() {
      try {
        this.weatherData = await axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${this.route.query.lat}&lon=${this.route.query.long}&exclude=part&appid=fd8ce65b37acb411221348466254432a&units=imperial`);
        // cal current date & time
        this.localOffset = new Date().getTimezoneOffset() * 60000;
        this.utc = this.weatherData.data.current.dt * 1000 + this.localOffset;
        this.weatherData.data.currentTime =
          this.utc + 1000 * this.weatherData.data.timezone_offset;

        // cal hourly weather offset
        this.weatherData.data.hourly.forEach((hour) => {
          this.utcForHourly = hour.dt * 1000 + this.localOffset;
          hour.currentTime =
            this.utcForHourly + 1000 * this.weatherData.data.timezone_offset;
        });

        await new Promise((res) => setTimeout(res, 1000));

        return this.weatherData

      }catch (error) {
        console.log(error)
      }
    },
    removeCity() {
      this.cities = JSON.parse(localStorage.getItem("savedCities"));
      this.updatedCities = this.cities.filter( city => city.id !== this.route.query.id );
      localStorage.setItem("savedCities", JSON.stringify(this.updatedCities));
      this.router.push({
        name: 'home',
      })
    }
  }
}
</script>

<style scoped>

</style>
