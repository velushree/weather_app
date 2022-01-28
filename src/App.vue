<template>
  <div
    v-cloak
    id="app"
    :class="{
      warm: this.warm,
      cold: this.cold,
      hot: this.hot,
      thunder: this.thunderstorm,
      rain: this.rain,
      invalid: this.invalid,
    }"
  >
    <main>
      <div id="i">
        <div class="search-box" v-cloak>
          <input
            id="autocomplete"
            type="text"
            class="search-bar"
            placeholder="Enter City"
            v-model="city"
            @keypress.enter="getweather()"
            @input="autocomplete()"
          />
          <br />

          <div class="statename" v-for="data in this.temp_array" :key="data.id">
            <ul class="list-group">
              <br />
              <li
                v-if="city"
                class="list-group-item"
                aria-current="true"
                @click="setcity(data)"
              >
                {{ data.name }}, <span>({{ data.country }})</span><br />

                <small>(lat:{{ data.lat }}),(lng:{{ data.lng }}) </small>
              </li>
            </ul>
          </div>
          <button
            class="button"
            v-if="searchbarempty"
            @click="currentcoordinates()"
            in
            line="true"
          >
            <fa icon="map-marker-alt" />
            <h3></h3>
          </button>
        </div>
      </div>
      <div>
        <div class="weather-wrap" v-if="weatherdata" v-cloak>
          <div class="location-box">
            <div class="location">
              {{ weatherdata.name }} ({{ weatherdata.sys.country }})
            </div>
            <div class="date">{{ datebuilder() }}</div>
          </div>
          <div class="weather-box" v-cloak>
            <div class="temp">{{ converttemp(weatherdata.main.temp) }}°c</div>
            <div class="weather">
              {{ checkweather(weatherdata.weather[0]) }},{{
                weatherdata.main.humidity
              }}
            </div>
          </div>
        </div>
        <div v-else-if="searchbarempty" class="alternate" v-cloak>
          Search Bar is Empty ... !
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import cities from "./data/cities.json";
export default {
  data() {
    return {
      searchbarempty: "empty",
      cities_data: cities,
      temp_array: "",
      yourlocation: "",
      latitude: "0",
      longitude: "0",
      thunderstorm: false,
      hot: false,
      rain: false,
      cold: false,
      warm: false,
      city: "",
      weatherdata: "",
      invalid: false,
      error: "",
    };
  },
  methods: {
    autorefresh() {
      setTimeout(() => {
        window.location.reload();
      }, 1000);
    },
    setcity(data) {
      document.getElementById("autocomplete").value = data.name;
      this.city = data.name;
      this.getweather();
    },
    autocomplete() {
      let search = document.getElementById("autocomplete").value;

      let alpha = require("./data/cities.json");
      let matches = alpha.filter(function (i) {
        let regex = new RegExp(`^${search}`, "gi");
        return i.name.match(regex);
      });
      this.temp_array = this.city.length > 0 ? matches : [];
      this.searchbarempty = this.city.length === 0 ? "empty" : "";
      if (this.city.length === 0) {
        this.weatherdata = "";
      }
      if (this.city.length > 0) {
        this.weatherdata = "";
      }
    },
    checkweather(data) {
      this.thunderstorm = data.main == "Thunderstorm" ? "thunder" : "";
      this.rain = data.main == "Rain" ? "rain" : "";
      return data.description;
    },
    converttemp(temp) {
      let degree = temp - 273;
      let x = Math.round(degree);
      this.warm = x >= 30 ? "warm" : "";
      this.hot = x < 30 && x >= 27 ? "hot" : "";
      this.cold = x <= 0 ? "cold" : "";
      return x;
    },
    checkfetch(response) {
      if (response.status == "404") {
        this.invalid = true;
        this.error = "error";

        setTimeout(() => {
          alert("Enter a valid city name");
        }, 1500);
      } else {
        this.invalid = false;
        this.error = "";
      }
      return response;
    },
    async currentcoordinates() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          var lat = position.coords.latitude;
          var lon = position.coords.longitude;
          let url = `https://api.openweathermap.org/geo/1.0/reverse?lat=${lat}&lon=${lon}&limit=5&appid=8e5d5c219dd7b7df4db4f188d6f99cb0`;
          fetch(url).then((response) => {
            response.json().then((data) => {
              this.yourlocation = data[0].name;
              this.city = this.yourlocation;
              this.getweather();
            });
          });
        });
      } else {
        console.log("Oops! Your Browser does not support Geolocation");
      }
    },
    getweather() {
      let z = `https://api.openweathermap.org/data/2.5/weather?appid=8e5d5c219dd7b7df4db4f188d6f99cb0&q=${this.city}`;
      fetch(z)
        .then((res) => {
          if (res.ok) {
            this.checkfetch(res);
            return res.json();
          } else {
            alert("invalid city name");
            this.autorefresh();
          }
        })
        .then((data) => {
          this.weatherdata = data;
          this.temp_array = [];
        })
        .catch((error) => {
          console.log(error);
        });
    },
    datebuilder() {
      let d = new Date();
      let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "august",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day}-${date} ${month} (${year})`;
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: "montserrat", sans-serif;
}
#i {
  display: inline;
}
#app {
  background-image: url("./assets/cold1-bg.jpg");
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
#app.warm {
  background-image: url("./assets/warm1-bg.jpg");
}
#app.cold {
  background-image: url("./assets/ice1-bg.jpg");
}
#app.rain {
  background-image: url("./assets/rainy1-bg.jpg");
}
#app.thunder {
  background-image: url("./assets/thunder-bg.jpg");
}
#app.hot {
  background-image: url("./assets/warm-bg.jpg");
}
#app.invalid {
  background-image: url("./assets/invalid.jpg");
}
#app.searching {
  background-image: url("./assets/cold1-bg.jpg");
}

main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    66 rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}
.search-box {
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 90%;
  height: 65px;
  padding: 20px;
  font-weight: bold;
  font-family: "Oswald", sans-serif;
  font-style: italic;
  font-size: 25px;
  appearance: none;
  color: grey;
  border: none;
  outline: none;
  background: whitesmoke;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: grey;
  background-color: rgb(195, 223, 238);
  border-radius: 16px 16px 16px 16px;
  transition: 0.6s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: whitesmoke;
  border-radius: 16px 0px 16px 0px;
  color: grey;
}
.location-box .location {
  padding: 17px;
  color: #fff;
  font-size: 37px;
  font-weight: 650;
  font-family: "Oswald", sans-serif;
  font-style: italic;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.location-box .date {
  padding: 12px;
  color: #fff;
  font-size: 38px;
  font-weight: 650;
  font-family: "Oswald", sans-serif;
  font-style: italic;
  padding: 15px;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.weather-box {
  text-align: center;
  padding: 17px;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: whitesmoke;
  font-family: "Pacifico", cursive;
  font-size: 90px;
  font-weight: 650;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .weather {
  padding: 9%;
  color: aliceblue;
  font-size: 50px;
  font-weight: 650;
  font-family: "Pacifico", cursive;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  text-align: center;
}
.alternate {
  padding: 2px;
  color: aliceblue;
  font-size: 35px;
  font-family: "Pacifico", cursive;
  font-weight: 650;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
#app.invalid {
  background-image: url("./assets/invalid.jpg");
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
.inline {
  padding: 15px;
  color: aliceblue;
  font-size: 30px;
  font-family: "Pacifico", cursive;
  font-weight: 650;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.alpha {
  padding: 15px;
  color: aliceblue;
  font-size: 30px;
  font-family: "Pacifico", cursive;
  font-weight: 650;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.button {
  float: right;
  background-color: rgb(195, 223, 238);
  align-self: center;
  border: none;
  outline: none;
  color: #f5f5f5;
  padding: 6px 22px;
  font-size: 30px;
  border-radius: 50%;
}
.button:hover {
  transition: 0.6s;
  color: black;
  background-color: whitesmoke;
}
.list-group-item {
  background-color: rgb(195, 223, 238);
  color: grey;
  width: 75%;
  height: 63px;
  text-align: left;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  outline: none;
  font-size: 24x;
  font-weight: bold;
}
.list-group-item:hover {
  font-style: italic;
  color: grey;
  background-color: whitesmoke;
  transition: 0.5s;
  border-radius: 16px 0px 16px 0px;
  outline: none;
}
span {
  color: black;
}
small {
  font-style: italic;
}
[v-cloak] {
  display: none;
}
</style>