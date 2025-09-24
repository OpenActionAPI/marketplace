<script lang="ts">
	import { page } from "$app/stores";
	import { onMount } from "svelte";

	import DOMPurify from "dompurify";
	import { marked } from "marked";
	import { baseUrl } from "marked-base-url";

	import ArrowSquareOut from "phosphor-svelte/lib/ArrowSquareOut";
	import DownloadSimple from "phosphor-svelte/lib/DownloadSimple";

	type Plugin = { repository: string; name: string; author: string; download_url: string | undefined };

	let pluginDetails: Plugin | 0;
	let readme = "<strong>Loading plugin details...</strong>";
	let downloadCount = 0;

	async function getTotalDownloadCount(repo: string): Promise<number> {
		const response = await fetch("https://api.github.com/repos/" + repo + "/releases");
		if (!response.ok) return 0;
		const releases = await response.json();

		let totalCount = 0;
		for (const release of releases) {
			for (const asset of release.assets) {
				totalCount += asset.download_count;
			}
		}

		return totalCount;
	}

	async function getReadme(repo: string): Promise<string> {
		const renderer = new marked.Renderer();
		renderer.link = function (token) {
			const rendered = marked.Renderer.prototype.link.call(this, token);
			return rendered.replace("<a", `<a target="_blank" `);
		};
		marked.use({ renderer });
		const urls = [
			"https://raw.githubusercontent.com/" + repo + "/main/README.md",
			"https://raw.githubusercontent.com/" + repo + "/main/readme.md",
			"https://raw.githubusercontent.com/" + repo + "/master/README.md",
			"https://raw.githubusercontent.com/" + repo + "/master/readme.md",
		];
		for (const url of urls) {
			const response = await fetch(url);
			if (response.ok) {
				marked.use(baseUrl(url));
				return await marked.parse(DOMPurify.sanitize(await response.text()).replace(/<a/g, '<a target="_blank" '));
			}
		}
		return await marked.parse("**Plugin README file not found**\n\n[View plugin on GitHub](https://github.com/" + repo + ")");
	}

	onMount(async () => {
		const response = await fetch("https://openactionapi.github.io/plugins/catalogue.json");
		if (!response.ok) return;

		const data = await response.json();
		pluginDetails = data[$page.params.id!];
		if (!pluginDetails) {
			pluginDetails = 0;
			return;
		}

		const repo = pluginDetails.repository.split("/")[3] + "/" + pluginDetails.repository.split("/")[4];
		readme = await getReadme(repo);
		downloadCount = await getTotalDownloadCount(repo);
	});

	function openDeepLink() {
		const iframe = document.getElementById("deeplink-iframe") as HTMLIFrameElement;
		iframe.src = "opendeck://installPlugin/" + $page.params.id;
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

<h1 class="px-6 py-4 font-semibold text-center md:text-left text-2xl text-neutral-200 bg-neutral-900">
	<a href="/">OpenAction Marketplace</a>
</h1>

{#if pluginDetails == 0}
	<div class="m-4 text-center md:text-left text-red-200">
		<h1 class="mb-1 font-semibold text-2xl">404: Plugin Not Found</h1>
		<h1 class="text-xl">Please check that the plugin ID was entered correctly</h1>
	</div>
{:else if pluginDetails}
	<div class="m-6 md:m-8">
		<div class="flex flex-col md:flex-row items-center md:items-start justify-between">
			<div class="flex flex-row items-center md:items-start">
				<img
					src={"https://openactionapi.github.io/plugins/icons/" + $page.params.id + ".png"}
					alt={pluginDetails.name}
					class="size-16 md:size-48 rounded-2xl"
				/>
				<div class="flex flex-col ml-8">
					<div class="text-3xl text-neutral-200">{pluginDetails.name}</div>
					<div class="flex items-center mt-2 text-lg text-neutral-400">
						<span class="mr-2">by</span>
						<img
							src={"https://avatars.githubusercontent.com/" + pluginDetails.repository.split("/")[3]}
							alt="Avatar URL"
							class="size-7 mr-1.5 rounded-full"
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
					class="active:translate-y-0.5 px-8 py-1.5 md:py-3 font-semibold text-lg text-neutral-100 bg-indigo-600 rounded-l-lg"
				>
					Install
				</button>

				<button
					on:click={async () =>
						window.open(
							(pluginDetails as Plugin).download_url ?? (pluginDetails as Plugin).repository + "/releases/latest",
						)}
					class="active:translate-y-0.5 ml-1 p-2 md:p-3.5 text-lg text-neutral-100 bg-indigo-600 rounded-r-lg"
				>
					<ArrowSquareOut size={24} />
				</button>

				{#if downloadCount}
					<div class="flex flex-row ml-8 text-neutral-300">
						<span class="mr-1 text-lg"> {downloadCount} </span>
						<DownloadSimple size={28} />
					</div>
				{/if}
			</div>
		</div>

		<div class="md:mt-8 p-6 border-4 border-neutral-600 rounded-xl">
			<p class="readme text-neutral-300">
				{@html readme}
			</p>
		</div>
	</div>
{/if}
