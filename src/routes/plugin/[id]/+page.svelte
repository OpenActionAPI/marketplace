<script lang="ts">
	import { page } from "$app/stores";
	import { marked } from "marked";
	import { onMount } from "svelte";

	import ArrowSquareOut from "phosphor-svelte/lib/ArrowSquareOut";
	import DownloadSimple from "phosphor-svelte/lib/DownloadSimple";

	let pluginDetails: { repository: string; name: string; author: string; download_url: string | undefined };
	let readme = "## Loading plugin details...";
	let download_count = 0;

	async function getTotalDownloadCount(repo: string): Promise<number> {
		const response = await fetch("https://api.github.com/repos/" + repo + "/releases");
		const releases = await response.json();

		let totalCount = 0;
		for (let release of releases) {
			for (let asset of release.assets) {
				totalCount += asset.download_count;
			}
		}

		return totalCount;
	}

	async function getReadme(repo: string): Promise<string> {
		let response = await fetch("https://raw.githubusercontent.com/" + repo + "/master/README.md");
		return await response.text();
	}

	onMount(async () => {
		const response = await fetch("https://openactionapi.github.io/plugins/catalogue.json");
		if (!response.ok) return;

		const data = await response.json();
		pluginDetails = data[$page.params.id];
		if (!pluginDetails) {
			// @ts-expect-error
			pluginDetails = -1;
			return;
		}

		const repo = pluginDetails.repository.split("/")[3] + "/" + pluginDetails.repository.split("/")[4];
		readme = await getReadme(repo);
		download_count = await getTotalDownloadCount(repo);
	});

	function openDeepLink() {
		let iframe = document.getElementById("deeplink-iframe") as HTMLIFrameElement;
		iframe.src = "opendeck://installPlugin/" + $page.params.id;
	}

	function getDownloadAction(): string {
		if (pluginDetails.download_url) return pluginDetails.download_url;
		return pluginDetails.repository + "/releases/latest";
	}
</script>

<svelte:head>
	{#if pluginDetails}
		<title>{pluginDetails.name} - OpenAction Marketplace</title>
	{/if}
	<link
		rel="icon"
		type="image/x-icon"
		href={"https://openactionapi.github.io/plugins/icons/" + $page.params.id + ".png"}
	/>
</svelte:head>

{#if pluginDetails}
	<div class="mx-[2.5%] my-[2.5%] p-10 w-[95vw] h-[90vh] flex flex-col bg-neutral-700 rounded-3xl">
		<div class="flex flex-col md:flex-row items-center md:items-start justify-between">
			<div class="flex flex-row items-center md:items-start">
				<img
					src={"https://openactionapi.github.io/plugins/icons/" + $page.params.id + ".png"}
					alt={pluginDetails.name}
					class="size-16 md:size-48 rounded-2xl"
				/>
				<div class="ml-8 flex flex-col">
					<div class="text-3xl text-neutral-200">{pluginDetails.name}</div>
					<div class="mt-2 flex items-center text-lg text-neutral-400">
						<span class="mr-2">by</span>
						<img
							src={"https://avatars.githubusercontent.com/" +
							pluginDetails.repository.split("/")[3]}
							alt="Avatar URL"
							class="mr-1.5 size-7 rounded-full"
						/>
						<a
							href={"https://github.com/" + pluginDetails.repository.split("/")[3]}
							class="underline"
						>
							{pluginDetails.repository.split("/")[3]}
						</a>
					</div>
				</div>
			</div>

			<div class="my-8 md:mt-0 flex flex-row items-center">
				<iframe title="deeplink-iframe" id="deeplink-iframe" class="hidden"></iframe>

				<button
					on:click={() => openDeepLink()}
					class="px-8 py-1.5 md:py-3 active:translate-y-0.5 font-semibold text-lg text-neutral-100 bg-indigo-600 rounded-l-lg"
				>
					Install
				</button>

				<button
					on:click={async () => window.open(getDownloadAction())}
					class="ml-1 p-2 md:p-3.5 active:translate-y-0.5 text-lg text-neutral-100 bg-indigo-600 rounded-r-lg"
				>
					<ArrowSquareOut size={24} />
				</button>

				<div class="ml-8">
					<p class="flex flex-row text-neutral-300">
						<span class="mr-1 text-lg"> {download_count} </span>
						<DownloadSimple size={28} />
					</p>
				</div>
			</div>
		</div>

		<div class="md:mt-8 p-6 h-full overflow-scroll border-4 border-neutral-600 rounded-xl">
			<p class="readme text-neutral-300">
				{@html marked.parse(readme)}
			</p>
		</div>
	</div>
{:else if pluginDetails == -1}
	<div class="m-auto p-4 items-center">
		<h1 class="font-bold text-3xl text-neutral-200">404 Plugin not found</h1>
		<h1 class="font-semibold text-xl text-neutral-200">Please check that the plugin ID was entered correctly</h1>
	</div>
{/if}
