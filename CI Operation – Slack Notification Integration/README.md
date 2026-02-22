# Generic CI Operation – Slack Notification Integration

##  Overview

This document describes the implementation of Slack notifications in a Jenkins CI pipeline.  
The objective is to send automated build status updates (Success/Failure) to a designated Slack channel using a secure Bot User OAuth token.

---

##  Objective

To integrate Jenkins with Slack and enable automated notifications for:

- Build Success
- Build Failure
- CI Execution Status

This ensures real-time visibility and structured communication within the DevOps workflow.

---

##  Architecture

Slack App (Bot)
↓
Bot OAuth Token (xoxb-...)
↓
Jenkins Credential (Secret Text)
↓
Slack Plugin (Bot User Enabled)
↓
Jenkins Declarative Pipeline
↓
Slack Channel Notification


---

##  Implementation Steps

---

### 1️ Create Slack App

1. Navigate to: https://api.slack.com/apps
2. Click **Create New App**
3. Select **From Scratch**
4. Choose Workspace

---

### 2️ Configure Bot Token Scopes

Go to:
OAuth & Permissions → Bot Token Scopes

Add the following scopes:

- `chat:write`
- `chat:write.public`

Click **Install to Workspace**.

---

### 3️ Retrieve Bot OAuth Token

After installation, copy:
Bot User OAuth Token (xoxb-xxxxxxxx)


This token will be used inside Jenkins.

---

### 4️ Add Slack Token in Jenkins

Navigate to:
Manage Jenkins → Credentials → Global

Create new credential:

- Kind: **Secret Text**
- ID: `Slack-Notification-Token`
- Secret: `xoxb-xxxxxxxxxxxx`

Save.

---

### 5️ Configure Slack Plugin in Jenkins

Go to:

Manage Jenkins → Configure System → Slack


Configure:

- Credential: `Slack-Notification-Token`
- Default Channel: `#ci-operation-notifications`
- Enable:  Custom Slack App Bot User

Click **Test Connection** → Must show **Success**

Save configuration.

---

### 6️ Prepare Slack Channel

Create Slack channel:
#ci-operation-notifications
Invite the bot:


/invite @jenkins-bot


Ensure bot joins successfully.

---

##  Jenkins Pipeline Configuration

Use a Declarative Pipeline:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Simulating build..."
            }
        }

        stage('Test') {
            steps {
                echo "Simulating test..."
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#ci-operation-notifications',
                message: """
                 Build Successful
                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """
            )
        }

        failure {
            slackSend(
                channel: '#ci-operation-notifications',
                message: """
                 Build Failed
                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """
            )
        }
    }
}
