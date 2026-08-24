# Tools and Tool Calling

## Overview

An LLM (Large Language Model) is primarily a system that processes text and generates text. By itself, it does not automatically have access to live external information or the ability to take actions in other systems.

For example, an LLM by itself cannot reliably:

- Check the current weather
- Look up a live stock price
- Query a company's database
- Send an email
- Create a calendar event
- Call a REST API
- Run a program
- Place an order

**Tools** give the LLM access to capabilities outside the model itself.

A useful mental model is:

```text
LLM = Brain
Tool = External capability / Action
Application = The system connecting them
```

The overall flow is:

```text
User
  ↓
LLM
  ↓
Decides a tool is needed
  ↓
Tool Call
  ↓
Application executes the tool
  ↓
External system / API / code
  ↓
Tool result
  ↓
LLM
  ↓
Final response
  ↓
User
```

The key idea is that the LLM generally **requests** a tool call; the application or tool runtime **executes** it.

---

# What Is a Tool?

A **tool** is a function, API, program, or external capability that an LLM-enabled application makes available to the model.

Examples include:

```text
Weather API
Search engine
Calculator
Database query
Python function
Calendar API
Email service
Payment API
Internal company system
```

A tool normally has three important pieces of information:

1. **Name** — what the tool is called
2. **Description** — what the tool does and when it should be used
3. **Parameters** — what information the tool needs

For example:

```text
Tool: get_weather

Description:
Get the current weather for a location.

Parameters:
location: string
```

The model uses this information to decide whether the tool is appropriate and what arguments to provide.

---

# Why Tools Are Needed

LLMs have an important limitation: the model's internal knowledge is not the same thing as live access to the world.

Suppose a user asks:

> What is the weather in Boston right now?

A model cannot simply rely on information learned during training because weather changes constantly.

Instead, a tool can retrieve current information:

```text
User question
     ↓
Need current weather
     ↓
Weather tool
     ↓
Live weather API
     ↓
Current conditions
```

The same idea applies to other tasks.

```text
Question about a company order
        ↓
Database tool

Question about today's news
        ↓
Search tool

Calculate 17,438 × 92
        ↓
Calculator tool

Book a meeting
        ↓
Calendar tool
```

---

# What Is Tool Calling?

**Tool calling** is the process where the LLM decides that a tool should be used and produces a structured request describing the tool and its arguments.

For example, the user says:

```text
"What is the weather in Boston today?"
```

The model may produce a tool call conceptually like:

```json
{
  "name": "get_weather",
  "arguments": {
    "location": "Boston, MA"
  }
}
```

This does **not** necessarily mean that the LLM itself executed the weather API.

It means:

> "I want the application to run `get_weather` using `Boston, MA`."

The application receives the request, executes the function or API call, and returns the result to the model.

---

# Real Example: Checking the Weather

Consider this conversation.

## Step 1: User Asks a Question

```text
User:
Should I bring an umbrella when I go to Boston today?
```

The application sends the user's message to the LLM along with the tools it has available.

Available tool:

```text
get_weather(
    location: string
)
```

The model reasons about the task:

```text
The user is asking about current weather.
Current weather changes over time.
A weather tool is available.
Therefore, use the weather tool.
```

---

# Step 2: LLM Requests the Tool

The model returns a structured tool call such as:

```json
{
  "name": "get_weather",
  "arguments": {
    "location": "Boston, MA"
  }
}
```

At this point, the model has not necessarily contacted the weather service itself.

The request is passed back to the application.

```text
LLM
 ↓
Tool Call
 ↓
Application
```

---

# Step 3: The Application Executes the Tool

The application looks at the tool name and arguments.

Conceptually, it might execute:

```python
def get_weather(location):
    # Call the real weather API
    return weather_api.get_current_weather(location)
```

The weather service might return:

```json
{
  "location": "Boston, MA",
  "temperature_f": 72,
  "condition": "Rain",
  "precipitation_probability": 80
}
```

The important point is that **the application executed the function**.

---

# Step 4: Tool Result Goes Back to the LLM

The application sends the tool result back into the conversation context.

Conceptually:

```text
User:
Should I bring an umbrella when I go to Boston today?

Assistant:
[Tool Call: get_weather("Boston, MA")]

Tool:
72°F, Rain, 80% chance of precipitation
```

Now the model has current information it did not have before.

---

# Step 5: LLM Produces the Final Answer

The model reads the original question and the tool result and generates a human-friendly answer:

```text
LLM:
Yes. Boston is expected to have rain today, with about an
80% chance of precipitation, so bringing an umbrella would
be a good idea.
```

The final conversation becomes:

