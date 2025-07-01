

# 🔐 Secure Secrets with AWS Secrets Manager

**Cloud Security | Credential Management | AWS IAM**  
*Built and documented by Mounika Addula*

---

## 🔎 Project Overview

This project demonstrates how I transitioned a FastAPI web application from using **hardcoded AWS credentials** to a **secure, runtime secret retrieval model** using **AWS Secrets Manager**. The goal was to securely access S3 buckets without exposing sensitive keys in source code or Git history.

---

## 🛠️ Tools & Services Used

- AWS Secrets Manager  
- AWS IAM  
- AWS S3  
- FastAPI, Uvicorn  
- Python (`boto3`, `venv`)  
- Git & GitHub  

---

### Step 1: Exploring and Understanding the Insecure Web App Code

In the initial phase of this project, I began by cloning a FastAPI-based web application from GitHub. The purpose of the app was to list all Amazon S3 buckets within an AWS account. My objective was to understand how the application was designed and identify where and how it managed AWS authentication.

I cloned the repository to my local machine using the following Git command:


git clone <repository-url>

Once the project was downloaded, I opened it in my code editor and began inspecting the files. The main logic of the application was located in app.py. Upon reviewing it, I observed that AWS credentials were not written directly in the file, but were instead being imported from a separate module named config.py.

When I opened config.py, I found that the AWS credentials were hardcoded directly into the source code using plain text Python strings. Specifically, the following sensitive variables were declared:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

AWS_REGION

This design introduces several critical security risks:

The credentials could be committed to version control, exposing them publicly or to internal teams unintentionally.

It violates security best practices by storing secrets in plaintext.

It makes rotating keys and managing access significantly harder.

This discovery confirmed that the application required a major improvement in how it handled secret management. This insight laid the foundation for refactoring the app to use AWS Secrets Manager for runtime credential access.

![Hardcoded AWS Credentials in config.py](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/93eb2eb45c7a9ccde75121a3795c373a6230f065/assets/Screenshot%202025-06-25%20200736.png)


---

### Step 2: Setting Up the Insecure Environment Locally

With the hardcoded credentials identified in `config.py`, I proceeded to simulate how the application behaves in a real-world insecure configuration. This setup allowed me to validate that the AWS credentials were indeed being consumed by the application and to highlight the risks of storing secrets directly in code.

#### 🧱 Hardcoding Sample AWS Credentials

Inside `config.py`, I replaced the placeholder strings with sample AWS credentials:


AWS_ACCESS_KEY_ID = "FAKEACCESSKEY123456"
AWS_SECRET_ACCESS_KEY = "FAKESECRETKEY789012"
AWS_REGION = "us-east-2"
These values were then used in the application logic via the list_s3_buckets() function defined in app.py, which imports and uses them with the boto3 client.

📂 Verifying Project File Structure
I opened a terminal and navigated into the cloned project directory using:

bash
Copy
Edit
cd secure-secrets-project
I verified that all the essential files were in place:

app.py - the FastAPI application logic

config.py - containing the hardcoded AWS credentials

requirements.txt - listing the necessary Python packages

Dockerfile - for containerizing the app (optional for this stage)

Once everything looked correct, I saved the changes to config.py and prepared to run the application in its insecure configuration, setting the stage for testing AWS access in the next step.

![Hardcoded Sample Credentials](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/9b4c16b3472199f4439f47934f9493ffe066f46c/assets/Screenshot%202025-06-23%20132948.png)


---

### Step 3: Setting Up and Activating the Python Virtual Environment
To ensure that project dependencies were isolated and manageable, I created a Python virtual environment within the project directory. This helps avoid conflicts with global packages and maintains a clean environment for the application.

I initialized the virtual environment by running:

**python3 -m venv venv**

After the environment was created, I activated it locally. Since I was using a Windows system, I executed:

**.\venv\Scripts\activate**

Once the environment was active, I installed all necessary dependencies specified in the `requirements.txt` file by running:

**pip install -rrequirements.txt **  These installed core packages are  needed for the app to function, including:
- **boto3** for AWS SDK integration
- **fastapi** for building the API
- **uvicorn** for serving the FastAPI app

With the environment prepared and packages installed, the project was now ready to be executed using the insecure credentials set up from the earlier step.


---


