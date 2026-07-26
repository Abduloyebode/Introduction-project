# Step 3: AI document extraction

## Instructions

Add an AI-powered document-processing feature.

Your task is to:

1. Allow authenticated users to upload a text-based PDF.
2. Send the extracted text to an LLM using structured output. (OpenAI, see new 1password link for API key)
3. Extract:

   * Document title
   * Summary
   * Important dates
   * Key obligations or action items
   * Suggested risk level
4. Save the original file metadata and extracted result.
5. Display the result clearly in the application.
6. Handle unsupported files, invalid AI responses and API failures.
7. Document required environment variables and expected AI costs.
8. Submit the work through a focused pull request and deploy it after approval.

Do not send secrets, sensitive documents or personal data to the AI provider during development.

## Success criteria

The task is complete when:

* A valid PDF can be uploaded and processed.
* AI output is validated before being saved.
* Extracted information is displayed clearly.
* Failed or malformed responses produce useful errors.
* Uploaded files are handled safely.
* API keys remain server-side and are not committed.
* Basic tests cover successful and failed processing.
* Required configuration and limitations are documented.
* The application builds and deploys successfully.