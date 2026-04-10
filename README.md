# Ultimate Guide to Setting Up Cloudflare Workers for Your Free 24/7 Nanobot Agent AI  
**Creates Files & Code • Builds Entire Apps • Own HF Storage Buckets • Own GitHub • Own Kaggle • No Credit Card Ever**

**Last updated:** April 2026  
**Target audience:** Builders of autonomous AI coding agents that live on messages and ship real artifacts.

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
- [11. R2 for Temporary File/Code Generation](#11-r2-for-temporary-filecode-generation)
- [12. The Agent’s Own Accounts Setup (HF Buckets + GitHub + Kaggle)](#12-the-agents-own-accounts-setup-hf-buckets--github--kaggle)
- [13. Tool Functions: Upload to HF Storage Buckets](#13-tool-functions-upload-to-hf-storage-buckets)
- [14. Tool Functions: Push to GitHub](#14-tool-functions-push-to-github)
- [15. Tool Functions: Upload to Kaggle](#15-tool-functions-upload-to-kaggle)
- [16. Full Agent Flow – Example “Build a Full App”](#16-full-agent-flow--example-build-a-full-app)
- [17. Advanced: Agents SDK + Multi-Step Reasoning](#17-advanced-agents-sdk--multi-step-reasoning)
- [18. Custom Domain + Professional Replies](#18-custom-domain--professional-replies)
- [19. Local Dev, Testing & Logs](#19-local-dev-testing--logs)
- [20. Troubleshooting](#20-troubleshooting)
- [21. Scaling & Upgrade Path](#21-scaling--upgrade-path)
- [22. Resources](#22-resources)

## 1. Why This Stack for Your Full Agent?
- Cloudflare Workers (free): on-demand, 10 ms CPU budget (but your code stays under 8 ms — LLM + API calls are pure I/O).
- HF Storage Buckets (new 2026): S3-like mutable storage on the Hub — perfect for checkpoints, artifacts, datasets.
- GitHub Free: unlimited repos, full REST API for auto-commits.
- Kaggle Free: dataset/notebook uploads via API.
- Everything triggered by a single message → agent reasons → creates → stores → replies with links.
- Zero idle cost, global edge, no credit card.

## 2. Free Tier Limits (All Confirmed 2026)
| Service                  | Key Free Limits                              | Perfect for Your Agent                  |
|--------------------------|----------------------------------------------|-----------------------------------------|
| Cloudflare Workers       | 100k requests/day, 10 ms CPU/request        | Webhook + orchestration                |
| HF Storage Buckets       | Public: best-effort + 100 GB total          | Mutable artifacts & files              |
| GitHub                   | 5,000 API calls/hour with token             | Create/push repos & files              |
| Kaggle                   | Unlimited dataset/notebook uploads (free acct) | Share datasets & kernels            |
| R2 (optional)            | 10 GB-month free, 1M Class A ops            | Temp zip generation                    |

## 3–7. (Same as previous guide — skipped here for brevity; copy from your existing .md if you have it. The Quick Start and Wrangler setup are unchanged.)

## 8. Message-Triggered Webhook Basics (Telegram Example)
(Use the exact code from the previous huge .md — it’s still perfect.)

## 9. Adding AI Smarts + Tool-Calling LLM
Use Grok, OpenAI, Anthropic, etc. with tool calling.  
Store `LLM_API_KEY` as a Worker secret.

The LLM is instructed once (in system prompt) to output JSON tool calls when needed:
- `create_hf_bucket_files`
- `github_push`
- `kaggle_upload_dataset`
- etc.

## 10–11. Persistence & R2
(Existing sections — unchanged.)

## 12. The Agent’s Own Accounts Setup (HF Buckets + GitHub + Kaggle)
All are **free forever** and require **no credit card**.

### 12.1 Hugging Face (Storage Buckets)
1. Go to https://huggingface.co/join → create account (or use existing).
2. Create a new **Storage Bucket**:
   - Go to https://huggingface.co/settings/storage
   - Click “Create bucket” → name it e.g. `nanobot-artifacts`
   - Make it public or private (private is free up to 100 GB).
3. Generate a token: Settings → Access Tokens → New token → “Write” role.

### 12.2 GitHub
1. Create a new GitHub account for the bot (e.g. `nanobot-ai-agent`).
2. Go to Settings → Developer settings → Personal access tokens (classic) → Generate new token.
3. Scopes: `repo`, `workflow` (full repo access).

### 12.3 Kaggle
1. Create a new Kaggle account for the bot.
2. Go to Account → Create API Token → download `kaggle.json`.
3. You’ll use the username + key in Worker secrets.

## Store All Tokens as Worker Secrets (Never in code)
```bash
wrangler secret put HF_TOKEN
wrangler secret put GITHUB_TOKEN
wrangler secret put KAGGLE_USERNAME
wrangler secret put KAGGLE_KEY
wrangler secret put LLM_API_KEY   # already had this
```

## 13. Tool Functions: Upload to HF Storage Buckets
Add this to your Worker (uses HF’s REST-compatible API for buckets):

```js
async function uploadToHFBucket(files, bucketName, env) {
  const base = `https://huggingface.co/api/buckets/${env.HF_USERNAME}/${bucketName}`;
  const results = [];

  for (const file of files) {  // files = [{path: "checkpoint.bin", content: Uint8Array}]
    const res = await fetch(`${base}/${file.path}`, {
      method: "PUT",
      headers: {
        Authorization: `Bearer ${env.HF_TOKEN}`,
        "Content-Type": "application/octet-stream",
      },
      body: file.content,
    });
    results.push({ file: file.path, status: res.status });
  }
  return results;
}
```

## 14. Tool Functions: Push to GitHub
```js
async function pushToGitHub(repoName, files, commitMessage, env) {
  const owner = env.GITHUB_OWNER || "nanobot-ai-agent";
  const repo = `${owner}/${repoName}`;

  // First create repo if it doesn't exist
  await fetch(`https://api.github.com/user/repos`, {
    method: "POST",
    headers: { Authorization: `token ${env.GITHUB_TOKEN}`, "Content-Type": "application/json" },
    body: JSON.stringify({ name: repoName, private: false }),
  });

  // Create or update files via GitHub API (contents endpoint)
  for (const file of files) {
    await fetch(`https://api.github.com/repos/${repo}/contents/${file.path}`, {
      method: "PUT",
      headers: { Authorization: `token ${env.GITHUB_TOKEN}`, "Content-Type": "application/json" },
      body: JSON.stringify({
        message: commitMessage,
        content: btoa(file.content),  // base64 for text files
      }),
    });
  }
  return `https://github.com/${repo}`;
}
```

## 15. Tool Functions: Upload to Kaggle
(Kaggle API uses multipart; simple version for datasets)

```js
async function uploadToKaggle(datasetTitle, files, env) {
  // Kaggle expects a metadata JSON + files
  const form = new FormData();
  form.append("metadata", JSON.stringify({ title: datasetTitle, isPrivate: false }));
  for (const file of files) {
    form.append("file", new Blob([file.content]), file.path);
  }

  const res = await fetch("https://www.kaggle.com/api/v1/datasets/create/version", {
    method: "POST",
    headers: {
      Authorization: `Basic ${btoa(`${env.KAGGLE_USERNAME}:${env.KAGGLE_KEY}`)}`,
    },
    body: form,
  });
  return await res.json();
}
```

## 16. Full Agent Flow – Example “Build a Full App”
System prompt for your LLM:
> You are Nanobot, an autonomous coding agent. You have tools: uploadToHFBucket, pushToGitHub, uploadToKaggle, etc. When the user asks to build something, plan step-by-step, generate all files, then call the appropriate tools. Always reply with public links.

In the webhook handler:
```js
// After parsing userText
const toolCalls = await callLLMWithTools(userText, history, env); // returns array of tool calls

for (const call of toolCalls) {
  if (call.tool === "pushToGitHub") {
    const url = await pushToGitHub(call.args.repoName, call.args.files, call.args.message, env);
    await sendTelegramMessage(chatId, `✅ Pushed to GitHub: ${url}`);
  }
  // same for HF & Kaggle
}
```

## 17. Advanced: Agents SDK + Multi-Step Reasoning
Bind a Durable Object as your agent brain (persists across messages). Use Cloudflare Agents SDK for built-in tool calling + memory.

## 18–22. (Same as previous — custom domain, logs, troubleshooting, resources.)

---

**You now have the complete blueprint.**  
Your nanobot can:
- Receive “Build me a React + FastAPI todo app with dataset”
- LLM plans → generates files
- Pushes full repo to **its own GitHub**
- Uploads training artifacts/checkpoints to **its own HF Storage Bucket**
- Publishes the dataset to **its own Kaggle**

All on free Cloudflare Workers, triggered instantly by any message.
