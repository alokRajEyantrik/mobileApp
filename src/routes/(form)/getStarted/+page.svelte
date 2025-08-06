<script>
    import { onMount } from 'svelte';
    import { loanData } from '$lib/stores/loanData';
    import { preprocessSchemaBindings } from '$lib/utils/schemaUtils';
    import formSchema from '$lib/config/formSchema.json';
    import jsonLogic from 'json-logic-js';

    // Import your JSON mappings
    import gstStateCodes from '$lib/config/gstStateCodes.json';
    import pincode_IN_Selected from '$lib/config/pincode_IN_Selected.json';

    import { writable, get } from 'svelte/store';

    // Currently selected loan name
    let selectedLoan = '';

    // Current page index in the multi-page form
    let currentPageIndex = 0;

    // The processed form schema after applying the selected loan context
    let schema;

    // Reactive store for GST state validation error (stored centrally)
    const gstStateError = writable('');

    // Initialize on mount
    onMount(() => {
        selectedLoan = ($loanData && $loanData.loanName) || '';
        schema = preprocessSchemaBindings(formSchema, selectedLoan);
    });

    // Utility: sanitize keys by replacing spaces with underscores
    function sanitizeKey(value) {
        if (!value) return '';
        return value.replace(/\s+/g, '_');
    }

    // Resolve bindsTo key dynamically with placeholders replaced
    function resolveBindsTo(question, answers, selectedLoan) {
        if (!question.bindsTo_template) return question.bindsTo || question.id;

        return question.bindsTo_template.replace(/\{([^}]+)\}/g, (_, key) => {
            if (key === 'q1_loanName') return sanitizeKey(selectedLoan);
            const val = answers[key];
            return typeof val === 'string' ? sanitizeKey(val) : val || '';
        });
    }

    // Reprocess schema whenever selectedLoan changes
    $: schema = preprocessSchemaBindings(formSchema, selectedLoan);

    // Current answers for selected loan
    $: currentAnswers = $loanData[selectedLoan] ?? {};

    // Combined answers object with all keys resolved and filled with current answers or empty string
    $: combinedAnswers = (() => {
        const combined = {};
        for (const page of schema.pages) {
            for (const q of page.questions) {
                const key = resolveBindsTo(q, currentAnswers, selectedLoan);
                if (key) {
                    combined[key] = currentAnswers[key] ?? '';
                }
            }
        }
        combined['q1_loanName'] = selectedLoan;
        return combined;
    })();

    // Current page object
    $: currentPage = schema.pages[currentPageIndex];

    // Questions visible on current page by evaluating showWhen with JSONLogic
    $: visibleQuestions = currentPage.questions.filter(q => isQuestionVisible(q, combinedAnswers));

    // Handler for loan selection change
    function onLoanChange(event) {
        selectedLoan = event.target.value || '';
        schema = preprocessSchemaBindings(formSchema, selectedLoan);
        currentPageIndex = 0;
    }

    // Resolve an option's value, handling JSONLogic reference
    function getOptionValue(value) {
        if (typeof value === 'object' && value.var) {
            return combinedAnswers[value.var] || '';
        }
        return value;
    }

    // Helper to directly update loanData's answer by key (for State field updates)
    function updateAnswerByKey(key, value) {
        loanData.update(data => {
            if (!data[selectedLoan]) data[selectedLoan] = {};
            data[selectedLoan][key] = value;
            data.loanName = selectedLoan;
            return data;
        });
    }

    // Extract GST state code and update the State field automatically
    function updateStateFromGST(gstNumber) {
        if (!gstNumber || gstNumber.length < 2) {
            updateAnswerByKey('State', '');
            gstStateError.set('');
            return;
        }

        const stateCode = gstNumber.substring(0, 2);
        const stateName = gstStateCodes[stateCode];

        if (!stateName) {
            updateAnswerByKey('State', '');
            gstStateError.set('Invalid GST state code');
            return;
        }

        if (!pincode_IN_Selected[stateName]) {
            updateAnswerByKey('State', '');
            gstStateError.set("We're not serve their");
            return;
        }

        updateAnswerByKey('State', stateName);
        gstStateError.set('');
    }

    // Update answer on user input
    function updateAnswer(question, value) {
        if (question.id === 'q1_loanName') {
            selectedLoan = value;
            schema = preprocessSchemaBindings(formSchema, selectedLoan);
            currentPageIndex = 0;
        }

        const key = resolveBindsTo(question, currentAnswers, selectedLoan);

        loanData.update(data => {
            if (!data[selectedLoan]) data[selectedLoan] = {};
            data[selectedLoan][key] = value;
            data.loanName = selectedLoan;
            return data;
        });

        // If GST Number changed, update State accordingly
        if (key === 'GSTNumber') {
            updateStateFromGST(value);
        }
    }

    // Evaluate question visibility using JSONLogic
    function isQuestionVisible(question, formData) {
        if (!question.showWhen) return true;
        return jsonLogic.apply(question.showWhen, formData);
    }

    // Pagination handlers
    function goNext() {
        if (currentPageIndex < schema.pages.length - 1) {
            currentPageIndex += 1;
        }
    }

    function goPrev() {
        if (currentPageIndex > 0) {
            currentPageIndex -= 1;
        }
    }

    // Check if all visible required questions on the current page have answers
    function allRequiredAnswered() {
        return currentPage.questions
            .filter(q => q.required && isQuestionVisible(q, combinedAnswers))
            .every(q => {
                const key = resolveBindsTo(q, combinedAnswers, selectedLoan);
                const val = currentAnswers[key];
                return val !== undefined && val !== '' && val !== null;
            });
    }

    // Check if question currently has validation error
    function hasValidationError(question, answers) {
        return !!getValidationErrorMessage(question, answers);
    }

    // Get specific validation error message based on required, customValidator, and GST state error
    function getValidationErrorMessage(question, answers) {
        const key = resolveBindsTo(question, answers, selectedLoan);
        const val = answers[key];

        if (question.required && (val === undefined || val === '' || val === null)) {
            return question.errorMessage?.required || 'This field is required';
        }

        if (question.validation?.condition) {
            const isInvalid = jsonLogic.apply(question.validation.condition, answers);
            if (isInvalid) {
                return question.errorMessage?.message || 'Invalid value';
            }
        }

        // For State field, check GST state error store to show its error message
        if (question.bindsTo === 'State' || question.id === 'q_state') {
            const gstErr = get(gstStateError);
            if (gstErr) {
                return question.errorMessage?.stateNotServed || gstErr;
            }
        }

        return null;
    }

    // Next button enabled check
    $: isNextEnabled = (() => {
        let enabled = true;

        if (currentPage.nextButtonVisibility) {
            enabled =
                currentPage.nextButtonVisibility.mode.includes('allRequiredAnswered') &&
                allRequiredAnswered();
        }

        for (const q of currentPage.questions) {
            if (isQuestionVisible(q, combinedAnswers) && hasValidationError(q, combinedAnswers)) {
                enabled = false;
                break;
            }
        }

        return enabled;
    })();
