<script lang="ts">
	export let text: string;
	export let icon: string;
	export let className: string = '';
</script>
<template>
	<section class={"flex py-4 px-6 border shadow-md " + className}>
		<aside class="pr-4 basis-14 text-2xl flex"><p>{icon}</p></aside>
		<div>
			<p class="text-lg text-gray-700">{text}</p>
			<div class="text-xl"><slot></slot></div>
		</div>
	</section>
</template>
