# Prompt para implementação do backend e scripts de banco de dados — Remarketing

Você é uma IA de engenharia de software responsável por implementar **todo o backend** e **todos os scripts de banco de dados ainda não gerados** para a evolução de Remarketing/Ofertas/Descontos Progressivos.

## Contexto obrigatório
- O frontend já está implementado.
- Não proponha novo frontend.
- Não proponha modelo de tabela por conta própria.
- Não invente nomes de colunas, tabelas, entidades ou contratos se eles estiverem definidos nas referências que serão fornecidas junto com este prompt.
- Você deve usar **exatamente** as referências de tabelas, campos, contratos, padrões e exemplos que serão passados em conjunto com este prompt.
- Caso exista conflito entre este prompt e as referências estruturais fornecidas, **as referências fornecidas prevalecem para nomes e estrutura**, mas **as regras de negócio deste prompt devem ser preservadas**.
- Sua responsabilidade inclui:
  1. Implementar backend completo
  2. Implementar regras de negócio
  3. Implementar validações
  4. Implementar processamento de uploads
  5. Implementar persistência
  6. Implementar consultas para o frontend já existente
  7. Implementar recálculo de progresso e resumo financeiro
  8. Gerar scripts de banco de dados faltantes
  9. Gerar migrations/DDL/DML necessários de acordo com o padrão do projeto
  10. Gerar testes automatizados compatíveis com o projeto, sempre que o padrão existente comportar isso

## Objetivo funcional
Implementar a evolução do módulo de Remarketing para suportar:
- segmentação de liberação de ofertas por categoria de concessionário
- parametrização por uploads independentes
- upload de ofertas com novos campos
- clusterização/filtro por categoria de veículo
- desconto progressivo por categoria
- desconto à vista por oferta
- prioridade/restrição por grupo exclusivo
- barra de progresso e mensagens dinâmicas
- resumo financeiro dinâmico

---

# Regras de negócio obrigatórias

## 1. Segmentação de início de oferta por categoria de concessionário
Devem existir configurações independentes de início de oferta por categoria de concessionário:
- PLATINUM
- GOLD
- SILVER
- STANDARD

Regras obrigatórias:
- cada categoria deve possuir configuração independente de data/hora de início
- encerramento da oferta continua global
- o toggle de encerrar oferta continua global
- a listagem/visibilidade de ofertas deve respeitar a categoria do grupo/DN do usuário logado
- uma oferta só pode estar disponível para um DN quando a janela correspondente à categoria do seu grupo permitir

---

## 2. Uploads independentes de parâmetros
O sistema deve suportar uploads independentes para os conjuntos de parâmetros abaixo:
- tabela de DNs
- tabela de grupo de DNs
- tabela de categorias de grupos de DNs
- tabela de categorias de veículos

### 2.1. Tabela de DNs
As referências fornecidas devem contemplar campos equivalentes a:
- DN
- CNPJ
- usuários com acesso à funcionalidade de ofertas

Regras obrigatórias:
- usar esta carga para determinar identidade/associação do DN
- usar esta carga para determinar permissão de acesso à funcionalidade de ofertas
- usar esta carga para determinar permissão de geração/extração de relatórios

### 2.2. Tabela de grupo de DNs
As referências fornecidas devem contemplar campos equivalentes a:
- nome do grupo
- categoria do grupo
- DNs do grupo
- limite de veículos para compra consignada
- limite em reais para compras +90 e cessão
- opções de faturamento disponíveis para o grupo

Regras obrigatórias:
- usar esta carga para vincular DN a grupo
- usar esta carga para identificar a categoria do grupo
- usar esta carga para validar tipos de faturamento permitidos
- usar esta carga para validar limites financeiros quando aplicável
- usar esta carga para expor ao frontend as opções de CNPJ e faturamento do grupo logado

### 2.3. Tabela de categorias de grupos de DNs
As referências fornecidas devem contemplar campos equivalentes a:
- categoria do grupo
- descrição
- limite de desconto %
- limite de desconto em R$
- % de desconto por veículo das categorias B e C

Mapeamento de negócio conhecido:
- A = PLATINUM
- B = GOLD
- C = SILVER
- D = STANDARD

Regras obrigatórias:
- usar esta carga para determinar teto percentual de desconto progressivo
- usar esta carga para determinar teto monetário de desconto progressivo
- usar esta carga para determinar percentual de desconto por veículo elegível
- mesmo que a lógica ativa consolidada esteja focada em veículo categoria C, o processamento deve suportar os dados disponibilizados nas referências

### 2.4. Tabela de categorias de veículos
As referências fornecidas devem contemplar campos equivalentes a:
- categoria
- descrição

