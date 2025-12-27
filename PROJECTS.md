# Related Projects & Working Code

This repository contains **documentation and architecture** for the Poor People App. For working code examples and tools, see the projects below.

---

## 🤖 PPA Mobilization Agent Lite

**Status:** ✅ Open Source & Available  
**Repository:** [github.com/FotiosMpouris/ppa-mobilization-agent-lite](https://github.com/FotiosMpouris/ppa-mobilization-agent-lite)  
**Video Tutorial:** 🎬 _Coming Soon_

### What It Is

A local-first AI content generation tool that helps activists, creators, and community organizers amplify their message on Nostr and other platforms.

**Key Features:**

- Runs entirely on your local computer (no AWS needed)
- Uses your own OpenAI and Nostr keys
- Train it on your PDFs for context-aware content
- Generate posts in seconds with consistent voice
- Memory system prevents repetition

**Tech Stack:**

- Python 3.12
- OpenAI API (GPT-4)
- Nostr protocol (NIP-01, NIP-07)
- Local PDF processing
- Simple web UI (localhost)

**Why We Built It:**

This tool was created to support the PPA's communication strategy but is useful for anyone who needs to maintain a consistent voice across social media while staying in control of their keys and data.

---

## 🏗️ PPA Core Application

**Status:** 🚧 In Development  
**Repository:** _Private (for now)_

The main Poor People App platform implementing the architecture described in this repository.

**Current Progress:**

- ✅ Next.js 15 frontend scaffolding
- ✅ AWS infrastructure (EC2, Lambda, DynamoDB)
- ✅ Nostr relay configuration (strfry)
- 🚧 NDK integration (in progress)
- 🚧 Trust layer implementation (in progress)
- ⏳ Sentinel verification workflows (planned)

**Tech Stack:**

- Next.js 15 + TypeScript
- NDK (Nostr Development Kit) v2.14.33
- AWS Lambda + DynamoDB
- Self-hosted Nostr relay (strfry)

**Timeline:** Prototype demo targeted for Q1 2025

---

## 🎯 Future Projects

### PPA Sentinel Dashboard

Admin interface for Sentinels to manage verification workflows, review disputes, and monitor trust metrics.

**Status:** Planned for Q2 2025

### PPA Hub Representative Portal

Geographic coordination tools for local hub representatives to onboard communities and configure service packages.

**Status:** Planned for Q2 2025

### PPA Mobile Apps

Native iOS/Android applications for better mobile UX.

**Status:** Under consideration (web-first approach for now)

---

## 💡 Want to Build Something Related?

If you're interested in building tools or extensions that work with PPA's infrastructure, we'd love to collaborate!

**Potential projects:**

- Nostr client plugins for PPA events
- Trust score calculators using Grapevine
- Geographic matching algorithms
- Service package templates
- Community onboarding tools

**Get in touch:** See [CONTRIBUTING.md](CONTRIBUTING.md) for contact details.

---

## 🔗 Additional Resources

- **PPA White Paper:** _Coming to GiveSendGo campaign page_
- **Nostr Protocol:** [github.com/nostr-protocol/nostr](https://github.com/nostr-protocol/nostr)
- **Grapevine Trust Network:** [github.com/mplorentz/grapevine](https://github.com/mplorentz/grapevine)
- **strfry Relay:** [github.com/hoytech/strfry](https://github.com/hoytech/strfry)
