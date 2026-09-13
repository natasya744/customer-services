# Evolution API Setup Guide

This guide explains how to set up Evolution API as your WhatsApp provider for the Customer Service application.

## Overview

Evolution API is an open-source WhatsApp API that allows your application to send and receive WhatsApp messages programmatically. It serves as the WhatsApp provider for this project, bridging the customer service app with WhatsApp.

## Why Evolution API?

The Customer Service application needs a WhatsApp provider to:
- Send messages to customers on WhatsApp
- Receive incoming WhatsApp messages
- Manage WhatsApp instances (sessions)
- Handle QR code authentication for WhatsApp Business

Evolution API wraps the underlying Baileys library (the same library used by WhatsApp Web) and exposes a REST API for all WhatsApp operations.

## Prerequisites

- A WhatsApp Business account or a regular WhatsApp number
- Node.js installed locally
- Docker (recommended for running Evolution API)
- A Supabase project (already set up for this project)

## Step 1: Install Evolution API

### Option A: Docker (Recommended)

```bash
git clone https://github.com/evolution-foundation/evolution-api.git
cd evolution-api
docker-compose up -d
```

The server will start on port 8080 by default.

### Option B: Direct Install

```bash
git clone https://github.com/evolution-foundation/evolution-api.git
cd evolution-api
npm install
npm run build
npm start
```

## Step 2: Create a WhatsApp Instance

Once Evolution API is running, create a new WhatsApp instance:

```bash
curl -X POST http://localhost:8080/instance/create \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "customer-service",
    "integration": "whatsapp"
  }'
```

This creates a new WhatsApp session named `customer-service`.

## Step 3: Connect WhatsApp via QR Code

After creating the instance, you need to scan a QR code to connect your WhatsApp number:

```bash
curl http://localhost:8080/instance/connect/customer-service
```

This returns a QR code. Scan it using the WhatsApp app:
1. Open WhatsApp on your phone
2. Go to **Settings** → **Linked Devices** → **Link a Device**
3. Scan the QR code displayed in the Evolution API response

Once scanned, the WhatsApp number is connected to the Evolution API instance.

## Step 4: Get Your API Credentials

You need two values for the Customer Service app configuration:

| Credential | Description | How to Get It |
| --- | --- | --- |
| **EVOLUTION_API_URL** | URL where Evolution API is running | `http://localhost:8080` for local, or your server URL |
| **EVOLUTION_API_KEY** | API key for authentication | Find in Evolution API dashboard or check the `.env` file after setup |

To find your API key, check the Evolution API instance settings:

```bash
curl http://localhost:8080/instance/fetch-instances
```

Look for the `apiKey` field in the response.

## Step 5: Configure the Backend

Edit the backend `.env` file at `/Users/astridputrianastasyah/customer services/backend/.env.example`:

```bash
# --- Evolution API (browser auth) ---
EVOLUTION_API_URL=http://localhost:8080
EVOLUTION_API_KEY=your-evolution-api-key-here
```

Copy this to `.env` in the backend folder and fill in your actual values:

```bash
cd /Users/astridputrianastasyah/customer services/backend
cp .env.example .env
```

Update the variables with your real values:
- `EVOLUTION_API_URL`: The URL where Evolution API is running
- `EVOLUTION_API_KEY`: The API key from Step 4

## Step 6: Configure the Frontend

If the frontend needs to access Evolution API directly (e.g., for QR code display), add to `frontend/.env.example`:

```bash
VITE_EVOLUTION_API_URL=http://localhost:8080
VITE_EVOLUTION_API_KEY=your-evolution-api-key-here
```

Then copy to `.env` in the frontend folder.

## Step 7: Test the Connection

Verify the Evolution API is working and the instance is connected:

```bash
# Check instances
curl http://localhost:8080/instance/fetch-instances

# Check instance connection status
curl http://localhost:8080/instance/connection-status/customer-service
```

You should see the instance status as `CONNECTED`.

## Step 8: Send a Test Message

Send a test WhatsApp message to verify everything works:

```bash
curl -X POST http://localhost:8080/message/sendText/customer-service \
  -H "Content-Type: application/json" \
  -H "apikey: your-evolution-api-key-here" \
  -d '{
    "number": "5511999999999",
    "text": "Hello from Customer Service!"
  }'
```

## Step 9: Integrate with the Customer Service App

### Backend Integration

The backend Python server will use the Evolution API to:
- **Send messages**: Forward customer service responses to WhatsApp
- **Receive messages**: Listen for incoming WhatsApp messages via webhooks
- **Manage instances**: Create, connect, and disconnect WhatsApp instances

The backend already has `EVOLUTION_API_URL` and `EVOLUTION_API_KEY` in its `.env.example`. Wire these into the backend configuration module when building out the WhatsApp service layer.

### Webhook Setup

Configure Evolution API to send events to your backend:

```bash
curl -X POST http://localhost:8080/webhook/set/webhook \
  -H "Content-Type: application/json" \
  -H "apikey: your-evolution-api-key-here" \
  -d '{
    "webhook": {
      "enabled": true,
      "url": "http://localhost:5000/webhook/evolution",
      "webhook_by_events": true
    }
  }'
```

This forwards all WhatsApp events (messages, connections, QR codes) to your backend at `/webhook/evolution`.

## Step 10: Key Event Types

The following events are available via webhook from Evolution API:

| Event | Description |
| --- | --- |
| `messages_set` | New message received |
| `messages_upsert` | Message receipt confirmation |
| `messages_edit` | Message edited |
| `messages_update` | Message updated |
| `messages_delete` | Message deleted |
| `connection_update` | Connection status changed |
| `qrcode_updated` | New QR code generated |
| `instance_create` | Instance created |
| `instance_delete` | Instance deleted |
| `send_message` | Message sent |
| `contacts_set` | Contact updated |
| `chats_set` | Chat updated |
| `groups_upsert` | Group created/updated |
| `call` | Incoming call |

## Step 11: Run Everything Together

1. Start Evolution API (Docker or direct)
2. Start your Supabase project
3. Start the backend (`uv run uvicorn app.main:app --reload`)
4. Start the frontend (`pnpm dev`)

## Troubleshooting

| Issue | Solution |
| --- | --- |
| Instance not connecting | Make sure WhatsApp is not already linked to another device |
| QR code not showing | Restart the instance: `POST /instance/restart/customer-service` |
| API key not working | Check the instance API key: `GET /instance/fetch-instances` |
| Messages not sending | Verify the instance is `CONNECTED` before sending |
| Webhook not receiving events | Check the webhook URL is accessible from the Evolution API server |

## Reference

- [Evolution API Documentation](https://docs.evolutionfoundation.com.br/en/evolution-api/)
- [Evolution API GitHub Repository](https://github.com/evolution-foundation/evolution-api)
- [Full Environment Variables Reference](https://docs.evolutionfoundation.com.br/en/evolution-api/configuration/env)
