## Study Sheet: Latest Era System Design - Part 2 (Advanced System Design with Node.js)

### 1. **Introduction to Advanced System Design**
In **Part 2**, we delve into more complex architectures and techniques often used in large-scale systems. This includes **serverless architectures**, **CQRS (Command Query Responsibility Segregation)**, **event sourcing**, and handling **distributed transactions**. These approaches are used to optimize performance, scalability, and fault tolerance in modern systems, particularly when working with **Node.js** in distributed systems.

---

### 2. **Serverless Architecture**

#### 2.1 **What is Serverless Architecture?**
Serverless architecture is a cloud-computing execution model where the cloud provider runs the server, and dynamically manages the allocation of machine resources. Developers only need to focus on code and logic, not on infrastructure.

- **AWS Lambda**, **Google Cloud Functions**, and **Azure Functions** are popular serverless platforms.
- **Node.js** works well in serverless environments due to its asynchronous nature.

#### 2.2 **Node.js in a Serverless Environment**
- Serverless applications run on a **function-as-a-service (FaaS)** model where each function handles a specific part of the logic.
  
**Example:**
- A simple Lambda function in **Node.js** to process an incoming request:
```javascript
exports.handler = async (event) => {
    const result = await processOrder(event.orderId);
    return { statusCode: 200, body: JSON.stringify(result) };
};
```
- Deploy the function via AWS Lambda or any FaaS provider, and it will automatically scale to handle millions of requests as needed.

#### 2.3 **Benefits of Serverless for Node.js**
- **Auto-scaling**: Automatically scales based on traffic.
- **Cost-efficiency**: You only pay for what you use.
- **High Availability**: Cloud providers ensure functions are distributed across multiple availability zones.

---

### 3. **Command Query Responsibility Segregation (CQRS)**

#### 3.1 **Overview of CQRS**
- **CQRS** is a pattern that separates the reading (query) and writing (command) responsibilities of a system.
- It is useful in complex systems where read and write workloads are fundamentally different and can be optimized separately.

#### 3.2 **CQRS in Node.js**
- **Commands** handle operations that modify the system state (e.g., updating a database).
- **Queries** handle operations that retrieve data (e.g., fetching data from a database or cache).

**Example:**
- **Command Service**: Responsible for handling user creation, updates, and deletion.
  ```javascript
  app.post('/users', (req, res) => {
      createUser(req.body)
      .then(result => res.status(201).json(result))
      .catch(err => res.status(500).send(err));
  });
  ```
- **Query Service**: Responsible for retrieving user data.
  ```javascript
  app.get('/users/:id', (req, res) => {
      getUserById(req.params.id)
      .then(user => res.status(200).json(user))
      .catch(err => res.status(500).send(err));
  });
  ```

#### 3.3 **Benefits of CQRS**
- **Optimized for Performance**: Reads and writes are handled separately, which allows each path to be optimized independently.
- **Scalability**: You can scale read services differently from write services, depending on the load.

---

### 4. **Event Sourcing**

#### 4.1 **What is Event Sourcing?**
- **Event Sourcing** is an architectural pattern where changes to the system state are stored as a series of events. Each event represents a fact that has happened in the system.
- Rather than storing the current state of an entity, event sourcing stores a history of changes.

#### 4.2 **Event Sourcing in Node.js**
- In an event-sourced system, the current state of an entity is reconstructed by replaying all its past events.

**Example:**
- Suppose we have a simple **bank account** system where each transaction (deposit or withdrawal) is stored as an event.
  
  **Event Representation:**
  ```json
  {
    "eventType": "deposit",
    "amount": 100,
    "timestamp": "2023-10-15T12:00:00Z"
  }
  ```

  **Reconstructing the Current Balance**: To get the current balance, we replay all the events (deposits and withdrawals) for an account and compute the final balance.

#### 4.3 **Benefits of Event Sourcing**
- **Auditability**: Full history of every change is stored.
- **Flexibility**: You can reconstruct the state of the system at any point in time.
- **Event-Driven**: Pairs well with event-driven architectures, enabling easy integration with other services or systems.

