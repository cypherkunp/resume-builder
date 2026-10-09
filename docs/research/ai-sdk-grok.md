# AI SDK call to Grok with a per-request API key

Read on 2026-10-10 from the current unversioned docs and the current provider source. This note does not choose config-panel fields or where the person keeps the key in the app.

## Question

How does the AI SDK call Grok (xAI) with an API key the person supplied at runtime, and can that call return a structured review: a list of suggestions, each with proposed text and a comment?

Constraints already settled: the key is configured in the app by the person, there are no user accounts, and each suggestion is accepted or declined one at a time. The question here is only the provider, the model id, the structured-output API, and whether a TanStack Start server function can take the key on each request and call the provider without storing the key on the server.

## Answer

Yes. Install `@ai-sdk/xai` and `ai`. Build a provider with `createXai({ apiKey })` from the key on that request, call `generateText` with model id `grok-4.7`, and set `output` to `Output.object({ schema })`. `generateObject` is the old API: it is deprecated, and the current structured-data guide does not use it.

A Zod object whose `suggestions` array has a string for the proposed text and a string for the comment is inside the schema types both the AI SDK and xAI document. xAI lists structured outputs as a capability of `grok-4.7`.

A TanStack Start server function can receive that key in its `data` argument and pass it to `createXai` inside the handler. The handler runs only on the server. `createServerFn` does not document writing `data` to env, disk, or a database, so the function does not have to store the key. The key still crosses the network to the app server for that request. Use `method: 'POST'`. The default method is `GET`, and the current client puts a GET payload in the query string.

## Provider

Package: `@ai-sdk/xai`. The provider page says to install it with `pnpm add @ai-sdk/xai`.

The default instance reads the key from the environment:

```ts
import { xai } from '@ai-sdk/xai';
```

For a key supplied at runtime, the same page says to construct an instance:

```ts
import { createXai } from '@ai-sdk/xai';

const xai = createXai({
  apiKey: 'your-api-key',
});
```

Documented provider settings:

- `apiKey` (`string`): sent on the `Authorization` header. Defaults to the `XAI_API_KEY` environment variable.
- `baseURL` (`string`): default prefix `https://api.x.ai/v1`.
- `headers` and `fetch`: optional request customization. `fetch` is not a place the docs say to put the key.

`apiKey` is a provider-instance setting. The documented `providerOptions.xai` fields (`reasoningEffort`, `serviceTier`, `store`, and the rest listed on that page) do not include a per-call API key. Create the provider inside the request that has the key.

Since AI SDK 7, `xai(modelId)` uses the xAI Responses API. `xai.responses(modelId)` is the same API explicitly. `xai.chat(modelId)` is the legacy Chat Completions API.

xAI's Grok 4.7 page lists its API as the Responses API. It also says Chat Completions is unchanged for encrypted reasoning, and that prompt-cache routing on Chat Completions uses the `x-grok-conv-id` header. That page does not show a Chat Completions sample for `grok-4.7`. The AI SDK samples for this model use `xai('grok-4.7')` or `xai.responses('grok-4.7')`.

## Model id

Use `grok-4.7`.

- xAI's models page says that for everything except dedicated audio, image, and video models, including code and chat, use Grok 4.7. The model name in the pricing table is `grok-4.7`.
- The model page lists model name `grok-4.7`, text and image input, text output, a 500,000-token context window, and capabilities function calling, structured outputs, and reasoning. Reasoning efforts are `low`, `medium`, `high` (default), and `xhigh`.
- The current AI SDK xAI page uses `grok-4.7` as the example model id passed to `xai(...)` and `xai.responses(...)`.

The capabilities table on that AI SDK page does not have a `grok-4.7` row. It lists older ids (`grok-4.6`, `grok-4.5`, `grok-4.20-reasoning`, `grok-4.20-non-reasoning`, and others) with object generation checked, and it says the table is popular models only and that any provider model id string can be passed. Object generation for `grok-4.7` is stated by xAI (structured outputs: yes) and by xAI's own AI SDK sample, not by a row in that table.

`grok-4.7` fast is a different serving path. xAI says it is not available on the public xAI API. It is not the id to pass here.

## Structured output

Current API, from the AI SDK structured-data guide and the Output reference:

```ts
import { generateText, Output } from 'ai';
import { createXai } from '@ai-sdk/xai';
import { z } from 'zod';

const review = z.object({
  suggestions: z.array(
    z.object({
      proposedText: z.string().describe('Text proposed for this suggestion.'),
      comment: z.string().describe('Where the change belongs and why.'),
    }),
  ),
});

const xai = createXai({ apiKey });

const { output } = await generateText({
  model: xai('grok-4.7'),
  output: Output.object({
    name: 'Review',
    description: 'A list of suggestions for one resume.',
    schema: review,
  }),
  prompt,
});
```

`output` is the validated object. The property names above only show that the call can require a list of suggestions with proposed text and a comment. They are not a decision about stored fields.

`Output.object({ schema, name?, description? })` accepts a Zod schema, a Valibot schema, or a JSON schema. The result is checked against the schema. `.describe(...)` on a property is passed through as guidance to the model. `name` and `description` on the output are optional provider hints.

`Output.array({ element, minItems?, maxItems? })` is the other documented way to return a list. Use it if the whole result should be the array rather than an object that holds the array. Each completed element is validated. `minItems` and `maxItems` are sent to the provider when it supports them, and the SDK also checks the finished array.

xAI's structured-outputs page shows the same call with this provider: `generateText({ model: xai.responses('grok-4.7'), output: Output.object({ schema }) })`. That sample logs `result._output`. The AI SDK reference documents the return field as `output`, not `_output`. Use `output`.

