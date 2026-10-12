# Implementation Plan: Divulgação e descoberta de campanhas solidárias

**Branch**: `001-descoberta-campanhas` | **Date**: 2026-10-10 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-descoberta-campanhas/spec.md` and stack choices supplied with `/speckit-plan`.

## Summary

Construir uma aplicação web em React/Vite e Node.js/Express para cadastrar campanhas,
permitir busca pública por cidade ou CEP e exibir necessidades e atualizações. O SQLite
armazena campanhas e seus conteúdos; cada campanha recebe um token aleatório de gerenciamento,
cujo hash autoriza operações administrativas apenas naquele recurso. A busca é manual, não
usa geolocalização do dispositivo e não persiste a localização digitada.

## Technical Context

**Language/Version**: JavaScript ESM; Node.js 24 LTS

**Primary Dependencies**: Express 5, React 19, Vite (versão estável compatível com Node 24),
better-sqlite3, Vitest, Supertest e Testing Library para React

**Storage**: SQLite em arquivo local, com migrations SQL e `better-sqlite3`; catálogo geográfico
do Brasil derivado do GeoNames e importado localmente

**Testing**: Vitest para regras e integração; Supertest para HTTP; Testing Library para
interações de interface. Testes de negócio não dependem de rede e usam um adaptador ViaCEP falso.

**Target Platform**: Navegadores desktop e móveis; servidor Node.js em uma única máquina com
acesso ao arquivo SQLite local

**Project Type**: Aplicação web com backend e frontend separados no mesmo repositório

**Performance Goals**: Sem metas de throughput ou latência definidas para o MVP; a busca ordena
todas as campanhas não encerradas, conforme a especificação

**Constraints**: Não solicitar geolocalização do dispositivo; não persistir nem registrar o local
digitado pelo apoiador; dados de exemplo exclusivamente fictícios; o token bruto não é gravado,
exposto em URL ou incluído em logs; SQLite e WAL ficam na máquina do processo, não em filesystem
de rede

**Scale/Scope**: MVP de uma instância, quatro jornadas especificadas, catálogo geográfico brasileiro
e volume inicial pequeno; limites de carga e disponibilidade não foram fornecidos

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design.*

| Gate | Status | Evidence in this plan |
|------|--------|-----------------------|
| Simplicidade e escopo do MVP | PASS | Um backend, um frontend, SQLite; sem pagamentos, chat, comentários ou app nativo. |
| Critérios verificáveis antes do código | PASS | Especificação contém cenários de aceitação e requisitos funcionais verificáveis. |
| Testes automatizados de regras | PASS | Vitest cobre regras; integração HTTP verifica autenticação por token e persistência. |
| pt-BR, responsividade e teclado | PASS | Frontend e roteiro de validação incluem idioma, breakpoints e operação por teclado. |
| Privacidade por padrão | PASS | Busca manual; entrada não persistida nem registrada; CEP enviado a ViaCEP somente após ação explícita e sem cópia local. |
| Dados pessoais fictícios | PASS | Fixtures usam campanhas, organizações e credenciais inteiramente fictícias. |

## Project Structure

### Documentation (this feature)

```text
specs/001-descoberta-campanhas/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
│   └── openapi.yaml
└── quickstart.md
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── app.js
│   ├── server.js
│   ├── routes/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   └── db/
│       └── migrations/
└── tests/
    ├── unit/
    └── integration/

frontend/
├── src/
│   ├── api/
│   ├── components/
│   ├── pages/
│   └── styles/
└── tests/
```

**Structure Decision**: Dois projetos simples, `backend/` e `frontend/`, com instalação e
comandos independentes; sem monorepo de pacotes, camada compartilhada ou separação em serviços.
O backend concentra validação, regras, autorização por token, geocodificação e acesso SQLite.
O frontend consome o contrato HTTP e nunca decide autorização por conta própria.

## Design Decisions

- O cadastro válido publica a campanha imediatamente; não há rascunho, aprovação ou edição de
  campanha nesta entrega, pois esses fluxos não foram especificados.
- A localização do evento é pública e persistida como rótulo informado pelo organizador e
  coordenada aproximada do município. O local digitado pelo apoiador é resolvido por requisição,
  usado para ordenar resultados e descartado ao final.
- Cidade é resolvida pelo catálogo local GeoNames. Para CEP, o backend consulta ViaCEP depois do
  envio explícito do formulário, usa apenas município/UF para obter a coordenada municipal e não
  armazena a entrada. O formulário informa essa consulta externa. Testes substituem ViaCEP por um
  adaptador determinístico.
- A ordem é calculada pela distância em linha reta entre coordenadas municipais; não há raio
  máximo. Empates usam data de início e identificador para resultado determinístico.
- O token tem 256 bits aleatórios, pertence a uma campanha e é enviado no cabeçalho Bearer. O
  servidor persiste somente SHA-256 do token, compara hashes em tempo constante e o devolve
  somente na resposta de cadastro. A interface o mostra para cópia, mantém uso em memória e não
  implementa recuperação de token no MVP.
- O SQLite usa foreign keys, consultas parametrizadas e transações curtas. WAL é adequado à
  instância única; escritores continuam serializados. Não usar banco em filesystem de rede.
- Datas de campanha são datas civis `YYYY-MM-DD`, com término inclusivo, comparadas usando
  `America/Sao_Paulo`.

## Constitution Check (Post-Design)

Todos os gates continuam PASS. A separação frontend/backend e a dependência externa de CEP são
necessárias para as tecnologias escolhidas e para o fluxo de busca solicitado; não introduzem
funcionalidades de usuário além da especificação. A requisição a ViaCEP é explicitada no fluxo,
não é registrada nem armazenada pela aplicação, e pode falhar sem impedir buscas por cidade.

## Complexity Tracking

Sem violações da constituição. Não há projeto adicional, autenticação de apoiadores, mecanismo de
pagamento, serviço de geocodificação de dispositivo ou infraestrutura distribuída.