```text
User:
Should I bring an umbrella when I go to Boston today?

LLM:
I need current weather information.

LLM → Tool:
get_weather("Boston, MA")

Tool:
72°F, Rain, 80% precipitation chance.

LLM:
Yes. Bring an umbrella; rain is expected today.
```

---

# The Important Separation of Responsibilities

One of the most important concepts is understanding **who does what**.

```text
                 USER
                   ↓
                LLM
          "Which tool do I need?"
                   ↓
              TOOL CALL
                   ↓
             APPLICATION
          "Execute the tool."
                   ↓
          EXTERNAL SYSTEM
                   ↓
              TOOL RESULT
                   ↓
                LLM
          "Understand the result."
                   ↓
            FINAL ANSWER
                   ↓
                 USER
```

The LLM is responsible for deciding what it needs and formatting the request.

The application is responsible for actually executing the tool.

The external system performs the actual operation or provides the data.

---

# Tool Definitions / Schemas

The LLM needs to know what tools are available.

A tool is commonly described using a schema.

For example:

```json
{
  "name": "get_weather",
  "description": "Get the current weather for a location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state or country"
      }
    },
    "required": ["location"]
  }
}
```

Think of this as a **menu for the LLM**.

The model sees:

```text
Tool:
get_weather

What it does:
Gets current weather

What it needs:
location
```

The model can then decide whether that tool matches the user's request.

---

# Tools Are Not the Same as Knowledge

This distinction is important.

An LLM can answer some questions from the knowledge encoded in its model parameters.

For example:

```text
User:
What is Docker?
```

A tool may not be necessary.

The model can generate an explanation from what it learned during training.

But consider:

```text
User:
What is the current price of Docker's stock?
```

Now live information may be required.

```text
Static knowledge
      ↓
Model may answer directly

Live information
      ↓
Use a tool
```

---

# Another Real Example: Bank Account Balance

Imagine an internal banking application with this tool:

```text
get_account_balance(account_id)
```

The user says:

```text
User:
How much money is in my checking account?
```

The LLM knows that the answer is not contained in its general knowledge.

It needs the user's actual account data.

The flow becomes:

```text
User
 ↓
"How much money is in my checking account?"
 ↓
LLM
 ↓
Request account balance tool
 ↓
Application authenticates the user
 ↓
Database / Banking System
 ↓
$2,450.17
 ↓
LLM
 ↓
"Your current checking balance is $2,450.17."
```

The LLM did not magically know the balance.

It obtained the information through the tool.

---

# A Tool Can Also Perform an Action

Tools are not only for retrieving information.

They can also **do something**.

For example:

```text
send_email()
create_calendar_event()
create_ticket()
charge_customer()
restart_server()
```

Consider:

```text
User:
Schedule a meeting with Sarah tomorrow at 2 PM.
```

The model may determine that a calendar tool is needed.

```text
LLM → create_calendar_event

{
  "title": "Meeting with Sarah",
  "date": "tomorrow",
  "time": "2:00 PM",
  "attendee": "Sarah"
}
```

The application executes the calendar API call.

The calendar service responds with something like:

```text
Event created successfully.
```

The LLM can then tell the user:

```text
Your meeting with Sarah has been scheduled for tomorrow at 2 PM.
```

This is how an LLM can move from **generating text** to **taking actions**.

---

# Tool Calling Is Different From the LLM Calling an API Directly

A common misunderstanding is:

> "The LLM called the API."

A more accurate mental model is:

```text
LLM
 ↓
Requests a tool call
 ↓
Application
 ↓
Executes API/function
```

The model typically does not have unrestricted access to your infrastructure.

The application decides which tools exist and controls how they are executed.

This is important for **security, permissions, validation, and reliability**.

---

# Multiple Tools

An application can provide many tools to the same model.

For example:

```text
Available Tools

1. get_weather()
2. search_web()
3. calculate()
4. get_customer()
5. send_email()
6. create_ticket()
```

The model chooses which one is appropriate based on the user's request.

Example:

```text
User:
What will the weather be in Boston tomorrow?
        ↓
      Weather
```

```text
User:
Search the web for the latest Terraform release.
        ↓
      Web Search
```

```text
User:
What is 19,284 × 73?
        ↓
      Calculator
```

```text
User:
Create a support ticket for my broken laptop.
        ↓
      Ticket System
```

---

# Multiple Tool Calls in One Task

A more advanced application can require multiple tools for one request.

Consider:

```text
User:
Find the cheapest flight from Boston to Dallas next Friday and put it on my calendar.
```

The application might provide:

```text
search_flights()
create_calendar_event()
```

The model may need to perform the workflow:

