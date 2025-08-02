<script lang="ts">
	// import { House  } from '@lucide/svelte';
	export let Icon: typeof import('svelte').SvelteComponent | null = null;
	export let groupVal: string | null = null;
	export let groupName: string;
	export let groupValue: string;
	export let showValue: string = ''; // optional fallback
	export let customLabel: string = ''; // NEW — display label instead of groupValue if provided
	export let groupId: string;
	export let onClick = () => {};
	export let onChange = () => {};
	export let className = '';

	function handleClick(event: Event) {
		if (!groupVal || groupVal !== groupValue) {
			onClick(event);
		}
	}

	$: renderLabel = customLabel || showValue || groupValue;
</script>

<div class="flex w-full items-center">
	<label
		for={groupId}
		class={`relative cursor-pointer font-normal text-sm flex border px-4 w-full py-[0.8rem] rounded-md border-iconColor items-center ${className}`}
	>
		<input
			type="radio"
			id={groupId}
			name={groupName}
			class="sr-only"
			bind:group={groupVal}
			value={groupValue}
			on:click={handleClick}
			on:change={onChange}
		/>

		<div class="flex items-center">
			{#if Icon}
				<svelte:component this={Icon} />
			{/if}

			<span class="ml-2">{@html renderLabel}</span>
		</div>
	</label>
</div>

<style>
	label:has(input:checked) {
		background-color: black;
		color: white;
		border: 2px solid #fcb650;
	}
</style>
