# skills

a directory of jason's personal, public skills

## list all skills

~~~bash
npx skills add jasonharmongit/skills --list --full-depth
~~~

## install a single skill

~~~bash
npx skills add jasonharmongit/skills --skill <skill-name> --full-depth
~~~

## install all skills

~~~bash

# all skills for a specific agent, global install
npx skills add jasonharmongit/skills --skill '*' -a <agent> -g -y --full-depth
~~~

`--full-depth` is required so skills under `solutionize/` are discovered (they live outside the default scan paths).

this will install the proper `.<agent>` directory under your home directory. I tried with a few different ones and it seemed to always just install under a `.agents` directory.

valid agents list:

~~~
amp, antigravity, augment, bob, claude-code, openclaw, cline, codebuddy, codex, command-code, continue, cortex, crush, cursor, deepagents, droid, firebender, gemini-cli, github-copilot, goose, junie, iflow-cli, kilo, kimi-cli, kiro-cli, kode, mcpjam, mistral-vibe, mux, opencode, openhands, pi, qoder, qwen-code, replit, roo, trae, trae-cn, warp, windsurf, zencoder, neovate, pochi, adal, universal
~~~
