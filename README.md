# SolarEase Outbound AI Sales Caller

An n8n workflow for an AI-powered outbound sales caller for SolarEase Solutions, a residential solar installation company.

This project was built for the **AI Agent Arena hackathon**, held on **05/11/2025**, for Round 1: Outbound AI Sales Caller.

## What It Does

The workflow:

1. Reads leads from a Google Sheet.
2. Filters for leads with a phone number whose `call_status` is blank, `pending`, or `new`.
3. Builds a personalized Vapi outbound call using the lead's first name and lead source.
4. Runs a concise solar sales conversation with FAQ and objection handling.
5. Asks and records the required qualification data in separate Google Sheets columns.
6. Marks the lead as `calling`, waits 90 seconds between calls, and records call failures.
7. Receives Vapi end-of-call events through a webhook.
8. Normalizes the result, logs the call data to Google Sheets, and sends an outcome-specific follow-up email through Gmail.

## Company Information Used by the Agent

- **Company:** SolarEase Solutions
- **Address:** 123 GreenTech Avenue, San Diego, CA 92101
- **Hours:** Monday to Friday, 9:00 AM to 6:00 PM Pacific Time
- **Estimated price range:** $10,000 to $25,000, depending on system size and site
- **Offer:** Lower utility bills, flexible financing through cash, loan, or lease, and expert installation

The agent is instructed to use only these facts. Questions outside this information are handed off to a solar consultant during the follow-up.

## Conversation Flow

The Vapi assistant:

- Opens professionally and gives a short solar value proposition.
- Asks one question at a time and acknowledges each answer.
- Handles the expected objections: source of the phone number, being busy, and not being interested.
- Identifies itself as the SolarEase AI assistant when asked.
- Requests a preferred follow-up time when the lead is interested or asks for a callback.
- Ends politely and returns structured call data for the n8n workflow.

## Qualification Data

Each answer is written to its own Google Sheets column:

`roof_type`, `avg_monthly_bill`, `home_size_sqft`, `daytime_occupancy`, `heavy_appliances_count`, `ceiling_fans_count`, `utility_provider`, `peak_tariff`, `shading_concerns`, `preferred_financing`, and `budget_expectation`.

The workflow also records `outcome`, `callback_time`, `call_summary`, `call_duration`, `ended_reason`, `call_status`, and `last_called_at`.

## Outcomes and Follow-up Emails

The workflow supports these final outcomes:

- `Interested`
- `Not Interested`
- `Callback Requested`
- `No Response/Hang-up`

For a lead with an email address, the outcome router sends one of the following Gmail messages:

- **No response or hang-up:** `We Tried Reaching You - SolarEase Solutions`
- **Not interested:** `Thank You for Your Time - SolarEase Solutions`
- **Interested:** `Thanks for Your Interest - Next Steps with SolarEase`

Unanswered calls, voicemail, connection failures, calls shorter than ten seconds, and invalid model outcomes are safely classified as `No Response/Hang-up`.

## Files

- `SolarEase - Outbound AI Sales Caller.json` - n8n workflow export.
- `AI Agents Arena Round 1 (1).pdf` - hackathon problem statement.

## Setup

### 1. Import the workflow

1. Open n8n.
2. Import `SolarEase - Outbound AI Sales Caller.json`.
3. Configure the Google Sheets, Vapi, and Gmail credentials on the relevant nodes.

### 2. Configure the call settings

In **Call Settings**, replace the Vapi phone number placeholder and confirm the results webhook URL. The default assistant name is `Alex`.

In **Get Leads**, **Mark Calling**, **Mark Call Failed**, and **Log To Sheet**, replace `YOUR_SHEET_ID` with the Google Sheet ID and use the `Leads` sheet.

### 3. Prepare the `Leads` sheet

Create a header row containing the input fields `row_number`, `first_name`, `phone`, `email`, `lead_source`, and `call_status`, plus the qualification and result columns listed above. Each lead should have a unique `row_number`.

### 4. Configure Vapi

Set the Vapi authentication on **Vapi Outbound Call** and configure the Vapi phone number referenced by `vapi_phone_number_id`. Configure Vapi to post end-of-call events to the n8n production webhook:

`/webhook/solarease-vapi-eoc`

The call request includes the assistant instructions, company facts, objection handling, metadata, and a structured qualification schema.

### 5. Configure Gmail

Connect Gmail credentials to the three email nodes. Emails are sent only when the lead has an email address and the outcome matches one of the configured routes.

## Running the Workflow

Use **Manual Trigger** to start a batch. The workflow processes eligible leads one at a time, updates each lead before waiting 90 seconds, and continues through the batch. Vapi calls send their final results asynchronously to the webhook branch.

Before using real leads, test with a small sheet and verify that:

- The Vapi call is created successfully.
- The webhook receives an end-of-call event.
- Every qualification answer maps to the correct column.
- The final outcome and email route are correct.
- Failed calls are marked `call_failed` and do not block the remaining batch.

## Hackathon Deliverables Covered

- Working outbound AI sales caller workflow.
- Structured Google Sheets logging with separate qualification columns.
- Outcome-based follow-up emails for no response or hang-up, not interested, and interested leads.

## Privacy and Usage

Use this workflow only with leads and contact data for which the appropriate consent and calling permissions are available. Review local telemarketing, recording, privacy, and email requirements before production use.
