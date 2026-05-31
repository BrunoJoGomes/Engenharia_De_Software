# 5. Rastreabilidade

A tabela abaixo conecta todos os artefatos produzidos neste trabalho, permitindo verificar que cada fatia é completamente coberta por requisito, classes, entidades de banco, diagrama comportamental e casos de teste.

## Tabela Principal

| Fatia | Requisito(s) | Classes envolvidas | Entidades MER | Diagrama comportamental | Casos de teste |
|---|---|---|---|---|---|
| **Fatia 1** — Paciente busca médico e agenda consulta | RF-004, RF-005, RF-010 | `Paciente`, `Medico`, `Clinica`, `Agenda`, `SlotDisponivel`, `Consulta`, `ServicoNotificacao` | `PACIENTE`, `MEDICO`, `CLINICA`, `AGENDA`, `SLOT_DISPONIVEL`, `CONSULTA` | Sequência ([Seção 3 — Fatia 1](03-comportamental-fatia1.md)) | TC-FATIA1-01, TC-FATIA1-02 |
| **Fatia 2** — Ciclo de vida da consulta | RF-005, RF-006, RF-007, RF-009 | `Consulta`, `Teleconsulta`, `Paciente`, `Medico`, `Prontuario`, `ServicoNotificacao` | `CONSULTA`, `TELECONSULTA`, `PRONTUARIO` | Estados ([Seção 3 — Fatia 2](03-comportamental-fatia2.md)) | TC-FATIA2-01, TC-FATIA2-02 |
| **Fatia 3** — Teleconsulta com anotações e receita | RF-006, RF-007, RF-008 | `Medico`, `Paciente`, `Consulta`, `Teleconsulta`, `Prontuario`, `Anotacao`, `Receita`, `ItemReceita`, `ServicoVideo` | `CONSULTA`, `TELECONSULTA`, `PRONTUARIO`, `ANOTACAO`, `RECEITA`, `ITEM_RECEITA` | Atividades ([Seção 3 — Fatia 3](03-comportamental-fatia3.md)) | TC-FATIA3-01, TC-FATIA3-02 |

---

## Mapeamento Requisitos → Fatias

| Requisito | Prioridade | Fatia(s) que cobrem |
|---|---|---|
| RF-004 — Buscar Médicos | Must Have | Fatia 1 |
| RF-005 — Agendar Consulta | Must Have | Fatia 1, Fatia 2 |
| RF-006 — Teleconsulta | Must Have | Fatia 2, Fatia 3 |
| RF-007 — Registrar Consulta | Must Have | Fatia 2, Fatia 3 |
| RF-008 — Emitir Receita | Should Have | Fatia 3 |
| RF-009 — Histórico Médico | Must Have | Fatia 2 (conclusão persiste no prontuário) |
| RF-010 — Notificações | Should Have | Fatia 1 (confirmação), Fatia 2 (cancelamento) |

---

## Mapeamento Tipos de Diagrama → Fatias

| Tipo de diagrama | Fatia | Justificativa da escolha |
|---|---|---|
| Sequência | Fatia 1 | Foco em troca de mensagens entre componentes com ordem temporal e fragmentos alt para conflito de horário |
| Estados | Fatia 2 | Foco no ciclo de vida do objeto Consulta com transições, guardas e timeouts |
| Atividades | Fatia 3 | Foco em fluxo com decisões condicionais, múltiplos atores em swimlanes e loop de itens de receita |

Os três tipos de diagrama exigidos (cobertura de pelo menos 2 dos 3) são atendidos com os três tipos distintos utilizados.

---

## Mapeamento Classes → Entidades MER

| Classe (Diagrama de Classes) | Entidade(s) MER | Observação |
|---|---|---|
| `Usuario` (abstrata) | `USUARIO` | Tabela base da estratégia de herança por subclasse |
| `Paciente` | `PACIENTE` | Tabela especializada com FK para USUARIO |
| `Medico` | `MEDICO` | Tabela especializada com FK para USUARIO e CLINICA |
| `Administrador` | `ADMINISTRADOR` | Tabela especializada com FK para USUARIO e CLINICA |
| `Clinica` | `CLINICA` | Mapeamento direto |
| `Agenda` | `AGENDA` | Mapeamento direto |
| `SlotDisponivel` | `SLOT_DISPONIVEL` | Mapeamento direto |
| `Consulta` | `CONSULTA` | Mapeamento direto |
| `Teleconsulta` | `TELECONSULTA` | Mapeamento direto |
| `Prontuario` | `PRONTUARIO` | Materializado como tabela para controle de acesso |
| `Anotacao` | `ANOTACAO` | Mapeamento direto |
| `Receita` | `RECEITA` | Mapeamento direto |
| `ItemReceita` | `ITEM_RECEITA` | Mapeamento direto |
| `ServicoVideo` (interface) | — | Não persistida; representa integração externa |
| `ServicoNotificacao` (interface) | — | Não persistida; representa integração externa |
