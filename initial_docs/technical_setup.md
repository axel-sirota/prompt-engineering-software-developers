**Barclays Courses**

**Technical Setup Specification**

Prepared by Axel Sirota

For Datacouch infrastructure provisioning

April 2026

## **Courses covered**

* Generative AI: Prompt Engineering for Software Developers (3 days)

* Generative AI for Developers (3 days)

* Cohort size: 25 students per course

# **1\. Architecture Overview**

Each student gets two things:

* **A personal Ubuntu VM (EC2).** Thin client, low spec. This is where the student opens a terminal and a browser, reaches the AWS console, and runs local Python if anything truly needs to stay local. Mirrors the 'my laptop' of a Colab workflow.

* **A personal SageMaker Studio user profile** inside a shared domain. From the Studio UI the student launches a JupyterLab Space pinned to a single instance type. All real work (notebooks, training, RAG, chatbot) happens here. Inside the Space, everything runs localhost: pip install works, ChromaDB runs local, API keys go in env vars, code uses localhost URLs. No SageMaker training jobs, no endpoints, no distributed training.

One shared S3 bucket per course holds the datasets, plus one shared scratch bucket students can write to.

The stack:

Barclays student laptop (browser \+ SSH)

        |

        \+--\> Ubuntu VM (per student, t3.medium)

        |       python3, git, awscli, vscode, pytorch, transformers

        |

        \+--\> AWS Console / SageMaker Studio

                Studio Domain (1)

                 \+-- User Profile: student-01..student-25

                 |     JupyterLab Space

                 |       pinned instance type (per course)

                 |         localhost: notebook \+ ChromaDB \+ training

                 |

                 \+-- S3 buckets (us-east-2)

                       barclays-prompt-eng-data   (read-only)

                       barclays-genai-devs-data   (read-only)

                       barclays-training-scratch  (read/write)

# **2\. AWS Region**

us-east-2 (Ohio) for all resources.

# **3\. Ubuntu VM (per student)**

## **Spec**

| Item | Value |
| :---- | :---- |
| AMI | Ubuntu Server 24.04 LTS (latest) |
| Instance type | t3.medium (2 vCPU, 4 GB RAM) |
| Root volume | 30 GB gp3 |
| Count | 25 per course (50 total if both run concurrently) |
| Access | SSH (key pair) plus browser-accessible VS Code Remote or code-server |
| Public IP | Yes, or behind a Datacouch bastion (Datacouch choice) |
| Security group | Inbound 22 from instructor and student IPs, egress all |

## **Pre-installed software (bake into AMI)**

Install the following during AMI creation so every student starts from the same image:

\# System

sudo apt update && sudo apt upgrade \-y

sudo apt install \-y build-essential curl wget unzip jq tree htop

\# Python 3.11 \+ pip \+ venv

sudo apt install \-y python3.11 python3.11-venv python3-pip

python3 \-m pip install \--upgrade pip

\# Git

sudo apt install \-y git

git \--version   \# expect \>= 2.30

\# AWS CLI v2

curl 'https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip' \-o awscliv2.zip

unzip awscliv2.zip && sudo ./aws/install

aws \--version   \# expect aws-cli/2.x

\# VS Code (desktop install; students SSH-remote from their laptop, or use code-server)

sudo snap install code \--classic

\# Python libs (CPU-only, for local experimentation)

pip install \\

  torch \--index-url https://download.pytorch.org/whl/cpu \\

  transformers \\

  datasets \\

  accelerate \\

  sentence-transformers \\

  chromadb \\

  openai \\

  anthropic \\

  pymupdf \\

  beautifulsoup4 \\

  jupyter \\

  jupyterlab \\

  pandas numpy scikit-learn matplotlib

No Docker. Per instructor request.

## **AWS credentials on the VM**

Each student's VM should have AWS CLI configured with that student's IAM access key. Simplest path: Datacouch drops the key into \~/.aws/credentials per VM before class, tied to that student's IAM user.

# **4\. SageMaker Studio Domain**

## **Domain**

| Item | Value |
| :---- | :---- |
| Domain name | barclays-training |
| Region | us-east-2 |
| Authentication | IAM (one IAM user per student) |
| Network | VPC-only or public, Datacouch preference. Public is simpler. |
| Default execution role | SageMakerStudentExecutionRole (see Section 5\) |
| Storage | Default EFS (Studio-managed) |

## **User profiles**

25 user profiles per course: student-01, student-02, ..., student-25. Each profile shares the same domain and the same execution role. Each gets its own EFS home directory automatically.

## **JupyterLab Space instance type (pinned per course)**

