<template>
    <div class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer mb-5">
      <div class="flex flex-col flex-1">
        <h2 class="max-md:text-xl md:text-3xl tracking-wide">{{ city.city }}</h2>
        <h3 class="tracking-wide">{{ city.state }}</h3>
      </div>

      <div class="flex flex-col gap-2">
        <p class="max-md:text-xl md:text-3xl self-end tracking-wide">
          {{ Math.round(city.weather.main.temp) }}&deg;
        </p>
        <div class="flex gap-2">
        <span class="text-xs">
          H:
          {{ Math.round(city.weather.main.temp_max) }}&deg;
        </span>
          <span class="text-xs">
          L:
          {{ Math.round(city.weather.main.temp_min) }}&deg;
        </span>
        </div>
      </div>
    </div>

</template>

<script>
export default {
  name: "CityCard",
  props: {
    city: {
      type: Object,
      default: {}
    }
  }
}
</script>
