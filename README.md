[![Contributors][contributors-shield]][contributors-url]
[![Issues][issues-shield]][issues-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]

# InsightStream: A Cloud-Native Real-Time RAG System for News-Augmented LLMs

**Introduction**

InsightStream is a cloud-native Retrieval-Augmented Generation (RAG) system developed as part of an undergraduate thesis in the Computer Engineering and Software Systems (CESS) program at Ain Shams University (2024–2025). It enhances the capabilities of Large Language Models (LLMs) by incorporating real-time news, addressing the limitations of static training datasets in time-sensitive domains such as journalism, analytics, and decision-making.

This implementation leverages modern AWS-managed services, infrastructure-as-code principles, and distributed microservices to deliver scalable, maintainable, and secure information retrieval augmented with the most current data.

---

## 📋 Abstract

InsightStream presents a cloud-based, real-time RAG architecture that combines semantic search with contextual generation to deliver accurate, timely, and grounded responses. It achieves this through an event-driven pipeline that continuously ingests, processes, and indexes news articles from various sources.

**Key Contributions:**

* **Cloud-Native Architecture**: Modular design utilizing AWS EC2, Lambda, S3, Kinesis, API Gateway, Cognito, and CloudFront.
* **Real-Time Data Pipeline**: Asynchronous ETL pipeline powered by Kinesis and EventBridge.
* **Semantic Vector Storage**: Integration with Qdrant for high-performance similarity search.
* **LLM Integration**: DeepSeek API integration for low-latency, accurate generation.
* **Infrastructure as Code**: Fully Terraform-managed deployments with isolated environments.

---

## 🏗️ Architecture Overview

The system employs a modular microservices architecture deployed entirely on AWS using Terraform. It follows best practices for scalability, security, and cost optimization.

### Components

* **Frontend**: React + TypeScript app hosted on S3 and distributed via CloudFront.
* **Authentication**: AWS Cognito with JWT-based OAuth 2.0 authentication.
* **API Gateway**: HTTP API Gateway for secure, authenticated routing.
* **Microservices**: Deployed on EC2 using custom AMIs (g4dn.xlarge for GPU tasks, t3.medium for preprocessing).
* **Vector Database**: Qdrant Docker container on EC2 for low-latency semantic search within VPC.
* **Storage**: Multi-tier S3 buckets for raw, processed, and archived data.
* **ETL Pipeline**: Kinesis streams and EventBridge Scheduler.
* **Monitoring**: AWS CloudWatch and Parameter Store for observability and secure config.

### System Flow

1. **News Ingestion**: REST API pulls from newsdata.io; raw data saved to S3.
2. **Preprocessing**: spaCy-based NLP pipeline; data cleaned and normalized.
3. **Semantic Chunking**: Using LangChain's `SemanticChunker`.
4. **Embedding Generation**: intfloat/e5-base-v2 model via SentenceTransformers.
5. **Vector Storage**: Embeddings stored in Qdrant.
6. **User Query Processing**: JWT-authenticated queries routed through API Gateway and Lambda.
7. **Query Rewriting**: Raw input is processed through the QueryRewriter module (see `deepseek_llm.py`), which applies rule-based normalization (lowercasing, punctuation stripping) and thesaurus lookup (e.g., "sell"→"sale") to ensure consistency between user queries and metadata filters.
8. **Document Retrieval**: Cosine similarity used to retrieve top-k semantically similar documents.
9. **Response Generation**: DeepSeek generates contextual responses.

---

## 🔍 Query Rewrite System

InsightStream includes an intelligent query rewriting system that enhances user queries before retrieval to improve search accuracy and relevance.

### Query Processing Pipeline

The QueryRewriter module (implemented in `deepseek_llm.py`) performs several transformations:

1. **Rule-based Normalization**: 
   - Lowercasing for consistent matching
   - Punctuation stripping and standardization
   - Whitespace normalization

2. **Thesaurus Lookup**: 
   - Semantic expansion of terms (e.g., "sell" → "sale")
   - Context-aware synonym replacement
   - Domain-specific terminology mapping

3. **Query Enhancement**:
   - Converts brief queries into more descriptive questions
   - Adds context and specificity for better retrieval

### Example Query Transformation

```
Original Query: "Trump's Tariffs"
Rewritten Query: "What are the effects and implications of Trump's tariffs on trade and economy"
```

This transformation helps the system understand user intent and retrieve more relevant news articles by expanding the query scope and adding contextual keywords.

---

## 🎥 Demo Video

See InsightStream in action! This demo showcases the complete workflow from user query to AI-generated response with real-time news context.


