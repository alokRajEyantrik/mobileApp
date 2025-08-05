<script>
  import { onMount } from "svelte";
  import { loanData } from "$lib/stores/loanData";
  import { preprocessSchemaBindings } from "$lib/utils/schemaUtils";
  import formSchema from "$lib/config/formSchema.json";
  import jsonLogic from "json-logic-js";

  let selectedLoan = "";
  let schema;

  onMount(() => {
    selectedLoan = $loanData?.loanName || "";
    schema = preprocessSchemaBindings(formSchema, selectedLoan);
  });

  function sanitizeKey(value) {
    if (!value) return "";
    return value.replace(/\s+/g, "_");
  }

  function resolveBindsTo(question, answers, selectedLoan) {
    if (!schema) return "";
    if (!question.bindsTo_template) return question.bindsTo || question.id;

    const questions = schema?.pages?.[0]?.questions || [];
    const q2ProductTypeQuestion = questions.find((q) => q.id === "q2_productType");
    const q3HaveAnyObligationQuestion = questions.find((q) => q.id === "q3_haveAnyObligation");

    const vars = {
      q1_loanName: sanitizeKey(selectedLoan),
      q2_productType: "",
      q3_haveAnyObligation: "",
    };

    if (q2ProductTypeQuestion) {
      const key = resolveBindsTo(q2ProductTypeQuestion, answers, selectedLoan);
      vars.q2_productType = sanitizeKey(answers[key] || "");
    }
    if (q3HaveAnyObligationQuestion) {
      const key = resolveBindsTo(q3HaveAnyObligationQuestion, answers, selectedLoan);
      vars.q3_haveAnyObligation = sanitizeKey(answers[key] || "");
    }

    return question.bindsTo_template.replace(/\{([^}]+)\}/g, (_, key) => vars[key] || "");
  }

  $: currentAnswers = $loanData[selectedLoan] ?? {};

  $: combinedAnswers = {
    q1_loanName: selectedLoan,
    q2_productType:
      currentAnswers[
        resolveBindsTo(
          schema?.pages?.[0]?.questions.find((q) => q.id === "q2_productType"),
          currentAnswers,
          selectedLoan
        )
      ] || "",
    q3_haveAnyObligation:
      currentAnswers[
        resolveBindsTo(
          schema?.pages?.[0]?.questions.find((q) => q.id === "q3_haveAnyObligation"),
          currentAnswers,
          selectedLoan
        )
      ] || "",
    ...currentAnswers,
  };

  $: visibleQuestions = schema?.pages?.[0]?.questions
    ? schema.pages[0].questions.filter((q) => isQuestionVisible(q, combinedAnswers))
    : [];

  // Clear loanType from currentAnswers completely if question hidden
  $: if (schema?.pages?.[0]?.questions) {
    const loanTypeQ = schema.pages[0].questions.find((q) => q.id === "q4_loanType");
    if (loanTypeQ) {
      const isVisible = isQuestionVisible(loanTypeQ, combinedAnswers);
      const answerKey = resolveBindsTo(loanTypeQ, combinedAnswers, selectedLoan);
      if (!isVisible && currentAnswers[answerKey]) {
        updateAnswer(loanTypeQ, null); // null triggers key removal
      }
    }
  }

  function onLoanChange(event) {
    selectedLoan = event.target.value || "";
    schema = preprocessSchemaBindings(formSchema, selectedLoan);
  }

  function getOptionValue(value) {
    if (typeof value === "object" && value.var) {
      return combinedAnswers[value.var] || "";
    }
    return value;
  }

  function updateAnswer(question, value) {
    if (question.id === "q1_loanName") {
      selectedLoan = value;
      schema = preprocessSchemaBindings(formSchema, selectedLoan);
    }
    const key = resolveBindsTo(question, combinedAnswers, selectedLoan);

    loanData.update((data) => {
      if (!data[selectedLoan]) data[selectedLoan] = {};
      if (value === "" || value === null || value === undefined) {
        delete data[selectedLoan][key];
      } else {
        data[selectedLoan][key] = value;
      }
      data.loanName = selectedLoan;
      return data;
    });
  }

  function isQuestionVisible(question, formData) {
    if (!question.showWhen) return true;
    return jsonLogic.apply(question.showWhen, formData);
  }
</script>

{#each visibleQuestions as question (question.id)}
  <div style="margin-bottom: 1rem;" class="flex flex-col gap-2">
    <label for={question.id}>{question.question}</label>

    {#if question.type === "radio"}
      {#each question.options as opt (opt.value)}
        <label>
          <input
            type="radio"
            name={question.id}
            value={typeof opt.value === "object" ? JSON.stringify(opt.value) : opt.value}
            checked={
              currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] ===
              getOptionValue(opt.value)
            }
            on:change={() => updateAnswer(question, getOptionValue(opt.value))}
          />
          {typeof opt.label === "object" && opt.label.var
            ? combinedAnswers[opt.label.var] || opt.label.var
            : opt.label}
        </label>
      {/each}
    {:else if question.type === "text"}
      <input
        id={question.id}
        type="text"
        value={currentAnswers[resolveBindsTo(question, combinedAnswers, selectedLoan)] || ""}
        on:input={(e) => updateAnswer(question, e.target.value)}
      />
    {/if}
  </div>
{/each}

<hr />

<div class="bg-black text-white p-4 max-h-64 overflow-auto">
  <h3>Debug: currentAnswers</h3>
  <pre>{JSON.stringify(currentAnswers, null, 2)}</pre>
  <h3>Debug: combinedAnswers</h3>
  <pre>{JSON.stringify(combinedAnswers, null, 2)}</pre>
</div>
