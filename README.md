# industrial-data-architecture-azure
Arquitetura de dados industrial end-to-end no Azure utilizando Medallion Architecture (Delta Lake). Solução escalável para ingestão de sensores IoT via MQTT/OPC-UA, com processamento em tempo real (Spark Streaming), governança avançada, rastreabilidade via SCD2 e consumo híbrido.

![Diagrama armazenamento_industrial](https://github.com/user-attachments/assets/b9bb2572-6582-4a56-b1e9-91e35e600810)


Esta arquitetura foi desenvolvida para ambientes industriais que exigem ingestão contínua de dados de sensores IoT, processamento em tempo real, conformação analítica com controle histórico (SCD2), governança ponta a ponta e consumo híbrido via Power BI e Data Warehouse. Ela combina tecnologias modernas como Delta Lake, Spark Structured Streaming, Azure Purview e Fabric Lakehouse.

## Camadas da Arquitetura / Architecture Layers

## 1. Fonte de Dados / Data Source
Sensores IoT (MQTT, OPC-UA)
Geração de eventos industriais em tempo real.

## 2. Ingestão / Ingestion
Azure IoT Edge + Azure IoT Hub
Coleta local e ingestão segura na nuvem.

## 3. Armazenamento / Storage
Data Lake Gen2 com containers:
raw/: dados brutos e imutáveis.
silver/: dados limpos e padronizados.
gold/: dados conformados para consumo analítico.

## 4. Processamento / Processing
Databricks Spark Structured Streaming + Delta Lake
Processamento contínuo com checkpoint e tolerância a falhas.

## 5. Conformação Analítica / Analytical Conformation
SCD Tipo 2 aplicado em dimensões como sensores e equipamentos.
Controle de histórico com valid_from, valid_to, is_current.

## 6. Consumo / Consumption
Lakehouse (Fabric): leitura direta do gold via Direct Lake.
Data Warehouse: modelo analítico com dimensões e fatos.
Power BI: dashboards operacionais e estratégicos.

## 7. Governança / Governance
Azure Purview: catálogo, linhagem e glossário de dados.
Azure Monitor: alertas, métricas e observabilidade operacional.

## Fluxo de Dados / Data Flow
SENSORES → IoT Edge → IoT Hub → RAW
       ↘ Spark Streaming + Checkpoint → SILVER → GOLD
            ↘ SCD2 → DW (dim/fact)
            ↘ Lakehouse (Fabric) → Power BI
Governança transversal: Purview + Monitor

## Checklists Auditáveis / Audit-Ready Checklists

## Data Lake

[x] Containers separados: raw, silver, gold

[x] Particionamento por tempo e domínio

[x] Metadados: origem, batch_id, record_hash

## Streaming + Delta

[x] Checkpoint configurado

[x] Idempotência garantida

[x] Reprocessamento seguro

## SCD2

[x] Dimensões com controle temporal

[x] MERGE com comparação de atributos

[x] Fatos com lookup temporal

## DW

[x] Schemas: stg, dim, fact

[x] Partições por data

[x] Integridade referencial

## Governança

[x] Purview ativo com catálogo e linhagem

[x] Monitor com alertas e métricas

[x] Segurança via RBAC e Key Vault

## Scripts Técnicos / Technical Scripts

## MERGE SCD2 (SQL)

MERGE dim.sensor AS d
USING stg.sensor AS s
ON d.sensor_id = s.sensor_id AND d.is_current = 1
WHEN MATCHED AND (
    ISNULL(d.location,'') <> ISNULL(s.location,'') OR
    ISNULL(d.status,'')   <> ISNULL(s.status,'')
) THEN
  UPDATE SET d.valid_to = SYSUTCDATETIME(), d.is_current = 0
WHEN NOT MATCHED BY TARGET THEN
  INSERT (sensor_id, location, status, valid_from, valid_to, is_current)
  VALUES (s.sensor_id, s.location, s.status, SYSUTCDATETIME(), '9999-12-31', 1);

## Lookup Temporal para Fatos / Temporal Lookup for Facts

SELECT f.*, d.sensor_sk
FROM stg.sensor_events f
JOIN dim.sensor d
  ON f.sensor_id = d.sensor_id
 AND f.event_time >= d.valid_from
 AND f.event_time <  d.valid_to;

## Licença / License

Este projeto segue a licença MIT. Sinta-se livre para adaptar, publicar e contribuir. This project is licensed under the MIT License. Feel free to adapt, publish, and contribute.

## Créditos / Credits

Desenvolvido por Elias Zimbeti. Arquitetura inspirada em padrões industriais com foco em rastreabilidade, governança e consumo híbrido. Developed by Elias Zimbeti. Architecture inspired by industrial patterns focused on traceability, governance, and hybrid consumption.

## Idiomas / Languages

Este README está disponível em português e inglês. Traduções adicionais são bem-vindas via pull request. This README is available in Portuguese and English. Additional translations are welcome via pull request.
