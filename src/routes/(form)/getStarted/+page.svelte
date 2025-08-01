<script>
	import FormCard from '$lib/components/FormCard.svelte';
	import ProgressBar from '$lib/components/ProgressBar.svelte';
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
	import { tick } from 'svelte';

	let homeLoanOptions = [
		{ label: 'Home Loan', value: 'Home Loan', groupName: 'loanName', groupId: 'HL', Icon: House },
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
			groupId: 'LAP',
			groupName: 'loanName',
			Icon: HousePlus
		},
		{
			label: 'Personal Loan',
			value: 'Personal Loan',
			groupId: 'PL',
			groupName: 'loanName',
			Icon: PersonStanding
		},
		{
			label: 'Business Loan',
			value: 'Business Loan',
			groupId: 'BL',
			groupName: 'loanName',
			Icon: Factory
		},
		{
			label: 'Professional Loan',
			value: 'Professional Loan',
			groupId: 'ProfessionalLoan',
			groupName: 'loanName',
			Icon: BriefcaseBusiness
		}
	];

	let typeOfLoanOptions = [
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
	];

	$: formData = {
		loanName: null,
		loanType: null,
		tenure: null,
		income: null,
		city: null,
		employmentType: null,
		creditScore: null
	};

	let scrollContainer;
	let secondQuestionAnchor;

	$: if (formData.loanName) {
		tick().then(() => {
			if (scrollContainer && secondQuestionAnchor) {
				// Offset to push the second card closer to the top
				const scrollTo = secondQuestionAnchor.offsetTop - 24;
				scrollContainer.scrollTo({
					top: scrollTo,
					behavior: 'smooth'
				});
			}
		});
	}
</script>

<div class="flex flex-col gap-4 justify-center items-center w-full mx-auto">
	<h2 class=" font-bold">Let's Rock!</h2>
	<div class="flex-grow w-full max-w-md">
		<ProgressBar currentStep={1} />
	</div>
	<!-- Scrollable container -->
	<div
		bind:this={scrollContainer}
		class="max-w-[480px] mx-auto h-screen overflow-y-auto flex flex-col gap-[3rem] "
	>
		<FormCard
			question="What are you looking for?"
			description="Get started by letting us know a little bit about what you need"
			options={homeLoanOptions}
			bind:selectedOption={formData.loanName}
		/>

		{#if formData.loanName}
			<!-- Anchor with margin to scroll past the first question -->
			<div bind:this={secondQuestionAnchor} class="h-0 mt-[-1rem]"></div>

			<FormCard
				question="What type of loan are you planning?"
				description="It will help us to identify your requirements, specifically"
				options={typeOfLoanOptions}
				bind:selectedOption={formData.loanType}
			/>
		{/if}
	</div>
</div>
