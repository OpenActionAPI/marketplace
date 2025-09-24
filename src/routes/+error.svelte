<script>
	import { page } from "$app/stores";
</script>

<h1 class="px-6 py-4 font-semibold text-center md:text-left text-2xl text-neutral-200 bg-neutral-900">
	<a href="/">OpenAction Marketplace</a>
</h1>
<h1 class="m-4 font-semibold text-center md:text-left text-2xl text-red-200">{$page.status}: {$page.error?.message}</h1>
