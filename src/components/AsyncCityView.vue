<template>
	<div class="flex flex-col flex-1 items-center">
		<!-- BANNER -->
		<div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
			<p>You are currently previewing this city, click the "+" icon to start tracking this city.</p>
		</div>
		<!-- Weather Overview -->
		<div class="flex flex-col items-center text-white py-12">
			<h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
			<p class="text-sm mb-12">
				{{
					new Date(weatherData.currentTime).toLocaleDateString("en-us", {
						weekday: "short",
						day: "2-digit",
						month: "long",
					})
				}}
				{{
					new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
						timeStyle: "short",
					})
				}}
			</p>
			<p class="text-8xl mb-8">{{ Math.round(weatherData.current.temp) }}&deg;</p>
			<div class="text-center">
				<p>Feels Like {{ Math.round(weatherData.current.feels_like) }}&deg;</p>
				<p class="capitalize">
					{{ weatherData.current.weather[0].description }}
				</p>
				<img
					class="w-[150px] h-auto"
					:src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
					:alt="`Weather icon for ${route.params.city}`"
				/>
			</div>
		</div>

		<hr class="border-white/10 border w-full" />

		<!-- Hourly Weather -->
		<div class="bg-weather-secondary w-full mx-auto">
			<div class="max-w-screen-md w-full py-12 mx-auto">
				<swiper
					:slides-per-view="10"
					:space-between="0"
					:scrollbar="{ draggable: true }"
					class="min-h-[30vh] !flex items-center justify-center !z-0"
				>
					<swiper-slide
						class="!flex flex-col gap-4 items-center !text-white cursor-grab"
						v-for="hourData in weatherData.hourly"
						:key="hourData.dt"
					>
						<p class="whitespace-nowrap text-md">
							{{
								new Date(hourData.currentTime).toLocaleTimeString("en-us", {
									hour: "numeric",
								})
							}}
						</p>
						<img
							class="w-auto h-[50px] object-cover"
							:src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
							:alt="`Weather icon for ${route.params.city} in the hour of ${new Date(
								hourData.currentTime
							).toLocaleTimeString('en-us', { hour: 'numeric' })}`"
						/>
						<p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
					</swiper-slide>
				</swiper>
			</div>
		</div>

		<hr class="border-white/10 border w-full" />

		<!-- Weekly Weather -->
		<div class="max-w-screen-md w-full py-12">
			<div class="mx-8 text-white">
				<h2 class="mb-4">7 Day Forecast</h2>
				<div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center">
					<p class="flex-1">
						{{
							new Date(day.dt * 1000).toLocaleDateString("en-us", {
								weekday: "long",
							})
						}}
					</p>
					<img
						class="w-[50px] h-[50px] object-cover"
						:src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
						:alt="`Weather icon for ${route.params.city} for day ${new Date(
							day.dt * 1000
						).toLocaleDateString('en-us', {
							weekday: 'long',
						})}`"
					/>
					<div class="flex gap-2 flex-1 justify-end">
						<p>H: {{ Math.round(day.temp.max) }}</p>
						<p>L: {{ Math.round(day.temp.min) }}</p>
					</div>
				</div>
			</div>
		</div>
		<div
			@click="removeCity"
			class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
		>
			<i class="fa-solid fa-trash"></i>
			<p>Remove City</p>
		</div>
	</div>
</template>

<script setup>
	// import Swiper core and required modules
	import { Navigation, Pagination, Scrollbar } from "swiper/modules";

	// Import Swiper Vue.js components
	import { Swiper, SwiperSlide } from "swiper/vue";

	// Import Swiper styles
	import "swiper/css";
	import "swiper/css/navigation";
	import "swiper/css/pagination";
	import "swiper/css/scrollbar";

	const onSwiper = (swiper) => {
		console.log(swiper);
	};
	const onSlideChange = () => {
		console.log("slide change");
	};

	import axios from "axios";
	import { useRoute, useRouter } from "vue-router";

	const route = useRoute();
	const router = useRouter();

	const getWeatherData = async () => {
		try {
			const weatherData = await axios.get(
				`https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}
				&lon=${route.query.lng}&exclude={part}&appid=${
					import.meta.env.VITE_OPEN_WEATHER_KEY
				}&units=imperial`
			);

			// Calculate current date & time
			const localOffset = new Date().getTimezoneOffset() * 60000;
			const utc = weatherData.data.current.dt * 1000 + localOffset;
			weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset;

			// Calculate hourly weather offset
			weatherData.data.hourly.forEach((hour) => {
				const utc = hour.dt * 1000 + localOffset;
				hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
			});

			// Flicker Delay
			await new Promise((res) => setTimeout(res, 1000));

			return weatherData.data;
		} catch (error) {
			console.log(error);
		}
	};
	const weatherData = await getWeatherData();

	const removeCity = () => {
		const cities = JSON.parse(localStorage.getItem("savedCities"));

		const updatedCities = cities.filter((city) => city.id !== route.query.id);
		localStorage.setItem("savedCities", JSON.stringify(updatedCities));
		router.push({
			name: "home",
		});
	};
</script>
