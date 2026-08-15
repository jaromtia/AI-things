# dotfiles

Personal Claude Code configuration synced across machines.

## Setup on a new laptop

```bash
git clone <this-repo-url> ~/dotfiles
rm -rf ~/.claude/skills          # remove if it already exists
ln -s ~/dotfiles/claude-skills ~/.claude/skills
```

## Adding a new skill
Create it under `claude-skills/`, then:
```bash
cd ~/dotfiles && git add -A && git commit -m "add <skill> skill" && git push
```

## Syncing on an existing laptop
```bash
cd ~/dotfiles && git pull
```
