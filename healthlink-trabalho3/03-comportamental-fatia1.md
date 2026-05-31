# 3. Modelagem Comportamental — Fatia 1

## Fatia 1: Paciente busca médico e agenda consulta

**Tipo de diagrama escolhido: Diagrama de Sequência**

**Justificativa:** Escolhemos o diagrama de sequência para esta fatia porque ela envolve coordenação entre múltiplos componentes em ordem temporal precisa: o paciente interage com o frontend, que aciona o backend, que consulta a agenda do médico e, ao confirmar o agendamento, dispara o serviço de notificação. Existe ainda um caminho crítico de erro (slot já ocupado por concorrência) que é melhor visualizado com fragmentos `alt`. Um diagrama de estados seria menos adequado aqui porque o foco não é o ciclo de vida de um objeto, mas a troca de mensagens entre participantes.

---

## Diagrama de Sequência

```mermaid
sequenceDiagram
    actor Paciente
    participant Frontend
    participant BackendAgendamento as Backend (Agendamento)
    participant BackendAgenda as Backend (Agenda)
    participant BancoDados as Banco de Dados
    participant ServicoNotificacao as Serviço de Notificação

    Paciente->>Frontend: Informa especialidade e localidade
    Frontend->>BackendAgendamento: GET /medicos?especialidade=X&localidade=Y
    BackendAgendamento->>BancoDados: SELECT médicos disponíveis
    BancoDados-->>BackendAgendamento: Lista de médicos
    BackendAgendamento-->>Frontend: Lista de médicos com disponibilidade
    Frontend-->>Paciente: Exibe médicos encontrados

    Paciente->>Frontend: Seleciona médico e clica em "Ver horários"
    Frontend->>BackendAgenda: GET /agenda/{medicoId}/slots-disponiveis
    BackendAgenda->>BancoDados: SELECT slots WHERE disponivel = true AND medico_id = X
    BancoDados-->>BackendAgenda: Lista de slots disponíveis
    BackendAgenda-->>Frontend: Slots disponíveis
    Frontend-->>Paciente: Exibe calendário com horários

    Paciente->>Frontend: Seleciona slot e tipo (presencial/teleconsulta)
    Frontend->>BackendAgendamento: POST /consultas {pacienteId, slotId, tipo}

    Note over BackendAgendamento, BancoDados: Verificação e bloqueio atômico (transação)

    BackendAgendamento->>BancoDados: BEGIN TRANSACTION
    BackendAgendamento->>BancoDados: SELECT slot FOR UPDATE WHERE id = slotId AND disponivel = true
    
    alt Slot disponível
        BancoDados-->>BackendAgendamento: Slot encontrado e bloqueado
        BackendAgendamento->>BancoDados: INSERT consulta (status = AGENDADA)
        BackendAgendamento->>BancoDados: UPDATE slot SET disponivel = false
        BackendAgendamento->>BancoDados: COMMIT
        BancoDados-->>BackendAgendamento: OK
        BackendAgendamento--)ServicoNotificacao: Evento: consultaAgendada(consultaId)
        ServicoNotificacao--)Paciente: E-mail: Consulta confirmada com detalhes
        ServicoNotificacao--)Medico: E-mail: Nova consulta agendada
        BackendAgendamento-->>Frontend: 201 Created {consultaId, detalhes}
        Frontend-->>Paciente: "Consulta agendada com sucesso!"
    else Slot já ocupado (conflito de concorrência)
        BancoDados-->>BackendAgendamento: Nenhuma linha retornada
        BackendAgendamento->>BancoDados: ROLLBACK
        BackendAgendamento-->>Frontend: 409 Conflict {erro: "Horário não disponível"}
        Frontend->>BackendAgenda: GET /agenda/{medicoId}/slots-disponiveis
        BackendAgenda-->>Frontend: Lista atualizada de slots
        Frontend-->>Paciente: "Horário indisponível. Escolha outro horário."
    end
```

---

## Notas sobre o diagrama

**Transação atômica:** O passo crítico de verificação e reserva do slot é executado dentro de uma transação com `SELECT FOR UPDATE` para garantir que dois pacientes não reservem o mesmo horário simultaneamente. Isso é a regra de negócio mais importante desta fatia.

**Notificações assíncronas:** As notificações (E-mails) são disparadas como eventos assíncronos (`--)` em vez de chamadas síncronas. Isso evita que uma falha no serviço de e-mail impeça a confirmação do agendamento.

**Atores envolvidos:** Paciente, Frontend, dois módulos de backend (separados por domínio), Banco de Dados e Serviço de Notificação — demonstrando a interação entre múltiplos componentes exigida pela seleção de escopo.