---

### 5. **Handling Distributed Transactions**

#### 5.1 **Challenges of Distributed Transactions**
In distributed systems, transactions often span multiple services or databases, making it difficult to maintain atomicity, consistency, isolation, and durability (ACID).

- Distributed transactions should avoid **locking** resources across multiple services as it creates bottlenecks.
  
#### 5.2 **Techniques to Handle Distributed Transactions**

##### **5.2.1 Two-Phase Commit (2PC)**
- **Two-Phase Commit** ensures all services involved in a transaction either commit or rollback together.

  **Example**:
  - **Phase 1**: The **prepare phase** where all services report they are ready to commit.
  - **Phase 2**: The **commit phase** where all services commit their changes.

  **Limitations**: It introduces blocking behavior and is prone to failures.

##### **5.2.2 Saga Pattern**
- **Saga Pattern** is a sequence of local transactions where each transaction triggers the next one. If one transaction fails, the previous transactions can be compensated or undone.
  
  **Example:**
  - **Scenario**: A payment service deducts money from a user’s account, and the inventory service reserves the item. If inventory fails, the payment service initiates a rollback (refund).
  
  **Node.js Example**:
  ```javascript
  async function handleOrder(order) {
      try {
          await deductPayment(order.userId, order.amount);
          await reserveItem(order.itemId);
      } catch (error) {
          await rollbackPayment(order.userId, order.amount);
      }
  }
  ```

#### 5.3 **Benefits of the Saga Pattern**
- **Resilient**: Each service handles its own rollback, making the system more fault-tolerant.
- **Asynchronous**: Sagas operate asynchronously, which is well-suited for distributed systems.

---

### 6. **Event-Driven Architecture with Kafka**

#### 6.1 **Kafka as an Event Streaming Platform**
- **Apache Kafka** is a distributed event streaming platform that enables high-throughput, low-latency event ingestion and processing.
- **Node.js** applications can use Kafka to build real-time, event-driven architectures.

#### 6.2 **Using Kafka in Node.js**
- **Kafka Producer**: Responsible for publishing events.
  ```javascript
  const { Kafka } = require('kafkajs');
  const kafka = new Kafka({ clientId: 'my-app', brokers: ['kafka-broker1'] });

  const producer = kafka.producer();
  await producer.connect();
  await producer.send({
      topic: 'order-created',
      messages: [{ value: JSON.stringify({ orderId: 123, status: 'created' }) }],
  });
  ```

- **Kafka Consumer**: Responsible for subscribing and processing events.
  ```javascript
  const consumer = kafka.consumer({ groupId: 'order-group' });
  await consumer.connect();
  await consumer.subscribe({ topic: 'order-created' });

  await consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
          const order = JSON.parse(message.value.toString());
          processOrder(order);
      },
  });
  ```

#### 6.3 **Benefits of Event-Driven Systems with Kafka**
- **Decoupled Services**: Producers and consumers are decoupled, allowing each service to scale independently.
- **High Throughput**: Kafka can handle millions of events per second.

---

### 7. **GraphQL in Node.js for Optimized Data Queries**

#### 7.1 **GraphQL Overview**
- **GraphQL** is a query language for APIs, enabling clients to request only the data they need.
  
#### 7.2 **Node.js with GraphQL**
- Using libraries like **Apollo Server** or **Express GraphQL** with Node.js allows developers to expose flexible APIs.
  
**Example:**
```javascript
const { ApolloServer, gql } = require('apollo-server');

const typeDefs = gql`
  type Query {
    user(id: ID!): User
  }
  type User {
    id: ID
    name: String
    age: Int
  }
`;

const resolvers = {
  Query: {
    user: (parent, args, context) => {
      return getUserById(args.id);  // Fetch user data from DB
    },
  },
};

const server = new ApolloServer({ typeDefs, resolvers });
server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

#### 7.3 **Benefits of GraphQL**
- **Efficient Data Fetching**: Clients request exactly what they need.
- **Single Endpoint**: All queries and mutations are handled via a single endpoint.
