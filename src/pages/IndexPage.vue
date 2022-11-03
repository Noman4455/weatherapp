<template>
  <q-page class="flex column" :class="bgClass">
    <!-- input field -->
    <div class="col q-pt-md q-px-md">
      <q-input
        class="input"
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        rounded
        standout
        label="Search"
        label-color="white"
        bg-color="blue-5"
      >
        <template v-slot:prepend>
          <q-icon name="place" color="white" />
        </template>
        <template v-slot:append>
          <q-icon
            name="search"
            color="white"
            @click="getWeatherBySearch"
            class="cursor-pointer"
          />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-Bold">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin relative-position">
          <spam>{{ Math.round(weatherData.main.temp) }}</spam>
          <spam
            class="text-h3 text-weight-thin relative-position"
            style="top: -35px"
            >&deg;c</spam
          >
        </div>
      </div>

      <div class="col text-center">
        <q-img
          width="100px"
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col">
        <div class="text-center text-white text-h3">
          Welcome For <br />
          Weather App
        </div>

        <div class="text-center q-pt-xl text-white">
          <q-btn @click="getLocation" class="col" no-caps flat>
            <q-icon left size="2em" name="fa-solid fa-location-crosshairs" />
            <div class="text-h6 q-py-md">Find My Location</div>
          </q-btn>
        </div>
      </div>
    </template>

    <div class="col town"></div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";

export default {
  name: "IndexPage",

  data() {
    return {
      search: ref(""),
      weatherData: null,
      lati: null,
      long: null,
      apiUrl: "http://api.openweathermap.org/data/2.5/weather",
      apiKey: "bf5a686ed6879c1c889e4a90e761c499",
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios
          .get(
            "https://api.freegeoip.app/json/8.8.8.8?apikey=f2482ae0-b643-11ec-9eb2-43b3380b6ed3"
          )
          .then((response) => {
            this.lati = response.data.latitude;
            this.long = response.data.longitude;
            this.getWeatherByCoords();
          });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position: ", position);
          this.lati = position.coords.latitude;
          this.long = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lati}&lon=${this.long}&appid=${this.apiKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to top, #2c3e50, #3498db)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.town
  flex: 0 0 100px
  background: url(../assets/town.png)
  background-size: contain
  background-position: bottom center
</style>

