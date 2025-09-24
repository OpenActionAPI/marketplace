<script lang="ts">
	import { onMount } from "svelte";

	let data: { [id: string]: { name: string; repository: string; author: string } };
	let searchString = "";

	onMount(async () => {
		const response = await fetch("https://openactionapi.github.io/plugins/catalogue.json");
		data = await response.json();
	});
</script>

<svelte:head>
	<title>OpenAction Marketplace</title>
	<link rel="icon" href="/favicon.png" />
</svelte:head>

<div class="flex flex-col md:flex-row items-center justify-between px-6 py-4 bg-neutral-900">
	<h1 class="font-semibold text-2xl text-neutral-200">OpenAction Marketplace</h1>
	<input
		bind:value={searchString}
		placeholder="Search for plugins"
		class="mt-6 md:mt-0 mx-4 md:mr-0 p-3 w-full md:w-72 h-8 text-center text-sm text-neutral-300 bg-neutral-700 rounded-lg"
	/>
</div>

{#if data}
	<div class="grid gap-x-4 gap-y-4 p-8" style="grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))">
		{#each Object.entries(data) as [id, plugin]}
			{#if plugin.name.toLowerCase().includes(searchString.toLowerCase())}
				<a href={"/plugin/" + id} class="flex flex-col items-center p-5 text-xl bg-neutral-700 rounded-3xl">
					<img
						src={"https://openactionapi.github.io/plugins/icons/" + id + ".png"}
						alt={plugin.name}
						class="size-32 rounded-2xl"
						loading="lazy"
					/>
					<p class="pt-4 !font-semibold text-[18px] text-neutral-300 text-nowrap">
						{plugin.name}
					</p>
					<p class="text-[16px] text-neutral-300">
						by {plugin.author}
					</p>
				</a>
			{/if}
		{/each}
	</div>
{:else}
	<h2 class="m-4 text-center md:text-left text-xl text-neutral-200">Loading plugins...</h2>
{/if}
