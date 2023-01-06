<template>
  <v-app id="inspire">
    <Navbar/>
    <v-main>
      <RouterView/>
    </v-main>
  </v-app>
</template>

<script>
import Navbar from '@/components/NavBar.vue'

  export default {
    components:{
      Navbar
    }
  }
</script>