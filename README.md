# Gmail-to-Jira Ticket Automation Workflow

*n8n • Gmail API • Jira Cloud API • AI Agent Integration*

---

## Overview

This workflow automates **Jira ticket creation from Gmail**.

When a user sends or forwards an email with predefined keywords or structure, the automation automatically extracts the **issue title, description, assignee, and priority**, and creates a corresponding **Jira issue** with all required fields populated.

The workflow also sends a confirmation email reply to the user once the ticket is successfully created.

---

## Architecture

### Input

- Gmail message containing:
  - Issue title in subject
  - Description in body
  - Tags like `Assignee:`, `Priority:` (optional)

### Processing

- Gmail trigger detects the new email
- HTTP Request node connects to Jira API
- JavaScript nodes extract structured data
- AI Agent (optional) can summarize or normalize the ticket description
- Jira issue is created using REST API
- Confirmation email is sent back to the user

### Output

- Jira issue created with:
  - Title
  - Description
  - Assignee
  - Priority
  - Labels or tags
- Confirmation message sent to the user

---

## Workflow Diagram

```
Gmail Trigger → HTTP Request → JS Parsing → AI Agent (optional) → Jira Issue Create → Reply to Email
```

---

## Detailed Workflow Description

### 1. Gmail Trigger Node

- **Type:** Gmail → "Check for new emails"
- **Filters:**
  - Label: `jira` (optional)
  - Subject contains keywords like `#issue` or `#ticket`

This node captures new incoming or sent emails intended for Jira issue creation.

---

### 2. HTTP Request Node (Optional Pre-Validation)

- Validates connectivity or retrieves Jira project metadata.
- Endpoint example:

```http
GET https://your-domain.atlassian.net/rest/api/3/project
```

---

### 3. JavaScript Nodes (Code1, Code2)

These parse the Gmail email payload and extract:

- **Subject → Issue Title**
- **Body → Description**
- **Custom tags → Priority, Assignee**

**Example JavaScript logic:**

```javascript
const email = items[0].json;
const body = email.snippet || email.body;
const priority = body.match(/Priority:\s*(\w+)/i)?.[1] || "Medium";
const assignee = body.match(/Assignee:\s*(\w+)/i)?.[1] || "unassigned";

return [{
  json: {
    title: email.subject || "No Subject",
    description: body,
    priority,
    assignee
  }
}];
```

---

### 4. AI Agent Node (Optional)

Enhances ticket clarity using OpenAI or Azure OpenAI:

- Summarizes lengthy email bodies
- Converts informal text into structured Jira-ready description
- Detects urgency or missing information

**Example prompt:**

```
Rewrite the following email as a clear Jira issue description.
Identify the main problem, impact, and any missing details.
```

---

### 5. Jira Issue Creation Node

- **Method:** POST
- **Endpoint:**

```http
POST https://your-domain.atlassian.net/rest/api/3/issue
```

**Request Body Example:**

```json
{
  "fields": {
    "project": { "key": "SUPPORT" },
    "summary": "{{ $json.title }}",
    "description": "{{ $json.description }}",
    "assignee": { "name": "{{ $json.assignee }}" },
    "priority": { "name": "{{ $json.priority }}" },
    "issuetype": { "name": "Task" }
  }
}
```

**Headers:**

```
Authorization: Basic <base64_email:api_token>
Content-Type: application/json
```

---

### 6. Merge & Reply to Gmail Node

After successful ticket creation:

- Jira response is merged with the original email info
- A reply email is sent confirming ticket creation

**Example response:**

```
Subject: ✅ Jira Ticket Created: SUPPORT-123

Body: Your issue has been logged successfully. View it here:
https://your-domain.atlassian.net/browse/SUPPORT-123
```

---

## Example Gmail Input

**Subject:** `[Issue] Unable to log in to dashboard`

**Body:**

```
User reports login failure on staging environment.
Assignee: JohnDoe
Priority: High
```

---

## Expected Jira Output

| Field       | Value                                              |
|-------------|----------------------------------------------------|
| Project     | SUPPORT                                            |
| Summary     | Unable to log in to dashboard                      |
| Description | User reports login failure on staging environment. |
| Assignee    | JohnDoe                                            |
| Priority    | High                                               |
| Issue Type  | Task                                               |

---

## Business Impact

- ✅ Eliminates manual ticket entry
- ✅ Standardizes issue capture from Gmail
- ✅ Integrates seamlessly with existing Jira Cloud setup
- ✅ Saves 3–5 minutes per issue creation on average

---

## Technology Stack

- **n8n** – Workflow automation engine
- **Gmail API** – Email trigger and reply
- **Jira Cloud REST API** – Ticket creation
- **JavaScript Nodes** – Data parsing
- **AI Agent (OpenAI)** – Natural language enhancement

---

## Setup Instructions

### Prerequisites

1. n8n instance (cloud or self-hosted)
2. Gmail account with API access enabled
3. Jira Cloud workspace with API token
4. OpenAI API key (optional, for AI enhancement)

### Configuration Steps

1. **Gmail Authentication**
   - Enable Gmail API in Google Cloud Console
   - Create OAuth2 credentials
   - Add credentials to n8n

2. **Jira Authentication**
   - Generate API token from Atlassian account settings
   - Create base64 encoded string: `email:api_token`
   - Store in n8n credentials

3. **Workflow Import**
   - Import workflow JSON into n8n
   - Configure all credential fields
   - Test with sample email

4. **Email Template Setup**
   - Create Gmail label "jira" (optional)
   - Define subject line format
   - Document tag format for users

---

## Troubleshooting

### Common Issues

**Issue:** Jira API returns 401 Unauthorized
- **Solution:** Verify API token is correct and base64 encoded properly

**Issue:** Email not triggering workflow
- **Solution:** Check Gmail label filters and n8n trigger configuration

**Issue:** Missing assignee in created ticket
- **Solution:** Ensure assignee username matches Jira user exactly

**Issue:** AI Agent timeout
- **Solution:** Increase timeout setting or simplify prompt

---
**Last Updated:** November 2025
