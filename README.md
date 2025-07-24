# 🤖 Chatbot using n8n + LangChain + Google Gemini + Pinecone + Google Drive (RAG Workflow)

This project implements an intelligent, document-aware chatbot using **n8n**, **LangChain**, **Google Gemini**, **Pinecone**, and **Google Drive**. It integrates a Retrieval-Augmented Generation (RAG) pipeline to provide contextual and accurate answers based on dynamic documents uploaded to Google Drive.

## 📌 Key Features

* 📂 Automatically ingests new files from a specific Google Drive folder
* 🔍 Splits and embeds documents using Google Gemini embeddings
* 🧠 Stores vectors in Pinecone for fast semantic retrieval
* 💬 Responds to user chat messages using LangChain Agent + Gemini Chat
* 🧾 Follows a strict system prompt for reliable, grounded, and friendly support answers

## 🛠️ Prerequisites

Before setting up the workflow, make sure you have:

* ✅ n8n self-hosted or cloud instance
* ✅ Pinecone account & vector index setup
* ✅ Google Cloud Project with:
   * Google Drive API enabled
   * PaLM/Gemini API access enabled
* ✅ Google Service Account OAuth2 credentials added to n8n
* ✅ Your documents uploaded to a specific Google Drive folder

## 📁 Project Structure

```sql
chatbot-main/
│
├── README.md         ← You are here
├── workflow.json     ← Exported n8n workflow file (like the one shared above)
```

## ⚙️ Step-by-Step Workflow Setup

### 1. **Trigger: Google Drive File Upload**
* **Node**: `Google Drive Trigger`
* **Purpose**: Triggers when a new file is uploaded to a specific Google Drive folder.
* **Settings**:
   * Folder ID: `1R2MUhV6yIxULqK39B42dGAVVdht7IpOp`
   * Event: `fileCreated`

### 2. **Download the Uploaded File**
* **Node**: `Download file`
* **Purpose**: Downloads the new file using its `fileId`.

### 3. **Document Processing and Embedding Pipeline**

#### a. **Recursive Text Splitter**
* **Node**: `Recursive Character Text Splitter`
* **Purpose**: Prepares large documents for embedding by splitting them into manageable chunks.

#### b. **Default Data Loader**
* **Node**: `Default Data Loader`
* **Purpose**: Loads and processes binary document data.

#### c. **Google Gemini Embedding**
* **Node**: `Embeddings Google Gemini`
* **Model**: `models/embedding-001`
* **Purpose**: Converts text chunks into embeddings for semantic search.

#### d. **Pinecone Vector Store**
* **Node**: `Pinecone Vector Store`
* **Purpose**: Stores embeddings into Pinecone under namespace `siddhi-info`.

### 4. **LangChain AI Agent (Chatbot Brain)**

#### a. **Chat Trigger**
* **Node**: `When chat message received`
* **Purpose**: Listens for incoming chat messages via webhook.

#### b. **Google Gemini Chat Model**
* **Node**: `Google Gemini Chat Model`
* **Model**: `models/gemini-1.5-flash`
* **Purpose**: Handles chat completions using Gemini LLM.

#### c. **Simple Memory**
* **Node**: `Simple Memory`
* **Purpose**: Provides conversational context/memory for the agent.

#### d. **LangChain AI Agent**
* **Node**: `AI Agent`
* **Purpose**: Acts as the central LangChain agent to handle message logic.
* **Prompt Design**: Strict prompt enforcing no hallucination, empathetic tone, and policy-based factual responses.

### 5. **Retrieval-Augmented Tool: Answering with Vector Store**

#### a. **Vector Store 1**
* **Node**: `Pinecone Vector Store1`
* **Purpose**: Retrieves most relevant documents from Pinecone during chat.

#### b. **Google Gemini Embedding (Query)**
* **Node**: `Embeddings Google Gemini1`
* **Purpose**: Embeds the user's query.

#### c. **Google Gemini Chat Model 2**
* **Node**: `Google Gemini Chat Model1`
* **Model**: `models/gemini-2.5-flash`
* **Purpose**: Uses retrieved knowledge to generate response.

#### d. **Tool Integration**
* **Node**: `Answer questions with a vector store`
* **Purpose**: Combines vector search + chat model into a callable tool for the agent.

## 🚀 How to Deploy

1. **Clone this repository**
2. **Import the workflow into your n8n instance**
3. **Set up credentials**:
   * `Google Drive OAuth2`
   * `Pinecone API`
   * `Google Gemini (PaLM) API`
4. **Activate the workflow**
5. **Upload documents to Google Drive folder**
6. **Send messages to the webhook endpoint** (chatbot will respond based on retrieved data)

## Required Services & Credentials

### 1. Google Cloud Platform Setup

#### Enable APIs
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing one
3. Enable the following APIs:
   - Google Drive API
   - Google AI (Gemini) API

#### Create Service Account
1. Navigate to **IAM & Admin** → **Service Accounts**
2. Click **Create Service Account**
3. Add these roles:
   - Google Drive API access
   - AI Platform access
4. Generate and download JSON key file

#### Get API Keys
1. Go to **APIs & Services** → **Credentials**
2. Create **API Key** for Gemini AI
3. Restrict the key to necessary APIs only

### 2. Pinecone Setup

