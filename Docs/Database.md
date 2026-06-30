# Esquema de Banco de Dados e Lógica de Relatórios

Este documento detalha a estrutura de armazenamento no SQLite (arquivo único local) e a lógica matemática para a extração de métricas de desempenho e operação da espetaria **Espetô**.

---

## 1. Estrutura do Banco de Dados (SQLite)

O sistema utilizará uma única tabela achatada (*flat table*) para simplificar ao máximo as operações de CRUD, garantindo alta performance e facilidade de manutenção sem a necessidade de chaves estrangeiras complexas para os itens.

### Tabela: `pedidos`

| Coluna | Tipo SQLite | Descrição |
| :--- | :--- | :--- |
| `id_db` | `INTEGER` | Chave primária única universal, autoincrementada (PK). |
| `id_diario` | `INTEGER` | O número do pedido para o cliente (ex: Pedido 01, 02). Reseta a cada turno. |
| `data_operacao` | `TEXT` | Data do turno aberto (`YYYY-MM-DD`). |
| `cliente_nome` | `TEXT` | Nome ou apelido do cliente. |
| `mesa_local` | `TEXT` | Mesa, balcão, ou referência de localização do cliente. |
| `garcom_nome` | `TEXT` | Nome do garçom responsável pelo pedido. |
| `tipo_espeto` | `TEXT` | Sabor/Tipo do espeto (ex: Carne, Frango, Queijo). |
| `status_atual` | `TEXT` | Estado em tempo real: `fila`, `churrasqueira`, `pronto`, `finalizado`, `cancelado`. |
| `ts_criado_em` | `TEXT` | Timestamp exato de quando o pedido entrou na Fila (ISO 8601). |
| `ts_churrasqueira_em` | `TEXT` | Timestamp exato de quando iniciou o preparo na churrasqueira (ISO 8601). |
| `ts_pronto_em` | `TEXT` | Timestamp exato de quando o espeto foi liberado no balcão (ISO 8601). |
| `ts_entregue_em` | `TEXT` | Timestamp exato de quando o garçom retirou para entrega (ISO 8601). |
| `ts_cancelado_em` | `TEXT` | Timestamp de cancelamento, se aplicável (`NULL` por padrão). |

> **Nota Técnica:** Como o SQLite não possui um tipo `DATETIME` nativo, os timestamps serão armazenados como `TEXT` no formato ISO 8601 (`YYYY-MM-DD HH:MM:SS`). Este formato permite a ordenação correta e a utilização de funções de data e hora nativas do SQLite (`julianday` ou `strftime`) para cálculos diretos de diferença de tempo.

---

## 2. Motor de Relatórios (Lógica e Cálculos)

A extração de métricas baseia-se na subtração matemática dos timestamps registrados de forma automatizada e oculta durante o ciclo de vida de cada pedido finalizado.

### 2.1. Métricas de Tempo por Etapa

* **Tempo na Fila:** `ts_churrasqueira_em - ts_criado_em`
  * *Indicador:* Nível de sobrecarga da churrasqueira na absorção de novos pedidos (tempo de espera antes de entrar na grelha).
* **Tempo de Preparo (Grelha):** `ts_pronto_em - ts_churrasqueira_em`
  * *Indicador:* Velocidade de cocção/preparo da equipe na churrasqueira, ideal para monitorar a eficiência por tipo de carne.
* **Tempo de Retirada (Balcão):** `ts_entregue_em - ts_pronto_em`
  * *Indicador:* Eficiência logística dos garçons em retirar os pedidos prontos da bancada.
* **Tempo Total (Percepção do Cliente):** `ts_entregue_em - ts_criado_em`
  * *Indicador:* Qualidade geral do atendimento e tempo de espera final experimentado pelo cliente.

### 2.2. Relatórios de Gestão e Painéis

Os agrupamentos e agregações abaixo servem tanto para alimentar a tela de estatísticas dos garçons em tempo real quanto para auditorias históricas com o sistema fechado:

* **Métricas do Painel de Garçons (Turno Atual):**
  * Para o dia corrente (`data_operacao`), o sistema calcula por `garcom_nome`:
    * **Pedidos Coletados:** Contagem total de registros do garçom.
    * **Em Preparo:** Contagem onde `status_atual` está como `fila` ou `churrasqueira`.
    * **Finalizados:** Contagem onde `status_atual` é `finalizado`.
    * **Tempo Média de Retirada:** Média calculada (`AVG`) do *Tempo de Retirada (Balcão)* para todos os pedidos concluídos pelo garçom no dia, atualizada instantaneamente a cada entrega.

* **Gargalos de Production por Espeto:**
  * *Métrica:* Média do **Tempo de Preparo (Grelha)**.
  * *Agrupamento:* Por `tipo_espeto`. Permite identificar quais produtos demandam mais tempo de fogo.

* **Desempenho de Logística por Garçom (Histórico):**
  * *Métrica:* Média do **Tempo de Retirada (Balcão)**.
  * *Agrupamento:* Por `garcom_nome` em períodos selecionados, útil para avaliar a consistência da equipe.

* **Saúde do Turno (Visão Geral):**
  * *Métrica:* Média do **Tempo Total** e **Tempo na Fila**.
  * *Agrupamento:* Por blocos de horário (intervalos de 1 hora) com base no `ts_criado_em`, mapeando os picos de estrangulamento do atendimento.
