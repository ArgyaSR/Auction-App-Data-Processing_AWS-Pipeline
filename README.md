# 🔨 Auction Data Pipeline

[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![Apache Kafka](https://img.shields.io/badge/Kafka-3.6-orange.svg)](https://kafka.apache.org/)
[![Apache Spark](https://img.shields.io/badge/Spark-3.5-yellow.svg)](https://spark.apache.org/)
[![AWS](https://img.shields.io/badge/AWS-Free%20Tier-orange.svg)](https://aws.amazon.com/free/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A production-grade, end-to-end data pipeline for processing real-time auction bids and batch analytics. Built with modern data engineering practices using Apache Kafka, Spark Structured Streaming, PostgreSQL, Apache Airflow, and AWS services.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Data Flow](#data-flow)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Testing](#testing)
- [Monitoring](#monitoring)
- [Contributing](#contributing)

## 🎯 Overview

This pipeline solves the data engineering challenges of a high-traffic auction platform:

- **Problem**: An auction web app experiencing growth pains—slow page loads, application downtime, and inability to process thousands of concurrent bids
- **Solution**: Event-driven architecture with Kafka for ingestion decoupling, Spark for scalable processing, and a medallion data architecture for clean analytics

### Key Capabilities

| Capability | Metric |
|------------|--------|
| Bid Processing Throughput | 10,000+ events/second |
| End-to-End Latency | < 500ms (streaming path) |
| Data Freshness | Real-time for active auctions |
| Historical Analytics | Daily batch aggregations |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATA SOURCES                                    │
├─────────────────┬─────────────────┬─────────────────┬───────────────────────┤
│   Auction API   │   IoT/POS       │   Batch Files   │   Historical Data     │
│   (Real-time)   │   Devices       │   (CSV/JSON)    │   (S3)                │
└────────┬────────┴────────┬────────┴────────┬────────┴──────────┬────────────┘
         │                 │                 │                   │
         ▼                 ▼                 ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INGESTION LAYER                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                      Apache Kafka (KRaft Mode)                       │    │
│  │  Topics: auction.bids | auction.items | auction.users | auction.txn │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
┌───────────────────────────────┐ ┌───────────────────────────────────────────┐
│      STREAM PROCESSING        │ │           BATCH PROCESSING                 │
│  ┌─────────────────────────┐  │ │  ┌─────────────────────────────────────┐  │
│  │  Spark Structured       │  │ │  │  Apache Spark (Batch Jobs)          │  │
│  │  Streaming              │  │ │  │  - Daily aggregations               │  │
│  │  - Real-time bid        │  │ │  │  - Historical analytics             │  │
│  │    validation           │  │ │  │  - ML feature engineering           │  │
│  │  - Price updates        │  │ │  └─────────────────────────────────────┘  │
│  │  - Fraud detection      │  │ │                                           │
│  └─────────────────────────┘  │ │  Orchestrated by Apache Airflow           │
└───────────────┬───────────────┘ └─────────────────────┬─────────────────────┘
                │                                       │
                ▼                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            STORAGE LAYER                                     │
│                         (Medallion Architecture)                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │   BRONZE     │───▶│   SILVER     │───▶│    GOLD      │                   │
│  │   (S3)       │    │ (PostgreSQL) │    │  (DuckDB/    │                   │
│  │  Raw Events  │    │   Cleaned    │    │   Analytics) │                   │
│  └──────────────┘    └──────────────┘    └──────────────┘                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VISUALIZATION LAYER                                 │
│  ┌─────────────────────────┐    ┌─────────────────────────────────────┐     │
│  │       Metabase          │    │      Application API                │     │
│  │   (BI Dashboards)       │    │   (Real-time auction data)          │     │
│  └─────────────────────────┘    └─────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Ingestion** | Apache Kafka (KRaft) | Event streaming, decoupling |
| **Stream Processing** | Spark Structured Streaming | Real-time transformations |
| **Batch Processing** | Apache Spark | Daily aggregations, ETL |
| **Orchestration** | Apache Airflow | Workflow scheduling |
| **Operational DB** | PostgreSQL | Application data, Silver layer |
| **Data Lake** | AWS S3 / LocalStack | Bronze layer, raw storage |
| **Analytics DB** | DuckDB | OLAP queries, Gold layer |
| **Visualization** | Metabase | Dashboards, reporting |
| **Infrastructure** | Terraform | Infrastructure as Code |
| **Containerization** | Docker Compose | Local development |

## 🚀 Quick Start

### Prerequisites

- Docker Desktop 4.0+ with WSL2 backend (Windows)
- Python 3.11+
- Make (optional, for convenience commands)
- AWS CLI (for cloud deployment)

### One-Command Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/auction-data-pipeline.git
cd auction-data-pipeline

# Copy environment template
cp .env.example .env

# Start all services
make up

# Or without Make:
docker compose up -d
```

### Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| Kafka UI | http://localhost:8080 | - |
| Airflow | http://localhost:8081 | airflow / airflow |
| Metabase | http://localhost:3000 | Setup on first visit |
| PostgreSQL | localhost:5432 | auction / auction123 |

### Generate Sample Data

```bash
# Start the data generator (produces to Kafka)
make generate-data

# Or run directly:
python -m src.data_generator.kafka_producer --events 10000 --rate 100
```

### Run the Pipeline

```bash
# Start Spark streaming job
make run-streaming

# Trigger batch processing via Airflow UI or:
make run-batch
```

## 📁 Project Structure

```
auction-data-pipeline/
├── README.md                       # This file
├── docker-compose.yml              # All services orchestration
├── Makefile                        # Convenience commands
├── .env.example                    # Environment template
├── requirements.txt                # Python dependencies
├── pyproject.toml                  # Project metadata
│
├── src/
│   ├── __init__.py
│   ├── data_generator/             # Synthetic data generation
│   │   ├── __init__.py
│   │   ├── generator.py            # Core data generation logic
│   │   ├── schemas.py              # Data models (Pydantic)
│   │   ├── kafka_producer.py       # Stream events to Kafka
│   │   └── batch_generator.py      # Generate batch files
│   │
│   ├── ingestion/                  # Data ingestion layer
│   │   ├── __init__.py
│   │   ├── kafka_consumer.py       # Base Kafka consumer
│   │   └── s3_ingestion.py         # S3 batch ingestion
│   │
│   ├── transformation/             # Processing layer
│   │   ├── __init__.py
│   │   ├── spark_streaming.py      # Spark Structured Streaming
│   │   ├── batch_processing.py     # Spark batch jobs
│   │   └── transformations.py      # Business logic transforms
│   │
│   └── serving/                    # Data serving layer
│       ├── __init__.py
│       ├── postgres_loader.py      # Load to PostgreSQL
│       └── duckdb_analytics.py     # DuckDB analytics queries
│
├── dags/                           # Airflow DAGs
│   ├── __init__.py
│   ├── auction_streaming_dag.py    # Streaming job management
│   └── batch_processing_dag.py     # Daily batch ETL
│
├── sql/                            # Database schemas
│   ├── init.sql                    # Initial setup
│   ├── bronze_schema.sql           # Raw data schema
│   ├── silver_schema.sql           # Cleaned data schema
│   └── gold_schema.sql             # Analytics schema
│
├── terraform/                      # Infrastructure as Code
│   ├── main.tf                     # Main configuration
│   ├── variables.tf                # Input variables
│   ├── outputs.tf                  # Output values
│   ├── s3.tf                       # S3 bucket config
│   ├── rds.tf                      # RDS PostgreSQL config
│   └── iam.tf                      # IAM roles and policies
│
├── tests/                          # Test suite
│   ├── __init__.py
│   ├── conftest.py                 # Pytest fixtures
│   ├── test_generator.py           # Data generator tests
│   ├── test_transformations.py     # Transform logic tests
│   └── integration/                # Integration tests
│       └── test_pipeline.py
│
├── config/                         # Configuration files
│   ├── spark-defaults.conf         # Spark configuration
│   └── log4j2.properties           # Logging config
│
├── docs/                           # Documentation
│   ├── ARCHITECTURE.md             # Detailed architecture
│   ├── SETUP.md                    # Setup instructions
│   └── adr/                        # Architecture Decision Records
│       ├── 001-kafka-over-kinesis.md
│       └── 002-spark-over-flink.md
│
└── .github/
    └── workflows/
        └── ci.yml                  # GitHub Actions CI/CD
```

## 🌊 Data Flow

### Real-Time Path (Streaming)

1. **Bid Placed** → Auction API publishes event to Kafka `auction.bids` topic
2. **Spark Streaming** consumes events, validates bids, updates current prices
3. **PostgreSQL** receives validated bid records (Silver layer)
4. **Application** queries PostgreSQL for current auction state

### Batch Path (Analytics)

1. **Airflow** triggers daily at 2 AM UTC
2. **Spark Batch** reads from Bronze (S3), applies transformations
3. **Gold Layer** aggregations written to DuckDB/PostgreSQL
4. **Metabase** dashboards refresh with new data

### Data Schema Overview

```
Users (1) ─────────< Bids (∞)
  │                    │
  │                    │
  └──< Items (∞) >─────┘
         │
         │
    Transactions (∞)
```

## ⚙️ Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `KAFKA_BOOTSTRAP_SERVERS` | Kafka broker addresses | `localhost:9092` |
| `POSTGRES_HOST` | PostgreSQL host | `localhost` |
| `POSTGRES_DB` | Database name | `auction` |
| `AWS_REGION` | AWS region for S3 | `us-east-1` |
| `S3_BUCKET` | Data lake bucket name | `auction-data-lake` |

### Spark Tuning

Edit `config/spark-defaults.conf` for production workloads:

```properties
spark.executor.memory=2g
spark.executor.cores=2
spark.sql.shuffle.partitions=200
```

## ☁️ Deployment

### AWS Free Tier Deployment

```bash
cd terraform

# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Deploy infrastructure
terraform apply

# Get outputs (RDS endpoint, S3 bucket name)
terraform output
```

### Cost Estimation (Free Tier)

| Service | Monthly Cost |
|---------|--------------|
| S3 (5GB) | $0 |
| RDS db.t3.micro | $0 |
| Lambda (if used) | $0 |
| **Total** | **$0-5** |

## 🧪 Testing

```bash
# Run all tests
make test

# Run with coverage
make test-coverage

# Run specific test file
pytest tests/test_generator.py -v

# Run integration tests (requires Docker)
make test-integration
```

## 📊 Monitoring

### Kafka Metrics (via Kafka UI)

- Consumer lag per partition
- Message throughput
- Topic sizes

### Spark Metrics

Access Spark UI at `http://localhost:4040` during job execution:
- Stage progress
- Memory usage
- Shuffle statistics

### Custom Dashboards (Metabase)

Pre-configured dashboards include:
- Real-time bidding activity
- Daily revenue trends
- User engagement metrics
- Auction success rates

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Apache Kafka](https://kafka.apache.org/) for the streaming platform
- [Apache Spark](https://spark.apache.org/) for data processing
- [Confluent](https://developer.confluent.io/) for Kafka documentation
- [DuckDB](https://duckdb.org/) for embedded analytics

---

**Built with ❤️ for learning modern data engineering**
