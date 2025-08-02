<script lang="ts">
	import { onMount, tick } from 'svelte';
	import { get } from 'svelte/store';
	import { currentStep, formData } from '$lib/stores/formStepper';
	import { visibleQuestions } from '$lib/stores/visibleQuestions';
	import QuestionRadioForm from '$lib/components/QuestionRadioForm.svelte';
	import { loadPipeline, updatePipelineValue } from '$lib/utils/updatePipeline';

	let container: HTMLDivElement;

	async function scrollToTop() {
		await tick();
		container?.scrollTo({ top: 0, behavior: 'smooth' });
	}

	async function nextStep() {
		const $form = get(formData);
		const $questions = get(visibleQuestions);
		const step = get(currentStep);

		if (step >= $questions.length) {
			console.warn('Reached end of questions');
			return;
		}

		const currentQ = $questions[step];
		const key = currentQ?.contextKey;

		if (key && $form[key]) {
			currentStep.set(step + 1);
			await scrollToTop();
		} else {
			console.warn('Missing value for:', key, 'Current formData:', $form);
		}
	}

	async function prevStep() {
		const step = get(currentStep);
		if (step > 0) {
			currentStep.set(step - 1);
			await scrollToTop();
		}
	}
</script>

<div class="h-screen w-full flex flex-col justify-between items-center bg-gray-100 p-4">
	<!-- Question container -->
	<div bind:this={container} class="flex-1 w-full flex justify-center items-center">
		{#if $visibleQuestions.length > 0 && $currentStep < $visibleQuestions.length}
			{#each $visibleQuestions as q, index}
				{#if index === $currentStep}
					<div class="w-full max-w-xl">
						<QuestionRadioForm
							question={q.question}
							description={q.info}
							options={q.options}
							type={q.type}
							bind:selectedOption={$formData[q.contextKey]}
							on:change={(e) => {
								updatePipelineValue(q.contextKey, e.detail.value, formData);
							}}
						/>
					</div>
				{/if}
			{/each}
		{:else if $currentStep >= $visibleQuestions.length}
			<!-- All steps complete -->
			<div class="text-center text-lg font-semibold">✅ All questions answered!</div>
		{/if}
	</div>

	<!-- Buttons -->
	<div class="w-full max-w-xl flex justify-between items-center mt-4">
		{#if $currentStep > 0}
			<button on:click={prevStep} class="px-4 py-2 rounded bg-gray-300 text-black">
				⬅️ Back
			</button>
		{:else}
			<div></div>
		{/if}

		{#if $formData[$visibleQuestions[$currentStep]?.contextKey]}
			<button on:click={nextStep} class="px-4 py-2 rounded bg-blue-500 text-white ml-auto">
				Next ➡️
			</button>
		{:else}
			<button
				disabled
				class="px-4 py-2 rounded bg-blue-200 text-white ml-auto opacity-60 cursor-not-allowed"
			>
				Next ➡️
			</button>
		{/if}
	</div>
</div>

<pre class="text-xs text-gray-500">
	step: {$currentStep}
	<!-- visible: {JSON.stringify($visibleQuestions, null, 2)} -->
	data: {JSON.stringify($formData, null, 2)}
</pre>
