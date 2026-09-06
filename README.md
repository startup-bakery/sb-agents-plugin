# Startup Bakery Agents

<img src="plugins/sb-agents/assets/sb-agents-logo.png" alt="Startup Bakery" width="128">

Install **SB Agents** from this marketplace to use the agents enabled for your Startup Bakery account in Codex or Claude Code.

## Codex

Add this GitHub repository as a plugin marketplace, then install SB Agents:

```sh
codex plugin marketplace add startup-bakery/sb-agents-plugin
codex plugin add sb-agents@sb-agents-plugin
```

Start a new task after installation so Codex loads the plugin and its tools.

## Claude Code

Run these commands inside Claude Code:

```text
/plugin marketplace add startup-bakery/sb-agents-plugin
/plugin install sb-agents@sb-agents-plugin
```

## Connect your account

Complete the gateway's OAuth sign-in when prompted, then ask:

> Show me which Startup Bakery agents I can use.

Choose an available agent. If more than one compatible tenant is available, choose the tenant in chat. Access depends on your account's entitlements; installing the plugin does not grant access to an agent or tenant.

The plugin connects to `https://sb-agents-gateway.onrender.com/mcp`. Agent instructions are loaded from the gateway when needed. Authentication is handled by your client; there are no credentials to paste into this repository.

## Downloadable packages

- [Codex ZIP](dist/openai/sb-agents-codex.zip)
- [Claude plugin bundle](dist/claude/sb-agents-claude.plugin)
- [SHA-256 checksums](SHA256SUMS)

Marketplace installation is the documented setup path above. The archives contain the same plugin for clients that support importing packages.

## Repository contents

This repository distributes the plugin manifests, bootstrap skill, brand assets, marketplace catalogs, and packages. The gateway runs separately. Changes to server-owned agent guides do not require a plugin release.

This is a Startup Bakery marketplace. Listing here does not imply inclusion in an OpenAI or Anthropic official catalog.

See [security reporting](SECURITY.md). For Claude marketplace details, see the [Claude Code documentation](https://code.claude.com/docs/en/plugin-marketplaces).
