# codex-delegate

Delega implementação de código ao Codex CLI. Use SEMPRE que precisar escrever ou editar arquivos `.py`, `.js`, `.html`, `.css` ou `.sql`.

## Regra de Arquivo

| Tipo | Executor |
|---|---|
| `.py` `.js` `.html` `.css` `.sql` | **Codex** via esta skill |
| `.md` `.json` `.toml` `.yaml` `.env` | Claude escreve diretamente |

Nunca use `Edit` ou `Write` em arquivos de código. Se você se pegar prestes a fazer isso, pare e use esta skill.

## Passo 1 — Descobrir contexto disponível

Verifique quais destes arquivos existem em `$(pwd)`:

```bash
ls CLAUDE.md AGENTS.md CHECKPOINT.md graphify-out/graph.json 2>/dev/null
```

Inclua no prompt apenas os que existem. Não mencione os ausentes.

## Passo 2 — Montar o prompt em /tmp/codex_prompt.md

Escreva um arquivo `/tmp/codex_prompt.md` com esta estrutura:

```
## Contexto do Sistema

Você está trabalhando em um repositório em <caminho absoluto do pwd>.
Leia os seguintes arquivos antes de implementar (apenas os que existem):

- ./CLAUDE.md — instruções do projeto, stack, padrões, gotchas, permissões por perfil
- ./CHECKPOINT.md — estado atual e decisões recentes (se existir)
- ./graphify-out/graph.json — grafo de dependências do codebase (se existir)

## Restrições Críticas

<extraia do CLAUDE.md as restrições mais importantes para a tarefa atual>

Exemplos comuns:
- Nunca expor valores monetários a perfis não-admin
- Usar context manager padrão do projeto para banco de dados
- Seguir padrão de serialização existente
- Nunca interpolar variáveis diretamente em SQL

## Tarefa

<descrição clara e testável do que implementar>

## Arquivos Relevantes

<lista de paths que Claude identificou como relevantes para a tarefa>

## Critérios de Conclusão

<o que deve funcionar para considerar a tarefa completa>
```

## Passo 3 — Executar o Codex

```bash
codex exec --full-auto \
  --output-last-message /tmp/codex_result.md \
  -C $(pwd) \
  "$(cat /tmp/codex_prompt.md)"
```

Aguarde a conclusão. O output do Codex será salvo em `/tmp/codex_result.md`.

Se o arquivo `/tmp/codex_result.md` estiver vazio ou o comando retornar erro, verifique:
- Se o projeto está marcado como trusted em `~/.codex/config.toml`
- Se `codex login` foi executado e está autenticado
- Não prossiga para o Passo 4 com output vazio

## Passo 4 — Ler o resultado

```bash
cat /tmp/codex_result.md
```

Leia também os arquivos modificados pelo Codex para entender as mudanças.

## Passo 5 — Validar (Devil's Advocate)

Avalie criticamente o que o Codex produziu:

1. **Completude**: A tarefa foi totalmente implementada ou ficou incompleta?
2. **Segurança**: Há SQL injection, XSS, exposição de dados sensíveis?
3. **Padrões**: Segue os padrões do projeto (context manager, serialização, etc)?
4. **Regressões**: Alguma funcionalidade existente pode ter sido quebrada?
5. **Testes**: O código pode ser verificado? O serviço sobe sem erro?

**Se aprovado:** prossiga com verificação e commit.

**Se rejeitado:** adicione as correções específicas ao `/tmp/codex_prompt.md` como uma seção `## Correções Necessárias` e re-execute o Passo 3.

Limite máximo: 3 iterações. Se após 3 tentativas o resultado ainda for rejeitado, pare e consulte o usuário — o problema pode ser na especificação da tarefa, não na implementação.

## Passo 6 — Verificar funcionamento

Rode o serviço ou testes relevantes para confirmar que a implementação funciona antes de commitar.

## Instalação em Novo Servidor

Pré-requisitos:
1. Codex CLI instalado (`npm install -g @openai/codex` ou equivalente)
2. Autenticado (`codex login`)
3. Projeto marcado como trusted em `~/.codex/config.toml`:
   ```toml
   [projects."/caminho/do/projeto"]
   trust_level = "trusted"
   ```
   Isso acontece automaticamente ao aceitar o prompt na primeira execução.
4. Esta skill copiada para `~/.claude/skills/codex-delegate/SKILL.md`
