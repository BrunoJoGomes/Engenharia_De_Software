# 0. Seleção de Escopo

## 0.1 Subsistemas e Atores (recapitulação)

O HealthLink é organizado em torno de três atores principais — **Paciente**, **Médico** e **Administrador da Clínica** — e três subsistemas funcionais:

| Subsistema | Atores principais |
|---|---|
| **Cadastro & Configuração** | Administrador, Médico |
| **Agendamento & Busca** | Paciente, Médico |
| **Teleconsulta & Prontuário** | Paciente, Médico |

## 0.2 Fatias Selecionadas

### Fatia 1 — Paciente busca médico e agenda consulta

**Histórias/requisitos cobertos:** RF-004 (Buscar Médicos), RF-005 (Agendar Consulta), RF-010 (Notificações de confirmação).

**Por que é representativa:**
- É **Must Have** absoluto — sem agendamento, a plataforma não tem razão de existir.
- Envolve **dois subsistemas** (Busca e Agendamento) e **dois atores** (Paciente ativa, Médico passivo via agenda configurada).
- Tem **regra de negócio não-trivial**: o sistema precisa verificar disponibilidade em tempo real, bloquear horários já reservados e confirmar o agendamento atomicamente para evitar conflitos de concorrência.

**O que esperamos aprender:**
- Como modelar um **fluxo com verificação de disponibilidade** e bloqueio de horário (race condition).
- Como representar a **orquestração entre paciente, sistema e serviço de notificação**.
- A estrutura de classes `Consulta`, `Agenda`, `SlotDisponivel` e seus relacionamentos.

---

### Fatia 2 — Ciclo de vida da consulta (do agendamento à conclusão)

**Histórias/requisitos cobertos:** RF-005 (Agendar Consulta), RF-006 (Teleconsulta), RF-007 (Registrar Consulta), RF-009 (Histórico Médico).

**Por que é representativa:**
- Envolve **três atores** (Paciente, Médico e Sistema de notificações) e atravessa todos os subsistemas.
- Tem **ciclo de vida com estados bem definidos**: `Agendada → Confirmada → Em Andamento → Concluída | Cancelada`.
- Captura a **transição crítica** de estado que não pode ser revertida (consulta em andamento não pode ser cancelada normalmente).

**O que esperamos aprender:**
- Como modelar um **objeto com estados complexos** e eventos que disparam transições.
- Como representar **regras de cancelamento** (pode cancelar até X horas antes) e **timeouts** (médico não entra na sala → consulta não realizada).
- A diferença entre o estado da `Consulta` e o estado da `Teleconsulta` (sala de vídeo).

---

### Fatia 3 — Médico conduz teleconsulta, registra anotações e emite receita

**Histórias/requisitos cobertos:** RF-006 (Teleconsulta), RF-007 (Registrar Consulta), RF-008 (Emitir Receita).

**Por que é representativa:**
- É o **core clínico** da plataforma — sem isso, o HealthLink seria apenas um sistema de agendamento.
- Envolve **múltiplos subsistemas**: videochamada (integração externa), prontuário eletrônico e geração de receita digital.
- Tem **regras de negócio específicas**: receita só pode ser emitida após consulta estar em andamento ou concluída; anotações são vinculadas ao paciente e visíveis apenas ao médico que atendeu.

**O que esperamos aprender:**
- Como modelar um **fluxo com integração externa** (serviço de vídeo) e fallback.
- Como representar **decisões condicionais** (emitir ou não receita, prescrever um ou mais medicamentos).
- A relação entre `Consulta`, `Prontuario`, `Anotacao` e `Receita` no diagrama de classes.

---

## 0.3 Cobertura dos Critérios

| Critério | Fatia 1 | Fatia 2 | Fatia 3 |
|---|---|---|---|
| Must Have do MoSCoW | ✅ RF-004, RF-005 | ✅ RF-005, RF-006, RF-007 | ✅ RF-006, RF-007 + Should Have RF-008 |
| Múltiplos subsistemas/atores | ✅ 2 subsistemas, 2 atores | ✅ 3 atores, todos subsistemas | ✅ 3 subsistemas, integração externa |
| Regras de negócio não-triviais | ✅ Conflito de horário, atomicidade | ✅ Estados + timeouts + cancelamento | ✅ Receita condicional, acesso restrito |

Todos os critérios obrigatórios estão cobertos nas três fatias.

---

## 0.4 O que Fica de Fora (e Por Quê)

As seguintes funcionalidades do Trabalho 2 **não serão modeladas** neste trabalho:

- **RF-001 (Gerenciar Clínica) e RF-002 (Cadastrar Médicos):** fluxos de configuração administrativa, predominantemente CRUD. Não há regra de negócio significativa que justifique modelagem aprofundada.
- **RF-003 (Configurar Agenda do Médico):** funcionalidade de suporte à Fatia 1 — a agenda configurada pelo médico é pré-condição do agendamento, mas o ato de configurar em si é um CRUD de slots de disponibilidade. Fica implícito como pré-condição das fatias.
- **RF-009 (Histórico Médico):** parcialmente coberto pela Fatia 2 (a conclusão da consulta registra no histórico). A visualização isolada do histórico é CRUD de leitura simples, sem regra de negócio nova.
- **RF-010 (Notificações):** tratado como efeito colateral das Fatias 1 e 2 (confirmação de agendamento, lembrete). Modelar o subsistema de notificações isoladamente não acrescentaria aprendizado relevante.
- **RF-011 (Relatórios):** funcionalidade administrativa, Could Have no MoSCoW. Sem regra de negócio de domínio relevante.
