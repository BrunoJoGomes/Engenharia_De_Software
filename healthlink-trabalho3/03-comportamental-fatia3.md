# 3. Modelagem Comportamental — Fatia 3

## Fatia 3: Médico conduz teleconsulta, registra anotações e emite receita

**Tipo de diagrama escolhido: Diagrama de Atividades**

**Justificativa:** Escolhemos o diagrama de atividades para esta fatia porque o fluxo envolve **decisões condicionais**, **ações paralelas** e **múltiplos atores em raias distintas** (médico, paciente e sistema de vídeo). O médico conduz o atendimento, o paciente participa passivamente na sala, e o sistema de vídeo opera em paralelo gerenciando a conexão. Ao final, há uma bifurcação: o médico decide se emite ou não uma receita, e se registra ou não anotações — ambas são opcionais mas têm regras. Um diagrama de atividades com swimlanes deixa visível quem faz o quê e em que ordem, o que um diagrama de estados não capturaria com a mesma clareza.

---

## Diagrama de Atividades

```mermaid
flowchart TD
    inicio([Consulta em status CONFIRMADA]) --> medico_acessa

    subgraph MEDICO ["🩺 Médico"]
        medico_acessa[Acessa painel de consultas]
        medico_acessa --> clica_iniciar[Clica em 'Iniciar Consulta']
    end

    subgraph SISTEMA ["⚙️ Sistema"]
        valida_horario{Horário de início\natingido?}
        cria_sala[Cria sala de vídeo\nno ServicoVideo]
        gera_links[Gera links para\nMédico e Paciente]
        atualiza_status[Atualiza Consulta\npara EM_ANDAMENTO]
        erro_horario[Exibe: 'Consulta ainda\nnão pode ser iniciada']
    end

    subgraph PACIENTE ["👤 Paciente"]
        recebe_link[Recebe link de acesso\nvia e-mail/notificação]
        entra_sala[Entra na sala de vídeo]
    end

    subgraph VIDEO ["📹 Serviço de Vídeo"]
        sala_criada[Sala criada com ID único]
        monitor[Monitora qualidade\nda conexão]
        encerra_sala[Encerra sala]
    end

    clica_iniciar --> valida_horario
    valida_horario -- Não --> erro_horario
    erro_horario --> medico_acessa

    valida_horario -- Sim --> cria_sala
    cria_sala --> sala_criada
    sala_criada --> gera_links
    gera_links --> atualiza_status
    gera_links --> recebe_link

    atualiza_status --> medico_entra[Médico entra na sala]
    recebe_link --> entra_sala
    medico_entra --> monitor
    entra_sala --> monitor

    monitor --> atendimento[Atendimento em andamento\nMédico e Paciente conversam]

    atendimento --> decisao_anotacao{Médico deseja\nregistrar anotação?}

    decisao_anotacao -- Sim --> preenche_anotacao[Preenche: diagnóstico,\nobservações, texto livre]
    preenche_anotacao --> salva_anotacao[Sistema salva Anotacao\nvinculada à Consulta\ne ao Prontuário]
    salva_anotacao --> decisao_receita

    decisao_anotacao -- Não --> decisao_receita{Médico deseja\nemitir receita?}

    decisao_receita -- Sim --> preenche_receita[Preenche itens da receita:\nmedicamento, dosagem, posologia]
    preenche_receita --> add_item{Adicionar\noutro item?}
    add_item -- Sim --> preenche_receita
    add_item -- Não --> assina_receita[Médico assina receita digitalmente]
    assina_receita --> envia_receita[Sistema envia receita\nao paciente por e-mail\ne vincula ao Prontuário]
    envia_receita --> encerra_consulta

    decisao_receita -- Não --> encerra_consulta

    encerra_consulta[Médico clica em 'Encerrar Consulta'] --> encerra_sala
    encerra_sala --> atualiza_concluida[Sistema atualiza\nConsulta para CONCLUÍDA]
    atualiza_concluida --> fim([Consulta concluída —\nProntuário disponível])
```

---

## Notas sobre o diagrama

**Swimlanes (raias):** O diagrama usa quatro raias — Médico, Sistema, Paciente e Serviço de Vídeo. Isso torna explícito que a criação da sala e o envio de links acontecem no sistema, enquanto o médico e o paciente aguardam. A separação de responsabilidades é visível sem necessidade de texto adicional.

**Validação de horário:** A atividade `valida_horario` representa uma regra de negócio real — o sistema não deve permitir que o médico inicie uma consulta antes do horário marcado (com uma janela de tolerância de, por exemplo, 5 minutos).

**Anotação e receita são opcionais e independentes:** O fluxo modela ambas como decisões separadas. O médico pode registrar uma anotação sem emitir receita, emitir receita sem anotação, ambas, ou nenhuma. Isso reflete a realidade clínica onde nem toda consulta resulta em prescrição.

**Loop de itens de receita:** O nó `add_item` com retorno para `preenche_receita` representa que uma receita pode conter múltiplos medicamentos, cada um com dosagem e posologia específicas — mapeando diretamente para a classe `ItemReceita` do diagrama de classes.

**Integração com `ServicoVideo`:** A raia de serviço de vídeo isola a dependência externa. Se o serviço de vídeo falhar, o impacto é contido — o sistema pode tratar o erro na transição `cria_sala → sala_criada` sem afetar o restante do fluxo (consulta presencial não passa por essa raia).
