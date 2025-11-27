<template>
  
   <div v-if="error" 
   class="error-message container">
    <i class="fa-solid fa-triangle-exclamation"></i>
    <div>{{ error }}</div>
  </div>

  <div v-else-if="loaded" 
  class="info-cards-container container p-0 mb-5">
    <div class="card-container-info">
        
        <div
        ref="home"  
        class="card-main-info">
           <div class="first-row-info">
                <h2 class="place">{{name}}</h2>
                <h3 class="mb-1 day">{{ currentDay }}</h3>
                <p class="hour mb-0">{{time}}</p>
              </div>
            
                <div class="temp">
                  <h2 class="descrition-of-sky">
                        {{description}}
                  </h2>
                  <h1 class="weather-temp">{{temperature}}&deg;</h1>
                <div class="text-small-data">
                 {{ isDay }} 
                 <i :class="[dayNightIconClasses, isDay === 'day' ? 
                 'icon-rotate' : 
                 'icon-glow']"></i>
                </div>
              
            </div>

            <div class="last-row-info">
             <div v-if="currentEvent">
              
             <div class="event-icon">
             <span v-if="currentEvent === 'sunrise'">
             <i class="fa-solid fa-cloud-sun"></i>
            </span>
             <span v-else><i class="fa-solid fa-cloud-moon"></i></span>
             </div>
              <div class="sunrise-status">{{ currentEvent === 'sunrise' ? 'Sunrise' : 'Sunset' }}</div>
            <div class="sunrise-time">at {{ nextEventTime }}</div>
             </div>

             <div v-else>
               
              <div class="event-icon">
               <span v-if="isDay === 'day'">
                <i class="fa-solid fa-cloud-sun"></i></span>
               <span v-else><i class="fa-solid fa-cloud-moon"></i></span>
               </div>

              <div class="sunrise-status">
              <small>next</small>
                {{ isDay === 'day' ? 'Sunset' : 'Sunrise' }}</div>
             <div class="sunrise-time">at {{ nextEventTime }}</div>
             </div>
              </div>
        </div>
  
        <div class="card-2 w-100">
        <div class="table-container">
          <table class="table-details">
            <tbody>
                <tr>
                    <th><i class="fa-solid fa-water">

                    </i>
                    <div class="table-details-text">Sea level</div></th>
                    <td v-if="sea_level > 0">
                        {{sea_level}}</td>
                    
                    <td v-else></td>
                </tr>

                <tr>
                    <th><i class="fa-solid fa-droplet"></i>
                      <div class="table-details-text">Humidity</div></th>
                    <td>{{humidity}}</td>
                </tr>

                <tr>
                    <th>
                    <i class="fa-solid fa-wind"></i>
                    <div class="table-details-text">Wind</div></th>
                    <td>{{wind}}</td>
                </tr>
            </tbody>
        </table>
        <div class="more-info-card">
          
          <div class="small-info">
            <i class="fa-solid fa-temperature-half"></i>
          <th class="small-info-text">Feels like:</th> 
          <td>{{ feels_like }}°C</td>
        </div>
         <div class="small-info">
          <i class="fa-solid fa-cloud"></i>
          <th class="small-info-text">Cloudiness:</th> 
          <td>{{ clouds }}%</td>
        </div>
        <div class="small-info">
          <i class="fa-solid fa-water"></i>
         
          <th class="small-info-text">Dew point:</th>      
         <td>{{ dewPoint }}°C</td>
        </div>

      </div>
        </div>
 <DaysWeather v-if="loaded" :cityname="city"></DaysWeather>
        

        
        </div>

    </div>
  
     <div ref="calendar-weather" 
     class="card-calendar-weather">
      <div class="hourly-elements">
       <div class="hourly-title">Hourly forecast</div>
       
       <div class="houry-mini-data">
    <div><i class="fa-solid fa-location-dot"></i>{{name}}</div>
    <div><i class="fa-regular fa-clock"></i>{{time}}</div>
    <div><i class="fa-regular fa-calendar-days"></i>{{date}}</div>
       </div>
     
    </div>
        <div v-if="hourlyForecast.length" class="hourly-forecast">

  <div class="hour-card" 
       v-for="(hour, i) in hourlyForecast.slice(0, 12)" 
       :key="i">
    <p>{{ hour.hour }}</p>
    <img :src="hour.icon" :alt="hour.description">
    <p class="hour-temp">{{ hour.temp }}°C</p>
    <p class="hour-description">{{ hour.description }}</p>
  </div>

