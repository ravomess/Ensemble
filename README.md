# Rachi's Plugin Marketplace

A Claude/Cowork plugin marketplace hosting:

- **ensemble** — multi-model AI orchestrator. Relays one prompt across Claude, ChatGPT, Gemini, Grok, DeepSeek, Perplexity, Groq, and Mistral using your own logged-in browser sessions. No API keys.

## Add this marketplace (one time)

In Cowork, open plugin settings (Settings > Capabilities) and add this repository's URL as a marketplace, then install "ensemble" and restart Cowork.

CLI equivalent:
```
/plugin marketplace add <your-github-username>/ensemble-marketplace
/plugin install ensemble@rachi-plugins
```

## Requirements to run ensemble

- The Claude in Chrome extension, installed and connected (needed only for the non-Claude models).
- Be logged in to the AI services you want to use, in that browser.

See `ensemble/README.md` for usage and `ensemble/CHANGELOG.md` for version history.

## Updating

Edit files under `ensemble/`, bump the version in `ensemble/.claude-plugin/plugin.json`, add a CHANGELOG entry, and commit (or re-upload on GitHub). People who added this marketplace can then pull the update.
