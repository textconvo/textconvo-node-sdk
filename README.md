<h1 align="center">TextConvo SDK for Node.js</h1>

<p align="center"><strong>Official Node.js and TypeScript SDK for the TextConvo API.</strong></p>

<p align="center">
  <a href="https://textconvo.ai">Website</a> &nbsp;&middot;&nbsp;
  <a href="https://textconvo.ai/docs">Developer Docs</a> &nbsp;&middot;&nbsp;
  <a href="https://textconvo.ai/docs#lead-ingestion-api">API Reference</a> &nbsp;&middot;&nbsp;
  <a href="https://textconvo.ai/contact-us">Support</a>
</p>

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/status-in%20development-db6d28?style=flat-square">
  <a href="https://textconvo.ai/docs"><img alt="Docs" src="https://img.shields.io/badge/docs-textconvo.ai%2Fdocs-1f6feb?style=flat-square"></a>
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT-2ea043?style=flat-square"></a>
  <img alt="Node" src="https://img.shields.io/badge/node-18%2B-339933?style=flat-square">
</p>

---

> **Not released yet.** This repository is the public home of the official Node.js SDK. It currently holds the intended surface, the roadmap, and nothing else. There is no package on npm and no implementation in `src/` — that is deliberate rather than an oversight.
>
> **Use today:** [textconvo-api-examples](https://github.com/textconvo/textconvo-api-examples) has production-shaped Node.js and TypeScript examples with retry, idempotency, and HMAC signing. Or generate a typed client from [textconvo-openapi](https://github.com/textconvo/textconvo-openapi).
>
> Watch this repository to hear about the first release.

## Why an SDK when the API is two calls?

Because the boring parts are where integrations break. The SDK will own idempotency keys, retry classification, backoff with jitter, timeouts, HMAC signing, and webhook verification, so your code does not have to.

## Planned installation

```bash
npm install @textconvo/sdk
```

## Planned usage

The shape below is what we are building towards. Treat it as a design sketch, not a contract, until 1.0.

```ts
import { TextConvo } from '@textconvo/sdk';

const textconvo = new TextConvo({
  apiKey: process.env.TEXTCONVO_API_KEY!,
  sourceKey: process.env.TEXTCONVO_SOURCE_KEY!,
  // Optional, only when signing is enabled for your source
  hmacSecret: process.env.TEXTCONVO_HMAC_SECRET
});

// Idempotency, retries, and backoff are handled for you.
const accepted = await textconvo.leads.ingest({
  phone: '+15035551234',
  firstName: 'Jane',
  lastName: 'Doe',
  email: 'jane.doe@example.com',
  customFields: { roofAgeYears: '12' }
});

console.log(accepted.ingestionRequestId, accepted.duplicate);
```

```ts
// Webhook verification, framework agnostic
import { verifyWebhook } from '@textconvo/sdk/webhooks';

app.post('/webhooks/textconvo', express.raw({ type: 'application/json' }), (req, res) => {
  const event = verifyWebhook({
    rawBody: req.body,
    signature: req.get('X-TextConvo-Signature'),
    timestamp: req.get('X-TextConvo-Timestamp'),
    secret: process.env.TEXTCONVO_WEBHOOK_SECRET!
  });

  res.sendStatus(200);
  queue.push(event);
});
```

## Planned structure

```
textconvo-node-sdk/
├─ src/
│  ├─ index.ts            # TextConvo client entry point
│  ├─ client.ts           # transport: auth, timeouts, retries, backoff
│  ├─ errors.ts           # typed error hierarchy, retryable vs terminal
│  ├─ resources/
│  │  └─ leads.ts         # leads.ingest()
│  ├─ webhooks/
│  │  ├─ verify.ts        # signature and timestamp verification
│  │  └─ events.ts        # discriminated union of event types
│  └─ types.ts            # generated from textconvo-openapi
├─ test/
├─ examples/
└─ docs/
```

## Design commitments

**Zero runtime dependencies.** Node 18+ has fetch and crypto. We will not pull in a HTTP stack.

**Types generated from the specification.** [textconvo-openapi](https://github.com/textconvo/textconvo-openapi) is the source; hand-written types drift.

**Safe by default.** Idempotency keys generated automatically, retries only for 429 and 5xx, exponential backoff with jitter, constant-time signature comparison, and secrets read from the environment.

**Errors you can branch on.** A typed hierarchy rather than string matching on messages.

**ESM and CommonJS.** Both, with correct type exports.

**Semantic versioning.** 0.x while the surface moves, 1.0 when it stops.

## Roadmap

| Milestone | Contents | Status |
| --- | --- | --- |
| 0.1.0 &mdash; Alpha | Client, auth, `leads.ingest`, typed errors, retries, HMAC signing | Planned |
| 0.2.0 &mdash; Webhooks | Verification helper, typed event union, framework adapters | Planned |
| 0.3.0 &mdash; Ergonomics | Structured logging hooks, request instrumentation, pagination helper | Planned |
| 0.4.0 &mdash; Beta | Full test suite, docs site, examples for Express, Fastify, Next.js | Planned |
| 1.0.0 &mdash; GA | Stable surface, semantic-versioning guarantees, published on npm | Planned |
| Post-1.0 | Channel-send operations, contact retrieval, and message status as those endpoints ship | Planned |

See [coverage](https://github.com/textconvo/textconvo-api-examples/blob/main/docs/COVERAGE.md) for what the API supports today.

## See it live

Submit the [contact form](https://textconvo.ai/contact-us) and you get a direct line to **Ria**, the TextConvo AI orchestrator &mdash; call her for a live voice demo, or text her and watch the SMS AI reply in real time. A human follows up within one business day, and the same form is how API credentials, a source key, and a webhook secret are issued.

Handed a TextConvo QR code at an event or in a demo? Scanning it opens the same conversation. The form is simply the path that works for everyone.

## Feedback wanted, before the code exists

This is the cheapest moment to change the design. Tell us:

- Does the method naming read naturally in your codebase?
- Which framework adapter do you need first?
- Do you want promises only, or an event-emitter style for webhook handling?

[Open an issue](https://github.com/textconvo/textconvo-node-sdk/issues/new/choose) — opinions now are worth more than pull requests later.

## Contributing

We are not accepting implementation pull requests until the alpha surface is agreed. Design feedback, documentation fixes, and roadmap discussion are very welcome. See [CONTRIBUTING.md](https://github.com/textconvo/.github/blob/main/CONTRIBUTING.md).

## Security

[SECURITY.md](https://github.com/textconvo/.github/blob/main/SECURITY.md) — never open a public issue for a vulnerability.

## License

[MIT](LICENSE) &copy; TextConvo
