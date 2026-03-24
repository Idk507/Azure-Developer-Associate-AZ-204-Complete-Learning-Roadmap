**Azure Service Bus Queues from a Developer’s Perspective (Python Implementation)**

From a developer’s view, Azure Service Bus Queues are just a **reliable mailbox in the cloud**.  
Your code (sender) puts a message in → Service Bus stores it safely → your other code (receiver) picks it up, processes it, and deletes it.  
No direct connection between sender and receiver needed. They can be in different apps, different servers, different times — even one can be offline.

You use the official **azure-servicebus** Python package (v7.x as of 2026).  
It’s super easy, supports both **sync** and **async** (I’ll show both), and works with two authentication methods:
- Connection string (easiest for learning)
- Microsoft Entra ID (passwordless — recommended for production)

### 1. End-to-End Architecture (Simple Diagram in Text)

```
[Producer App]                  [Azure Service Bus Queue]                  [Consumer App / Worker]
(e.g. Website / API)   ───Send───►   (Durable storage + FIFO)   ◄───Receive & Process───
- Python Flask/Django   - Stores messages safely           - Background worker
- Azure Function        - Peek-Lock (safe mode)           - Azure App Service / VM
- Mobile backend        - Dead-Letter Queue (DLQ)         - Processes order, payment, etc.
                          - Sessions / Duplicate detection
```

**Temporal decoupling** = Producer doesn’t wait for Consumer.  
**Load leveling** = 1000 orders come in fast → consumer processes slowly at its own pace.  
**Competing consumers** = 5 worker instances can all listen to the same queue → auto load balancing.

### 2. Complete Workflow (Message Life Cycle)

1. **Create infrastructure** (once in Azure Portal):
   - Service Bus Namespace (Standard or Premium tier)
   - Queue inside it (e.g. name: `orders-queue`)

2. **Sender flow**:
   - Connect to namespace
   - Create `ServiceBusMessage` (body + properties + optional session_id)
   - Send (single / list / batch)

3. **Queue stores** the message durably (even if your apps crash)

4. **Receiver flow** (default = Peek-Lock mode):
   - Connect and receive message(s)
   - Message is **locked** (no other consumer can take it)
   - Process the message (business logic)
   - If successful → `complete_message()` (delete forever)
   - If failed → `abandon_message()` (put back) or `dead_letter_message()` (move to DLQ)
   - If processing takes too long → lock expires → message returns automatically

5. **Monitoring** (Azure Portal or Azure Monitor):
   - Message count, dead-letter count, throughput

That’s the full end-to-end flow.

### 3. Real-World Use Case: E-commerce Order Processing

**Scenario**:
- User places order on website → order JSON is sent to queue.
- Separate worker processes payment, updates inventory, sends email.
- If worker fails 10 times → message goes to Dead-Letter Queue (you can fix it later).

This is a classic **load leveling + reliable background job** pattern.

### 4. Python Code Implementation (Complete & Runnable)

#### Step 0: Setup (Do Once)
```bash
pip install azure-servicebus
pip install azure-identity   # only for passwordless auth
```

**Get your connection string** (easiest way):
- Azure Portal → Your Namespace → Shared access policies → RootManageSharedAccessKey → Copy **Primary Connection String**

**OR** use passwordless (Entra ID) — assign “Azure Service Bus Data Owner” role to your user.

#### Sender Code (orders_sender.py) — Sync Version (Easiest to Understand)

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage
import json
import os

# === CONFIG ===
CONNECTION_STR = os.getenv("SERVICE_BUS_CONNECTION_STR")  # or paste directly
QUEUE_NAME = "orders-queue"

def send_order(order_data):
    message = ServiceBusMessage(
        body=json.dumps(order_data),           # JSON as string body
        content_type="application/json",
        properties={"Priority": "High"}        # your custom metadata
    )
    
    with ServiceBusClient.from_connection_string(CONNECTION_STR) as client:
        with client.get_queue_sender(QUEUE_NAME) as sender:
            sender.send_messages(message)
            print(f"✅ Order sent: {order_data['order_id']}")

# Example usage
if __name__ == "__main__":
    order = {
        "order_id": "ORD-12345",
        "customer": "Dhanush",
        "items": ["Laptop", "Mouse"],
        "total": 1299.99
    }
    send_order(order)
```

#### Receiver Code (orders_processor.py) — Sync Version with Continuous Loop

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage
import json
import os
import time

# === CONFIG ===
CONNECTION_STR = os.getenv("SERVICE_BUS_CONNECTION_STR")
QUEUE_NAME = "orders-queue"

def process_order(message_body):
    order = json.loads(message_body)
    print(f"🔄 Processing order {order['order_id']} for {order['customer']}")
    # Add your real logic here: payment, inventory, email...
    # If something goes wrong: raise exception or use dead_letter

with ServiceBusClient.from_connection_string(CONNECTION_STR) as client:
    with client.get_queue_receiver(QUEUE_NAME) as receiver:
        print("👀 Waiting for orders...")
        for msg in receiver:                     # This loop runs forever
            try:
                print(f"📥 Received: {msg}")
                process_order(msg.body)
                receiver.complete_message(msg)   # SUCCESS → delete
                print("✅ Order completed")
                
            except Exception as e:
                print(f"❌ Failed: {e}")
                receiver.dead_letter_message(msg, reason="Processing error", error_description=str(e))
                # OR receiver.abandon_message(msg) to retry later
```

**How to run end-to-end**:
```bash
python orders_sender.py
python orders_processor.py
```

You will see sender print “Order sent”, receiver instantly picks it up and processes.

#### Batch Sending (for high volume — better performance)

```python
with client.get_queue_sender(QUEUE_NAME) as sender:
    batch = sender.create_message_batch()
    for i in range(50):
        batch.add_message(ServiceBusMessage(f"Order batch {i}"))
    sender.send_messages(batch)
```

#### Async Version (for production — non-blocking)

Just change `azure.servicebus` → `azure.servicebus.aio` and use `async def` + `await`.  
The code is almost identical (see official docs for exact async examples).

### 5. Important Developer Concepts You Must Know

- **Receive Modes**:
  - Peek-Lock (default, safe) → use `complete_message`
  - Receive & Delete → risky (message lost if crash)

- **Dead-Letter Queue (DLQ)**:
  ```python
  # To read DLQ
  with client.get_queue_receiver(QUEUE_NAME, sub_queue="deadletter") as dlq_receiver:
      for msg in dlq_receiver:
          print("Dead lettered:", msg)
  ```

- **Sessions** (for strict ordering of related messages):
  Set `session_id` when sending and receive with same `session_id`.

- **Auto Lock Renewal** (if processing takes > lock duration):
  Use `AutoLockRenewer` class.

- **Scheduled Messages**:
  ```python
  message = ServiceBusMessage("Delayed message")
  message.scheduled_enqueue_time_utc = datetime.utcnow() + timedelta(minutes=30)
  ```

### Best Practices from Developer Experience

- Always use Peek-Lock + complete/abandon/dead-letter.
- Keep processing fast (< lock duration, default 1 min).
- Monitor DLQ regularly (use Azure Functions or Logic App).
- Use batch sending for performance.
- Prefer passwordless auth + least privilege roles.
- Store connection string in Azure Key Vault or environment variables.
- For exactly-once: enable Duplicate Detection on queue + use unique MessageId.

This is **complete end-to-end** — from architecture to running Python code with a real use case.  
You can copy-paste these scripts, create a queue, and have a working system in 10 minutes.

