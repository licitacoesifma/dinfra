# SIGOBRAS — Sistema de Gestão de Obras · NJP/DINFRA/IFMA

Sistema de gestão dos processos técnicos do **Núcleo de Projetos (NJP)** da Diretoria de Infraestrutura (DINFRA) do IFMA. Controla a distribuição de processos SUAP entre servidores por especialidade técnica, o checklist de entregáveis de cada uma e a carga de trabalho da equipe, incluindo um cronograma visual (Gantt).

> Aplicação **100% front-end**, em um único arquivo HTML autocontido, sem build, sem servidor e sem dependências externas.

## Como usar

Basta abrir o arquivo no navegador — não requer instalação, servidor ou conexão com internet:

```bash
# Windows — duplo clique no Explorer, ou:
start index.html
```

Não há passo de build: `index.html` é o produto final.

## Stack

- HTML5 + CSS3 puro + JavaScript ES6 *vanilla*
- **Zero dependências externas** (sem frameworks, sem CDN, sem bundler) — decisão deliberada para manter o sistema portátil e utilizável offline
- Todo o estado vive em memória (variáveis JS); **não há persistência** — os dados voltam ao estado inicial a cada `F5` (ver [Limitações conhecidas](#limitações-conhecidas))

## Modelo de negócio

Um processo SUAP chega ao NJP e é dividido em **demandas por especialidade técnica**. Cada demanda tem seu próprio servidor responsável e seu próprio checklist de entregáveis — não existe mais o modelo antigo de "1 processo → 1 servidor → fases lineares".

### Tipo do processo e regra de fases

Cada processo é classificado como:

| Tipo | Disciplinas | Regra |
|---|---|---|
| **Edificação** | ARQ, EST, HS, ELE, LOG, CIP, CLI | Cada disciplina percorre as fases **próprias de Edificação + as fases de Implantação** da mesma disciplina (quando existirem) |
| **Implantação** | ARQ, HS, ELE, LOG, CIP, ORC | Cada disciplina percorre **somente** as fases inerentes à Implantação |

EST e CLI não têm fases equivalentes em Implantação, então nesses casos a mesclagem não adiciona itens. Os entregáveis de cada disciplina/tipo são definidos nas constantes `CHECKLIST_EDIFICACAO` e `CHECKLIST_IMPLANTACAO` (extraídas da planilha de referência `Check List de Projetos.xlsx`), e mesclados centralmente pela função `checklistDefsForTipo(tipo, disc)`.

### Disciplinas

| Sigla | Disciplina |
|---|---|
| ARQ | Arquitetura |
| EST | Estrutural |
| HS | Hidrossanitário |
| ELE | Elétrica |
| LOG | Lógica |
| CIP | CIP / Incêndio |
| CLI | Climatização |
| ORC | Orçamento Final |

### Status de uma demanda

`Aguardando atribuição` → `Em execução` → `Concluída`, ou `Bloqueada` quando algum item do checklist tem impedimento registrado.

### Regra de limite (Arquitetos/Urbanistas)

Servidores marcados como `ehArqUrbProjetos` têm um **limite configurável** de processos de projeto simultâneos (editável pelo Chefe NJP em tempo real). Ao atingir o limite, uma nova atribuição exige **deliberação de fila** — decisão formal do Chefe NJP, com justificativa auditável de no mínimo 20 caracteres, vinculada aos NUPs envolvidos.

## Módulos / telas

Navegação pela barra lateral, controlada por `goView(v)`:

1. **Dashboard Executivo** — visão macro: contagem de processos, status, especialidades pendentes/bloqueadas/concluídas.
2. **Gestão NJP** — lista de processos SUAP, expansível por especialidade, com checklist de entregáveis, atribuição de servidor e registro de impedimento/conclusão.
3. **Servidores & Carga** — carga de trabalho por servidor, capacidade, fila de prioridade e alerta de limite atingido.
4. **Gantt de Projetos** — cronograma visual (Jan–Dez do ano corrente), duas visões:
   - **Por Processo**: uma linha por processo + uma linha por especialidade, mostrando também **em qual fase (entregável) o servidor está** naquela demanda específica.
   - **Por Servidor**: agrupa as demandas por responsável, com indicador de carga circular.
5. **Usuários do Sistema** *(visível apenas para o perfil Gerencial)* — cadastro e gestão dos usuários com acesso ao SIGOBRAS: adicionar, editar, desativar/reativar e excluir usuários.

### Perfis de acesso (simulados)

Selecionável no topo da tela (`changeRole`), sem autenticação real — apenas ajusta o que é exibido/editável:

| Perfil | Quem | Pode fazer |
|---|---|---|
| **Gerencial** | Chefe NJP | Criar processos, atribuir servidores, deliberar fila, editar limites, gerenciar usuários |
| **Estratégico** | Reitoria | Visão somente leitura |
| **Alimentador** | Servidor Técnico | Concluir/registrar impedimento nos itens do próprio checklist |

A tela **Usuários do Sistema** e o item de navegação correspondente ficam ocultos para os perfis Estratégico e Alimentador. O acesso direto via `goView('usuarios')` também é bloqueado.

## Estrutura do arquivo

`index.html` (~1840 linhas) é organizado em blocos comentados dentro de um único `<script>`:

```
DISCIPLINAS, CHECKLIST_EDIFICACAO, CHECKLIST_IMPLANTACAO   → catálogos estáticos
usuarios                                                    → cadastro de usuários do sistema
njpServidores, processos                                    → dados/estado (seed inicial)
HELPERS                                                     → getServidor, statusDemanda, progressoDemanda...
NAVEGAÇÃO                                                   → goView, changeRole, _setAdminNav, updateBadges
USUÁRIOS                                                    → renderUsuarios, abrirModalNovoUsuario,
                                                               editarUsuario, confirmUsuario,
                                                               toggleAtivoUsuario, excluirUsuario
RENDER PRINCIPAL / DASHBOARD / NJP / SERVIDORES             → renderDashboard, renderNJP, renderServidores
MODAIS                                                      → Novo Processo, Atribuir Servidor, Deliberar Fila,
                                                               Impedimento, Concluir Item, Usuário
GANTT                                                       → renderGantt, renderGanttProcessos, renderGanttServidores,
                                                               faseAtualDemanda
TOGGLE / HELPERS                                            → closeModal, showToast, editLimite
```

O padrão de renderização é simples: cada `render*()` retorna uma *string* HTML, e `render()` a injeta em `#mainContent.innerHTML` a cada mudança de estado — não há virtual DOM nem framework reativo.

## Limitações conhecidas

- **Sem persistência**: todo o estado é perdido ao recarregar a página (variáveis JS em memória). Planejado: Firebase Firestore + GitHub Pages.
- **Sem autenticação real**: o perfil de acesso é selecionado manualmente no topo da tela. A integração com Firebase Auth (login via conta @ifma.edu.br) está planejada junto à persistência.
- **Sem integração real com o SUAP**: NUPs e dados são inseridos manualmente.
- Nomes de servidores das disciplinas não-ARQ são **fictícios**, pendentes de substituição pelos nomes reais.
- Datas de início das especialidades herdam a data de entrada do processo (`dataEntrada`) — ainda não é possível definir uma data de início por especialidade.
- Sem exportação (PDF/Excel) dos relatórios ou do Gantt.
- Sem indicação visual diferenciada (ex. hachura) para barras bloqueadas no Gantt.

## Roadmap

- [ ] Persistência + autenticação real (Firebase Firestore + Firebase Auth + GitHub Pages)
- [ ] Substituição dos nomes fictícios pelos servidores reais (EST, HS, ELE, LOG, CIP, CLI)
- [ ] Integração com API REST do SUAP
- [ ] Notificação por e-mail ao reordenar fila
- [ ] Exportação PDF/Excel do Gantt e relatórios
- [ ] Data de início editável por especialidade
- [ ] Hachura visual para impedimento no Gantt
- [ ] Modo PWA / offline-first
- [ ] Modelar seções "Estudos Iniciais" e "Documentos para Licitação" da planilha

## Origem dos dados

Os checklists de entregáveis por disciplina foram extraídos da planilha de referência `Check List de Projetos.xlsx`, fornecida pelo NJP/DINFRA, que também traz as seções *Estudos Iniciais* e *Documentos para Licitação* — ainda não modeladas no sistema.
