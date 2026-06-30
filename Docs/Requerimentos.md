# Documento de Requisitos e Regras de Negócio

Este documento define os requisitos funcionais, não funcionais e as regras de negócio para o sistema de gerenciamento, otimização de fila e controle de preparo da espetaria **Espetô**.

---

## 1. Visão Geral do Sistema

Um aplicativo desktop desenvolvido para otimizar o fluxo de pedidos, balancear a carga de trabalho dos garçons e gerenciar a capacidade da churrasqueira. O objetivo é rastrear os pedidos, estimar tempos de espera precisos, otimizar a ordem de preparo através de uma fila dinâmica e evitar o extravio de comandas.

## 2. Telas e Navegação (UI/UX)

O fluxo do sistema é dividido em 7 áreas modulares para cobrir todo o ciclo de vida do pedido e a gestão do bar:

* **Tela 01 - Início e Gestão:** Tela de login/abertura do turno. Contém uma aba gerencial (protegida por senha) para configuração de parâmetros globais da espetaria, como a capacidade física da churrasqueira.
* **Tela 02 - Operação de Pedidos (Caixa):** Formulário rápido de cadastro de novos pedidos.
* **Tela 03 - Fila Dinâmica (Pré-Churrasqueira):** Interface exclusiva para os pedidos aguardando espaço na grelha. Ordena itens por pontuação baseada em tempo e origem.
* **Tela 04 - Painel da Churrasqueira (Em Preparo):** Visualização dos pedidos ativos na grelha, exibindo os temporizadores individuais, alertas de cores mantidos da fila e a ocupação da churrasqueira.
* **Tela 05 - Balcão de Retirada (Prontos):** Interface dedicada aos espetos que já foram assados e estão aguardando a coleta pelo garçom ou cliente.
* **Tela 06 - Histórico de Pedidos:** Tela para visualização de todos os pedidos finalizados e cancelados, além da geração de relatórios de dias anteriores.
* **Tela 07 - Dashboard de Equipe:** Painel em tempo real com estatísticas de desempenho logístico do turno atual.

## 3. Requisitos Funcionais (RF)

* **RF01 - Cadastro de Pedido com Origem:** O sistema deve registrar pedidos categorizando sua origem: *Casa* (Garçom ou Totem) ou *Rua* (cliente externo, sem garçom).
* **RF02 - Cadastro Geral de Equipe:** O sistema deve permitir o cadastro de funcionários atribuindo perfis distintos: *Admin*, *Garçom*, *Churrasqueiro* e *Outro* (campo aberto para descrição).
* **RF03 - Geração de Identificador:** O sistema deve gerar automaticamente um ID sequencial para cada novo pedido registrado no dia.
* **RF04 - Registro Oculto de Timestamps:** Captura e registro em segundo plano do horário exato em que o pedido transita entre as etapas.
* **RF05 - Gestão e Transição de Status:** O sistema deve possuir botões de ação explícitos para mover o pedido pelo fluxo operacional: *Na Fila* -> *Na Churrasqueira* -> *Pronto para Entrega* -> *Finalizado*.
* **RF06 - Janela de Reversão (Desfazer):** Ao realizar qualquer transição de status (ex: enviar para a grelha ou finalizar), o sistema deve exibir um aviso temporário de 15 segundos permitindo desfazer a ação, revertendo o pedido ao status anterior.
* **RF07 - Alertas Visuais de SLA (Tempo de Serviço):** O sistema deve alterar visualmente a indicação do pedido com base no tempo total decorrido desde a sua criação, independente da etapa em que se encontra. A cor de alerta não reseta ao avançar de tela.

| Tempo Total | Indicador Visual | Status |
| :--- | :--- | :--- |
| 0 a 20 min | Verde | Dentro do prazo |
| 20 a 25 min | Amarelo | Atenção |
| 29 a 30 min | Texto piscando intensamente | Crítico |
| > 30 min | Vermelho | Atrasado |

* **RF08 - Fila Dinâmica (Priorização Automática):** O sistema deve reordenar os pedidos da etapa *Na Fila* aplicando pesos diferentes. Pedidos da *Casa* possuem multiplicadores maiores de prioridade que pedidos da *Rua*.
* **RF09 - Prevenção de Fome (Starvation):** Pedidos da *Rua* que ultrapassarem um limite de tolerância de atraso recebem um bônus de pontuação crítico para escalar a fila.
* **RF10 - Alocação Inteligente de Totem:** Pedidos originados no *Totem* devem ser automaticamente atribuídos ao garçom logado com o menor volume de pedidos ativos e menor tempo médio de retirada.
* **RF11 - Gestão de Parâmetros (Admin):** A interface gerencial deve permitir a alteração de variáveis do sistema, como a capacidade total ($N$) de espetos simultâneos na churrasqueira.
* **RF12 - Estimativa de Capacidade (Grelha):** Exibição de um indicador visual de lotação da churrasqueira baseado no parâmetro $N$ e no espaço exigido por cada tipo de espeto.
* **RF13 - Estimativa de Tempo de Espera:** Ao cadastrar um novo pedido, o sistema exibe uma estimativa de tempo (ETA) com base na ocupação da churrasqueira e nas médias de preparo do tipo do espeto no dia.

## 4. Requisitos Não Funcionais (RNF)

* **RNF01 - Interface de Alta Visibilidade (UI/UX):** Otimizada para leitura rápida em ambientes com pouca luz e fumaça (Alto Contraste / Dark Mode).
* **RNF02 - Persistência Centralizada (SQLite):** Armazenamento em arquivo único local para garantir alta performance de leitura e cálculo heurístico em tempo real.
* **RNF03 - Processamento Assíncrono:** As rotinas de ordenação da Fila Dinâmica e atualização de cronômetros não devem interromper a fluidez da operação no caixa.
* **RNF04 - Resiliência Operacional:** Recuperação de estado e recálculo de cronômetros em caso de falha de energia, utilizando os timestamps persistidos.

## 5. Regras de Negócio (RN)

* **RN01 - Isolamento de Turnos:** O ID do pedido para o cliente é diário. Relatórios e lógicas de média diária devem agrupar os dados estritamente pela data do turno aberto.
* **RN02 - Cálculo Estrito de Timestamps:** Relatórios analíticos utilizam a diferença matemática real salva no banco de dados.
* **RN03 - SLA por Categoria:** O tempo tolerável e a cor de alerta consideram se o cliente é interno (prioridade na experiência de mesa) ou externo (tolerância maior).
* **RN04 - Adiantamento Restrito:** O sistema só pode sugerir o adiantamento de um pedido se isso não estender o SLA do próximo pedido prioritário e se houver capacidade ociosa física na grelha.
* **RN05 - Controle de Acesso:** A aba gerencial, incluindo a função de criar novos administradores ou alterar parâmetros da churrasqueira, é restrita mediante validação de senha.
* **RN06 - Segurança da Senha Mestre:** A senha global de administração deve ser armazenada no banco de dados exclusivamente na forma de um *hash* criptográfico seguro, impedindo a leitura direta em caso de acesso indevido ao arquivo.
