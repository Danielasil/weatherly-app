<template>
 <div class="days-tab text-center">
    <div v-if="loading" 
    class="loading">Loading...</div>

    <ul v-else class="p-0">
        <li v-for="day in forecast" :key="day.date" class="li_active">
            <div class="py-3"><img :src="day.iconUrl" /></div>
            <div class="py-3">{{ getDayName(day.date) }}</div>
            <div class="py-3">{{ day.temperature }}&deg;</div>
        </li>
    </ul>
 </div>
</template>

<script>
import axios from 'axios';
import { defineComponent } from 'vue';
import moment from 'moment';

export default defineComponent({
  props: {
    cityname: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      forecast: [],
      loading: true
    };
  },
  async mounted() {
    await this.fetchWeatherData();
  },
  methods: {
    async fetchWeatherData() {
      const apiKey = '8331cdef4f10633c84fd856ce65588b0';
      const city = this.cityname;
      const url = `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`;

      this.loading = true;
      try {
        const response = await axios.get(url);
        const forecastData = response.data.list;

        const filteredData = forecastData
          .map(item => ({
            date: moment(item.dt_txt.split(' ')[0]),
            temperature: Math.round(item.main.temp),
            description: item.weather[0].description,
            iconUrl: `https://openweathermap.org/img/w/${item.weather[0].icon}.png`
          }))
          .reduce((acc, item) => {
            if (!acc.some(day => day.date.isSame(item.date, 'day'))) {
              acc.push(item);
            }
            return acc;
          }, [])
          .slice(1, 5);

        this.forecast = filteredData;
      } catch (error) {
        console.error('Error fetching weather data:', error);
      } finally {
        this.loading = false;
      }
    },
    getDayName(date) {
      return date.format('ddd');
    }
  }

});
</script>


<style>
.days-tab{
    border-radius: 20px;
    margin: 0!important;
    display: flex;
    justify-content: center!important;
    align-items: center!important;
    align-content: center!important;
    padding-top: 15px;
    height: 60%;
     backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(
    to bottom,
    rgba(223, 238, 241, 0) 0%,
    rgba(255, 255, 255, 0.18) 40%,
    rgba(255, 255, 255, 0) 100%
  );
}

.days-tab::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px; 
  border-radius: inherit;

  background: linear-gradient(
    to right,
    rgba(255, 255, 255, 0.533),
    rgba(255, 255, 255, 0.2)
  );

  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
          mask-composite: exclude;

  pointer-events: none;
}
.loading{
    color: white;
}
ul{
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-evenly;
    align-items: center;
    
}
li{
    display: inline-block;
    list-style: none;
    height: 30vh;
    width: 19%;
    font-size: 16px;
   
    display: flex;
    flex-direction: column;
    padding-top: 20px;
}
span{
    display: block;
    margin-bottom: 5px;
    font: 100% sans-serif;
    height: 35px;
}
.li_active{
   background: linear-gradient(
    to bottom,
    rgba(150, 202, 212, 0.528) 0%,
    rgba(152, 152, 152, 0.18) 40%,
    rgba(255, 255, 255, 0) 100%
  );
    color: white;
    border-radius: 10px;
   

  
}
.li_active:hover{
    transform: scale(1.2);
    transition: transform 0.2s ease;
}
.li_active_temp{
    display: inline-block;
    background-color: #222831;
    color: white;
    transition: background-color 0.5s;
    border-radius: 10px;
}
.li_active_temp:hover{
    transform: scale(1.2);
    transition: transform 0.1s ease;
    background: white;
    border-radius: 10px;
    color: #191a1f;
}
</style>
