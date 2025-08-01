<script>
	import { fade, slide } from 'svelte/transition';
	import { tick } from 'svelte';
	import QuestionRadioForm from '$lib/components/QuestionRadioForm.svelte';

	let questions = [
		{
			question: 'What are you looking for?',
			description: 'Let us know your loan requirement',
			options: [
				{ label: 'Home Loan', value: 'Home Loan' },
				{ label: 'Plot Loan Only', value: 'Plot Loan Only' },
				{ label: 'Personal Loan', value: 'Personal Loan' }
			],
			key: 'loanName'
		},
		{
			question: 'What kind of loan is it?',
			description: 'Choose your loan type',
			options: [
				{ label: 'New Loan', value: 'New Loan' },
				{ label: 'Balance Transfer Only', value: 'Balance Transfer Only' }
			],
			key: 'loanType'
		}
	];

	let currentStep = 0;
	let formData = {};

	const nextStep = async () => {
		if (currentStep < questions.length - 1) {
			currentStep++;
			await tick();
			scrollToTop();
		}
	};

	let scrollContainer;
	function scrollToTop() {
		scrollContainer.scrollTo({ top: 0, behavior: 'smooth' });
	}
</script>

<div class="h-screen w-full overflow-hidden flex items-center justify-center bg-gray-100">
	<div bind:this={scrollContainer} class="h-full w-full flex items-center justify-center p-4">
		{#each questions as q, index}
			{#if index === currentStep}
				<!-- Outer div to apply one transition -->
				<div in:fade={{ duration: 400 }} class="w-full max-w-xl">
					<!-- Inner div to apply another transition -->
					<div in:slide={{ y: 50 }}>
						<QuestionRadioForm
							question={q.question}
							description={q.description}
							options={q.options}
							bind:selectedOption={formData[q.key]}
						/>
						<button on:click={nextStep} class="mt-6 px-4 py-2 bg-blue-500 text-white rounded">
							Next
						</button>
					</div>
				</div>
			{/if}
		{/each}
	</div>
</div>
