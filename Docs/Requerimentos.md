# Documento de Requisitos e Regras de Negócio

Este documento define os requisitos funcionais, não funcionais e as regras de negócio para o sistema de gerenciamento, otimização de fila e controle de preparo da espetaria **Espetô**.

---

## 1. Visão Geral do Sistema

Um aplicativo desktop desenvolvido para otimizar o fluxo de pedidos, balancear a carga de trabalho dos garçons e gerenciar a capacidade da churrasqueira. O objetivo é rastrear os pedidos, estimar tempos de espera precisos, otimizar a ordem de preparo através de uma fila dinâmica e evitar o extravio de comandas.

## 2. Telas e Navegação (UI/UX)

O sistema será estruturado em um fluxo de navegação contendo 5 áreas principais:

* **Tela 01 - Gestão de Turno (Início):** A tela onde o operador decide se vai "Abrir um novo dia/turno" ou entrar no modo "Apenas Relatórios" para consultar dados passados.
* **Tela 02 - Operação de Pedidos (Caixa):** Formulário rápido de cadastro de novos pedidos.
* **Tela 03 - Fila Dinâmica (Pré-Churrasqueira):** Interface exclusiva para os pedidos que ainda não foram para a grelha. Ordena os itens automaticamente através de um sistema de pontuação baseado no tempo de espera e na origem do cliente. Destaca visualmente sugestões de adiantamento (ex: itens rápidos que cabem na grelha atual).
* **Tela 04 - Painel da Churrasqueira (Em Preparo):** Visualização em lista/cards dos pedidos ativos na grelha, exibindo os temporizadores individuais, alertas de cores e a estimativa de ocupação física da churrasqueira.
* **Tela 05 - Dashboard de Garçons:** Painel atualizado em tempo real com estatísticas do turno atual (pedidos atribuídos, em preparo, finalizados e o tempo médio corrente de retirada do balcão).

## 3. Requisitos Funcionais (RF)

* **RF01 - Cadastro de Pedido com Origem:** O sistema deve registrar pedidos categorizando sua origem: *Casa* (Garçom ou Totem) u *Rua* (cliente externo, sem garçom).
* **RF02 - Geração de Identificador:** O sistema deve gerar automaticamente um ID sequencial para cada novo pedido registrado no dia.
* **RF03 - Registro Oculto de Timestamps:** Captura e registro em segundo plano do horário exato em que o pedido transita entre as etapas.
* **RF04 - Gestão de Status:** Transição lógica dos estados: *Na Fila* -> *Na Churrasqueira* -> *Pronto para Entrega* -> *Finalizado*.
* **RF05 - Temporizadores Independentes:** Cronômetros distintos na interface para monitorar o tempo gasto nas etapas ativas.
* **RF06 - Fila Dinâmica (Priorização Automática):** O sistema deve reordenar os pedidos da etapa *Na Fila* aplicando pesos diferentes. Pedidos da *Casa* possuem multiplicadores maiores de prioridade que pedidos da *Rua*.
* **RF07 - Prevenção de Fome (Starvation):** Pedidos da *Rua* que ultrapassarem um limite de tolerância de atraso devem receber um bônus de pontuação crítico para escalar a fila e impedir espera infinita.
* **RF08 - Alocação Inteligente de Totem:** Para pedidos originados no *Totem*, o sistema deve automaticamente atribuir um garçom ativo no turno. A lógica deve buscar o garçom com menor volume de pedidos em andamento e menor tempo médio de retirada.
* **RF09 - Estimativa de Capacidade (Grelha):** O sistema deve reconhecer o "tamanho/espaço" de cada tipo de espeto e exibir um indicador visual de capacidade total da churrasqueira, barrando ou alertando sobre superlotação.
* **RF10 - Estimativa de Tempo de Espera:** Ao cadastrar um novo pedido, o sistema deve fornecer uma estimativa de tempo (ETA) com base na ocupação atual da churrasqueira e nas médias de preparo do tipo do espeto no dia.
* **RF11 - Alertas Visuais de SLA (Preparo):** O sistema deve alterar visualmente a indicação do pedido com base no tempo total decorrido, evoluindo de Verde para Amarelo e Vermelho, com alertas intermitentes em caso de atraso crítico.
* **RF12 - Finalização e Cancelamento:** Controles para confirmar a entrega ao cliente ou cancelar itens, mantendo o histórico de timestamps inalterado.
* **RF13 - Monitoramento de Eficiência (Tempo Real):** A Tela 05 deve exibir as métricas instantâneas dos garçons para auxiliar na coordenação logística do balcão.

## 4. Requisitos Não Funcionais (RNF)

* **RNF01 - Interface de Alta Visibilidade (UI/UX):** Otimizada para leitura rápida em ambientes com pouca luz e fumaça (Alto Contraste / Dark Mode).
* **RNF02 - Persistência Centralizada (SQLite):** Armazenamento em arquivo único local para garantir alta performance de leitura e cálculo heurístico em tempo real.
* **RNF03 - Processamento Assíncrono:** As rotinas de ordenação da Fila Dinâmica e atualização de cronômetros não devem interromper o cadastro no caixa.
* **RNF04 - Resiliência Operacional:** Recuperação de estado e recálculo de cronômetros em caso de falha de energia, utilizando os timestamps persistidos.

## 5. Regras de Negócio (RN)

* **RN01 - Isolamento de Turnos:** O ID do pedido para o cliente é diário. Relatórios e lógicas de média diária devem agrupar os dados estritamente pela data do turno aberto.
* **RN02 - Cálculo Estrito de Timestamps:** Relatórios analíticos utilizam a diferença matemática real salva no banco de dados.
* **RN03 - SLA por Categoria:** O tempo tolerável e a cor de alerta devem considerar se o cliente é interno (prioridade na experiência de mesa) ou externo (tolerância maior).
* **RN04 - Adiantamento Restrito:** O sistema (via IA ou heurística) só pode sugerir o adiantamento de um pedido (furar a fila) se isso não estender o SLA do próximo pedido prioritário e se houver capacidade ociosa física na grelha para aquele tipo de espeto.
* **RN05 - Transição Lógica:** Pedidos de carne/preparo térmico não podem pular a etapa *Na Churrasqueira*.
