# Anti-Patterns When Instructing Claude Code

* [Avoid Open-Ended "Chore Commands" to Model in Architecture Files]()
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
3. __If all Team Members have MCP Servers Configured:__ Since you cannot lock the MCP configuration into the project repository, you should manage your dev team synchronization through documentation and onboarding steps -- add a dedicated section in your project's main `README.md` file instructing developers on how to configure their local environment before running Claude Code

   __Project's main `README.md`
   
   ```markdown
   ## 🤖 Claude Code Setup
   This project leverages specific Model Context Protocol (MCP) servers to help Claude Code fetch live architectural guidance.

   Ensure your global `~/.claude/config.json` includes the `microsoft-learn` MCP server:
   ```json
   {
    "mcpServers": {
      "microsoft-learn": {
        "command": "node",
        "args": ["/path/to/microsoft-learn-server/index.js"]
      }
    }
   }
   ```
   ```

   __Add a section to `CLAUDE.md`__

   ```markdown
   ## Architectural Validation Rules
   - When analyzing architectural styles, use the attached `microsoft-learn` MCP server tool definitions to fetch the latest .NET Minimal API design patterns.
   - Compare live Microsoft Learn findings against the local guidelines in `.docs/Architecture/minimal-api-architecture.md`.
   ```