</div>

<div v-if="coldestHour && warmestHour" 
class="extremes-container">
  <div class="coldest">
    <i class="fa-regular fa-snowflake"></i>
    <div class="extremes-text">Coldest hour</div> 
    {{ coldestHour.hour }}
    <i class="fa-solid fa-circle dot-icon"></i> 
    {{ coldestHour.temp }}°C
  </div>

  <div class="warmest">
    <i class="fa-solid fa-mug-hot"></i>
    <div class="extremes-text">Warmest hour</div>
     {{ warmestHour.hour }} 
    <i class="fa-solid fa-circle dot-icon"></i>
    {{ warmestHour.temp }}°C
  </div>
</div>
        </div>

  </div>

</template>

<script>
import { defineComponent } from 'vue'
import axios from 'axios'

import DaysWeather from './DaysWeather.vue'

export default defineComponent({
  name: 'myWeather',
  components: { DaysWeather },

  props: {
    city: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      loaded: false, 
      temperature: null,
      description: null,
      iconUrl: null,
      date: null,
      time: null,
      country: null, 
      name: null,
      sea_level: null,
      wind: null,
      humidity: null,
      feels_like: null,
      clouds: null,
      error: null, 
      isDay: null,
      currentEvent: null, 
      nextEventTime: null,
      hourlyForecast: [],
      monthNames: [
        "Jan", "Feb", "Mar",
        "Apr", "May", "Jun", "Jul",
        "Aug", "Sep", "Oct",
        "Nov", "Dec"
      ],
       coldestHour: null,
      warmestHour: null,
      dayNames: [
      "Sunday", "Monday", "Tuesday",
      "Wednesday", "Thursday", "Friday", "Saturday"],
      currentDay: null,
    }
  },
   watch: {
    city: {
      immediate: true,     
      handler(newCity) {
        if (newCity) this.fetchWeather(newCity);
      }
    }
  },
  computed: {
  dayNightIconClasses() {
    return this.isDay === 'day' ? 'fa-solid fa-sun' : 'fa-solid fa-moon';
  }
  },
  methods: {
  changeLocation() {
    window.location.reload();
  },
  setCity(newCity) {
  this.fetchWeather(newCity)
  },
  toAMPM(timeString) {
    let [hours, minutes] = timeString.split(":").map(Number);

    const ampm = hours >= 12 ? "PM" : "AM";
    hours = hours % 12 || 12;

    return `${hours}:${minutes.toString().padStart(2, "0")} ${ampm}`;
  },
  
  scrollToSection(sectionName) {
    const section = this.$refs[sectionName]
    if (section) {
      section.scrollIntoView({ behavior: "smooth", block: "start" })
    } else {
      console.warn(`La sección '${sectionName}' no existe en Weather.vue`)
    }
  },

  async fetchWeather(city) {
     this.error = null;
    try {
      const response = await axios.get(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=8331cdef4f10633c84fd856ce65588b0`
      );

      const weatherData = response.data;

      this.temperature = Math.round(weatherData.main.temp);
      this.clouds = weatherData.clouds.all;
      this.description = weatherData.weather[0].description;
      this.name = weatherData.name;
      this.wind = weatherData.wind.speed;
      this.sea_level = weatherData.main.sea_level;
      this.country = weatherData.sys.country;
      this.humidity = weatherData.main.humidity;
      this.feels_like = Math.round(weatherData.main.feels_like);
      this.iconUrl = `https://openweathermap.org/img/w/${weatherData.weather[0].icon}.png`;
      
      const d = new Date();
      this.currentDay = this.dayNames[d.getDay()];
      this.date = d.getDate() + ' ' + this.monthNames[d.getMonth()] + ' ' + d.getFullYear();
      const hourMinute = d.getHours().toString().padStart(2,'0') + ':' + d.getMinutes().toString().padStart(2,'0');
      this.time = this.toAMPM(hourMinute);

      const localTimestamp = Math.floor(Date.now() / 1000) + weatherData.timezone;
      const sunrise = weatherData.sys.sunrise + weatherData.timezone;
      const sunset = weatherData.sys.sunset + weatherData.timezone;

      this.isDay = localTimestamp > sunrise && localTimestamp < sunset ? "day" : "night";
     
      const forecastResp = await axios.get(
      `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=8331cdef4f10633c84fd856ce65588b0&units=metric`
      );

      this.hourlyForecast = forecastResp.data.list.map(item => {
      const rawHour = item.dt_txt.slice(11, 16); 

  return {
    dt_txt: item.dt_txt,
    hour: this.toAMPM(rawHour),
    temp: Math.round(item.main.temp),
    description: item.weather[0].description,
    icon: `https://openweathermap.org/img/wn/${item.weather[0].icon}.png`
  };
});

  if (this.hourlyForecast.length) {
  
  this.coldestHour = this.hourlyForecast.reduce((min, h) =>
    h.temp < min.temp ? h : min
  );

  this.warmestHour = this.hourlyForecast.reduce((max, h) =>
    h.temp > max.temp ? h : max
  );
}
      const formatHour = (ts) => {
        const date = new Date(ts * 1000);
        return date.getHours().toString().padStart(2, "0") + ":" +
              date.getMinutes().toString().padStart(2, "0");
      };

      this.sunriseTime = this.toAMPM(formatHour(sunrise));
      this.sunsetTime = this.toAMPM(formatHour(sunset));
 
      const oneHour = 3600;

      if (localTimestamp >= sunrise - oneHour && localTimestamp <= sunrise + oneHour) {
        this.currentEvent = "sunrise";
        this.nextEventTime = this.sunriseTime;
      }
      else if (localTimestamp >= sunset - oneHour && localTimestamp <= sunset + oneHour) {
        this.currentEvent = "sunset";
        this.nextEventTime = this.sunsetTime;
      }
      else {
        this.currentEvent = null;
        this.nextEventTime = localTimestamp < sunrise ? this.sunriseTime : this.sunsetTime;
      }

      this.$emit('weather-updated', {
        description: this.description,
        isDay: this.isDay
      });

      this.loaded = true; 
      this.$emit('weather-updated', { description: this.description, isDay: this.isDay });

    } catch (error) {
      this.error = "City not found. Please write the name correctly.";
      console.error("Error in the data:", error.response?.data?.message || error.message);
      this.$emit("weather-error");
    }
  }
},


expose: ['fetchWeather', 'setCity', 'scrollToSection']
})
</script>

