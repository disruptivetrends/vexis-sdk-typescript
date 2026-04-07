<p align="center">
  <h1 align="center">@vexis/sdk</h1>
  <p align="center">Official TypeScript SDK for the VEXIS AI Governance Platform</p>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@vexis/sdk"><img src="https://img.shields.io/npm/v/@vexis/sdk.svg?style=flat-square&color=cb3837" alt="npm version"></a>
  <a href="https://www.npmjs.com/package/@vexis/sdk"><img src="https://img.shields.io/npm/dm/@vexis/sdk.svg?style=flat-square" alt="npm downloads"></a>
  <a href="https://github.com/disruptivetrends/vexis-sdk-typescript/actions"><img src="https://img.shields.io/github/actions/workflow/status/disruptivetrends/vexis-sdk-typescript/ci.yml?style=flat-square" alt="CI"></a>
  <a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat-square" alt="License"></a>
  <a href="https://docs.vexis.io"><img src="https://img.shields.io/badge/docs-vexis.io-5A67D8?style=flat-square" alt="Documentation"></a>
</p>

---

Every AI interaction your application makes — governed, audited, and optionally anchored to the blockchain. In one line of code.

- **Zero dependencies** — uses native `fetch`, works in Node.js 18+, Deno, Bun, and Edge Runtimes
- **Multi-modal** — text, images, audio, documents, code
- **Enterprise-grade** — retry with exponential backoff, circuit breaker, typed errors
- **On-prem ready** — point to any VEXIS Gateway endpoint

## Installation

```bash
npm install @vexis/sdk
# or
pnpm add @vexis/sdk
# or
yarn add @vexis/sdk
```

## Quick Start

```typescript
import { Vexis } from '@vexis/sdk';

const vexis = new Vexis({ apiKey: process.env.VEXIS_API_KEY! });

// Verify any prompt before sending it to an LLM
const result = await vexis.verify({ prompt: userInput });

if (result.decision === 'BLOCKED') {
  console.error(`Blocked: ${result.reason}`);
  return;
}

// result.decision is 'ALLOWED' or 'MODIFIED'
// result.output contains the (possibly sanitized) text
// result.traceId links to the immutable audit trail
```

## API Reference

### Constructor

```typescript
const vexis = new Vexis({
  apiKey: 'gp_live_xxx',              // Required — project or agent API key
  baseUrl: 'https://gateway.vexis.io', // Custom endpoint for on-prem
  timeout: 30_000,                     // Request timeout in ms
  maxRetries: 3,                       // Retry attempts on transient failures
  retryBaseDelay: 500,                 // Base delay for exponential backoff
  headers: { 'X-Tenant': 'acme' },    // Custom headers on every request
  circuitBreakerThreshold: 5,          // Failures before circuit opens
  circuitBreakerCooldown: 30_000,      // Cooldown before half-open retry
});
```

### `verify(request)` — Core governance check

```typescript
const result = await vexis.verify({
  prompt: 'Transfer $50,000 to account DE89370400440532013000',
  metadata: { userId: 'u_123', department: 'finance' },
  attachments: [{
    contentType: 'application/pdf',
    data: base64EncodedPdf,
    filename: 'contract.pdf',
  }],
  context: {
    mcpServer: 'https://mcp.internal.corp',
    toolName: 'bank_transfer',
    chainDepth: 2,
    sourceSystem: 'crewai',
    sessionId: 'sess_abc',
  },
});
```

**Returns `VerifyResponse`:**

| Field | Type | Description |
|-------|------|-------------|
| `decision` | `'ALLOWED' \| 'BLOCKED' \| 'MODIFIED' \| 'ERROR'` | Governance decision |
| `output` | `string` | Sanitized output (PII redacted if MODIFIED) |
| `reason` | `string` | Human-readable explanation |
| `traceId` | `string` | Unique audit trail ID |
| `integrityHash` | `string` | SHA-256 hash for tamper detection |
| `shouldAnchor` | `boolean` | Whether trace will be anchored to Flare blockchain |
| `flareStatus` | `string` | `LOCAL_ONLY`, `PENDING`, `ANCHORED`, `SKIPPED`, `FAILED` |
| `flareTxHash` | `string \| null` | Blockchain transaction hash (after anchoring) |
| `contentType` | `string` | Detected content type |
| `findings` | `Finding[]` | Security findings (PII, secrets, policy violations) |
| `latencyMs` | `number` | Round-trip latency in milliseconds |

