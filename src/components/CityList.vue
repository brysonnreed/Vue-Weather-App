<template>
	<div v-for="city in savedCities" :key="city.id">
		<CityCard :city="city" @click="goToCityView(city)" />
	</div>

	<p v-if="savedCities.length === 0">
		No Locations added. To start tracking a location, search in the field above.
	</p>
</template>

<script setup>
	import { ref } from "vue";
	import axios from "axios";
	import CityCard from "./CityCard.vue";
	import { useRouter } from "vue-router";

	const savedCities = ref([]);

	const getCities = async () => {
		if (localStorage.getItem("savedCities")) {
			savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

			// Define array for all get request we will need
			const requests = [];

			// Push the axios requests for each city in local storage
			savedCities.value.forEach((city) => {
				requests.push(
					axios.get(
						`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}
					&lon=${city.coords.lng}&appid=${import.meta.env.VITE_OPEN_WEATHER_KEY}&units=imperial`
					)
				);
			});

			// Wait until all request have been completed
			const weatherData = await Promise.all(requests);

			// Flicker Delay
			await new Promise((res) => setTimeout(res, 1000));

			weatherData.forEach((value, index) => {
				savedCities.value[index].weather = value.data;
			});
		}
	};

	await getCities();

	const router = useRouter();

	const goToCityView = (city) => {
		router.push({
			name: "cityView",
			params: { state: city.state, city: city.city },
			query: {
				lat: city.coords.lat,
				lng: city.coords.lng,
				id: city.id,
			},
		});
	};
</script>