<style>
.hour-temp{
  font-size: 20px!important;
  
}
.hour-description{
  font-size: 17px;
  font-weight: 200;
}
.extremes-container{
  
  margin-left: 20px;
  margin-right: 20px;
  display: flex;
  flex-direction: row;
  font-size: 18px;
  padding-left: 10px;
  padding-right: 10px;
  margin-top: 30px;
  align-items: center;
  justify-content: space-between;

}
.dot-icon{
  font-size: 5px;
}
.coldest, .warmest{
  display: flex;
  align-items: center;
  gap: 15px;
  font-weight: 200;
}
.hourly-forecast {
  display: flex;
  height: auto;
  overflow-x: auto;
  overflow-y: hidden;
 
  padding: 10px;
  align-items: center;
  gap: 30px;
  background-color: rgba(0, 0, 0, 0.087);
  border-radius: 10px;
  margin-top: 5px;
  
  margin-left: 20px;
  margin-right: 20px;

  padding-left: 30px;
  padding-top: 30px;
  padding-bottom: 30px;
  padding-right: 30px;
}
.hourly-forecast::-webkit-scrollbar {
  height: 8px; 
}
.houry-mini-data{
  
  display: flex;
  align-items: center;
  padding-left: 20px;
  padding-right:  10px;
  gap:30px;
  font-size: 20px;
  
  i{
  font-size: 18px;
  padding-right:  10px;
  }
}

