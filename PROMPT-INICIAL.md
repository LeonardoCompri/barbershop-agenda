# Passo a passo e prompts para o Claude Code

## Antes da Fase 0 (uma vez)

**1. No seu repositório de agentes**, copie a pasta `maestria/` deste pacote para a raiz dele e faça o commit.

**2. Crie o projeto e ligue os agentes:**
```bash
mkdir -p ~/projetos && cd ~/projetos
unzip /mnt/c/Users/SEU_USUARIO/Downloads/maestria-barbearia-pacote.zip -d /tmp/maestria-pacote
cp -r /tmp/maestria-pacote/maestria-barbearia .
# (passo 1) cp -r /tmp/maestria-pacote/repo-agentes/maestria ~/projetos/SEU-REPO-DE-AGENTES/
cd maestria-barbearia

# link para a pasta maestria/ do seu repositório de agentes (ajuste o caminho)
mkdir -p .claude
ln -s ~/projetos/SEU-REPO-DE-AGENTES/maestria .claude/agents
ls .claude/agents               # deve listar os 6 arquivos .md

nvm install && nvm use          # Node 24 do .nvmrc
docker compose up -d            # PostgreSQL 18 + Mailpit
docker compose ps               # os dois "running"/"healthy"
git init && git add . && git commit -m "Especificação, regras dos agentes e infraestrutura"
claude
```
No Claude Code, digite `/agents` e confira os 6 agentes. O link `.claude/agents` fica fora do git (está no `.gitignore`), porque aponta para um caminho da sua máquina.

Pendências que não travam o início:
- confirmar a cidade na seção 2 de `docs/ESPECIFICACAO.md` (antes da Fase 1, por causa do seed);
- criar o OAuth Client do Google antes da Fase 2 (o passo a passo será gerado em `docs/google-login.md`);
- WhatsApp Business só na Fase 5.

---

## Prompt 1 — Kickoff (cole uma vez)

```
Vamos construir o sistema da Maestria Barbearia: site de divulgação da marca, agendamento online com login Google e backoffice com acesso por código. Tudo responsivo.

Leia CLAUDE.md e docs/ESPECIFICACAO.md por completo antes de qualquer coisa.

Contexto importante: sou desenvolvedor sênior em PHP/Laravel e Vue, mas este é meu primeiro contato com NestJS e Next.js. Quero que o código seja o mais simples possível, com comentários didáticos em português e comparações com Laravel/Vue, para eu aprender durante o projeto.

Trabalhe com os agentes de .claude/agents/ (cada um deve ler o seu arquivo em docs/agentes/ antes de trabalhar):
- tech-lead: planeja cada fase e define os contratos da API
- backend-nest: implementa a pasta api/
- frontend-next: implementa a pasta web/
- ui-designer: identidade visual futurista, tema escuro (padrão) e claro
- qa-tester: testes e validação das regras de agendamento
- mentor: revisa a simplicidade e escreve as notas de aprendizado

Agora:
1. Use o tech-lead para criar docs/PROGRESSO.md com o checklist de todas as fases e o plano detalhado da Fase 0.
2. Me mostre o plano e espere minha aprovação antes de implementar.
```

---

## Prompt 2 — Executar uma fase (repita para cada fase)

```
Plano aprovado. Execute a Fase N seguindo o plano do tech-lead:
- delegue cada tarefa ao agente indicado;
- depois de cada implementação, rode o qa-tester;
- ao final, peça ao mentor a revisão de simplicidade e a nota docs/aprendizado/fase-N.md;
- atualize docs/PROGRESSO.md.
No fim, me diga o que testar manualmente e quais conceitos novos apareceram.
Não adiante nada da fase seguinte.
```

---

## Prompt 3 — Planejar a próxima fase

```
Fase N concluída e testada por mim. Use o tech-lead para planejar a Fase N+1 e me mostre o plano antes de implementar.
```

---

## Prompts úteis no dia a dia

```
Use o mentor para me explicar como funciona <arquivo ou conceito>, comparando com Laravel/Vue.
```
```
Use o qa-tester para tentar quebrar o fluxo de agendamento e me reporte os problemas.
```
```
Use o ui-designer para revisar a tela <rota> no celular (375px) e nos dois temas.
```
```
Encontrei este bug: <descrição>. Use o agente certo para corrigir e o qa-tester para criar um teste que impeça a volta dele.
```
