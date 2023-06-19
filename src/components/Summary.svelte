<script>
	import Card from '@specialdoom/proi-ui/Card.svelte';

	export let title = '';
	export let description = '';
</script>

<div class="summary">
	<h2>{title}</h2>
	<p>{description}</p>
</div>

<style>
	.summary {
		display: flex;
		flex-direction: column;
		justify-content: center;
		padding: 12px;
	}

	h2 {
		margin-top: 0;
		height: 1.5rem;
		font-size: 1.5rem;
		text-decoration: underline;
	}
</style>
