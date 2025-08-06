# 📘 Projeto de Integração: Monitoramento de Bancos PostgreSQL com Prometheus, Grafana e Alertmanager

## Objetivo

Implementar um sistema de monitoramento em tempo real para múltiplos bancos de dados PostgreSQL utilizados nos projetos de topografia da empresa. A solução irá coletar, armazenar e exibir métricas de performance dos bancos, além de disparar alertas automáticos em canais como Discord e Telegram quando anomalias forem detectadas.

---

## Componentes da Solução

### 1. Prometheus
- Função: Coleta e armazena métricas dos bancos via exporters.
- Scrape interval: 15 segundos.
- Local: Container Docker.
- Configuração via `prometheus.yml`.

### 2. Grafana
- Função: Exibição de dashboards com métricas em tempo real.
- Autenticação: admin/admin (inicial).
- Dashboards: métricas de PostgreSQL, performance, conexões, locks, etc.
- Local: Container Docker.

### 3. Alertmanager
- Função: Gerenciamento e roteamento de alertas gerados pelo Prometheus.
- Destinos de alerta:
  - Discord (via Webhook)
  - Telegram (via Bot e Chat ID)
- Local: Container Docker.
- Configuração via `alertmanager.yml`.

### 4. PostgreSQL Exporter
- Função: Exporta métricas dos bancos de dados PostgreSQL.
- Implementação: Um container por banco (mesmo IP, databases diferentes).
- Configuração via variável de ambiente `DATA_SOURCE_NAME`.
- Porta interna padrão: 9187 (externa varia por instância).
- Imagem: `prometheuscommunity/postgres-exporter`.

---

## Arquitetura da Solução (Docker)

Todos os serviços serão orquestrados via `docker-compose`, rodando na **mesma máquina física onde ocorre o processamento padrão**.

### Containers previstos:

| Serviço               | Quantidade | Observação                                 |
|-----------------------|------------|--------------------------------------------|
| Prometheus            | 1          | Coleta todas as métricas                   |
| Grafana               | 1          | Dashboards                                 |
| Alertmanager          | 1          | Alertas                                    |
| PostgreSQL Exporter   | n (≥ 1)    | Um por banco PostgreSQL monitorado         |

---

## Fluxo de Dados

```text
[PostgreSQL Exporter (1 por banco)] ---> [Prometheus] ---> [Grafana]
                                          |
                                          +--> [Alertmanager] ---> [Discord / Telegram]
---

## Estratégia de Exportação

Os bancos estão no mesmo IP, porém são bancos distintos (nomes diferentes).

Para facilitar a geração de alertas por banco individualmente, será utilizado 1 exporter por banco, cada um configurado com:

DATA_SOURCE_NAME apontando para o respectivo banco

Porta externa única (ex: 9187, 9188, 9189...)

---

### Estratégia de Alerta

Alertas definidos no Prometheus (via regras).

Envio via Alertmanager para: Discord (webhook direto), Telegram (via bot + chat_id)

Exemplos de condições a serem monitoradas: Alto uso de CPU (se disponível via exporter), Muitas conexões simultâneas, Locks ou deadlocks ativos, Replicação atrasada, Queries lentas


---

### Etapas da Implementação

## Estrutura de Arquivos

Criar arquivos: docker-compose.yml, prometheus.yml, alertmanager.yml

---

### Configuração dos Exporters

Para cada banco: Definir variável DATA_SOURCE_NAME, Mapear porta externa exclusiva, Configuração do Prometheus, Adicionar targets para todos os exporters, Configuração do Alertmanager, Incluir webhook do Discord, Incluir bot e chat_id do Telegram, Subida dos Containers, Executar docker-compose up -d, Importação de Dashboards no Grafana, Usar dashboards públicos da Grafana Labs para teste (ex: ID 9628)

---

### Testes de Alertas

Simular condições para verificar funcionamento do Alertmanager

### Observações de Segurança

Os dados de conexão com os bancos devem ser armazenados com segurança.

Ideal: mover credenciais para arquivos .env ou volumes montados externos ao Git.

- Não expor portas dos exporters publicamente.

- Usar rede Docker isolada (monitoramento) para comunicação interna.

### Pendências Técnicas

- Verificar se todos os bancos permitem conexão externa da máquina de monitoramento.

- Validar permissões do usuário PostgreSQL (mínimo: leitura nas views do sistema).

- Confirmar configuração correta do bot do Telegram.


### Próximos Passos

 - Validar lista completa dos bancos a serem monitorados

 - Gerar arquivos de configuração

 - Subir ambiente de testes

 - Validar alertas reais

 Avaliar necessidade de dashboards customizados
  
