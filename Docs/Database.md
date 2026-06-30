# Esquema de Banco de Dados e Lógica Analítica

Este documento detalha a estrutura de armazenamento no SQLite (flat table) e as lógicas matemáticas (heurísticas) para ordenação de filas e extração de métricas operacionais da espetaria **Espetô**.

---

## 1. Estrutura do Banco de Dados (SQLite)

Para manter a alta performance em consultas analíticas e ordenações de fila em tempo real, adotamos o modelo de tabela achatada sem chaves estrangeiras complexas.

### Tabela: `pedidos`

| Coluna | Tipo SQLite | Descrição |
| :--- | :--- | :--- |
| `id_db` | `INTEGER` | Chave primária única universal, autoincrementada (PK). |
| `id_diario` | `INTEGER` | O número do pedido para o cliente. Reseta a cada turno. |
| `data_operacao` | `TEXT` | Data do turno aberto (`YYYY-MM-DD`). |
| `cliente_nome` | `TEXT` | Nome, apelido do cliente ou "Totem". |
| `origem_pedido` | `TEXT` | `mesa`, `totem` ou `rua`. Define a prioridade do algoritmo. |
| `mesa_local` | `TEXT` | Número da mesa, balcão ou "Para Levar". |
| `garcom_nome` | `TEXT` | Nome do responsável, preenchido manualmente ou via alocação automática (Totem). `NULL` para pedidos da rua. |
| `tipo_espeto` | `TEXT` | Sabor/Tipo do espeto (ex: Carne, Frango, Queijo). |
| `espaco_grelha` | `INTEGER` | Representa o volume físico que o espeto ocupa (ex: Queijo = 1, Picanha = 2). |
| `status_atual` | `TEXT` | `fila`, `churrasqueira`, `pronto`, `finalizado`, `cancelado`. |
| `ts_criado_em` | `TEXT` | Timestamp exato da entrada do pedido (ISO 8601). |
| `ts_churrasqueira_em` | `TEXT` | Timestamp do início do preparo térmico (ISO 8601). |
| `ts_pronto_em` | `TEXT` | Timestamp da liberação no balcão (ISO 8601). |
| `ts_entregue_em` | `TEXT` | Timestamp da retirada final (ISO 8601). |

---

## 2. Motor Heurístico e Lógicas de Fila

Antes da implementação de modelos preditivos (IA), o sistema operará com equações baseadas em regras e pesos.

### 2.1. Lógica da Fila Dinâmica (Priorização)
A tela de pré-churrasqueira ordena os itens em ordem decrescente com base na pontuação dinâmica. A pontuação é recalculada a cada minuto pela interface:

* **Variáveis:**
  * `Delta_Espera`: (Hora Atual) - `ts_criado_em` (em minutos).
  * `Peso`: Mesa = 1.0, Totem = 1.0, Rua = 0.6.
* **Cálculo Base:**
  $Pontuacao = Delta\_Espera \times Peso$
* **Escalonamento Crítico (Prevenção de Fome):** Se um pedido da Rua atingir um `Delta_Espera` > 30 minutos, um fator multiplicador agressivo é somado, forçando o pedido a subir para o topo da grelha.

### 2.2. Lógica de Alocação de Garçom (Pedidos do Totem)
Ao receber um `origem_pedido = 'totem'`, o sistema varre os garçons ativos no turno `data_operacao` e escolhe o ideal (menor score de sobrecarga):

* **Score de Sobrecarga:** $Score = (Pedidos\_Ativos \times 2) + Tempo\_Medio\_Retirada\_Em\_Minutos$
* O garçom com o **menor Score** é associado automaticamente à coluna `garcom_nome` do novo pedido.

### 2.3. Estimativa de Tempo e Capacidade
* **Disponibilidade da Churrasqueira:** $Capacidade\_Livre = Capacidade\_Total\_Fisica - \sum espaco\_grelha$ (onde status_atual = 'churrasqueira').
* **Tempo Estimado (ETA Inicial):** Calculado com base na Média Móvel Simples (SMA) do *Tempo de Preparo (Grelha)* dos últimos 10 espetos idênticos finalizados no dia, somado ao tempo de espera projetado pela Fila Dinâmica.

---

## 3. Relatórios e Métricas de Gestão

A extração de dados ocorre através da subtração dos timestamps. 

### 3.1. Métricas Base de Tempo
* **Tempo na Fila:** `ts_churrasqueira_em - ts_criado_em`
* **Tempo de Preparo (Grelha):** `ts_pronto_em - ts_churrasqueira_em`
* **Tempo de Retirada (Balcão):** `ts_entregue_em - ts_pronto_em`
* **Tempo Total do Cliente:** `ts_entregue_em - ts_criado_em`

### 3.2. Agrupamentos Analíticos
* **Eficiência Logística (Balcão):** Média do *Tempo de Retirada*, agrupado por `garcom_nome`. Utilizado para mapear quem entrega rápido e quem esfria o produto.
* **Gargalo por Produto:** Média do *Tempo de Preparo*, agrupado por `tipo_espeto`. Variável crucial para alimentar o motor de ETA e eventuais modelos futuros de IA.
* **Rastreamento de Extravios e Atrasos:** Identificação de outliers onde o `Tempo Total` excede o Desvio Padrão do turno, isolando pedidos perdidos na operação.
