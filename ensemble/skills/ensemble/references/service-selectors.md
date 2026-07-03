# Service Selectors & Interaction Patterns

URLs and interaction notes for each supported AI service. **These UIs change often.** Treat the CSS selectors below as hints, not guarantees — the reliable primary path is the `find` tool with natural-language descriptions, falling back to `read_page` / `get_page_text`. Use the selectors only as a fast first guess.

> Reliability note: all of these are React / ProseMirror editors. Prefer the Claude in Chrome `form_input` or typing tools to enter text — they dispatch the events the editors expect. Setting `innerText`/`value` directly often leaves the editor's internal state empty, so Send submits nothing. Only use the JS injection helper (bottom of this file) when typing is impractical. **Always read the input back and confirm it holds your text before submitting** — this is a required step, not optional (see "verify before submitting" below).

---

## Claude (answered in-session — no browser)

**Do not drive claude.ai in the browser.** You are already running inside a Claude (Cowork) session, so Claude's response is produced directly here:

1. Take the prompt for this round (the original prompt in Independent mode, or the critique-wrapped prompt in Critique mode).
2. Answer it yourself, in this session, as Claude would in a normal chat.
3. Store the answer as Claude's response for this round.

No URL, no selectors, no extension needed for Claude. This is faster and more reliable than browser automation, and avoids claude.ai's automation flagging.

---

## Gemini (gemini.google.com)

**URL:** `https://gemini.google.com/app`

