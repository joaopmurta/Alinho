# Documento de Requisitos e Regras de Negócio

Este documento define os requisitos funcionais, não funcionais e as regras de negócio para o sistema de gerenciamento de fila e tempo de preparo da espetaria **Espetô**.

---

## 1. Visão Geral do Sistema

Um aplicativo desktop desenvolvido para otimizar o fluxo de pedidos entre os garçons e a churrasqueira. O objetivo é garantir rastreabilidade, controle preciso do tempo de preparo e redução de perdas por pedidos extraviados ou atrasados durante os horários de pico.

## 2. Telas e Navegação (UI/UX)

O sistema será estruturado em um fluxo de navegação contendo, a princípio, 4 telas principais:

* **Tela 01 - Gestão de Turno (Início):** A tela inicial onde o operador decide se vai "Abrir um novo dia/turno" (iniciando o registro de novos pedidos) ou entrar no modo "Apenas Relatórios" para consultar dados passados.
* **Tela 02 - Operação de Pedidos (Caixa/Churrasqueira):** A tela principal de trabalho. Deve conter o formulário rápido de cadastro de novos pedidos e a visualização em lista/cards dos pedidos ativos, com seus respectivos cronômetros e alertas de cores.
* **Tela 03 - Painel de Garçons:** Um *dashboard* atualizado em tempo real mostrando as estatísticas do turno atual para cada garçom (pedidos coletados, em preparo, finalizados e o SLA de retirada do balcão).
* **Tela 04 - Configurações (Cadastros):** Tela auxiliar para registrar e gerenciar a lista de Garçons ativos e o Cardápio (Tipos de Espetos), alimentando os campos de seleção da Tela 02.

## 3. Requisitos Funcionais (RF)

* **RF01 - Cadastro de Pedido:** O sistema deve permitir a inserção rápida de um novo pedido contendo: Nome do Cliente, Mesa/Localização, Nome do Garçom e Tipo do Espeto.
* **RF02 - Geração de Identificador:** O sistema deve gerar automaticamente um ID (inteiro sequencial) para cada novo pedido registrado no dia.
* **RF03 - Registro Oculto de Timestamps:** O sistema deve capturar e registrar automaticamente (em segundo plano) o horário exato em que o pedido entra em cada uma das etapas do fluxo.
* **RF04 - Gestão de Status:** O sistema deve permitir a transição do pedido entre os seguintes estados:
  1. *Na Fila*
  2. *Na Churrasqueira* (Em preparo)
  3. *Pronto para Entrega*
  4. *Finalizado*
* **RF05 - Temporizadores Independentes:** O sistema deve acionar cronômetros distintos na interface para monitorar o tempo gasto em cada status ativo (Fila, Churrasqueira e Pronto).
* **RF06 - Alertas Visuais de SLA (Tempo de Serviço):** O sistema deve alterar visualmente a indicação do pedido com base no tempo total, conforme as seguintes faixas:
  * `0 a 20 min`: Indicador Verde (Dentro do prazo).
  * `20 a 25 min`: Indicador Amarelo (Atenção).
  * `29 a 30 min`: O texto do horário deve piscar intensamente (Crítico).
  * `> 30 min`: Indicador Vermelho (Atrasado).
* **RF07 - Reforço de Atraso:** Para pedidos no status Vermelho (>30 min), o sistema deve emitir um alerta visual de reforço a cada 3 minutos via pop-up que não obstrua a tela.
* **RF08 - Finalização de Pedido:** O sistema deve possuir um comando/botão para confirmar a entrega ao cliente, movendo o pedido para uma visualização de "Histórico".
* **RF09 - Cancelamento/Edição:** O sistema deve permitir a alteração dos dados de um pedido ou o seu cancelamento antes da finalização.
* **RF10 - Busca e Filtragem:** O sistema deve possuir um campo de busca rápida por Nome do Cliente, Mesa ou ID do pedido.
* **RF11 - Monitoramento de Garçons em Tempo Real:** A Tela 03 deve calcular e exibir, a cada novo pedido finalizado no turno atual, as métricas em tempo real de cada profissional.
* **RF12 - Módulo de Relatórios Históricos:** Permitir a geração de relatórios de dias anteriores (acessados pela Tela 01) sem a necessidade de abrir um turno de vendas.
* **RF13 - Módulo de Relatórios (Preparo):** O sistema deve permitir a emissão de relatório calculando o tempo médio que cada Tipo de Espeto leva no status *Na Churrasqueira*.
* **RF14 - Módulo de Relatórios (Gargalos):** O sistema deve permitir visualizar o tempo médio geral de espera na etapa *Na Fila*.
* **RF15 - Módulo de Relatórios (Logística):** O sistema deve permitir visualizar o tempo médio que os pedidos aguardam no balcão (*Pronto para Entrega*), podendo ser agrupado por Garçom.

## 4. Requisitos Não Funcionais (RNF)

* **RNF01 - Interface de Alta Visibilidade (UI/UX):** A interface gráfica deve utilizar alto contraste, fontes grandes e cores intuitivas, otimizada para leitura rápida em ambientes com pouca luz ou muita fumaça (ex: Dark Mode).
* **RNF02 - Persistência Centralizada:** Os dados de todos os dias devem ser armazenados em um único arquivo de banco de dados SQLite salvo em um diretório fixo e mapeado pelo programa (ex: `\dados_app\banco_espeto.db`), garantindo fácil acesso e backups rápidos.
* **RNF03 - Responsividade da Interface:** As rotinas de contagem de tempo (cronômetros) e as animações (piscar alertas) não devem travar ou causar lentidão na inserção de novos pedidos.
* **RNF04 - Resiliência e Recuperação de Estado:** Em caso de fechamento acidental ou queda de energia, o sistema, ao ser reiniciado, deve recalcular os cronômetros ativos baseando-se nos timestamps salvos no banco de dados, sem perda da contagem de tempo.
* **RNF05 - Distribuição:** O sistema deve ser compilado como um arquivo executável único para facilitar a instalação na máquina operacional.

## 5. Regras de Negócio (RN)

* **RN01 - Reinício do ID:** O ID sequencial dos pedidos deve ser reiniciado (voltar para 1) no início de cada novo dia ou turno de operação.
* **RN02 - Cálculo de Tempos (Timestamps):** As métricas de relatórios devem ser estritamente baseadas na diferença matemática entre os horários (timestamps) salvos no banco de dados, independentemente de atrasos visuais na interface.
* **RN03 - Manutenção do SLA (Alertas):** A regra de mudança de cor considera o **tempo total** do pedido (desde a entrada *Na Fila* até ser entregue), refletindo a percepção real de espera do cliente.
* **RN04 - Prevenção de Exclusão:** Um pedido só pode ser fisicamente excluído do banco de dados por um perfil administrador; cancelamentos normais na operação devem apenas registrar o timestamp de cancelamento e manter o histórico.
* **RN05 - Transição Lógica de Status:** Um pedido não pode pular a etapa *Na Churrasqueira* (ir direto de *Na Fila* para *Pronto*), a menos que seja classificado como um item que não exige preparo térmico.
* **RN06 - Isolamento de Turnos:** O ID do pedido para o cliente é diário, mas o ID do banco de dados é universal. Os relatórios devem agrupar os dados estritamente pela data do turno aberto.
* **RN07 - Padronização de Entradas:** Não deve ser permitida a digitação livre nos campos de "Garçom" e "Tipo de Espeto" na criação do pedido, evitando inconsistências nos relatórios (exigindo uso do cadastro prévio).