.hourly-title{
 font-size: 25px;
}
.hourly-elements{
  display: flex;
  margin-left: 40px;
  margin-right: 30px;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
  font-weight: 200;
}

.hourly-forecast::-webkit-scrollbar-thumb {

  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
}

.hour-card {
  min-width: 150px; 
  border-radius: 10px;
  padding: 10px;
  padding-top: 50px;
  padding-bottom: 50px;  text-align: center;
  flex-shrink: 0; 
  background: linear-gradient(
    to bottom,
    rgba(150, 202, 212, 0.528) 0%,
    rgba(152, 152, 152, 0.18) 40%,
    rgba(255, 255, 255, 0) 100%
  );
  display: flex;
  flex-direction: column;
  gap: 20px;
  align-items: center;

}
.hour-card:hover{
    transform: scale(1.1);
    transition: transform 0.2s ease;
}
.hour-card img {
  width: 50px;
  height: 50px;
  margin: 5px 0;
}



.weather-temp{
    margin:0;
    font-size: 120px;
    font-family: "Montserrat", sans-serif!important;
    font-weight: 200!important;
    margin-bottom: 25px;
    margin-top: 15px;
    padding-left: 15px;
   
}
.descrition-of-sky{
   
    display: flex;
    align-items: center;
    justify-content: center;
    align-content: center;
    text-align: center;
    font-size: 30px;
 
 
}
.temp-icon{
height: 60px!important;
}

.first-row-info{
  width: 100%;
  display: flex;
  text-align: left;
  align-items: start;
  flex-direction: column;
  padding-left: 5%;
 
}

.last-row-info{
  width: 100%;
  display: flex;

  align-items: end;
  justify-content: end;
  flex-direction: column;
  padding-right: 5%;
}

.card-calendar-weather::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px;
  border-radius: inherit;
  background: linear-gradient(to right,
    rgba(255, 255, 255, 0.9),
    rgba(255, 255, 255, 0.2));
  -webkit-mask:
  linear-gradient(#fff 0 0) content-box,
  linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}
.card-calendar-weather{
  cursor: default;

  height: auto;
  border-radius: 20px;
  margin-top: 4%;
  overflow: hidden;  
  color: white !important;

  padding-top: 2%;
  padding-bottom: 2%;
  box-sizing: border-box !important;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(to bottom,
    rgba(223, 238, 241, 0) 0%,
    rgba(255, 255, 255, 0.18) 40%,
    rgba(255, 255, 255, 0) 100%);
}

.card-main-info {
 cursor: default;
  width: 100%;
  height: 67vh;
  border-radius: 20px;
  position: relative;
  overflow: hidden;  
  color: white !important;
  display: flex;
  padding-top: 2%;
  padding-bottom: 2%;
  box-sizing: border-box !important;
  justify-content: space-between;
  flex-direction: column;
  align-items: center;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(to bottom,
    rgba(223, 238, 241, 0) 0%,
    rgba(255, 255, 255, 0.18) 40%,
    rgba(255, 255, 255, 0) 100%);
}
.card-main-info::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px;
  border-radius: inherit;
  background: linear-gradient(to right,
    rgba(255, 255, 255, 0.9),
    rgba(255, 255, 255, 0.2));
  
  -webkit-mask:
  linear-gradient(#fff 0 0) content-box,
  linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}
