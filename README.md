## Setup

### 1. X Requirements

To use the X API in this workflow, the following setup is required:

#### 1.1 X Account

*   A standard X account is required.
  
*   **Email verification** is mandatory.
    
*   **Mobile phone number verification** is mandatory.  
    X does not allow API access for accounts without a verified phone number.
 

#### 1.2 X Developer Account

*   Create an X Developer account.
    
*   Create a new **App** under the developer dashboard.
     <img width="1188" height="571" alt="image" src="https://github.com/user-attachments/assets/1055609b-3111-45ab-a4b4-20b2fd8463d0" />

    
*   Enable **OAuth 2.0 (Authorization Code Flow)**.
    <img width="1275" height="423" alt="image" src="https://github.com/user-attachments/assets/c7e79781-e60a-449d-a985-11ee1ec841a0" />
    <img width="892" height="689" alt="image" src="https://github.com/user-attachments/assets/c3bc6207-e418-4375-9995-a9411c497f7d" />

    
*   Obtain:
    *   Client ID
        
    *   Client Secret
        
    *   Callback URL (must be HTTPS)
        <img width="1140" height="683" alt="image" src="https://github.com/user-attachments/assets/6c005607-de4f-485c-8c4e-91d549abc377" />
        <img width="357" height="687" alt="image" src="https://github.com/user-attachments/assets/49fc0770-5644-4500-9fb8-073046299f97" />


#### 1.3. X Limitations
1. [X API v2 - X](https://docs.x.com/x-api/introduction)
2. [Rate limits - X](https://docs.x.com/x-api/fundamentals/rate-limits?utm_source=chatgpt.com)

For **production use** of the X API, a **paid (Pro) subscription is required**.
This is because:
*   **Direct Messages (DMs)** access is restricted to paid tiers.
    
*   **Higher API rate limits** needed for production workloads are only available on paid plans.
    
*   Free or basic access tiers are **not suitable for automated or scalable workflows**.
    
* * *

### 2. n8n Requirements

#### 2.1 HTTPS Requirement (Mandatory)

*   n8n **must be accessible via a public HTTPS URL**.
    
*   Localhost or HTTP-only deployments will **not work** with X OAuth.
    
*   Self-signed certificates are not supported.
    
Valid options include:
*   n8n Cloud

*   ngrok
    
*   VPS deployment with Nginx + Let’s Encrypt
    
*   Reverse proxy with a trusted TLS certificate
    




#### 2.2 n8n Credentials Configuration

Create the following credentials in n8n:

##### X OAuth2 Credential

*   Credential type: **X OAuth2 API**
    
*   OAuth flow: Authorization Code
    
*   Required fields:
    *   Client ID
        
    *   Client Secret
        
    *   OAuth Callback URL (from your HTTPS n8n instance)

<img width="1205" height="634" alt="image" src="https://github.com/user-attachments/assets/6b68d059-fa0e-4e67-bb1f-54a69e54319f" />

        
This credential is reused X APIs

* * *

### 3. Workflow (Draft Trial)

#### 3.1 Workflow

<img width="1309" height="268" alt="image" src="https://github.com/user-attachments/assets/83b7c3c0-62f2-48ca-a8b6-e3aa1eda12bc" />




#### 3.2 Workflow Behavior

*   The workflow polls mentions on a schedule.
    
*   Each mention is processed.
    
*   Replies are posted as **direct threaded replies** using `in_reply_to_tweet_id`.

   
#### 3.3 Workflow Details

**Steps Breakdown**

1.  Set Up:
    *   This node initializes variables for the workflow. It assigns values like the username (`doc_and_tell`).
        
2.  Get User ID:
    *   This node makes an HTTP request to the X API to retrieve the user ID of the `@doc_and_tell` account.
        
    *   The request is authenticated using predefined credentials.
        
3.  Get Mentions:
    *   Once the user ID is fetched, this node queries the X API to fetch all mentions for the `@doc_and_tell` account.

    *   The request is authenticated using predefined credentials.
        
4.  Extract Mentions Data:
    *   This node processes the mention data extracted in the previous step and prepares it for further processing.

    *   The request is authenticated using predefined credentials.
        
5.  Loop Over Items:
    *   This node loops over the mentions and handles them in batches.
        
6.  Basic LLM Chain:
    *   The workflow uses a predefined LLM prompt to generate supportive replies to the mentions retrieved from X.
        
    *   The prompt is designed to ensure the reply follows specific output rules such as escaping line breaks and outputting plain text.
        
7.  Groq Chat Model:
    *   This node interacts with the Groq chat model for further processing. It integrates with the Basic LLM Chain to refine the generated replies.
        
8.  Reply in Thread:
    *   This node sends the generated reply back to the X API, posting the response to the thread where the mention occurred.

     *   The request is authenticated using predefined credentials.
        
9.  Schedule Trigger:
    *   The workflow is triggered at specified intervals (scheduled) to continuously monitor for new mentions.
    * It works after publishing the workflow.

**Connections**

*   Set Up → Get User ID → Get Mentions → Extract Mentions Data → Loop Over Items → Basic LLM Chain → Groq Chat Model → Reply in Thread
    
*   The "Schedule Trigger" starts the workflow and connects to **Set Up**, initiating the entire process

**Credentials**

*   **X OAuth2 API Credentials**: Used for authenticating the requests to the X API for fetching user information, mentions, and posting replies.
    
*   **Groq API**: Used for interacting with the Groq chat model.

* * *

### 4. Extension & Future Improvements (Notes)

Recommended extensions for production use:
*   Store replied tweet IDs (to avoid duplicate replies)
    
*   Filter mentions that already have replies

*   Add DMs processing too.
    
*   Add rate-limit handling and retry logic
    
*   Add moderation or safety checks before posting
    
*   Persist workflow state in a database
    
These can be implemented without changing the core setup above.
