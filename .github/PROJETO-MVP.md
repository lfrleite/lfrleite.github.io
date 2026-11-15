# Projeto MVP — controle editorial e laboratórios

> **Atualizado em:** 15/11/2026. **Status:** planejamento em andamento.  
> Repositórios: [blog](https://github.com/lfrleite/lfrleite.github.io) · [scripts e laboratórios](https://github.com/lfrleite/Ruiz-Online).  
> **Política desta etapa:** não alterar artigos existentes nem executar alterações no Azure; organizar somente documentação de planejamento e Issues.

## 1. Objetivo e linha de base

Construir um portfólio público de artigos operacionais, demonstrados em laboratório próprio e com referências oficiais, para fortalecer as contribuições à comunidade Microsoft (foco em Azure e Microsoft 365).

- **Meta editorial:** 20 artigos em 2026, condicionados à validação técnica e à publicação efetiva.
- **Último artigo integralmente concluído segundo o autor:** 24/04/2026.
- **Baseline do repositório:** 28 arquivos Markdown em `_posts` (4 de 2025; 24 datados de 2026).
- **Situação conhecida:** 7 artigos anteriores considerados concluídos; 1 artigo AIB está publicado mas com execução final pendente; outros 20 estão em `published: false`.
- **Para alcançar 20 artigos de 2026:** existem 3 integralmente concluídos em 2026, portanto faltam **17** conclusões/publicações efetivas (incluindo a regularização do AIB, caso aprovado). Essa métrica não significa que 17 laboratórios já foram executados.
- **Regra de datas:** distinguir data de redação, execução de laboratório e publicação. Não registrar data passada como publicação efetiva quando não ocorreu.
- A base privada de conhecimento inclui **30 sugestões editoriais**; não se confundem com os **20 rascunhos** que atualmente existem em `_posts`. Não publicar conteúdo interno da base.

## 2. Painel de execução

| Prioridade | Frente | Escopo | Issue | Dependência / evidência |
| --- | --- | --- | --- | --- |
| P0 | Azure Image Builder | Artigo já público, incompleto | [#1](https://github.com/lfrleite/lfrleite.github.io/issues/1) | Confirmar RBAC, criar templates, builds, versions e VMs de teste |
| P1 | Resource Graph | Rascunhos 012 e 013 | [#2](https://github.com/lfrleite/lfrleite.github.io/issues/2) | KQL real, paginação, interpretação dos resultados |
| P1 | Update Manager | Rascunhos 009, 010, 011, 014, 015 | [#3](https://github.com/lfrleite/lfrleite.github.io/issues/3) | Avaliação/instalação, waves, pre-event, histórico |
| P1 | FinOps | Rascunhos 017, 018, 019, 020 | [#4](https://github.com/lfrleite/lfrleite.github.io/issues/4) | Start/Stop, órfãos, licenciamento e comprometimento |
| P1 | Policy e redes | Rascunhos 021, 022, 023, 024 | [#5](https://github.com/lfrleite/lfrleite.github.io/issues/5) | Remediation, NAT, Private Link/DNS e VNet Flow Logs |
| P2 | Pipeline e Gallery | Rascunhos 025, 026 | [#6](https://github.com/lfrleite/lfrleite.github.io/issues/6) | Depende do AIB concluído |
| P2 | Especializados | Rascunhos 016, 027, 028 | [#7](https://github.com/lfrleite/lfrleite.github.io/issues/7) | Hotpatch, Trusted Launch e AKS |
| P1 | Novo artigo | Debian 13 x Update Manager | [#8](https://github.com/lfrleite/lfrleite.github.io/issues/8) | Verificar suporte oficial e dados de laboratório |
| P0 | Editorial/MVP | Controle de evidências e publicações | [#9](https://github.com/lfrleite/lfrleite.github.io/issues/9) | Checklist e registro de contribuição |
| P0 | Scripts | Revisão de código auxiliar | [Ruiz-Online #5](https://github.com/lfrleite/Ruiz-Online/issues/5) | Conferir execução antes de incorporar ao post |

### Rascunhos localizados (`published: false`)

| Pasta de imagens | Arquivo em `_posts/2026/` | Frente | Status |
| --- | --- | --- | --- |
| 009 | `2026-10-06-azure-update-manager-do-zero.md` | Update Manager | Texto pronto para validar; prints ausentes |
| 010 | `2026-10-08-azure-update-manager-escopo-dinamico-tags.md` | Update Manager | Texto pronto para validar; prints ausentes |
| 011 | `2026-10-13-azure-update-manager-snapshot-pre-evento.md` | Update Manager | Texto pronto para validar; prints ausentes |
| 012 | `2026-10-15-azure-resource-graph-consultas-essenciais.md` | ARG | Texto pronto para validar; prints ausentes |
| 013 | `2026-10-20-azure-resource-graph-so-fim-de-suporte.md` | ARG | Texto pronto para validar; prints ausentes |
| 014 | `2026-10-22-azure-update-manager-relatorio-compliance.md` | Update Manager | Texto pronto para validar; prints ausentes |
| 015 | `2026-10-27-azure-update-manager-troubleshooting.md` | Update Manager | Texto pronto para validar; prints ausentes |
| 016 | `2026-10-29-hotpatch-windows-server-2025-azure.md` | Especializados | Texto pronto para validar; prints ausentes |
| 017 | `2026-11-03-start-stop-vms-azure-managed-identity.md` | FinOps | Texto pronto para validar; prints ausentes |
| 018 | `2026-11-05-azure-recursos-orfaos-resource-graph.md` | FinOps | Texto pronto para validar; prints ausentes |
| 019 | `2026-11-10-azure-hybrid-benefit-auditoria.md` | FinOps | Texto pronto para validar; prints ausentes |
| 020 | `2026-11-12-azure-reservations-vs-savings-plan.md` | FinOps | Texto pronto para validar; prints ausentes |
| 021 | `2026-11-17-azure-policy-tags-obrigatorias-herdadas.md` | Policy e redes | Texto pronto para validar; prints ausentes |
| 022 | `2026-11-19-azure-default-outbound-access-saida-explicita.md` | Policy e redes | Texto pronto para validar; prints ausentes |
| 023 | `2026-11-24-azure-private-endpoint-dns-privado.md` | Policy e redes | Texto pronto para validar; prints ausentes |
| 024 | `2026-11-26-azure-vnet-flow-logs-migracao.md` | Policy e redes | Texto pronto para validar; prints ausentes |
| 025 | `2026-12-01-azure-image-builder-github-actions-oidc.md` | Pipeline | Texto pronto para validar; prints ausentes |
| 026 | `2026-12-03-azure-compute-gallery-versionamento-limpeza.md` | Pipeline | Texto pronto para validar; prints ausentes |
| 027 | `2026-12-08-azure-trusted-launch-vms-existentes.md` | Especializados | Texto pronto para validar; prints ausentes |
| 028 | `2026-12-10-aks-node-image-upgrade-auto-upgrade.md` | Especializados | Texto pronto para validar; prints ausentes |

> Em 15/11/2026 as pastas `assets/img/009` até `assets/img/028` têm **somente uma imagem de capa cada**; o Markdown referencia capturas ainda inexistentes. Não inferir sucesso do laboratório pela presença do roteiro.

## 3. Plano de aproximadamente três semanas

**Janela de planejamento: 15/11 a 06/12/2026** (meta de trabalho, não promessa de publicação).

1. **Semana 1 — AIB e consultas de leitura:** concluir o AIB, registrar versão de imagem e teste real; executar ARG e tarefas FinOps simples.
2. **Semana 2 — Update Manager:** reutilizar o mesmo laboratório para avaliação, waves, snapshots, consultas de compliance e troubleshooting.
3. **Semana 3 — Governança e especializações:** Policy, redes, pipelines AIB e, conforme custo e capacidade de testes, Hotpatch, Trusted Launch e AKS.

A ordem muda quando um laboratório apresentar bloqueio técnico ou risco de custo. Priorizar artigos com procedimento validado, não a quantidade.

## 4. Definição de pronto (Definition of Done)

Um artigo só pode ser marcado como **validado para publicação** se todos os itens estiverem concluídos:

- [ ] Fontes oficiais Microsoft Learn verificadas e URLs acessíveis; limitações e pontos não confirmados descritos.
- [ ] Pré-requisitos e custos/limpeza definidos; commands realmente testados em laboratório próprio.
- [ ] Evidências mínimas: configuração inicial, execução, resultado/erro, validação funcional e cleanup quando aplicável.
- [ ] Prints próprios legíveis, compatíveis com o texto e **sem dados de clientes ou identificadores confidenciais**.
- [ ] Scripts correspondem à versão testada do repositório auxiliar e não expõem credenciais.
- [ ] Front matter Jekyll/Chirpy, título/slug, `Fala pessoALL!`, contexto anonimizado, troubleshooting, conclusão e documentação oficial.
- [ ] Texto para LinkedIn até 1.300 caracteres com três hashtags e link para artigo público.
- [ ] Data de publicação real, URL e impacto da contribuição registrados.

Estados sugeridos: **rascunho → lab pendente → lab em execução → evidências capturadas → revisão técnica/editorial → publicado**; usar **bloqueado** quando necessário.

## 5. Primeiro ponto de retomada: AIB

Estado observado em laboratório pessoal em 15/11:

- Resource Group e Azure Compute Gallery do laboratório pessoal existem.
- Providers registrados; VNet, três subnets (build, ACI, test), NAT Gateway e Managed Identity do laboratório existentes.
- Quatro Image Definitions (Windows Server 2022/2025, Ubuntu 24.04 e Debian 13) em `Succeeded`.
- **Ainda não existem** Image Templates, Image Versions nem VMs de validação nesse Resource Group.
- A consulta de RBAC usada inicialmente retornou erro por incompatibilidade entre `--all` e `--scope`; **não há diagnóstico concluído de acesso**.

Próxima ação: conferir as role assignments e a custom role da Managed Identity em **modo leitura**. Somente depois iniciar o template Ubuntu 24.04. Nenhuma etapa de build foi executada nesta fase de organização.

## 6. Regras para novas conversas

Para retomar um assunto sem carregar o histórico, abrir a Issue da frente e informar somente **número/link da Issue + resultado da última execução**. Os checklists e os links devem permanecer nela.

Este documento contém **apenas informações públicas ou fictícias**, nunca dados internos da base de conhecimento, arquivos corporativos, prints de clientes, IDs de assinaturas ou valores reais.
