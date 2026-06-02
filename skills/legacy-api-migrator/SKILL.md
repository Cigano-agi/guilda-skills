---
name: legacy-api-migrator
description: Migrates legacy third-party API/provider integrations from one stack to another (e.g. Node/NestJS → .NET, or any framework A → framework B) and scaffolds brand-new provider adapters from API docs. Reads an existing integration or provider documentation, extracts auth/endpoints/contracts/status-maps, and generates all adapter files following your target architecture exactly — preserving behavior, not improving it. Includes a human review gate on the status map before any file is written.
---

# Legacy API Migrator

Generates provider/integration adapters in a target stack by reading either an existing legacy integration (source of truth) or the provider's API documentation. The goal is mechanical, faithful porting of many third-party integrations without drifting from the house architecture.

## Commands

Parse the user's input to determine the mode:

| Input pattern | Mode |
|---|---|
| `migrate <PROVIDER>` | MIGRATE — read legacy adapter, generate target-stack equivalent |
| `new <URL or description>` | NEW — read provider docs, generate adapter from scratch |
| `validate <PROVIDER>` | VALIDATE — audit status-map coverage of an existing target adapter |
| `list` | LIST — show providers that exist in the legacy stack but are missing in the target |
| (no args) | INTERACTIVE — ask which mode and which provider |

## Configuration — EDIT BEFORE USE

Define these paths up front (do not ask the user repeatedly). Verify each exists before proceeding; if one is missing, stop and ask for the correct path.

```
LEGACY_REPO    = <path to the source/legacy codebase>
TARGET_REPO    = <path to the destination codebase>
DOCS_LIBRARY   = <optional: path to curated reference docs — glossary, enums, architecture notes>
```

## Target Project Structure (MUST follow exactly)

Map out, once, where each generated artifact lands in the target repo, then make every generation conform. A typical layered/clean architecture splits the work into:

```
target-repo/
├── Application layer/
│   ├── Constants            ← add provider slug constant
│   ├── Options/<Provider>Options          ← NEW: typed config binding
│   └── Credentials/<Provider>CredentialsResolver  ← NEW: resolves per-provider creds
├── Infrastructure layer/
│   └── Providers/<Provider>/              ← NEW DIRECTORY
│       ├── <Provider>Constants            ← endpoints, headers, statuses, messages
│       ├── <Provider>StrategyService      ← the adapter itself
│       └── Contracts/
│           ├── <Provider>Request          ← request DTO
│           └── <Provider>Response          ← response DTO
└── API / composition root/
    ├── DependencyInjection                ← EDIT (register client + adapter + resolver)
    ├── Startup / Program                  ← EDIT (bind Options)
    └── appsettings / config               ← EDIT (add config section)
```

Adapt the exact folder names to the target's conventions — the principle is: contracts, adapter, constants, options, credentials, and three registration edits.

## MIGRATE Mode — Step by Step

### Phase 0: Preconditions (optional but recommended)
Confirm the target toolchain is installed and the target repo already builds clean before generating anything. Abort early with a clear message if it does not — otherwise the final build check is meaningless.

### Phase 1: Read the legacy source
Locate and read the ENTIRE legacy adapter for the provider — strategy/service file plus any interface or type definitions and any payload normalizer. This is the source of truth. Read it; do not guess.

If `DOCS_LIBRARY` exists, also read the relevant architecture/glossary/enum notes so generation matches house conventions. Treat its absence as graceful degradation, not an error.

### Phase 2: Analyze the legacy adapter
Extract these facts from the actual code (never assume):

| Fact | Where to find it |
|---|---|
| **Auth type** | init/setup — `Bearer`, `Basic`, `x-api-key`, OAuth token exchange, mTLS cert |
| **Endpoints** | HTTP calls in each method — extract URL paths |
| **Supported methods** | which methods have a REAL implementation vs FAILED stubs |
| **Amount/number format** | passthrough vs unit conversion (e.g. ÷100) — read the actual code |
| **Status map** | the status-mapping method or inline switch/map |
| **Request shape** | which fields the provider expects |
| **Response shape** | which fields come back; how id / token / status are extracted |
| **Webhook/callback pattern** | how the callback URL is constructed |
| **Special behaviors** | UUID generation, splits, extra headers, customer fields |

### Phase 3: Select the base class / pattern
Pick the target adapter base type by matching the auth pattern observed in the legacy `init`:

| Auth pattern in legacy code | Target base |
|---|---|
| `Bearer <token>` or `x-api-key` header | simple token base |
| OAuth2 (client_id + client_secret → access_token) | OAuth base |
| Basic auth (base64 user:pass) | basic-auth base |
| mTLS certificate + OAuth | OAuth base + certificate handler |

