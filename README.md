
# Rag-chatbot Deployment Guide

This guide provides step-by-step instructions for deploying the `Rag-chatbot` application to AWS using Amazon ECS and Fargate, automated with Apache Airflow.

## Prerequisites

1. **AWS Account**: Ensure you have an active AWS account.
2. **AWS CLI**: Install and configure the AWS CLI.
    ```sh
    pip install awscli
    aws configure
    ```
3. **Docker**: Install Docker to build and test your container locally.
4. **Apache Airflow**: Ensure you have Apache Airflow installed and running. Follow the [official installation guide](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html) if you need help setting it up.

## Step 1: Clone the Repository

Clone the GitHub repository to your local machine:
```sh
git clone https://github.com/aminzadenoori/Rag-chatbot.git
cd Rag-chatbot
```

## Step 2: Add Your ECS Task Definition

Create an `ecs-task-definition.json` file with your ECS task definition and place it in your GitHub repository (`Rag-chatbot`):

```json
{
  "family": "rag-chatbot-task",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "rag-chatbot",
      "image": "<your-account-id>.dkr.ecr.<your-region>.amazonaws.com/rag-chatbot:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8501,
          "hostPort": 8501
        }
      ]
    }
  ],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512"
}
```

## Step 3: Define the Airflow DAG

Create a new DAG file in your Airflow DAGs directory, e.g., `deploy_rag_chatbot.py`, with the following content:

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'deploy_rag_chatbot',
    default_args=default_args,
    description='A DAG to deploy Rag-chatbot to AWS',
    schedule_interval=timedelta(days=1),
    start_date=datetime(2023, 7, 21),
    catchup=False,
)

# Step 1: Clone the repository
clone_repo = BashOperator(
    task_id='clone_repo',
    bash_command='git clone https://github.com/aminzadenoori/Rag-chatbot.git /tmp/Rag-chatbot',
    dag=dag,
)

# Step 2: Build Docker image
build_docker_image = BashOperator(
    task_id='build_docker_image',
    bash_command='cd /tmp/Rag-chatbot && docker build -t rag-chatbot .',
    dag=dag,
)

# Step 3: Create ECR repository
create_ecr_repo = BashOperator(
    task_id='create_ecr_repo',
    bash_command='aws ecr create-repository --repository-name rag-chatbot || true',
    dag=dag,
)

# Step 4: Authenticate Docker to ECR
authenticate_docker = BashOperator(
    task_id='authenticate_docker',
    bash_command='$(aws ecr get-login --no-include-email --region us-east-1)',
    dag=dag,
)

# Step 5: Tag and push Docker image to ECR
push_docker_image = BashOperator(
    task_id='push_docker_image',
    bash_command=(
        'docker tag rag-chatbot:latest <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/rag-chatbot:latest && '
        'docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/rag-chatbot:latest'
    ),
    dag=dag,
)

# Step 6: Register ECS Task Definition
register_task_definition = BashOperator(
    task_id='register_task_definition',
    bash_command='aws ecs register-task-definition --cli-input-json file:///tmp/Rag-chatbot/ecs-task-definition.json',
    dag=dag,
)

# Step 7: Create ECS Service
create_ecs_service = BashOperator(
    task_id='create_ecs_service',
    bash_command=(
        'aws ecs create-service --cluster rag-chatbot-cluster --service-name rag-chatbot-service --task-definition rag-chatbot-task '
        '--desired-count 1 --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[<your-subnet-id>],securityGroups=[<your-security-group-id>],assignPublicIp=ENABLED}"'
    ),
    dag=dag,
)

clone_repo >> build_docker_image >> create_ecr_repo >> authenticate_docker >> push_docker_image >> register_task_definition >> create_ecs_service
```

## Step 4: Configure Your Airflow Environment

1. **Place the DAG**: Ensure the `deploy_rag_chatbot.py` file is placed in the Airflow DAGs directory.
2. **Set Environment Variables**: Make sure that your environment variables, such as AWS credentials and region information, are set correctly in the environment where Airflow is running.

## Step 5: Run the DAG

1. **Access the Airflow Web UI**: Open your Airflow web UI.
2. **Enable the DAG**: Enable the `deploy_rag_chatbot` DAG.
3. **Trigger the DAG**: Manually trigger the DAG to start the deployment process.

## Summary of AWS Services Used

1. **Amazon Elastic Container Registry (ECR)**: To store your Docker images.
2. **Amazon Elastic Container Service (ECS)**: To manage your Docker containers.
3. **AWS Fargate**: To run your containers without managing servers.
4. **Amazon Virtual Private Cloud (VPC)**: To define your network environment.
5. **AWS Identity and Access Management (IAM)**: To manage permissions and roles.

## Additional Steps

- **Monitoring**: Use the Airflow UI to monitor the status of each task.
- **Security**: Ensure that your AWS credentials are managed securely.
- **Customization**: Modify the DAG and Bash commands as needed to fit your specific deployment environment and requirements.

By following these steps, you should be able to successfully deploy your `Rag-chatbot` application to AWS, leveraging the power of Apache Airflow for orchestration.
```

Save this content in a `README.md` file in your repository. This will provide clear instructions on how to deploy your `Rag-chatbot` application to AWS using Apache Airflow for automation.
