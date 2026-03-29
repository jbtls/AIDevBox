# AI Dev Box

[![Vibecoded](https://img.shields.io/badge/vibecoded-yes-blue.svg)](#)
[![Testing Only](https://img.shields.io/badge/testing-only-orange.svg)](#)
[![Not for Production](https://img.shields.io/badge/not_for_production-red.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## ⚠️ Disclaimer

**This project is vibecoded and intended for testing and development purposes only.**

It is NOT recommended for production use. Use at your own risk.

## Motivation

This project provides an **isolated environment** for testing AI coding agents. The VM has no access to your host system, protecting your machine from unintended modifications or side effects that AI agents might cause during testing.

Useful for:
- Safely testing AI agent behavior
- Experimenting with different AI tools without risking your host system
- Sandbox development workflows

## What is this?

AI Dev Box sets up a ready-to-use Ubuntu virtual machine with AI coding assistants:

- **OpenCode** — AI coding assistant via OpenRouter
- **GitHub Copilot CLI** — GitHub's command-line AI pair programmer

## Repository Structure

```
AIDevBox/
├── Vagrantfile          # VM configuration (not visible inside the VM)
├── .env.template        # Environment variables template
├── LICENSE              # MIT License
├── README.md            # This file
└── project/             # Your coding workspace — synced to /project in the VM
    ├── README.md        # Workspace notes
```

The `project/` subfolder is the **only** directory shared with the VM. The `Vagrantfile` and all other repository files remain on your host machine and are never accessible inside the VM. This prevents AI coding agents from reading or modifying the VM configuration.

## How to Develop

The `project/` folder is synced to `/project` inside the VM. Place your code there — changes made on your host machine are immediately available inside the VM, and vice versa.

```bash
# Start the VM
vagrant up

# SSH into the VM
vagrant ssh

# Work on your code normally — changes sync automatically
# Note: the Vagrantfile is NOT visible inside the VM

# When done, suspend or halt the VM
vagrant suspend   # saves state, faster startup
vagrant halt      # powers off, slower startup but cleaner
```

## Prerequisites

- [VirtualBox](https://www.virtualbox.org/) (version 6.1+)
- [Vagrant](https://www.vagrantup.com/) (version 2.2.0+)
- **Optional:**
  - `GH_TOKEN` — GitHub personal access token (for Copilot CLI authentication)
  - `OPENROUTER_API_KEY` — OpenRouter API key (for OpenCode authentication)

## Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd AIDevBox

# Copy the environment template
cp .env.template .env

# Add your API keys to .env (optional — see Environment Variables below)

# Start the virtual machine
vagrant up
```

> **Tip:** Put your project code inside the `project/` folder — that is the workspace shared with the VM.

## Configuration

Edit the `INSTALL` array in `Vagrantfile` to choose which tools to install:

```ruby
# Options: "opencode", "copilot", or both
INSTALL = ["opencode", "copilot"]
```

| Value | Installs |
|-------|----------|
| `["opencode"]` | OpenCode only |
| `["copilot"]` | Copilot CLI only |
| `["opencode", "copilot"]` | Both tools |

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GH_TOKEN` | For Copilot | GitHub personal access token with Copilot access |
| `OPENROUTER_API_KEY` | For OpenCode | OpenRouter API key for AI model access |

## Getting a GitHub Token

1. Go to [GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/personal-access-tokens)
2. Generate new token
3. Set token access:
   - **Resource owner:** Your GitHub username
   - **Permissions:** Copilot (grant access)
4. Copy the token to your `.env` file as `GH_TOKEN`

## Getting an OpenRouter API Key

1. Sign up at [openrouter.ai](https://openrouter.ai/)
2. Navigate to API keys section
3. Create a new key
4. Copy the key to your `.env` file as `OPENROUTER_API_KEY`

## OpenCode + GitHub OAuth

Instead of using an API key, you can authenticate OpenCode with GitHub OAuth:

```bash
# Inside the VM
opencode

# At the prompt
/connect github
```

This will open a browser window for GitHub authentication.

## Troubleshooting

### API Key Authentication Issues

**Symptom:** OpenCode or Copilot CLI fails to authenticate.

**Solutions:**

1. Verify your `.env` file contains valid keys (no extra spaces, quotes, or trailing characters)
2. For Copilot: ensure `GH_TOKEN` has Copilot access granted
3. For OpenCode: try OAuth authentication instead (`/connect github` inside OpenCode)
4. If keys are correct but auth fails, the token may be expired or revoked — generate a new one

## Resource Tuning

The default VM configuration requires:

- **RAM:** 4 GB
- **CPUs:** 4

Adjust based on your host machine:

| Host RAM | Recommended VM RAM | Recommended CPUs |
|----------|-------------------|------------------|
| 8 GB | 2-4 GB | 2-4 |
| 16 GB | 4-8 GB | 4-6 |
| 32 GB+ | 8-16 GB | 6-8 |

Edit the `vb.memory` and `vb.cpus` values in `Vagrantfile` to adjust.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
