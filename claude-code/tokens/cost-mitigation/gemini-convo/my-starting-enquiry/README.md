# Cost Mitigation Tips for Claude Code Subscriptions

> Created entirely based on my convo with Gemini and asking it at the end to generate a markdown
>
> * __Notice this page resides in a path that clearly specifies `claude-code` as an ascendant node, yet this page is generated based on convo with Gemini__
> * Working with Gemini IMO gives you an excellent overview independent of your actual git repo projects -- once you create actual projects, use your licensed AI assistant (with access to the latest LLM versions hopefully), in this case Claude Code based on this page's file path, and __start a new convo using this Gemini convo as a guidance__
>   * For example, this Gemini convo provided a great starting point in at least the following real projects
>     * GitHub Template project: [__Dotnet Minimal API with AI Rules__](https://github.com/hasilo-org/gh-template-for-dotnet-minimal-api-with-ai-rules)
> * Interesting Observations
>   * Per below section [Block .NET Noise via `.claudeignore`](#block-net-noise-via-claudeignore), Gemini refers to `.claudeignore` as something Claude Code supports natively but once you start conversing with Claude Code, you realize this is not true -- in fact, Claude Code understands your intent and correctly tells you that you should use `.claude/settings.json` containing a section `permissions:deny`

## Model Tip

To stop premium models from chewing through your tokens during basic tasks, developers often use the `/model opusplan` alias. This tells the harness to use the heavy intelligence of __Opus__ to outline the initial engineering blueprint, but automatically swaps down to __Sonnet__ to write the actual code and run the tests—giving you deep reasoning at a fraction of the token cost.

## Pro Subscription

Having a __Pro subscription__ changes your approach to token management, as you are operating within a __monthly usage pool__ rather than watching a direct dollar meter tick up on every keystroke.

Because Anthropic dynamically calculates your usage based on model weight, running high-tier models will exhaust your Pro subscription allowance much faster.

### How the Pro Subscription Handles Your Tokens

* __The Multiplier Effect:__ Under a Pro plan, a single token processed by a high-end model like Claude __Fable 5.1__ or Claude __Opus__ consumes significantly more of your monthly subscription credit than a token processed by Claude __Sonnet__.
* __The Caching Advantage:__ The Pro subscription fully respects Claude Code's built-in __Prompt Caching__. When you are iterating on a single file, the harness caches your repository structure. Subsequent commands only cost a fraction of your pool, provided you keep your agent sessions focused and don't bounce between entirely different codebases in a single conversation.

### Pro Tips for Maximizing Your Monthly Pool

* __Audit Your Usage Locally:__ Run the `/stats` command inside Claude Code at the end of a session. It will break down exactly how many tokens were sent, how many were successfully cached, and which models were invoked, helping you see what is draining your subscription.
* __Use Compact Sessions:__ When a bug is fixed, close the session (`/exit` or `/clear`). Keeping a massive session open while you move on to a brand-new task forces the harness to resubmit old conversation history, rapidly wasting your Pro credit on unneeded context.

## Specific Tips Related to Dotnet

### Block .NET Noise via `.claudeignore`

By default, the harness will try to index or search through everything in your folder. If it reads compiled binaries, it wastes a massive number of tokens. Create a `.claudeignore` file in the root of your solution directory and add the following lines:

```text
# Ignore compiled outputs and tooling artifacts
**/bin/
**/obj/
**/.vs/
**/.idea/
**/.userprefs

# Ignore local user packages and publish folders
**/publish/
**/out/

# Ignore trace logs and test results
**/*.trx
**/*.log
```

__Why this saves tokens:__ This prevents Claude Code from accidentally reading huge, auto-generated files in your `obj/` or `bin/` directories during a repository-wide search, keeping your cached context completely clean.

### Streamline `dotnet test` Output

When Claude Code runs an execution loop to verify your code using `dotnet test`, a massive text dump of passing tests wastes token space. You want to force the harness to only look at failures.

When giving tasks to the agent, instruct it to use the minimal logger flag:

* Instead of letting it run a raw `dotnet test`, tell it: __Run tests using `dotnet test --logger:\"console;verbosity=minimal\`"__
* This keeps the harness from ingestion of thousands of lines of "Passed" strings, focusing tokens strictly on stack traces and compiler errors.

### Leverage Solution Caching

Because your repo is mid-sized, it easily fits into the model's prompt cache. The golden rule to save your Pro pool here is __continuity__. Try to bundle your tasks into one logical session. If you ask it to fix a dependency injection issue, and then immediately ask it to write a unit test for that issue in the same session, the second request will read from the cache for almost free.

### Configure your Claude Code Harness Efficiently

This depends on what you use per project -- common things you might use in projects might be __.NET Aspire | Docker Compose | xUnit__

* Integrating __.NET Aspire__ and __Docker Compose__ into a mid-sized repository introduces a distinct token risk for your Claude Code Pro plan: __log bloat__. When the agent orchestrates multiple containerized resources or handles Aspire AppHost state changes, the background logs can easily chew through thousands of tokens in seconds.
* By configuring your Claude Code harness specifically for an Aspire, Compose, and xUnit architecture, you can keep your token consumption efficient while keeping the agent effective.

#### 1. Optimize Your `.claudeignore` for Aspire & Containers

Add these specific orchestrator and container patterns to your root `.claudeignore` file immediately to stop the agent from wasting context on localized runtime states:

```text
# Ignore Aspire dashboard and orchestrator runtime states
**/AppHost/bin/
**/AppHost/obj/
.aspire/

# Ignore local container and docker configurations
.dockerignore
**/docker-compose.override.yml

# Ignore xUnit artifacts
**/TestResults/
```

#### 2. Configure Claude Code for xUnit & Minimal Verbosity

By default, `dotnet test` with xUnit outputs a lot of conversational filler. You can save massive amounts of token space by forcing the agent to use compact logging formats.
Run this command inside your Claude Code terminal to set a persistent system prompt constraint:

```bash
/config append systemPrompt "When running tests for this xUnit project, always append '--logger:console;verbosity=minimal' or '-q' to the dotnet test command to prevent token waste on passing test logs."
```

#### 3. Handle Aspire & Docker Orchestration Efficiently

Because you frequently switch between Aspire and Docker Compose, the agent will naturally try to boot your entire local microservices architecture to test small code changes. __This will drain your Pro pool instantly__.

Follow these strict operational rules when prompting the agent:

* __Isolate the Target:__ Do not tell the agent to *"Fix the user service and make sure the app runs."* Instead, explicitly restrict its tool scope: *"Fix the user service class library. Do not boot the Aspire AppHost or Docker Compose; only run the specific xUnit project `UserService.Tests` to verify."*
* __Pipe Logs to Files:__ If the agent must debug a live orchestration issue (e.g., a broken OTLP telemetry connection or a failing container dependency), never let it run `docker compose up` or `dotnet run --project AppHost` directly in the open terminal. Instruct it to dump output to a temporary log file:
  * *Run the Aspire AppHost, stream the output to a `logs.txt` file, and stop it after 10 seconds. Then search the `logs.txt` file for the specific exception."*
  * This prevents the agent's harness from absorbing thousands of lines of live, repeating telemetry traffic into its active prompt cache.

To optimize this setup even further, let Gemini know:

* Do you use __.NET Aspire Service Defaults__ to automatically wire up your OpenTelemetry (OTLP) packages?
* Do you rely on any __cloud-based tooling__ (like Azure Monitor or Grafana Cloud) to mirror this setup, or is it completely local?

Gemini can tailor your testing and observability workflows based on your answer.

#### 4. Leverage xUnit Traits for Agentic Runs

If your test suite contains integration tests that require live containers, use xUnit `[Fact]` attributes or `[Trait]` categories to separate fast unit tests from heavy container tests. You can then instruct Claude: *"Only execute unit tests via `dotnet test --filter Category=Unit`"* to ensure it stays in a fast, low-token loop.

