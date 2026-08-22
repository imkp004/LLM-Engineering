# Routers and Abstraction Layers

When building LLM applications, an application may need to interact with multiple models and providers.

For example:

```text
OpenAI
Anthropic
Google
Ollama
Open-Weight Models
```

Each provider can have different APIs, model names, request formats, and capabilities.

**Routers** and **abstraction layers** help solve this problem.

---

# Abstraction Layer

An **abstraction layer** hides implementation details behind a common interface.

Instead of writing completely different code for every LLM provider, an application can use a consistent interface.

Without an abstraction layer:

```text
Application
   ├── OpenAI-specific code
   ├── Anthropic-specific code
   ├── Google-specific code
   └── Ollama-specific code
```

With an abstraction layer:

```text
             Application
                  ↓
          Abstraction Layer
          ↙       ↓       ↘
      OpenAI  Anthropic  Ollama
```

The application communicates with the abstraction layer instead of directly depending on the implementation details of each provider.

---

# Why Abstraction Is Useful

Different providers can have different APIs.

For example, without an abstraction layer, the code might need to know:

```text
OpenAI API
    ↓
Specific endpoint
Specific request format
Specific authentication
Specific response format

Anthropic API
    ↓
Different endpoint
Different request format
Different authentication
Different response format
```

An abstraction layer can normalize these differences.

The application can work with a common concept such as:

```python
response = model.generate(messages)
```

The abstraction layer handles the provider-specific implementation underneath.

This creates **separation of concerns**.

```text
Application Logic
       ↓
LLM Abstraction
       ↓
Provider-Specific Implementation
       ↓
Model
```

---

# Router

A **router** is a component that decides **where a request should go**.

In an LLM application, a router can determine which model or provider should handle a particular request.

For example:

```text
                    Request
                       ↓
                     Router
                  ↙    ↓    ↘
              Model A Model B Model C
```

The router makes a routing decision based on rules or conditions.

---

# Example of LLM Routing

Suppose an application receives three types of requests:

```text
Simple question
Complex reasoning problem
Cheap high-volume request
```

The router could send them to different models:

```text
                    User Request
                         ↓
                       Router
                    ↙    ↓     ↘
                  ↓      ↓       ↓
             Small Model  Reasoning Model  Cheap Model
```

For example:

```text
Simple request
    ↓
Fast / inexpensive model

Complex coding problem
    ↓
More capable reasoning model

High-volume classification
    ↓
Small efficient model
```

The user does not necessarily need to know which model handled the request.

---

# Router vs Abstraction Layer

These concepts are related, but they perform different jobs.

### Abstraction Layer

Answers:

> **"How can the application communicate with different models using a common interface?"**

### Router

Answers:

> **"Which model or provider should handle this request?"**

A useful way to remember it:

```text
Abstraction
= Hides differences

Router
= Makes the choice
```

---

# Using Both Together

A real LLM application can use both.

```text
                     Application
                          ↓
                  Abstraction Layer
                          ↓
                       Router
                    ↙    ↓    ↘
                   ↓     ↓      ↓
                OpenAI Anthropic Ollama
                   ↓      ↓      ↓
                Model   Model   Model
```

The abstraction layer provides a consistent interface.

The router decides where the request should be sent.

The provider-specific implementation handles the actual API communication.

---

# Example

Suppose an application has three models:

```text
Model A → Fast and inexpensive
Model B → Highly capable
Model C → Local open-weight model
```

The application receives:

```text
"Summarize this short document."
```

The router might determine that Model A is sufficient.

```text
Request
   ↓
Router
   ↓
Model A
   ↓
Response
```

For:

```text
"Analyze this complex algorithm and identify potential flaws."
```

the router could select Model B.

```text
Request
   ↓
Router
   ↓
Model B
   ↓
Response
```

For sensitive data that must remain on local infrastructure:

```text
Request
   ↓
Router
   ↓
Local Model
   ↓
Response
```

---

# Routing Strategies

A router can make decisions using different criteria.

### 1. Model Capability

```text
Easy task → Small model
Hard task → Powerful model
```

### 2. Cost

```text
Low-cost request → Cheap model
High-value request → More expensive model
```

### 3. Latency

```text
Real-time request → Fast model
Non-urgent request → More capable model
```

### 4. Availability

If one provider is unavailable:

```text
Primary Model
     ↓
Unavailable
     ↓
Router
     ↓
Fallback Model
```

### 5. Data Requirements

```text
Sensitive Data
     ↓
Local / Private Model

Public Data
     ↓
Cloud Model
```

---

# Router as a Decision Layer

A router can therefore be thought of as a **decision-making layer** between the application and the models.

```text
                    Application
                         ↓
                    Routing Layer
                         ↓
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Model A      Model B     Model C
```

The router does not necessarily perform the inference itself.

It determines **which model should perform the inference**.

---

# Router vs Inference Profile

These concepts can sound similar, but they are not exactly the same.

An **inference profile** generally describes **how or where a particular model is invoked**, including the model configuration or deployment endpoint.

A **router** is responsible for **choosing among available models or endpoints**.

For example:

```text
Router
  ↓
Chooses Model B
  ↓
Inference Configuration/Profile
  ↓
Invokes Model B
  ↓
Inference
  ↓
Response
```

Therefore:

```text
Router
= Which model should handle this?

Inference Profile
= How/where should that model be invoked?
```

The exact meaning of "inference profile" can vary depending on the platform being used, so it should not be treated as a universal synonym for a router.

---

# Key Takeaways

* An **abstraction layer** hides provider-specific implementation details.
* A **router** decides which model or provider should handle a request.
* Abstraction allows applications to use a common interface.
* Routing allows applications to dynamically select different models.
* A router can consider capability, cost, latency, availability, or data requirements.
* Routers and abstraction layers can be used together.
* A router is about **selection**.
* An abstraction layer is about **hiding implementation differences**.

### Easy Mental Model

```text
Abstraction Layer
        ↓
"Don't worry about how each provider works."

Router
        ↓
"Choose which provider/model should handle this."

Inference
        ↓
"Actually run the model and generate the response."
```