https://github.com/user-attachments/assets/5c5885b8-a826-4e1b-bf55-33740c67629e


### What the Demo Shows:

* **User Authentication**: Login process using AWS Cognito
* **Real-Time Query Processing**: Live demonstration of news-augmented responses
* **Source Attribution**: Transparent display of retrieved news articles
* **Chat History Management**: Creating and managing conversation threads
* **System Performance**: Response times and accuracy in real-world scenarios

*Demo Duration: ~3 minutes | Covers end-to-end user experience*

---

## 📁 Project Structure

```
.
├── backend/                        # Core backend services and logic
│   ├── data_extraction/            # News data fetching from APIs
│   │   └── api_fetcher.py          # Main API client for newsdata.io
│   ├── data_preprocessing/         # Data cleaning and transformation
│   │   ├── preprocess.py           # Main preprocessing pipeline
│   │   ├── information_extraction.py  # NLP and entity extraction
│   │   ├── information_extraction.ipynb  # Jupyter notebook for analysis
│   │   └── InsightStream_Preprocessing.ipynb  # Complete preprocessing workflow
│   ├── embedding_service/          # Vector embedding generation and storage
│   │   ├── embedder.py            # Text embedding using SentenceTransformers
│   │   ├── qdrant_store.py        # Qdrant vector database operations
│   │   └── qdrant_migration.py    # Data migration between Qdrant instances
│   ├── lambda_code/               # AWS Lambda functions
│   │   ├── index.js               # Main Lambda handler for API Gateway
│   │   └── package.json           # Node.js dependencies
│   ├── llm/                       # Large Language Model integration
│   │   ├── deepseek_llm.py        # DeepSeek API client and query rewriting
│   │   └── prompt_engineering.py  # Prompt templates and optimization
│   ├── rag_app/                   # RAG application core
│   │   ├── rag.py                 # Main RAG logic and orchestration
│   │   ├── rag_server.py          # Flask/FastAPI server for RAG endpoints
│   │   └── run_rag_app.py         # Application entry point
│   ├── retrieval/                 # Document retrieval and search
│   │   └── retriever.py           # Semantic search and similarity matching
│   └── tests/                     # Unit and integration tests
│       ├── test_embedder.py       # Embedding service tests
│       ├── test_preprocess.py     # Preprocessing pipeline tests
│       └── test_vectorDB.py       # Vector database tests
├── infrastructure/                # Infrastructure as Code (Terraform)
│   ├── envs/                      # Environment-specific configurations
│   │   ├── dev/                   # Development environment
│   │   │   ├── main.tf            # Dev infrastructure definition
│   │   │   ├── dev.tfvars         # Dev environment variables
│   │   │   └── variables.tf       # Variable definitions
│   │   └── prod/                  # Production environment
│   │       ├── main.tf            # Prod infrastructure definition
│   │       ├── prod.tfvars        # Prod environment variables
│   │       └── variables.tf       # Variable definitions
│   └── modules/                   # Reusable Terraform modules
│       ├── api_gateway/           # AWS API Gateway configuration
│       ├── cognito/               # AWS Cognito authentication setup
│       ├── dynamodb/              # DynamoDB tables for chat history
│       ├── ec2/                   # EC2 instances for RAG and Qdrant
│       ├── frontend-hosting/      # S3 + CloudFront for frontend
│       ├── eventbridge-scheduler/ # Automated scheduling for data ingestion
│       ├── iam/                   # IAM roles and policies
│       ├── kinesis/               # Kinesis streams for data pipeline
│       ├── lambda/                # Lambda function deployment
│       └── vpc/                   # Virtual Private Cloud setup
├── frontend/                      # React + TypeScript web application
│   ├── src/                       # Source code
│   │   ├── Components/            # Reusable React components
│   │   ├── Context/               # React context providers
│   │   ├── pages/                 # Application pages/routes
│   │   ├── services/              # API service layer
│   │   └── types/                 # TypeScript type definitions
│   ├── public/                    # Static assets
│   │   └── assets/                # Images, icons, logos
│   ├── compose.yaml               # Docker Compose configuration
│   ├── Dockerfile                 # Container build instructions
│   ├── package.json               # Node.js dependencies and scripts
│   └── vite.config.ts             # Vite bundler configuration
├── scripts/                       # Deployment and utility scripts
│   ├── deploy_frontend.ps1        # PowerShell frontend deployment
│   ├── setup_env.sh               # Environment setup script
│   ├── user_data.sh               # EC2 instance initialization
│   └── qd_user_data.sh            # Qdrant instance setup
└── docs/                          # Documentation and research materials
    └── Final Thesis.pdf           # Complete thesis document
```

