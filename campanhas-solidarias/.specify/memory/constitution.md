<!--
Sync Impact Report
Version change: uninitialized scaffold -> 1.0.0
Modified principles: none; initial principles established below
Added sections: Product and Privacy Constraints; Development Workflow
Removed sections: none
Follow-up TODOs: none
-->
# Plataforma de Campanhas Solidárias Constitution

## Core Principles

### I. Simplicidade e Escopo do MVP
A primeira versão MUST conter apenas o menor conjunto de funcionalidades que permita
cadastrar uma campanha solidária e que o público a encontre perto de si, do início ao fim.
Cada funcionalidade MUST estar prevista na especificação aprovada; funcionalidades adicionais
MUST NOT ser implementadas. Isso mantém o produto focado no fluxo essencial.

### II. Critérios de Aceitação Antes do Código
Antes da implementação, toda funcionalidade MUST ter critérios de aceitação verificáveis,
registrados na especificação. A implementação MUST demonstrar conformidade com esses critérios;
requisitos sem critérios verificáveis MUST ser esclarecidos antes de virar código.

### III. Regras de Negócio Testadas
Regras de negócio MUST ter testes automatizados. Alterações nessas regras MUST incluir ou atualizar
testes que verifiquem seus resultados esperados e casos inválidos relevantes.

### IV. Acessibilidade e Idioma
A interface MUST estar em português do Brasil, adaptar-se a dispositivos móveis e desktops e
permitir operação por teclado. Critérios de aceitação MUST tornar essas condições verificáveis
para os fluxos implementados.

### V. Privacidade por Padrão
O repositório MUST conter somente dados pessoais fictícios. A localização do usuário MUST ser
acessada somente após permissão explícita e MUST NOT ser armazenada. A recusa ou ausência de
permissão MUST NOT resultar em coleta de localização.

## Restrições de Produto e Privacidade
A plataforma contempla campanhas solidárias, incluindo doação de roupas, alimentos e adoção de
animais. Esse domínio não autoriza, por si só, a inclusão de funcionalidades adicionais: cada
fluxo precisa constar na especificação aprovada. Dados usados em exemplos, demonstrações e testes
devem ser fictícios. Localização aproximada para descoberta de campanhas só pode ser obtida com
consentimento explícito e não pode ser persistida.

## Fluxo de Desenvolvimento
Toda proposta de funcionalidade MUST definir critérios de aceitação verificáveis antes da
implementação. Revisões MUST confirmar que a mudança corresponde à especificação e não amplia o
escopo. Testes automatizados MUST cobrir as regras de negócio afetadas. A validação da interface
deve cobrir português do Brasil, comportamento responsivo e navegação por teclado nos fluxos
alterados.

## Governance
Esta constituição orienta a especificação, implementação e revisão do produto. Toda proposta de
alteração MUST registrar a justificativa e atualizar esta constituição quando mudar um princípio
ou restrição. A revisão de cada mudança MUST verificar conformidade com estes princípios e com
os critérios de aceitação da funcionalidade.

O versionamento segue SemVer: MAJOR para remoção ou redefinição incompatível de princípios,
MINOR para inclusão de princípio ou expansão material de governança e PATCH para esclarecimentos
sem mudança semântica. A aprovação da alteração deve ocorrer antes da adoção da nova versão.

**Version**: 1.0.0 | **Ratified**: 2026-10-10 | **Last Amended**: 2026-10-10
