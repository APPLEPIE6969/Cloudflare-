# Ultimate Guide to Setting Up Cloudflare Workers for Your Free 24/7 Nanobot Agent AI  
**Creates Files & Code • Builds Entire Apps • Own HF Storage Buckets • Own GitHub • Own Kaggle • R2 Instant Serving • D1 Relational DB • No Credit Card Ever**

**Last updated:** April 2026  
**Target audience:** Builders of autonomous AI coding agents that respond to messages, plan/generate code, build full apps, and ship artifacts across platforms.

## Table of Contents
- [1. Why This Stack for Your Full Agent?](#1-why-this-stack-for-your-full-agent)
- [2. Free Tier Limits (All Confirmed 2026)](#2-free-tier-limits-all-confirmed-2026)
- [3. Prerequisites](#3-prerequisites)
- [4. Creating Your Cloudflare Account (No CC)](#4-creating-your-cloudflare-account-no-cc)
- [5. Quick Start: Dashboard-Only Method](#5-quick-start-dashboard-only-method)
- [6. Pro Setup: Wrangler CLI + Local Dev](#6-pro-setup-wrangler-cli--local-dev)
- [7. Your First Worker – Hello World](#7-your-first-worker--hello-world)
- [8. Message-Triggered Webhook Basics (Telegram Example)](#8-message-triggered-webhook-basics-telegram-example)
- [9. Adding AI Smarts + Tool-Calling LLM](#9-adding-ai-smarts--tool-calling-llm)
- [10. Persistence: KV + Durable Objects for Agent Memory](#10-persistence-kv--durable-objects-for-agent-memory)
- [11. R2 for Temporary File/Code Generation & App Serving](#11-r2-for-temporary-filecode-generation--app-serving)
- [12. Adding D1 Relational Database (Structured Storage)](#12-adding-d1-relational-database-structured-storage)
- [13. The Agent’s Own Accounts Setup (HF Buckets + GitHub + Kaggle)](#13-the-agents-own-accounts-setup-hf-buckets--github--kaggle)
- [14. Tool Functions: Upload to HF Storage Buckets](#14-tool-functions-upload-to-hf-storage-buckets)
- [15. Tool Functions: Push to GitHub](#15-tool-functions-push-to-github)
- [16. Tool Functions: Upload to Kaggle](#16-tool-functions-upload-to-kaggle)
- [17. Full Agent Flow – Example “Build a Full App”](#17-full-agent-flow--example-build-a-full-app)
- [18. Advanced: Agents SDK + Multi-Step Reasoning](#18-advanced-agents-sdk--multi-step-reasoning)
- [19. Custom Domain + Professional Replies](#19-custom-domain--professional-replies)
- [20. Local Dev, Testing & Logs](#20-local-dev-testing--logs)
- [21. Troubleshooting](#21-troubleshooting)
- [22. Scaling & Upgrade Path](#22-scaling--upgrade-path)
- [23. Resources](#23-resources)

## 1. Why This Stack for Your Full Agent?
Cloudflare Workers powers a truly free, on-demand, globally distributed backend. Your nanobot wakes instantly on incoming messages (webhook), uses an external LLM for reasoning/tool-calling, generates code/files, builds complete apps, then stores them across:
- **R2** — instant public download/preview URLs for generated apps (zero egress fees).
- **D1** — structured metadata/logs (project history, links, timestamps).
- **HF Storage Buckets** — mutable artifacts/checkpoints/datasets.
- **GitHub** — version-controlled repos the user can fork.
- **Kaggle** — published datasets/notebooks.

Everything stays under the **10 ms CPU limit** per request because heavy work (LLM calls, uploads) is pure I/O wait time. No idle costs, no credit card, perfect for your always-ready coding agent.

## 2. Free Tier Limits (All Confirmed 2026)
| Service                  | Key Free Limits (2026)                               | Notes for Your Agent                            |
|--------------------------|------------------------------------------------------|-------------------------------------------------|
| Workers                  | 100,000 requests/day, **10 ms CPU per request**, 128 MB memory | Orchestration stays safe under 8 ms            |
| D1                       | 10 DBs/account, 500 MB per DB, 5 GB total storage, 5M rows read/day, 100k rows written/day | Project metadata & logs                        |
| R2                       | 10 GB-month storage, 1M Class A ops/month, 10M Class B ops/month, **free egress** | App files, zips, instant serving               |
| KV                       | 1 GB storage, generous daily reads/writes            | Fast session data                              |
| Durable Objects          | Limited classes/storage on free (realistic for agents) | Per-chat stateful memory                       |
| HF Storage Buckets       | Best-effort + up to ~100 GB (account-dependent)      | Mutable ML artifacts                           |
| GitHub                   | 5,000 API calls/hour per token                       | Repo & file operations                         |
| Kaggle                   | Unlimited dataset/notebook uploads (free account)    | Sharing data                                   |

**CPU Tip**: Only your JavaScript execution counts toward 10 ms. `fetch()` to LLM, R2, D1, HF, GitHub, or Kaggle is free I/O.

## 3. Prerequisites
- Modern browser.
- Node.js 18+ installed.
- Git (optional but recommended).
- Telegram Bot Token (create via @BotFather on Telegram).
- API key for your preferred LLM (Grok, OpenAI, etc.).
- Free accounts on Hugging Face, GitHub, and Kaggle.
- Basic JavaScript knowledge (TypeScript is supported).

## 4. Creating Your Cloudflare Account (No CC)
1. Go to https://dash.cloudflare.com/sign-up/workers-and-pages.
2. Sign up with your email and a strong password.
3. Verify your email address.
4. You are automatically on the **Free plan** — no payment method is asked for Workers, D1, R2, KV, or Durable Objects.

## 5. Quick Start: Dashboard-Only Method
1. In the dashboard, go to **Workers & Pages** (left sidebar).
2. Click **Create application** → **Create Worker**.
3. Give it a name like `nanobot-ai-agent`.
4. Click **Deploy**.
5. Click **Edit code** to open the in-browser editor.
6. Paste example code from later sections and click **Save and deploy**.
7. Copy your Worker URL (e.g., `https://nanobot-ai-agent.yourname.workers.dev`).

Ideal for quick tests.

## 6. Pro Setup: Wrangler CLI + Local Dev (Recommended)
```bash
# Install Wrangler globally
npm install -g wrangler

# Login (opens your browser)
wrangler login

# Create a new project
npx create-cloudflare@latest nanobot-ai-agent --type=worker
cd nanobot-ai-agent

# Run locally with hot reload (supports D1, R2, KV simulation)
wrangler dev
```

For an even better base, use the official Agents starter:
```bash
npx create-cloudflare@latest --template cloudflare/agents-starter
```

## 7. Your First Worker – Hello World
In `src/index.js` (or `.ts`):

```js
export default {
  async fetch(request, env) {
    return new Response("🟢 Nanobot AI Agent is alive and ready to build apps! 🚀\nSend me a message to get started.", {
      headers: { "Content-Type": "text/plain" },
    });
  },
};
```

Deploy it:
```bash
wrangler deploy
```

Open the Worker URL in your browser to verify.

## 8. Message-Triggered Webhook Basics (Telegram Example)
Create your bot with @BotFather and get the token. Set it as a secret later.

Basic webhook handler (add to `src/index.js`):

```js
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    if (request.method === "POST" && url.pathname === "/webhook") {
      const update = await request.json();
      return handleTelegramUpdate(update, env);
    }
    return new Response("Nanobot ready!");
  },
};

async function handleTelegramUpdate(update, env) {
  const chatId = update.message?.chat?.id;
  if (!chatId) return new Response("OK");

  const text = update.message.text || "No text";
  // TODO: Call LLM + tools here

  await fetch(`https://api.telegram.org/bot${env.TELEGRAM_TOKEN}/sendMessage`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      chat_id: chatId,
      text: `🤖 Received: ${text}\nProcessing with tools...`,
    }),
  });

  return new Response("OK");
}
```

Set the webhook once (replace `<TOKEN>` and your Worker URL):
```
https://api.telegram.org/bot<TOKEN>/setWebhook?url=https://your-worker.workers.dev/webhook
```

## 9. Adding AI Smarts + Tool-Calling LLM
Store your LLM key securely:
```bash
wrangler secret put LLM_API_KEY
wrangler secret put TELEGRAM_TOKEN
```

Example Grok call (adapt for OpenAI/Anthropic):

```js
async function callLLMWithTools(prompt, history, env) {
  const response = await fetch("https://api.x.ai/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${env.LLM_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "grok-beta",
      messages: [{ role: "system", content: "You are Nanobot, an autonomous coding agent with tools for HF, GitHub, Kaggle, R2, D1." }, ...history, { role: "user", content: prompt }],
      tools: [ /* define your tool schemas here */ ],
    }),
  });
  const data = await response.json();
  return data.choices[0].message; // handle tool_calls
}
```

**System prompt tip**: Instruct the LLM to output structured tool calls when it needs to generate files, upload, push, etc.

## 10. Persistence: KV + Durable Objects for Agent Memory
KV for simple key-value (conversation history).

Durable Objects (via Agents SDK) for stateful per-chat agents that remember across messages.

Bind in `wrangler.toml` and use `env.MY_KV.put/get`.

## 11. R2 for Temporary File/Code Generation & App Serving
R2 gives instant public URLs for generated apps.

One-time: Dashboard → R2 → Create bucket `nanobot-apps`.

Add to `wrangler.toml`:
```toml
[[r2_buckets]]
binding = "MY_R2"
bucket_name = "nanobot-apps"
```

Upload example:
```js
async function uploadAppToR2(files, projectName, env) {
  const prefix = `apps/${Date.now()}-${projectName}/`;
  for (const file of files) {  // [{path: "index.html", content: "..."}]
    await env.MY_R2.put(prefix + file.path, file.content, {
      httpMetadata: { contentType: getMimeType(file.path) },
    });
  }
  return `https://pub-${env.MY_R2.bucket_name}.r2.dev/${prefix}`;
}
```

## 12. Adding D1 Relational Database (Structured Storage)
For logging generated apps, metadata, history.

One-time: Dashboard → D1 → Create database `nanobot-db`.

Add to `wrangler.toml`:
```toml
[[d1_databases]]
binding = "DB"
database_name = "nanobot-db"
database_id = "<your-db-id-from-dashboard>"
```

Example schema & usage:
```js
// Init table (run once)
await env.DB.exec(`
  CREATE TABLE IF NOT EXISTS generated_apps (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    chat_id TEXT,
    project_name TEXT,
    github_url TEXT,
    r2_url TEXT,
    hf_bucket TEXT,
    kaggle_url TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  );
`);

async function logGeneratedApp(chatId, projectName, urls, env) {
  await env.DB.prepare(`
    INSERT INTO generated_apps (chat_id, project_name, github_url, r2_url, hf_bucket, kaggle_url)
    VALUES (?, ?, ?, ?, ?, ?)
  `).bind(chatId, projectName, urls.github, urls.r2, urls.hf, urls.kaggle).run();
}
```

Query later for history.

## 13. The Agent’s Own Accounts Setup (HF Buckets + GitHub + Kaggle)
All free, no credit card.

**Hugging Face**:
- Create account at huggingface.co.
- Go to settings/storage → Create bucket (e.g., `nanobot-artifacts`).
- Settings → Access Tokens → New write token.

**GitHub**:
- New account (e.g., nanobot-ai-agent).
- Settings → Developer settings → Personal access tokens (classic) → Generate with `repo` scope.

**Kaggle**:
- New account.
- Account page → Create API Token (use username + key).

Store as secrets:
```bash
wrangler secret put HF_TOKEN
wrangler secret put HF_USERNAME
wrangler secret put GITHUB_TOKEN
wrangler secret put KAGGLE_USERNAME
wrangler secret put KAGGLE_KEY
```

## 14. Tool Functions: Upload to HF Storage Buckets
```js
async function uploadToHFBucket(files, bucketName, env) {
  const base = `https://huggingface.co/api/buckets/${env.HF_USERNAME}/${bucketName}`;
  const results = [];
  for (const file of files) {
    const res = await fetch(`${base}/${file.path}`, {
      method: "PUT",
      headers: {
        Authorization: `Bearer ${env.HF_TOKEN}`,
        "Content-Type": "application/octet-stream",
      },
      body: file.content,
    });
    results.push({ path: file.path, status: res.status });
  }
  return results;
}
```

## 15. Tool Functions: Push to GitHub
```js
async function pushToGitHub(repoName, files, commitMessage, env) {
  const owner = "nanobot-ai-agent"; // your bot username
  const repo = `${owner}/${repoName}`;

  // Create repo if needed
  await fetch("https://api.github.com/user/repos", {
    method: "POST",
    headers: { Authorization: `token ${env.GITHUB_TOKEN}`, "Content-Type": "application/json" },
    body: JSON.stringify({ name: repoName, private: false }),
  });

  for (const file of files) {
    await fetch(`https://api.github.com/repos/${repo}/contents/${file.path}`, {
      method: "PUT",
      headers: { Authorization: `token ${env.GITHUB_TOKEN}`, "Content-Type": "application/json" },
      body: JSON.stringify({
        message: commitMessage,
        content: btoa(typeof file.content === "string" ? file.content : new TextDecoder().decode(file.content)),
      }),
    });
  }
  return `https://github.com/${repo}`;
}
```

## 16. Tool Functions: Upload to Kaggle
```js
async function uploadToKaggle(datasetTitle, files, env) {
  const form = new FormData();
  form.append("metadata", JSON.stringify({ title: datasetTitle, isPrivate: false }));
  for (const file of files) {
    form.append("file", new Blob([file.content]), file.path);
  }

  const res = await fetch("https://www.kaggle.com/api/v1/datasets/create/version", {
    method: "POST",
    headers: { Authorization: `Basic ${btoa(`${env.KAGGLE_USERNAME}:${env.KAGGLE_KEY}`)}` },
    body: form,
  });
  return await res.json();
}
```

## 17. Full Agent Flow – Example “Build a Full App”
In `handleTelegramUpdate`, after getting user text:

```js
const llmResult = await callLLMWithTools(text, history, env);

// Process tool calls from LLM (parse tool_calls array)
for (const toolCall of llmResult.tool_calls || []) {
  if (toolCall.function.name === "pushToGitHub") {
    const args = JSON.parse(toolCall.function.arguments);
    const url = await pushToGitHub(args.repoName, args.files, args.message, env);
    await sendTelegramMessage(chatId, `✅ Pushed to GitHub: ${url}`);
  }
  // Similarly handle R2 upload, D1 log, HF, Kaggle, etc.
}

// Log to D1
await logGeneratedApp(chatId, projectName, { github: url, r2: r2Url, ... }, env);
```

**Recommended system prompt**: "You are Nanobot, an autonomous coding agent. Use tools to generate files, upload to R2/HF, push to GitHub, publish to Kaggle, and log to D1. Always return public links."

## 18. Advanced: Agents SDK + Multi-Step Reasoning
Use the official Cloudflare Agents SDK + Durable Objects for long-running, stateful agents with built-in tool calling and memory. Bind your agent class in `wrangler.toml` for persistent per-user sessions.

## 19. Custom Domain + Professional Replies
Add your domain in Cloudflare dashboard → Workers → Triggers → Add custom domain.  
Make replies rich with Markdown links to R2, GitHub, HF, Kaggle.

## 20. Local Dev, Testing & Logs
```bash
wrangler dev          # local with bindings
wrangler tail         # live logs
wrangler deploy       # push to production
```

Use ngrok or Cloudflare Tunnel for local webhook testing.

## 21. Troubleshooting
- Webhook not firing? Double-check `setWebhook` URL and that it returns 200.
- CPU limit hit? Keep JS minimal; offload to LLM + I/O.
- Secret not found? Re-run `wrangler secret put`.
- D1/R2 not bound? Verify `wrangler.toml` and redeploy.
- HF/GitHub rate limits? Monitor usage; free tiers are generous for personal agents.

## 22. Scaling & Upgrade Path
Stay free until ~3,000–5,000 daily messages or heavy storage. Upgrade to Workers Paid ($5/mo) for higher limits (30s CPU, more requests, larger D1) only when needed. All services scale smoothly.

## 23. Resources
- Cloudflare Workers Docs: https://developers.cloudflare.com/workers/
- D1 Docs: https://developers.cloudflare.com/d1/
- R2 Docs: https://developers.cloudflare.com/r2/
- Agents SDK: https://github.com/cloudflare/agents
- Hugging Face Buckets: https://huggingface.co/docs/huggingface_hub/guides/buckets
- Telegram Bot API: https://core.telegram.org/bots/api
