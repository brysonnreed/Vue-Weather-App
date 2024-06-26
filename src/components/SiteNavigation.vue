<template>
	<header class="sticky top-0 bg-weather-primary shadow-lg z-50">
		<nav
			class="container flex flex-col sm:flex-row items-center justify-between gap-4 text-white py-6"
		>
			<RouterLink :to="{ name: 'home' }">
				<div class="flex items-center gap-3">
					<i class="fa-solid fa-sun text-2xl" />
					<p class="text-2xl">The local Weather</p>
				</div>
			</RouterLink>
			<div class="flex gap-3">
				<!-- Info Icon -->
				<i
					@click="toggleModal"
					class="fa-solid fa-circle-info text-xl hover:text-weather-secondary transition-colors duration-150 cursor-pointer"
				></i>
				<!-- Plus Icon -->
				<i
					v-if="route.query.preview"
					@click="addCity"
					class="fa-solid fa-plus text-xl hover:text-weather-secondary transition-colors duration-150 cursor-pointer"
				></i>
			</div>
			<BaseModal :modalActive="modalActive" @close-modal="toggleModal">
				<div class="text-black">
					<h1 class="text-2xl mb-1">About:</h1>
					<p class="mb-4">
						The Local Weather allows you to track the current and future weather of cities of your
						choosing.
					</p>
					<h2 class="text-2xl">How it works:</h2>
					<ol class="list-decimal list-inside mb-4">
						<li>Search for your city by entering the name into the search bar.</li>
						<li>
							Select a city within the results, this will take you to the current weather for your
							selection.
						</li>
						<li>
							Track the city by clicking on the "+" icon in the top right. This will save the city
							to view at a later time on the home page.
						</li>
					</ol>

					<h2 class="text-2xl">Removing a city</h2>
					<p>
						If you no longer wish to track a city, simply select the city within the home page. At
						the bottom of the page, there will be am option to delete the city.
					</p>
				</div>
			</BaseModal>
		</nav>
	</header>
</template>

<script setup>
	import { RouterLink, useRouter } from "vue-router";
	import BaseModal from "./BaseModal.vue";
	import { ref } from "vue";
	import { uid } from "uid";
	import { useRoute } from "vue-router";

	const route = useRoute();
	const router = useRouter();

	const modalActive = ref(null);
	const toggleModal = () => {
		modalActive.value = !modalActive.value;
	};

	const savedCities = ref([]);

	// NOTE: Add city to local storage
	const addCity = () => {
		// If local storage already exists then add it to savedCities
		if (localStorage.getItem("savedCities")) {
			savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
		}
		// Define format for local storage
		const locationObj = {
			id: uid(),
			state: route.params.state,
			city: route.params.city,
			coords: {
				lat: route.query.lat,
				lng: route.query.lng,
			},
		};

		// Adding the city to variable and setting it to local storage
		savedCities.value.push(locationObj);
		localStorage.setItem("savedCities", JSON.stringify(savedCities.value));

		// Remove preview query from URL
		let query = Object.assign({}, route.query);
		delete query.preview;
		query.id = locationObj.id;
		router.replace({ query });
	};
</script>