Mapeamento de negócio conhecido:
- A = PRIME
- B = SELECT
- C = ESSENTIAL

Regras obrigatórias:
- usar esta carga para filtro, clusterização, exibição e regras de desconto

---

## 3. Evolução do upload de ofertas
A planilha de upload de ofertas deve passar a contemplar, obrigatoriamente, as novas colunas:
- CATEGORIA DO VEÍCULO
- DESCONTO PAGAMENTO À VISTA
- GRUPO EXCLUSIVO

### 3.1. Regras da coluna CATEGORIA DO VEÍCULO
- coluna obrigatória no layout
- valores válidos: A, B, C
- deve ser usada na segmentação, clusterização, filtro e regras da oferta

### 3.2. Regras da coluna DESCONTO PAGAMENTO À VISTA
- coluna obrigatória no layout
- valor numérico percentual
- deve ser aplicada exclusivamente para compras à vista
- não deve interferir em outros tipos de faturamento
- deve ser exibida no card quando aplicável

### 3.3. Regras da coluna GRUPO EXCLUSIVO
- coluna obrigatória no layout
- o conteúdo da célula pode ser vazio quando não houver exclusividade, conforme regras aceitas no projeto/referências
- quando preenchida, deve conter um grupo válido
- o veículo deve ser ofertado apenas para DNs pertencentes a esse grupo
- esse grupo deve ter prioridade de compra do veículo
- não contemplar múltiplos grupos no mesmo campo

### 3.4. Regras gerais do upload de ofertas
O processamento deve:
- validar presença das novas colunas
- rejeitar arquivos com valores inválidos
- persistir corretamente os novos dados
- aplicar segmentação por categoria do veículo
- aplicar regra de desconto à vista
- aplicar regra de prioridade/restrição por grupo exclusivo

---

## 4. Regras de visualização de veículos
O backend deve atender o frontend já implementado para a tela de visualização/listagem de veículos.

### 4.1. Filtro por categoria
Deve suportar filtro por categoria do veículo:
- PRIME
- SELECT
- ESSENTIAL

O filtro deve seguir o mesmo comportamento dos demais filtros existentes.

### 4.2. Dados dos cards
Cada card deve possuir backend compatível para prover, no mínimo, os dados abaixo de acordo com as referências existentes:
- categoria/flag visual do veículo
- link de cautelar
- modelo
- localização (cidade/UF)
- placa, cor, KM
- valor FIPE
- valor de compra
- mensagem/informação de desconto à vista
- dados de faturamento
- opções de CNPJ do grupo
- tipos de faturamento permitidos
- estado/status de compra

### 4.3. Regras de exibição do desconto à vista no card
- exibir a informação “Pague à vista e economize X%” apenas quando houver desconto à vista aplicável para a oferta
- se o DN/grupo logado não tiver opção de faturamento à vista disponível, não exibir esse informativo
- o valor do desconto à vista deve vir do upload de ofertas

---

## 5. Regra de desbloqueio do desconto progressivo
O desconto progressivo só pode ser desbloqueado quando **as duas condições abaixo** forem atendidas simultaneamente:
1. existir no mínimo 3 veículos no pacote/compra/reserva considerado pelo fluxo
2. existir no mínimo 1 veículo da categoria C (ESSENTIAL) nesse pacote

Regras explícitas que devem ser respeitadas:
- 3 veículos sem nenhum categoria C -> não desbloqueia
- 2 veículos, mesmo que ambos categoria C -> não desbloqueia
- 3 veículos com pelo menos 1 categoria C -> desbloqueia

Observação obrigatória:
- esta regra de desbloqueio é fixa de negócio e deve existir no código/backend

---

## 6. Regra de acumulação do desconto progressivo
Depois de desbloquear o desconto:
- cada novo veículo da categoria C adicionado deve acumular desconto
- não é necessário adicionar mais 3 veículos para novos incrementos
- a acumulação ocorre a cada novo C elegível
- o desconto gerado pelos veículos C é aplicado sobre o valor total da compra/pacote, e não apenas sobre o valor dos veículos C

Interpretação obrigatória:
- veículos C geram o desconto
- o desconto incide sobre o total do pacote
- após o desbloqueio, adicionar um novo veículo C deve provocar recálculo e aumento de desconto, respeitando os tetos

---

## 7. Controle de limite/teto do desconto progressivo
O desconto progressivo deve respeitar simultaneamente dois limites vindos da parametrização da categoria/grupo:
1. limite máximo de desconto percentual
2. limite máximo de desconto em valor monetário

