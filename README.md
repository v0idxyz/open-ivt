# OpenIVT

**Open-source Invalid Traffic detection at ad-tech scale.**

OpenIVT is a full-stack platform that catches the bots, click farms, spoofed devices, and cookie-fraud rings that quietly drain digital advertising budgets. It ships **157 threat detectors**, a browser signal-collection snippet, a Go detection engine built for **500M+ scans/day**, and a React dashboard to see it all — no black boxes, no per-scan billing.

> Most IVT vendors are a closed API and an invoice. OpenIVT is the whole stack, open for you to read, run, and extend.

---

## Why OpenIVT?

- 🕵️ **157 detectors, fully documented** — Bot automation, click fraud, impression fraud, device spoofing, malware injection, network anomalies, session fraud, behavioral tells, and audience/cookie fraud. Every rule is inspectable, not a mystery score.
- ⚡ **Built for scale** — Kafka → worker pool → ClickHouse pipeline engineered for half a billion scans a day, with Redis-backed caching and aggregation.
- 🍪 **Cookie-fraud detection that's genuinely novel** — Catches "orphan" DMP/identity cookies (BlueKai, LiveRamp, UID2, Lotame, Adobe AAM) injected to inflate CPMs, and detects audience-cookie *replay attacks* by correlating value hashes across IPs in real time.
- 🔒 **Privacy-respecting by design** — Cookie values never leave the browser in plaintext; only a lightweight fingerprint is emitted.
- 🖥️ **Own your data** — Everything runs on your infrastructure. Your signals, your database, your rules.

---

## How It Works

```
Browser Snippet (TypeScript)
  ├── Main beacon     — page-load signals, sent immediately
  ├── Auction beacon  — Prebid/SSP signals, ~500ms delayed
  └── NSFW beacon     — on-device image classification (TF.js), fires when idle
         │
         ▼
  Go Ingest API  →  Kafka  →  Worker Pool  →  157 Detectors  →  ClickHouse
                                   ↑
                              Redis (cache + cookie-collision aggregation)

  Dashboard (React)  →  Go API  →  PostgreSQL (config)  +  ClickHouse (analytics)
```

Four components, one system:

| Component | Stack | Role |
|---|---|---|
| **Browser Snippet** | TypeScript | ~500B loader + ~12KB probe bundle collects client-side signals |
| **Detection Server** | Go | Runs 157 detectors concurrently against every signal |
| **Dashboard** | React / Vite | Manage threats, inspect events, browse the catalog, view analytics |
| **Data Pipeline** | Kafka · ClickHouse · Redis | High-throughput ingest, storage, and caching |

---

## Threat Coverage

| Tier | Count | Examples |
|---|---|---|
| 🔴 **High Risk** | 117 | Headless/WebDriver bots, click & impression fraud, canvas/WebGL spoofing, ad injection, proxy/VPN, session hijacking, cookie-jar injection |
| 🟠 **Medium Risk** | 24 | Suspicious referrers, viewability spoofing, floor manipulation, adult-content misrepresentation |
| 🟡 **Low Risk** | 15 | Old browsers, mobile emulation, timezone/IP mismatch, incognito, unusual signals |
| ⚪ **MFA Context** | 1 | Suppresses false positives on legitimate multi-factor auth flows |

The full catalog — code, name, signals, and detection logic for all 157 detectors — lives in the dashboard under **Threat Catalog**.

---

## Quick Start

**Prerequisites:** Docker & Docker Compose · Go 1.22+ · Node.js 20+

```bash
# 1. Start infrastructure (Postgres, Redis, ClickHouse, Kafka, Zookeeper)
docker-compose up -d

# 2. Build the browser snippet
cd snippet && npm install && npm run build

# 3. Build & run the detection server
cd ../server && go build -o bin/ivtd ./cmd/ivtd && ./bin/ivtd

# 4. Build & run the dashboard
cd ../dashboard && npm install && npm run dev
```

Then drop the loader into any page you want to protect:

```html
<script src="https://your-cdn.com/ivt-loader.min.js"></script>
<script>
  window.__ivt = window.__ivt || { q: [], t: Date.now(), v: '1.0.0' };
  window.__ivt.q.push(['config', {
    endpoint: 'https://your-server.com/api/v1/signals',
    apiKey: 'YOUR_API_KEY',
    probesUrl: 'https://your-cdn.com/ivt-probes.min.js'
  }]);
</script>
```

Full setup, API reference, and detector documentation are in the [system README](ivt/ivt-system-opensource/ivt-system/README.md).

---

## License

Open source. Read it, run it, break it, improve it.