.card-container-info{
  gap: 10px;
  display: flex;
margin-top: 10px;
}
.card-main-info:hover{ 
  transform: scale(1.03); 
  transition: transform 0.5s ease; 
  z-index: 1;
}
.temp{
  color: rgb(255, 255, 255);
  display: flex;
  align-content: center;
  justify-content: space-between;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  
  height: 200px;
}
.day{

  display: flex;
  align-content: center;
  justify-content: center;
  align-items: center;
  font-size: 25px;
  
}
.date{
    
    display: flex;
    align-content: center;
    justify-content: center;
    align-items: center;
}
.hour{
    
    display: flex;
    align-content: center;
    justify-content: center;
    align-items: center;
    font-size: 18px;
    height: 20px;
    margin-left: 1px;
    font-weight: 200;
}
.place{
    display: flex;
    align-content: center;
    justify-content: center;
    align-items: end;
    gap: 10px;
    font-size: 60px;
}
.sunrise-status{
    display: flex;
    align-content: center;
    justify-content: center;
    align-items: center;
    font-size: 25px;
    flex-direction: column;

    small{
      height: 22px;
    }
}
.sunrise-time{
     display: flex;
    align-content: center;
    justify-content: end;
    align-items: end;
    font-size: 18px;
    font-weight: 200;

   
}
.main-div:hover{
    transform: scale(1.1);
    transition: transform 0.5s ease;
    z-index: 1;
}
.card-2{
    width: 100%!important;
    border-radius: 20px !important;
    border: none !important;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
    gap: 10px;
}
.table-details{
    margin-left: 0;
    font-family: "Montserrat", sans-serif!important;
    font-weight: 300!important;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;

}
.small-info{
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: space-evenly;
 }

.more-info-card{
   border-radius: 20px !important;
   width: 50%;
   height: 100%;
   display: flex;
   justify-content: center;
   align-items: center;
   flex-direction: column;
   color: white;
   cursor: default;
   backdrop-filter: blur(12px);gap: 25px;
   -webkit-backdrop-filter: blur(12px);
   
   background: linear-gradient(
    to bottom,
    rgba(49, 81, 88, 0.859) 0%,
    rgba(99, 117, 167, 0.19) 40%,
    rgba(255, 255, 255, 0) 100%,
  );
}

.more-info-card::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px;
  border-radius: inherit;
  background: linear-gradient(to right,
    rgba(255, 255, 255, 0.533),
    rgba(255, 255, 255, 0.2));
  -webkit-mask:
  linear-gradient(#fff 0 0) content-box,
  linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}
.more-info-card:hover{
transform: scale(1.03);
 transition: transform 0.5s ease; 
  z-index: 1;
}

.table-details:hover{ 
  transform: scale(1.03);
  transition: transform 0.5s ease; 
  z-index: 1; }
  
.h1-left{
    position: absolute;
    bottom: 25px;
    left: 16px;
    font-size: 3vw;
}
.h3-left{
    position: absolute;
    left: 16px;
    font-size: 2vw;
    line-height: 0.5;
}
.h3-left small{
    font-size: 1rem;
}

table{
  width: 50%;
  padding-left: 6%;
  padding-right: 6%;
  height: 140px;
  border-radius: 20px;
  text-align: left;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(
    to bottom,
    rgba(254, 254, 254, 0.111) 0%,
    rgba(150, 150, 150, 0) 40%,
    rgba(255, 255, 255, 0) 100%
  );
}
.table-details::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px;
  border-radius: inherit;
  background: linear-gradient(to right,
    rgba(255, 255, 255, 0.533),
    rgba(255, 255, 255, 0.2));
  -webkit-mask:
  linear-gradient(#fff 0 0) content-box,
  linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}

.table-container{
  display: flex;
  flex-direction: row;
  gap: 10px;
  height: 200px;
}
tr{
  display: flex;
  justify-content: center;
  gap: 5px;
  margin-bottom: 16px;
  margin-top: 16px;
}

th{
    font-weight: 500 !important;
    display: flex;
    flex-direction: row;
    width: 100%;
    gap: 10px;
    align-items: center;
    justify-content: start;
    margin-right: 5px;
 
}

