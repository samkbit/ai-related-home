# Claude Code will Create Skills as Necessary

## Differentiating from Plugins/Skills Install

* This sits outside of the scope of __Claude Code will Create Skills as Necessary__
* You can always install plugins (such as those from __Official Claude Plugins__) which will then install necessary components including __skills__
* You can also install skills directly
  ```bash
  npx agent-skills-cli add <github-owner>/<repo-name>
  ```
  * Depending on what project you are working on, some 3rd party tools and libraries may instruct you to install specific skills this way

## What I Mean by *Claude Code will Create Skills as Necessary*

* As you converse with Claude Code -- more specifically, as you create structured rules for Claude Code in a given project -- Claude Code will create __skills__ as it sees fit. You can hint at it if you want
* Examples
  * In project [`GhHasiloOrg|gh-template-for-dotnet-minimal-api-with-ai-rules` __Dotnet Minimal API with AI Rules__](https://github.com/hasilo-org/gh-template-for-dotnet-minimal-api-with-ai-rules), in the midst of convo while updating structured rules, I said the following
    ```
     In your above response titled "I can do on request", yes do them. As for "6. Scaffolding workflow:",
     perhaps get user inputs up-front quickly but in a systematic manner -- I want to prevent a scenario
     where you proceed with scaffolding workflow and ask for user inputs late in the process when user might
     have already left the computer for a coffee break, thinking it's gonna take a while in uninterrupted
     manner -- maybe you will need to create a skill for this, with various categories for user inputs, up to you.
    ```
    * I simply hinted at it (based on my previous experience with it) and result was exactly that -- Claude Code must have thought it was the way to go. There was a new `SKILL.md` created in `.claude/skills/scaffold-api`
