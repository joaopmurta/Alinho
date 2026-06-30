Documento de Requisitos e Regras de Negócio - Sistema Espetaria

Este documento define os requisitos funcionais, não funcionais e as regras de negócio para o sistema de gerenciamento de fila e tempo de preparo da espetaria.

1. Visão Geral do Sistema

Um aplicativo desktop (Python) projetado para otimizar o fluxo de pedidos entre os garçons e a churrasqueira, garantindo rastreabilidade, controle de tempo de preparo e redução de perdas por pedidos extraviados ou atrasados durante horários de pico.

2. Requisitos Funcionais (RF)

O que o sistema deve fazer de forma prática.

RF01 - Cadastro de Pedido: O sistema deve permitir a inserção rápida de um novo pedido contendo: Nome do Cliente, Mesa/Localização, Nome do Garçom e Tipo do Espeto.

RF02 - Geração de Identificador: O sistema deve gerar automaticamente um ID (inteiro sequencial) para cada novo pedido registrado no dia.

RF03 - Registro Oculto de Timestamps: O sistema deve capturar e registrar automaticamente (em segundo plano) o horário exato em que o pedido entra em cada uma das etapas do fluxo.

RF04 - Gestão de Status: O sistema deve permitir a transição do pedido entre os seguintes estados:

Na Fila

Na Churrasqueira (Em preparo)

Pronto para Entrega

Finalizado

RF05 - Temporizadores Independentes: O sistema deve acionar cronômetros distintos na interface para monitorar o tempo gasto em cada status ativo (Fila, Churrasqueira e Pronto).

RF06 - Alertas Visuais de SLA (Tempo de Serviço): O sistema deve alterar visualmente a indicação do pedido com base no tempo total, conforme as seguintes faixas:

0 a 20 min: Indicador Verde (Dentro do prazo).

20 a 25 min: Indicador Amarelo (Atenção).

29 a 30 min: O texto do horário deve piscar intensamente (Crítico).

> 30 min: Indicador Vermelho (Atrasado).

RF07 - Reforço de Atraso: Para pedidos no status Vermelho (>30 min), o sistema deve emitir um alerta visual de reforço a cada 3 minutos.

RF08 - Finalização de Pedido: O sistema deve possuir um comando/botão para confirmar a entrega ao cliente, movendo o pedido para uma visualização de "Histórico".

RF09 - Cancelamento/Edição: O sistema deve permitir a alteração dos dados de um pedido ou o seu cancelamento antes da finalização.

RF10 - Busca e Filtragem: O sistema deve possuir um campo de busca rápida por Nome do Cliente, Mesa ou ID do pedido.

RF11 - Módulo de Relatórios (Preparo): O sistema deve permitir a emissão de relatório calculando o tempo médio que cada Tipo de Espeto leva no status Na Churrasqueira.

RF12 - Módulo de Relatórios (Gargalos): O sistema deve permitir visualizar o tempo médio geral de espera na etapa Na Fila.

RF13 - Módulo de Relatórios (Logística): O sistema deve permitir visualizar o tempo médio que os pedidos aguardam no balcão (Pronto para Entrega), podendo ser agrupado por Garçom.

3. Requisitos Não Funcionais (RNF)

Como o sistema deve se comportar tecnicamente.

RNF01 - Interface de Alta Visibilidade (UI/UX): A interface gráfica deve utilizar alto contraste, fontes grandes e cores intuitivas, otimizada para leitura rápida em ambientes com pouca luz ou muita fumaça (ex: Dark Mode).

RNF02 - Persistência Local: Os dados devem ser armazenados localmente utilizando um banco de dados leve e embutido (ex: SQLite), sem dependência de internet ou servidores externos.

RNF03 - Responsividade da Interface: As rotinas de contagem de tempo (cronômetros) e as animações (piscar alertas) não devem travar ou causar lentidão na inserção de novos pedidos.

RNF04 - Resiliência e Recuperação de Estado: Em caso de fechamento acidental ou queda de energia, o sistema, ao ser reiniciado, deve recalcular os cronômetros ativos baseando-se nos timestamps salvos no banco de dados, sem perda da contagem de tempo.

RNF05 - Distribuição: O sistema deve ser compilado como um arquivo executável único (ex: .exe para Windows) para facilitar a instalação no computador do caixa/churrasqueira sem necessidade de configurar ambiente Python.

4. Regras de Negócio (RN)

Condições e restrições que guiam as operações do sistema.

RN01 - Reinício do ID: O ID sequencial dos pedidos deve ser reiniciado (voltar para 1) no início de cada novo dia ou turno de operação.

RN02 - Cálculo de Tempos (Timestamps): As métricas de relatórios devem ser estritamente baseadas na diferença matemática entre os horários (timestamps) salvos no banco de dados, independentemente de atrasos na interface visual.

RN03 - Manutenção do SLA (Alertas): A regra de mudança de cor (Verde -> Amarelo -> Vermelho) considera o tempo total do pedido (desde a entrada Na Fila até ser entregue), refletindo a percepção real de espera do cliente.

RN04 - Prevenção de Exclusão: Um pedido só pode ser fisicamente excluído do banco de dados por um perfil administrador (se implementado); cancelamentos normais na operação devem apenas registrar o timestamp de cancelamento e manter o histórico.

RN05 - Transição Lógica de Status: Um pedido não pode pular a etapa Na Churrasqueira (ir direto de Na Fila para Pronto), a menos que seja classificado como um item que não exige preparo térmico.
