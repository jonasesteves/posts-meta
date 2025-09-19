# Microservices Course Challenge 2 - Posting System with Messaging

Implement the system: **Posts**

You must develop a distributed system composed of two microservices that communicate asynchronously using RabbitMQ. The system will be responsible for receiving text posts, processing the content of these texts in the background, and storing the processing result.

The system must allow:

- Create a new post.
- Retrieve the details of a post.
- List posts with pagination.

The post processing includes counting the words in the text and calculating an estimated value based on the number of words.

---

## 1. Microservice: PostService

- Expose a REST API for creating and retrieving posts.
- Send new posts to the queue `text-processor-service.post-processing.v1.q`.
- Consume the processed results from the queue `post-service.post-processing-result.v1.q`.
- Store the post and the processing result.
- Use H2 for persistence.
- Generate the post ID with UUID.
- Correctly link the processed result to the original post.

### Endpoints

**POST /api/posts**

- Creates a new post.
- Sends post data to the queue.
- Returns `201 Created` and a `PostOutput` in the response body.

**GET /api/posts/{postId}**

- Returns `PostOutput`.
- Returns `200 OK` if successful.
- Returns `404 Not Found` if the post does not exist.

**GET /api/posts**

- Returns a paginated list of `PostSummaryOutput`.
- Supports parameters `page` and `size`.
- Returns `200 OK` with the structure:

```json
{
  "page": 0,
  "size": 10,
  "totalElements": 45,
  "totalPages": 5,
  "content": [ /* list of PostSummaryOutput */ ]
}
```

### DTOs used in PostService

**PostInput** – Model for creation:

```json
{
  "title": "string",
  "body": "string",
  "author": "string"
}
```

**PostOutput** – Detailed display model:

```json
{
  "id": "string",
  "title": "string",
  "body": "string",
  "author": "string",
  "wordCount": 123,
  "calculatedValue": 12.3
}
```

**PostSummaryOutput** – Simplified model:

```json
{
  "id": "string",
  "title": "string",
  "summary": "string",
  "author": "string"
}
```

### Validations in PostService

- `title`, `body`, and `author` are required.
- `body` must contain non-empty text.
- `id` must be UUID and unique.
- `summary` contains the first 3 lines of the body.

---

## 2. Microservice: TextProcessorService

- Consume the queue `text-processor-service.post-processing.v1.q`.
- Process the `body` field of the received post.
- Calculate:
    - The number of words in the post body.
    - The estimated value (`words * $0.10`).
- Send the processing result to the queue `post-service.post-processing-result.v1.q`.

### Message format consumed (queue `text-processor-service.post-processing.v1.q`)

```json
{
  "postId": "string", // UUID
  "postBody": "string"
}
```

### Message format produced (queue `post-service.post-processing-result.v1.q`)

```json
{
  "postId": "string", // UUID
  "wordCount": 123,
  "calculatedValue": 12.3
}
```

---

## 3. General Rules for Both Microservices

- Create the **Exchange** type as needed.
- Configure a **DLQ** for each queue.
- Configure queues and exchanges via Java.
- Always use DTOs for REST and messaging communications.
- Configure Jackson serializer for JSON in messages.
- To send a message directly to a queue without a named Exchange, use:

```java
rabbitTemplate.convertAndSend(queueName, messagePayload);
```

---

## 4. Challenge Tasks

- Implement **PostService** with REST endpoints and queue send/consume logic.
- Implement **TextProcessorService** with post queue consumer and result queue producer.
- Ensure correct persistence of data in **PostService**, including processed fields.
- Define input and output DTOs.
- Ensure HTTP responses are correct (`200`, `404`, `204`).
- Test the full flow with RabbitMQ running.

---

## Tips

- Use logs to record and debug message exchanges.
- Use the package model you prefer.
- Create Service classes if desired.  