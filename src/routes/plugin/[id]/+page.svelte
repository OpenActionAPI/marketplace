<script lang="ts">
	import { page } from "$app/stores";
	import { marked } from "marked";
	import { onMount } from "svelte";

	import ArrowSquareOut from "phosphor-svelte/lib/ArrowSquareOut";
	import DownloadCount from "../../../components/DownloadCount.svelte";

	let pluginDetails: { repository: string; name: string; author: string; download_url: string };

	let readme: string | undefined;
	let download_count: number;

	let response: Response;
	let loading = true;

	async function getDownloadAction(response: any, r: string) {
		if (pluginDetails.download_url) return pluginDetails.download_url;
		return pluginDetails.repository + "/releases/latest";
	}

	function getTotalDownloadCount(response: any) {
		let totalCount = 0;

		for (let release in response) {
			for (let asset in response[release].assets) {
				totalCount += response[release].assets[asset].download_count;
			}
		}

		return totalCount;
	}

	async function getReadme(repo: string): Promise<string> {
		let readme = await fetch(
			"https://raw.githubusercontent.com/" +
				repo +
				"/" +
				"master" +
				"/README.md",
		);
		return await readme.text();
	}

	function trimRepo(r: string) {
		return r.split("/")[3] +
			"/" +
			r.split("/")[4];
	}

	onMount(async () => {
		try {
			const response = await fetch("https://ninjadev64.github.io/openaction-plugins/catalogue.json");
			if (!response.ok) throw new Error("Network response was not ok");

			let data = await response.json();
			pluginDetails = data[$page.params.id];

			if (!pluginDetails) return;
		} catch (err: any) {
			console.error(err.message);
		} finally {
			let repo = trimRepo(pluginDetails.repository);
			response = await fetch("https://api.github.com/repos/" + repo + "/releases");
			response = await response.json();
			readme = await getReadme(repo);
			download_count = getTotalDownloadCount(response);

			loading = false;
		}
	});

	function openDeepLink() {
		// @ts-ignore
		document.getElementById("deeplink-iframe").src ="opendeck://installPlugin/" + $page.params.id;
    }
</script>

<svelte:head>
	{#if !loading && pluginDetails}
		<title>{pluginDetails.name} - OpenAction Marketplace</title>
		<link
			rel="icon"
			type="image/x-icon"
			href={"https://ninjadev64.github.io/openaction-plugins/icons/" +
			$page.params.id +
			".png"}
		/>
	{/if}
</svelte:head>

{#if !loading && pluginDetails}
	<div class="ml-[5%] mt-[2.5%] p-10 w-[90%] flex flex-col rounded-3xl bg-[#31363F]">
		<div class="flex flex-col md:flex-row items-center justify-between">
			<div class="flex">
				<img
					src={"https://ninjadev64.github.io/openaction-plugins/icons/" +
					$page.params.id +
					".png"}
					alt="Plugin Icon"
					class="size-[70px] md:size-[200px] md:rounded-3xl rounded-2xl"
				/>
				<div class="ml-8 mt-0 flex flex-col">
					<div class="text-3xl text-white">
						{pluginDetails.name}
					</div>
					<p class="mt-2 flex text-lg text-slate-400">
						<span class="mr-1 text-lg"> by </span>
						<img
							src={"https://avatars.githubusercontent.com/" +
							pluginDetails.repository.split("/")[3]}
							alt="Avatar URL"
							class="size-[30px] rounded-3xl"
						/>
						<span class="mr-1 text-lg"></span>
						<a
							href={"https://github.com/" +
							pluginDetails.repository.split("/")[3]}
							class="underline"
						>{pluginDetails.repository.split("/")[3]}</a>
					</p>
				</div>
			</div>

			<div class="md:mb-40 mb-10 mt-5 md:mt-0 flex">
				<iframe title="deeplinkiframe" id="deeplink-iframe" style="display: none;"></iframe>
				
				<button
					on:click={() => openDeepLink()}
					class="pb-1.5 pl-8 pr-8 pt-1.5 md:pb-3 md:pt-3 cursor-pointer text-lg text-slate-100 active:translate-y-0.5 bg-blue-600 rounded-l-lg"
				>
					Install
				</button>

				<button
					on:click={async () => {
						window.open(await getDownloadAction(response, trimRepo(pluginDetails.repository)));
					}}
					class="ml-1 p-3 cursor-pointer text-lg text-slate-100 bg-blue-600 rounded-r-lg"
				>
					<ArrowSquareOut size={24} />
				</button>

				<div class="ml-8 mt-2">
					<DownloadCount count={download_count} />
				</div>
			</div>
		</div>

		<div class="mt-0 md:mt-5 p-6 h-96 overflow-scroll rounded-xl border-4 border-slate-600">
			<p class="readme text-slate-300">
				{@html marked.parse(readme!)}
			</p>
		</div>
	</div>
{/if}

{#if !pluginDetails}
	<div class="m-auto p-4 items-center">
		<h1 class="font-bold text-3xl text-white">404 Plugin Not found</h1>
		<h1 class="font-semibold text-xl text-white">Please check the plugin ID was spelled correctly</h1>
	</div>
{/if}
