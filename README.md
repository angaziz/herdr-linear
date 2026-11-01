# herdr-linear

A [Herdr](https://herdr.dev) mod: your Linear issues in a side panel next to your
terminals. Browse by status and project, open an issue's details and sub-issues,
change status, priority, assignee and agent, and start an agent on an issue with
your own prompt.

It talks to Linear through the [`linear` CLI](https://github.com/schpet/linear-cli)
and its login, so Herdr stores no API key. macOS only for now.

## Install

Ask your agent:

```
Read ~/.config/herdr-app/AGENTS.md, then add this mod: https://github.com/angaziz/herdr-linear
```

Or by hand:

1. Install and log in to the CLI. Editing needs a personal API key with write access.

   ```sh
   brew install schpet/tap/linear
   linear auth login
   ```

2. Copy the mod into Herdr:

   ```sh
   cp -R mods/linear ~/.config/herdr-app/mods/
   herdr-app check ~/.config/herdr-app/mods/linear
   ```

3. Give it a place beside Files and Changes. Herdr's shipped layout and tab bar are
   read-only, so clone them:

   ```sh
   herdr-app do layout.clone '{"name":"main","as":"my-main"}'
   herdr-app do mod.clone '{"name":"tabs","as":"my-tabs"}'
   ```

   - In `~/.config/herdr-app/layouts/my-main.json`, add
     `{ "id": "linear", "type": "block", "content": { "mod": "linear" } }` to the
     `panel-body` stack's children, and change the `tabs` block's mod to `my-tabs`.
   - In `~/.config/herdr-app/mods/my-tabs/main.js`, add `"linear"` to `enabled`, and
     a button next to Changes:
     `.child(panelButton("linear", "Linear", "mods/linear/linear.svg"))`.

The first time the panel opens, a welcome screen checks the CLI, your login and
whether your key can write.

[RECIPE.md](RECIPE.md) describes the mod in full, for rebuilding it from scratch.
