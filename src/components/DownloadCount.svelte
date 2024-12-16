<script lang="ts">
	import DownloadSimple from "phosphor-svelte/lib/DownloadSimple";

	export let count: number | undefined;
</script>

<p class="flex flex-row text-slate-300">
	<span class="mr-1 text-lg"> {count} </span>
	<DownloadSimple size={28} />
</p>