---

## 🚀 Local Installation

If you want to run InsightStream locally for development or testing purposes, follow these steps:

### 1. Clone the Repository

```bash
git clone https://github.com/Tamerihab/InsightStream.git
cd InsightStream
```

### 2. Create a Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Make sure your `requirements.txt` includes:

```txt
streamlit>=1.28.0
sentence-transformers>=2.2.0
qdrant-client>=1.6.0
python-dotenv>=1.0.0
pandas>=1.5.0
requests>=2.28.0
spacy>=3.4.0
```

**Additional Setup for spaCy:**
```bash
python -m spacy download en_core_web_sm
```

### 4. Set Up API Keys

Create a `.env` file in the project root with the following variables:

```env
QDRANT_API_URL=<your_qdrant_cloud_url>
QDRANT_API_KEY=<your_qdrant_api_key>
DEEPSEEK_API_KEY=<your_deepseek_api_key>
NEWS_API_KEY=<your_news_api_key>
```

**Note**: The DeepSeek API key is required for LLM responses and query rewriting functionality. The News API key is used for data ingestion (if running the full pipeline).

### 5. Usage

#### Start the Application

Navigate to the RAG application directory and run the Streamlit app:

```bash
cd backend/rag_app
streamlit run rag.py
```

#### System Features

The local RAG application provides the following features:

- **Interactive Chat Interface**: Ask questions through a user-friendly chat interface
- **Real-time Document Retrieval**: Search through your document collection using semantic similarity
- **Source Attribution**: View the sources used to generate each response with clickable references
- **Configurable Search Parameters**: Adjust the number of documents retrieved (top_k) and similarity threshold
- **Date Filtering**: Filter documents by date range for time-sensitive queries
- **Chat Management**: Clear chat history, export conversations, and view analytics
- **Smart Query Processing**: Handles small talk and identity questions appropriately

#### Using the Interface

1. **Ask Questions**: Enter your query in the chat input at the bottom
2. **Adjust Settings**: Use the sidebar to:
   - Change the number of documents retrieved (1-20)
   - Adjust minimum similarity threshold (0.0-1.0)
   - Enable date filtering for time-sensitive searches
3. **View Sources**: Click "View Sources" buttons to see the documents used for each response
4. **Manage Chat**: Use sidebar options to clear chat or export conversation history
5. **Quick Start**: Try the suggested sample questions for new conversations

#### Chat Features

- **Smart Response Handling**: The system intelligently handles different types of queries:
  - Regular questions about your documents
  - Small talk and greetings
  - Identity questions about the AI assistant
- **Source References**: Responses include numbered references [1], [2] that link to source documents
- **Chat Analytics**: View statistics about your conversation history
- **Export Capability**: Download your chat history as a text file

### Local Requirements

- **Python**: 3.8+
- **Streamlit**: Web framework for the user interface
- **DeepSeek API**: For LLM-powered responses and query rewriting
- **SentenceTransformers**: For embedding model (intfloat/e5-base-v2)
- **Qdrant**: Vector database for semantic search (cloud or local instance)
- **spaCy**: For natural language processing
- **pandas**: For data manipulation
- **python-dotenv**: For environment variable management

### Local Troubleshooting

#### Common Issues

1. **Qdrant Connection Errors**:
   - Ensure the correct `QDRANT_API_URL` and `QDRANT_API_KEY` are set in `.env`
   - Verify your Qdrant instance is running and accessible

2. **DeepSeek API Errors**:
   - Check that `DEEPSEEK_API_KEY` is properly set in your `.env` file
   - Verify your API key has sufficient credits

3. **Missing Dependencies**:
   - Install all required packages: `pip install -r requirements.txt`
   - Ensure you have the correct versions of streamlit, qdrant-client, and sentence-transformers

4. **Streamlit Not Launching**:
   - Verify you're in the correct directory: `backend/rag_app`
   - Check if port 8501 is available or specify a different port: `streamlit run rag.py --server.port 8502`

5. **Empty or No Search Results**:
   - Verify your Qdrant collection has data: check collection status in Qdrant dashboard
   - Try lowering the similarity threshold in the sidebar
   - Ensure your embedding model matches the one used to create the vector database

6. **Slow Response Times**:
   - Check your internet connection for API calls to DeepSeek
   - Consider using a local Qdrant instance instead of cloud for faster retrieval
   - Reduce the number of documents retrieved (top_k) in settings

---

## ☁️ Cloud Deployment Guide

### Requirements