th, td{
    font-size: 18px;
    color: white;
}
td{
  display: flex;
  text-align: right;
}
.table-details-text{
  cursor: default;
}
.change-btn{
    background-image:linear-gradient(to right, #667eea , #764ba2);
    border: none !important;
    color: white !important;
}
.change-btn:hover{
    transform: scale(0.9);
    transition: transform 0.3s ease;
    
}
.icon-location{
    margin-right: 0px;
    font-size: 24px;
}
.error-message {
  width: 100%;
  height: 60vh;

  color: white;
  display: flex!important;
  flex-direction: column;
  justify-content: center!important;
  align-content: center!important;
  align-items: center;    
  text-align: center;
  backdrop-filter: blur(12px);
  border-radius: 20px;
  padding:20px 50px;
  font-size: 18px;
  gap: 15px;
  background: rgba(97, 97, 97, 0.355);
}

.error-message::before {
  content: "";
  position: absolute;
  inset: 0;
  padding: 1px;
  border-radius: inherit;
  background: linear-gradient(to right,
    rgba(255, 255, 255, 0.9),
    rgba(255, 255, 255, 0.2));
  
  -webkit-mask:
  linear-gradient(#fff 0 0) content-box,
  linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}
.error-message i {
  font-size: 40px;
  margin-bottom: 10px;
}
.text-small-data{
display: flex;
align-items: center;
justify-content: center;
font-size: 20px;
gap: 8px;
font-weight: 200;
}
.text-small-dot{
  font-size: 5px;
}
.fa-sun {
  color: #f3f2ed;
}

.fa-moon {
  color: #ccd5e8;
}
.icon-glow {
  animation: glow 2.5s infinite;
}
.event-icon {
  
  display: flex;
  justify-content: center; 
  align-items: center; 
  
  span{
  display: flex;
  justify-content: center; 
  align-items: center;
  font-size: 22px;
  }
}


@keyframes glow {
  0% { opacity: 0.5; }
  50% { opacity: 1; }
  100% { opacity: 0.5; }
}
.icon-rotate {
  animation: rotate 6s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@media screen and (max-width: 768px) {
  .hourly-elements {
    display: flex;
    flex-direction: column;
    margin-bottom: 5px;
    align-items: start;
   
}
  .card-calendar-weather {
    margin-bottom: 10%;
  }
  .houry-mini-data {
    display: flex;
    align-items: start;
    padding-left: 0px;
    padding-right: 0px;
    font-size: 15px;

    i{
      font-size: 15px;
    }
}
.extremes-container{
  
  display: flex;
  flex-direction:row;
  margin-top: 10px;
   align-items: start;
   font-size: 15px;
  

}
.extremes-text{
  display: none;
}
}
@media (min-width: 0px) and (max-width: 450px) {
 .error-message{
  flex-direction: column;

 }
 
  .table-container {
    flex-direction: column;
    height: auto;
  }

  .table-details,
  .more-info-card {
    width: 100% !important;
  }
  .more-info-card{
    padding-top: 15px;
    padding-bottom: 15px;
  }
  .days-tab{
    padding-top: 15px;
    padding-bottom: 15px;
  }
  .extremes-container{
  
  display: flex;
  flex-direction:column;
  margin-top: 20px;
  gap: 8px;
  margin-bottom: 10px;
   align-items: center;
  

}
 .houry-mini-data {
    display: none;
}

.hourly-elements {
    display: flex;
    flex-direction: column;
    margin-bottom: 10px;
    margin-top: 10px;
    align-items: center;
   
}
.weather-temp{
margin-top: 0px;
margin-bottom: 0px;
}
.descrition-of-sky{font-size: 20px;}
}
@media (min-width: 100px) and (max-width: 970px) {
.card-container-info{
  gap: 10px;
  display: flex;
  flex-direction: column;
}
.days-tab{
    padding-top: 15px;
    padding-bottom: 15px;
  }
}
@media (min-width: 970px) and (max-width: 1000px) {
.small-info-text{
  font-size: 14px;
  display: none;
}
.small-info{
  font-size: 14px;
}
.table-details-text{
 display: none;
}
}
</style>
