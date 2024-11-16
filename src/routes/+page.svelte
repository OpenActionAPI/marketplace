<script lang="ts">
	import { onMount } from "svelte";
	import Plugin from "../components/Plugin.svelte";

	let loading = true;
	let data: { [id: string]: { name: string; repository: string; author: string } };
	let searchString = "";

	onMount(async () => {
		try {
			const response = await fetch(
				"https://ninjadev64.github.io/openaction-plugins/catalogue.json",
			);

			data = await response.json();
		} catch (err: any) {
			console.error(err.message);
		} finally {
			loading = false;
		}
	});
</script>

<svelte:head>
	<title>OpenAction Marketplace</title>
	<link rel="icon" href="/favicon.png" />
</svelte:head>

<div class="p-3">
	<div class="p-2 px-8 flex flex-col md:flex-row items-center justify-between">
		<h1 class="text-2xl md:text-3xl text-white font-bold pb-1.5">OpenAction Marketplace</h1>
		<input
			bind:value={searchString}
			placeholder="Search for plugins"
			class="mt-8 md:mt-4 mr-4 ml-4 md:mr-0 p-4 h-12 w-full md:w-72 lg:w-96 text-center rounded-xl text-white bg-[#31363F]"
		/>
	</div>

	{#if !loading}
		<grid>
			<div class="m-auto p-8 gap-4 gap-y-12 grid-container grid">
				{#each Object.entries(data) as [key, plugin]}
					{#if plugin.name.toLowerCase().includes(searchString.toLowerCase())}
						<Plugin {plugin} id={key}></Plugin>
					{/if}
				{/each}
			</div>
		</grid>
	{/if}
</div>
