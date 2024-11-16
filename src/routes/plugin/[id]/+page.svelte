<script lang="ts">
	import { page } from "$app/stores";
	import { marked } from "marked";
	import { onMount } from "svelte";

	import ArrowSquareOut from "phosphor-svelte/lib/ArrowSquareOut";
	import DownloadCount from "../../../components/DownloadCount.svelte";

	let pluginDetails: { repository: string; name: string; author: string; download_url: string | undefined };
	let readme = "";
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
		const response = await fetch("https://ninjadev64.github.io/openaction-plugins/catalogue.json");
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
		href={"https://ninjadev64.github.io/openaction-plugins/icons/" + $page.params.id + ".png"}
	/>
</svelte:head>

{#if pluginDetails}
	<div class="mx-[2.5%] my-[2.5%] p-10 w-[95vw] h-[90vh] flex flex-col bg-[#31363F] rounded-3xl">
		<div class="flex flex-col md:flex-row items-center md:items-start justify-between">
			<div class="flex flex-row items-center md:items-start">
				<img
					src={"https://ninjadev64.github.io/openaction-plugins/icons/" + $page.params.id + ".png"}
					alt={pluginDetails.name}
					class="size-16 md:size-48 rounded-2xl md:rounded-3xl"
				/>
				<div class="ml-8 flex flex-col">
					<div class="text-3xl text-white">{pluginDetails.name}</div>
					<div class="mt-2 flex items-center text-lg text-slate-400">
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
					class="px-8 py-1.5 md:py-3 active:translate-y-0.5 text-lg text-slate-100 bg-blue-600 rounded-l-lg"
				>
					Install
				</button>

				<button
					on:click={async () => window.open(getDownloadAction())}
					class="ml-1 p-2 md:p-3.5 active:translate-y-0.5 text-lg text-slate-100 bg-blue-600 rounded-r-lg"
				>
					<ArrowSquareOut size={24} />
				</button>

				<div class="ml-8">
					<DownloadCount count={download_count} />
				</div>
			</div>
		</div>

		<div class="md:mt-8 p-6 h-full overflow-scroll border-4 border-slate-600 rounded-xl">
			<p class="readme text-slate-300">
				{@html marked.parse(readme)}
			</p>
		</div>
	</div>
{:else if pluginDetails == -1}
	<div class="m-auto p-4 items-center">
		<h1 class="font-bold text-3xl text-white">404 Plugin not found</h1>
		<h1 class="font-semibold text-xl text-white">Please check that the plugin ID was entered correctly</h1>
	</div>
{/if}