### Step 4: Running the Insecure App and Observing Credential Errors

With the virtual environment set up and dependencies installed, I proceeded to run the application using the insecure placeholder credentials defined in `config.py`.

To launch the web app, I executed the following command from the terminal while inside the project directory:

**python app.py**

This started the FastAPI application via Uvicorn, and the server was successfully launched on:

**http://0.0.0.0:8000**

I opened this URL in my browser, which displayed the homepage of the web app. From there, I clicked the **"View My S3 Buckets"** button to initiate a call to the AWS S3 service and attempt to retrieve a list of available buckets.

![View My S3 Buckets](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/ce3d3767dc12d0f2ea1b946044486d165897db87/assets/Screenshot%202025-06-25%20201134.png)

Instead of listing any S3 buckets, the application returned the following JSON error:


{"error": "An error occurred (InvalidAccessKeyId) when calling the ListBuckets operation: The AWS Access Key Id you provided does not exist in our records."}

This message confirmed that:

The application was configured to use the AWS credentials from config.py

The placeholder credentials were being passed to the AWS SDK (boto3)

AWS correctly rejected the request because the access key was invalid

While the server and API were functioning correctly from a code perspective, this step validated that the app could not connect to AWS services with the current configuration. It served as the foundation for transitioning toward secure, real credential management using AWS IAM and Secrets Manager in the next stages.

![Invalid AWS Credentials Error](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/6299bb28d8c593b8f75f9b2506ce39d996367d3f/assets/Screenshot%202025-06-25%20201156.png)


---


### Step 5: Creating and Configuring AWS Credentials for the App

To enable secure communication between the application and AWS services, I needed to set up valid AWS credentials through Identity and Access Management (IAM). This would allow the app to authenticate properly and perform operations such as listing S3 buckets.

#### 🔐 Setting Up Programmatic Access in IAM

I began by logging into the **AWS Management Console** and navigating to the **IAM** (Identity and Access Management) service. From there, I took the following steps:

1. Selected **Users** from the left-hand menu.
2. Choose my IAM user or create a new one dedicated to this project.
3. Enabled **Programmatic Access** to generate an access key pair (Access Key ID + Secret Access Key).

#### 🪣 Creating a Test S3 Bucket

To validate the integration between the FastAPI app and AWS S3, I also created a test bucket:

- Opened the **Amazon S3** service
- Clicked **Create Bucket**
- Named the bucket (e.g., `fastapi-test-bucket`) and selected the **same region** configured in the app (`us-east-2`)
- Left the default settings and created the bucket

This S3 bucket would later be used to test whether the application can list available storage resources using the AWS SDK.

#### 📥 Generating and Storing Credentials

After creating the IAM user and enabling programmatic access, AWS immediately provided a downloadable `.csv` file containing:

- **Access Key ID**
- **Secret Access Key**

I downloaded the `.csv` file securely and stored it in a safe location. This is critical, as the **Secret Access Key is only shown once** at the time of creation. If lost, a new key must be generated.

These credentials would be temporarily placed in `config.py` (in the next step) before transitioning to a more secure solution using AWS Secrets Manager.


![IAM Access Key Creation](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/813d3bbd5c45e648dffae8183fa18ec1b479795e/assets/Screenshot%202025-06-26%20090425.png)



---


### Step 6: Updating the App to Use Real AWS Credentials

With valid IAM credentials now created and a test S3 bucket available, I moved forward with updating the application to use the real AWS access keys. This step was necessary to validate that the application could successfully connect to AWS S3 and retrieve bucket data when configured with the correct credentials.

#### 🛠️ Replacing Placeholder Values in config.py

I stopped the running instance of the FastAPI application by terminating the process in the terminal.

Then, I opened the `config.py` file and updated the hardcoded placeholder credentials with the actual values obtained from the IAM `.csv` file:


AWS_ACCESS_KEY_ID = "My-real-access-key-id"
AWS_SECRET_ACCESS_KEY = "My-real-secret-access-key"
AWS_REGION = "us-east-2"
It was important to ensure that the AWS_REGION specified in the file matched the region where the S3 bucket had been created (in this case, us-east-2).

After making the changes, I saved the file so that the application would now be able to authenticate correctly with AWS services.

