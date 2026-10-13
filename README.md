# Incident Response Playbook Runner

An automated security incident reporting system built with **n8n**. Employees describe an incident in plain language through a web form. An AI agent classifies it, the matching response steps are read from a Google Sheet playbook, the incident is logged, and the security team is alerted on Telegram. A second, scheduled workflow reminds the team about incidents that are still open.

> Training project (n8n automation bootcamp). Use test data only.

## The problem

When something suspicious happens (a phishing email, a lost laptop), employees often do not know what to do next. Reports arrive in different formats, and open incidents can be forgotten.

## How it works

1. **Report:** the employee fills a web form (name, email, what happened).
2. **Validate:** the workflow checks the input and returns a clear error page if it is invalid.
3. **Classify:** an AI agent (Google Gemini) picks one of five incident types and gives a one-sentence reason. When a report mentions a domain, the agent can call a tool that checks it with Google DNS.
4. **Playbook:** the response steps for that type are read from a Google Sheet. The AI never writes the steps.
5. **Log and alert:** the incident is added to the sheet and an alert is sent to the security team on Telegram.
6. **Result:** the employee sees the incident ID, the detected type and the steps to take now.

A second workflow (**Watchdog**) runs every hour, finds incidents that are not closed and older than one hour, and sends a Telegram reminder.

## Incident types

Phishing, Lost or Stolen Device, Malware Suspected, Account Compromise, Other.

`Other` is the safety net: if the AI is unsure or returns something invalid, the type becomes `Other` and the incident is marked `needs_review`.

## Tools

| Tool | Role |
|---|---|
| n8n Cloud | Runs both workflows |
| Webhook | Receives the report from the form |
| Google Gemini (AI Agent) | Classifies the incident |
| HTTP Request Tool (`dns.google`) | Checks that a domain exists |
| Google Sheets | Playbook steps and incident log |
| Telegram Bot | Alerts and reminders |
| Schedule Trigger | Hourly watchdog |

## Security design

- Input is validated in the browser and again in the workflow.
- API keys live only in n8n Credentials, never in the page or the code.
- Values shown on the result page are HTML-escaped (prevents XSS).
- The AI treats report text as untrusted data and never follows instructions inside it.

## Limits

- The DNS check only shows whether a domain exists. It is a hint, not a verdict on safety.
- The AI can misclassify, so people should review every alert.
- The playbook steps are general practice, not an official policy.

## Possible next steps

- Add a link reputation check (for example Google Safe Browsing).
- Add a new incident type by adding one row to the `playbooks` tab.
- Move the log from Google Sheets to a database.

## Setup

1. Import the two workflows from `workflows/` into n8n.
2. Create credentials in n8n for Google Gemini, Google Sheets and Telegram (keys stay in n8n).
3. Upload `Incident_Response_Sheet.xlsx` to Google Sheets (tabs: `playbooks`, `incidents`).
4. Set your Telegram chat ID and the sheet in the relevant nodes.
5. Publish both workflows.
6. In `incident_form.html`, set the form `action` to your n8n **Production** webhook URL and host the page (for example on Netlify).

## Screenshots

![Report form](screenshots/1_report_form.png)
![Result page (sample data)](screenshots/2_result_page_sample.png)
![Workflow](screenshots/3_workflow_diagram.png)
