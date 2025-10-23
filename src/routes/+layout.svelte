<script>
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<!-- Gotta make a wrapper. Cool. Love those... -->
<div class="w-screen h-svh grid place-content-center">
	<div class="w-fit h-fit bg-surface-900 p-2 rounded-2xl">
		{@render children?.()}
	</div>
</div>