This completed the insecure-to-functional transition. In the following steps, I transitioned away from hardcoding secrets and introduced secure storage via AWS Secrets Manager.


---



### Step 7: Successfully Running the Web App with Real AWS Credentials

With the real IAM credentials now saved in `config.py`, I proceeded to test the full integration between the FastAPI application and AWS S3 services.

#### 🚀 Restarting the Application

I restarted the web app by running the following command:

**python3 app.py**

The Uvicorn development server launched without any issues, indicating that the application successfully loaded the required AWS credentials and initialized the `boto3` client.

#### 🌐 Verifying S3 Access in the Browser

I opened my browser and navigated to:

**http://0.0.0.0:8000**

From the homepage, I clicked on the **“View My S3 Buckets”** button, which triggered the endpoint responsible for listing all S3 buckets associated with the authenticated AWS account.

This time, instead of an error, the app returned a valid **JSON response** containing the names and metadata of the S3 buckets I had created.

#### ✅ Outcome

This confirmed that:
- The application was correctly using the real credentials
- AWS authentication and permissions were properly configured
- The app’s logic for listing buckets via `boto3` was functioning as intended

At the same time, this test reinforced a key takeaway: **storing credentials directly in source code poses a major security risk**. Hardcoded secrets are vulnerable to accidental leaks, Git history exposure, and collaboration risks.

In the next phase, I transitioned away from static secrets and implemented **AWS Secrets Manager** for secure, encrypted, runtime credential retrieval.

![S3 Buckets Listed Successfully](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/d2270beec7e891c0f732ef0e9a33ed2f2444b152/assets/Screenshot%202025-06-23%20171218.png)


---



### Step 8: Simulating an Insecure Code Push and Observing GitHub Secret Scanning in Action

To better understand how GitHub protects against accidental credential leaks, I simulated a real-world scenario where insecure code with hardcoded AWS secrets is committed and pushed to a public repository.

#### 🔁 Forking the Repository

I signed into my GitHub account and forked the original repository into my account. Forking created a **public copy of the project**, effectively replicating how a developer might unknowingly publish sensitive information to a visible platform.

Unlike cloning, forking simulates a more realistic developer workflow, where contributions may include misconfigured or vulnerable files.

#### ⚙️ Initializing Git and Linking to GitHub

After making sure the app contained hardcoded credentials in `config.py`, I initialized a local Git repository and connected it to my fork:

**Initialize Git in the project directory:**

git init

**Link the local repo to the GitHub fork:**

git remote set-url origin `<your-forked-repo-url>`

#### 🧪 Staging and Committing Insecure Code

I then staged and committed all files, including `config.py` with real or placeholder secrets:

git add.  
git commit -m "Updated config.py with hardcoded credentials"

![Staging and Committing Insecure Code](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/11e48bfea060707ce64716fd774d8a5bd679ad5c/assets/Screenshot%202025-06-23%20174448.png)

#### 🚨 Attempting to Push to GitHub

Next, I attempted to push the code using:

git push origin main

But before the push was allowed to complete, GitHub intercepted the attempt. Because the commit contained what appeared to be **hardcoded AWS secrets**, GitHub's **Secret Scanning** feature flagged the push. It prevented the upload of insecure content and returned an error message like:

remote: ERROR: AWS secret key found in commit.
remote: Your push was rejected due to detected secrets. Please remove them and try again.


This step validated the power of GitHub’s built-in security features for protecting sensitive data, but also emphasized the importance of never storing secrets in code in the first place.

![GitHub Secret Scanning Rejection](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/d374b6d6fcfeb65927ae9a00338296d52646c6b4/assets/Screenshot%202025-06-23%20174344.png)


---



### Step 9: Creating and Configuring a Secure Secret in AWS Secrets Manager

To eliminate the risk of exposing sensitive AWS credentials in source code, I migrated the access keys to a **secure and encrypted secret** stored in **AWS Secrets Manager**. This ensures that secrets are dynamically retrieved at runtime rather than embedded in the application.

#### 🔐 Storing a New Secret

I logged into the **AWS Management Console**, navigated to **Secrets Manager**, and selected the option:

**Store a new secret**

- I chose **Other type of secret**, allowing me to define custom key-value pairs for the application.
- Entered the actual AWS credentials as:
  - `AWS_ACCESS_KEY_ID` → *pasted actual access key*
  - `AWS_SECRET_ACCESS_KEY` → *pasted actual secret key*

