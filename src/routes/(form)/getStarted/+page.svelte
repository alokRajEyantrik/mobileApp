<script>
	import FormCard from '$lib/components/FormCard.svelte';
	import {
		House,
		LandPlot,
		HousePlus,
		PersonStanding,
		Factory,
		BriefcaseBusiness
	} from '@lucide/svelte';
	let options = [
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
	];

	$: formData = {
		loanName: null,
		loanAmount: null,
		tenure: null,
		income: null,
		city: null,
		employmentType: null,
		creditScore: null
	};

	$: console.log(formData.loanName, 'selectedOption');
</script>

<section class="max-w-[480px] mx-auto">
	<FormCard
		pageTitle="Let's Rock!"
		step={3}
		question="What are you looking for?"
		description="Get started by letting us know a little bit about what you need"
		{options}
		bind:selectedOption={formData.loanName}
	/>
</section>
