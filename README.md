# Herald: Personalized Bulk Email Workflow (n8n)

Herald is an n8n workflow that sends personalized bulk emails with a human in the loop. AI writes **one** email for your whole list, you review and approve it, and then Herald sends it to every contact. Only the recipient's name changes in each email.

It is built to replace the slow, repetitive copy-and-paste of sending the same email to many people.

---

## How it works

Think of it as an assembly line. Each step is a node in n8n.

1. **Form:** You fill in a short form with the purpose of the email, the tone, the sender name, and where your contact list is.
2. **Read contacts:** Herald reads your Google Sheet and skips empty, invalid, or already-sent rows.
3. **AI draft:** An AI model writes one email body (with a `{{name}}` placeholder) and a subject line.
4. **Human approval:** The draft is sent to you for review. Nothing goes out until you approve. If you reject it, you can give feedback and the AI redrafts (up to 3 rounds).
5. **Personalize and loop:** Herald goes through contacts one at a time and swaps `{{name}}` for each person's name.
6. **Attachments:** Optionally attaches one file for everyone, or a different file per person.
7. **Send:** Emails go out through SMTP, so it works with Gmail, Yahoo, Proton, Outlook, and company mail servers.
8. **Pacing:** A short random pause between sends helps avoid spam flags, and a daily cap protects your sending limit.
9. **Logging:** Each row in your sheet is marked `sent` or `failed`, and you get a summary at the end.

---

## What you need

- An [n8n](https://n8n.io) account (n8n Cloud, or self-hosted)
- A Google Sheet with your contacts
- An email account with SMTP access
- An AI model API key (Anthropic Claude or OpenAI)
- A channel for approval messages (Gmail, Telegram, or Slack)
- Google Drive access (only needed for "different file per person" attachments)

---

## Setup

### 1. Import the workflow
1. In n8n, create a new workflow.
2. Click the **three dots (⋯)** at the top right and choose **Import from file**.
3. Select `herald-workflow.json` from this repository.

### 2. Add your credentials
Open each node showing a warning and connect your own credentials. This repository does not contain any passwords or keys.

| Node | Credential needed |
|---|---|
| Read Contacts / Update Status | Google Sheets |
| Draft Email with AI | Anthropic or OpenAI API key |
| Wait for Approval | Gmail, Telegram, or Slack |
| Send Email | SMTP (see below) |
| Fetch Attachment (optional) | Google Drive |

### 3. SMTP settings (Gmail example)

| Field | Value |
|---|---|
| Host | `smtp.gmail.com` |
| Port | `465` |
| User | your full Gmail address |
| Password | a Gmail **app password** (not your normal password) |
| SSL/TLS | On |
| Client Host Name | leave empty |

Create an app password here: https://myaccount.google.com/apppasswords
(2-Step Verification must be switched on for your Google account first.)

Other providers use different hosts. Check your provider's SMTP settings page, or your hosting provider's email setup page for company email.

### 4. Prepare your contact sheet

Create a Google Sheet with these columns:

| name | email | status | attachment_file |
|---|---|---|---|
| Ada | ada@example.com | | |
| Tunde | tunde@example.com | | |

- Leave `status` empty. Herald fills it with `sent` or `failed`.
- `attachment_file` is only used in "different file per person" mode. Put the file name from your Google Drive there.

A sample file is in `sample-contacts.csv`.

---

## Running Herald

1. **Test first.** Turn on test mode so only the first 3 contacts receive the email, and use addresses you own.
2. Open the form URL from the Form Trigger node and fill it in.
3. Review the AI draft when it arrives and click **Approve**.
4. Check your test inboxes (and spam folder) for the email, correct name, and attachment.
5. Check that the `status` column in your sheet changed to `sent`.
6. When everything looks right, turn off test mode and run it on your full list.

---

## Attachment modes

- **No attachment:** emails are sent without a file.
- **Same file for everyone:** upload one file in the form and it is attached to every email. This is the most common case.
- **Different file per person:** Herald fetches the file named in the `attachment_file` column from Google Drive. If a file is missing, that person is skipped and the error is logged.

---

## Limits and good practice

- **Gmail sends about 200 emails per day.** The daily cap setting (default 200) stops the run and marks the rest as `pending`.
- Always test with a few addresses you control before a real send.
- Only email people who expect to hear from you.
- Never commit real contact lists, app passwords, or API keys to this repository.

---

## Troubleshooting

| Problem | What to check |
|---|---|
| SMTP login fails | Use an app password, not your normal password. Check host and port. |
| Name shows as `{{name}}` in the received email | The personalization step is not replacing the placeholder. Check the node's output for the `name` field. |
| Emails land in spam | Slow down the pacing, use a real sender name, and avoid spammy wording. |
| Same person emailed twice | Make sure the status update step writes `sent` back to the correct row. |
| Approval never arrives | Check the approval channel credential and that the node is connected. |

Tip: after a run, click any node in n8n to see exactly what data went in and what came out. Most problems show up there.

---

## Repository structure

```
herald-n8n-workflow/
├── README.md
├── herald-workflow.json
├── sample-contacts.csv
└── prompts/
    └── build-prompt.md
```

---

## Roadmap

- One-click Google and Microsoft sign-in instead of SMTP
- Support for email sending services for higher volume
- A simple front-end app for non-technical users
- Open rate and delivery reporting

---

## Author

Built by [Your name]. Feel free to open an issue or suggestion.