These fields were securely encrypted using **AWS Key Management Service (KMS)**, with the default key:  
**aws/secretsmanager**

This guarantees encryption at rest and access control via IAM policies.

#### 📝 Naming and Reviewing the Secret

I assigned a clear and descriptive name to the secret (e.g., `FastAPI_S3_Credentials`) so it could be referenced easily from my application.

Before finalizing the setup:
- Skipped optional settings like tags, resource-based policies, region replication, and automatic rotation.
- Reviewed all inputs to confirm that the configuration was accurate.

#### 📦 Capturing the Integration Snippet

Once the setup was confirmed, AWS provided a **Python code snippet** that demonstrates how to retrieve the secret securely using the **boto3** library.

This code snippet:
- Authenticates with Secrets Manager
- Retrieves the secret at runtime
- Returns a dictionary of key-value pairs
- Will replace the insecure `config.py` block containing hardcoded credentials

![Captured the Python sample code](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/d7f7aba2d0b50eb0502889e23b186e9feb40e18d/assets/Screenshot%202025-06-23%20180746.png)

#### ✅ Finalizing the Secret

I clicked **Store** to finalize creation. The secret was now:
- **Securely stored and encrypted**
- **Accessible only via proper IAM permissions**
- **Ready for application integration**


![Secrets Manager - Secure Secret Created](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/f795d2a00b5b86b6a51efc304eade6c9f423c0a7/assets/Screenshot%202025-06-23%20181221.png)



---


### Step 10: Integrating AWS Secrets Manager with config.py

After successfully creating and storing the secret in AWS Secrets Manager, the next step was to integrate it into the FastAPI application. The goal was to **dynamically fetch credentials at runtime**, removing any hardcoded values from the source code and aligning with security best practices.

#### 📥 Retrieving the Integration Code

From the AWS Secrets Manager dashboard:
- I selected my stored secret
- Clicked **“See sample code”**
- Switched to the **Python 3** tab
- Copied the complete code block beginning from the first `import` statement

This sample code includes everything needed to securely fetch the secret via the AWS SDK for Python (`boto3`).

#### 🧠 Updating `config.py` for Secure Runtime Retrieval

I opened the project in **Visual Studio Code** and navigated to `config.py`.

Steps taken:
- Replaced all existing static credential code with the AWS-provided sample
- Removed the placeholder comment `# Your code goes here` from inside the `get_secret()` function
- Reviewed and confirmed the structure of the integration

Key highlights of the updated logic:

- `import boto3` — Loads the AWS SDK for Python
- `get_secret()` — Function that securely retrieves credentials from Secrets Manager
- Uses `boto3.client('secretsmanager')` and specifies the region (`us-east-2`)
- Handles errors using a `try/except` block with `ClientError`
- Converts the retrieved JSON string into a usable Python dictionary

#### 🔐 Security Outcome

With this update:
- All AWS secrets are **fetched securely at runtime**
- No credentials are stored in version control
- The app is now compliant with modern **secret management and cloud security practices**

This change not only enhances security but also lays the foundation for:
- Seamless secret rotation
- Access control via IAM roles and policies
- Audit trails through AWS CloudTrail logging

![config.py – Integrated with AWS Secrets Manager](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/de5935068a62a7193db287d5693fb493969d707d/assets/Screenshot%202025-06-23%20182208.png)


---


### Step 11: Extract and Assign Secrets in config.py

After securely integrating AWS Secrets Manager into `config.py` with the `get_secret()` function, the next step was to **map the retrieved credentials to the variables used by the application** — without exposing any secrets in plain text.

#### 🧩 First: Import the JSON Module

To handle the JSON-formatted response returned by Secrets Manager, I added the following import statement at the top of the file:

import json

This enabled me to parse the retrieved secret into a usable Python dictionary.

#### 🧠 Second: Modify get_secret() to Return Parsed JSON

At the end of the `get_secret()` function, I updated the return statement to decode the response as a Python dictionary:

return json.loads(secret)

This ensured that calling `get_secret()` would now give me direct access to the AWS keys and region values stored in the secret.

#### 🔧 Third: Assign Retrieved Secrets to App Variables

