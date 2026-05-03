# 🚀 Context-Aware HTML Generator (n8n + PostgreSQL)

## 📌 Overview

This project implements a **stateful AI-powered HTML generator chatbot** using **n8n, OpenAI, and PostgreSQL**.

Unlike basic prompt-based generators, this system supports **multi-turn conversations**, allowing users to iteratively refine and update previously generated HTML.

---

## 🧠 Architecture

```
Webhook (chat input)
   ↓
Check / Create Session
   ↓
Fetch Last HTML (PostgreSQL)
   ↓
IF (first prompt?)
   ├── Yes → Generate fresh HTML
   └── No  → Modify existing HTML
   ↓
OpenAI Node
   ↓
Store (prompt + updated HTML)
   ↓
Respond to Webhook
```

---

## ⚙️ Key Components

### 1. Webhook (Chat Input)

* Entry point for user requests
* Accepts JSON payload:

```json
{
  "session_id": "user123",
  "prompt": "Create a landing page"
}
```

---

### 2. Session Management

Handles user-specific state using `session_id`.

#### Responsibilities:

* Create new session if not exists
* Maintain continuity across prompts

---

### 3. PostgreSQL (State Storage)

Stores latest HTML and user prompt.

#### Table Schema:

```sql
CREATE TABLE sessions (
    session_id TEXT PRIMARY KEY,
    latest_html TEXT,
    last_prompt TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 4. First Prompt Detection

Determines whether:

* New HTML should be generated
* Existing HTML should be modified

#### Logic:

* If no record OR `latest_html` is NULL → First prompt
* Else → Modify existing HTML

---

### 5. OpenAI Processing

#### First Prompt:

Generates complete HTML page

#### Subsequent Prompts:

Modifies previously generated HTML based on new instructions

#### Example Prompt (Modify Mode):

```
Here is the existing HTML:
{{previous_html}}

User request:
{{prompt}}

Modify the HTML accordingly.
Return full updated HTML.
```

---

### 6. Data Persistence

Stores:

* Latest HTML (state)
* Last user prompt

Uses UPSERT logic:

```sql
INSERT INTO sessions (session_id, latest_html, last_prompt, updated_at)
VALUES ($1, $2, $3, NOW())
ON CONFLICT (session_id)
DO UPDATE SET
  latest_html = EXCLUDED.latest_html,
  last_prompt = EXCLUDED.last_prompt,
  updated_at = NOW();
```

---

### 7. Response Handling

Returns generated HTML directly to the client.

---

## 🔄 Workflow Behavior

### 🟢 First Request

* No existing session
* Generates fresh HTML

### 🔵 Follow-up Request

* Fetches previous HTML
* Applies incremental updates

---

## 💡 Design Decisions

### ✅ HTML as State

Instead of storing full chat history, only the latest HTML is stored.

**Benefits:**

* Faster processing
* Lower token usage
* Simpler architecture

---

### ✅ Stateless API + Stateful Backend

* API remains simple
* State handled via database

---

### ✅ Single Workflow Design

Avoids complexity of multiple workflows and ensures atomic operations.

---

## 🧪 Testing

### First Request

```json
{
  "session_id": "demo123",
  "prompt": "Create a portfolio website"
}
```

### Second Request

```json
{
  "session_id": "demo123",
  "prompt": "Add a contact form"
}
```

---

## 🚀 Future Enhancements

* Version history of HTML
* Live preview hosting
* Multi-page website generation
* Authentication & user management
* UI-based chatbot interface

---

## ⚠️ Best Practices

* Use **parameterized queries** (avoid SQL injection)
* Limit HTML size if scaling
* Add error handling for OpenAI failures
* Validate user input

---

## 🏁 Conclusion

This architecture provides a **scalable and efficient approach** to building a context-aware HTML generation system with minimal complexity and maximum flexibility.

It is well-suited for:

* Freelancing projects
* SaaS tools
* Rapid prototyping

---

## 📬 Contact

For improvements or collaboration, feel free to extend this project further.
