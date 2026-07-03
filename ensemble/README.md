# Ensemble Plugin

Multi-model AI orchestrator for Cowork. Sends your prompt to multiple AI services using your existing browser sessions — no API keys required.

## Supported Models

| Model | Service | Cost |
|-------|---------|------|
| Claude | **this Cowork session** (no browser) | Included |
| ChatGPT | chatgpt.com | Free / Plus subscription |
| Gemini | gemini.google.com | Free / Google One |
| Grok | grok.com | Free / X Premium |
| DeepSeek | chat.deepseek.com | Free |
| Perplexity | perplexity.ai | Free / Pro |
| Groq (Llama) | groq.com | Free |
| Mistral | chat.mistral.ai | Free tier |

## Requirements

- **Claude answers directly in your Cowork session — no browser needed.** A Claude-only run needs nothing extra.
- **The Claude in Chrome extension must be installed and connected for the *other* models** (ChatGPT, Gemini, Grok, DeepSeek, Perplexity, Groq, Mistral). Ensemble drives those services through your browser using the Claude-in-Chrome tools; without the extension connected they cannot be clicked, typed into, or read. Install/connect it before running any browser-driven model.
- You must be logged into each service you want to use, in the same browser.

## Setup

1. Install and connect the **Claude in Chrome** extension.
2. Log into the AI services you want to use in that browser.
3. Install this plugin in Cowork.
4. Say "run ensemble" or "ask all models [your prompt]".

## How to Use

Just ask naturally:
- "Run ensemble: what are the best practices for code review?"
- "Ask Claude and Gemini to compare these two approaches: [paste text]"
- "Multi-model run, 2 rounds with critique, then synthesize"

Cowork will walk you through model selection, configuration, and run the orchestration.

## Features

- **Multiple rounds** — models iterate and improve across rounds
- **Critique mode** — each model reviews the previous model's response
- **Independent mode** — each model answers the prompt fresh
- **Manual pacing** — pause after each response to review or edit
- **Synthesis** — Claude reads all responses and produces a final best answer
- **File attachment** — attach a text file and its contents are included in the prompt
- **Session export** — save the full session as a markdown file

## Notes

- You must be logged into each service in your browser before running.
- Browser automation is slower than API calls — a 3-model run takes ~30-60 seconds.
- If a service fails or isn't logged in, it's skipped and the run continues.
- **Terms of service:** automating consumer chat UIs (ChatGPT, Gemini, Mistral, Perplexity, etc.) may be against those services' terms of use. Use this on accounts you control and at your own discretion.
