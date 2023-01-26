<template>
   <h2>Yo</h2>
</template>

<script>
import axios from 'axios'

  export default {
    data() {
      return{
        drawer: null,
        searchCity: '',
        searchResults: [],
        show: false,
      }
    },
    methods:{
      showResults() {
        this.show = true
      },
      closeResults() {
        this.show = false
      },
      findCity() {
        if(this.searchCity.length >= 2){
          axios
          .get("https://geocoding-api.open-meteo.com/v1/search?name="+this.searchCity+"&language=fr")
          .then((response) => 
          {
            this.searchResults = response.data.results;
            if(this.searchResults == null){
              this.searchResults = [];
            }
          });
        }
        if(this.searchCity.length < 2) {
          this.searchResults = [];
        }
      },
      findMeteo(latitude, longitude)
      {
        axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          console.log(response.data)
        });
      },
      getPosition(){
        navigator.geolocation.getCurrentPosition((position) => {
          this.findMeteo(position.coords.latitude, position.coords.longitude);
        });
      },
    }
  }
</script>

<style scoped>
.v-toolbar{
  overflow: visible;
}
.searchBar {
  position: relative;
  width: 100%;
  
}
.v-list {
  position: absolute;
  background-color: red;
  width: 100%;
  max-width: 560px;
  left: 59px;
}

.close-results {
  background: transparent;
  width: 100vw;
  height: 100vh;
  position: absolute;
  right: -58px;
}
</style>