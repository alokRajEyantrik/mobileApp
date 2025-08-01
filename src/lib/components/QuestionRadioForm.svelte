<script lang="ts">
	import type { SvelteComponent } from 'svelte';
	import Radio from './Radio.svelte';

	type Option = {
		label: string;
		value: string;
		groupName: string;
		groupId: string;
		Icon: typeof SvelteComponent;
	};

	export let question: string;
	export let description: string;
	export let options: Option[] = [];
	export let selectedOption: string | null = null;
</script>

<div class="flex flex-col justify-between h-full">
	<div class="flex flex-col gap-4">
		<div class="flex flex-col gap-1">
			<h2 class="font-bold text-md tracking-wide">{question}</h2>
			<p class=" font-normal italic text-sm text-gray-600">{description}</p>
		</div>
		<div class="flex flex-col gap-2">
			{#each options as option, index}
				<Radio
					groupValue={option.value}
					bind:groupVal={selectedOption}
					groupName={option.groupName}
					groupId={option.groupId}
					Icon={option.Icon}
				/>
			{/each}
		</div>
	</div>
</div>
