---
name: english-check
version: 1.0.0
description: Check an English sentence for grammar errors, explain the fix, classify each error, and save it via the english-practice MCP server. Invoke with /english-tutor-plugin:english-check <sentence>.
argument-hint: <English sentence to check>
allowed-tools:
  - mcp__english-practice__list_error_types
  - mcp__english-practice__save_sentence
---

# english-check

The user wrote an English sentence (given as the argument, or in the message if no argument).
Check it, classify every mistake, save the result through the `english-practice` MCP server,
then reply in the fixed format below. Follow the steps in order. Do not skip step 1.

## Steps

1. **Call `list_error_types` first.** You need the existing category names before classifying.

2. **Check the sentence.** For each mistake record:
   - `error_snippet`: the exact wrong fragment copied from the original sentence.
   - `suggestion`: what it should be.
   - `error_type`: a category name, chosen by these rules:
     - **Reuse an existing name** from step 1 whenever one fits, even if you would have phrased it differently. Never create a synonym of an existing type (`verb tense` when `tense` exists, `vocabulary` when `word choice` exists).
     - Only when nothing fits, create a new name: lowercase, short, singular noun phrase, no punctuation except hyphens or spaces. Also pass `description` (one line) so the category is documented.
     - Suggested starter names when the list is empty or nothing fits:
       `tense`, `article`, `preposition`, `subject-verb agreement`, `word order`, `word choice`, `plural`, `spelling`, `capitalization`, `punctuation`, `missing word`, `extra word`, `pronoun`, `comparative`, `countable noun`.
   - One mistake = one error item. A sentence can have several.
   - Only flag real errors. Do not rewrite style, register, or wording that is already acceptable.
   - Decide the full `corrected_text`: the whole sentence with every fix applied, nothing else changed.

3. **Call `save_sentence`** with:
   - `original_text`: the sentence exactly as the user wrote it.
   - `corrected_text`: from step 2. If the sentence is already correct, identical to `original_text`.
   - `explanation`: one or two sentences in English explaining the fixes. Empty string if no errors.
   - `source`: `skill`.
   - `errors`: the list from step 2. **A correct sentence is still saved, with an empty `errors` list**, so that correct-rate statistics are possible later.

4. **Reply** in exactly this format, in 繁體中文 for the labels and 說明, keeping the English sentences as-is:

```
原句：<original_text>
修正：<corrected_text>
說明：<one or two sentences in 繁體中文 explaining why; for a correct sentence write「沒有錯誤」>
錯誤：
- [<error_type>] <error_snippet> → <suggestion>
```

   - If there are no errors, the 錯誤 section is a single line: `錯誤：無`.
   - After the block, add one short line: `已儲存（id <sentence_id>）`.
   - Nothing else. No preamble, no extra tips.

## Edge cases

- The argument is empty: ask the user for the sentence and stop. Do not call any tool.
- The input is clearly not an English sentence (code, Chinese, a URL, a single word): say so in one line and stop. Do not save.
- Multiple sentences: treat them as one record. `original_text` is the whole input.
- `save_sentence` returns an error: show the error message as-is and tell the user the backend may not be running (`docker compose up -d`). Still show the correction.
