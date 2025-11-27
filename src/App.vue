<template>
  <div 
  :class="{ 'app-init-data': !showWeather, 
  'app': showWeather }"
  :style="showWeather ? 
  { backgroundImage: appBackground } : 
  { backgroundImage: `url(${bg_default})`}"
>

    <div 
      ref="location" 
      class="header container p-5"
      :class="{ 'header-centered': !showWeather, 'header-top': showWeather }"
    >
      <h1 
  class="mb-5 header-title petit text-appear"
  v-if="!showWeather"
  v-html="animatedTitle"
></h1>

      <div :class="{ 'search-row-init': !showWeather, 'search-row': showWeather }"
      >
        <div 
        class="searchbar">
         <input
  type="text"
  class="input"
  v-model="city"
  placeholder="Enter a city..."
  @keyup.enter="searchWeather"
/>
        </div>

        <button 
          class="btn-search btn btn-primary"
          @click="searchWeather"
        >
          <div class="text-search-btn">Search</div>
          <i class="fa-solid fa-search search-icon"></i>
        </button>
      </div>
    </div>

   <Weather
  ref="weatherComponent"
  :city="searchedCity"
  v-if="true" 
  @weather-updated="updateBackground"
  @weather-error="setDefaultBackground"
/>
<div :class="{ 'movil-nav-bar-invisible': !showWeather, 
  'movil-nav-bar': showWeather }">
 <div  @click="goTo('location')"
       class="button-nav"><i class="fa-solid fa-location-arrow"></i></div>

 <div  @click="goTo('home')"
       class="button-nav"><i class="fa-solid fa-house"></i></div>

 <div @click="goTo('calendar-weather')"
 class="button-nav"><i class="fa-solid fa-calendar"></i></div> 

</div>

  </div>
</template>

<script>
import { defineComponent } from 'vue'
import Weather from './components/weather.vue'
import axios from "axios"

import bg_clear from '@/assets/bg_clear.jpg'
import bg_rain from '@/assets/bg_rain.jpg'
import bg_snow from '@/assets/bg_snow.jpg'
import bg_cloudy from '@/assets/bg_cloudy.jpg'
import bg_night from '@/assets/bg_night.jpg'
import bg_default from '@/assets/bg_default.jpg'
import bg_fewclouds from '@/assets/bg_fewclouds.jpg'

import bg_fog from '@/assets/bg_fog.jpg'
import bg_heavysnow from '@/assets/bg_heavysnow.jpg'
import bg_lightsnow from '@/assets/bg_lightsnow.jpg'

import bg_lightrain from '@/assets/bg_lightrain.jpg'
import bg_thunderstorm from '@/assets/bg_thunderstorm.jpg'
import bg_sleet from '@/assets/bg_sleet.jpg'
import bg_scatteredclouds from '@/assets/bg_scatteredclouds.jpg'