```text
User Request
     ↓
LLM
     ↓
search_flights()
     ↓
Flight Results
     ↓
LLM evaluates results
     ↓
create_calendar_event()
     ↓
Calendar Confirmation
     ↓
LLM
     ↓
Final Response
```

This shows why tool calling becomes the foundation of more advanced **AI agents**.

An agent can repeatedly decide:

```text
What do I know?
        ↓
What do I need?
        ↓
Which tool can provide it?
        ↓
Use the tool
        ↓
What should I do next?
        ↓
Repeat
```

---

# Tool Calling vs Function Calling

The terms **tool calling** and **function calling** are often used in very similar ways.

A function call can be thought of as a specific kind of tool call where the tool is represented as a function with structured arguments.

For example:

```python
def get_weather(location):
    ...
```

The model could request:

```json
{
  "name": "get_weather",
  "arguments": {
    "location": "Boston, MA"
  }
}
```

Different AI platforms use different terminology, but the basic idea is the same:

```text
LLM chooses a capability
        ↓
Provides structured arguments
        ↓
Application executes it
        ↓
Result goes back to LLM
```

---

# Why Structured Tool Calls Matter

Suppose the model simply wrote:

```text
I think you should call the weather function for Boston.
```

A program would have to guess what the model meant.

Structured output is much easier for software to handle:

```json
{
  "name": "get_weather",
  "arguments": {
    "location": "Boston, MA"
  }
}
```

Now the application can reliably determine:

```text
Tool = get_weather
Argument = Boston, MA
```

This is why tool calling systems use structured schemas.

---

# Tool Calling and Security

Giving an LLM tools also introduces security risks.

A tool such as:

```text
restart_server()
```

could have serious consequences if called incorrectly.

An even more sensitive example is:

```text
transfer_money()
```

The application should not blindly execute every request generated by the model.

A secure application can add:

- Authentication
- Authorization
- Input validation
- Permission checks
- Rate limits
- Confirmation steps
- Logging
- Audit trails
- Human approval

For example:

```text
User:
Transfer $10,000 to account XYZ.
        ↓
LLM
        ↓
Tool Call
        ↓
Application
        ↓
Check user permissions
        ↓
Require confirmation
        ↓
Execute transfer
```

The LLM should not be treated as a trusted security boundary.

The application must enforce permissions.

---

# Tools Turn an LLM Into a System That Can Act

Without tools:

```text
User
 ↓
LLM
 ↓
Text Response
```

With tools:

```text
User
 ↓
LLM
 ↓
Decision
 ↓
Tool
 ↓
External World
 ↓
Result
 ↓
LLM
 ↓
Response
```

This is a major shift.

The model is no longer limited to generating information from its internal parameters and conversation context.

It can now interact with external systems.

---

# Tools and AI Agents

Tool calling is one of the building blocks of **AI agents**.

A simple chatbot might do:

```text
User → LLM → Answer
```

An agent can do something more like:

```text
User
 ↓
LLM
 ↓
Decide what to do
 ↓
Tool
 ↓
Observe result
 ↓
Reason about result
 ↓
Another tool
 ↓
Observe result
 ↓
Final answer
```

The important difference is that an agent can perform a **multi-step loop** using tools rather than producing only a single response.

---

# Complete Mental Model

The easiest way to remember tool calling is:

```text
                USER
                  ↓
            "What do I need?"
                  ↓
                 LLM
                  ↓
      Can I answer without a tool?
             ↙          ↘
           YES           NO
            ↓             ↓
      Final Answer     Choose Tool
                            ↓
                        Tool Call
                            ↓
                       Application
                            ↓
                     Run Function/API
                            ↓
                       Tool Result
                            ↓
                           LLM
                            ↓
                     Final Answer
                            ↓
                           USER
```

The central idea is:

> **The LLM decides what tool it needs and provides the arguments; the application executes the tool and returns the result.**

That separation is fundamental to building reliable LLM applications.

---

# Key Takeaways

- A **tool** gives an LLM access to an external capability.
- Tools can retrieve information or perform actions.
- **Tool calling** allows the model to request a specific tool with structured arguments.
- The LLM usually does not execute the tool itself; the surrounding application does.
- The tool result is returned to the LLM as additional context.
- The LLM can then turn the raw tool result into a natural-language response.
- Tool schemas tell the model what tools exist, what they do, and what parameters they require.
- Multiple tools can be available at the same time.
- One task can require multiple tool calls.
- Tool calling is one of the core building blocks of **AI agents**.
- Because tools can perform real-world actions, authorization, validation, and other application-level security controls are essential.

## One-Line Summary

```text
LLM = decides what needs to happen
Tool = provides the capability
Application = safely executes the capability
Tool Result = gives the LLM the information needed for the next step
```