### `check(prompt)` — Quick text-only verification

```typescript
const { decision } = await vexis.check('Is this prompt safe?');
```

### `verifyWithFile(prompt, filePath)` — File attachment (Node.js only)

```typescript
const result = await vexis.verifyWithFile(
  'Analyze this document for compliance',
  './report.pdf'
);
```

### `listPolicies(env?)` — List active policies

```typescript
const { policies } = await vexis.listPolicies('prod');
```

### `health()` — Gateway health check

```typescript
const health = await vexis.health();
console.log(health.status); // 'healthy'
```

### `diagnostics()` — SDK diagnostics

```typescript
const diag = vexis.diagnostics();
// { sdkVersion, baseUrl, timeout, maxRetries, circuitState }
```

## Error Handling

All errors extend `VexisError` with structured metadata:

```typescript
import { VexisError, VexisRateLimitError } from '@vexis/sdk';

try {
  await vexis.verify({ prompt: input });
} catch (err) {
  if (err instanceof VexisRateLimitError) {
    // err.retryAfterMs — wait this long before retrying
    await sleep(err.retryAfterMs);
    return retry();
  }
  if (err instanceof VexisError) {
    console.error(err.code, err.statusCode, err.requestId);
  }
}
```

| Error Class | Code | Retryable | When |
|-------------|------|:---------:|------|
| `VexisAuthenticationError` | `AUTHENTICATION_FAILED` | No | Invalid or expired API key |
| `VexisRateLimitError` | `RATE_LIMITED` | Yes | Quota exceeded (includes `retryAfterMs`) |
| `VexisValidationError` | `VALIDATION_ERROR` | No | Malformed request (includes `field`) |
| `VexisTimeoutError` | `TIMEOUT` | Yes | Gateway didn't respond in time |
| `VexisCircuitOpenError` | `CIRCUIT_OPEN` | No | Too many consecutive failures |

## Framework Integration

VEXIS SDKs work with any LLM framework. Dedicated adapters with deeper integration are available:

| Framework | Package | Integration |
|-----------|---------|-------------|
| LangChain | `vexis-langchain` | `VexisCallbackHandler` — automatic governance on every LLM call |
| CrewAI | `vexis-crewai` | `VexisGovernance` plugin — task-level governance per crew |
| OpenAI Agents SDK | `vexis-openai-agents` | Middleware hook for the official OpenAI framework |
| Microsoft AGT | `vexis-agt-adapter` | Policy distribution from VEXIS → AGT local enforcement |
| Claude Code | `vexis-governance` | MCP-native governance for every tool call |

## On-Premise / Self-Hosted

Point to your internal VEXIS Gateway:

```typescript
const vexis = new Vexis({
  apiKey: process.env.VEXIS_API_KEY!,
  baseUrl: 'https://gateway.internal.acme.corp:8080',
  timeout: 10_000,
  maxRetries: 5,
});
```

## MCP Context (Agentic AI)

When your agent calls tools via MCP, pass the context for full audit trails:

```typescript
const result = await vexis.verify({
  prompt: 'Execute bank transfer',
  context: {
    mcpServer: 'https://banking-mcp.corp.internal',
    toolName: 'transfer_funds',
    chainDepth: 3,
    sourceSystem: 'crewai',
    sessionId: 'agent_session_42',
  },
});
```

## Requirements

- Node.js 18+ (uses native `fetch`)
- Also works in Deno, Bun, Cloudflare Workers, Vercel Edge

## Links

- [Documentation](https://docs.vexis.io/sdk/typescript)
- [API Reference](https://docs.vexis.io/api)
- [Dashboard](https://app.vexis.io)
- [GitHub](https://github.com/disruptivetrends/vexis-sdk-typescript)
- [Changelog](https://github.com/disruptivetrends/vexis-sdk-typescript/blob/main/CHANGELOG.md)

## License

[Apache 2.0](./LICENSE)