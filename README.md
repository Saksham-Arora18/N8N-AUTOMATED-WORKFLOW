# CODE SUMMARIZER Workflow

**Automated AI-powered code analysis and documentation generator using n8n + Groq**

Send code snippets via email → Get AI-generated summaries + inline comments → Receive results via email. Built for developers, students, and technical teams.

---

## 🎯 Overview

This n8n workflow automates code analysis at scale. It reads code from incoming emails, leverages Groq's fast LLM inference to:
- **Generate concise code summaries** (what does this code do?)
- **Add inline comments** (explaining logic, functions, edge cases)
- **Merge results** and send everything back as a formatted email

---

## ✨ Key Features

- ⚡ **Fast processing** — Groq API delivers LLM responses in seconds
- 📧 **Email-triggered** — Send code via Gmail, get analysis back
- 🔄 **Parallel execution** — Summarization and comments happen simultaneously
- 🧠 **Smart analysis** — Groq's models understand code context and best practices
- 🛠️ **Fully customizable** — Adjust prompts, models, and output format

---

## 🔧 How It Works

### Workflow Steps

1. **Get Trigger (Gmail)** — Listens for incoming emails with code attachments/content
2. **Groq Chat Model (Summarization)** *(parallel)*— Summarizes the code 
3. **Groq Chat Model (Comments)** *(parallel)* — Generates inline code comments
4. **CODE_SUMMARY Node** — Extracts/formats summary output
5. **COMMENTS_ADDITION Node** — Extracts/formats comments output
6. **Merge** — Combines both results
7. **Aggregate** — Consolidates data
8. **Edit Fields** — Formats final output
9. **Basic LLM Chain** — Final processing (Converting to HTML)
10. **Send Message (Email)** — Returns formatted analysis back to user

**Key architectural choice:** Steps 2 & 3 run in parallel for speed.

---

## 📋 Prerequisites

### Required Accounts & Keys

1. **Gmail Account**
   - Gmail address where workflow receives emails
   - Google account with app passwords enabled (if 2FA is on)

