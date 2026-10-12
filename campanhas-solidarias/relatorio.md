# Relatório – Spec-Driven Development

- **Disciplina:** Tópicos especiais em Desenvolvimento de Software
- **Professor:** Francisco Carlos Souza
- **Alunos:** Camila Bregalda e Emanuel Hennig

## 1. Produto escolhido e escopo do MVP

Plataforma web para divulgar campanhas solidárias e ajudar apoiadores a encontrar
ações próximas. 

### Usuários
 
- **Apoiador:** consulta campanhas sem precisar de cadastro e busca ações perto de si.
- **Organizador:** cadastra uma campanha e administra seu conteúdo usando um token
  específico daquela campanha, gerado no momento do cadastro.
  
### Funcionalidades do MVP
 
1. **Cadastro e publicação de campanhas** com categoria, título, descrição,
   instituição, local e período. Categoria, local e datas são obrigatórios, e a data
   final não pode anteceder a inicial.
2. **Busca pública** por cidade ou CEP informado manualmente, com filtros por
   categoria e instituição. Exibe todas as campanhas ainda não encerradas, da mais
   próxima à mais distante, sem raio máximo.
3. **Página pública da campanha**, com suas necessidades em aberto e atualizações de
   andamento.
4. **Administração de necessidades e atualizações** pelo token gerado no cadastro;
   uma necessidade atendida deixa de aparecer como aberta.

### Regras de negócio que sustentam os critérios de aceitação
 
- Uma campanha só é publicada com categoria, local e período válidos.
- Campanhas encerradas não aparecem na busca.
- O token de uma campanha só autoriza ações naquela campanha.
- Necessidade marcada como atendida sai da lista de necessidades em aberto.
  
### Fora do escopo
 
Aplicativo móvel nativo, pagamentos ou doações financeiras, chat, comentários do
público, notificações push, moderação automática e avaliações de campanhas. Declarar
esses itens evita que o agente acrescente funcionalidades não pedidas.

### Requisitos transversais
 
Interface em português do Brasil, responsiva e operável por teclado; testes
automatizados das regras de negócio; dados pessoais apenas fictícios no repositório.

### Processo de criação
 
O trabalho seguiu o fluxo do Spec Kit, e cada etapa gerou um artefato:
 
| Etapa | Artefato | O que foi feito |
|---|---|---|
| `/speckit.constitution` | `constitution.md` | Registrou os princípios que limitam todas as etapas seguintes. |
| `/speckit.specify` | `spec.md` | Descreveu problema, usuários, histórias, requisitos, critérios de sucesso e itens fora do escopo, sem citar tecnologia. |
| Revisão manual | `spec.md` | Conferi personas, regras de publicação, estado das necessidades e escopo excluído (detalhes na seção 3). |
| `/speckit.clarify` | `spec.md` atualizado | Resolveu ambiguidades de autorização e de busca. |
| `/speckit.plan` | `plan.md`, modelo de dados, contratos | Definiu Node.js/Express, SQLite, React/Vite, modelo de dados e contrato HTTP. |
| `/speckit.tasks` | `tasks.md` | Decompôs o trabalho em tarefas por história, com os testes antes das implementações. |
 
**Constituição.** Ela prioriza a simplicidade do MVP, exige critérios verificáveis
antes do código, testes automatizados das regras de negócio, interface acessível em
pt-BR e privacidade. Também proíbe dados pessoais reais no repositório e determina
que a localização do apoiador não seja armazenada. Na prática, esses princípios
funcionam como filtro: qualquer requisito ou tarefa que gere dado pessoal sem
necessidade, não tenha critério verificável ou fuja do MVP contraria a constituição.

## 2. Agente de IA e versão do Spec Kit
 
- **Spec Kit:** 1.1.3
- **Modelo:** GPT-6 Luna
- **Agente/integração e ambiente:** GitHub Copilot no VS Code, modo Agent

## 3. O que o /speckit.clarify perguntou e o que mudou

### Revisão manual antes do clarify
 
Entre o `/speckit.specify` e o `/speckit.clarify`, o arquivo `spec.md` criado foi revisado
e verificado as personas, as regras de publicação, o estado das necessidades e o escopo excluído.
Essa leitura apontou duas lacunas importantes:
 