This is the one lab-instance decision, one size for every student in that course.

| Course | Instance type | Specs | Cost per hour |
| :---- | :---- | :---- | :---- |
| Prompt Engineering for Software Developers | ml.t3.medium | 2 vCPU, 4 GB, no GPU | \~$0.05 |
| Generative AI for Developers | ml.g4dn.xlarge | 4 vCPU, 16 GB, 1x T4 16 GB VRAM | \~$0.74 |

Rationale:

* Prompt Eng is API calls plus PyMuPDF, BeautifulSoup, ChromaDB, and web search. No GPU needed, ever.

* Gen AI for Devs runs full-finetune Flan-T5, transfer-learn DistilBERT, LoRA, QLoRA, and quantization-aware training. The T4 is the Colab-equivalent GPU and the smallest AWS option that comfortably handles all of these with small educational datasets. Instructor confirmed educational-scale, no need for A10G or larger.

## **Space storage**

Default 100 GB EBS per Space for the Gen AI for Devs course (model checkpoints, HF cache). 30 GB is enough for Prompt Eng.

## **Lifecycle and auto-shutdown**

No idle shutdown. Datacouch handles tearing Spaces down after class.

## **SageMaker Studio image**

Default SageMaker Distribution image (latest, includes PyTorch, Transformers, CUDA). Students pip install anything missing from within the notebook. No custom images required.

# **5\. IAM Permissions**

Permissions were validated through three review cycles to confirm coverage for the class flow plus headroom for any remote-training scenarios that might arise mid-course. Final policy is below.

## **Per-student IAM user**

One IAM user per student. Each IAM user gets:

* AWS Console access so they can open SageMaker Studio in the browser.

* Access key plus secret so they can aws configure on the Ubuntu VM.

* Assignment to a group barclays-students with the policy below.

## **barclays-students group policy**

{

  "Version": "2012-10-17",

  "Statement": \[

    {

      "Sid": "SageMakerStudioAccess",

      "Effect": "Allow",

      "Action": \[

        "sagemaker:CreatePresignedDomainUrl",

        "sagemaker:DescribeDomain",

        "sagemaker:DescribeUserProfile",

        "sagemaker:ListApps",

        "sagemaker:CreateApp",

        "sagemaker:DeleteApp",

        "sagemaker:DescribeApp",

        "sagemaker:CreateSpace",

        "sagemaker:UpdateSpace",

        "sagemaker:DeleteSpace",

        "sagemaker:DescribeSpace",

        "sagemaker:ListSpaces"

      \],

      "Resource": "\*"

    },

    {

      "Sid": "SageMakerTrainingAndInferenceJustInCase",

      "Effect": "Allow",

      "Action": \[

        "sagemaker:CreateTrainingJob",

        "sagemaker:DescribeTrainingJob",

        "sagemaker:StopTrainingJob",

        "sagemaker:ListTrainingJobs",

        "sagemaker:CreateProcessingJob",

        "sagemaker:DescribeProcessingJob",

        "sagemaker:StopProcessingJob",

        "sagemaker:CreateModel",

        "sagemaker:CreateEndpoint",

        "sagemaker:CreateEndpointConfig",

        "sagemaker:InvokeEndpoint",

        "sagemaker:DeleteEndpoint",

        "sagemaker:DeleteEndpointConfig",

        "sagemaker:DeleteModel",

        "sagemaker:DescribeEndpoint",

        "sagemaker:ListEndpoints"

      \],

      "Resource": "\*"

    },

    {

      "Sid": "PassRoleToSageMaker",

      "Effect": "Allow",

      "Action": "iam:PassRole",

      "Resource": "arn:aws:iam::\<ACCOUNT\_ID\>:role/SageMakerStudentExecutionRole",

      "Condition": {

        "StringEquals": { "iam:PassedToService": "sagemaker.amazonaws.com" }

      }

    },

    {

      "Sid": "S3DatasetsRead",

      "Effect": "Allow",

      "Action": \["s3:GetObject", "s3:ListBucket"\],

      "Resource": \[

        "arn:aws:s3:::barclays-prompt-eng-data",

        "arn:aws:s3:::barclays-prompt-eng-data/\*",

        "arn:aws:s3:::barclays-genai-devs-data",

        "arn:aws:s3:::barclays-genai-devs-data/\*"

      \]

    },

    {

      "Sid": "S3ScratchBucketReadWrite",

      "Effect": "Allow",

      "Action": \[

        "s3:GetObject",

        "s3:PutObject",

        "s3:DeleteObject",

        "s3:ListBucket"

      \],

      "Resource": \[

        "arn:aws:s3:::barclays-training-scratch",

        "arn:aws:s3:::barclays-training-scratch/\*"

      \]

    },

    {

      "Sid": "ECRReadForDeepLearningContainers",

      "Effect": "Allow",

      "Action": \[

        "ecr:GetAuthorizationToken",

        "ecr:BatchCheckLayerAvailability",

        "ecr:GetDownloadUrlForLayer",

        "ecr:BatchGetImage",

        "ecr-public:GetAuthorizationToken",

        "ecr-public:BatchCheckLayerAvailability",

        "ecr-public:GetDownloadUrlForLayer",

        "ecr-public:BatchGetImage"

      \],

      "Resource": "\*"

    },

    {

      "Sid": "CloudWatchLogsForTrainingJobs",

      "Effect": "Allow",

      "Action": \[

        "logs:CreateLogGroup",

        "logs:CreateLogStream",

        "logs:PutLogEvents",

        "logs:DescribeLogStreams",

        "logs:GetLogEvents"

      \],

      "Resource": "arn:aws:logs:\*:\*:log-group:/aws/sagemaker/\*"

    },

    {

      "Sid": "BedrockInvokeOptional",

      "Effect": "Allow",

      "Action": \[

        "bedrock:InvokeModel",

        "bedrock:InvokeModelWithResponseStream",

        "bedrock:ListFoundationModels"

      \],

      "Resource": "\*"

    },

    {

      "Sid": "IAMSelfInspection",

      "Effect": "Allow",

      "Action": \[

        "iam:GetUser",

        "iam:ListAttachedUserPolicies",

        "iam:ListGroupsForUser"

      \],

      "Resource": "arn:aws:iam::\*:user/${aws:username}"

    },

    {

      "Sid": "ServiceQuotasRead",

      "Effect": "Allow",

      "Action": \[

        "servicequotas:ListServiceQuotas",

        "servicequotas:GetServiceQuota"

      \],

      "Resource": "\*"

    }

  \]

}

