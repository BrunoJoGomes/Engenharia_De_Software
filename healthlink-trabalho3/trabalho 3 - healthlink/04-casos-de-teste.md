# 4. Casos de Teste

Seis casos de teste seguindo o padrão IEEE 829-2008, dois por fatia. Para cada fatia, pelo menos um caso de fronteira ou caminho de erro.

---

## Fatia 1 — Paciente busca médico e agenda consulta

### TC-FATIA1-01 — Agendamento bem-sucedido (caminho feliz)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA1-01 |
| **Fatia / Requisito** | Fatia 1 — RF-004 (Buscar Médicos), RF-005 (Agendar Consulta) |
| **Pré-condições** | Paciente autenticado no sistema; médico cadastrado na especialidade "Cardiologia" com ao menos um slot disponível para os próximos 7 dias; tipo de consulta: teleconsulta. |
| **Dados de entrada** | Especialidade: "Cardiologia"; localidade: "São Paulo/SP"; slot selecionado: 2026-06-10 às 10:00; tipo: TELECONSULTA. |
| **Passos** | 1. Paciente acessa "Buscar médicos e clínicas". 2. Informa especialidade "Cardiologia" e localidade "São Paulo". 3. Sistema exibe lista de médicos disponíveis. 4. Paciente seleciona um médico e clica em "Ver horários". 5. Sistema exibe calendário com slots disponíveis. 6. Paciente seleciona o slot de 10:00 do dia 10/06. 7. Paciente confirma o agendamento como teleconsulta. |
| **Resultado esperado** | Sistema registra a consulta com status `AGENDADA`; slot é marcado como `disponivel = false`; paciente recebe e-mail de confirmação com data, horário e nome do médico; médico recebe notificação de nova consulta. |
| **Critério de aprovação** | (a) Consulta aparece no histórico do paciente com status AGENDADA. (b) Slot não aparece mais disponível na agenda do médico. (c) Dois e-mails enviados (paciente e médico) em até 60 segundos. |
| **Severidade em caso de falha** | **Crítica** — falha no agendamento inviabiliza a função principal da plataforma. |

---

### TC-FATIA1-02 — Conflito de concorrência no agendamento (caso de fronteira)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA1-02 |
| **Fatia / Requisito** | Fatia 1 — RF-005 (Agendar Consulta), regra de atomicidade de slot |
| **Pré-condições** | Dois pacientes autenticados (Paciente A e Paciente B) simultaneamente; um único slot disponível no dia 10/06 às 14:00 para o mesmo médico. |
| **Dados de entrada** | Paciente A e Paciente B submetem agendamento para o mesmo slotId simultaneamente (diferença de menos de 200ms entre as requisições). |
| **Passos** | 1. Paciente A seleciona slot 14:00 e clica em "Confirmar". 2. Paciente B seleciona o mesmo slot 14:00 e clica em "Confirmar" milissegundos depois. 3. Sistema processa ambas as requisições. |
| **Resultado esperado** | Exatamente uma das requisições é bem-sucedida (HTTP 201). A outra recebe HTTP 409 Conflict com mensagem "Horário indisponível". O slot fica reservado apenas para o paciente que chegou primeiro. Nenhuma consulta é criada em estado inconsistente. |
| **Critério de aprovação** | (a) Banco de dados contém exatamente 1 consulta para aquele slot. (b) Slot marcado como `disponivel = false`. (c) O paciente recusado vê a mensagem de erro e o calendário atualizado sem o slot ocupado. |
| **Severidade em caso de falha** | **Crítica** — double-booking gera dois pacientes para o mesmo horário, comprometendo o atendimento e a confiança na plataforma. |

---

## Fatia 2 — Ciclo de vida da consulta

### TC-FATIA2-01 — Cancelamento dentro do prazo permitido (caminho feliz)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA2-01 |
| **Fatia / Requisito** | Fatia 2 — RF-005 (Agendar Consulta), regra de cancelamento |
| **Pré-condições** | Consulta existente com status `AGENDADA` para 2026-06-10 às 10:00; horário atual é 2026-06-09 às 20:00 (14 horas de antecedência — dentro do prazo de 2h mínimo). Paciente autenticado. |
| **Dados de entrada** | Paciente seleciona a consulta no histórico e clica em "Cancelar". Motivo: "Impossibilidade de comparecer". |
| **Passos** | 1. Paciente acessa "Histórico de consultas". 2. Localiza a consulta agendada. 3. Clica em "Cancelar consulta". 4. Informa motivo e confirma. |
| **Resultado esperado** | Consulta muda de status `AGENDADA` para `CANCELADA`; slot do médico é liberado (`disponivel = true`); paciente e médico recebem e-mail de cancelamento; motivo fica registrado na consulta. |
| **Critério de aprovação** | (a) Status da consulta no banco é CANCELADA. (b) Slot aparece disponível novamente na agenda do médico. (c) Motivo de cancelamento persistido. (d) E-mails enviados para ambas as partes. |
| **Severidade em caso de falha** | **Alta** — falha no cancelamento prende o slot indevidamente, prejudicando outros pacientes. |

