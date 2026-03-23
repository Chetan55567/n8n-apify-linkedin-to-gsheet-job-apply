# LinkedIn Posts to Sheets + AI Job Apply (n8n + Apify)

This repository contains an n8n workflow that:

1. Searches LinkedIn posts using an Apify actor.
2. Stores posts in Google Sheets.
3. Uses an AI API to extract hiring/job details from each post.
4. Builds an email draft and sends job application emails via Gmail.
5. Writes applied jobs and parsed AI output back to Google Sheets.



## Prerequisites

- Running n8n instance (`1.0+` recommended)
- Apify account and token
- Google Cloud OAuth credentials configured in n8n for:
  - Google Sheets
  - Google Drive
  - Gmail
- AI provider endpoint compatible with OpenAI-style `/v1/chat/completions`

## Setup Steps

1. Import workflow in n8n:
   - Workflows -> Import from File -> select `linkedin_posts_to_sheets_workflow.json`

2. Create/connect credentials in n8n:
   - Google Sheets OAuth2
   - Google Drive OAuth2
   - Gmail OAuth2

3. Update placeholders in node parameters:
   - `{{APIFY_API_TOKEN}}`
   - `{{GOOGLE_SHEET_ID}}`
   - `{{AI_API_BASE_URL}}` (example: `https://api.openai.com/v1/chat/completions`)
   - `{{AI_API_KEY}}`
   - `{{DRIVE_FILE_ID}}`
   - `{{CANDIDATE_NAME}}`
   - `{{CANDIDATE_PHONE}}`
   - `{{CURRENT_COMPANY}}`

4. Re-select credential references in these nodes (important):
   - Google Sheets nodes
   - Google Drive `Download file`
   - Gmail `Send a message`

5. Prepare Google Sheets tabs used by the flow:
   - `posts`
   - `deduplicate`
   - `parsedaioutput`
   - `parsedaideduplicate`
   - `appliedemailjobs`

6. Review AI prompts:
   - `AI Extract Job Info`
   - `Email Preparation AI`

7. Optional (recommended for n8n OpenAI credits):
   - Instead of HTTP Request nodes, you can replace:
     - `AI Extract Job Info`
     - `Email Preparation AI`
   - Use the n8n `OpenAI` node to leverage free n8n OpenAI credits (if available on your n8n plan).

8. Run a manual test:
   - Execute `Manual Trigger`
   - Confirm rows are appended and email path works

## How to Swap HTTP Request to OpenAI Node

1. Add an `OpenAI` node after each of these points in flow:
   - Replace `AI Extract Job Info` HTTP Request node.
   - Replace `Email Preparation AI` HTTP Request node.
2. In each `OpenAI` node, set:
   - Resource: `Text`
   - Operation: `Message a Model`
   - Model: same model currently used in your HTTP JSON body (for example `Kimi`).
3. Move the existing prompt content from each HTTP node `messages` field into the OpenAI node `Messages` input.
4. Keep downstream parsing nodes unchanged (`ParsedJobInfo` and `Parse Email Output`), then test once with `Manual Trigger`.

## Workflow Notes

- The workflow avoids duplicate applications by checking `appliedemailjobs.post_url`.
- It only sends email when `apply_email` exists and looks valid (`contains @`).
- There are wait nodes to throttle downstream operations.

## Recommended Production Hardening

- Move all secrets to n8n credentials or environment variables.
- Add rate limiting/retry policy for AI and Apify requests.
- Add error notification node (Slack/Email) for failed executions.
- Enable execution logs with retention policy.

## Publish to GitHub Checklist

- [ ] Confirm all placeholders are still placeholders in JSON
- [ ] Confirm no personal data in prompts
- [x] MIT license file is included
- [ ] Add `.gitignore` for local n8n/export artifacts if needed

## License and Liability

This workflow is provided as-is for educational and automation purposes.

By using, modifying, or distributing this workflow, you agree that you are solely responsible for any damages, losses, account issues, legal claims, or compliance violations resulting from its use.

The workflow author and contributors provide no warranty and accept no liability.

## Disclaimer

Please follow LinkedIn, Apify, Google, and your AI provider terms of service and applicable local laws before running automation at scale.
