# AtlasHub Sales OS — FOUNDATION 01
**Estado:** diretiva aprovada para descoberta e piloto técnico; **deploy não autorizado nesta etapa**  
**Base:** [melgarafael/DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) · licença MIT  
**Fork:** [atlashub-digital/ATLASHUB-SALES-OS](https://github.com/atlashub-digital/ATLASHUB-SALES-OS)  
**Data da diretiva:** 2026-10-28

## 1. Objetivo

Avaliar e adotar o DeskcommCRM como **núcleo único de CRM, vendas, atendimento omnicanal e agentes comerciais de IA** para operações AtlasHub. A plataforma poderá servir como base de um futuro **AtlasHub Sales OS**, SaaS de subscrição e serviços geridos de AI Workforce.

A prioridade é colocar um **piloto interno real, controlado e reversível** em funcionamento antes de desenvolver módulos próprios ou comercializar. O projeto original já contém funções de multiempresa, branding, agentes/MCP e cobrança de revendedor: validar no runtime antes de recriar.

**Princípio:** reutilizar o que existe; configurar antes de personalizar; uma única fonte de verdade para dados comerciais; manter sincronização com o upstream.

## 2. Limites de autorização

Nesta fase, é autorizado **investigar, documentar, testar sem dados reais e preparar PRs revisáveis**. **Não** é autorizado, sem decisão posterior:
- Alterar/deixar offline sistemas atuais, mudar DNS, ligar números WhatsApp reais, ativar IA em conversas reais ou executar migrações destrutivas.
- Criar recursos externos pagos, enviar mensagens a clientes, publicar produto pago, ativar inscrições externas ou recolher dados pessoais de produção.
- Transferir segredos, bases de dados, backups ou dados de clientes para o repositório. O fork é **público**.
- Fazer alterações grandes de identidade, arquitetura ou código do upstream sem especificação/justificação e testes.
- Usar instaladores remotos não auditados, `latest` flutuante em produção ou fazer deploy inadvertido da imagem upstream.

## 3. Premissas técnicas verificadas no fork

- App Next.js 16/React 19/TypeScript; Supabase Postgres/Auth/Realtime/Storage, RLS por `organization_id`.
- Inbox, contactos, funil, agentes nativos, RAG, MCP, API, automações e workers; canal WhatsApp via WAHA/NOWEB, com opções de integração oficial Meta a avaliar.
- White-label por instalação (`/admin/marca`) e por organização. **Domínio próprio por organização não é nativo**, portanto oferecer instância dedicada quando necessário.
- Cobrança do **administrador da instalação** às organizações: planos, cotas, trial, checkout Stripe/Asaas e gestão do ciclo de assinatura no código em `lib/cobranca/`, `app/api/v1/admin/cobranca/`; funcionalidade exige validação em **sandbox**.
- O pacote upstream inclui scripts de instalação/atualização e imagens pré-compiladas. Um fork por si só **não garante** que a imagem publicada corresponda ao código AtlasHub.
- A documentação tem pontos históricos divergentes; o comportamento efetivo do código e a reprodução dos testes prevalecem para a avaliação.

Fontes internas: [AGENTS](../../AGENTS.md) · [Arquitetura](../../ARCHITECTURE.md) · [White-label](../../docs/white-label.md) · [Cobrança de revendedor](../../docs/adr/0004-cobranca-do-revendedor.md).

## 4. Arquitetura-alvo mínima

```text
                          AtlasHub Sales OS
                  (instalação/painel AtlasHub)
                                |
             +------------------+------------------+
             |                  |                  |
      Tenant AtlasHub     Tenant piloto B     Clientes futuros
             |                  |
             +------------------+------------------+
                                |
                  DeskcommCRM / API / MCP
                  |       |        |       |
                 CRM    Inbox     Agentes  Billing
                  |       |        |       |
                  +-------+--------+-------+
                                |
                    Supabase dedicado + workers
                                |
                   Integrações externas controladas
                         n8n / Hermes / APIs
```

- **DeskcommCRM = sistema de registo comercial.** Evitar dividir contactos, pipeline, conversas e transições de negócio por sistemas paralelos.
- **Agentes nativos = primeiro motor de execução.** Hermes só em fluxos externos especializados quando houver contrato técnico claro e sem execução duplicada. Integração possível por MCP/API, com tokens de escopo mínimo, auditoria e handoff.
- **n8n = orquestração entre sistemas**, não CRM alternativo.
- **Chatwoot/Evolution/Typebot existentes = legado em avaliação**, sem desativação neste piloto.
- **Supabase dedicado** ao staging deste fork, sem reutilização de bases dos outros produtos.
- **SaaS partilhado** apenas após testes de isolamento multiempresa; **instância dedicada** como opção enterprise.

## 5. Ordem de trabalho — fases e gates

### F0 — Inventário técnico e estratégia de fork
- [ ] Fixar SHA base e release upstream de referência. Não assumir que o `main` mais recente é release estável.
- [ ] Ler `CLAUDE.md`, `AGENTS.md`, `docs/index.md`, `ARCHITECTURE.md`, `SECURITY.md`, `docs/white-label.md` e ADR-0004.
- [ ] Inventariar módulos: auth/RBAC/RLS, contactos/CRM/inbox, canais, agentes/MCP, workers, branding e cobrança. Classificar **pronto / configuração / adaptação / lacuna**.
- [ ] Auditar `hostgator-setup-kit`, `Dockerfile*`, compose, scripts `update.sh` e fontes de `APP_IMAGE`; identificar referências ao upstream e risco de deploy da imagem errada.
- [ ] Propor imagem AtlasHub versionada e pipeline CI/deploy/rollback reproduzível. Não exigir compilações na VPS operacional.
- [ ] Definir regra de atualização upstream: remote `upstream`, branch de integração, comparação de release, PRs pequenos, testes, documentação de conflitos.
- [ ] Produzir ADR de topologia, recursos/custos e matriz de convivência/substituição das ferramentas atuais.

### F1 — Staging isolado
- [ ] Selecionar ambiente, orçamento e isolamento de rede; conferir CPU/RAM/storage e dependências sem tocar na infraestrutura em produção.
- [ ] Criar banco Supabase exclusivamente do piloto, envs seguros e backups/restauração comprovados.
- [ ] Deploy de commit/imagem AtlasHub fixos; HTTPS, health checks, logs, crons/worker e rollback.
- [ ] Criar tenant piloto AtlasHub com utilizadores e dados de demonstração.
- [ ] Testar autenticação, perfis, CRM, funil, registo de contactos e relatórios.

### F2 — White-label mínimo
- [ ] Nome, cores, logo e ícone pelos pontos de configuração nativos.
- [ ] Validar login, emails, favicon/manifest/PWA, marcas por organização e mensagens transacionais.
- [ ] Documentar referências públicas de branding que precisem de PR pontual; preservar atribuições e MIT.
- [ ] Evitar renomear pacotes, cookies, headers, tabelas e caminhos de API apenas por estética.

### F3 — Canal e agente controlados
- [ ] Um canal/número **de teste** e uma conversa sintética completa de entrada → inbox → funil → handoff humano → auditoria.
- [ ] Um agente comercial de teste com regras, guardrails e teto de gasto, sem contactos externos.
- [ ] Uma chamada MCP com escopo mínimo ligada a Hermes; provar isolamento do tenant e idempotência, sem motor redundante.
- [ ] Integração n8n via API/webhook somente em sandbox.
- [ ] Avaliar WAHA/NOWEB versus Meta WhatsApp Cloud API para produção comercial, com riscos, custos e condições de uso.

### F4 — SaaS e pagamentos (apenas sandbox)
- [ ] Validar módulos de planos/cotas/assinaturas e fluxos administrativos da instalação.
- [ ] Exercitar Stripe e/ou Asaas em sandbox: trial, checkout, webhook validado, retry, cancelamento, falha, suspensão e reativação.
- [ ] Provar isolamento inter-tenant de REST, MCP, realtime, storage, workers e caminhos service-role.
- [ ] Analisar LGPD, retenção, exportação/apagamento, políticas de canais, termos e segurança operacional.
- [ ] Modelar custos por cliente (infra, mensagens, tokens, suporte) e planos START/GROWTH/WORKFORCE **sem preços definitivos**.

## 6. Critérios de aceitação

### GO interno
1. Deploy AtlasHub **com imagem correspondente ao fork** e commit fixado; procedimento documentado de rollback.
2. Tenant AtlasHub operacional, com login, RBAC, CRM e funil.
3. E2E do canal de teste até ao registo CRM, incluindo handoff e auditoria.
4. Agente nativo funcional e integração MCP externa demonstrada com limites e sem vazamentos.
5. Testes de isolamento entre pelo menos dois tenants, incluindo tentativas negativas.
6. Restore de backup comprovado, healthchecks e workers/crons comprovados.
7. White-label mínimo configurado e relatório claro de lacunas, risco e custo.

### GO SaaS externo (gate posterior, não implícito)
- Pagamentos sandbox e gestão da assinatura testados; segurança/isolamento revisados.
- Canal comercial e termos de utilização validados.
- Suporte, SLA, política de dados, cancelamento e recuperação documentados.
- Custos unitários e pricing aprovados.
- Aprovação explícita antes de receber clientes reais.

## 7. Primeira entrega de engenharia solicitada

**PR F0 / Auditoria e arquitetura**, com:
- `docs/atlashub/F0-INVENTARIO.md` — funcionalidades medidas com evidências de código.
- `docs/atlashub/F0-TOPOLOGIA.md` — componentes, opções de hosting e custos estimados.
- `docs/atlashub/F0-INTEGRACOES.md` — matriz de convivência/substituição com Chatwoot, Evolution API, Typebot, n8n e Hermes.
- `docs/atlashub/F0-DEPLOY.md` — processo de build/deploy da **imagem AtlasHub**, estratégia de release/rollback, envs e ameaças.
- `docs/atlashub/F0-DECISAO.md` — resumo GO/NO-GO para montar o staging.

**Prova exigida:** código/commit exato, comandos e resultados verificáveis, sem incluir credenciais ou dados de clientes. Antes de modificar código, apresentar o inventário e riscos.

## 8. Nota de colaboração

Este documento é a **diretiva AtlasHub** para o fork e não altera a licença do upstream. Contribuições upstream continuam possíveis. Quando exigências específicas AtlasHub divergirem da doutrina editorial do projeto original, separar decisões locais em documentação/PRs, sem fingir que as funcionalidades já existem.

**Próximo passo operacional:** executar F0; só então iniciar staging F1.