export default defineComponent({
  name: 'App',
  components: { Weather },
  data() {
  return {
    city: '',
    searchedCity: '', 
    showWeather: false,
    appBackground: `url(${bg_default})`,
    title: "Weatherly",
  }
},computed: {
  animatedTitle() {
    return this.title
      .split("")
      .map((letter, i) => 
        `<span style="animation-delay:${i * 0.08}s">${letter}</span>`
      )
      .join("");
  }
},
  methods: {
   async searchWeather() {
  if (!this.city.trim()) return;

   this.searchedCity = this.city;
   this.showWeather = true;
  if (this.$refs.weatherComponent) {
    this.$refs.weatherComponent.setCity(this.searchedCity);
  }
   },
   updateBackground({ description, isDay }) {
  const desc = description.toLowerCase()

  const backgrounds = {
    snow: `url(${bg_snow})`,
    heavysnow: `url(${bg_heavysnow})`,
    lightsnow: `url(${bg_lightsnow})`,
    rain: `url(${bg_rain})`,
    lightrain: `url(${bg_lightrain})`,
    thunderstorm:`url(${bg_thunderstorm})`,
    sleet:`url(${bg_sleet})`,
    fog:`url(${bg_fog})`,
    fewclouds: `url(${bg_fewclouds})`,
    cloudy: `url(${bg_cloudy})`,
    scatteredclouds: `url(${bg_scatteredclouds})`,
    clear: `url(${bg_clear})`,
    night: `url(${bg_night})`,
  }

  if (isDay === "night") {
    this.appBackground = backgrounds.night
    return
  }

  if (desc.includes("heavy") && desc.includes("snow")) {
    this.appBackground = backgrounds.heavysnow
  }
  else if (desc.includes("light") && desc.includes("snow")) {
    this.appBackground = backgrounds.lightsnow
  }
  else if (desc.includes("snow")) {
    this.appBackground = backgrounds.snow
  }
  else if (desc.includes("scattered clouds")) {
    this.appBackground = backgrounds.scatteredclouds
  }
  else if (desc.includes("few clouds")) {
    this.appBackground = backgrounds.fewclouds
  }
  else if (desc.includes("cloud")) {
    this.appBackground = backgrounds.cloudy
  }
  else if (desc.includes("fog") || desc.includes("mist") || desc.includes("haze")) {
    this.appBackground = backgrounds.fog
  }
  else if (desc.includes("light rain")) {
    this.appBackground = backgrounds.lightrain
  }
  else if (desc.includes("rain")) {
    this.appBackground = backgrounds.rain
  }
  else if (desc.includes("thunder")) {
    this.appBackground = backgrounds.thunderstorm
  }
  else if (desc.includes("sleet")) {
    this.appBackground = backgrounds.sleet
  }
  else if (desc.includes("clear")) {
    this.appBackground = backgrounds.clear
  }
  else {
    this.appBackground = backgrounds.clear
  }
   },
   setDefaultBackground() {
   this.appBackground = `url(${bg_default})`;
   },
   goTo(section) {
    if (section === 'location') {
      const el = this.$refs[section]
      if (el) {
        el.scrollIntoView({ behavior: 'smooth', block: 'center' })
      }
    } else if (this.$refs.weatherComponent) {
      this.$refs.weatherComponent.scrollToSection(section)
    }
   },
   getUserLocation() {
  if (!navigator.geolocation) {
    console.warn("Geolocation not supported");
    return;
  }

  navigator.geolocation.getCurrentPosition(
    async (pos) => {
      const lat = pos.coords.latitude;
      const lon = pos.coords.longitude;

      try {
        const response = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=8331cdef4f10633c84fd856ce65588b0&units=metric`
        );

        const cityName = response.data.name;

        this.city = cityName;
        this.searchedCity = cityName;
        this.showWeather = true;

        if (this.$refs.weatherComponent) {
          this.$refs.weatherComponent.setCity(cityName);
        }

      } catch (error) {
        console.error("Error getting city from coordinates:", error);
      }
    },

    (error) => {
      console.warn("User denied geolocation:", error);
    }
  );
},
   },
  mounted() {
  const imgs = [bg_clear, bg_rain, bg_snow, bg_cloudy, bg_night, bg_default];
  imgs.forEach(src => {
    const img = new Image();
    img.src = src;
  });
  this.getUserLocation();
},

})
</script>

<style>

.button-nav{
  color: rgb(255, 255, 255)!important;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 2.5em;
  cursor: pointer;
}

.button-nav:hover{
  opacity: 0.5;
  transition: opacity 0.3s ease;
}

.movil-nav-bar-invisible{
  display: none;
}
.movil-nav-bar{
  position:fixed;
  bottom: 0;
  height: 8%;
  width: 100vw;
  left: 0;
  z-index: 10000;
  display: flex;
  justify-content: space-around;
  align-items: center;
   backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(
    to bottom,
    rgba(213, 232, 243, 0.247) 0%,
    rgba(150, 150, 150, 0.362) 40%,
    rgba(255, 255, 255, 0) 100%
  );
}
.text-appear {
  display: inline-block;
  font-size: inherit;
  font-family: "Montserrat", sans-serif !important;
}

.text-appear span {
  font-family: "Montserrat", sans-serif !important;
  cursor: default;
  opacity: 0;
  display: inline-block;
  transform: translateY(10px);
  animation: letterAppear 0.45s ease forwards;
}

@keyframes letterAppear {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
body {
  font-family: "Montserrat", sans-serif !important;
    height: 100%;
}

.petit {
  font-family: "Petit Formal Script", cursive;
}

.search-row{
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 10px;
  gap: 15px;
}

.search-row-init{
   gap: 15px;
  display: flex;
  flex-direction: row;
   align-items: center;
  justify-content: center;
}

.app-init-data{
  height: 100vh;
  padding: 30px;
  background-image: url("@/assets/bg_default.jpg")  !important;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat ;
  background-blend-mode: overlay ;
  
}

.app{
  position: relative;
  min-height: 100vh; 
  padding: 10px;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat ;

  background-color: #484f69;
  display: flex;
  justify-content: center;
  align-content: center;
  flex-direction: column;
}

.input::placeholder {
  color: #fefefe !important;
  opacity: 0.6 !important;
  font-size: 14px;
  font-weight: 500 !important;
  letter-spacing: 0.5px;
}

input:focus {
  outline: 1px solid white !important;
  border: none;
}

input {
  padding-top: 8px;
  padding-left: 15px;
  border-radius: 8px;
  padding-bottom: 8px;
  width: 100%!important;
  border: none !important;
  color: #fefefe !important;
  background-color: rgba(140, 161, 174, 0.3) !important;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}
.header {
  border-radius: 20px;
  color: #ffffff;
  text-align: center;
  transition: all 0.6s ease;
  margin-bottom: 10px;
}
.header-centered {
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 50vh; 
  margin-top: 150px;
}

button{
  i{
    
    display: flex;
    justify-content: center;
    align-content: center!important;
  
    font-size: 15px;
  }
}
.header-top {
 
  height: auto;
  margin-top: 2rem;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: linear-gradient(
    to bottom,
    rgba(213, 232, 243, 0.247) 0%,
    rgba(150, 150, 150, 0.1) 40%,
    rgba(255, 255, 255, 0) 100%
  );
  box-sizing: border-box;
}
.header-top::before {
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
  -webkit-mask-composite: xor;mask-composite: exclude;
  pointer-events: none;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.6s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.header-title {
  font-weight: 800 !important;
  font-size: 4.5em;
}

.btn-search {
  background-image: linear-gradient(to right,
   #698899, #5d93ba);
   background: linear-gradient(90deg, #698899,#5d93ba, #747d9f);
  border: none !important;
  display: flex!important;
  flex-direction: row!important;
  justify-content: center;
  align-content: center!important;
  background-size: 300% 300%;
  transition: background-position 0.5s ease;
}
.btn-search:hover {
  animation: moveGradient 3s ease infinite;
}

.search-icon {
  margin-left: 3px;
  
}
.text-search-btn{
  display: flex;
}

@keyframes moveGradient {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

@media (min-width: 0px) and (max-width: 380px) {
.header-title{
  font-size: 10px!important;
white-space: nowrap;
display: inline-block;
margin-right: 4%;
}
.text-search-btn{
display: none;
}
.btn-search{
  padding-top: 10px!important;
  padding-bottom: 10px!important;
}
}

@media (min-width: 0px) and (max-width: 450px) {
  .button-nav{
    font-size: 30px;
  }
 .header {
  display: flex;
}

.search-row-init{
flex-direction: column;
gap: 10px;
min-width: 150px;
}
.header-title {
  font-size: 40px !important;
  font-weight: 800 !important;
}
input {
height: 50px;
}
.input::placeholder {
  color: #fefefe !important;
  opacity: 0.6 !important;
  font-size: 18px;
  font-weight: 500 !important;
  letter-spacing: 0.5px;
}
}

@media (min-width: 700px) and (max-width: 2000px) {
.header-title {
 font-size: 6em;
  font-weight: 800 !important;
}
input {

width: 350px !important;
}
.input::placeholder {
  color: #fefefe !important;
  opacity: 0.6 !important;
  font-size: 15px;
  font-weight: 500 !important;
  letter-spacing: 0.5px;
}
}

@media screen and (max-width: 768px) {
  .movil-nav-bar {
    display: flex;
  }
}

@media screen and (min-width: 769px) {
  .movil-nav-bar {
    display: none;
  }
}
</style>
