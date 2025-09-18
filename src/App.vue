<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-weather-primary">
    <SiteNavigation />
    <RouterView />
  </div>
</template>



<script>
import { RouterLink, RouterView } from 'vue-router'
import SiteNavigation from './components/navigation_bar.vue'
export default {
  components: {
    RouterView,
    SiteNavigation
  }
}
</script>


<style scoped>

</style>
