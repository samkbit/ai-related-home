# Anti-Patterns When Instructing Claude Code

* [Avoid Open-Ended "Chore Commands" to Model in Architecture Files](#avoid-open-ended-chore-commands-to-model-in-architecture-files)
  * [The Fix](#the-fix)

## Avoid Open-Ended "Chore Commands" to Model in Architecture Files

```
In the .docs/ folder, let's say I put /Architecture/minimal-api-architecture.md and in it, I write
"Connect with microsoft-learn MCP and get the latest architectural styles that are suitable for this
 project -- from my research so far, the styles are clean architecture, modular monolith and vertical
slice with CQRS although I understand they may overlap. Present user with options to choose from."
Is this type of instruction recommended?
```

__No, this type of instruction is not recommended__ for your architecture document.

An architecture file should contain __deterministic constraints and project facts__, rather than conversational, open-ended "chore commands" to the model.

The above instruction introduces 3 anti-patterns.

1. __Hardcoding "Chore" Instructions in Reference Docs:__ Reference files inside `.docs/` should act as source-of-truth documentation.
2. __Assuming a Specific MCP Server is Autoloaded:__ Claude Code cannot arbitrarily invent an MCP server or query an API unless the host system has that specific MCP server installed and configured
3. __Mixing Up LLM "Knowledge" with Real-Time Searching:__ Advanced models already have a deep, foundational understanding of standard .NET patterns like Clean Architecture, Modular Monoliths, and Vertical Slice Architecture with CQRS. Forcing it to go out to an external web tool just to pull down generic definitions of these styles __wastes tokens and time__.

### The Fix 

1. __Keep `.docs/Architecture/minimal-api-architecture.md` Factual__
   * Rewrite the file so it specifies what you __want__, not how it should __behave__.
      ```markdown
      # Minimal API Architecture Guidelines

      ## Approved Architectural Styles
      The project evaluates three potential architectural styles for our .NET Minimal APIs:
      - **Clean Architecture** (Separation of concerns via Core, Application, Infrastructure, WebAPI)
      - **Modular Monolith** (Bounded contexts isolated into independent modules)
      - **Vertical Slice Architecture** with CQRS (Feature-centric grouping using MediatR)
      
      ## Technical Constraints
      - Before scaffolding any endpoints, the user must explicitly choose one of these styles.
      - If live architectural guidance or updated .NET syntax is required, utilize a search tool to check current Microsoft Learn recommendations.
      ```
2. __Put the Interactive Instruction in `CLAUDE.md`__
   * Put the prompt-level instruction in your `CLAUDE.md` so Claude knows how to interact with you:
      ```markdown
      ## Scaffolding Workflow
      - When asked to initialize or scaffold a new microservice or API boundary, first read `.docs/Architecture/minimal-api-architecture.md`.
      - Present the architectural style options to the user and require an explicit selection before writing any code.
      ```
3. __Optimize `CLAUDE.md` for Zero-Tool Defaults__
   * Even if all your dev team members have MCP Servers configured (or the project's main `README.md` has a prereq section on how to configure MCP), __keep `CLAUDE.md` deterministic and treat live search as a manual, on-demand command__ -- otherwise Claude Code might __over-eagerly trigger the `microsoft-learn` MCP tool that can easily balloon your token consumption__.
   * Add a section in `CLAUDE.md` instructing Claude to use its own internal knowledge or local files, and to only use the MCP as a backup resource if it gets stuck.
      ```markdown
      ## Architectural Validation Rules
      - Rely on `.docs/Architecture/minimal-api-architecture.md` as the primary source of truth for architectural styles.
      - Only query the `microsoft-learn` MCP server if explicitly commanded by the user, or if local documentation is missing critical syntax for .NET Minimal APIs.
      ```
   * A better alternative is to not add the above section (if tokens are to used wisely) and instead, manually __invoke prompts for heavy lifting__ whenever you think is necessary. For example, you may ask in the slash prompt
      ```
      > Claude, use the microsoft-learn MCP to check if there are any 2026 updates to Vertical
        Slice architecture for Minimal APIs, and compare it to our doc in
        .docs/Architecture/minimal-api-architecture.md.
      ```
