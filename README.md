# HealthLink – Plataforma de Telemedicina

## Sobre o Projeto

O **HealthLink** é uma plataforma de telemedicina desenvolvida com o objetivo de conectar pacientes, médicos e clínicas em um ambiente digital seguro e eficiente. A solução permite o agendamento de consultas, realização de teleconsultas, gerenciamento de agendas médicas, emissão de receitas e acompanhamento do histórico clínico dos pacientes.

Este repositório reúne os artefatos produzidos ao longo da disciplina de Engenharia de Software/Modelagem, organizados em três etapas evolutivas do desenvolvimento do sistema.

---

## Objetivo do Sistema

O HealthLink busca facilitar o acesso a serviços de saúde por meio da tecnologia, permitindo que pacientes encontrem profissionais, realizem consultas remotas e acompanhem seu histórico médico, enquanto médicos e administradores possuem ferramentas para gestão de atendimentos e clínicas.

---

# Estrutura dos Trabalhos

## Trabalho 1 – Visão Geral do Produto e Wireframes

Nesta etapa foi realizada a concepção inicial do sistema, definindo sua proposta de valor, público-alvo e funcionalidades principais.

### Artefatos produzidos

* Documento de Visão Geral do Produto
* Identificação dos atores do sistema
* Definição dos objetivos e escopo do projeto
* Levantamento inicial das funcionalidades
* Wireframes das principais telas
* Fluxos básicos de navegação

### Objetivo

Estabelecer uma compreensão clara do problema a ser resolvido e criar uma representação visual inicial da interface do sistema.

---

## Trabalho 2 – Documento de Requisitos e Casos de Uso

Nesta etapa foram especificados os requisitos do sistema utilizando técnicas formais de Engenharia de Requisitos.

### Artefatos produzidos

* Documento de Requisitos
* Requisitos Funcionais classificados utilizando MoSCoW
* Requisitos Não Funcionais
* Identificação dos atores
* Diagrama de Casos de Uso UML
* Descrição das funcionalidades do sistema

### Objetivo

Definir formalmente o comportamento esperado do sistema e estabelecer uma base sólida para as etapas de análise e modelagem.

---

## Trabalho 3 – Modelagem do Sistema

Nesta etapa foram produzidos os principais modelos estruturais e comportamentais do sistema, com foco em três funcionalidades consideradas críticas para o negócio.

### Fatias Verticais Modeladas

1. Paciente busca médico e agenda consulta
2. Ciclo de vida da consulta
3. Médico realiza teleconsulta, registra atendimento e emite receita

### Artefatos produzidos

* Seleção e justificativa das fatias verticais
* Diagrama de Classes UML (representado em Mermaid)
* Modelo Entidade-Relacionamento (MER)
* Diagramas comportamentais

  * Diagrama de Sequência
  * Diagrama de Estados
  * Diagrama de Atividades (quando aplicável)
* Casos de Teste
* Matriz de Rastreabilidade

### Objetivo

Transformar os requisitos definidos anteriormente em modelos capazes de representar a estrutura, os dados e o comportamento do sistema, servindo como base para futuras implementações.

---

# Organização do Repositório

```text
.
├── trabalho-1/
│   ├── visao-geral-produto.md
│   └── wireframes/
│
├── trabalho-2/
│   ├── documento-requisitos.md
│   └── diagrama_uml
│
├── trabalho-3/
│   ├── 00-selecao-de-escopo.md
│   ├── 01-diagrama-de-classes.md
│   ├── 02-mer.md
│   ├── 03-comportamental-fatia1.md
│   ├── 03-comportamental-fatia2.md
│   ├── 03-comportamental-fatia3.md
│   ├── 04-casos-de-teste.md
│   └── 05-rastreabilidade.md
```

---

# Tecnologias e Notações Utilizadas

* UML (Unified Modeling Language)
* Diagramas Mermaid
* Modelo Entidade-Relacionamento (MER)
* Markdown
* GitHub

---

# Equipe

Projeto acadêmico desenvolvido para a disciplina de Engenharia de Software, tendo como estudo de caso a plataforma de telemedicina **HealthLink**.
Alunos: Bianca Lima Rodrigues e Bruno José Oliveira Gomes

---

# Evolução do Projeto

| Trabalho   | Foco Principal           | Resultado                                            |
| ---------- | ------------------------ | ---------------------------------------------------- |
| Trabalho 1 | Visão do produto         | Entendimento do problema e prototipação inicial      |
| Trabalho 2 | Engenharia de requisitos | Definição formal das funcionalidades                 |
| Trabalho 3 | Modelagem do sistema     | Representação estrutural e comportamental da solução |
