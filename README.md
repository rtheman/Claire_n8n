# Cl(ai)re

**Cl(ai)re** is an AI-powered personal scheduling assistant built on top of [n8n](https://n8n.io/), Google Calendar, and [ElevenLabs](https://elevenlabs.io). She handles appointment scheduling through natural conversation — so you can book, reschedule, check availability, and cancel meetings just by talking (or typing).

## How It Works

Cl(ai)re is made up of two layers:

1. **The brain — ElevenLabs + a custom system prompt**: Defines Cl(ai)re's personality, scheduling rules, language handling, and how she interprets your requests. She's warm, professional, and gets straight to the point.

2. **The hands — an n8n workflow via MCP**: An MCP Server Trigger exposes four Google Calendar tools (get events, create, update, delete) that Cl(ai)re calls dynamically based on what you ask her to do.

## Try Cl(ai)re

Cl(ai)re is live — visit [rtheman.com](https://rtheman.com) and find her in the bottom-right corner of the page. She currently speaks **English and German**.

## What She Can Do

- Check Rich's availability across a 6-month window
- Book new appointments (with confirmation before anything is scheduled)
- Reschedule or cancel existing meetings
- Handle timezones automatically — Cl(ai)re is based in Berlin (CET/CEST) but aware of other timezones
- Converse in **English and German** (coming soon: Afrikaans and Cantonese)
- Protect your privacy — never reveals the details of existing calendar events, only whether a slot is free or busy

## Stack

- **n8n** — workflow automation and MCP server
- **Google Calendar API** — via OAuth2
- **ElevenLabs** — voice interface and conversational AI
- **Model Context Protocol (MCP)** — bridges the AI layer to the calendar tools

## Structure

- `/n8n_workflows` — n8n exported workflow JSON files (MCP server connecting to Google Calendar)
- `/elevenlabs_system_prompt` — Cl(ai)re's system prompt defining her behavior, rules, and personality

## Setup

1. Import workflow `.json` files from `/n8n_workflows` into your n8n instance
2. Configure Google Calendar OAuth2 credentials in n8n
3. Configure ElevenLabs API credentials in n8n
4. Load the system prompt from `/elevenlabs_system_prompt` into your ElevenLabs agent
5. Set required environment variables (see `.env.example`)

## Requirements

- n8n instance (hosted on Hostinger's VPS)
- ElevenLabs API key
- Google Calendar API credentials (OAuth2)
