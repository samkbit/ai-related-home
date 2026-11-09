# Adding Open Items from Claude Run to ```TODO.md```

## What are Open Items

* As you engage with Claude Code in a given project, you should follow my __structured rules__ -- there will likely be many topics you want to cover before you start convo or in the midst of convo or even after the project has covered all current topics. In such cases, simply itemize them in __```TODO.md```__ which should exist in the project root as a __sibling to ```README.md``` and ```CLAUDE.md```__
* Especially __important to itemize in ```TODO.md```__ are the open items mentioned in __Claude Code report after each run__
  * These open items are placed under various titles but they are self-explanatory
  * Some examples of what I have seen under different titles are listed below in sub-sections
  * > __Claude Code realizes what you are doing with `TODO.md` and in subsequent runs, it may directly start adding to with in addition to showing the items in the run report__

### `Things to check or decide:`

- Keycloak placeholders: The realm template relies on ${ENV_VAR} substitution at import time. I believe Keycloak supports this, but verify it for the version you pin.
- Cognito client check: The Cognito doc leaves the client_id check as an outline, because the aud claim is missing from access tokens. You still need to decide how AllowedClientIds is configured.
- Claims transformation: The IClaimsTransformation in authorization.md is a stub with the steps listed in comments, not working code.
- Test layout: The integration-testing section proposes a separate KeycloakTests project (or a Category trait). That's a new convention you may want to adjust.
- Rule 8 wording: It says test passwords must be "clearly labelled dev-only". jwt-bearer-keycloak.md makes that concrete as a description field in the realm JSON.
- Routing line: CLAUDE.md points to .docs/Authentication/secrets-management.md, which is among the drafts. The other routing lines send Claude to the folders.

### `Needs you (I can't touch .docs/)`

> You probably want to take care of this immediately instead of itemizing in `TODO.md`

1. Copy the ten drafts into .docs/. Until you do, the CLAUDE.md routing lines point to files that don't exist.
2. Fix the MediatR mention at minimal-api-architecture.md:9. It still contradicts rule 3.
3. Review the drafted code snippets. The Keycloak ${ENV_VAR} placeholder substitution, the Aspire API and the collector config are the parts most likely to be wrong for your versions.

### `Older items still open from my first review`

- .wiki/ name: Decide whether to rename it (GitHub reserves <repo>.wiki).
- Test libraries: FluentAssertions v8+ is commercially licensed and Moq had the SponsorLink controversy. This is the same kind of licensing concern as your MediatR ban.
- README: It still has no usage instructions, and it contains a "Temporary and to be DELETED" section.
- Docs fixes: Small fixes in options-pattern.md and test-strategy.md (the missing using, csharp fence tags and the CancellationToken example).

### Devising new open item based on Claude run report

> Run report included a line item listed below -- it was obvious this should be a TODO item
 
__Unattended runs:__ Permission prompts will still pause the run unless you pre-approve the dotnet commands and file writes. The skill says so at confirmation. I didn't add allow rules to the shared settings.json, since that's your call.

In ```TODO.md```, I added the following section

```markdown
## Unattended runs

- Permission prompts will still pause the run unless you pre-approve the dotnet commands and file writes. The skill says so at confirmation. I didn't add allow rules to the shared settings.json, since that's your call.
```

