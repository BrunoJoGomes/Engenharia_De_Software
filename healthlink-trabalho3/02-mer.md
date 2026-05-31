# 2. Modelo Entidade-Relacionamento (MER)

O MER abaixo contempla apenas as entidades persistentes das três fatias modeladas. Notação Crow's Foot (usada consistentemente ao longo do documento).

## 2.1 Diagrama

```mermaid
erDiagram

    USUARIO {
        uuid id PK
        varchar(100) nome
        varchar(150) email
        varchar(255) senha_hash
        varchar(20) telefone
        varchar(20) tipo_usuario
        timestamp data_cadastro
    }

    PACIENTE {
        uuid id PK
        uuid usuario_id FK
        varchar(14) cpf
        date data_nascimento
        varchar(50) cartao_saude
    }

    MEDICO {
        uuid id PK
        uuid usuario_id FK
        uuid clinica_id FK
        varchar(20) crm
        varchar(100) especialidade
        decimal(10_2) valor_consulta
    }

    ADMINISTRADOR {
        uuid id PK
        uuid usuario_id FK
        uuid clinica_id FK
    }

    CLINICA {
        uuid id PK
        varchar(100) nome
        varchar(18) cnpj
        varchar(300) endereco
        varchar(20) telefone
    }

    AGENDA {
        uuid id PK
        uuid medico_id FK
    }

    SLOT_DISPONIVEL {
        uuid id PK
        uuid agenda_id FK
        timestamp data_hora_inicio
        timestamp data_hora_fim
        boolean disponivel
        varchar(20) tipo_atendimento
    }

    CONSULTA {
        uuid id PK
        uuid paciente_id FK
        uuid medico_id FK
        uuid slot_id FK
        varchar(20) tipo
        varchar(20) status
        timestamp data_agendamento
        timestamp data_realizacao
        text motivo_cancelamento
    }

    TELECONSULTA {
        uuid id PK
        uuid consulta_id FK
        varchar(100) sala_id
        text link_paciente
        text link_medico
        timestamp data_hora_inicio
        timestamp data_hora_fim
        varchar(20) status_sala
    }

    PRONTUARIO {
        uuid id PK
        uuid paciente_id FK
    }

    ANOTACAO {
        uuid id PK
        uuid consulta_id FK
        uuid medico_id FK
        uuid prontuario_id FK
        text texto
        varchar(300) diagnostico
        text observacoes
        timestamp data_registro
    }

    RECEITA {
        uuid id PK
        uuid consulta_id FK
        uuid medico_id FK
        uuid paciente_id FK
        uuid prontuario_id FK
        timestamp data_emissao
        date validade
        boolean assinada
    }

    ITEM_RECEITA {
        uuid id PK
        uuid receita_id FK
        varchar(200) medicamento
        varchar(100) dosagem
        text posologia
        int quantidade
        text observacoes
    }

    %% Relacionamentos
    USUARIO ||--o| PACIENTE : "é um"
    USUARIO ||--o| MEDICO : "é um"
    USUARIO ||--o| ADMINISTRADOR : "é um"

    CLINICA ||--|{ MEDICO : "emprega"
    CLINICA ||--|| ADMINISTRADOR : "gerenciada por"

    MEDICO ||--|| AGENDA : "possui"
    AGENDA ||--|{ SLOT_DISPONIVEL : "composta por"

    PACIENTE ||--|{ CONSULTA : "realiza"
    MEDICO ||--|{ CONSULTA : "atende"
    SLOT_DISPONIVEL ||--o| CONSULTA : "reservado em"

    CONSULTA ||--o| TELECONSULTA : "possui"
    CONSULTA ||--o| ANOTACAO : "gera"
    CONSULTA ||--o| RECEITA : "origina"

    PACIENTE ||--|| PRONTUARIO : "tem"
    PRONTUARIO ||--|{ ANOTACAO : "contém"
    PRONTUARIO ||--|{ RECEITA : "contém"

    RECEITA ||--|{ ITEM_RECEITA : "composta por"
```

## 2.2 Decisões e Divergências em Relação ao Diagrama de Classes

### Herança → Tabela por subclasse

No diagrama de classes, `Paciente`, `Medico` e `Administrador` herdam de `Usuario`. No MER, optamos pela estratégia de **tabela por subclasse**: uma tabela `USUARIO` contém os atributos comuns (autenticação, nome, email) e cada subclasse tem sua própria tabela com chave estrangeira apontando para `USUARIO`.

Essa escolha foi feita porque cada subclasse tem atributos específicos relevantes (CRM para médico, CPF para paciente) e as consultas por tipo de ator são frequentes e precisam ser eficientes. A alternativa (tabela única com discriminador) geraria muitas colunas nulas.

### Atributos calculados — não persistidos

O método `tempoRestanteParaCancelamento()` na classe `Consulta` é derivado de `data_realizacao` menos o instante atual. Não é persistido no banco — é calculado em tempo de execução. Da mesma forma, a duração da teleconsulta é derivada de `data_hora_inicio` e `data_hora_fim` da `TELECONSULTA`.

### `PRONTUARIO` como tabela explícita

No diagrama de classes, `Prontuario` é um agregador lógico. No MER, materializamos como tabela pois serve como raiz de acesso para controle de permissão — toda leitura de `ANOTACAO` ou `RECEITA` pode verificar se o `prontuario_id` pertence ao paciente do médico em atendimento, centralizando a lógica de autorização.

### Relacionamento N:N — não ocorre neste modelo

Não há relacionamentos muitos-para-muitos neste recorte, pois um `SLOT_DISPONIVEL` pertence a apenas uma `CONSULTA` e uma `Consulta` gera no máximo uma `Receita`. Portanto, não foi necessária tabela associativa.
