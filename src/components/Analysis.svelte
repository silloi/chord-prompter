<script lang="ts">
	import Accordion from '@specialdoom/proi-ui/Accordion.svelte';
	import AccordionItem from '@specialdoom/proi-ui/AccordionItem.svelte';

	export let analysis = '';
</script>

<Accordion>
	<AccordionItem title="Analysis">
		{analysis}
	</AccordionItem>
</Accordion>
