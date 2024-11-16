<script lang="ts">
	import { onMount } from "svelte";
	import Plugin from "../components/Plugin.svelte";

	let data: { [id: string]: { name: string; repository: string; author: string } };
	let searchString = "";

	onMount(async () => {
		const response = await fetch("https://ninjadev64.github.io/openaction-plugins/catalogue.json");
		data = await response.json();
	});
</script>

<div class="p-3">
	<div class="py-2 px-8 flex flex-col md:flex-row items-center justify-between">
		<h1 class="pb-1.5 font-bold text-2xl md:text-3xl text-white">OpenAction Marketplace</h1>
		<input
			bind:value={searchString}
			placeholder="Search for plugins"
			class="mt-8 md:mt-4 mx-4 md:mr-0 p-4 w-full md:w-72 lg:w-[32rem] h-12 text-center text-white bg-[#31363F] rounded-xl"
		/>
	</div>

	{#if data}
		<div class="p-8 gap-x-4 gap-y-12 grid-container grid">
			{#each Object.entries(data) as [key, plugin]}
				{#if plugin.name.toLowerCase().includes(searchString.toLowerCase())}
					<Plugin {plugin} id={key}></Plugin>
				{/if}
			{/each}
		</div>
	{/if}
</div>