* AWS CLI (v2+)
* Terraform (v1.0+)
* Node.js (v18+)
* Python (v3.9+)
* Docker (optional)

### Steps

```bash
git clone https://github.com/Tamerihab/InsightStream.git
cd InsightStream
```

**Important**: The provided Terraform configuration uses custom Amazon Machine Images (AMIs) pre-configured with necessary dependencies. To proceed, you must either:

1. **Create your own custom AMIs** that include all required software and packages.
2. **Modify the `main.tf` files** to use standard public AWS AMIs (e.g., Amazon Linux 2) and install dependencies at launch via user data scripts.  
   ⚠️ *Note: This is not recommended for production deployments, as some packages are large and may cause instance setup timeouts or partial configurations during initialization.*

Update `prod.tfvars`:

```hcl
api_key           = "<newsdata.io key>"
qdrant_api_key    = "<qdrant api key>"
qdrant_api_url    = "<qdrant api url>"
deepseek_api      = "<DeepSeek key>"
```

Provision Infrastructure:

```bash
cd infrastructure/envs/prod
terraform init
terraform apply -var-file="prod.tfvars"
```

Deploy Frontend:

```powershell
cd scripts
./deploy_frontend.ps1
```

---

## 🔐 Security Highlights

* **Authentication**: OAuth 2.0 + JWT with AWS Cognito.
* **Network Isolation**: Private subnets for EC2, VPC Endpoints for S3/Kinesis.
* **Encryption**: S3 server-side encryption.
* **IAM**: Fine-grained permissions per service.

---

## 📊 Performance & Evaluation

### Query Benchmarks

* **Avg Response Time**: ~7 seconds


### Resource Utilization

* **Vector Database**: 70-100ms query latency with internal Qdrant on EC2
* **Cost**: \~\$0.0015/query | \$450/month for 10K queries/day
* **Optimization**: 97% cost savings running RAG model on EC2 instance instead of SageMaker

---

## 💡 API Usage

The system provides several endpoints for different functionalities:

### RAG Query Endpoint

```http
POST https://<api-gateway-url>/query
Authorization: Bearer <jwt>
Content-Type: application/json
{
  "query": "What happened in Gaza today?",
  "conversation_id": "conv_1234567890_abc123def"
}
```

### Chat History Management

**Create New Conversation:**
```http
POST https://<api-gateway-url>/chat-history/conversations
Authorization: Bearer <jwt>
Content-Type: application/json
{
  "title": "Gaza Conflict Discussion"
}
```

**Get User Conversations:**
```http
GET https://<api-gateway-url>/chat-history/conversations
Authorization: Bearer <jwt>
```

**Get Conversation Messages:**
```http
GET https://<api-gateway-url>/chat-history/messages?conversation_id=conv_1234567890_abc123def
Authorization: Bearer <jwt>
```

**Delete Conversation:**
```http
DELETE https://<api-gateway-url>/chat-history/conversations/{conversation_id}
Authorization: Bearer <jwt>
```

### System Health & Status

**Health Check:**
```http
GET https://<api-gateway-url>/health
Authorization: Bearer <jwt>
```

**System Status:**
```http
GET https://<api-gateway-url>/status
Authorization: Bearer <jwt>
```

### Response

```json
{
  "answer": "Based on recent reports, the Israeli army conducted several operations in Gaza today targeting Hamas infrastructure. The situation remains volatile with ongoing tensions between both sides...",
  "sources": [
    {
      "content": "Full text content of the relevant news article...",
      "metadata": {
        "Title": "Gaza Conflict Update: Latest Developments",
        "Published Date": "2025-01-15 10:30:00",
        "Link": "https://example.com/news/gaza-update",
        "Entities": ["Gaza", "Hamas", "Israel", "Military"]
      },
      "score": 0.892
    },
    {
      "content": "Additional relevant article content...",
      "metadata": {
        "Title": "Middle East Tensions Escalate",
        "Published Date": "2025-01-15 08:45:00",
        "Link": "https://example.com/news/middle-east-tensions",
        "Entities": ["Middle East", "Conflict", "Gaza Strip"]
      },
      "score": 0.785
    }
  ],
  "status": "success",
  "metadata": {
    "processing_time": 2.34,
    "total_time": 2.67,
    "num_sources": 2,
    "query_type": "rag_search",
    "request_id": "req_1234567890abcdef",
    "thread_id": 140234567890,
    "active_requests_count": 3
  }
}
```

## 🚧 Troubleshooting

