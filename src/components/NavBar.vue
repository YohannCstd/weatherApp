<template>
   <v-app-bar >
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
        <div class="searchBar">
        <v-text-field @focus="showResults" v-model="searchCity" placeholder="Rechercher..." hide-details append-inner-icon="mdi-magnify" variant="solo" @input="findCity"/>

        <div class="close-results" @click="closeResults" v-if="searchResults.length > 0 && show">
          <v-list @click.stop>
            <v-list-item @click="findMeteo(result.name,result.latitude,result.longitude)" v-for="result in searchResults" :key="result.id">
              {{ result.name }} - {{ result.timezone }}
            </v-list-item>
          </v-list>
        </div>
      </div>
      
      <v-btn @click="getPosition()" icon>
        <v-icon>mdi-crosshairs-gps</v-icon>
      </v-btn>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer">
      <!--  -->
    </v-navigation-drawer>
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
      findMeteo(name,latitude, longitude){
        axios
          .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&hourly=temperature_2m")
          .then((response) => 
          {
            //Generer la date d'aujourd'hui et mettre les minutes à 0
            let date = new Date();
            date.setMinutes(0);
            let dateString = date.toISOString().slice(0, 16);
            let today = response.data.hourly.time;
            let temperatureToday = response.data.hourly.temperature_2m
            let temperature = "";
            //Chercher la temperature a la date et heure d'aujourd'hui
            for(let i = 0; i < today.length; i++){
              if(dateString == today[i]){
                temperature = temperatureToday[i];
              } 
            }
            console.log("à "+name+" il fait "+temperature+" le "+date)
          });
      },
      getPosition(){
        navigator.geolocation.getCurrentPosition((position) => {
          this.findMeteo("Mystere", position.coords.latitude, position.coords.longitude);
        });
      }
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