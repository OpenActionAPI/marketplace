<script>
	import { page } from "$app/stores";
</script>

<div class="m-auto p-4 items-center">
	<h1 class="font-bold text-3xl text-white">{$page.status}: {$page.error?.message}</h1>
</div>
