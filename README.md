# Simplified Job Hunting Automation Workflow

## Description

This n8n workflow automates job hunting by checking LinkedIn and Indeed (via Adzuna) every 4 hours for roles like 'AI Engineer', 'ML Engineer', 'Python Developer', 'Data Scientist', 'Cybersecurity Analyst', 'Security Engineer', or 'Threat Intelligence Specialist'. It focuses on remote or hybrid jobs in Europe or North America with salaries of at least €4,000/month, skipping frontend-heavy jobs (like website design) or unpaid internships. Jobs are scored based on mentions of AI, machine learning, cybersecurity, or Python. Matches are saved to a Notion table for tracking applications. Top jobs (score ≥80) send Telegram alerts, and daily email summaries arrive at 8:00 AM. The workflow also contacts recruiters or founders on LinkedIn, sending up to 10 personalized messages daily at 9:00 AM, avoiding duplicates within 30 days, and logs messages in a Notion table. It tracks application numbers, response times, and interview rates, saving stats in a Notion table. Optimized for speed, it caches LinkedIn searches and retries failed connections.

## Features

- **Job Search**: Checks LinkedIn and Indeed every 4 hours.
- **Filtering**: Keeps remote/hybrid jobs in Europe/North America, salaries ≥€4,000/month; skips frontend jobs and internships.
- **Scoring**: Scores jobs (+30 for AI/ML, +20 for Python, +30 for cybersecurity).
- **Storage**: Saves jobs to Notion with batch processing.
- **Notifications**: Telegram alerts for top jobs; daily email summaries.
- **LinkedIn Outreach**: Sends 10 personalized messages daily, avoids duplicates, uses caching.
- **Analytics**: Tracks applications, response times, and interview rates.
- **Error Handling**: Retries failed API calls, sends detailed error emails.

## What You Need

- **n8n Account**: Free account on n8n.io.
- **Accounts for Services**:
  - LinkedIn (jobs and messaging)
  - Adzuna (Indeed jobs)
  - Notion (storing jobs, messages, stats)
  - Telegram (notifications)
  - Email (e.g., Gmail for summaries and alerts)
- **Notion Tables**: For jobs, messages, stats, and caching LinkedIn searches.
- **Telegram Chat ID**: For alerts.
- **Email Address**: For summaries and error alerts.

## Step-by-Step Guide for Beginners

