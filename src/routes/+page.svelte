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

<div class="p-3">
	<div class="mx-8 mt-6 flex flex-col md:flex-row items-center justify-between">
		<h1 class="pb-1.5 font-bold text-2xl md:text-3xl text-neutral-200">OpenAction Marketplace</h1>
		<input
			bind:value={searchString}
			placeholder="Search for plugins"
			class="mt-6 md:mt-0 mx-4 md:mr-0 p-4 w-full md:w-72 lg:w-[32rem] h-12 text-center text-neutral-300 bg-neutral-700 rounded-xl"
		/>
	</div>

	{#if data}
		<div class="p-8 gap-x-4 gap-y-12 grid" style="grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))">
			{#each Object.entries(data) as [id, plugin]}
				{#if plugin.name.toLowerCase().includes(searchString.toLowerCase())}
					<a href={"/plugin/" + id} class="p-5 items-center flex flex-col text-xl bg-neutral-700 rounded-3xl">
						<img
							src={"https://openactionapi.github.io/plugins/icons/" + id + ".png"}
							alt={plugin.name}
							class="size-32 rounded-2xl"
							loading="lazy"
						/>
						<p class="pt-4 !font-semibold text-[18px] text-neutral-300">
							{plugin.name}
						</p>
						<p class="text-[16px] text-neutral-300">
							by {plugin.author}
						</p>
					</a>
				{/if}
			{/each}
		</div>
	{/if}
</div>
