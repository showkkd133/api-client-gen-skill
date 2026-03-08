# api-client-gen-skill

A Claude Code skill that generates TypeScript API clients from OpenAPI/Swagger specifications. Zero config, zero dependencies (unless you choose axios).

## What it does

Point it at an OpenAPI spec (URL, file path, or pasted YAML/JSON) and it generates:

- **TypeScript types** -- all request/response interfaces from `components.schemas`
- **HTTP client** -- fetch-based (zero deps) or axios-based, with auth, retries, and interceptors
- **API functions** -- one typed function per endpoint, organized by tags
- **React Query hooks** (optional) -- `useQuery` for GET, `useMutation` for POST/PUT/DELETE, with query key factories and cache invalidation
- **SWR hooks** (optional) -- `useSWR` and `useSWRMutation` wrappers

### Features

| Feature | Details |
|---------|---------|
| Type generation | Full OpenAPI-to-TypeScript mapping including enums, oneOf/allOf, nullable, nested objects |
| Auth support | Bearer token, API key (header/query), Cookie, OAuth2 with token refresh |
| Error handling | Typed `ApiClientError` with status, body, url, method |
| Retry logic | Configurable retry count and delay, only retries on 5xx/network errors |
| Interceptors | `onRequest` and `onResponse` hooks for custom logic |
| Code style | Auto-adapts to your project's tsconfig, ESLint, Prettier, and Biome config |
| File organization | Types in one file, API functions split by tag, hooks split by tag |

## Installation

### Claude Code (recommended)

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "skills": {
    "api-client-gen": {
      "source": "github:showkkd133/api-client-gen-skill"
    }
  }
}
```

Or copy `skills/api-client-gen/SKILL.md` to `~/.claude/skills/api-client-gen/SKILL.md`.

### Manual

```bash
git clone https://github.com/showkkd133/api-client-gen-skill.git
cp skills/api-client-gen/SKILL.md ~/.claude/skills/api-client-gen/SKILL.md
```

## Usage

In Claude Code, say any of:

- "generate api client from https://petstore.swagger.io/v2/swagger.json"
- "generate api client" (will look for openapi.yaml in your project)
- "openapi client"
- "from swagger generate client"

### What happens

1. Fetches and validates the OpenAPI spec
2. Reads your project's tsconfig/eslint/prettier config for code style
3. Detects existing dependencies (axios, react-query, swr)
4. Generates `types.ts` with all interfaces
5. Generates `client.ts` with HTTP client (fetch or axios)
6. Generates `api/*.ts` with typed API functions per tag
7. Optionally generates `hooks/*.ts` with React Query or SWR hooks
8. Runs `tsc --noEmit` to verify everything compiles

### Output structure

```
src/api-client/
  index.ts          # re-exports everything
  client.ts         # HTTP client with auth, retry, interceptors
  types.ts          # all TypeScript interfaces and enums
  api/
    users.ts        # API functions grouped by tag
    orders.ts
    auth.ts
  hooks/            # optional
    useUsers.ts     # React Query or SWR hooks
    useOrders.ts
    useAuth.ts
```

## Type Mapping

| OpenAPI | TypeScript |
|---------|-----------|
| `string` | `string` |
| `string` + `enum` | `'a' \| 'b' \| 'c'` literal union |
| `integer` / `number` | `number` |
| `boolean` | `boolean` |
| `array` | `T[]` |
| `object` | `interface` |
| `$ref` | reference to interface |
| `oneOf` / `anyOf` | `A \| B` union |
| `allOf` | `A & B` intersection |
| `nullable: true` | `T \| null` |
| `string` + `binary` | `File \| Blob` |

## Auth Support

| Method | How it works |
|--------|-------------|
| Bearer token | `getToken` callback in client config |
| API key (header) | `apiKey` in client config |
| API key (query) | auto-appended to URL params |
| Cookie/session | `credentials: 'include'` via `onRequest` |
| OAuth2 | `getToken` with auto-refresh logic |

## License

MIT