</script>

<!-- Render all visible questions on current page -->
{#each visibleQuestions as question (question.id)}
    <div style="margin-bottom: 1rem;" class="flex flex-col gap-2">
        <label for={question.id}>{question.question}</label>

        {#if question.type === 'radio'}
            {#each question.options as opt}
                <label>
                    <input
                        type="radio"
                        name={question.id}
                        value={getOptionValue(opt.value)}
                        checked={currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] === getOptionValue(opt.value)}
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
                class="border border-gray-300 p-2 rounded outline-none"
                readonly={question.uiMeta?.readonly}
                value={currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] || ''}
                on:input={(e) => updateAnswer(question, e.target.value)}
            />
        {/if}

        <!-- Display error message from validation -->
        {#if getValidationErrorMessage(question, combinedAnswers)}
            <p style="color: red; font-weight: bold; margin-top: 0.5rem;">
                {getValidationErrorMessage(question, combinedAnswers)}
            </p>
        {/if}
    </div>
{/each}

<!-- Navigation -->
<div style="margin-top: 1rem;">
    {#if currentPageIndex > 0}
        <button on:click={goPrev}>Previous</button>
    {/if}

    {#if currentPageIndex < schema.pages.length - 1}
        <button disabled={!isNextEnabled} class="bg-green-400" on:click={goNext}>Next</button>
    {/if}
</div>

<hr />

<!-- Debug info -->
<div class="bg-black text-white p-4">
    <h3>Debug: currentAnswers</h3>
    <pre>{JSON.stringify(currentAnswers, null, 2)}</pre>

    <h3>Debug: combinedAnswers</h3>
    <pre>{JSON.stringify(combinedAnswers, null, 2)}</pre>

    <h3>Debug: GST State Error</h3>
    <pre>{$gstStateError}</pre>
</div>