| Issue              | Solution                                   |
| ------------------ | ------------------------------------------ |
| Auth Failure       | Check Cognito user pool and JWT expiration |
| Timeout            | Inspect EC2 instance status in CloudWatch  |
| Missing Config     | Validate SSM Parameter Store values        |
| Qdrant Connection  | Verify Docker container status on EC2      |
| AMI Not Found      | Update AMI IDs in Terraform or use standard AMIs |
| Service Not Starting | Check systemd logs: `journalctl -u <service-name>` |
| Terraform Failures | Ensure valid `.tfvars` and AWS credentials |

Logs are available via:

* `CloudWatch > Log Groups > /aws/lambda/*`
* `API Gateway > Stage Logs`
* `EC2 > Systems Manager > Session Logs`
* `EC2 systemd logs > journalctl -u <service-name>`

---

## 📆 Timeline & Cost Summary

* **March–June 2025 Total Cost**: \$418.81

  * NewsData API: \$300
  * AWS: \$118.81


## 🗄️ Vector Database Architecture

InsightStream uses a **dual Qdrant setup** to optimize both cost and performance:

### Architecture Details

* **Production Qdrant**: Docker container running on EC2 instance within private VPC subnet
* **External Qdrant**: Cloud-hosted Qdrant service used for initial data storage and development
* **Data Migration**: Automated Python script migrates embeddings from external to internal Qdrant

### Benefits

* **Reduced Latency**: Internal VPC deployment eliminates internet routing overhead
* **Cost Optimization**: Avoid per-query charges from external Qdrant service
* **Network Security**: Vector database isolated within private subnet
* **Performance**: Direct EC2-to-EC2 communication for faster retrieval

### Important Note

The codebase includes migration scripts (`qdrant_migration.py`) that handle data transfer from external to internal Qdrant instances. **These scripts are system-specific and should be ignored by end users** - they are included for completeness but require specific SSM parameters and network configurations that won't apply to other deployments.

For new deployments, simply provision a fresh Qdrant Docker container and populate it directly with your embeddings.

---

## 🖥️ Custom AMI Infrastructure

InsightStream utilizes **custom Amazon Machine Images (AMIs)** for rapid, consistent deployments across all EC2 instances:

### AMI Details

* **Base Image**: Deep Learning OSS Nvidia Driver AMI GPU PyTorch 2.7 (Amazon Linux 2023)
* **Custom Configurations**: Pre-installed dependencies, Python environments, and application scripts
* **Script Location**: All deployment scripts located at `/home/ec2-user/scripts/`
* **Auto-Start Services**: RAG server configured with systemd for automatic startup on instance launch

### Deployment Benefits

* **Zero-Configuration Deployment**: Instances are production-ready upon launch
* **Consistent Environment**: Eliminates "works on my machine" issues
* **Faster Scaling**: No need for runtime dependency installation
* **GPU Optimization**: Pre-configured CUDA drivers and PyTorch optimizations

### Service Management

The RAG EC2 instance includes a systemd service file that automatically starts the RAG server when the instance boots, ensuring high availability and seamless deployments.

**Note**: The custom AMIs referenced in `main.tf` are environment-specific. For new deployments, you'll need to create your own AMIs or modify the Terraform configuration to use standard AWS AMIs with appropriate user data scripts.

---

## 🎓 Academic Context

This system was developed as part of a Bachelor of Computer Engineering and Software Systems thesis at **Ain Shams University** (2024–2025).

* **Advising Professor**: Dr. Mahmoud Mounir Mahmoud

### Research Goals

* Design scalable architectures for RAG.
* Measure real-time semantic retrieval and generation accuracy.
* Develop a reusable Infrastructure-as-Code framework.


### Future Work

* Advanced Retrieval Mechanisms
* Enhanced Query Processing
* Improved Observability
* Advanced Chat Features

---

> **Disclaimer**: This system is intended for educational and research purposes. Production-grade deployments should include additional compliance, security hardening, and CI/CD measures.
---

[contributors-shield]: https://img.shields.io/github/contributors/Tamerihab/InsightStream.svg?style=for-the-badge
[contributors-url]: https://github.com/Tamerihab/InsightStream/graphs/contributors

[issues-shield]: https://img.shields.io/github/issues/Tamerihab/InsightStream.svg?style=for-the-badge
[issues-url]: https://github.com/Tamerihab/InsightStream/issues

[forks-shield]: https://img.shields.io/github/forks/Tamerihab/InsightStream.svg?style=for-the-badge
[forks-url]: https://github.com/Tamerihab/InsightStream/fork

[stars-shield]: https://img.shields.io/github/stars/Tamerihab/InsightStream.svg?style=for-the-badge
[stars-url]: https://github.com/Tamerihab/InsightStream/stargazers
