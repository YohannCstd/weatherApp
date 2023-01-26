<template>
  <v-app id="inspire">
    <v-app-bar >
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
        <div class="searchBar">
        <v-text-field @focus="showResults" v-model="searchCity" placeholder="Rechercher..." hide-details append-inner-icon="mdi-magnify" variant="solo" @input="findCity"/>

          <div class="close-results" @click="closeResults" v-if="searchResults.length > 0 && show">
            <v-list class="v-list-searchbar" @click.stop>
              <v-list-item @click="findMeteoWithName(result.name,result.latitude,result.longitude)" v-for="result in searchResults" :key="result.id">
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

    <v-main>
      <v-card class="mx-auto mt-4" max-width="368">
        <v-card-item :title="this.villeName">
          <template v-slot:subtitle>
            <v-icon icon="mdi-alert" size="18" color="error" class="mr-1 pb-1"></v-icon>
            Extreme Weather Alert
          </template>
        </v-card-item>

        <v-card-text class="py-0">
          <v-row style="align:center" no-gutters>
            <v-col class="text-h3" cols="6">
              {{ this.villeTemperature }}&deg;C
            </v-col>

            <v-col cols="6" class="text-right">
              <v-icon color="error" icon="mdi-weather-hurricane" size="88"></v-icon>
            </v-col>
          </v-row>
        </v-card-text>

        <div class="d-flex py-3 justify-space-between">
          <v-list-item density="compact" prepend-icon="mdi-weather-windy">
            <v-list-item-subtitle>{{ this.villeVent}} km/h</v-list-item-subtitle>
          </v-list-item>

          <v-list-item density="compact" prepend-icon="mdi-weather-pouring">
            <v-list-item-subtitle>{{ this.villeHumidite }}%</v-list-item-subtitle>
          </v-list-item>
        </div>

        <v-expand-transition>
          <div v-if="expand">
            <div class="py-2">
              <v-slider v-model="time" :max="6" :step="1" :ticks="labels" class="mx-4" color="primary" density="compact" hide-details show-ticks="always" thumb-size="10"></v-slider>
            </div>

            <v-list class="bg-transparent">
              <v-list-item v-for="item in forecast" :key="item.day" :title="item.day" :append-icon="item.icon" :subtitle="item.temp"></v-list-item>
            </v-list>
          </div>
        </v-expand-transition>

        <v-divider></v-divider>

        <v-card-actions>
          <v-btn @click="expand = !expand;">
            {{ !expand ? 'Full Report' : 'Hide Report' }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-main>
  </v-app>
</template>

<script>
import axios from 'axios';


export default {
  name: 'App',
  data () {
    return {
      //NavBar
      drawer: null,
      searchCity: '',
      searchResults: [],
      show: false,
      //HomeView
      labels: { 0: 'LU', 1: 'MA', 2: 'MER', 3: 'JE', 4: 'VE', 5: 'SA', 6: 'DI' },
      expand: false,
      time: 0,
      forecast: [
        { day: 'Mardi', icon: 'mdi-white-balance-sunny', temp: '24\xB0/12\xB0' },
        { day: 'Mercredi', icon: 'mdi-white-balance-sunny', temp: '22\xB0/14\xB0' },
        { day: 'Jeudi', icon: 'mdi-cloud', temp: '25\xB0/15\xB0' },
      ],
      villeName: "--",
      villeTemperature: "--",
      villeVent: "--",
      villeHumidite: "--",
    }
  },
  mounted (){
    this.getPosition();
  },
  methods:
  {
    // Affiche les résultats en dessous de searchbar
    showResults() {
        this.show = true
      },
    // Ferme les résultats en dessous de searchbar
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
    getPosition(){
      navigator.geolocation.getCurrentPosition((position) => {
        this.getCityName(position.coords.latitude, position.coords.longitude)
        this.findMeteo(position.coords.latitude, position.coords.longitude);
        this.findMeteoForThreeDay(position.coords.latitude, position.coords.longitude);
        this.searchCity = '';
      });
    },
    findMeteo(latitude, longitude)
    {
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          //Fermeture de la searchbar
          if(this.show) this.show =false;
          //Ajout des valeurs de la temparature et du vent en fonction de la réponse de l'API
          this.villeTemperature = response.data.current_weather.temperature;
          this.villeVent =  response.data.current_weather.windspeed;
          //Ajout de l'humidité
          response.data.hourly.time.forEach((humidity,index) => {
            if(humidity == response.data.current_weather.time){
              this.villeHumidite = response.data.hourly.relativehumidity_2m[index];
            }
          });
          // Vider la searchbar
          this.searchCity = '';
          this.searchResults = [];
        });
    },
    findMeteoWithName(name,latitude, longitude)
    {
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          //Fermeture de la searchbar
          if(this.show) this.show =false;
          this.villeTemperature = response.data.current_weather.temperature;
          this.villeVent =  response.data.current_weather.windspeed;
          this.villeName = name;
          //Ajout de l'humidité
          response.data.hourly.time.forEach((humidity,index) => {
            if(humidity == response.data.current_weather.time){
              this.villeHumidite = response.data.hourly.relativehumidity_2m[index];
            }
          });
          // Vider la searchbar
          this.searchCity = '';
          this.searchResults = [];
        });
    },
    findMeteoForThreeDay(latitude, longitude)
    {
      let url ="https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&start="+"2023-01-26"+"&end="+"2023-01-29"+"&current_weather=true";
      axios
        .get(url)
        .then((response) => 
        {
          console.log(response.data);
        });
    },





    getCityName(lat, lng) {
      axios.get("https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat="+lat+"&lon="+lng)
      .then((response) => 
      {
        // Ajout du nom de la ville en fonction de l'api
        this.villeName = response.data.address.city;
        // Vide la searchbar
        this.searchCity = '';
        this.searchResults = [];
      })
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
.v-list-searchbar {
  position: absolute;
  background-color: grey;
  left: 59px;
  right: 58px;
  margin-top: 2px;
}

.close-results {
  background: transparent;
  width: 100vw;
  height: 100vh;
  position: absolute;
  right: -58px;
}
</style>