# Memory Injection: Making Prompt Injections and Jailbreaks Persistent in ChatGPT, Claude, and Gemini

## Overview

This work explores memory injections against Anthropic Claude (Sonnet 4.6), Google Gemini (Flash), and OpenAI ChatGPT (Instant 5.3). Memory injection attacks exploit native, long-term memory-based features to store prompt injections. As a result, the effects of the injection persist across chat sessions. All of NIST's goals identified in [NIST's report on adversarial machine learning](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-2e2025.pdf) are explored:

1. **Privacy compromise**: within a Claude organization, user conversations are exfiltrated to the attacker's Google Drive account;
2. **Availability breakdown**: the model answers "Offline, please try again" to the user prompt;
3. **Integrity violation**: generation of disinformation and harmful content;
4. **Misuse enablement**: jailbreaks that enable general misuse and support the goals above.

In particular, Claude organizations can be attacked such that all of the goals above affect all users within the workspace.

The features explored throughout this work are: memories, MCP connectors, Claude skills, Claude enterprise search, Claude projects, and Claude organization instructions (i.e., organization-wide system prompts).

## Organization

This repository is divided as follows:

- `Reproduction steps and artifacts`: step-by-step instructions to reproduce the attacks, including auxiliary material when necessary;
- `Datasets`: prompts and other strings used throughout the work and the corresponding model outputs;
- `Screenshots`: attack proofs.
