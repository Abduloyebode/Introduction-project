# Step 6: Front-end design and UX judgement

## Preface
Front end design and judgement is not something AI coding agents well natively today and is a big differentiator between "AI-slop" and production grade quality apps. Please spend 1-2 hours reading about https://impeccable.style/ on the website and the workflows. This step will requie you to use your judgement a lot and having a basic understanding of how creating polished designs with AI will help you a lot.

## Instructions

Improve the application’s interface using Impeccable and your own design judgement.

Your task is to:

1. Install Impeccable in the project:

```bash
npx impeccable install
```

2. Run `/impeccable init` and review the generated `PRODUCT.md` and `DESIGN.md`.
3. Audit the existing application and identify the most important usability and visual problems.
4. Improve the main workflow list, document-processing flow and processing-result page.
5. Use Impeccable to explore alternatives, but choose and refine the final direction yourself.
6. Ensure the interface has:

   * Clear hierarchy
   * Consistent spacing and typography
   * Useful loading, empty, error and success states
   * Responsive layouts
   * Accessible forms and controls
7. Avoid changing backend behaviour or adding unrelated features.
8. Submit the changes through a focused pull request containing:

   * The main design problems identified
   * Before and after screenshots
   * The alternatives considered
   * Why the final design was chosen

## Success criteria

The task is complete when:

* Impeccable is installed and its design context is committed.
* The main workflows are visually consistent and easy to understand.
* Important actions and system states are clear.
* The interface works well on desktop and mobile.
* Accessibility basics are preserved.
* The final design feels intentional rather than like an unedited AI-generated template.
* The developer can explain their decisions without relying only on Impeccable’s suggestions.
* The pull request includes useful before and after evidence.
* Existing functionality still works and available checks pass.
