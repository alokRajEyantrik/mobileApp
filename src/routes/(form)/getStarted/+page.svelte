<script>
	import { onMount } from 'svelte';
	import { loanData } from '$lib/stores/loanData';
	import { preprocessSchemaBindings } from '$lib/utils/schemaUtils';
	import formSchema from '$lib/config/formSchema.json';
	import jsonLogic from 'json-logic-js';

	let selectedLoan = '';
	let currentPageIndex = 0;

	let schema;

	onMount(() => {
		selectedLoan = ($loanData && $loanData.loanName) || '';
		schema = preprocessSchemaBindings(formSchema, selectedLoan);
	});

	// Sanitize strings to be safe keys (replace spaces with underscore)
	function sanitizeKey(value) {
		if (!value) return '';
		return value.replace(/\s+/g, '_');
	}

	// Resolve bindsTo_template placeholders in keys with actual sanitized values
	function resolveBindsTo(question, answers, selectedLoan) {
		if (!question.bindsTo_template) return question.bindsTo || question.id;

		const vars = {
			q1_loanName: sanitizeKey(selectedLoan),
			q2_productType:
				answers[
					resolveBindsTo(
						schema.pages[0].questions.find((q) => q.id === 'q2_productType'),
						answers,
						selectedLoan
					)
				] || '',
			q3_haveAnyObligation:
				answers[
					resolveBindsTo(
						schema.pages[0].questions.find((q) => q.id === 'q3_haveAnyObligation'),
						answers,
						selectedLoan
					)
				] || ''
		};

		const resolvedKey = question.bindsTo_template.replace(
			/\{([^}]+)\}/g,
			(_, key) => vars[key] || ''
		);
		return resolvedKey;
	}

	$: schema = preprocessSchemaBindings(formSchema, selectedLoan);

	// Raw answers store for the selected loan
	$: currentAnswers = $loanData[selectedLoan] ?? {};

	// Build combinedAnswers with fully resolved keys for JsonLogic evaluation
	$: combinedAnswers = {
		q1_loanName: selectedLoan,
		q2_productType:
			currentAnswers[
				resolveBindsTo(
					schema.pages[0].questions.find((q) => q.id === 'q2_productType'),
					currentAnswers,
					selectedLoan
				)
			] || '',
		q3_haveAnyObligation:
			currentAnswers[
				resolveBindsTo(
					schema.pages[0].questions.find((q) => q.id === 'q3_haveAnyObligation'),
					currentAnswers,
					selectedLoan
				)
			] || '',
		...currentAnswers
	};

	// Current page based on page index
	$: currentPage = schema.pages[currentPageIndex];

	// Filter visible questions on the current page
	$: visibleQuestions = currentPage.questions.filter((q) => isQuestionVisible(q, combinedAnswers));

	function onLoanChange(event) {
		selectedLoan = event.target.value || '';
		schema = preprocessSchemaBindings(formSchema, selectedLoan);
		currentPageIndex = 0; // Reset to first page on loan change
	}

	// Helper for options that have value as JSONLogic vars
	function getOptionValue(value) {
		if (typeof value === 'object' && value.var) {
			return combinedAnswers[value.var] || '';
		}
		return value;
	}

	// Save answer with fully resolved key
	function updateAnswer(question, value) {
		if (question.id === 'q1_loanName') {
			selectedLoan = value;
			schema = preprocessSchemaBindings(formSchema, selectedLoan);
			currentPageIndex = 0;
		}

		const key = resolveBindsTo(question, combinedAnswers, selectedLoan);

		loanData.update((data) => {
			if (!data[selectedLoan]) data[selectedLoan] = {};
			data[selectedLoan][key] = value;
			data.loanName = selectedLoan; // also update global loanName
			return data;
		});
	}

	// Visibility logic for questions
	function isQuestionVisible(question, formData) {
		if (!question.showWhen) return true;
		return jsonLogic.apply(question.showWhen, formData);
	}

	// Navigation functions
	function goNext() {
		console.log("click hua hai next")
		if (currentPageIndex < schema.pages.length - 1) {
			currentPageIndex += 1;
		}
	}

	function goPrev() {
		if (currentPageIndex > 0) {
			currentPageIndex -= 1;
		}
	}

	// Check if all required questions are answered on current page (and visible)
	function allRequiredAnswered() {
		return currentPage.questions
			.filter((q) => q.required && isQuestionVisible(q, combinedAnswers))
			.every((q) => {
				const key = resolveBindsTo(q, combinedAnswers, selectedLoan);
				const val = currentAnswers[key];
				return val !== undefined && val !== '' && val !== null;
			});
	}

	// Enable Next button if conditions allow
	$: isNextEnabled = currentPage.nextButtonVisibility
		? currentPage.nextButtonVisibility.mode.includes('allRequiredAnswered') && allRequiredAnswered()
		: true;
</script>

<!-- Dropdown for selecting loan if needed -->
<select on:change={onLoanChange} bind:value={selectedLoan}>
	<option value="">Select a Loan</option>
	{#each Object.keys($loanData) as loan}
		<option value={loan}>{loan}</option>
	{/each}
</select>

<!-- Render visible questions of current page -->
{#each visibleQuestions as question (question.id)}
	<div style="margin-bottom: 1rem;" class="flex flex-col gap-2">
		<label for={question.id}>{question.question}</label>

		{#if question.type === 'radio'}
			{#each question.options as opt}
				<label>
					<input
						type="radio"
						name={question.id}
						value={typeof opt.value === 'object' ? JSON.stringify(opt.value) : opt.value}
						checked={currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] ===
							getOptionValue(opt.value)}
						on:change={() => updateAnswer(question, getOptionValue(opt.value))}
					/>
					{typeof opt.label === 'object' && opt.label.var
						? combinedAnswers[opt.label.var] || opt.label.var
						: opt.label}
				</label>
			{/each}
		{:else if question.type === 'text'}
			<input
				id={question.id}
				type="text"
				value={currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] || ''}
				on:input={(e) => updateAnswer(question, e.target.value)}
			/>
		{/if}
	</div>
{/each}

<!-- Navigation buttons -->
<div style="margin-top: 1rem;">
	{#if currentPageIndex > 0}
		<button on:click={goPrev}>Previous</button>
	{/if}

	{#if currentPageIndex < schema.pages.length - 1}
		<button class="bg-green-400" on:click={goNext} disabled={!isNextEnabled}>Next</button>
	{/if}
</div>

<hr />

<!-- Debug info -->
<div class="bg-black text-white p-4">
	<h3>Debug: currentAnswers</h3>
	<pre>{JSON.stringify(currentAnswers, null, 2)}</pre>
	<h3>Debug: combinedAnswers</h3>
	<pre>{JSON.stringify(combinedAnswers, null, 2)}</pre>
</div>
