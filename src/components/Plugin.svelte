<script lang="ts">
	export let plugin;
	export let id: string;
</script>

<a href={"/plugin/" + id} class="p-5 items-center flex flex-col text-xl bg-[#31363F] rounded-3xl">
	<img
		src={"https://ninjadev64.github.io/openaction-plugins/icons/" + id + ".png"}
		alt="Plugin Icon"
		class="size-32 rounded-2xl"
	/>
	<p class="pt-4 !font-semibold text-[18px] text-slate-300">
		{plugin.name}
	</p>
	<p class="text-[16px] text-slate-300">
		by {plugin.author}
	</p>
</a>
