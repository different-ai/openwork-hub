---
name: clawdtalk
description: |
  Give your AI agent a real phone number for voice calls and SMS.

  Triggers when user mentions:
  - "make a call", "phone call", "voice call"
  - "send SMS", "text message"
  - "ClawdTalk", "phone number for AI"
---

ClawdTalk gives any AI agent a real phone number with voice calling (inbound + outbound) and SMS capabilities via WebSocket. No server required - just connect and start taking calls.

## Quick Usage (Already Configured)

### Make an outbound call
```bash
source .skill.config && wscat -c "$CLAWDTALK_WS_URL" -x '{"type":"call","to":"+15551234567"}'
```

### Send an SMS
```bash
source .skill.config && wscat -c "$CLAWDTALK_WS_URL" -x '{"type":"sms","to":"+15551234567","body":"Hello from OpenWork!"}'
```

### Listen for inbound calls
```bash
source .skill.config && wscat -c "$CLAWDTALK_WS_URL"
# Incoming call event: {"type":"call","from":"+15559876543","call_id":"..."}
```

## WebSocket Protocol

Connect to your ClawdTalk WebSocket endpoint (provided after signup at https://clawdtalk.com).

### Outbound Messages

| Type | Fields | Description |
|------|--------|-------------|
| `call` | `to` (E.164) | Initiate an outbound call |
| `sms` | `to`, `body` | Send a text message |
| `answer` | `call_id` | Answer an incoming call |
| `hangup` | `call_id` | End a call |
| `audio` | `call_id`, `audio` (base64) | Send audio to call |
| `dtmf` | `call_id`, `digits` | Send DTMF tones |

### Inbound Events

| Type | Fields | Description |
|------|--------|-------------|
| `call` | `from`, `call_id` | Incoming call |
| `sms` | `from`, `body` | Incoming SMS |
| `audio` | `call_id`, `audio` | Audio from call |
| `transcription` | `call_id`, `text` | Speech-to-text |
| `hangup` | `call_id` | Call ended |
| `error` | `message` | Error occurred |

## Using with AI Agents

### Node.js Example
```javascript
const WebSocket = require('ws');
const ws = new WebSocket(process.env.CLAWDTALK_WS_URL);

ws.on('message', (data) => {
  const event = JSON.parse(data);
  
  if (event.type === 'call') {
    // Handle inbound call - auto-answer
    ws.send(JSON.stringify({ type: 'answer', call_id: event.call_id }));
  }
  
  if (event.type === 'transcription') {
    // Process speech with your AI
    const response = await yourAI.respond(event.text);
    // Send TTS audio back (base64)
    ws.send(JSON.stringify({ type: 'audio', call_id: event.call_id, audio: response.audio }));
  }
});
```

## Common Gotchas

- Phone numbers must be in E.164 format: `+1XXXXXXXXXX`
- WebSocket URL is per-agent, tied to your ClawdTalk account
- Audio is base64-encoded PCM (16kHz, 16-bit, mono)
- Inbound calls timeout if not answered within 30 seconds
- SMS body limit: 160 characters (GSM-7) or 70 characters (UCS-2)

## First-Time Setup

### 1. Sign up for ClawdTalk
Visit https://clawdtalk.com to create an account and get your agent's phone number.

### 2. Get your WebSocket URL
After signup, copy your WebSocket endpoint from the dashboard.

### 3. Store credentials
```bash
echo 'CLAWDTALK_WS_URL="wss://clawdtalk.com/ws/YOUR_AGENT_ID"' >> skills/clawdtalk/.skill.config
echo 'CLAWDTALK_PHONE="+15551234567"' >> skills/clawdtalk/.skill.config
```

### 4. Install wscat (for CLI usage)
```bash
npm install -g wscat
```

### 5. Test the connection
```bash
source .skill.config && wscat -c "$CLAWDTALK_WS_URL"
```

## Resources

- [ClawdTalk Website](https://clawdtalk.com)
- [GitHub Client](https://github.com/team-telnyx/clawdtalk-client)
- [Telnyx Voice API](https://developers.telnyx.com/docs/api/v2/overview)
