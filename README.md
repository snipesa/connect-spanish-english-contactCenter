# Real-Time Chat Translation in Amazon Connect Using WebSockets

## 📌 Overview
This project enables **real-time chat translation** in **Amazon Connect's default agent workspace** using **API Gateway WebSockets** and **AWS Lambda**.

✅ **Works with Amazon Connect's default UI** (no custom agent workspace needed).
✅ **Supports real-time bidirectional translation** (English ↔ Spanish).
✅ **Ensures customers & agents only see translated messages**.

---

## 🏗️ Architecture Diagram
```
+-------------+      +---------------------+       +--------------------+       +-------------------+
|  Customer   | ---> | WebSocket API GW    | --->  | Lambda (Translate) | --->  | Amazon Connect UI |
| (Spanish)   |      | (Captures Messages) |       | (Translates Text)  |       | (Displays English)|
+-------------+      +---------------------+       +--------------------+       +-------------------+

+-------------+      +---------------------+       +--------------------+       +-------------------+
|  Agent      | ---> | WebSocket API GW    | --->  | Lambda (Translate) | --->  | Customer (Spanish)|
| (English)   |      | (Captures Messages) |       | (Translates Text)  |       | Sees only Spanish |
+-------------+      +---------------------+       +--------------------+       +-------------------+
```

---

## 🛠️ How It Works

1️⃣ **Amazon Connect starts a chat session** → Lambda registers session with **WebSocket API Gateway**.
2️⃣ **Customer sends a message (Spanish)** → WebSocket forwards to Lambda.
3️⃣ **Lambda translates message (Spanish → English)** → Returns translated text to **Amazon Connect UI**.
4️⃣ **Agent sends a message (English)** → WebSocket forwards to Lambda.
5️⃣ **Lambda translates message (English → Spanish)** → Returns translated text to **customer's chat**.

---

## 🚀 Setup Instructions

### 1️⃣ Create WebSocket API Gateway
1. Go to **AWS API Gateway** → **Create WebSocket API**.
2. Define **routes**:
   - `$connect` → Handles new connections.
   - `$disconnect` → Handles disconnections.
   - `$default` → Processes chat messages.
3. Deploy the WebSocket API.

### 2️⃣ Deploy Lambda Function
```python
import json
import boto3

translate = boto3.client('translate')
apigateway = boto3.client('apigatewaymanagementapi', endpoint_url="https://your-websocket-url")

def lambda_handler(event, context):
    connection_id = event['requestContext']['connectionId']
    body = json.loads(event['body'])
    original_text = body.get('message', '')
    sender_role = body.get('sender', '')  # "agent" or "customer"
    
    source_lang, target_lang = ("es", "en") if sender_role == "customer" else ("en", "es")
    translated_text = translate.translate_text(
        Text=original_text,
        SourceLanguageCode=source_lang,
        TargetLanguageCode=target_lang
    )['TranslatedText']
    
    response = {"translatedText": translated_text, "originalText": original_text, "sender": sender_role}
    apigateway.post_to_connection(ConnectionId=connection_id, Data=json.dumps(response))
    
    return {"statusCode": 200, "body": json.dumps("Message translated successfully")}
```

### 3️⃣ Modify Amazon Connect Contact Flow
1. **Invoke AWS Lambda Block** to register WebSocket session at chat start.
2. **Set Working Queue** to route chat to available agents.
3. **Transfer to Queue** → Agent receives translated messages in real-time.

---

## 🎯 Benefits of This Approach
✅ **No custom agent UI required** (uses default Amazon Connect chat interface).
✅ **Real-time chat translation without delays**.
✅ **Handles bidirectional translation seamlessly** (Spanish ↔ English).
✅ **Scales with multiple chat sessions dynamically**.

---

## 📌 Next Steps
Would you like:
✅ **A CloudFormation template to automate deployment?**
✅ **A working demo with sample chat UI?**
✅ **Further optimization for additional languages?**

Let me know how you'd like to proceed! 🚀

