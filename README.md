# AI-things

## Claude Code skill sync

`~/.claude/skills` is a symlink into `claude-skills/` in this repo, so custom
Claude Code skills stay in sync across machines via git instead of manual
copying.

### Setup on a new laptop

```bash
gh auth login   # if not already logged in
git clone https://github.com/jaromtia/AI-things.git ~/dotfiles
rm -rf ~/.claude/skills   # remove if it already exists
ln -s ~/dotfiles/claude-skills ~/.claude/skills
```

### Adding a new skill

Create it under `claude-skills/`, then:

```bash
cd ~/dotfiles && git add -A && git commit -m "add <skill> skill" && git push
```

### Syncing on an existing laptop

```bash
cd ~/dotfiles && git pull
```
