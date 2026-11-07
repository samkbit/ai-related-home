# Most Important No-BS AI Concepts to Grasp #

## Agent | Harness | Model (LLM)

* __Agent:__ is the __whole damn car__
  * Examples: ```Claude Code (owner: Anthropic) | Codex (owner: ChatGPT) | GH Copilot (owner: Microsoft) | etc.```
* __Harness:__ is the __transmission, chassis, steering, and brakes__ of the car
  * It is the __execution layer__ (tools, memory, guardrails, feedback loops, and command loops) that wraps around the __model (LLM)__
* __Model (LLM):__ is the __engine__ of the car
  * It is the __brain__ of the whole agent

## Agent Loops

* __```TODO```__: Does this refer to feedback loops, and command loops of the execution layer i.e. the harness?
* References
  * [`Yt|DavidOndrej` My Agentic Engineering Workflow (after 15,700 sessions)](https://youtu.be/F8rc-ULd4_4?is=WrWwE4XX0iTBJvYy)

## Tokens

* Regardless of which AI Coding Agent(s) you have __paid subscription(s)__ for, you want to be able to __maximize the monthly tokens__ while not exceeding the tokens -- it almost sounds like your mobile data plan
* AI Coding Agent allows you to switch among its own models only and the __highest tokens are used by the best model__ -- __```opus 5.*``` in case of __Claude Code__
* There does not exist an AI Coding Agent that allows you to switch models from different model providers and utilize the tokens from your paid subscriptions to those models (via different Coding Agents) -- instead, this AI Coding Agent will have its own subscription plans and token limits
  * The closest example is __GitHub Copilot__ as AI Coding Agent which does not natively own its own models. Instead, it allows you to switch models from Claude Code, ChatGPT and Gemini. If you already have paid subscription to, say Claude Code, you cannot utilize its tokens in GH Copilot when you switch model to one of the models from Claude Code

## Cloud Agents

* Instead of running on your computer, agents run in the cloud
* When you buy subscriptions for any AI Coding Agent, you usually want to install and run it on your computer right away -- however, they also provide free cloud agents to certain extent
* IMO, the more you use cloud agents, the more you are locked-in to their ecosystem -- there is no AI agent federation to help mix and match underlying models and the harnesses
* The only exception seems to be `Cloud Room` (where David Ondrej is a founder) but I have not researched fully

