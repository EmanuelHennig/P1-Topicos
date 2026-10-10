# Feature Specification: Divulgação e descoberta de campanhas solidárias

**Feature Branch**: `Não criada (nenhum hook de branch configurado)`

**Created**: 2026-10-10

**Status**: Draft

**Input**: User description: Plataforma web para divulgação e descoberta de campanhas solidárias, conforme escopo e regras fornecidos.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar e publicar campanha (Priority: P1)

O organizador informa os dados de uma campanha solidária e a publica para que apoiadores possam encontrá-la. Os dados incluem categoria, título, descrição, instituição responsável, local e período de realização.

**Why this priority**: Sem campanhas publicadas, não há conteúdo para apoiar ou descobrir; este é o ponto inicial do MVP.

**Independent Test**: Um administrador cadastra uma campanha válida e confirma que ela passa a aparecer na descoberta pública; tentativas sem categoria, local ou período não a publicam.

**Acceptance Scenarios**:

1. **Given** que o administrador preenche categoria, título, descrição, instituição, local e datas válidas, **When** publica a campanha, **Then** a campanha fica disponível para consulta pública.
2. **Given** que falta categoria, local, data de início ou data de término, **When** o administrador tenta publicar, **Then** a publicação é recusada e os campos necessários são indicados.
3. **Given** que a data de término é anterior à data de início, **When** o administrador tenta publicar, **Then** a publicação é recusada e o período precisa ser corrigido.

---

### User Story 2 - Encontrar campanhas próximas (Priority: P1)

O apoiador informa onde está, localiza campanhas próximas e refina os resultados por categoria ou nome da instituição para decidir qual visitar ou apoiar.

**Why this priority**: Resolve diretamente a dificuldade de apoiadores em saber quais campanhas estão acontecendo perto deles.

**Independent Test**: Com campanhas fictícias em locais e categorias distintos, o apoiador busca pela localização atual ou por cidade/CEP e verifica que os filtros retornam somente campanhas correspondentes e não encerradas.

**Acceptance Scenarios**:

1. **Given** que o apoiador concede permissão explícita para usar sua localização, **When** consulta campanhas próximas, **Then** são exibidas campanhas publicadas ordenadas pela proximidade, sem armazenar a localização do apoiador.
2. **Given** que o apoiador nega ou não concede permissão de localização, **When** escolhe busca manual, **Then** pode informar cidade ou CEP e consultar campanhas próximas desse local.
3. **Given** que há campanhas de várias categorias e instituições, **When** o apoiador filtra por categoria, por nome da instituição ou por ambos, **Then** cada resultado corresponde a todos os filtros aplicados.
4. **Given** que uma campanha tem data de término anterior à data atual, **When** o apoiador consulta campanhas próximas, **Then** essa campanha não aparece nos resultados.

---

### User Story 3 - Consultar necessidades e atualizações (Priority: P2)

O apoiador abre a página de uma campanha para consultar o que ela precisa e acompanhar postagens informativas sobre seu andamento.

**Why this priority**: Ajuda o público a entender como participar e mantém as informações da campanha acessíveis sem criar canais fora do escopo.

**Independent Test**: O apoiador abre uma campanha publicada e confirma que consegue consultar necessidades em aberto e atualizações publicadas.

**Acceptance Scenarios**:

1. **Given** uma campanha publicada com necessidades e atualizações, **When** o apoiador abre sua página, **Then** consulta a descrição, o local, o período, as necessidades em aberto e as atualizações.
2. **Given** uma necessidade marcada como atendida, **When** o apoiador consulta a lista de necessidades em aberto, **Then** essa necessidade não aparece como aberta.

---

### User Story 4 - Administrar necessidades e atualizações (Priority: P2)

O administrador responsável pela campanha cadastra necessidades e publica informações de andamento; somente esse administrador pode fazer essas publicações ou marcar uma necessidade como atendida.

**Why this priority**: Permite que organizadores comuniquem necessidades concretas e mantenham o público atualizado, preservando o controle da campanha.

**Independent Test**: Com uma campanha e um administrador identificados, testar publicação e atendimento de necessidades e publicação de atualizações por esse administrador e por outro usuário.

**Acceptance Scenarios**:

1. **Given** que o usuário é administrador da campanha, **When** cadastra uma necessidade, **Then** ela é associada à campanha e fica disponível ao público como aberta.
2. **Given** que o usuário é administrador da campanha, **When** publica uma atualização informativa, **Then** ela aparece na página pública da campanha.
3. **Given** que o usuário não é administrador da campanha, **When** tenta cadastrar ou alterar necessidades, marcar uma necessidade como atendida ou publicar uma atualização, **Then** a ação é recusada e o conteúdo não é alterado.
4. **Given** uma necessidade em aberto, **When** o administrador a marca como atendida, **Then** ela deixa de aparecer entre as necessidades em aberto.

### Edge Cases