1. **Autorização:** a especificação não dizia como restringir as ações
   administrativas, ou seja, quem poderia editar uma campanha. Na ideia inicial, todos
   os acessos a plataforma deveriam ser feitos por meio do login em uma conta, o que não
   era necessário para usuários que apenas estavam navegando atrás de informações sobre campanhas.
3. **Definição de "próximas":** a busca falava em campanhas próximas sem dizer qual
   seria o raio ou o critério.

A especificação inicial também oferecia localização do dispositivo com permissão do
usuário, com busca manual como alternativa. Essa escolha foi revista posterirmente.

| Pergunta do agente | Minha resposta | Mudança no spec.md |
|---|---|---|
| Como o organizador comprova que é o administrador autorizado da campanha? | Aceitei a opção A sugerida: conta de administrador com e-mail e senha; apoiadores anônimos. | A primeira revisão registrou conta para administrador e consulta pública sem cadastro. Essa decisão foi substituída depois, conforme a decisão mais recente abaixo. |
| Como a busca define quais campanhas são próximas o bastante para exibir? | Opção A: mostrar todas as campanhas não encerradas, ordenadas da mais próxima à mais distante. | A busca passou a declarar que não tem raio máximo; o requisito FR-008 e os cenários de busca foram atualizados. |

As duas perguntas coincidem com as lacunas que eu havia identificado na revisão
manual, o que indica que o `clarify` cobriu os pontos de maior impacto.

**Decisões posteriores no `/speckit.plan`**: Duas decisões mudaram durante o planejamento e fizeram a especificação ser revisada.
A instrução mais recente prevaleceu sobre a resposta dada no `clarify`:
 
| Tema | Antes (clarify) | Depois (plan) |
|---|---|---|
| Autenticação | Conta de administrador e usuário com e-mail e senha para navegar na plataforma | Token de gerenciamento gerado no cadastro e vinculado a uma única campanha; apoiadores seguem anônimos |
| Localização | Localização do dispositivo (com permissão) ou busca manual | Somente cidade ou CEP digitados manualmente, sem pedir permissão de geolocalização |
 
Essas mudanças foram incorporadas ao FR-005, FR-006 e FR-016, às histórias, às
entidades e às premissas. A entrada de busca não é persistida pela aplicação.

### Como a busca funciona e a questão da privacidade
 
Para ordenar os resultados, o plano usa coordenadas municipais aproximadas. Assim,
"mais próxima" significa mais próxima em nível de município, sem precisar de
localização exata. Uma busca por CEP consulta o ViaCEP somente após o envio explícito
do formulário. O serviço externo recebe o CEP, embora a aplicação não o armazene; a
retenção do provedor não é controlada pelo projeto e deve ser considerada antes da
implantação.

## 4. O que eu mudaria se refizesse
 
- **Declararia a busca como exclusivamente manual desde a primeira especificação**,
  sem localização do dispositivo, e explicaria como a distância municipal aproxima a
  ideia de "perto". Esse ponto não havia sido considerado inicialmente e exigiu uma
  grande revisão depois da criação das histórias.
- **Definiria o token de gerenciamento desde o início**, incluindo sua exibição no
  cadastro, o armazenamento apenas do hash e a consequência de não haver recuperação
  caso o organizador perca o token. Começamos o projeto sem decidir a forma de
  autenticação, o que causou grandes mudanças ao longo do trabalho; decidir isso
  cedo teria sido muito mais proveitoso.
- **Avaliaria e documentaria antes a consulta externa de CEP:** o ViaCEP recebe o
  valor digitado, e sua retenção está fora do controle da aplicação.
- **Tornaria os cenários de erro e privacidade explícitos na primeira versão**,
  incluindo CEP inválido, município ambíguo, serviço de CEP indisponível, ausência de
  persistência da busca e token de uma campanha usado em outra.
- **Manteria o raio sem limite como decisão deliberada do MVP.** Se o público ou o
  volume crescer, revisaria essa escolha com dados de uso e critérios mensuráveis, em
  vez de introduzir um limite arbitrário agora.
### Lição geral
 
As maiores revisões vieram de decisões que cruzam várias histórias (quem
autentica e como se localiza o apoiador). Definir essas decisões antes de escrever as
histórias evita retrabalho, porque elas afetam requisitos, entidades e premissas ao
mesmo tempo.
