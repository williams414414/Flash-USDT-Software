# Flash-USDT-Software
# Flash USDT Software https://drewztools.com/

Lightweight demo for monitoring **USDT (Tether)** `Transfer` events on Ethereum-compatible networks.

> **Purpose:** educational monitoring and analytics only. Not financial advice. No exploit, flash-loan, purchase, or fraudulent functionality is included.

## Features

* Connects to an Ethereum RPC/WebSocket provider
* Listens to USDT `Transfer` events in near real-time
* Logs transfers to the console and exposes a tiny REST endpoint with recent items
* Minimal, easy-to-read Node.js example (ethers.js)

## Quick start

```bash
git clone https://github.com/<your-username>/flash-usdt-software.git
cd flash-usdt-software
cp .env.example .env
# fill in RPC_WS_URL (or RPC_HTTP_URL) and optionally USDT_CONTRACT_ADDRESS
npm install
npm start
```
https://drewztools.com/
Then open: `http://localhost:4000/api/recent` to see recent transfers in JSON.

## Notes

* USDT uses **6 decimals** — convert raw token units for human-readable amounts (divide by 1e6).
* Do **not** commit `.env` or API keys. Use GitHub Secrets for CI/deploy.
* For production reliability use a paid websocket RPC provider (Alchemy, Infura, QuickNode), or run your own node.

## Resources & Contact

* Website / tools: [https://drewztools.com/](https://drewztools.com/)
* Telegram: @drewztooolz
* Phone / WhatsApp: +1 (770) 666–2531

## Disclaimer

For educational and monitoring use only. Do not use for illegal, deceptive, or exploitative activities. Follow applicable laws and platform policies.

## License

MIT
```js
// Simple USDT Transfer watcher (educational/demo)
require('dotenv').config();
const express = require('express');
const { ethers } = require('ethers');

const PORT = process.env.PORT || 4000;
const USDT = process.env.USDT_CONTRACT_ADDRESS || '0xdAC17F958D2ee523a2206206994597C13D831ec7';
const RPC_WS = process.env.RPC_WS_URL;
const RPC_HTTP = process.env.RPC_HTTP_URL;

const app = express();
let recent = [];

// minimal Transfer ABI
const ERC20_ABI = ["event Transfer(address indexed from, address indexed to, uint256 value)"];

async function startWatcher(onEvent) {
  let provider;
  if (RPC_WS) {
    provider = new ethers.WebSocketProvider(RPC_WS);
  } else if (RPC_HTTP) {
    provider = new ethers.JsonRpcProvider(RPC_HTTP);
  } else {
    throw new Error('Set RPC_WS_URL or RPC_HTTP_URL in .env');
  }

  const contract = new ethers.Contract(USDT, ERC20_ABI, provider);
  console.log('Listening for Transfer events on', USDT);

  contract.on('Transfer', (from, to, value, event) => {
    const tx = {
      hash: event.transactionHash,
      from,
      to,
      value: value.toString(),
      blockNumber: event.blockNumber,
      timestamp: Date.now()
    };
    onEvent(tx);
  });

  provider._websocket?.on('close', (code) => {
    console.warn('WebSocket closed with code', code);
  });
}

startWatcher((tx) => {
  recent.unshift(tx);
  if (recent.length > 200) recent.pop();
  console.log(`USDT Transfer: ${tx.value} raw units | ${tx.from} -> ${tx.to} | tx: ${tx.hash}`);
}).catch(err => {
  console.error('Watcher failed to start:', err.message || err);
  process.exit(1);
});

app.get('/api/recent', (req, res) => {
  res.json({ recent });
});

app.listen(PORT, () => console.log(`API running on http://localhost:${PORT}`));
```

Transform the way you send and receive cryptocurrency with Flash USDT — where stealth meets speed! 🚀
