<script>
	import { fade, slide } from 'svelte/transition';
	import { tick } from 'svelte';
	import QuestionRadioForm from '$lib/components/QuestionRadioForm.svelte';
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

	let currentStep = 0;
	let formData = {};
	let direction = 'forward'; // 'forward' or 'backward'

	const nextStep = async () => {
		if (formData[questions[currentStep].key]) {
			direction = 'forward';
			if (currentStep < questions.length - 1) {
				currentStep++;
				await tick();
				scrollToTop();
			}
		}
	};

	const prevStep = async () => {
		direction = 'backward';
		if (currentStep > 0) {
			currentStep--;
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
			// Use both fade and slide transitions together
			delay: 0,
			duration: 300,
			css: (t, u) => {
				const fadeStyle = fade(node, options).css(t, u);
				const slideStyle = slide(node, options).css(t, u);
				return `${fadeStyle}; ${slideStyle}`;
			}
		};
	}
</script>

<div class="h-screen w-full overflow-hidden flex items-center justify-center bg-gray-100">
	<div bind:this={scrollContainer} class="h-full w-full flex items-center justify-center p-4">
		{#each questions as q, index}
			{#if index === currentStep}
				<!-- Animate based on direction -->
				<div in:fadeSlide={{ x: direction === 'forward' ? 50 : -50 }} class="w-full max-w-xl">
					<div>
						<QuestionRadioForm
							question={q.question}
							description={q.description}
							options={q.options}
							bind:selectedOption={formData[q.key]}
						/>

						<!-- Buttons -->
						<div class="mt-6 flex justify-between">
							{#if currentStep > 0}
								<button on:click={prevStep} class="px-4 py-2 bg-gray-300 text-black rounded">
									Back
								</button>
							{/if}
							{#if formData[q.key]}
								<button
									on:click={nextStep}
									class="px-4 py-2 bg-blue-500 text-white rounded ml-auto"
								>
									Next
								</button>
							{/if}
						</div>
					</div>
				</div>
			{/if}
		{/each}
	</div>
</div>