Regra obrigatória:
- o sistema deve travar no primeiro limite atingido
- se atingir o teto percentual antes do monetário, não deve acumular além do teto percentual
- se atingir o teto monetário antes do percentual, não deve acumular além do teto monetário

Consequências obrigatórias:
- ao atingir o teto, o sistema deixa de acumular desconto progressivo adicional
- a barra/mensagem deve mudar para o estado de desconto máximo atingido
- os valores devolvidos ao frontend devem permanecer consistentes com o teto aplicado

---

## 8. Barra de progresso / mensagens dinâmicas
O backend deve fornecer informações suficientes para o frontend já implementado apresentar os estados de progresso.

### Estado 1 — início da compra
Mensagem de referência de negócio:
- “Desbloqueie o seu desconto a cada veículo comprado da categoria essential”
- “Compre 3 veículos, sendo 1 essential, para começar a acumular descontos sobre o valor total da compra.”

### Estado 2 — antes do desbloqueio
Mensagem de referência de negócio:
- “Desbloqueie o seu desconto a cada essential”
- “Adicione mais X veículos, para começar a acumular descontos sobre o valor total da compra.”

O backend deve fornecer os dados necessários para refletir:
- categoria em progresso
- quantidade restante para atingir o desconto
- necessidade de pelo menos um veículo categoria C

### Estado 3 — desbloqueado
Mensagem de referência de negócio:
- “Parabéns, você desbloqueou o desconto para o seu pacote.”
- “Compre mais veículos essential e economize até R$x no valor total da compra”

O backend deve retornar os dados necessários para suportar o valor projetado.

### Estado 4 — limite máximo atingido
Mensagem de referência de negócio:
- “Parabéns, você desbloqueou o desconto essential máximo”
- “Continue comprando à vista e garanta mais desconto”

Regras obrigatórias:
- ao atingir o teto, não sobe mais bloco de desconto progressivo
- o texto/mensagem deve migrar para o estado de máximo atingido

---

## 9. Regras de cálculo do desconto progressivo
Você deve implementar o cálculo respeitando os contratos, estruturas e modelos fornecidos junto com este prompt.

Independentemente do formato interno escolhido, as regras obrigatórias são:
- considerar a quantidade total de veículos no pacote/compra/reserva elegível
- considerar a quantidade de veículos categoria C elegíveis
- aplicar o percentual de desconto por veículo elegível definido na parametrização do grupo/categoria
- o desconto progressivo incide sobre o valor total do pacote
- o desconto progressivo só existe após o desbloqueio
- o valor calculado deve respeitar teto percentual e teto monetário
- o resultado final deve refletir o primeiro teto atingido

Também deve ser suportado o cálculo de **desconto projetado** no estado desbloqueado, com base na regra de negócio consolidada:
- calcular desconto total atual
- calcular desconto total projetado considerando a compra de +1 veículo C
- calcular o desconto adicional projetado como diferença entre os dois
- respeitar os tetos também na projeção
- considerar a média de valor dos veículos C disponíveis quando a referência/projeto exigir isso

---

## 10. Resumo financeiro
O backend deve fornecer os dados necessários para o resumo financeiro da tela, atualizado dinamicamente.

O resumo deve refletir, no mínimo, a composição lógica de:
- valor total da compra
- desconto de pagamento à vista
- desconto por categoria / desconto progressivo
- valor total da compra com descontos

Regras obrigatórias:
- recalcular sempre que houver alteração relevante do pacote/reserva/compra
- desconto à vista aplicado apenas aos itens comprados/reservados com faturamento à vista
- desconto progressivo calculado conforme as regras de desbloqueio e teto
- valores finais devem ser consistentes e auditáveis

---

## 11. Estados de compra e atualização em tempo real
O backend deve sustentar os estados de compra exibidos pelo frontend.

Regras obrigatórias:
- a cada ação de comprar/reservar, recalcular o desconto
- a cada alteração do pacote, recalcular progresso, resumo financeiro e estados derivados
- após compra, o item não deve continuar se comportando como disponível para compra indevida
- a resposta deve refletir corretamente o novo estado do item e do pacote

---

## 12. Regras de faturamento
As opções de faturamento devem vir da parametrização do grupo de DNs.

Tipos conhecidos de negócio:
- Consignado
- Cessão
- +90
- À Vista

Regras obrigatórias:
- só permitir faturamentos habilitados para o grupo do DN logado
- devolver ao frontend as opções válidas do grupo
- devolver ao frontend os CNPJs válidos do grupo, conforme referência fornecida
- validar limites financeiros relacionados aos tipos quando isso fizer parte das referências/regras do projeto

---