**First-run modal:** Gemini may show a "Supercharge Gemini with Personal Intelligence" opt-in (connect Gmail/Drive/etc.). **Dismiss it with "Not now"** — do NOT click "Get started" (that grants access to the user's Google apps). Only then is the input usable.

**Input:** find "Ask Gemini" / "Enter a prompt here" (a `contenteditable` div, usually `div.ql-editor[contenteditable="true"]`)

**Submit:** find "Send" / `button[aria-label="Send message"]` (the up-arrow at the right of the input). Clicking it is more reliable than Enter here.

**Response container:** the answer renders inside `<model-response>` → `message-content` → `.model-response-text .markdown`. To extract:

```javascript
const blocks = document.querySelectorAll('message-content .markdown, .model-response-text');
const last = blocks[blocks.length - 1];
last ? last.innerText : null;
```

**Gemini extraction order (resolves the apparent contradiction below):**
1. **Primary:** the last `<model-response>` element's `innerText` (the DOM query above).
2. **Fallback:** `read_page` to locate the response by accessibility label.
3. **Last resort:** `get_page_text` — for Gemini this often returns only page chrome and misses the answer, so use it only if 1 and 2 fail.

**New chat:** find the compose/new-chat control (top-left pencil), or navigate to `https://gemini.google.com/app`

**Wait for completion — Gemini is slow, animates, and the Stop button lies:**
- **Don't trust the Stop button as the done-signal.** Observed in testing: Gemini's Stop control lingers for several seconds *after* the text has fully rendered. Use **text-idle** instead — poll the response length and treat it as done when it stops growing for ~3–4 seconds.
- **Read the whole `model-response` element, not `.markdown`.** During Gemini's fade-in animation the inner `.markdown` / `.model-response-text` nodes read as empty even though text is visibly on screen. The container `document.querySelector('model-response').innerText` reflects the real length and is the reliable progress signal:

```javascript
const el = document.querySelector('model-response');
const stop = document.querySelector('button[aria-label*="Stop"], button[mattooltip*="Stop"]');
// done ≈ el.innerText length unchanged across two polls (~3-4s apart)
JSON.stringify({ len: el ? el.innerText.trim().length : 0, stopPresent: !!stop });
```

- **For final extraction, read the last `model-response` element's `innerText`** (see the Gemini extraction order above) — `get_page_text` is the last-resort fallback for Gemini, not the primary.
- **Ceiling: wait up to ~90 seconds** before any text appears (Gemini frequently sits on loading dots 30–60s on a cold prompt). If still zero text at the ceiling, it has likely hung — **retry once**: open a fresh `/app`, dismiss the modal, resubmit. If it hangs again, mark Gemini timed out and continue.

**Notes:**
- If the user has Gemini Advanced it is used automatically.
- Observed in testing: a cold first prompt can spin 30–60s before any text; budget for it.
- **The hang can recur even after the retry.** In one run Gemini produced nothing after ~85s, and again produced nothing after a fresh-chat retry. Don't loop indefinitely: one retry, and if it still hasn't rendered by the ceiling, mark Gemini timed out and continue. In Relay mode this is harmless — `currentDraft` simply passes to the next model unchanged.

---

## ChatGPT (chatgpt.com)

**URL:** `https://chatgpt.com/`

**Input:** The composer is now a **ProseMirror contenteditable div, not a `<textarea>`.** find "Message ChatGPT"; selector hint `div#prompt-textarea[contenteditable="true"]`. (Older `textarea` selectors are stale and will not match.)

**Submit:** find "Send message" / `button[data-testid="send-button"]`

**Response container:** find the last assistant message; selector hint `div[data-message-author-role="assistant"]` (take the last)

**New chat:** find "New chat", or navigate to `https://chatgpt.com/`

**Wait for completion:** Send re-enables / stop (square) button disappears / copy + feedback buttons appear, then text idle.

**Watch for interrupt/error UI before accepting an answer:**
- **"Continue generating"** — the answer was cut off; click it and keep waiting.
- **"Regenerate" / "Something went wrong" / a red error** — treat as `extraction_failed`/`service_changed_ui`; retry once (resubmit) before skipping.
- **Canvas detection (do this BEFORE extracting):** if the chat stream shows only the prompt + a "Thought for…" block with no answer body, or you can see an Edit/Copy/Download toolbar, the answer is in the **Canvas** side panel — extract it via the `<h1>`-container method below, not `get_page_text`.

**Notes:**
- Because the input is contenteditable, use the contenteditable injection path (or, preferably, the typing tool) — not the textarea `value` setter.
- Free accounts use a smaller model; Plus accounts use the better default automatically.
- **Watch for Canvas.** ChatGPT sometimes renders long, structured answers in its **Canvas panel** (a side document with an Edit/Copy/Download toolbar) instead of the chat stream. `get_page_text` targets the main chat article and **misses Canvas content** (you'll see only the prompt + "Thought for…" with no answer body). When that happens, extract the Canvas: find the answer's `<h1>` heading and read its nearest container, e.g.

```javascript
const h1 = [...document.querySelectorAll('h1')].find(h => h.innerText.trim().length > 0);
let n = h1;
for (let i=0;i<8 && n.parentElement;i++){ n = n.parentElement;
  const t = n.innerText||''; if (t.length>1500 && !/Chat history|Recents/.test(t)) break; }
n.innerText;
```
  Observed in testing: a final formatted itinerary rendered entirely in Canvas and returned empty from `get_page_text` until read this way.
- **Extraction: use `get_page_text`, not a raw `innerText` length check.** Observed in testing: querying the assistant message's `innerText` mid-render returned a misleadingly short value (~240 chars) for an answer that was actually thousands of characters. `get_page_text` captured the full response reliably. Use the Stop button disappearing as the completion signal, then extract with `get_page_text`.

---

## Grok (grok.com)

**URL:** `https://grok.com/` (also reachable via the Grok tab on x.com). **Login required** (X / Grok account) — if not logged in, classify `not_logged_in` and skip.

**Input:** find "Ask anything" / "What do you want to know?" — a contenteditable/textarea composer.

**Submit:** find the send arrow / "Submit", or Enter.

**Response container:** the last assistant message block (take the last).

**New chat:** find "New chat", or navigate to `https://grok.com/`.

**Wait for completion:** streaming stops / send re-enables, then text idle. Grok often pulls **live X posts** mid-answer, so it can pause partway — use the longer idle threshold and suspend the idle timer during any "searching X" indicator.

**Notes:**
- **Strength:** real-time X/Twitter data, breaking news, trends, public-opinion pulse; less-filtered voice.
- A model selector (Grok 4, etc.) may be present — use whatever's default; don't change it.
- **Privacy:** like Perplexity, it may route the prompt to web/X search — **avoid for confidential prompts** and warn the user before including it for anything private.

---

## DeepSeek (chat.deepseek.com)

**URL:** `https://chat.deepseek.com/`. **Login required** — if walled, classify `not_logged_in` and skip.

**Input:** find "Message DeepSeek" / the chat box — a `textarea` (selector hint `#chat-input`).

**Submit:** find the send button, or Enter.

**Response container:** the last assistant markdown block (take the last).

**Toggles (below the input):**
- **"DeepThink (R1)"** switches to the reasoning model — much slower, and it streams a long **visible thinking trace before the final answer**. Enable it only for hard math/coding/logic; otherwise leave off.
- **"Search"** enables web search. Leave off unless the task needs current info (and heed the privacy note).

**New chat:** find "New chat", or navigate to the URL.

**Wait for completion:** With DeepThink on there's a distinct **"Thinking…" phase** before any answer text — **suspend the idle timer through it** (don't time out early), then use text-idle once the real answer renders. Extract the **final answer** (after the thinking block), not the reasoning trace.

**Notes:**
- **Strength:** math, coding, step-by-step reasoning; adds useful **non-US diversity** to the ensemble (helps catch shared blind spots at synthesis).
- **Capacity:** DeepSeek has busy-server periods ("The server is busy, please try again later") — classify `quota_rate_limited`, back off and retry once, then skip.

---

## Groq (groq.com)

**URL (default):** `https://console.groq.com/playground` (login required). The public `groq.com` demo also exists but its DOM and availability vary — **default to the console playground** and only use the public demo if the user explicitly wants no-login.

**Input:** find "Enter your message" (a `textarea`)

**Submit:** find "Send" / press Enter

**Response container:** the last assistant message block

**New chat:** find a "Clear" / "New" control, or refresh

**Wait for completion:** input re-enables / streaming stops, then text idle

**Notes:**
- Use whatever model is already selected (e.g. Llama 3.3 70b) — don't change it.
- **Clear any prior playground conversation before submitting** (use Clear/New or reload) so the new prompt isn't appended to an old thread. After submitting, verify your prompt registered as a new user message before waiting on the answer.

---

## Mistral (chat.mistral.ai)

**URL:** `https://chat.mistral.ai/chat`

**Input:** find "Ask le Chat anything" (a `textarea`)

**Submit:** find "Send" / press Enter

**Response container:** last assistant message

**New chat:** find "New conversation", or navigate to the URL

**Wait for completion:** input re-enables, then text idle

**Notes:** Free tier available, login required.
- **Start a New conversation before submitting** so the prompt isn't appended to a previous chat; confirm it posted as a user message before waiting.

---

## Perplexity (perplexity.ai)

**URL:** `https://www.perplexity.ai/`

**Input:** find "Ask anything" (a `textarea`)

**Submit:** find "Submit" / press Enter

**Response container:** the main answer block

**New chat:** navigate to `https://www.perplexity.ai/` for a fresh prompt

**Wait for completion:** Perplexity searches the web first, so generation starts late and pauses — wait for sources to finish loading AND text to go idle (use the longer idle threshold).

**Notes:** Basic use needs no login; Pro features do.
- **Capture citations/sources separately.** Perplexity's value is its cited web sources — extract the sources list alongside the answer text and store both.
- **Privacy warning:** Perplexity is web-search-oriented and may route the prompt to web queries. **Avoid using it for confidential or sensitive prompts**, and warn the user before including it for anything private.

---

## General Fallback Strategy

Selectors will break when services update their UI. Order of operations:
1. **Primary:** `find` with natural language — `find("message input box")`, `find("send button")`, `find("last assistant response")`.
2. `read_page` to get the accessibility tree and identify the right element.
3. `javascript_tool` to query the DOM directly if needed.
4. If the page looks completely different, report it and ask the user to manually place focus in the chat input, then proceed from there.

## Entering the Prompt Safely

**Preferred:** use the Claude in Chrome `form_input`/typing tools to type the prompt into the located input. This works with the React/ProseMirror editors and updates their internal state correctly.

**Only if typing is impractical** (very long prompt, typing unsupported on the element), inject via JavaScript. **Never string-interpolate the prompt into the script** — a backtick or `${...}` in the user's text would break the script or execute code. Serialize the prompt with `JSON.stringify` and pass it as data:

```javascript
// promptText is provided to the page as a JSON-encoded string literal.
const PROMPT = JSON.parse(<JSON.stringify(promptText) goes here>);

// contenteditable editors (Claude, Gemini, ChatGPT/ProseMirror):
const el = document.querySelector('div[contenteditable="true"]');
el.focus();
// Use a paste/beforeinput-style insertion so the editor registers the change:
document.execCommand('insertText', false, PROMPT);
el.dispatchEvent(new Event('input', { bubbles: true }));

// plain textareas (Groq, Mistral, Perplexity):
const ta = document.querySelector('textarea');
const setter = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set;
setter.call(ta, PROMPT);
ta.dispatchEvent(new Event('input', { bubbles: true }));
```

**REQUIRED — verify before submitting (do not skip).** After typing or injecting, **read the input field back** and confirm it contains your full prompt:

```javascript
const el = document.querySelector('div[contenteditable="true"], textarea');
(el ? el.innerText || el.value : '').trim().length;   // must be > 0 and ~match your prompt length
```

If it reads empty or far too short, the editor didn't register the text — **retry with the `form_input`/typing tool** (don't just re-run the JS). Only submit once the field verifiably holds the prompt, then click Send rather than pressing Enter. Silent empty-submits are the single most common real failure; this check prevents them.

