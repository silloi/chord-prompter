<script lang="ts">
	import TextInput from '@specialdoom/proi-ui/TextInput.svelte';

	export let value = '';
	export let label = '';
	export let placeholder = '';
</script>

<div>
	<label for={label}>
		{label}
		<TextInput bind:value {placeholder} />
	</label>
</div>
