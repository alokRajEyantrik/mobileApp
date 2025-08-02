<script>
	import { currentStep, formData } from '$lib/stores/formStepper';
	import { tick } from 'svelte';
	import { get } from 'svelte/store';

	let direction = 'forward';

	// 	const nextStep = async () => {
	// 		const step = get(currentStep);
	// 		const data = get(formData);
	// 		if (data[questions[step].key]) {
	// 			direction = 'forward';
	// 			if (step < questions.length - 1) {
	// 				currentStep.set(step + 1);
	// 				await tick();
	// 				scrollToTop();
	// 			}
	// 		}
	// 	};

	// 	const prevStep = async () => {
	// 		const step = get(currentStep);
	// 		direction = 'backward';
	// 		if (step > 0) {
	// 			currentStep.set(step - 1);
	// 			await tick();
	// 			scrollToTop();
	// 		}
	// 	};
	//
</script>

<div class="max-w-[1200px] mx-auto bg-white shadow-md rounded-lg">
	
	<slot />

	<!-- Buttons -->
	<!-- <div class="mt-6 flex justify-between">
		{#if $currentStep > 0}
			<button on:click={prevStep} class="px-4 py-2 bg-gray-300 text-black rounded"> Back </button>
		{/if}
		{#if $formData[q.key]}
			<button on:click={nextStep} class="px-4 py-2 bg-blue-500 text-white rounded ml-auto">
				Next
			</button>
		{/if}
	</div> -->
</div>
