<template>
  <h2>Yo</h2>
</template>

<script>
import axios from 'axios';


export default {
  name: 'HomeView',
  data () {
    return {
      labels: { 0: 'LU', 1: 'MA', 2: 'MER', 3: 'JE', 4: 'VE', 5: 'SA', 6: 'DI' },
      expand: false,
      time: 0,
      forecast: [
        { day: 'Mardi', icon: 'mdi-white-balance-sunny', temp: '24\xB0/12\xB0' },
        { day: 'Mercredi', icon: 'mdi-white-balance-sunny', temp: '22\xB0/14\xB0' },
        { day: 'Jeudi', icon: 'mdi-cloud', temp: '25\xB0/15\xB0' },
      ],
      villeName: "",
      villeTemperature: 0,
      villeVent: 0,
      villeHumidite: 52,
    }
  },
  mounted (){
    this.getPosition();
  },
  methods:
  {
    getPosition(){
      navigator.geolocation.getCurrentPosition((position) => {
        this.getCityName(position.coords.latitude, position.coords.longitude)
        this.findMeteo(position.coords.latitude, position.coords.longitude);
      });
    },
    findMeteo(latitude, longitude)
    {
      axios
        .get("https://api.open-meteo.com/v1/forecast?latitude="+latitude+"&longitude="+longitude+"&current_weather=true&hourly=relativehumidity_2m")
        .then((response) => 
        {
          this.villeTemperature = response.data.current_weather.temperature;
          this.villeVent =  response.data.current_weather.windspeed;

        });
    },
    getCityName(lat, lng) {
      let url = "https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat="+lat+"&lon="+lng;
      axios.get(url)
      .then((response) => 
      {
        this.villeName = response.data.address.city;
      })
    }
  }
}
</script>