# Rachi's Plugin Marketplace

A Claude/Cowork plugin marketplace hosting:

- **ensemble** — multi-model AI orchestrator. Relays one prompt across Claude, ChatGPT, Gemini, Grok, DeepSeek, Perplexity, Groq, and Mistral using your own logged-in browser sessions. No API keys.

## Add this marketplace (one time)

In Cowork, open plugin settings (Settings > Capabilities), add this repository as a marketplace using its URL:

https://github.com/ravomess/Ensemble

Then install "ensemble" and fully quit and relaunch Cowork.

CLI equivalent:
```
/plugin marketplace add ravomess/Ensemble
/plugin install ensemble@rachi-plugins
```

## Requirements to run ensemble

- The Claude in Chrome extension, installed and connected (needed only for the non-Claude models; a Claude-only run needs nothing extra).
- Be logged in to the AI services you want to use, in that browser.

See `ensemble/README.md` for usage and `ensemble/CHANGELOG.md` for version history.

## Updating

Edit files under `ensemble/`, bump the version in `ensemble/.claude-plugin/plugin.json`, add a `CHANGELOG.md` entry, and commit (or re-upload on GitHub). Anyone who added this marketplace can then pull the update, no files to re-send.

## Layout
Ensemble/
├── .claude-plugin/
│   └── marketplace.json      catalog listing the plugin(s)
└── ensemble/                 the plugin
├── .claude-plugin/plugin.json
├── skills/ensemble/SKILL.md
├── skills/ensemble/references/
├── README.md
└── CHANGELOG.md