1. Sign up at [Pinecone](https://www.pinecone.io)
2. Create a new project
3. Create an index with these settings:
   - **Index Name**: `s-index` (or your preferred name)
   - **Dimensions**: `768` (for Google's embedding-001 model)
   - **Metric**: `cosine`
   - **Pod Type**: `p1.x1` (or suitable for your needs)
4. Note down your API key and environment

### 3. Google Drive Folder Setup

1. Create a dedicated folder in Google Drive for your knowledge base
2. Share the folder with your service account email
3. Copy the folder ID from the URL (the string after `/folders/`)

## n8n Workflow Configuration

### Import the Workflow

1. Copy the workflow JSON from this repository
2. In n8n, go to **Workflows** → **Import from JSON**
3. Paste the JSON and import

### Configure Credentials

#### Google Drive OAuth2 API
1. Go to **Settings** → **Credentials**
2. Add **Google Drive OAuth2 API** credential
3. Use your Google Cloud project credentials
4. Authorize access to Google Drive

#### Google Gemini (PaLM) API
1. Add **Google Gemini (PaLM) API** credential
2. Enter your Google AI API key

#### Pinecone API
1. Add **Pinecone API** credential
2. Enter your Pinecone API key and environment

### Configure Workflow Nodes

#### 1. Google Drive Trigger
- **Folder to Watch**: Set to your Google Drive folder ID
- **Event**: File Created
- **Poll Interval**: Every minute (adjust as needed)

#### 2. Pinecone Vector Store (Insert Mode)
- **Index**: Select your Pinecone index
- **Namespace**: `siddhi-info` (or your preferred namespace)
- **Mode**: Insert

#### 3. Pinecone Vector Store (Query Mode)
- **Index**: Same as above
- **Namespace**: Same as above
- **Mode**: Query (default)

### AI Agent System Message

Customize the system message to match your product/service:

```
You are a professional, friendly, and helpful customer support agent for [Your Product Name]. 

When a user asks a question, your job is to:
- Understand the user's intent clearly by analyzing the question
- Retrieve relevant documents or knowledge from the RAG tool
- Generate a concise, clear, and friendly response based only on retrieved information
- If the answer is not available, admit it honestly and offer to escalate

Always maintain a polite and empathetic tone. Use specific information from retrieved content and avoid hallucinating answers.
```

## 🧪 Testing the Setup

### Document Ingestion Test
1. Upload a test document to your Google Drive folder
2. Watch the workflow execution in n8n
3. Verify the document is processed and stored in Pinecone

### Chat Interface Test
1. Activate the workflow
2. Use the webhook URL to send test messages
3. Verify responses are based on your uploaded documents

## ⚡ Workflow Components Explained

### Document Processing Pipeline

1. **Google Drive Trigger**: Monitors specified folder for new files
2. **Download File**: Retrieves the file content from Google Drive
3. **Default Data Loader**: Processes various file formats (PDF, DOCX, TXT, etc.)
4. **Text Splitter**: Breaks documents into manageable chunks
5. **Embeddings**: Converts text chunks into vector representations
6. **Vector Store**: Stores embeddings in Pinecone with metadata

### Chat Response Pipeline

1. **Chat Trigger**: Receives user messages via webhook
2. **AI Agent**: Orchestrates the response process
3. **Vector Store Tool**: Searches for relevant documents
4. **Language Model**: Generates responses using Google Gemini
5. **Memory**: Maintains conversation context

## ⚙️ Configuration Options

### Text Splitting Settings
- **Chunk Size**: 1000 characters (default)
- **Chunk Overlap**: 200 characters (default)
- Adjust based on your document types and requirements

### Embedding Model
- Currently using `models/embedding-001`
- Produces 768-dimensional vectors
- Ensure Pinecone index dimensions match

### Language Models
- **Chat Model**: `models/gemini-1.5-flash` (primary)
- **Vector Tool Model**: `models/gemini-2.5-flash` (for RAG queries)
- Switch models based on performance and cost requirements

## 🌐 Deployment

### Webhook Configuration
1. Activate the workflow
2. Copy the webhook URL from the chat trigger
3. Integrate with your frontend application or chat platform

### Production Considerations
- Set up proper error handling and logging
- Configure rate limiting for the webhook
- Monitor Pinecone usage and costs
- Implement user authentication if needed
- Set up backup strategies for your knowledge base

## 📊 Monitoring & Maintenance

### Regular Tasks
- Monitor workflow execution logs
- Check Pinecone index usage and performance
- Update documents in Google Drive as needed
- Review and optimize system prompts

### Troubleshooting
- Check credential expiration dates
- Verify API quotas and limits
- Monitor error logs in n8n
- Test individual nodes for issues

## 🔒 Security Best Practices

1. **API Keys**: Store securely and rotate regularly
2. **Access Control**: Limit Google Drive folder access
3. **Rate Limiting**: Implement on webhook endpoints
4. **Data Privacy**: Ensure compliance with data protection regulations
5. **Monitoring**: Set up alerts for unusual activity

## 🎯 Customization Ideas

### Enhanced Features
- Multi-language support
- Document versioning
- User feedback collection
- Analytics and reporting
- Integration with ticketing systems

### Advanced Configurations
- Multiple knowledge bases (different namespaces)
- Custom document preprocessing
- Specialized embeddings for domain-specific content
- Hybrid search (vector + keyword)

## 💰 Cost Optimization

### Google Cloud
- Monitor API usage
- Set up billing alerts
- Use appropriate model tiers

### Pinecone
- Choose suitable pod types
- Monitor vector count
- Implement data retention policies
## 📄 License

This workflow configuration is provided as-is for educational and commercial use. Ensure compliance with all service provider terms of service.

---

**Last Updated**: January 2025  
**Version**: 1.0  
**Compatibility**: n8n 1.x, Google Gemini API, Pinecone
<img width="1915" height="924" alt="Image" src="https://github.com/user-attachments/assets/8329b1b6-1091-430d-b711-6220bf294a8f" />