- Uma tentativa de publicação com um ou mais campos obrigatórios ausentes deve indicar os campos pendentes e não tornar a campanha pública.
- Uma data de término anterior à data de início deve ser rejeitada; campanhas com término na data atual ainda podem aparecer na busca.
- A busca manual deve aceitar cidade ou CEP; entrada vazia ou não reconhecida deve permitir correção sem recorrer à localização do dispositivo.
- Negar permissão de localização não deve impedir o uso da busca manual nem causar armazenamento de coordenadas.
- Filtros combinados não devem incluir campanha que falhe em qualquer filtro aplicado.
- Necessidade atendida não deve reaparecer como aberta; tentativas de alteração por não administradores devem ser recusadas.
- Campanhas sem resultados correspondentes devem apresentar um estado vazio informativo, sem incluir campanhas encerradas.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: A plataforma MUST permitir que o administrador cadastre uma campanha com categoria, título, descrição, nome da instituição, local, data de início e data de término.
- **FR-002**: A categoria MUST ser uma das opções: doação de roupas, doação de alimentos, adoção de animais ou outras.
- **FR-003**: A plataforma MUST impedir a publicação de campanhas sem categoria, local, data de início ou data de término e MUST rejeitar data de término anterior à data de início.
- **FR-004**: A plataforma MUST disponibilizar para consulta pública as campanhas publicadas e permitir a abertura de uma página com seus dados cadastrados.
- **FR-005**: A plataforma MUST permitir busca de campanhas próximas à localização atual somente após permissão explícita do apoiador; MUST NOT armazenar a localização do apoiador.
- **FR-006**: Se o apoiador não conceder permissão de localização, a plataforma MUST permitir informar manualmente uma cidade ou CEP para buscar campanhas próximas desse local e MUST NOT armazenar o local informado.
- **FR-007**: A busca MUST permitir filtrar por categoria e por nome da instituição, individualmente ou em conjunto, e MUST exibir apenas campanhas que correspondam aos filtros.
- **FR-008**: A busca de próximas MUST excluir campanhas com data de término anterior à data atual e MUST ordenar os resultados por proximidade do local de busca.
- **FR-009**: A plataforma MUST permitir que o administrador da campanha cadastre necessidades com a descrição do item e a quantidade/unidade quando aplicável; necessidades novas ficam abertas para consulta pública.
- **FR-010**: A plataforma MUST permitir que o administrador marque uma necessidade aberta como atendida; necessidades atendidas MUST deixar de aparecer na lista de necessidades abertas.
- **FR-011**: A plataforma MUST permitir que o administrador publique atualizações informativas associadas à sua campanha e disponibilizá-las na página pública da campanha.
- **FR-012**: Somente o administrador responsável pela campanha MUST poder cadastrar ou alterar necessidades, marcar necessidades como atendidas e publicar atualizações; ações de outros usuários MUST ser recusadas.
- **FR-013**: A interface MUST estar em português do Brasil, adaptar-se a telas móveis e desktop e permitir navegação e operação por teclado.
- **FR-014**: Dados pessoais usados no repositório, em exemplos ou em testes MUST ser fictícios; dados reais de apoiadores não fazem parte do escopo.
- **FR-015**: A plataforma MUST NOT incluir aplicativo móvel nativo, pagamentos ou doações financeiras, chat, comentários do público, notificações push, moderação automática de conteúdo ou avaliações de campanhas.

### Key Entities *(include if feature involves data)*

- **Campanha**: ação solidária com categoria, título, descrição, instituição responsável, local, período e administrador responsável; pode ser publicada para consulta pública.
- **Administrador da campanha**: usuário autorizado a administrar uma campanha específica, publicar suas necessidades e atualizações e registrar o atendimento de necessidades.
- **Necessidade**: item ou recurso solicitado por uma campanha, com descrição e quantidade/unidade quando aplicável, e estado aberta ou atendida.
- **Atualização**: postagem informativa de andamento, publicada pelo administrador e associada a uma campanha.
- **Local de busca**: localização atual consentida ou cidade/CEP informado manualmente, usada somente para consultar e ordenar resultados; não é armazenada, independentemente de sua origem.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Em teste de usabilidade com dados prontos, pelo menos 9 de 10 apoiadores encontram uma campanha compatível com sua localização e filtros sem ajuda de um moderador.
- **SC-002**: Em testes de cadastro, 100% das campanhas com dados obrigatórios completos e período válido podem ser publicadas; 100% das tentativas sem dados obrigatórios ou com período invertido são recusadas.
- **SC-003**: Em testes de busca, 100% das campanhas encerradas são excluídas e todos os resultados exibidos correspondem à localização e aos filtros informados.
- **SC-004**: Em testes de autorização, 100% das tentativas de não administradores para publicar ou alterar necessidades e atualizações são recusadas.
- **SC-005**: Em testes de necessidades, 100% das necessidades marcadas como atendidas deixam de constar na lista de abertas.
- **SC-006**: Em avaliação dos fluxos públicos e administrativos, todas as tarefas especificadas podem ser concluídas por teclado e a interface permanece utilizável em telas móveis e desktop.

## Assumptions

- A busca apresenta campanhas publicadas cujo término seja hoje ou futuro, incluindo campanhas ainda não iniciadas; campanhas encerradas são excluídas.
- “Próximas” significa ordenar os resultados pela distância em relação ao local de busca; o MVP não impõe um raio máximo não especificado.
- Cidade e CEP são formas válidas de localização manual; a correspondência de CEP ou cidade será resolvida pela plataforma sem solicitar localização do dispositivo.
- Cada campanha tem um administrador identificável e a plataforma consegue verificar sua autorização; o método de criação de contas e autenticação não foi definido neste escopo.
- A categoria “outras” pode ser complementada pela descrição da campanha, sem criar categorias adicionais no MVP.
- Quantidade e unidade são informadas quando fizer sentido para a necessidade; necessidades que não dependem de quantidade podem ser descritas sem esses campos.
- Usuários de demonstração e exemplos de campanha usam dados fictícios; nenhuma informação pessoal real será incluída no repositório.
- Aplicativo móvel nativo, pagamentos/doações financeiras, chat, comentários públicos, notificações push, moderação automática e avaliações estão fora do MVP.
