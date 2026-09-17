# Guide: Connect Your Backend to Evolution API

> You already have Evolution API running. This guide tells you what to find and what to give me so we can wire up the `customer_develop` backend.

---

## What You Need to Find (3 Things)

### 1. Your Evolution API URL

Where is your Evolution API hosted? Look for something like:
- `http://localhost:8080` (local)
- `https://evo.yourdomain.com` (self-hosted with domain)
- `https://evo2.something.com` (hosted by a provider)

**How to test it:**
```bash
curl http://YOUR_URL_HERE/
```
If it responds, that's your `EVOLUTION_API_URL`.

---

### 2. Your API Key (`AUTHENTICATION_API_KEY`)

This is the key that authenticates every request to Evolution API. Here's how to find it:

**If you run it with Docker:**
```bash
# Check the container's environment variables
docker exec evolution_api env | grep AUTHENTICATION_API_KEY
```

**If you have the `.env` file on your server:**
```bash
cat /path/to/evolution-api/.env | grep AUTHENTICATION_API_KEY
```

**If you're using a hosting service (VPS, PaaS, etc.):**
- Check your hosting dashboard / environment variables / config file
- Look for `AUTHENTICATION_API_KEY` in your deployment settings

**If you can't find it, generate a new one:**
```bash
python3 -c "import secrets; print(secrets.token_hex(16).upper())"
```

Then you'll need to update it in your Evolution API server's `.env` file and restart the service.

> ⚠️ **Important:** The header name used in every API request is **`apikey`** (lowercase), not `Authorization`.

---

### 3. Your Instance Name

You need a WhatsApp instance already created (or you'll create one). Check what instances exist:

```bash
curl http://YOUR_EVOLUTION_API_URL/instance/fetchInstances \
  -H "apikey: YOUR_API_KEY"
```

This returns a list of instances. Note the `instanceName` for the one you want to use.

If you don't have an instance yet, create one:
```bash
curl -X POST http://YOUR_EVOLUTION_API_URL/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{
    "instanceName": "minha-instancia",
    "integration": "WHATSAPP-BAILEYS",
    "qrcode": true
  }'
```
Then scan the QR Code with WhatsApp on your phone.

---

## What You Need to Give Me

Reply with these three things and I'll set up the backend configuration:

| # | What | Example |
|---|------|---------|
| 1 | **`EVOLUTION_API_URL`** | `http://localhost:8080` or `https://evo.myapp.com` |
| 2 | **`EVOLUTION_API_KEY`** | `4A7B2C9D1E3F5A8B0C2D4E6F8A1B3C5D` |
| 3 | **`INSTANCE_NAME`** | `minha-instancia` |

---

## What Happens Next

Once you provide those three values, I will:

1. Update `backend/.env` with the correct values
2. Update `backend/.env.example` as reference
3. Update `backend/src/config.py` to include the Evolution API settings
4. Create the integration module that lets your backend call Evolution API endpoints

---

## Quick Reference — Common Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/instance/fetchInstances` | List all instances |
| `POST` | `/instance/create` | Create a new instance |
| `GET` | `/instance/connect/{name}` | Get QR Code |
| `GET` | `/instance/connectionState/{name}` | Check connection status |
| `POST` | `/message/send/{name}` | Send a message |
| `DELETE` | `/instance/delete/{name}` | Delete an instance |

All requests need: `Headers: { "apikey": YOUR_API_KEY }`

---

*Based on [Evolution API Documentation](https://docs.evolutionfoundation.com.br/evolution-api)*
