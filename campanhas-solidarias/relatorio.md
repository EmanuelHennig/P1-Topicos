# Relatório – Spec-Driven Development

## 1. Produto escolhido e escopo do MVP

Plataforma web para divulgar campanhas solidárias e ajudar apoiadores a encontrar
ações próximas. Os usuários são o apoiador, que consulta campanhas sem cadastro,
e o organizador, que cadastra uma campanha e administra seu conteúdo com um token
específico daquela campanha.

As quatro funcionalidades do MVP são:

1. Cadastro e publicação de campanhas com categoria, título, descrição, instituição,
	local e período. Categoria, local e datas são obrigatórios; a data final não pode
	anteceder a inicial.
2. Busca pública por cidade ou CEP informado manualmente, com filtros por categoria
	e instituição. Exibe todas as campanhas ainda não encerradas, da mais próxima à
	mais distante, sem raio máximo.
3. Página pública da campanha com suas necessidades em aberto e atualizações de
	andamento.
4. Administração das necessidades e atualizações pelo token gerado no cadastro da
	campanha; uma necessidade atendida deixa de aparecer como aberta.

Ficam fora do escopo: aplicativo móvel nativo, pagamentos ou doações financeiras,
chat, comentários do público, notificações push, moderação automática e avaliações
de campanhas. O projeto também exige interface em português do Brasil, responsiva,
operável por teclado, testes automatizados das regras de negócio e dados pessoais
fictícios no repositório.

## Processo de criação

O trabalho seguiu o fluxo do Spec Kit: primeiro foi registrada a constituição do
projeto; em seguida, a especificação descreveu problema, usuários, histórias,
requisitos, critérios de sucesso e itens fora do escopo. O `/speckit.clarify`
resolveu ambiguidades que afetavam autorização e busca. Depois, o `/speckit.plan`
definiu Node.js/Express, SQLite, React/Vite, modelo de dados e contrato HTTP; por
fim, o `/speckit.tasks` decompôs o trabalho em tarefas por história, incluindo testes
antes das implementações.

A constituição prioriza simplicidade do MVP, critérios verificáveis antes do código,
testes automatizados das regras de negócio, interface acessível em pt-BR e privacidade.
Ela também proíbe dados pessoais reais no repositório e determina que a localização
do apoiador não seja armazenada.

## 2. Agente de IA e versão do Spec Kit
Speckit 1.1.3
Modelo: GPT-6 Luna

## 3. O que o /speckit.clarify perguntou e o que mudou
| Pergunta do agente | Minha resposta | Mudança no spec.md |
|---|---|---|
| Como o organizador comprova que é o administrador autorizado da campanha? | Aceitei a opção A sugerida: conta de administrador com e-mail e senha; apoiadores anônimos. | A primeira revisão registrou conta para administrador e consulta pública sem cadastro. Essa decisão foi substituída depois, conforme a decisão mais recente abaixo. |
| Como a busca define quais campanhas são próximas o bastante para exibir? | Opção A: mostrar todas as campanhas não encerradas, ordenadas da mais próxima à mais distante. | A busca passou a declarar que não tem raio máximo; o requisito FR-008 e os cenários de busca foram atualizados. |

Antes do `/speckit.clarify`, a revisão manual já havia conferido as personas, as
regras de publicação, o estado de necessidades e o escopo excluído. Ela identificou
duas lacunas importantes: como restringir ações administrativas e como definir
“próximas” sem um raio especificado. A especificação inicial também oferecia
localização do dispositivo com permissão e busca manual como alternativa.

**Decisões posteriores no `/speckit.plan`**: a instrução mais recente substituiu
e-mail/senha por um token de gerenciamento gerado no cadastro e vinculado a uma
campanha. Apoiadores continuam anônimos. Também foi removida a localização do
dispositivo: a busca passou a aceitar somente cidade ou CEP digitados manualmente,
sem solicitar permissão de geolocalização. Essas decisões foram incorporadas ao
FR-005, FR-006 e FR-016, às histórias, às entidades e às premissas. A entrada de
busca não é persistida pela aplicação.

Para ordenar resultados, o plano usa coordenadas municipais aproximadas; uma busca
por CEP consulta o ViaCEP após o envio explícito do formulário. O serviço externo
recebe o CEP, embora a aplicação não o armazene; a retenção do provedor não é
controlada pelo projeto e deve ser considerada antes da implantação.

## 4. O que eu mudaria se refizesse

- Declararia desde a primeira especificação que a busca é exclusivamente manual,
	sem localização do dispositivo, e explicaria como a distância municipal aproxima
	a ideia de “perto”.
- Definiria o token de gerenciamento desde o início, incluindo sua exibição no
	cadastro, armazenamento apenas do hash e a consequência de não haver recuperação
	se o organizador perder o token.
- Avaliaria e documentaria antes a consulta externa de CEP: o ViaCEP recebe o valor
	digitado, e sua retenção está fora do controle da aplicação.
- Tornaria os cenários de erro e privacidade ainda mais explícitos na primeira
	versão, incluindo CEP inválido, município ambíguo, serviço de CEP indisponível,
	ausência de persistência da busca e token de uma campanha usado em outra.
- Manteria o raio sem limite como decisão deliberada do MVP; se o público ou volume
	crescer, revisaria essa escolha com dados de uso e critérios mensuráveis, em vez
	de introduzir um limite arbitrário agora.