2. **Groq API Account**
   - Sign up at [groq.com/groqcloud](https://console.groq.com)
   - Get your API key from the dashboard
   - Recommended model: `llama-3.3-70b-versatile` or `mixtral-8x7b-32768`

3. **n8n Workspace**
   - Self-hosted n8n or n8n cloud account
   - Access to the workflow editor

### Software/Tech Stack

- **n8n** (workflow automation)
- **Groq API** (LLM inference)
- **Gmail** (email trigger + output)
- No database required; stateless processing

---

## 🚀 Setup Instructions

### Step 1: Configure Gmail Trigger

1. Open the workflow editor
2. Click the **"Get Trigger" (Gmail Trigger)** node
3. Create a new Gmail credential:
   - Click **"Create new credential"**
   - Select **Gmail**
   - Authenticate with your Google account
   - Grant n8n permission to read emails
4. Set trigger conditions:
   - **From** — (optional) filter sender
   - **Subject** — e.g., `[CODE REVIEW]` or leave blank for all
   - **Mark as read** — enable to avoid duplicate processing

### Step 2: Configure Groq API Credentials

1. Click any **Groq Chat Model** node in the workflow
2. Create a new Groq credential:
   - Click **"Create new credential"**
   - Paste your Groq API key (from [console.groq.com](https://console.groq.com))
   - Name it `Groq API`
3. Repeat for all Groq Chat Model nodes (both summarization and comments nodes)

### Step 3: Customize Prompts (Optional)

Edit the LLM prompt nodes to match your style:

**Summarization prompt:**
```
Summarize this code in 2-3 sentences. Explain what it does, what problem it solves, and any key algorithms or patterns used.

CODE:
{{input}}
```

**Comments prompt:**
```
Add clear, concise inline comments to this code. Explain complex logic, function parameters, return values, and any non-obvious decisions. Keep comments brief.

CODE:
{{input}}
```

### Step 4: Set Email Output

1. Click the **"Send a message"** node
2. Configure Gmail output:
   - Select your Gmail credential
   - **To** — Use dynamic field: `{{$json.fromEmail}}` (replies to original sender)
   - **Subject** — `Re: Code Analysis - {{$json.filename}}`
   - **Body** — Format with summary + comments from merge output

### Step 5: Deploy & Test

1. Click **"Publish"** in top right
2. Send a test email with code snippet to your email
3. Check workflow executions log for debug info
4. Verify you receive the analysis email

---

## 📧 Usage

### Sending Code for Analysis

1. **Email format:**
   ```
   To: your-email@gmail.com
   Subject: [CODE REVIEW] analyze my function
   Body: Paste code snippet in email body OR attach .py/.js/.java file
   ```

2. **Example:**
   ```python
   def fibonacci(n):
       if n <= 1:
           return n
       return fibonacci(n-1) + fibonacci(n-2)
   ```

3. **Wait for response** — Usually 5-15 seconds depending on code length

### Response Email Format

You'll receive an email with:

```
Subject: Re: Code Analysis - [filename]

=== CODE SUMMARY ===
This function calculates Fibonacci numbers recursively...

=== INLINE COMMENTS ===
def fibonacci(n):  # Base case: handles n=0 and n=1
    if n <= 1:
        return n  # Return n directly
    # Recursively calculate F(n-1) + F(n-2)
    return fibonacci(n-1) + fibonacci(n-2)

=== ANALYSIS ===
[Additional insights from LLM Chain]
```

---

## 🎛️ Customization

### Change LLM Model

1. Click any **Groq Chat Model** node
2. Under **Model**, select from available options:
   - `llama-3.3-70b-versatile` (best for general code)
   - `mixtral-8x7b-32768` (fast, good for short snippets)
   - `llama-3.1-70b-versatile` (balanced)

### Adjust Temperature (Creativity)

- **Lower (0.3)** — Strict, deterministic comments
- **Higher (0.7)** — More creative explanations (good for learning)

Default: `0.5` (balanced)

### Support Multiple Languages

Edit prompts to specify language:
```
Analyze this JavaScript/Python/Java code and add comments...
```

### Filter Email Subjects

In Gmail Trigger node, set **Subject filter** to:
- `[CODE]` — Only process emails with `[CODE]` tag
- `code-review` — Only specific label

### Add Custom Processing

Insert nodes between **Aggregate** and **Send message** to:
- Store summaries in a database
- Post to Slack channel
- Save to Google Drive document
- Log to analytics service

---

## 🐛 Troubleshooting

### "Gmail authentication failed"
- **Solution:** Regenerate Google app password (Gmail security settings)
- Enable "Less secure app access" if not using 2FA
- Re-authenticate credential in n8n

### "Groq API error: Invalid API key"
- **Solution:** Copy API key again from [console.groq.com](https://console.groq.com)
- Check key isn't expired or revoked
- Ensure key is in Groq credential, not somewhere else

### "No email received back"
- **Check:** Executions log (click Executions tab)
- Look for errors in **"Send message"** node
- Verify **To** field is using correct dynamic variable
- Check Gmail spam folder

### "Workflow runs but no summary generated"
- **Check:** Groq Chat Model nodes have valid credential
- Verify code was extracted correctly from email
- Check prompt is well-formed (no broken variables)
- Test with shorter code snippet first

### "Rate limit exceeded"
- Groq has usage limits; wait a few minutes
- For high volume, reach out to Groq support for plan upgrade

---

## 📊 Performance & Limits

| Metric | Value |
|--------|-------|
| **Avg response time** | 5-15 seconds |
| **Max code length** | ~2000 lines (adjust based on model) |
| **Max emails/minute** | 10-60 (Groq plan dependent) |
| **Email processing** | Stateless (no database) |

---

## 🔐 Security Notes

- **API keys:** Store Groq key securely in n8n credential vault
- **Gmail access:** n8n only reads emails; doesn't delete originals
- **No data storage:** Code analyzed isn't permanently logged
- **Privacy:** Code content sent to Groq API; review Groq privacy policy

---

## 🚀 Next Steps & Enhancements

- [ ] Add Slack integration for team notifications
- [ ] Log results to Google Sheets for tracking
- [ ] Implement code quality scoring
- [ ] Add support for GitHub PR analysis
- [ ] Create Telegram bot version
- [ ] Multi-language support with language detection
- [ ] Add code complexity metrics (cyclomatic, LOC)

---

## 📞 Support & Questions

- **n8n docs:** [docs.n8n.io](https://docs.n8n.io)
- **Groq API docs:** [console.groq.com/docs](https://console.groq.com/docs)
- **n8n community:** [community.n8n.io](https://community.n8n.io)

---

## 📝 License

Free to use and modify for personal/team projects.

---

**Created:** 2026 | **Last Updated:** June 2026 | **Status:** Active & Maintained
