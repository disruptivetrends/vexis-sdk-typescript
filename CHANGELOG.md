# Changelog

All notable changes to `@vexis/sdk` will be documented in this file.

This project follows [Semantic Versioning](https://semver.org/).

## [0.5.0] — 2026-04-07

### Added
- Framework integration hints in README (LangChain, CrewAI, OpenAI Agents, Microsoft AGT, Claude Code)
- MCP context documentation and examples for agentic AI workflows
- Publish-ready README with badges for npmjs.com

### Changed
- Version bump from 0.4.0 to 0.5.0 (SDK Publishing milestone — Sprint 21)

## [0.4.0] — 2026-03-15

### Added
- Multi-modal attachments (image, audio, video, document, code)
- `verifyWithFile()` convenience method (Node.js, auto MIME detection)
- `RequestContext` for MCP/agentic context (`mcpServer`, `toolName`, `chainDepth`, `sourceSystem`, `sessionId`)
- Circuit breaker with configurable threshold and cooldown
- `VexisCircuitOpenError` for circuit breaker state
- `diagnostics()` method for runtime introspection
- Custom headers support for proxy/auth scenarios

### Changed
- Retry logic now uses exponential backoff with jitter (was linear)
- Request ID format changed to `vx_{timestamp}_{random}`

## [0.3.0] — 2026-02-01

### Added
- `check()` convenience method for text-only verification
- `listPolicies()` endpoint
- `health()` endpoint
- Typed error hierarchy (`VexisAuthenticationError`, `VexisRateLimitError`, `VexisValidationError`, `VexisTimeoutError`)
- `retryAfterMs` on rate limit errors

## [0.2.0] — 2026-01-15

### Added
- Retry with configurable max attempts
- Custom `baseUrl` for on-premise deployments
- `metadata` field on verify requests
- Request ID tracking (`X-Request-ID` header)

## [0.1.0] — 2025-12-01

### Added
- Initial release
- `verify()` method with `VerifyRequest` / `VerifyResponse`
- Bearer token authentication
- Basic error handling