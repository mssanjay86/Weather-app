<template>
    <div class="container">
        <h1>Weather App</h1>
        <div class="child-container">

            <div class="search-text">
                <input v-model="city" class="input-field-stylings" type="text" placeholder="Enter city">

            </div>
            <div class="search-btn">
                <button class="search-btn-stylings" @click="searchCity(city)">Search</button>
            </div>
            <div class="location">
                <p>{{ weatherData.cityName }}, <span>{{ weatherData.stateName }}</span>, <span>{{ weatherData.country
                }}</span>
                </p>
            </div>
            <div class="date">
                {{ today }}
            </div>
            <div class="weather-details">
                <div class="weather-child">
                    <img class="img-style" :src=weatherIcon alt="weather icon">
                </div>

                <div class="weather-child">
                    <div class="degree">{{ weatherData.temperature.toFixed(0) }}&#8451;</div>
                    <p>{{ weatherData.description }}</p>
                </div>
                <div class="weather-child right-border">

                </div>

                <div class="weather-others">
                    <div class="weather-child margin-spacing">
                        <div class="font-size-increase">23</div>
                        <p>High</p>
                    </div>
                    <div class="weather-child margin-spacing">
                        <div class="font-size-increase">{{ weatherData.windSpeed.toFixed(0) }} kph</div>
                        <p>Wind</p>
                    </div>

                    <div class="weather-child margin-spacing">
                        <div class="font-size-increase">{{ weatherData.sunrise }}</div>
                        <p>Sunrise</p>
                    </div>
                    <div class="weather-child margin-spacing">
                        <div class="font-size-increase">{{ weatherData.low }}</div>
                        <p>Low</p>
                    </div>
                    <div class="weather-child">
                        <div class="font-size-increase">{{ weatherData.humidity }}%</div>
                        <p>Humidity</p>
                    </div>
                    <div style="margin-left: 30px;" class="weather-child">
                        <div class="font-size-increase">{{ weatherData.sunset }}</div>
                        <p>Sunset</p>
                    </div>
                </div>

            </div>






        </div>

    </div>
</template>


<script lang="ts" setup>
import { onMounted, ref } from 'vue';


interface Root {
    coord: Coord
    weather: Weather[]
    base: string
    main: Main
    visibility: number
    wind: Wind
    clouds: Clouds
    dt: number
    sys: Sys
    timezone: number
    id: number
    name: string
    cod: number
}

interface Coord {
    lon: number
    lat: number
}

interface Weather {
    id: number
    main: string
    description: string
    icon: string
}

interface Main {
    temp: number
    feels_like: number
    temp_min: number
    temp_max: number
    pressure: number
    humidity: number
}

interface Wind {
    speed: number
    deg: number
}

interface Clouds {
    all: number
}

interface Sys {
    type: number
    id: number
    country: string
    sunrise: number
    sunset: number
}

interface weatherAttributes {

    country: string,
    cityName: string,
    stateName: string,
    temperature: number,
    high: number,
    low: number,
    windSpeed: number,
    sunrise: string,
    sunset: string,
    humidity: number,
    description: string,
    icon: string
}


const city = ref<string>('')
// city.value[0].toUpperCase();
const weatherData = ref<weatherAttributes>({
    country: "India",
    cityName: "",
    stateName: "Tamil Nadu",
    temperature: 0,
    high: 0,
    low: 0,
    windSpeed: 0,
    sunrise: "",
    sunset: "",
    humidity: 0,
    description: "",
    icon: ""
});

const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
let today: any = new Date();
today = today.toLocaleDateString("en-US", options);


onMounted(() => {
    searchCity("Chennai")
})
const weatherIcon = ref();
const searchCity = async (city: any) => {

    const fetchedData = await fetch("https://api.openweathermap.org/data/2.5/weather?q=" + city + "&appid=4d8fb5b93d4af21d66a2948710284366&units=metric")
        .then(res => res.json())
        .then(data => {

            return data;

        }).catch(error => {
            console.log(error);

        })

    weatherData.value.cityName = fetchedData.name
    weatherData.value.temperature = fetchedData.main['temp']
    weatherData.value.description = fetchedData.weather[0].description
    weatherData.value.high = fetchedData.main['temp_max']
    weatherData.value.low = fetchedData.main['temp_min']

    weatherData.value.sunrise = getLocalTime(fetchedData.sys.sunrise)
    weatherData.value.sunset = getLocalTime(fetchedData.sys.sunset)

    weatherData.value.windSpeed = fetchedData.wind['speed']
    weatherData.value.humidity = fetchedData.main['humidity']
    weatherIcon.value = `https://openweathermap.org/img/wn/${fetchedData.weather[0].icon}@2x.png`


    let getStateAndCountry = await getLocation(fetchedData.coord.lat, fetchedData.coord.lon)


}
const getLocalTime = (time: number) => {

    const date = new Date(time); // Convert seconds to milliseconds

    // Get the local date and time components from the Date object

    const localTime = date.toLocaleTimeString();


    return localTime;
}

interface Root {
    place_id: number
    licence: string
    osm_type: string
    osm_id: number
    lat: string
    lon: string
    place_rank: number
    category: string
    type: string
    importance: number
    addresstype: string
    name: string
    display_name: string
    address: Address
    boundingbox: string[]
}

interface Address {
    road: string
    neighbourhood: string
    suburb: string
    city_district: string
    city: string
    state_district: string
    state: string
    "ISO3166-2-lvl4": string
    postcode: string
    country: string
    country_code: string
}


interface StateAndCountry {
    state: Address,
    country: Address
}

async function getLocation(latitude: number, longitude: number) {
    let fetchValue: StateAndCountry;
    // Call the Nominatim API with the latitude and longitude
    let fetchedStateCountry = fetch(`https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat=${latitude}&lon=${longitude}`)
        .then(response => response.json())
        .then(data => {
            // Get the state and country from the response
            const state = data.address.state;
            const country = data.address.country;
            console.log(`State: ${state}, Country: ${country}`);

            weatherData.value.country = country;
            weatherData.value.stateName = state
        })


        .catch(error => console.log(error));
}


</script>
