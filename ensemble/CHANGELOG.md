# Changelog — Ensemble

All notable changes to the Ensemble plugin. Newest first.

## 0.7.2
- Added this CHANGELOG.

## 0.7.1
- No em-dashes: every model prompt (relay, critique, revision) and the synthesis now instruct models to avoid em-dashes (use commas, periods, parentheses, or colons). Because models often ignore style rules, the final answer/synthesis is also scrubbed of em-dashes before display.

## 0.7.0
- Run metrics shown when a run finishes: per-model wall-clock time (including page load, waiting, streaming, and retries), total run time, and estimated token counts (in/out) per model and overall. Token figures are clearly labeled estimates (chars / 4), since browser sessions do not report real usage.

## 0.6.0
- Added two models (now 8 total): Grok (grok.com) for real-time X/news/trends, and DeepSeek (chat.deepseek.com) for math, coding, and reasoning diversity.
- Task-based default model sets and orders updated (current events to Grok first; coding/math to DeepSeek first), plus a per-model "best used for" cheat sheet.

## 0.5.2
- Rounds now accept a custom number via the "Other" choice (no longer capped at 1 to 3).

## 0.5.1
- Model order is click-to-select: a recommended order plus alternatives as buttons, and a sequential one-click picker for 4 or more models. No typing required.

## 0.5.0
- Major reliability and safety hardening:
  - Prompt-injection defense: untrusted files and prior model outputs are wrapped in boundary tags and treated as data, not instructions.
  - Per-service preflight (logged in, input present, no modal/CAPTCHA/paywall/rate-limit) before sending.
  - Handoff capsule to control relay-prompt bloat over a length threshold.
  - Thinking-aware completion detection (idle timer suspended while a thinking/loader UI is visible).
  - Post-extraction validation (reject page chrome, echoed prompt, empty or truncated text; try the next method).
  - Skip/partial relay semantics: note skipped hops to the next model; keep and forward usable partial output.
  - Standard failure taxonomy and rate-limit handling in session.errors.
  - Synthesis quality: agreement treated as a signal not proof; configurable synthesizer (Claude, last model, chosen model, or compare-and-contrast); payload capped to each model's final variant.
  - Per-service fixes (ChatGPT Canvas/Continue/Regenerate; Gemini extraction order; Perplexity citations and privacy; Groq/Mistral clear prior conversation).
  - Required post-insert input verification (read the field back before submitting).
  - Run-state checklist near Execution; parallel Independent mode; trimmed duplicated structure; verbose-by-default display wrappers.

## 0.4.3
- Trimmed the skill description under the 1024-character limit, fixing "plugin validation failed" on save.

## 0.4.2
- Broadened natural-language triggers (fires without the literal word "ensemble").
- Model order made a separate, required setup question.

## 0.4.1
- Verbose live display: each model's full response is shown the moment it completes, not just a summary or the final.

## 0.4.0
- Post-run continuation loop: after the final answer, offer to revise, ask a follow-up, or run more rounds, building on the current draft instead of restarting.

## 0.3.1
- Gemini reliability: completion via text-idle (the Stop button lingers and is unreliable), read the whole model-response element during the fade-in animation, about 90s ceiling with one retry, and dismiss the "Personal Intelligence" consent modal with "Not now."
- ChatGPT: detect and extract answers rendered in the Canvas side panel; prefer get_page_text over a mid-render innerText check.

## 0.3.0
- Relay (build-on) is now the default mode: each model improves the answer so far instead of starting fresh.
- Setup asks number of rounds, model order, and per-round rotation (same, rotate, reverse, or custom).

## 0.2.1
- Claude answers in-session (no browser). The Claude in Chrome extension is required only for the non-Claude models; a Claude-only run needs nothing extra.

## 0.2.0
- Hardening review fixes: documented the Claude in Chrome extension dependency and terms-of-service note; fixed the ChatGPT ProseMirror composer selector; safe prompt injection via JSON.stringify; critique mode requires 2+ rounds; synthesis anonymizes model labels to reduce self-preference; rounded out plugin.json (license, keywords).

## 0.1.0
- Initial version: browser-driven multi-model orchestration across Claude, Gemini, ChatGPT, Groq, Mistral, and Perplexity, with Independent and Critique modes and optional synthesis.
