<script>
	import Accordion from '@specialdoom/proi-ui/Accordion.svelte';
	import AccordionItem from '@specialdoom/proi-ui/AccordionItem.svelte';

	export let title = '';
	export let description = '';
	export let analysis = '';
</script>

<div class="summary">
	<h2>{title}</h2>
	<Accordion>
		<AccordionItem title={description}>
			{analysis}
		</AccordionItem>
	</Accordion>
</div>

<style>
	.summary {
		display: flex;
		flex-direction: column;
		justify-content: center;
		padding: 12px;
		width: 100%;
	}

	h2 {
		margin-top: 0;
		height: 1.5rem;
		font-size: 1.5rem;
		text-decoration: underline;
		text-align: center;
	}
</style>
