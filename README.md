### README.md

# RAG Chatbot Deployment Showcase

## Project Description
This project demonstrates how to deploy a Retrieval-Augmented Generation (RAG) chatbot to AWS using various services and tools like EC2, Lambda, S3, API Gateway, Docker, and Terraform. The chatbot uses advanced language models for generating responses based on uploaded documents.

## Technologies Used
- AWS (EC2, Lambda, S3, API Gateway)
- Docker
- Streamlit
- Python
- LangChain
- Hugging Face Transformers
- FAISS
- Terraform
- GitHub Actions

## Getting Started

### Prerequisites
- AWS Account
- Terraform installed
- Docker installed
- GitHub Account
- Hugging Face API Token

### Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/aws-rag-chatbot-deployment-showcase.git
   cd aws-rag-chatbot-deployment-showcase
   ```

2. **Set up AWS credentials:**
   Ensure your AWS credentials are configured. You can do this by setting up the `~/.aws/credentials` file or using environment variables.

3. **Deploy the infrastructure using Terraform:**
   ```bash
   cd infrastructure
   terraform init
   terraform apply
   ```

4. **Build and push Docker image:**
   ```bash
   cd chatbot
   docker build -t your-dockerhub-username/chatbot:latest .
   docker push your-dockerhub-username/chatbot:latest
   ```

5. **Deploy the chatbot using GitHub Actions:**
   - Ensure you have set up secrets in your GitHub repository (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `HUGGINGFACE_API_TOKEN`).
   - Push changes to the repository to trigger the CI/CD pipeline.

### Usage
- Access the chatbot via the provided API Gateway endpoint.
- (Optional) Open the web interface (if implemented) to interact with the chatbot.

## Project Structure
```
aws-rag-chatbot-deployment-showcase/
├── chatbot/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── ...
├── infrastructure/
│   ├── main.tf
│   ├── variables.tf
│   └── ...
├── .github/
│   └── workflows/
│       ├── build.yml
│       └── deploy.yml
├── frontend/ (optional)
│   ├── index.html
│   └── ...
├── README.md
└── ...
```

## Documentation

### Chatbot Development
The chatbot is developed using Streamlit and LangChain, leveraging Hugging Face models for conversational capabilities. It supports document upload (PDF and TXT) and processes them to generate context-aware responses.

### Dockerization
The application is containerized using Docker. The `Dockerfile` includes the necessary instructions to build the Docker image.

### AWS Infrastructure Setup
Terraform scripts are used to provision AWS resources like EC2, Lambda, S3, and API Gateway. These scripts ensure the proper configuration and security settings for each service.

### CI/CD with GitHub Actions
GitHub Actions workflows are set up for continuous integration and continuous deployment. The workflows handle building the Docker image, running tests, and deploying the application to AWS.

### Streamlit Application
The Streamlit app allows users to upload documents, select a language model, and interact with the chatbot. It also provides feedback collection and answer generation features with performance evaluation.
```
