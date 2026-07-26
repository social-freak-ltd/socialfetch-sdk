# @socialfetch/sdk

[![npm version](https://img.shields.io/npm/v/@socialfetch/sdk.svg)](https://www.npmjs.com/package/@socialfetch/sdk)
[![Node.js](https://img.shields.io/node/v/@socialfetch/sdk.svg)](https://www.npmjs.com/package/@socialfetch/sdk)
[![License](https://img.shields.io/npm/l/@socialfetch/sdk.svg)](https://www.npmjs.com/package/@socialfetch/sdk)
[![Docs](https://img.shields.io/badge/docs-socialfetch.dev-0ea5e9)](https://www.socialfetch.dev/docs/sdk)

Official **TypeScript SDK** for [Social Fetch](https://www.socialfetch.dev) — a **social media scraper API** and **social data API** that returns live, normalized JSON for public profiles, posts, comments, transcripts, search, and metrics.

Call `/v1` endpoints with typed methods. Responses use a lightweight `Result` (`ok` / `err`) so expected API failures are data, not thrown exceptions.

**Get started:** [Docs](https://www.socialfetch.dev/docs) · [SDK guide](https://www.socialfetch.dev/docs/sdk) · [API reference](https://www.socialfetch.dev/docs/api) · [Pricing](https://www.socialfetch.dev/pricing) · [Dashboard & API keys](https://app.socialfetch.dev)

## Why Social Fetch

- **One API, 20+ platforms** — TikTok, Instagram, YouTube, X/Twitter, LinkedIn, Facebook, Reddit, Threads, Telegram, GitHub, Spotify, and more through a single client.
- **Live public data** — profiles, posts, reels, comments, video transcripts, hashtag search, engagement metrics, and ad-library lookups.
- **Typed TypeScript client** — generated from the public OpenAPI spec; request/response types ship with the package.
- **Result-based errors** — stable `code` + `requestId` for support and retries; optional `unwrap()` when you prefer thrown errors.
- **Pay-as-you-go credits** — no required subscription; [100 free credits](https://www.socialfetch.dev/pricing) on signup.

## Supported platforms

| Platform | SDK resource | Common use cases |
| --- | --- | --- |
| TikTok | `client.tiktok` | Profiles, videos, comments, shop / product data |
| Instagram | `client.instagram` | Profiles, posts, comments, hashtag & profile search |
| YouTube | `client.youtube` | Channels, videos, transcripts |
| X (Twitter) | `client.twitter` | Profiles, tweets / timeline |
| LinkedIn | `client.linkedin` | Profiles and public company / people data |
| Facebook | `client.facebook` | Pages, posts, comments, transcripts |
| Reddit | `client.reddit` | Subreddits, posts, comments |
| Threads | `client.threads` | Profiles and posts |
| Telegram | `client.telegram` | Public channel / message lookups |
| GitHub | `client.github` | Profiles, repositories |
| Spotify | `client.spotify` | Artist / track style public data |
| Google | `client.google` | Ad library advertisers & ads |
| Bluesky, Twitch, Pinterest, Rumble, Truth Social, Linktree, Hacker News, Web | matching `client.*` resources | Platform-specific + generic web extraction |

Full route coverage: [platforms](https://www.socialfetch.dev/platforms) · [API reference](https://www.socialfetch.dev/docs/api).

## Install

```bash
npm install @socialfetch/sdk
```

```bash
pnpm add @socialfetch/sdk
```

```bash
yarn add @socialfetch/sdk
```

## Requirements

- **Node.js** `18+` with a global `fetch` (the client also accepts a custom `fetch` implementation).
- **ESM-only package**. Use `import` rather than `require()`.

## Quick start

Create a client with your API key from the [dashboard](https://app.socialfetch.dev). The package wraps documented public routes: top-level `health()` and `ask()`, plus `auth`, `billing`, and per-platform resources (`tiktok`, `instagram`, `youtube`, `twitter`, and others).

```ts
import { SocialFetchClient, unwrap } from "@socialfetch/sdk";

const client = new SocialFetchClient({
	apiKey: process.env.SOCIALFETCH_API_KEY!,
});

const whoami = await client.auth.whoami();
if (!whoami.ok) {
	console.error(whoami.error.code, whoami.error.requestId);
} else {
	console.log(whoami.value.data.user);
}

// Optional: throw on failure (unexpected errors can still throw)
const tiktokProfile = unwrap(
	await client.tiktok.getProfile({ handle: "charlidamelio" }),
);

const twitterProfile = unwrap(
	await client.twitter.getProfile({ handle: "elonmusk" }),
);
const twitterTweets = unwrap(
	await client.twitter.getProfileTweets({ handle: "elonmusk" }),
);

const video = unwrap(
	await client.tiktok.getVideo({
		url: "https://www.tiktok.com/@nike/video/7587811642650545421",
	}),
);

const instagram = unwrap(
	await client.instagram.getProfile({ handle: "instagram" }),
);

const facebookPage = unwrap(
	await client.facebook.getProfile({
		url: "https://www.facebook.com/mantraindianfolsom",
	}),
);

const linkedinProfile = unwrap(
	await client.linkedin.getProfile({ handle: "marclouvion" }),
);

const youtubeChannel = unwrap(
	await client.youtube.getChannel({ handle: "MrBeast" }),
);

const githubProfile = unwrap(
	await client.github.getProfile({ handle: "torvalds" }),
);
```

Natural-language routing (one-shot lookup, not a chat):

```ts
const answer = unwrap(
	await client.ask({
		query: "How many TikTok followers does MrBeast have?",
	}),
);
```

## Resources

| Resource | Link |
| --- | --- |
| Product | [socialfetch.dev](https://www.socialfetch.dev) |
| Documentation | [socialfetch.dev/docs](https://www.socialfetch.dev/docs) |
| SDK guide | [socialfetch.dev/docs/sdk](https://www.socialfetch.dev/docs/sdk) |
| API reference | [socialfetch.dev/docs/api](https://www.socialfetch.dev/docs/api) |
| Pricing & credits | [socialfetch.dev/pricing](https://www.socialfetch.dev/pricing) |
| Platforms | [socialfetch.dev/platforms](https://www.socialfetch.dev/platforms) |
| Dashboard & API keys | [app.socialfetch.dev](https://app.socialfetch.dev) |
| npm package | [npmjs.com/package/@socialfetch/sdk](https://www.npmjs.com/package/@socialfetch/sdk) |
| Issues & support | [github.com/social-freak-ltd/socialfetch-sdk](https://github.com/social-freak-ltd/socialfetch-sdk) |

For questions or problems with the API or this package, open an [issue](https://github.com/social-freak-ltd/socialfetch-sdk/issues) or contact [support@socialfetch.dev](mailto:support@socialfetch.dev).

## Exported endpoint types

The SDK exports request and response types for the current public methods, so consumers can reuse the official shapes directly from `@socialfetch/sdk` instead of recreating mirror interfaces in their own code.

Common root exports include auth and billing responses such as `WhoamiResponse` and `BalanceResponse`, plus resource-specific aliases such as `InstagramProfileResponse`, `TikTokVideoCommentsResponse`, `TikTokProfileShowcaseProductsResponse`, `TikTokShopProductSearchResponse`, `TikTokShopProductsResponse`, `ListShopProductsParams`, `TwitterProfileTweetsResponse`, `FacebookPostTranscriptResponse`, `WebSearchResponse`, and `YouTubeVideoResponse`.

For example, if you're working with Instagram profile routes you can import both the params and response types directly from `@socialfetch/sdk`:

```ts
import type {
	GetInstagramProfileParams,
	InstagramProfileResponse,
} from "@socialfetch/sdk";

const params: GetInstagramProfileParams = {
	handle: "instagram",
};

function renderProfile(response: InstagramProfileResponse) {
	return response.data.username;
}
```

Auth and billing responses are also available from the package root:

```ts
import type { BalanceResponse, WhoamiResponse } from "@socialfetch/sdk";

function formatAccountSummary(
	whoami: WhoamiResponse,
	balance: BalanceResponse,
) {
	return {
		userId: whoami.data.user.id,
		remainingCredits: balance.data.balance,
	};
}
```

This applies across the SDK for other exported endpoint shapes too, including platform-specific request params, response payloads, and shared error/result types.

Useful shared exports include:

- `Result<T, E>` plus `ok()` / `err()` for Result-style control flow.
- `ExtractResultValue<R>` and `ExtractResultError<R>` for deriving the success or error side from a result-returning SDK method.
- `SocialFetchSdkError` for the normalized SDK error shape.
- `PublicApiErrorCode` and `PUBLIC_API_ERROR_CODES` for working with stable API error codes.
- `isSocialFetchSdkError` for narrowing unknown caught values.
- `SocialFetchUnwrapError` when using `unwrap()` in `try` / `catch`.
- `SocialFetchClientConfig` and `HealthResponse` for client setup and health checks.

### Result pattern example

If you want to keep expected failures as data instead of throwing, annotate your application helpers with the SDK's exported `Result` and error types:

```ts
import type {
	GetInstagramProfileParams,
	InstagramProfileResponse,
	Result,
	SocialFetchSdkError,
} from "@socialfetch/sdk";
import { SocialFetchClient } from "@socialfetch/sdk";

const client = new SocialFetchClient({
	apiKey: process.env.SOCIALFETCH_API_KEY!,
});

async function loadInstagramProfile(
	params: GetInstagramProfileParams,
): Promise<Result<InstagramProfileResponse, SocialFetchSdkError>> {
	return client.instagram.getProfile(params);
}
```

### Result type extraction example

If you want to derive types from SDK methods directly, use `ExtractResultValue<R>` and `ExtractResultError<R>`. They accept the method's `ReturnType` and handle the async `Promise<Result<...>>` shape for you:

```ts
import type {
	ExtractResultError,
	ExtractResultValue,
	SocialFetchClient,
} from "@socialfetch/sdk";

export type SocialFetchTikTokVideoResponse = ExtractResultValue<
	ReturnType<SocialFetchClient["tiktok"]["getVideo"]>
>;

export type SocialFetchTikTokVideoError = ExtractResultError<
	ReturnType<SocialFetchClient["tiktok"]["getVideo"]>
>;
```

### Error handling example

You can branch on the normalized error shape without inventing your own error interface:

```ts
import {
	PUBLIC_API_ERROR_CODES,
	type SocialFetchSdkError,
} from "@socialfetch/sdk";

function getRetryMessage(error: SocialFetchSdkError) {
	if (error.kind === "api") {
		const isKnownCode = (PUBLIC_API_ERROR_CODES as readonly string[]).includes(
			error.code,
		);

		if (isKnownCode && error.code === "temporarily_unavailable") {
			return `The API is temporarily unavailable. Request ID: ${error.requestId}`;
		}
	}

	if (error.kind === "client" && error.code === "network_error") {
		return "Network error. Please try again.";
	}

	return error.message;
}
```

### `unwrap()` example

If you prefer thrown errors at your application boundary, catch `SocialFetchUnwrapError` and read the typed `detail` field:

```ts
import {
	SocialFetchClient,
	SocialFetchUnwrapError,
	unwrap,
} from "@socialfetch/sdk";

const client = new SocialFetchClient({
	apiKey: process.env.SOCIALFETCH_API_KEY!,
});

try {
	const profile = unwrap(
		await client.instagram.getProfile({ handle: "instagram" }),
	);

	console.log(profile.data.username);
} catch (error) {
	if (error instanceof SocialFetchUnwrapError) {
		console.error(error.detail.code, error.detail.requestId);
	}
}
```

## Configuration

- **`apiKey`** (required) — API key from the dashboard, sent as the `x-api-key` header.
- **`baseUrl`** (optional) — API origin without a trailing path, e.g. `https://api.socialfetch.dev`. Defaults to production.
- **`fetch`** (optional) — Custom `fetch` implementation for custom Node.js and test environments.
- **`retryOnRetryAfter`** (optional, default off) — When `true` or `{ maxRetries?, maxWaitMs? }`, honor `Retry-After` on HTTP 503 / 429 and retry the same request (bulk scrapers). Leave off for latency-sensitive UIs.

The client sends its package version in the `User-Agent` header (for example `socialfetch-typescript-sdk/0.13.0`) so Social Fetch can measure SDK adoption and support compatibility. This is not separate telemetry — it is part of normal API requests.

When your installed SDK is behind the version bundled with the API deployment you are calling, responses may include:

- `X-SocialFetch-SDK-Latest` — recommended TypeScript SDK version for that API
- `X-SocialFetch-SDK-Update: recommended` — upgrade when convenient (requests are not blocked)

Compare with `SDK_VERSION` from `@socialfetch/sdk/version`, or check [npm releases](https://www.npmjs.com/package/@socialfetch/sdk) and the [SDK changelog](https://github.com/social-freak-ltd/socialfetch-sdk/releases).

## Errors

On `err`, the SDK exposes structured fields such as `code`, `requestId`, and HTTP metadata where applicable. See the docs for error semantics and retry guidance. You can also import `PUBLIC_API_ERROR_CODES` and `isSocialFetchSdkError` for typed handling.

## Related

- **Product:** [Social Fetch social media data API](https://www.socialfetch.dev)
- **Integrations:** [MCP / agent skills](https://www.socialfetch.dev/docs/integrations/skills) · [n8n node](https://www.npmjs.com/package/n8n-nodes-socialfetch)
- **Support:** [GitHub Issues](https://github.com/social-freak-ltd/socialfetch-sdk/issues) · [support@socialfetch.dev](mailto:support@socialfetch.dev)

> This repository is the public home for `@socialfetch/sdk` docs, releases, and feedback. Install from [npm](https://www.npmjs.com/package/@socialfetch/sdk); use Issues for bug reports and feature requests.