## 13. Regras de visibilidade e restrição por grupo exclusivo
Quando uma oferta possuir grupo exclusivo:
- apenas DNs pertencentes a esse grupo podem visualizar a oferta
- apenas DNs pertencentes a esse grupo podem reservar/comprar a oferta
- os demais DNs não devem receber essa oferta na vitrine elegível

Quando não houver grupo exclusivo:
- seguir regras normais de visibilidade por janelas, permissões e filtros

---

## 14. Regras de validação de uploads
### 14.1. Regras gerais
Todo processamento de upload deve:
- validar cabeçalhos obrigatórios
- validar tipos e formatos
- validar presença de colunas obrigatórias
- validar consistência com referências mestre
- persistir corretamente os dados válidos
- registrar/retornar erros de processamento conforme o padrão existente do projeto

### 14.2. Upload de ofertas
Deve bloquear/rejeitar quando ocorrer, entre outros casos:
- ausência das novas colunas obrigatórias
- categoria do veículo inválida
- desconto à vista inválido/não numérico
- grupo exclusivo inválido quando preenchido
- inconsistência estrutural impeditiva

### 14.3. Uploads de parâmetros
Devem validar consistência entre DN, grupo, categorias e permissões conforme as referências fornecidas.

---

## 15. Permissões e segurança
Regras obrigatórias:
- somente usuários permitidos podem acessar a funcionalidade de ofertas
- uploads administrativos devem respeitar o controle de acesso do projeto
- as validações de visibilidade devem sempre considerar DN, grupo, categoria do grupo e grupo exclusivo

---

## 16. Integração com frontend já implementado
A implementação deve ser compatível com o frontend existente.

Obrigatório:
- usar os contratos/DTOs/referências já existentes quando eles forem fornecidos
- se houver lacunas, derivar a implementação a partir dos exemplos e referências adicionais que serão enviados junto com este prompt
- não alterar comportamento esperado do frontend
- garantir que respostas contemplem todos os dados necessários para:
  - listagem
  - cards
  - progresso
  - resumo financeiro
  - permissões
  - filtros
  - estados de compra

---

## 17. Banco de dados e scripts
Você deve gerar **todos os scripts de banco de dados que ainda não existirem** para suportar a implementação.

Regras obrigatórias:
- usar o padrão de migrations/scripts/DDL do projeto
- não inventar nomes quando as referências forem fornecidas
- criar/ajustar estruturas necessárias para suportar as regras descritas neste prompt
- criar/ajustar constraints, índices, relacionamentos e scripts auxiliares conforme necessário para garantir integridade, performance e consistência
- se o projeto utilizar histórico/versionamento de migrations, seguir rigorosamente o padrão existente
- incluir scripts de rollback apenas se isso fizer parte do padrão do projeto

---

## 18. Testes e validações obrigatórias na implementação
A implementação deve cobrir, no mínimo, cenários equivalentes a:
- upload com novas colunas válidas
- upload com categoria inválida
- upload sem novas colunas
- aplicação de desconto à vista apenas em compras à vista
- prioridade/restrição por grupo exclusivo
- filtro por categoria
- desbloqueio correto com mínimo de 3 veículos e 1 categoria C
- não desbloqueio em combinações inválidas
- progressão de desconto com novos veículos C
- travamento no teto percentual
- travamento no teto monetário
- atualização correta do resumo financeiro
- cálculo de projeção de desconto adicional

Use o padrão de testes existente no projeto.

---

## 19. Regras de implementação
Ao implementar:
- priorize aderência às referências reais que serão fornecidas
- preserve integralmente as regras de negócio deste prompt
- não simplifique regras de desbloqueio, teto, visibilidade ou desconto à vista
- não omita validações de upload
- não omita recálculo dinâmico do pacote
- não omita a lógica de grupo exclusivo
- não omita a lógica de permissões
- não deixe scripts de banco pendentes

---

## 20. Resultado esperado da sua execução
Você deve entregar:
1. Implementação completa do backend necessário para suportar a funcionalidade descrita
2. Implementação dos processamentos de upload
3. Implementação das validações
4. Implementação dos cálculos e regras de negócio
5. Implementação das consultas/serviços/endpoints necessários para o frontend existente
6. Todos os scripts de banco/migrations/DDL pendentes
7. Testes automatizados compatíveis com o projeto
8. Observações objetivas sobre qualquer dependência estrutural faltante encontrada nas referências

## 21. Instrução final crítica
Não proponha estrutura abstrata, nem pseudo-solução genérica.
Implemente usando o contexto real, as referências reais, os nomes reais e os padrões reais que serão fornecidos junto com este prompt.
Se houver mais de uma forma de implementar, escolha a que melhor se encaixa na arquitetura e convenções já existentes no projeto.