## **SageMakerStudentExecutionRole**

This is the role the Studio Space assumes when it runs. Attach:

* **AmazonSageMakerFullAccess** (AWS managed). Pragmatic choice for a classroom. If Datacouch prefers tight least-privilege, that is acceptable too, but full access removes an entire class of in-class surprises.

* A custom inline policy granting read on the two dataset buckets and read/write on the scratch bucket.

Trust policy: sagemaker.amazonaws.com.

Inline policy:

{

  "Version": "2012-10-17",

  "Statement": \[

    {

      "Effect": "Allow",

      "Action": \["s3:GetObject", "s3:ListBucket"\],

      "Resource": \[

        "arn:aws:s3:::barclays-prompt-eng-data",

        "arn:aws:s3:::barclays-prompt-eng-data/\*",

        "arn:aws:s3:::barclays-genai-devs-data",

        "arn:aws:s3:::barclays-genai-devs-data/\*"

      \]

    },

    {

      "Effect": "Allow",

      "Action": \[

        "s3:GetObject",

        "s3:PutObject",

        "s3:DeleteObject",

        "s3:ListBucket"

      \],

      "Resource": \[

        "arn:aws:s3:::barclays-training-scratch",

        "arn:aws:s3:::barclays-training-scratch/\*"

      \]

    }

  \]

}

## **KMS note**

If Datacouch encrypts the dataset or scratch buckets with customer-managed KMS keys (SSE-KMS), students and the execution role will additionally need kms:Decrypt (and kms:GenerateDataKey for scratch writes) on the key. If buckets use SSE-S3 (default), no extra KMS permission is needed.

# **6\. Service Quotas (raise before class starts)**

Quota increases take 24 to 72 hours for AWS to approve. Submit early. Values below are padded for safety at roughly 2x the minimum needed.

| Quota | Minimum needed | Request this value | Why |
| :---- | :---- | :---- | :---- |
| Studio JupyterLab Apps running on ml.g4dn.xlarge | 25 | 50 | One per student in Gen AI for Devs, doubled |
| Studio JupyterLab Apps running on ml.t3.medium | 25 | 50 | One per student in Prompt Eng, doubled |
| Running On-Demand G and VT instances (vCPU) | 100 | 200 | g4dn.xlarge \= 4 vCPU x 25 students |
| Running On-Demand T instances (vCPU) | 100 | 200 | t3.medium Ubuntu VMs and Studio Spaces |
| ml.g4dn.xlarge for training job usage | 5 | 20 | Safety margin if anyone runs a training job |
| ml.g4dn.xlarge for processing job usage | 5 | 10 | In case of remote processing jobs |
| ml.g4dn.xlarge for endpoint usage | 5 | 10 | In case of remote inference |
| Studio domains | 1 | 3 | Room for a staging domain |
| User profiles per domain | 50 | 100 | 25 x 2 courses, doubled |