I appended the following logic at the end of `config.py` to extract the credentials and assign them to application-level variables:


# Retrieve credentials from Secrets Manager
credentials = get_secret()

# Extract the values; if AWS_REGION isn't in the secret, fallback to the  default
AWS_ACCESS_KEY_ID = credentials.get("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = credentials.get("AWS_SECRET_ACCESS_KEY")
AWS_REGION = credentials.get("AWS_REGION", boto3.session.Session().region_name or "us-east-2")
This section ensures:

Keys are retrieved dynamically

No plaintext secrets are written in the codebase

The app remains functional even if the AWS region is missing by falling back to the default region from the current AWS session

✅ Outcome

My FastAPI app is now fully integrated with AWS Secrets Manager

All secrets are securely fetched at runtime

The app follows AWS best practices for secret management, allowing future implementation of rotation, logging, and access policies

This marks the transition from an insecure, hardcoded prototype to a production-grade, secret-aware application.


---


### Step 12: Pushing Final Secure Code and Cleaning Commit History

After successfully refactoring the application to use AWS Secrets Manager, I prepared to push the final secure version of the code to GitHub. However, previous commits in the repository still contained hardcoded AWS credentials, which triggered GitHub's secret scanning protections and blocked the push.

#### 🔐 First: Commit the Secure Version

I began by staging and committing the updated project files that no longer included any exposed secrets:

git add .  
git commit -m "Updated config.py with Secrets Manager credentials"

#### 🚫 Second: Push Blocked by GitHub Secret Scanning

When I attempted to push the changes to GitHub using:

git push -u origin main

GitHub rejected the push because the **repository history** still contained commits with leaked credentials — even though the latest version of the code was secure.

#### 🔎 Third: Identify the Problematic Commit

To fix the issue, I located the commit that introduced the hardcoded credentials:
- Scrolled through my Git history in the terminal
- Identified the **commit ID** where `config.py` first included plain text secrets
- Copied the **first 7 characters** of the commit hash for use in the next step

#### 🧹 Fourth: Start an Interactive Rebase

To clean the Git history, I used:

git rebase -i --root

This command opened a list of all commits from the start of the repository. I found the line referencing the commit that contained hardcoded credentials.

#### ✂️ Fiveth: Drop the Insecure Commit

In the rebase editor:
- I changed the word `pick` to `drop` in front of the offending commit
- Saved and closed the editor

This removed the insecure commit from the project’s version history.

#### 🚀 Sixth: Force Push Clean History

With the problematic commit removed, I force-pushed the updated history to GitHub:

git push origin main --force

This time, the push was successful - GitHub's secret scanning confirmed that no sensitive data remained in the code **or** commit history.

#### ✅ Final Result

The repository now contains only secure, production-ready code. No secrets are exposed, and the commit history is clean, preventing any trace of leaked credentials from being accessible.


![Secure force push success](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/2e3af6338c21c8341a162741f9b084263d2b0b38/assets/Screenshot%202025-06-23%20184342.png)  



---



### Step 13: Resolving Merge Conflicts and Pushing a Clean Commit History

During the process of cleaning my Git history via an interactive rebase, I encountered a **merge conflict in `config.py`**. This conflict was triggered by overlapping code changes: one commit included **hardcoded AWS credentials**, and another introduced the **secure version using AWS Secrets Manager**.

#### ⚠️ Conflict Detection

After running:

git rebase -i --root

Git paused the rebase process and flagged a conflict in `config.py`, displaying conflict markers like the following:

<<<<<<< HEAD
AWS_ACCESS_KEY_ID = "AKIAXXXXXXXXXXXXXXXX"

AWS_SECRET_ACCESS_KEY = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

AWS_REGION = "us-east-2"
import boto3
from botocore exceptions import ClientError
import json

def get_secret():
# ... secure code ...


#### 🛠️ Manual Conflict Resolution

To fix this securely:
- I **deleted the insecure block** containing hardcoded credentials (between `<<<<<<< HEAD` and `=======`)
- I **retained only the secure code** that fetches secrets using `boto3` and AWS Secrets Manager
- I removed all Git conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)

After verifying the result, I saved and closed the file.

#### ✅ Completing the Rebase and Push

I then staged and committed the resolved file:

git add config.py  
git commit -m "Resolved merge conflict in config.py"