## Extracting Responses

After waiting for completion, extract the text of the last assistant message:

```javascript
const messages = document.querySelectorAll('[data-message-author-role="assistant"], .assistant-message, model-response, [data-testid="assistant-message"]');
const last = messages[messages.length - 1];
return last ? last.innerText : null;
```

If this returns nothing, use `get_page_text` and parse the response from the full page text, or `read_page` to find the response element by its accessibility label.

**REQUIRED — validate the extraction before accepting it.** Browser extraction can return page chrome, the user's own prompt echoed back, empty text, or stale content from a previous conversation. Before treating extracted text as the model's answer, sanity-check it:
- **Non-empty** and of **plausible length** (not a few stray words for a substantive prompt).
- **Not navigation/UI chrome** (e.g. doesn't start with "New chat / Search chats / Library", model name headers, sidebar items).
- **Not just the prompt echoed back** (compare against the prompt you submitted; high overlap = you grabbed the wrong node).
- **Coherent ending** (not cut mid-token; if it looks truncated, it may still be streaming — wait).

If any check fails, **try the next extraction method** (DOM container → `read_page` → `get_page_text` → Canvas method for ChatGPT) before accepting. Only store a response that passes. If nothing passes, log `extraction_failed` and skip per the error taxonomy.

**Rule of thumb from testing:** prefer `get_page_text` (or reading a whole response *container* element like `model-response`) over measuring a child node's `innerText` length while the model is still streaming. Mid-render, child nodes can read empty or truncated even though the answer is on screen — so use container-level idle-detection for "done" and `get_page_text` for the final pull. Treat **text going idle** as the completion signal; a lingering Stop button (seen on Gemini) is not reliable.