**For reference (current limit)**

**ml.g4dn.xlarge**

![][image1]

ml.t3.medium

![][image2]

Quota page: AWS Console, Service Quotas, Amazon SageMaker. Some quotas live under EC2 rather than SageMaker (the G and T vCPU quotas).

# **7\. S3 Buckets**

## **barclays-prompt-eng-data (us-east-2)**

Contents (Datacouch plus instructor populate before day 1):

* barclays-products/ with Barclays product brochures, T\&Cs, and FAQ PDFs for NLP preprocessing and RAG labs.

* sample-conversations/ with example customer queries for evaluation.

## **barclays-genai-devs-data (us-east-2)**

Contents:

* datasets/ with small educational datasets (IMDB subset for DistilBERT, a small summarization set, a small NER set, a tiny translation set such as a Multi30k subset).

* models/ optional, with local mirrors of Flan-T5-base, DistilBERT-base, in case HF Hub is slow or network-restricted from the Space.

## **barclays-training-scratch (us-east-2)**

Shared read/write bucket. Students use prefixes like s3://barclays-training-scratch/student-07/ to save checkpoints, artifacts, or intermediate outputs. No prefix isolation at the IAM level (keeps things simple), relies on convention.

## **Bucket configuration for all three**

* Versioning on.

* Public access fully blocked.

* Default encryption SSE-S3 (unless Barclays policy requires SSE-KMS, in which case see KMS note in Section 5).

* Lifecycle rule on scratch bucket: delete objects older than 30 days.

# **8\. API Keys (OpenAI, Anthropic)**

Instructor hands out keys verbally in class. Students paste them into a notebook cell as environment variables, Colab-style:

import os

os.environ\["OPENAI\_API\_KEY"\] \= "sk-..."

os.environ\["ANTHROPIC\_API\_KEY"\] \= "sk-ant-..."

Datacouch does not provision these and does not inject them via lifecycle configs. Keep the setup simple.

# **9\. What Datacouch Needs to Deliver**

Before day 1 of each course:

* 25 Ubuntu VMs with the AMI spec in Section 3, each reachable by SSH, one per student.

* 1 SageMaker Studio domain in us-east-2 with 25 user profiles.

* 25 IAM users in the barclays-students group, each with console password plus access key.

* Service quotas raised per Section 6\.

* 3 S3 buckets populated per Section 7\.

* A handout per student with: SSH command plus key for their Ubuntu VM, AWS Console URL, their IAM username plus password, their SageMaker Studio profile name, and the names of the shared dataset and scratch buckets.

* A sanity-check notebook that students run as lab 0: verifies Python version, torch.cuda.is\_available() (Gen AI for Devs only), S3 read access on both dataset buckets, S3 write access on scratch, HF Hub reachability, OpenAI and Anthropic connectivity with a throwaway key.

# **10\. Cost Estimate (rough)**

Assuming 3 days times 8 hours classroom time per course, 25 students per course:

| Course | Resource | Hours | Students | Approx total |
| :---- | :---- | :---- | :---- | :---- |
| Prompt Eng | ml.t3.medium Studio Space | 24 | 25 | $30 |
| Prompt Eng | t3.medium Ubuntu VM | 24 | 25 | $25 |
| Gen AI for Devs | ml.g4dn.xlarge Studio Space | 24 | 25 | $445 |
| Gen AI for Devs | t3.medium Ubuntu VM | 24 | 25 | $25 |
| All | S3, CloudWatch, misc |  |  | $15 |

Per-course rough totals: Prompt Eng around $55, Gen AI for Devs around $470. Costs assume Datacouch tears Spaces and VMs down at the end of class each day, or at least at the end of the course.

# **11\. Consistency with the Colab Workflow**

The design goal is that student code works the same way it would in Colab. Specifically:

* pip install \<x\> works.

* import torch; torch.cuda.is\_available() returns True in Gen AI for Devs, False in Prompt Eng. Intended.

* chromadb.PersistentClient(path="./chroma\_db") stores locally in the Space EFS home.

* OpenAI() and Anthropic() clients read from env vars students set at notebook top.

* from transformers import AutoModel; model \= AutoModel.from\_pretrained("google/flan-t5-base") downloads to the local HF cache on the Space.

* Any notebook written for Colab runs inside the Space with zero changes.

The Ubuntu VM is the consistent shell for anything that does not belong in the notebook (git clone, curl, aws s3 cp, local scripts).

End of specification.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>