---

### TC-FATIA2-02 — Tentativa de cancelamento de consulta em andamento (caso de fronteira/erro)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA2-02 |
| **Fatia / Requisito** | Fatia 2 — Regra de transição de estado: `EmAndamento` não permite cancelamento |
| **Pré-condições** | Consulta com status `EM_ANDAMENTO` (médico já iniciou o atendimento). Paciente autenticado e na sala. |
| **Dados de entrada** | Paciente tenta acessar a opção "Cancelar consulta" via API diretamente (simulando chamada maliciosa ou bug de frontend). Requisição: `PATCH /consultas/{id}/cancelar`. |
| **Passos** | 1. Consulta está com status EM_ANDAMENTO. 2. Paciente (ou frontend com bug) envia requisição de cancelamento. |
| **Resultado esperado** | Sistema retorna HTTP 422 Unprocessable Entity com mensagem "Não é possível cancelar uma consulta em andamento". Status da consulta permanece `EM_ANDAMENTO`. Nenhuma alteração persiste no banco. |
| **Critério de aprovação** | (a) Status permanece EM_ANDAMENTO após a tentativa. (b) Resposta da API contém código de erro 422 e mensagem explicativa. (c) Nenhum evento de cancelamento é disparado (sem e-mail enviado). |
| **Severidade em caso de falha** | **Crítica** — permitir cancelamento durante atendimento interrompe consulta em curso e gera inconsistência no prontuário. |

---

## Fatia 3 — Teleconsulta com registro de anotações e receita

### TC-FATIA3-01 — Emissão de receita após teleconsulta (caminho feliz)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA3-01 |
| **Fatia / Requisito** | Fatia 3 — RF-006 (Teleconsulta), RF-007 (Registrar Consulta), RF-008 (Emitir Receita) |
| **Pré-condições** | Consulta com status `EM_ANDAMENTO`; médico autenticado e na sala de teleconsulta; paciente conectado. |
| **Dados de entrada** | Anotação: "Paciente apresenta hipertensão leve. Pressão: 140/90."; Diagnóstico: "CID I10 - Hipertensão primária". Item de receita: Medicamento: "Losartana 50mg", Dosagem: "1 comprimido", Posologia: "1x ao dia pela manhã", Quantidade: 30. |
| **Passos** | 1. Médico preenche campo de anotação com diagnóstico e observações. 2. Médico clica em "Salvar anotação". 3. Médico clica em "Emitir receita". 4. Médico adiciona 1 item (Losartana). 5. Médico assina digitalmente e clica em "Enviar receita ao paciente". 6. Médico clica em "Encerrar consulta". |
| **Resultado esperado** | Anotação salva e vinculada à consulta e ao prontuário do paciente; receita gerada em PDF com assinatura digital do médico; receita enviada por e-mail ao paciente; receita disponível no histórico do paciente; consulta muda para status `CONCLUÍDA`. |
| **Critério de aprovação** | (a) Anotação aparece no prontuário do paciente, visível apenas ao médico que atendeu. (b) Receita em PDF gerada com dados corretos (médico, paciente, medicamento, data). (c) E-mail com receita entregue ao paciente. (d) Status da consulta = CONCLUÍDA. |
| **Severidade em caso de falha** | **Alta** — perda de receita ou anotação compromete o histórico clínico e a segurança do paciente. |

---

### TC-FATIA3-02 — Tentativa de emitir receita antes de iniciar a consulta (caso de fronteira/erro)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA3-02 |
| **Fatia / Requisito** | Fatia 3 — RF-008 (Emitir Receita), regra de pré-condição: consulta deve estar EM_ANDAMENTO ou CONCLUÍDA |
| **Pré-condições** | Consulta com status `AGENDADA` (ainda não iniciada). Médico autenticado. |
| **Dados de entrada** | Médico tenta acessar o endpoint de emissão de receita via API: `POST /consultas/{id}/receitas` com payload de receita válido. |
| **Passos** | 1. Consulta está com status AGENDADA. 2. Médico (ou integração externa) tenta criar uma receita para essa consulta. |
| **Resultado esperado** | Sistema retorna HTTP 422 com mensagem "Receita só pode ser emitida durante ou após a consulta". Nenhuma receita é criada no banco. O prontuário do paciente permanece inalterado. |
| **Critério de aprovação** | (a) Nenhum registro de receita criado no banco para aquela consultaId. (b) Resposta da API: 422 com mensagem clara. (c) Prontuário do paciente sem alterações. |
| **Severidade em caso de falha** | **Alta** — emissão de receita sem consulta realizada é uma violação clínica e legal grave. |