When unsure, read the actual base classes in the target repo before deciding.

### Phase 4: Generate files
Generate the adapter files, matching the EXACT style of an existing reference adapter. Prefer the **most recently modified** provider in the target repo as the primary style reference (read its git log); use the simplest existing provider as a fallback for minimal structure. House style — naming, layout, constant usage — comes from the latest merge, not the oldest example.

Produce these artifacts:
- **`<Provider>Constants`** — slug constant, endpoint paths, webhook secret/template, auth header name, user-facing messages (invalid-credentials, unknown-error), all status string constants, log messages, defaults. Nothing inline anywhere else.
- **`<Provider>StrategyService`** — the adapter. Inherit the correct base, declare supported keys, override init (validate creds, set base address + auth header) and execute (build request, call, parse response, return the canonical result). Use the constants — never inline strings. Log HTTP error responses. Do NOT add try/catch, tracing spans, or initialized-checks the base class already provides; do NOT carry over framework decorators from the source.
- **`<Provider>Request`** / **`<Provider>Response`** — DTOs whose serialized property names match the provider's API EXACTLY; class/property casing follows target conventions. Include all response fields present in the legacy response type.
- **`<Provider>Options`** — typed config bound to a config section; mark mandatory fields required; include only the fields this provider actually needs.
- **`<Provider>CredentialsResolver`** — lists supported slugs, resolves credentials, returns the canonical credentials record.

### Phase 5: HUMAN GATE — Status Map Review
**Before writing any files**, present the extracted status map for confirmation:

```
## GATE: Status Map Review for <PROVIDER>

Legacy status map:
  "pending" → Pending
  "paid"    → Paid
  "failed"  → Failed
  (unknown) → Failed

Confirm these mappings, or adjust before I write the files.
```

Ask the user to confirm. Only proceed after approval.

### Phase 6: Write files + registration
1. Create the provider directory.
2. Write all adapter files.
3. Show the registration snippets to add to existing files: the constants/slug entry, the DI registration (client + adapter + resolver), the Options binding, and the config section.
4. Ask whether to apply the registrations automatically or just show them as snippets.

### Phase 7: Verification
Build the target repo to confirm it compiles, report the result, and summarize what was generated. If the build fails: check whether the repo built before, then suspect a missing registration (DI / startup / config) or a namespace mismatch.

## NEW Mode (provider from docs)
Same as MIGRATE, except Phase 1 reads provider documentation instead of legacy code, and Phase 2 extracts the same facts from the docs. Status values usually live in a "transaction/resource states" table, webhook payload examples, or SDK enums. Add an explicit GATE to confirm any ambiguous format (e.g. amount in minor vs major units), and emit a `// TODO` comment for any field you could not determine from the docs.

## VALIDATE Mode
1. Read the target adapter for the provider.
2. Read the legacy adapter (if any) or the provider docs.
3. Compare status-map coverage: statuses present in the source but missing in the target = **GAP**; typos in status strings = **BUG**.
4. Report findings.

## LIST Mode
1. Glob the legacy adapters.
2. Glob the target adapters.
3. Show the diff — legacy providers not yet in the target, sorted simplest-first by source size.

## Critical Rules
1. **Number/amount format is whatever the legacy code does.** If the source passes a value straight through, the target does too; if it converts (e.g. ÷100), the target does too. Read the actual code — never assume.
2. **Never throw inside the execute path for expected failures.** Return the canonical failed result; let only truly unexpected exceptions bubble to the base class.
3. **Unknown status fails safe** — the status-map default is always the failure status.
4. **Use constants, never inline strings.** Every endpoint, header, status, and message lives in `<Provider>Constants`.
5. **Match naming exactly:** class names in the target's convention; slugs in the constants convention; serialized field names match the provider API character-for-character. When deriving a slug from a messy name, read the existing constants file first and follow the established pattern; GATE with the user if ambiguous.
6. **Do not add features the legacy version lacks.** A pure migration is the same behavior in a different language.
7. **Do not fake a stub.** If a legacy method only returns a FAILED stub, there is nothing real to port — report it and ask whether to emit a skeleton marked `// NOT IMPLEMENTED UPSTREAM`. Never invent an implementation.

## Acceptance Bar
"Compiles" is not the bar. The bar is **indistinguishable from code a senior engineer would have written by hand** in the target repo. Before opening a PR, score the generation: compiles clean, faithful to the legacy behavior (amount/status/endpoints identical), matches house style of the latest merge, status map complete with the GATE confirmed, and registrations that apply and run. Any axis failing means the rule that was violated should be folded back into this skill as an explicit instruction — every mistake that appears twice becomes a rule.
