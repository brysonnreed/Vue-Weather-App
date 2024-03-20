<template>
	<main class="container text-white">
		<div class="pt-4 mb-8 relative">
			<input
				type="text"
				v-model="searchQuery"
				@input="getSearchResults"
				placeholder="Search for a City or State"
				class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow[0px+1px_0_0_#004E71]"
			/>
			<ul
				v-if="mapboxSearchResults"
				class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
			>
				<p v-if="searchError">Sorry, something went wrong. Please try again.</p>
				<p v-if="!searchError && mapboxSearchResults.length === 0">
					No results match. Please try a different term.
				</p>
				<template v-else>
					<li
						v-for="searchResult in mapboxSearchResults"
						:key="searchResult.id"
						@click="previewCity(searchResult)"
						class="py-2 cursor-pointer"
					>
						{{ searchResult.place_name }}
					</li>
				</template>
			</ul>
		</div>
		<div class="flex flex-col gap-4">
			<Suspense>
				<CityList />
				<template #fallback>
					<CityCardSkeleton />
				</template>
			</Suspense>
		</div>
	</main>
</template>

<script setup>
	import CityList from "@/components/CityList.vue";
	import axios from "axios";
	import { ref } from "vue";
	import { useRouter } from "vue-router";
	import CityCardSkeleton from "@/components/CityCardSkeleton.vue";

	const router = useRouter();

	const previewCity = (searchResult) => {
		console.log(searchResult);
		const [city, state] = searchResult.place_name.split(", ");

		router.push({
			name: "cityView",
			params: { state: state, city: city },
			query: {
				lat: searchResult.geometry.coordinates[1],
				lng: searchResult.geometry.coordinates[0],
				preview: true,
			},
		});
	};

	const searchQuery = ref("");
	const queryTimeout = ref(null);

	const mapboxSearchResults = ref(null);

	const searchError = ref(null);

	const getSearchResults = () => {
		// Clear the timeout
		clearTimeout(queryTimeout.value);
		// Implenting Lazy Search
		queryTimeout.value = setTimeout(async () => {
			// Make sure that search query exists
			if (searchQuery !== "") {
				try {
					const res = await axios.get(
						`https://api.mapbox.com/geocoding/v5/mapbox.places/${
							searchQuery.value
						}.json?access_token=${import.meta.env.VITE_MAPBOX_KEY}&types=place`
					);
					mapboxSearchResults.value = res.data.features;
				} catch (error) {
					console.log(error);
					searchError.value = true;
				}
				return;
			}
			mapboxSearchResults.value = null;
		}, 300);
	};
</script>
