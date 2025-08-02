<script>
	import { fade, slide } from 'svelte/transition';
	import { tick } from 'svelte';
	import QuestionRadioForm from '$lib/components/QuestionRadioForm.svelte';
	import formConfig from '$lib/config/form-config';
	import {
		House,
		LandPlot,
		HousePlus,
		PersonStanding,
		Factory,
		Plus,
		ArrowLeftRight,
		BriefcaseBusiness
	} from '@lucide/svelte';
	import { currentStep, formData } from '$lib/stores/formStepper';
	import { get } from 'svelte/store';
	import ProgressBar from '$lib/components/ProgressBar.svelte';

	let questions = [
		{
			question: 'What are you looking for?',
			description: 'Let us know your loan requirement',
			options: [
				{
					label: 'Home Loan',
					value: 'Home Loan',
					groupName: 'loanName',
					groupId: 'HL',
					Icon: House
				},
				{
					label: 'Plot Loan Only',
					value: 'Plot Loan Only',
					groupName: 'loanName',
					groupId: 'Plot',
					Icon: LandPlot
				},
				{
					label: 'Loan Against Property (LAP)',
					value: 'Loan Against Property (LAP)',
					groupName: 'loanName',
					groupId: 'LAP',
					Icon: HousePlus
				},
				{
					label: 'Personal Loan',
					value: 'Personal Loan',
					groupName: 'loanName',
					groupId: 'PL',
					Icon: PersonStanding
				},
				{
					label: 'Business Loan',
					value: 'Business Loan',
					groupName: 'loanName',
					groupId: 'BL',
					Icon: Factory
				},
				{
					label: 'Professional Loan',
					value: 'Professional Loan',
					groupName: 'loanName',
					groupId: 'ProfessionalLoan',
					Icon: BriefcaseBusiness
				}
			],
			key: 'loanName'
		},
		{
			question: 'What type of loan are you planning?',
			description: 'It will help us to identify your requirements, specifically',
			options: [
				{
					label: 'New Loan',
					value: 'New Loan',
					groupName: 'loanType',
					groupId: 'newLoan',
					Icon: Plus
				},
				{
					label: 'Balance Transfer Only',
					value: 'Balance Transfer Only',
					groupName: 'loanType',
					groupId: 'balanceTransfer',
					Icon: ArrowLeftRight
				}
			],
			key: 'loanType'
		}
	];

	let direction = 'forward';
	$: showNext = false;

	const nextStep = async () => {
		const step = get(currentStep);
		const data = get(formData);
		if (data[questions[step].key]) {
			direction = 'forward';
			if (step < questions.length - 1) {
				currentStep.set(step + 1);
				await tick();
				scrollToTop();
			}
		}
	};

	const prevStep = async () => {
		const step = get(currentStep);
		direction = 'backward';
		if (step > 0) {
			currentStep.set(step - 1);
			await tick();
			scrollToTop();
		}
	};

	let scrollContainer;
	function scrollToTop() {
		scrollContainer.scrollTo({ top: 0, behavior: 'smooth' });
	}

	function fadeSlide(node, options) {
		return {
			delay: 0,
			duration: 300,
			css: (t, u) => {
				const fadeStyle = fade(node, options).css(t, u);
				const slideStyle = slide(node, options).css(t, u);
				return `${fadeStyle}; ${slideStyle}`;
			}
		};
	}

	function handleScroll(event) {
		const el = event.target;
		const threshold = 50; // You can tweak this
		const scrollBottom = el.scrollTop + el.clientHeight;
		const totalHeight = el.scrollHeight;

		showNext = scrollBottom + threshold >= totalHeight;
	}

	// $: console.log(formConfig, 'formConfig');
</script>

<div
	class="h-screen w-full overflow-hidden flex flex-col items-center justify-center bg-gray-100 p-4"
>
	<div class="flex justify-center">
		<!-- {#if $currentStep > 0} -->
		<button on:click={prevStep} class="px-4 py-2 bg-gray-300 text-black rounded"> Back </button>
		<!-- {/if} -->
		<h2 class=" font-bold">Let's Rock</h2>
	</div>
	<div class="flex-grow w-full max-w-md">
		<ProgressBar currentStep={1} />
	</div>
	<div
		bind:this={scrollContainer}
		class="h-full w-full flex items-center justify-center overflow-auto"
		on:scroll={handleScroll}
	>
		{#each questions as q, index}
			{#if index === $currentStep}
				<!-- Animate based on direction -->
				<div in:fadeSlide={{ x: direction === 'forward' ? 50 : -50 }} class="w-full max-w-xl">
					<div>
						<QuestionRadioForm
							question={q.question}
							description={q.description}
							options={q.options}
							bind:selectedOption={$formData[q.key]}
						/>

						<!-- Buttons -->
						<!-- <div class="mt-6 flex justify-between">
							{#if $currentStep > 0}
								<button on:click={prevStep} class="px-4 py-2 bg-gray-300 text-black rounded">
									Back
								</button>
							{/if}
							{#if $formData[q.key]}
								<button
									on:click={nextStep}
									class="px-4 py-2 bg-blue-500 text-white rounded ml-auto"
								>
									Next
								</button>
							{/if}
						</div> -->
					</div>
				</div>
			{/if}
		{/each}
	</div>
	{#if !showNext}
		<button
			type="button"
			on:click={nextStep}
			class="text-sm text-gray-400 text-center mt-4 animate-bounce">⬇️ Scroll to continue</button
		>
	{/if}
</div>