This guide is for people new to tech. Follow these steps to set up your job hunting tool. If stuck, see “Need Help?” or visit [n8n Community](https://community.n8n.io/).

### Step 1: Sign Up for n8n
1. Go to [n8n.io](https://n8n.io/) and click **Sign Up** for a free account.
2. Log in to see your n8n dashboard.
3. Check n8n’s website for a beginner’s video to understand the dashboard.

### Step 2: Add the Workflow
1. Copy the text from `job_hunting_workflow.json`.
2. In n8n, click **Workflows** (left menu), then **New** > **Import Workflow**.
3. Paste the text and click **Import**.
4. Name it “Job Hunt” and save. You’ll see connected boxes (like a flowchart).

### Step 3: Set Up Accounts
You need accounts for LinkedIn, Adzuna, Notion, Telegram, and email. Get “keys” (like passwords) for each.

- **LinkedIn**:
  1. Go to [LinkedIn Developer Portal](https://www.linkedin.com/developers/) and sign in.
  2. Click **Create App**, name it “Job Hunt”, link to your profile.
  3. Add “Sign In with LinkedIn” and “Jobs API” (you may need to apply).
  4. Copy **Client ID** and **Client Secret** from the app page.
  5. In n8n, go to **Credentials > Add Credential**, choose **HTTP Request OAuth2 API**, name it “LinkedIn API”, enter ID and Secret, and follow prompts to connect.
- **Adzuna (for Indeed)**:
  1. Visit [Adzuna Developer](https://developer.adzuna.com/) and sign up.
  2. Find your API key in your account settings.
  3. In n8n, add a credential for **HTTP Request API Key**, name it “Adzuna API”, paste the key.
- **Notion**:
  1. Go to [Notion Developers](https://www.notion.so/developers) and sign in.
  2. Click **Create New Integration**, name it “Job Hunt”, copy the **Internal Integration Token**.
  3. In n8n, add a credential for **Notion API**, name it “Notion API”, paste the token.
- **Telegram**:
  1. In Telegram, search for [BotFather](https://t.me/BotFather), send `/start`.
  2. Send `/newbot`, name it “JobHuntBot”, copy the **Bot Token**.
  3. In n8n, add a credential for **Telegram API**, name it “Telegram API”, paste the token.
  4. Message your bot (“Hi”), then use [GetIDs Bot](https://t.me/getidsbot) to get your **Chat ID**.
- **Email (Gmail)**:
  1. Use a Gmail account.
  2. Enable 2-factor authentication at [Google Account Security](https://myaccount.google.com/security).
  3. Create an **App Password**.
  4. In n8n, add a credential for **SMTP**, name it “SMTP Credentials”, enter your email, app password, SMTP host (`smtp.gmail.com`), and port (587).

### Step 4: Create Notion Tables
Make four tables in Notion for jobs, messages, stats, and caching:
1. In Notion, click **New Page**, choose **Table** > **New Database**.
2. Create tables: “Jobs”, “Messages”, “Stats”, and “Cache”.
3. Add these columns:

- **Jobs Table**:
  - Title (Title): Job title.
  - Company (Text): Company name.
  - Location (Text): Job location.
  - Salary (Number): Salary (€/year).
  - Score (Number): Job score.
  - Application Status (Select): “Not Applied”, “Applied”, “Interview”, “Responded”.
  - Application Date (Date): Application date.
  - Response Date (Date): Response date.
  - URL (URL): Job link.
  - Source (Text): “LinkedIn” or “Adzuna”.
  - Created Time (Created Time): Auto-added.
- **Messages Table**:
  - Name (Text): Contact’s name.
  - Company (Text): Company name.
  - LinkedIn Profile (URL): Contact’s LinkedIn URL.
  - Message Sent (Text): Message text.
  - Date Sent (Date): Message date.
  - Follow-Up Date (Date): Follow-up date.
  - Status (Select): “Awaiting Response”, “Responded”, “No Response”.
  - Created Time (Created Time): Auto-added.
- **Stats Table**:
  - Date (Date): Stats date.
  - Total Applications (Number): Application count.
  - Sources (Text): Job sources.
  - Average Response Time (Days) (Number): Response time.
  - Conversion Rate (%) (Number): Interview percentage.
- **Cache Table**:
  - Company (Text): Company name.
  - People (Text): LinkedIn search results.
  - Created Time (Created Time): Auto-added.

4. Share each table with your integration:
   - Open the table, click **Share** (top-right), select “Job Hunt” integration.
5. Get table IDs from the URL (e.g., `https://www.notion.so/your_workspace/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx?v=...`, ID is `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`).

### Step 5: Add Your Details
In n8n, open the workflow:
1. Double-click each box to edit:
   - In “Save to Notion”, “Fetch Daily Jobs”, “Fetch New Jobs”, “Fetch Analytics Data”, replace `{{your_notion_job_database_id}}` with Jobs table ID.
   - In “Check Outreach History”, “Log to Notion CRM”, replace `{{your_notion_crm_database_id}}` with Messages table ID.
   - In “Save Analytics”, replace `{{your_notion_analytics_database_id}}` with Stats table ID.
   - In “Check People Cache”, “Cache People”, replace `{{your_notion_cache_database_id}}` with Cache table ID.
   - In “Telegram Notification”, replace `{{your_telegram_chat_id}}` with your Chat ID.
   - In “Send Email Summary”, “Error Notification”, replace `{{your_email}}` with your email.
2. Save the workflow.

### Step 6: Test the Workflow
1. Open the workflow in n8n.
2. Click **Execute Node** on “LinkedIn Jobs API” and “Adzuna API (Indeed)” to check for jobs.
3. Run “Save to Notion” to see jobs in your Jobs table.
4. Run “Telegram Notification” to get a test message.
5. Run “Send Email Summary” to get an email.
6. Run “Fetch New Jobs” to “Log to Notion CRM” to test messaging (needs LinkedIn connection).
7. Run “Fetch Analytics Data” to “Save Analytics” to check stats.
8. If errors occur, check “Need Help?”.

### Step 7: Start the Workflow
1. In n8n, turn on the **Active** switch (top-right).
2. The workflow will:
   - Check jobs every 4 hours.
   - Send email summaries at 8:00 AM.
   - Send up to 10 LinkedIn messages at 9:00 AM.
   - Save stats daily at midnight.
3. Check Notion, Telegram, and email daily for updates.

## Setup for Advanced Users

### 1. Import Workflow
1. Copy JSON from `job_hunting_workflow.json`.
2. In n8n, go to **Workflows > New > Import Workflow**, paste, save as “Job Hunting Automation”.

### 2. Configure Credentials
In n8n, **Credentials > Add Credential**:
- **LinkedIn API (OAuth2)**:
  - Register at [LinkedIn Developer Portal](https://www.linkedin.com/developers/).
  - Enable `r_job_search`, `w_member_social`.
  - Add as “HTTP Request OAuth2 API” for “LinkedIn Jobs API”, “LinkedIn People Search”, “Send LinkedIn Message”.
- **Adzuna API (API Key)**:
  - Register at [Adzuna Developer](https://developer.adzuna.com/).
  - Add as “HTTP Request API Key” for “Adzuna API (Indeed)”.
- **Notion API**:
  - Create integration at [Notion Developers](https://www.notion.so/developers).
  - Add as “Notion API” for Notion nodes.
- **Telegram API**:
  - Get Bot Token from [BotFather](https://t.me/BotFather).
  - Add as “Telegram API” for “Telegram Notification”.
- **SMTP**:
  - Configure SMTP (e.g., Gmail: `smtp.gmail.com`, port 587).
  - Add as “SMTP” for “Send Email Summary”, “Error Notification”.

### 3. Set Up Notion Tables
Create four Notion databases, share with integration, get IDs from URLs:
- **Job Database**:
  - Title (Title), Company (Rich Text), Location (Rich Text), Salary (Number), Score (Number), Application Status (Select: Not Applied, Applied, Interview, Responded), Application Date (Date), Response Date (Date), URL (URL), Source (Rich Text), Created Time (Created Time).
- **CRM Database**:
  - Name (Rich Text), Company (Rich Text), LinkedIn Profile (URL), Message Sent (Rich Text), Date Sent (Date), Follow-Up Date (Date), Status (Select: Awaiting Response, Responded, No Response), Created Time (Created Time).
- **Analytics Database**:
  - Date (Date), Total Applications (Number), Sources (Rich Text), Average Response Time (Days) (Number), Conversion Rate (%) (Number).
- **Cache Database**:
  - Company (Rich Text), People (Rich Text), Created Time (Created Time).

Update nodes:
- `your_notion_job_database_id` in “Save to Notion”, “Fetch Daily Jobs”, “Fetch New Jobs”, “Fetch Analytics Data”.
- `your_notion_crm_database_id` in “Check Outreach History”, “Log to Notion CRM”.
- `your_notion_analytics_database_id` in “Save Analytics”.
- `your_notion_cache_database_id` in “Check People Cache”, “Cache People”.

### 4. Configure Notifications
- **Telegram**: Add Chat ID to “Telegram Notification”.
- **Email**: Add email to “Send Email Summary”, “Error Notification”.

### 5. Test Workflow
1. Test “LinkedIn Jobs API”, “Adzuna API (Indeed)” for job data.
2. Test “Filter Jobs”, “Score Jobs” for filtering/scoring.
3. Test “Save to Notion” for Job table entries.
4. Test “Telegram Notification”, “Send Email Summary”.
5. Test “Check People Cache” to “Log to Notion CRM” for outreach.
6. Test “Fetch Analytics Data” to “Save Analytics”.
7. Activate workflow.

## Need Help?
- **No Jobs**: Check API keys or tweak job titles/locations in “LinkedIn Jobs API”, “Adzuna API (Indeed)”.
- **Notion Issues**: Ensure tables are shared with integration, column names match.
- **No Telegram/Email**: Verify Chat ID, email, SMTP settings.
- **LinkedIn Messages Fail**: Check OAuth2 scopes, profile IDs.
- **Cache Issues**: Ensure Cache table has correct columns, clear old entries if needed.
- **Errors**: Check “Error Notification” emails for details.
- Visit [n8n Community](https://community.n8n.io/) or n8n’s help videos.

## Notes
- Replace placeholders (`{{your_notion_job_database_id}}`, `{{your_notion_crm_database_id}}`, `{{your_notion_analytics_database_id}}`, `{{your_notion_cache_database_id}}`, `{{your_telegram_chat_id}}`, `{{your_email}}`) before running.
- Monitor API limits (LinkedIn: 100 calls/day, Adzuna: 250 calls/day).
- Test one node at a time.
- Backup Notion tables.
# JobHuntAutomation
