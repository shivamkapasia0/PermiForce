
# PermiForce: Azure DevOps PR Approval System with Permit.io

  

A robust Pull Request approval system that enforces role-based access control for your Azure DevOps pipelines using Permit.io.

  

## Table of Contents

-  [Overview](#overview)

-  [Core Concepts](#core-concepts)

-  [User Roles and Permissions](#user-roles-and-permissions)

-  [Setup Instructions](#setup-instructions)

-  [Prerequisites](#prerequisites)

-  [Azure DevOps Setup](#azure-devops-setup)

-  [Permit.io Setup](#permitio-setup)

-  [Permission Structure](#permission-structure)

-  [Testing the System](#testing-the-system)

-  [Troubleshooting](#troubleshooting)

-  [Contributing](#contributing)

  

## Overview

  

PermiForce is a sophisticated Pull Request approval system that integrates Azure DevOps with Permit.io's fine-grained permission control. It ensures that only authorized personnel can approve and merge code changes into protected branches, maintaining code quality and security.

  

## Core Concepts

  

The system is built around three main components:

  

1.  **Role-Based Access Control (RBAC)**: Defines who can perform specific actions

2.  **Branch Protection**: Ensures protected branches (qa, uat, prod) have proper approvals

3.  **Automated Validation**: Checks permissions and approvals automatically in the CI/CD pipeline

  

## User Roles and Permissions

  

### Role-Based Access Structure

  

1.  **Developer Role**

- Primary permission: `CREATE_QA_PR`

- Can create Pull Requests to QA branch only

- Cannot approve any PRs

- Users: alice@permiforce.com, bob@permiforce.com

  

2.  **Team Lead Role**

- Permissions:

-  `CREATE_UAT_PR`: Can create PRs to UAT

-  `APPROVE_QA_PR`: Can approve PRs to QA

-  `APPROVE_UAT_PR`: Can approve PRs to UAT

- Cannot create or approve Production PRs

- User: carol@permiforce.com

  

3.  **Release Manager Role**

- Full access with all permissions:

-  `APPROVE_PROD_PR`: Approve Production PRs

-  `APPROVE_QA_PR`: Approve QA PRs

-  `APPROVE_UAT_PR`: Approve UAT PRs

-  `CREATE_PROD_PR`: Create Production PRs

-  `CREATE_QA_PR`: Create QA PRs

-  `CREATE_UAT_PR`: Create UAT PRs

- Highest level of authorization

- User: shivamsfdc.work@gmail.com

  

### Detailed Permission Matrix

  ![Permissions Image](https://raw.githubusercontent.com/shivamkapasia0/PermiForce/main/permissions.png)

![User Roles Image](https://raw.githubusercontent.com/shivamkapasia0/PermiForce/main/user_roles.png)

### Permission Flow Examples
1.  **QA Branch Flow**

```mermaid

graph TD

A[Developer] -->|Can Create| B[QA PR]

B --> C{Needs Approval}

C -->|Can Approve| D[Team Lead]

C -->|Can Approve| E[Release Manager]

```

  

2.  **UAT Branch Flow**

```mermaid

graph TD

A[Team Lead] -->|Can Create| B[UAT PR]

B --> C{Needs Approval}

C -->|Can Approve| D[Team Lead]

C -->|Can Approve| E[Release Manager]

```

  

3.  **Production Branch Flow**

```mermaid

graph TD

A[Release Manager] -->|Can Create| B[Production PR]

B --> C{Needs Approval}

C -->|Only RM Can Approve| D[Release Manager]

```

  

### Branch Protection Rules

  

```mermaid

graph TD

subgraph "QA Branch Protection"

QA[QA Branch] --> QAC[Creation Rights]

QA --> QAA[Approval Rights]

QAC --> QAC1[Developers]

QAC --> QAC2[Release Managers]

QAA --> QAA1[Team Leads]

QAA --> QAA2[Release Managers]

QA --> QAR[Requirements]

QAR --> QAR1[1 Approver]

end
```
  
```mermaid
graph TD
subgraph "UAT Branch Protection"

UAT[UAT Branch] --> UATC[Creation Rights]

UAT --> UATA[Approval Rights]

UATC --> UATC1[Team Leads]

UATC --> UATC2[Release Managers]

UATA --> UATA1[Team Leads]

UATA --> UATA2[Release Managers]

UAT --> UATR[Requirements]

UATR --> UATR1[1 Approver]

end
```
  
```mermaid
graph TD
subgraph "Production Branch Protection"

PROD[Production Branch] --> PRODC[Creation Rights]

PROD --> PRODA[Approval Rights]

PRODC --> PRODC1[Release Managers Only]

PRODA --> PRODA1[Release Managers Only]

PROD --> PRODR[Requirements]

PRODR --> PRODR1[1 Approver]

end

```

## Setup Instructions
### Prerequisites
- Azure DevOps account with admin access

- Permit.io account

- Node.js 18.x or higher

- Git

### Azure DevOps Setup

1.  **Create a New Pipeline**
	-   Navigate to the Azure DevOps project where you want to create the pipeline.
    
	-   Click on **Pipelines** in the left sidebar, then click on **New Pipeline**.
    
	-   Select the repository you wish to use for your pipeline.
    
	-   Choose **YAML** as the pipeline configuration option.
    
	-   Select **Start from scratch** or choose an existing YAML template if available.
    
	-   In the pipeline editor, use the default YAML configuration and ensure that `main-pipeline.yml` is set as the YAML file to be used for all branches.

2.  **Configure Variable Group**

- Create a variable group named `permit-variables`
- Add the following variables:
```
PERMIT_API_KEY: [Your Permit.io API Key]
PERMIT_PDP_URL: [Your Permit.io PDP URL]
USER_NAME: [User's Email For Simulation]
```

3.  **Branch Protection Rules**
- Go to Project Settings > Repositories
- Set up branch policies for qa, uat, and prod branches
- Enable "Require a minimum number of reviewers"

### Permit.io Setup

1.  **Create a New Project**

- Log in to Permit.io

- Create a new project named "PermiForce" or any you like.

2.  **Define Resources**

- Create a resource named "**DevOps**"

- Add the following permissions/roles:
![Permissions Image](https://raw.githubusercontent.com/shivamkapasia0/PermiForce/main/permissions.png)

4.  **Add Users**

![User Roles Image](https://raw.githubusercontent.com/shivamkapasia0/PermiForce/main/user_roles.png)
  

## Permission Structure

The APPROVE_PROD_PR permission is the highest level of approval permission in the system. It:

- Is exclusively granted to Release Managers

- Required for merging code into the production branch

- Cannot be delegated to lower-level roles

- Requires additional validation checks in the pipeline


## Testing the System

  

1.  **Test as Developer**

```bash
# Set pipeline variable
user_name: alice@permiforce.com
# Expected Results:
- Can create PR to QA ✅
- Cannot create PR to UAT ❌
- Cannot create PR to PROD ❌
```

2.  **Test as Team Lead**

```bash
# Set pipeline variable
user_name: carol@permiforce.com
# Expected Results:
- Can approve QA/UAT PRs ✅
- Cannot approve PROD PRs ❌
```

3.  **Test as Release Manager**

```bash
# Set pipeline variable
user_name: shivamsfdc.work@gmail.com
# Expected Results:
- Can approve all PRs ✅
- Can create PRs to any branch ✅
```

## Troubleshooting

  

Common issues and solutions:

  

1.  **Pipeline Permission Error**

- Check if USER_NAME is correctly set

- Verify Permit.io API key is valid

- Ensure user exists in Permit.io

  

2.  **PR Approval Failure**

- Verify approver has correct role assigned

- Check if branch policies are properly configured

- Ensure pipeline variables are set correctly

  

## Contributing

  

1. Fork the repository

2. Create your feature branch

3. Commit your changes

4. Push to the branch

5. Create a new Pull Request

  

## Technical Details

  

### System Architecture

  

```mermaid

graph TD

A[Azure DevOps] -->|Triggers| B[Main Pipeline]

B -->|Starts| C[Permission Server]

B -->|Runs| D[Check Access]

B -->|Runs| E[Check Approval]

B -->|Runs| F[Code Validation]

C -->|Connects to| G[Permit.io]

D -->|Validates via| C

E -->|Validates via| C

subgraph "Permission Flow"

G -->|Returns| H[Allow/Deny]

H -->|Influences| I[Pipeline Decision]

end

```

  

### Pipeline Components

  

| Component | File | Purpose | Key Features |

|-----------|------|---------|--------------|

| Main Pipeline |  `main-pipeline.yml`  | Orchestrates the entire process | - Triggers on PR<br>- Sets up environment<br>- Runs checks |

| Access Check |  `check-access.yml`  | Validates PR creation permissions | - Branch mapping<br>- Permission validation |

| Approval Check |  `check-approval.yml`  | Validates PR approver permissions | - Reviewer validation<br>- Role checking |

| Code Validation |  `check-code.yml`  | Validates code quality | - Quality checks<br>- Standards enforcement |

  

### Pipeline Flow

  

1.  **Pipeline Trigger** (`main-pipeline.yml`)

```yaml

trigger:

-  main

pr:

branches:

include:

-  qa

-  uat

-  prod

```

- Activates when PR is created targeting protected branches

- Sets up environment variables and starts permission server

  

2.  **Permission Server Setup**

```mermaid

sequenceDiagram

participant P as Pipeline

participant S as Server

participant D as Permit.io

P->>S: Start Server

S->>S: Initialize

S->>D: Connect

S->>P: Ready

Note over S,D: Continuous Connection

```

- Initializes Node.js server (`permit-server.js`)

- Establishes connection with Permit.io

- Provides REST endpoints for permission checks

  

3.  **Access Check Flow**

```mermaid

sequenceDiagram

participant U as User

participant P as Pipeline

participant S as Server

participant D as Permit.io

U->>P: Create PR

P->>S: Check Permission

S->>D: Validate

D-->>S: Response

S-->>P: Allow/Deny

P-->>U: Result

```

  

### Server Components

  

1.  **Permit Server** (`scripts/permit-server.js`)

```javascript

const  permit  =  new  Permit({

pdp:  process.env.PERMIT_PDP_URL,

token:  process.env.PERMIT_API_KEY,

});

```

  

| Endpoint | Purpose | Response |

|----------|---------|----------|

|  `/check-access`  | Validates PR creation | Allow/Deny with details |

|  `/check-pr-approval`  | Validates approver | Allow/Deny with role check |

|  `/health`  | Server health check | Status with environment info |

  

2.  **Permission Check** (`scripts/permit.js`)

```javascript

async  function  checkAccess(user,  action,  resource)  {

const  allowed  =  await  permit.check(user,  action,  resource);

return  allowed;

}

```

  

### Branch-Specific Workflows

  

1.  **QA Branch Workflow**

```mermaid

stateDiagram-v2

[*] --> CreatePR

CreatePR --> CheckPermission

CheckPermission --> ValidateApprover

ValidateApprover --> MergePR

ValidateApprover --> RejectPR

state CheckPermission {

[*] --> CREATE_QA_PR

CREATE_QA_PR --> [*]

}

state ValidateApprover {

[*] --> APPROVE_QA_PR

APPROVE_QA_PR --> [*]

}

```

  

2.  **UAT Branch Workflow**

```mermaid

stateDiagram-v2

[*] --> CreatePR

CreatePR --> CheckPermission

CheckPermission --> ValidateApprover

ValidateApprover --> MergePR

ValidateApprover --> RejectPR

state CheckPermission {

[*] --> CREATE_UAT_PR

CREATE_UAT_PR --> [*]

}

state ValidateApprover {

[*] --> APPROVE_UAT_PR

APPROVE_UAT_PR --> [*]

}

```

  

3.  **Production Branch Workflow**

```mermaid

stateDiagram-v2

[*] --> CreatePR

CreatePR --> CheckPermission

CheckPermission --> ValidateApprover

ValidateApprover --> MergePR

ValidateApprover --> RejectPR

state CheckPermission {

[*] --> CREATE_PROD_PR

CREATE_PROD_PR --> [*]

}

state ValidateApprover {

[*] --> APPROVE_PROD_PR

APPROVE_PROD_PR --> [*]

}

```

  

### Error Handling and Responses

  

1.  **Permission Errors**

```json

{

"error":  "Permission denied",

"details":  "User lacks APPROVE_PROD_PR permission",

"user":  "developer@permiforce.com",

"requiredPermission":  "APPROVE_PROD_PR"

}

```

  

2.  **Server Errors**

```json

{

"error":  "Server unavailable",

"status":  "error",

"details":  "Failed to connect to Permit.io"

}

```

  

### Monitoring and Logging

  

| Aspect | Tools | Purpose |

|--------|-------|---------|

| Permission Checks | Pipeline Logs | Track all permission validations |

| Server Health | Health Endpoint | Monitor server status |

| Error Tracking | Error Logs | Track system issues |

| Performance | Azure DevOps | Monitor pipeline performance |

  

### Common Troubleshooting Scenarios

  

| Issue | Possible Cause | Solution |

|-------|---------------|----------|

| PR Creation Failed | Invalid permissions | Check user role and permissions |

| Approval Failed | Unauthorized approver | Verify approver's role |

| Server Connection Error | Network/Config issues | Check server logs and config |

| Pipeline Timeout | Server not responding | Check server health endpoint |

  

### Best Practices

  

1.  **Permission Management**

- Regularly audit user roles

- Follow principle of least privilege

- Document permission changes

  

2.  **Pipeline Configuration**

- Keep environment variables secure

- Regular health checks

- Proper error handling

  

3.  **Monitoring**

- Set up alerts for failures

- Monitor permission denials

- Track approval patterns

  

---

  

For more information, contact the project maintainers or visit our documentation.

  

## Detailed Setup Instructions

  

### Prerequisites Installation

  

1.  **Node.js Setup**

```bash

# Install Node.js 18.x

curl -fsSL https://deb.nodesource.com/setup_18.x |  sudo  -E  bash  -

sudo apt-get install -y nodejs

  

# Verify installation

node --version

npm --version

```

  

2.  **Azure DevOps CLI**

```bash

# Install Azure CLI

curl -sL https://aka.ms/InstallAzureCLIDeb |  sudo  bash

# Install Azure DevOps extension

az extension add --name azure-devops

```

  

### Permit.io SDK Setup

  

1.  **Install Permit.io SDK**

```bash

npm install permitio

```

  

2.  **SDK Configuration**

```javascript

// Example SDK initialization

const  {  Permit  }  =  require('permitio');

  

const  permit  =  new  Permit({

pdp:  process.env.PERMIT_PDP_URL,  // PDP URL from Permit.io

token:  process.env.PERMIT_API_KEY,  // API Key from Permit.io

debug:  process.env.NODE_ENV  !==  'production'  // Enable debug logs in non-prod

});

```

  

3.  **SDK Usage Examples**

```javascript

// Check permissions

const  allowed  =  await  permit.check(user,  action,  resource);

  

// Get user permissions

const  permissions  =  await  permit.getPermissions(user);

  

// Sync roles

await  permit.syncRole(roleName,  permissions);

```

  

### Azure DevOps Pipeline Setup

  

1.  **Create Pipeline Files**

```bash

# Create pipeline directory

mkdir -p .azure/pipelines

  

# Create required files

touch .azure/pipelines/main-pipeline.yml

touch .azure/pipelines/check-access.yml

touch .azure/pipelines/check-approval.yml

touch .azure/pipelines/check-code.yml

```

  

2.  **Configure Variable Group**

```bash

# Using Azure CLI

az pipelines variable-group create \

--name permit-variables \

--variables \

PERMIT_API_KEY=""  \

PERMIT_PDP_URL="" \

USER_NAME=""  \

--authorize true

```

  

### YAML Workflows Architecture

  

```mermaid

graph TD

subgraph "Main Pipeline Flow"

A[main-pipeline.yml] -->|Triggers on PR| B[Setup Phase]

B -->|Start| C[Permission Server]

B -->|Configure| D[Environment]

C -->|Health Check| E[Server Ready]

D -->|Variables Set| E

E -->|Run| F[check-access.yml]

F -->|Success| G[check-approval.yml]

G -->|Success| H[check-code.yml]

end

  

subgraph "check-access.yml"

F -->|1| F1[Get User]

F1 -->|2| F2[Get Branch]

F2 -->|3| F3[Check Permission]

end

  

subgraph "check-approval.yml"

G -->|1| G1[Get Reviewers]

G1 -->|2| G2[Check Roles]

G2 -->|3| G3[Validate Approval]

end

  

subgraph "check-code.yml"

H -->|1| H1[Code Analysis]

H1 -->|2| H2[Quality Check]

end

```

  

### Pipeline Components Interaction

  

```mermaid

sequenceDiagram

participant MP as main-pipeline.yml

participant PS as Permission Server

participant CA as check-access.yml

participant CP as check-approval.yml

participant CC as check-code.yml

participant P as Permit.io

  

MP->>PS: Start Server

PS->>P: Connect

P-->>PS: Connected

PS-->>MP: Ready

  

MP->>CA: Run Access Check

CA->>PS: Check Permission

PS->>P: Validate

P-->>PS: Response

PS-->>CA: Result

CA-->>MP: Success/Failure

  

MP->>CP: Run Approval Check

CP->>PS: Validate Approvers

PS->>P: Check Roles

P-->>PS: Response

PS-->>CP: Result

CP-->>MP: Success/Failure

  

MP->>CC: Run Code Check

CC-->>MP: Success/Failure

```

  

### Environment Configuration

  

| Variable | Purpose | Example |

|----------|---------|---------|

|  `PERMIT_API_KEY`  | Authentication with Permit.io |  `pmt_xxxxxxxxxxxx`  |

|  `PERMIT_PDP_URL`  | PDP endpoint URL |  `https://cloudpdp.api.permit.io`  |

|  `USER_NAME`  | Current user's email |  `user@example.com`  |

|  `NODE_ENV`  | Environment mode |  `production`  |

  

### Server Endpoints Detail

  

| Endpoint | Method | Request Body | Response |

|----------|---------|-------------|-----------|

|  `/check-access`  | POST |  `{"user": "email", "targetBranch": "branch"}`  |  `{"allowed": bool, "message": "string"}`  |

|  `/check-pr-approval`  | POST |  `{"approvers": ["email"], "targetBranch": "branch"}`  |  `{"allowed": bool, "message": "string"}`  |

|  `/health`  | GET | None |  `{"status": "string", "environment": "string"}`  |
