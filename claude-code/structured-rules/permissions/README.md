# Adding Permission Rules in `.claude/settings.json`

## References

* Egress
  * [`GhHasiloOrg|gh-template-for-dotnet-minimal-api-with-ai-rules` __Dotnet Minimal API with AI Rules__](https://github.com/hasilo-org/gh-template-for-dotnet-minimal-api-with-ai-rules)

## Context

* If you state a rule that there should be no visible secrets in git repo, Claude Code internally implements that in `.claude/settings.json` so you might as well learn how to edit it directly if necessary
* A matching ```.gitignore``` needs to exist

## Examples

* From [`GhHasiloOrg|gh-template-for-dotnet-minimal-api-with-ai-rules` __Dotnet Minimal API with AI Rules__](https://github.com/hasilo-org/gh-template-for-dotnet-minimal-api-with-ai-rules)
  * `.claude/settings.json`
    ```json
    {
      "permissions": {
        "deny": [
          "Read(**/bin/**)",
          "Read(**/obj/**)",
          "Read(**/.vs/**)",
          "Read(**/.idea/**)",
          "Read(**/*.userprefs)",
          "Read(**/publish/**)",
          "Read(**/out/**)",
          "Read(**/*.trx)",
          "Read(**/*.log)",
          "Read(**/.aspire/**)",
          "Read(**/docker-compose.override.yml)",
          "Read(**/TestResults/**)",
          "Read(**/.env)",
          "Read(**/.env.local)",
          "Read(**/*.pfx)",
          "Read(**/*.pem)",
          "Read(**/appsettings.*.local.json)"
        ]
      }
    }
    ```
  * Matching entries in ```.gitignore``` (unmatched are marked)
    ```txt
    # Build output
    **/bin/
    **/obj/
    **/publish/
    **/out/
    
    # IDE / tooling
    .vs/
    .idea/
    *.user                  --> Not in .claude/settings.json
    *.userprefs
    
    # Test and log output
    **/TestResults/
    *.trx
    *.log
    
    # Aspire local state
    .aspire/
    
    # Local overrides and secrets (never commit)
    .env
    .env.local
    *.pfx
    *.pem
    appsettings.*.local.json
    docker-compose.override.yml
    
    # Claude Code local settings
    .claude/settings.local.json   --> Not in .claude/settings.json
    ```
 
