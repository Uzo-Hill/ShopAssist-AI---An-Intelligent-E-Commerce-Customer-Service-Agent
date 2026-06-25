# ShopAssist-AI-An-Intelligent-E-Commerce-Customer-Service-Agent

*![Intro_Image](https://github.com/Uzo-Hill/ShopAssist-AI---An-Intelligent-E-Commerce-Customer-Service-Agent/blob/main/Project_image/Architectural_Overview.jfif)*


An intelligent AI-powered e-commerce customer service agent that automatically resolves customer enquiries using real order data and company policies.

Built with CrewAI, MySQL, and Ollama, the agent can retrieve order information, apply company policies, and generate professional customer responses without human intervention.

---

## Project Overview

ShopAssist AI was built to demonstrate how AI Agents can automate customer support workflows by retrieving real business data and applying company policies before generating responses.

### Key Features

- AI-powered customer support automation
- Live order lookup from MySQL database
- Company policy retrieval
- Tool-calling with CrewAI
- Policy-compliant response generation
- Fully local deployment using Ollama
- No paid API required

The agent retrieves live order information from a MySQL database, checks company policies, and generates accurate customer support responses.

---
## Problem Statement

Traditional chatbots often rely on static rules or generic LLM knowledge, making it difficult to provide accurate responses to customer-specific enquiries.

ShopAssist AI addresses this challenge by:

1. Retrieving customer order information from a database.
2. Retrieving relevant company policies.
3. Passing retrieved information to the LLM.
4. Generating accurate and policy-compliant responses.

---

## Solution Architecture

```text
Customer Complaint
         │
         ▼
CrewAI Agent
         │
 ┌───────┴────────┐
 │                │
 ▼                ▼
Order Tool    Policy Tool
 │                │
 ▼                ▼
Orders DB    Policies DB
         │
         ▼
Retrieved Data
         │
         ▼
LLM (Llama 3.2)
         │
         ▼
Customer Response
```

---

## Agent Workflow

### 1. Customer Query

The customer submits an enquiry or complaint.

### 2. Order Retrieval

The Order Lookup Tool retrieves order details from database.

### 3. Policy Retrieval

The Policy Lookup Tool retrieves the relevant company policy.


### 4. Agent Reasoning

Retrieved order and policy information are passed to Llama 3.2 for reasoning.

### 5. Response Generation

The agent generates a professional customer support response.

---

## 🛠️ Tech Stack

| Component | Technology |
|------------|------------|
| Programming Language | Python |
| Agent Framework | CrewAI |
| LLM | Llama 3.2 |
| Model Serving | Ollama |
| Database | MySQL |
| Database Connector | mysql-connector-python |
| Validation | Pydantic |
| Environment Variables | python-dotenv |
| Development Environment | Jupyter Notebook |

---

## 🗄️ Database Design

### Orders Table

Stores customer order information.

**Fields Include:**

- Order ID
- Customer Name
- Item Purchased
- Order Date
- Expected Delivery Date
- Order Status
- Delay Reason
- Refund Eligibility

*![order_table](https://github.com/Uzo-Hill/ShopAssist-AI---An-Intelligent-E-Commerce-Customer-Service-Agent/blob/main/Project_image/order_table.PNG)*

### Policies Table

Stores official company policies used by the agent.

**Topics Include:**

- Refunds
- Late Deliveries
- Damaged Items
- Wrong Items
- Order Cancellations
- Processing Delays

*![policy_table](https://github.com/Uzo-Hill/ShopAssist-AI---An-Intelligent-E-Commerce-Customer-Service-Agent/blob/main/Project_image/policy_table.PNG)*


---

## Installation

### Clone Repository

```bash
git clone https://github.com/yourusername/ShopAssist-AI.git
cd ShopAssist-AI
```

### Install Dependencies

```bash
pip install -r requirements.txt
```
### Install Required Models

```bash
ollama pull llama3.2
```

---

## Running the Project

### Start Ollama

```bash
ollama serve
```

### Launch Jupyter Notebook

```bash
jupyter notebook
```
### Run the Notebook

Execute all notebook cells to initialize:

- Database connection
- Custom tools
- CrewAI agent
- Customer support workflow

---

## Core Implementation

### Database Connection

```python
# Try connecting to MySQL using the credentials from .env
connection = mysql.connector.connect(
    host=os.getenv("DB_HOST"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD"),
    database=os.getenv("DB_NAME")
)

# Run a simple test query
cursor = connection.cursor()
cursor.execute("SELECT COUNT(*) FROM orders")
result = cursor.fetchone()

print(f" Connected! Found {result[0]} orders in the database.")

cursor.close()
connection.close()
```

### LLM Configuration

```python
# Make sure Ollama is open and running on the computer

llm = LLM(
    model="ollama/llama3.2",
    base_url="http://localhost:11434"
)

print("Ollama LLM connected!")
```

### Order Lookup Tool

```python
# Step 1: Define what input the tool accepts
class OrderLookupInput(BaseModel):
    order_id: str = Field(..., description="The customer's order ID, e.g. ORD-1023")


# Step 2 : Build the tool that queries the orders table
class OrderLookupTool(BaseTool):
    name: str = "Order Lookup Tool"
    description: str = (
        "Looks up an order's status, item, delivery date, delay reason, "
        "and refund eligibility using the order ID. "
        "Use this whenever a customer mentions an order ID."
    )
    args_schema: Type[BaseModel] = OrderLookupInput

    def _run(self, order_id: str) -> str:
        # Connect to the database
        connection = mysql.connector.connect(
            host=os.getenv("DB_HOST"),
            user=os.getenv("DB_USER"),
            password=os.getenv("DB_PASSWORD"),
            database=os.getenv("DB_NAME")
        )
        cursor = connection.cursor(dictionary=True)  # Returns rows as dictionaries

        # Run the query, using %s to safely insert the order_id
        query = "SELECT * FROM orders WHERE order_id = %s"
        cursor.execute(query, (order_id.upper(),))

        order = cursor.fetchone()  # Get the matching row (or None)

        cursor.close()
        connection.close()

        # Return a clean summary, or an error message if not found
        if order:
            return (
                f"Order {order['order_id']} details:\n"
                f"  Customer: {order['customer_name']}\n"
                f"  Item: {order['item']}\n"
                f"  Status: {order['status']}\n"
                f"  Expected Delivery: {order['expected_delivery']}\n"
                f"  Reason: {order['reason']}\n"
                f"  Refund Eligible: {bool(order['refund_eligible'])}"
            )
        else:
            return f"❌ No order found with ID '{order_id}'. Please verify the order ID."

print("Order Lookup Tool built!")
```

### Policy Lookup Tool

```python
# Step 1 : Define what input the tool accepts
class PolicyLookupInput(BaseModel):
    topic: str = Field(..., description="Policy topic, e.g. 'refund', 'damaged item', 'wrong item'")


# Step 2 : Build the tool that queries the policies table
class PolicyLookupTool(BaseTool):
    name: str = "Policy Lookup Tool"
    description: str = (
        "Retrieves the company's official policy on a given topic, such as "
        "refunds, late delivery, damaged items, wrong items, or cancellations. "
        "Always check this before telling a customer what will happen next."
    )
    args_schema: Type[BaseModel] = PolicyLookupInput

    def _run(self, topic: str) -> str:
        connection = mysql.connector.connect(
            host=os.getenv("DB_HOST"),
            user=os.getenv("DB_USER"),
            password=os.getenv("DB_PASSWORD"),
            database=os.getenv("DB_NAME")
        )
        cursor = connection.cursor(dictionary=True)

        # Use LIKE to match the topic loosely (e.g. "refund" matches "refund")
        query = "SELECT topic, policy_text FROM policies WHERE topic LIKE %s"
        cursor.execute(query, (f"%{topic.lower()}%",))

        policy = cursor.fetchone()

        cursor.close()
        connection.close()

        if policy:
            return f"Policy on '{policy['topic']}':\n{policy['policy_text']}"
        else:
            return f"No specific policy found for '{topic}'. Escalate to a human agent."

print("Policy Lookup Tool built!")
```

### Agent Configuration

```python
# Instantiate both tools
order_tool = OrderLookupTool()
policy_tool = PolicyLookupTool()

# Build the agent
support_agent = Agent(
    role="Customer Service Representative",

    goal=(
        "Resolve customer enquiries and complaints accurately and politely, "
        "using real order data and company policy."
    ),

    backstory=(
        "You are a friendly, professional customer service representative "
        "for an online electronics store. You always check the order details "
        "and company policy before responding, and you never promise "
        "anything that goes against company policy."
    ),

    tools=[order_tool, policy_tool],
    llm=llm,
    verbose=True
)

print("Agent created!")
```

### Task Configuration

```python
# Simulated customer message — pick any order ID from order records
# New test case: damaged item complaint
customer_message = (
    "My order ORD-1015 arrived damaged, what can I do?"
)

task = Task(
    description=(
        f"A customer sent this message:\n"
        f'"{customer_message}"\n\n'
        f"1. Look up the order mentioned using the Order Lookup Tool.\n"
        f"2. Check the relevant policy using the Policy Lookup Tool.\n"
        f"3. Write a polite, clear reply stating what options the customer has.\n\n"
        f"IMPORTANT: Use the EXACT dates, reasons, and policy details "
        f"returned by the tools. Do not change, round, or guess any "
        f"figures — copy them exactly as given."
    ),

    expected_output=(
        "A polite customer service reply referencing the order status "
        "and clearly stating the customer's options (replacement or refund) "
        "based on policy."
    ),

    agent=support_agent
)

print("Task updated!")
```

### Crew Execution

```python
# Bring agent and task together and run
crew = Crew(
    agents=[support_agent],
    tasks=[task],
    verbose=True
)

# Run asynchronously to avoid the Jupyter event loop clash
async def run_crew():
    result = await crew.kickoff_async()
    print("\n========== FINAL CUSTOMER REPLY ==========")
    print(result)

await run_crew()
```

---

## Application Demo

### Customer Complaint


### Tool Execution

*![Tool_Execution](https://github.com/Uzo-Hill/ShopAssist-AI---An-Intelligent-E-Commerce-Customer-Service-Agent/blob/main/Project_image/Tool_Execution.PNG)*

### Final Customer Response

*![AgentResponse](https://github.com/Uzo-Hill/ShopAssist-AI---An-Intelligent-E-Commerce-Customer-Service-Agent/blob/main/Project_image/Final_Answer.PNG)*

---

## Evaluation

The system successfully retrieves customer order information and company policies before generating grounded customer support responses.

### Example

**Customer Query**

```text
My order ORD-1015 arrived damaged, what can I do?
```

**Response Summary**
```text
Order details retrieved successfully.

Status: Delivered - Damaged

The customer was offered:
- Free replacement with expedited shipping
- Full refund
```

This demonstrates that responses are generated using retrieved business data and company policies rather than generic model knowledge.

---

## Key Learnings

- AI Agent Development
- CrewAI Framework
- Tool Calling
- Database Integration
- MySQL Querying
- Prompt Engineering
- Local LLM Deployment
- Policy-Based AI Systems

---

## Future Improvements

- Multi-agent architecture
- Customer authentication
- Web-based chat interface
- WhatsApp integration
- Email support automation
- CRM integration
- Human escalation workflow

---

## ⚠️ Disclaimer

This project is intended for educational and demonstration purposes only.

Customer and order records used in this project are simulated and do not represent real customer information.

---

## Author

**Hillary C. Uzoh**

- GitHub: https://github.com/Uzo-Hill
- LinkedIn: https://www.linkedin.com/in/hillaryuzoh/















