# claude-skills

Skills globais para Claude Code.

## Skills disponíveis

| Skill | Descrição |
|---|---|
| [`codex-delegate`](codex-delegate/SKILL.md) | Delega implementação de código ao Codex CLI, reservando Claude para planejamento e validação |

---

## codex-delegate

Claude atua como orquestrador puro (planejamento, análise, Devil's Advocate). Todo código `.py`, `.js`, `.html`, `.css`, `.sql` é implementado pelo Codex CLI.

### Pré-requisitos

1. [Claude Code](https://claude.ai/code) instalado
2. [Codex CLI](https://github.com/openai/codex) instalado e autenticado:
   ```bash
   npm install -g @openai/codex
   codex login
   ```

### Instalação (1 comando)

```bash
mkdir -p ~/.claude/skills/codex-delegate && \
curl -fsSL https://raw.githubusercontent.com/LucassAlbu/claude-skills/main/codex-delegate/SKILL.md \
  -o ~/.claude/skills/codex-delegate/SKILL.md
```

### Marcar projeto como trusted no Codex

Na primeira vez que usar em um projeto, rode uma sessão interativa do Codex e aceite o prompt de trust. Isso escreve automaticamente em `~/.codex/config.toml`:

```toml
[projects."/caminho/do/projeto"]
trust_level = "trusted"
```

### Uso

No Claude Code, invoque via:
```
/codex-delegate
```

A skill aparece automaticamente na lista de skills disponíveis após a instalação.

### Como funciona

```
Claude planeja → monta prompt contextualizado → Codex implementa → Claude valida (Devil's Advocate) → aprova ou re-delega
```

Contexto auto-descoberto em runtime: `CLAUDE.md`, `CHECKPOINT.md`, `graphify-out/graph.json` — a skill detecta o que existe no projeto atual sem configuração manual.