To continue the rebase process, I ran:

git rebase --continue

Finally, I force-pushed the cleaned and secure commit history to GitHub:

git push --force

### Step 14: Final Verification – Clean Commit History

In this final step, I verified that the GitHub repository no longer contained **any commits with hardcoded AWS credentials**. This validation is critical to ensure that all sensitive information is completely removed, not only from the current code but also from the entire commit history.

#### 🔍 Verifying config.py

I navigated to my forked repository on GitHub and opened the `config.py` file in the **main branch**. Upon inspection, I confirmed:

- ✅ No hardcoded AWS keys or secrets
- ✅ Only secure integration code using **AWS Secrets Manager** was present

This confirmed that the working version of the file adhered to cloud security best practices.

#### 📜 Reviewing Commit History

To inspect historical changes, I clicked the **“History”** button at the top-right of the `config.py` file on GitHub.

I thoroughly reviewed each commit, ensuring that:
- No commit contained hardcoded credentials
- All commit messages reflected secure, relevant updates (e.g., "resolved conflict", "updated with Secrets Manager", etc.)

I also noticed that a few older commits from the **source repo** (before I forked it) included hardcoded keys,  but none of them belonged to me. This reinforced why it's critical to clean history **even after forking** from another source.

#### 🧼 Why This Step Matters

Simply removing secrets from the latest version of the code isn’t enough. If secrets still exist in prior commits, they can be extracted by anyone cloning or forking the repository. GitHub’s **secret scanning** can catch this, but it's our responsibility as developers to proactively remove such risks.

By **rebasing**, **resolving conflicts**, and **force-pushing a cleaned history**, I ensured that:

- No secrets exist in **any** commit
- The repository is now **safe for public sharing and collaboration**
- The project follows **secure development lifecycle (SDL)** standards

---

### ✅ Final Result

- The `config.py` file now securely retrieves credentials at runtime from **AWS Secrets Manager**
- No AWS keys or sensitive data remain in the codebase or commit history
- GitHub secret scanning is fully passed
- The project is production-ready, secure, and fully compliant with best practices for secret management

![Final Verification Screenshot](https://github.com/addula-mounika12/-Secure-Secrets-with-AWS-Secrets-Manager/blob/f6f3244b5493fe96321755189d8b01b5863fe5e1/assets/Screenshot%202025-06-23%20192537.png)


---


### Step 15: Deleting Resources to Clean Up

To wrap up the project and prevent unnecessary charges or leftover configurations, I carefully deleted all temporary resources created throughout the process. This ensures that the project concludes securely, cleanly, and cost-effectively.

---

#### 🗑️ 1. Deleted My Forked GitHub Repository

I began by removing my public repository to eliminate any remaining project artifacts from GitHub.

- Navigated to my **forked GitHub repo**
- Opened the **Settings** tab
- Scrolled to the **Danger Zone**
- Clicked **“Delete this repository”**
- Confirmed the action by typing the repo name when prompted

This step ensures no code containing prior test keys or project remnants remains publicly accessible.

---

#### 🔐 2. Deleted the AWS Secrets Manager Secret

Next, I removed the secret that held my AWS credentials.

- Visited the **AWS Secrets Manager Console**
- Selected **Secrets** from the left-hand menu
- Chose the secret (e.g., `aws-access-key`)
- Clicked **Delete** and confirmed the action

> AWS does not delete the secret immediately - instead, it schedules deletion with a recovery window (typically 7–30 days). If immediate deletion is needed, it can be done via the AWS CLI by modifying the recovery window.

---

#### 💻 3. Deleted the Local Repository

To clean up my development environment:

- Opened the terminal
- Navigated one directory above the project folder using:
  
  `cd ..`

- Deleted the local folder:

  `rm -rf your-project-folder-name`

This removed all files associated with the project from my machine.

---

#### 🧹 4. Optional: Deleted Residual AWS Resources

As a precaution, I performed a full audit across AWS services to ensure no test infrastructure remained:

- **IAM Console**: Checked for test users, access keys, and unused roles
- **EC2 Console**: Verified that all instances were stopped or terminated
- **S3 Buckets**: Deleted any temporary or test buckets created during development
- **CloudFormation**: Ensured no leftover stacks were still consuming resources


