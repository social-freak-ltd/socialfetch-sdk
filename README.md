# @socialfetch/sdk

Official TypeScript SDK for the [Social Fetch](https://www.socialfetch.dev) public HTTP API. Call `/v1` endpoints with typed methods; responses use a lightweight `Result` (`ok` / `err`) so expected API failures are data, not thrown exceptions.

## Resources

| Resource | Link |
| --- | --- |
| Product | [socialfetch.dev](https://www.socialfetch.dev) |
| Documentation | [socialfetch.dev/docs](https://www.socialfetch.dev/docs) |
| Dashboard & API keys | [app.socialfetch.dev](https://app.socialfetch.dev) |
| Source & issues | [github.com/social-freak-ltd/socialfetch](https://github.com/social-freak-ltd/socialfetch) (`packages/sdk`) |

For questions or problems with the API or this package, open an [issue](https://github.com/social-freak-ltd/socialfetch/issues) or contact [support@socialfetch.dev](mailto:support@socialfetch.dev).

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

- **Node.js** `18+` (uses the built-in `fetch` unless you supply your own).

## Quick start

Create a client with your API key from the [dashboard](https://app.socialfetch.dev). The package wraps the documented public routes: top-level `health()`, plus `auth`, `billing`, `facebook`, `tiktok`, `twitter`, `instagram`, `linkedin`, and `youtube` resources. Use the [documentation](https://www.socialfetch.dev/docs/sdk.mdx) and [API reference](https://www.socialfetch.dev/docs/api.mdx) when you need exact route coverage or raw HTTP details.

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
const profile = unwrap(
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

const instagramComments = unwrap(
	await client.instagram.getPostComments({
		url: "https://www.instagram.com/p/ABC123/",
	}),
);

const youtubeChannel = unwrap(
	await client.youtube.getChannel({ handle: "MrBeast" }),
);
```

## Exported endpoint types

The SDK also exports request and response types for supported endpoints, so consumers can reuse the official types instead of recreating mirror interfaces in their own code.

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

This applies across the SDK for other exported endpoint shapes too, including platform-specific request params, response payloads, and shared error/result types.

Useful shared exports include:

- `Result<T, E>` plus `ok()` / `err()` for Result-style control flow.
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

## Errors

On `err`, the SDK exposes structured fields such as `code`, `requestId`, and HTTP metadata where applicable. See the docs for error semantics and retry guidance. You can also import `PUBLIC_API_ERROR_CODES` and `isSocialFetchSdkError` for typed handling.
