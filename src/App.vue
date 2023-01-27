<template>
  <v-app id="inspire">
    <v-app-bar >
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
        <div class="searchBar">
        <v-text-field @focus="showResults" v-model="searchCity" placeholder="Rechercher..." hide-details append-inner-icon="mdi-magnify" variant="solo" @input="findCity"/>

          <div class="close-results" @click="closeResults" v-if="searchResults.length > 0 && show">
            <v-list class="v-list-searchbar" @click.stop>
              <v-list-item @click="findMeteoWithName(result.name,result.latitude,result.longitude,result.country_code,result.admin1)" v-for="result in searchResults" :key="result.id">
                {{ result.name }} - {{ result.admin1 }} ({{ result.country_code }})
              </v-list-item>
            </v-list>
          </div>
        </div>
      
      <v-btn @click="getPosition()" icon>
        <v-icon>mdi-crosshairs-gps</v-icon>
      </v-btn>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer">
      <h3 style="text-align: center;" class="mt-1">Historique</h3>
      <div v-if="searchHistory.length == 0" style="text-align: center;">
        <p>Votre historique est vide !</p>
      </div>
      <v-list>
        <v-list-item class="v-list-close" v-for="history in searchHistory" :key="history" @click="findMeteoWithName(history.villeName,history.latitude,history.longitude,history.countryCode,history.department)">
          {{ history.villeName }} - {{ history.department }} ({{ history.countryCode }}) <v-icon @click="removeCityFromHistory(history)" class="ml-2" small>mdi-close</v-icon>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-card class="mx-auto mt-4" max-width="368">
        <v-card-item :title="this.villeName">
          <template v-slot:subtitle>
            <v-icon icon="mdi-information-outline" size="18" class="mr-1 pb-1 mt-1"></v-icon>
            {{ this.descrpWeather }}
          </template>
        </v-card-item>

        <v-card-text class="py-0">
          <v-row style="align:center" no-gutters>
            <v-col class="text-h3" cols="6">
              {{ this.villeTemperature }}&deg;C
            </v-col>

            <v-col cols="6" class="text-right">
              <v-icon color="error" :icon="this.weatherIcon" size="88"></v-icon>
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
            <v-list class="bg-transparent">
              <v-list-item v-for="item in forecast" :key="item.day" :title="item.day" :append-icon="item.icon" :subtitle="item.temp"></v-list-item>
            </v-list>
          </div>
        </v-expand-transition>

        <v-divider></v-divider>

        <v-card-actions>
          <v-btn @click="expand = !expand">
            {{ !expand ? 'Rapport complet' : 'Fermer le rapport' }}
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
      searchHistory: [],
      show: false,
      //HomeView
      expand: false,
      forecast: [],
      villeName: "--",
      descrpWeather: "",
      villeTemperature: "--",
      villeVent: "--",
      villeHumidite: "--",
      weatherIcon: "",
    }
  },
  mounted (){
    // Au lancement de l'application la fonction se lance
    this.getPosition();
  },
  created() {
    // Récupérer l'historique de recherche stocké dans localStorage
    let storedHistory = localStorage.getItem('searchHistory');

    // Si l'historique existe, mettre à jour la propriété searchHistory du composant
    if (storedHistory) {
      this.searchHistory = JSON.parse(storedHistory);
    }
  },
  methods:
  {
    // Affiche les résultats en dessous de searchbar
    showResults() { this.show = true },
    // Ferme les résultats en dessous de searchbar
    closeResults() { this.show = false },
    // J'utilise cette fonction pour l'autocompletion
    findCity() {
      // Je commence à appeler l'api si la personne a mis au moins 2 caractères
      if(this.searchCity.length >= 2){
      axios
        .get("https://geocoding-api.open-meteo.com/v1/search?name="+this.searchCity+"&language=fr")
        .then((response) => 
        {
          // L'api me donne les 10 villes en fonction de ce que l'utilisateur a écrit
          this.searchResults = response.data.results;
          // Et s'il n'y a pas de résultat de retourne une liste vide
          if(this.searchResults == null){
            this.searchResults = [];
          }
        });
      }
      // Si il y a moins de deux caractères je vide la liste des résultats
      if(this.searchCity.length < 2) {
        this.searchResults = [];
      }
    },
    // Cette fonction permet de me donner le nom d'une ville en focntion de la latitude et la longitude
    // Ici elle me sert au démarage de l'application elle me donne le nom de l'endroit ou se situe l'utilisateur
    getCityName(lat, lng) {
      axios.get("https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat="+lat+"&lon="+lng)
      .then((response) => 
      {
        // Ajout du nom de la ville en fonction de l'api
        this.villeName = response.data.address.city;
      })
    },
    // Cette fonction permet de donner le nom de la ville 
    // et la météo d'aujourd'hui et les 3 jours suivant à la position de l'utilisateur
    getPosition(){
      navigator.geolocation.getCurrentPosition((position) => {
        this.getCityName(position.coords.latitude, position.coords.longitude)
        this.findMeteo(position.coords.latitude, position.coords.longitude);
        this.updateForecast(position.coords.latitude, position.coords.longitude);
        this.searchCity = '';
      });
    },
    // Cette fonction permet de donner la météo au démarrage de l'application en fonction de la latitude et longitude donnée
    findMeteo(latitude, longitude)
    {
      this.drawer = false;
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          //Fermeture de la searchbar
          if(this.show) this.show =false;
          // Message pour décrire la météo
          this.descrpWeather = this.getDescriptionWeatherCode(response.data.current_weather.weathercode);
          // Icon pour décrire la météo
          this.weatherIcon = this.getIconWeatherCode(response.data.current_weather.weathercode);
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
    // Cette fonction permet de donner la météo quand l'utilisateur utilise la searchbar 
    // en fonction du nom de la ville, de la latitude et longitude donnée
    findMeteoWithName(name,latitude, longitude,countryCode,department)
    {
      // Fermer la navbar drawer
      this.drawer = false;
      // Ajouter la ville recherchée à l'historique de recherche
      this.addInLocalStorage(name,department,latitude,longitude,countryCode),
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          // Mise à jour de forecast
          this.updateForecast(latitude,longitude)
          // Message pour décrire la météo
          this.descrpWeather = this.getDescriptionWeatherCode(response.data.current_weather.weathercode);
          // Icon pour décrire la météo
          this.weatherIcon = this.getIconWeatherCode(response.data.current_weather.weathercode);
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
    dateNowAndThreeday(){
      // Ici ces lignes de code me permettent d'avoir la date d'aujourd'hui au format yyyy-mm-dd
      // Ainsi que la date d'en trois jours
      const today = new Date();
      const dateStart = new Intl.DateTimeFormat('fr-FR', { year: 'numeric', month: '2-digit', day: '2-digit' }).format(new Date());
      const threeDaysLater = new Date(today.getTime() + (3 * 24 * 60 * 60 * 1000));
      const dateEnd = new Intl.DateTimeFormat('fr-FR', { year: 'numeric', month: '2-digit', day: '2-digit' }).format(threeDaysLater);
      return {dateStart, dateEnd};
    },
    // Cette fonction permet de mettre à jour la météo des 3 jours suivant.
    updateForecast(latitude,longitude) {
      // Je vide la météo des trois jours
      this.forecast = [];
      // Ces trois lignes permettent de savoir quel jour on est et d'avoir les trois suivant
      // Ex on est mardi -> mercredi,jeudi,vendredi
      const today = new Date();
      const dayOfWeek = today.getDay(); // 0 = dimanche, 1 = lundi, etc.
      const days = ['Dimanche', 'Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi'];
      let dateStart,dateEnd = this.dateNowAndThreeday();
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&start="+dateStart+"&end="+dateEnd+"&daily=temperature_2m_min&daily=temperature_2m_max&daily=weathercode&timezone=GMT")
        .then(response => {
          // Les trois variables suivantes me donnes les informations sur les 7 jours avenir
          const tempMax = response.data.daily.temperature_2m_max;
          const tempMin = response.data.daily.temperature_2m_min;
          const weatherCode = response.data.daily.weathercode;
          // Dans mon for j'ai mis le i = 1 car le 0 est le jour d'aujourd'hui
          // Puis j'ai mis 4 car je veux que les trois jours suivants
          for (let i = 1; i < 4; i++) {    
            let jour = {
              day: days[(dayOfWeek + i) % 7],
              icon: this.getIconWeatherCode(weatherCode[i]),
              temp: tempMax[i]+'\xB0/'+tempMin[i]+'\xB0'
            }
            this.forecast.push(jour);
          }
        })  
    },
    // Cette fonction permet d'interpreter le weatherCode renvoyé par l'api et ainsi envoyé la bonne icone
    getIconWeatherCode(code){
    let icon;
    switch (code) {
      case 0:
        icon = "mdi-weather-sunny";
        break;
      case 1:
        icon = "mdi-weather-partly-cloudy";
        break;
      case 2:
        icon = "mdi-weather-partly-cloudy";
        break;
      case 3:
        icon = "mdi-weather-partly-cloudy";
        break;
      case 45:
        icon = "mdi-weather-fog";
        break;
      case 48:
        icon = "mdi-weather-fog";
        break;
      case 51:
        icon = "mdi-weather-rainy";
        break;
      case 53:
        icon = "mdi-weather-rainy";
        break;
      case 55:
        icon = "mdi-weather-rainy";
        break;
      case 56:
        icon = "mdi-weather-snowy-rainy";
        break;
      case 57:
        icon = "mdi-weather-snowy-rainy";
        break;
      case 61:
        icon = "mdi-weather-pouring";
        break;
      case 63:
        icon = "mdi-weather-pouring";
        break;
      case 65:
        icon = "mdi-weather-pouring";
        break;
      case 66:
        icon = "mdi-weather-hail";
        break;
      case 67:
        icon = "mdi-weather-hail";
        break;
      case 71:
        icon = "mdi-weather-snowy";  
        break;
      case 73:
        icon = "mdi-weather-snowy";  
        break;
      case 75:
        icon = "mdi-weather-snowy";  
        break;
      case code == 77:
        icon = "mdi-weather-hail";
        break;
      case 80:
        icon = "mdi-weather-lightning-rainy";
        break;
      case 81:
        icon = "mdi-weather-lightning-rainy";
        break;
      case 82:
        icon = "mdi-weather-lightning-rainy";
        break;
      case 85:
        icon = "mdi-weather-snowy-rainy";
        break;
      case 86:
        icon = "mdi-weather-snowy-rainy";
        break;
      case code == 95:
        icon = "mdi-weather-lightning";
        break;
      case 96:
        icon = "mdi-weather-hail";
        break;
      case 99:
        icon = "mdi-weather-hail";
        break;
      default:
        icon = "";
        break;
    }
    return icon;
    },
    // Cette fonction permet d'interpreter le weatherCode renvoyé par l'api et ainsi envoyé la bonne description
    getDescriptionWeatherCode(code){
    let description = "";
    switch (code) {
      case 0:
        description = "Ciel clair";
        break;
      case 1:
        description = "Plutôt dégagé";
        break;
      case 2:
        description = "Partiellement nuageux"
        break;
      case 3:
        description = "Couvert";
        break;
      case 45:
        description = "Brouillard";
        break;
      case 48:
        description = "Dépôt de brouillard givré";
        break;
      case 51:
        description = "Bruine légère";
        break;
      case 53:
        description = "Bruine modérée"
        break;
      case 55:
        description = "Bruine dense";
        break;
      case 56:
        description = "Légère bruine verglaçante";
        break;
      case 57:
        description = "Dense bruine verglaçante";
        break;
      case 61:
        description = "Pluie faible";
        break;
      case 63:
        description = "Pluie modérée";
        break;
      case 65:
        description = "Forte pluie";
        break;
      case 66:
        description = "Légère pluie verglaçante";
        break;
      case 67:
        description = "Forte pluie verglaçante";
        break;
      case 71:
        description = "Légère chute de neige";  
        break;
      case 73:
        description = "Chute de neige modérée";
        break;
      case 75:
        description = "Forte chute de neige";
        break;
      case 77:
        description = "grêle";
        break;
      case 80:
        description = "Légères averses de pluie";
        break;
      case 81:
        description = "Averses de pluie modérées";
        break;
      case 82:
        description = "Violentes averses de pluie";
        break;
      case 85:
        description = "Légères averses de neige";
        break;
      case 86:
        description = "Fortes averses de neige";
        break;
      case 95:
        description = "Orage";
        break;
      case 96:
        description = "Orage avec grêle légère";
        break;
      case 99:
        description = "Orage avec grêle forte";
        break;
      default:
        description = "";
        break;
    }
    return description;
    },
    // Ajouter la ville recherchée à l'historique de recherche et en première ligne du tableau
    addInLocalStorage(name,department,latitude,longitude,countryCode){
      // Si la ville est déja dans l'historique on la supprime
      if (this.searchHistory.some(history => history.villeName === name) && this.searchHistory.some(history => history.latitude === latitude) && this.searchHistory.some(history => history.longitude === longitude)) {
        let index = this.searchHistory.findIndex(history => history.villeName === name && history.latitude === latitude && history.longitude === longitude);
        this.searchHistory.splice(index, 1);
      }  
      // On ajoute la ville en première ligne du tableau
      this.searchHistory.unshift({
        villeName: name,
        department: department,
        latitude: latitude,
        longitude: longitude,
        countryCode: countryCode
      });
      localStorage.setItem('searchHistory', JSON.stringify(this.searchHistory));
    },
    // La fonction permet de supprimer une ville qui se trouve dans l'historique
    removeCityFromHistory(history) {
      let index = this.searchHistory.indexOf(history);
      this.searchHistory.splice(index, 1);
      localStorage.setItem('searchHistory', JSON.stringify(this.searchHistory));
    },
  }
}
</script>
<style>
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

.v-list-close > .v-list-item__content {
  display: flex;
  width: 100%;
  justify-content: space-between;
}
</style>