xAI says that, for supported schema features, the response is guaranteed to match the schema. Supported types include `string`, `number`, `integer`, `boolean`, `null`, `enum`, `array`, and `object`. `additionalProperties` defaults to `false`. `minItems` and `maxItems` are guaranteed up to 256. A list of string fields fits those types. This note does not re-check how `@ai-sdk/xai` maps a particular Zod schema onto that subset.

If the model text cannot be parsed or does not match the schema, `generateText` rejects with `AI_NoObjectGeneratedError` (`NoObjectGeneratedError` in the import examples). If the call finishes with no output, reading `result.output` throws `AI_NoOutputGeneratedError`.

`generateObject` and `streamObject` were deprecated in AI SDK 6 (pull request 10754). The migration guide says to use `generateText` and `streamText` with `output`, and that the old functions will be removed in a future version. The current docs have no `generateObject` reference page. The function is still exported from `ai` on the `main` branch of `vercel/ai`, marked `@deprecated Use generateText with an output setting instead.` The current guide is `generateText` plus `Output.object`.

Streaming the same shape is `streamText` with the same `output`. The partial stream is `partialOutputStream`. A review that the person accepts or declines one suggestion at a time does not require streaming.

## Per-request API key

Documented path:

1. The person already has the key in the app. This note does not say which config fields hold it.
2. The caller passes the key in the server function `data` argument for that review.
3. The handler, which runs on the server, calls `createXai({ apiKey: data.apiKey })` and then `generateText`.
4. The provider sends that key on `Authorization` for the calls made with that instance.

Omitting `apiKey` makes the provider use `XAI_API_KEY` from the server environment. That is a key stored on the server. The runtime-supplied path does not need that variable.

The handler can return `output` and drop the key when the function returns. A plain object of strings is a serializable return value. Do not return the key.

Separate from the app server: on the Responses API, `providerOptions.xai.store` controls whether xAI stores the input messages and the model response for later retrieval. The AI SDK xAI page says it defaults to `true`. Set `store: false` to opt out. That flag is not the API key, and it is not app-server storage. A review prompt contains resume text, so the default retains that prompt at xAI.

## What a server function can and cannot do with the key

Official behavior, from the TanStack Start server-functions guide:

- `createServerFn({ method: 'POST' })` defines server-only logic. The client calls it like a function. The build replaces the implementation with an RPC stub, so the handler never ships to the browser. On the client, the call is a `fetch` to the app server.
- The function takes one `data` argument: `await reviewResume({ data: { apiKey, /* resume text */ } })`. The handler reads it as `{ data }`.
- A string key is a serializable input. Strict mode, the default, type-checks that validator inputs and handler results are serializable.
- The handler can call `createXai` and `generateText` and return the review object. Nothing in `createServerFn`, the validator, or the handler contract writes `data` to the environment, a file, or a database. Storing the key would be extra application code. The documented API does not require it.
- Server functions are same-origin RPC endpoints, reachable without going through the page that renders the button. If `src/start.ts` does not exist, Start installs `createCsrfMiddleware()` for server functions. That checks `Sec-Fetch-Site`, `Origin`, or `Referer`. It does not authenticate the caller, and it does not keep the key out of the request.
- Do not mark this function as a static server function. Those run at build time and cache the result.

The guide does not say where a non-form payload sits in the HTTP request. The current client in TanStack Router does. In `packages/start-client-core/src/client-rpc/serverFnFetcher.ts` on `main` (read 2026-10-10, not a release tag):

- `GET` serializes `{ data }` and appends it to the URL query string. The file comment says the payload is moved to the query string. `createServerFn()` with no method defaults to `GET`.
- `POST` sends that payload as the request body and sets `content-type` to `application/json` when the body is a string.

A key on a GET server function would be in the URL. Use `POST` so it is in the body instead. The body still reaches the app server. "Without storing the key on the server" means the function does not have to persist it. It does not mean the key never arrives there.

Not claimed, because no primary source here says so:

- That access logs, error logs, or the host retain or do not retain the POST body.
- That a CDN caches POST server-function responses by default. The guide does say not to send `Cache-Control: public` for a response that depends on who called, and it shows `Cache-Control: no-store` for sensitive data. A review derived from one person's resume and key is not public shared data.

## Sources

- AI SDK xAI provider: https://ai-sdk.dev/providers/ai-sdk-providers/xai
- AI SDK generating structured data: https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data
- AI SDK `Output` reference: https://ai-sdk.dev/docs/reference/ai-sdk-core/output
- AI SDK 5 to 6 migration (`generateObject` deprecation): https://ai-sdk.dev/docs/migration-guides/migration-guide-6-0
- `generateObject` still exported and deprecated in the `ai` package: https://github.com/vercel/ai/blob/main/packages/ai/src/generate-object/generate-object.ts
- xAI models, including the recommendation to use Grok 4.7: https://docs.x.ai/docs/models
- Grok 4.7 model page (`grok-4.7`, structured outputs): https://docs.x.ai/developers/models/grok-4.7
- Grok 4.7 developer guide (Responses API): https://docs.x.ai/developers/grok-4-7
- xAI structured outputs, including the AI SDK sample: https://docs.x.ai/developers/model-capabilities/text/structured-outputs
- TanStack Start server functions: https://tanstack.com/start/latest/docs/framework/react/guide/server-functions
- TanStack Start client RPC (GET query string, POST body): https://github.com/TanStack/router/blob/main/packages/start-client-core/src/client-rpc/serverFnFetcher.ts
