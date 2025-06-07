# 📚 Library Cloud System – Books & Loans Microservices

This project implements a containerized microservice architecture for managing a digital library system. It includes:

- **BooksService** – for managing books and ratings
- **LoansService** – for managing book loans
- **MongoDB** – for persistent data storage
- **NGINX Reverse Proxy** – for routing, access control, and load balancing

All components are managed via **Docker Compose**.

---

## 📁 Project Structure
```.
├── BooksService/
│ ├── app.py
│ ├── Dockerfile
│ ├── requirements.txt
│ └── libraryFunctions/
│ ├── BooksCollection.py
│ ├── RatingsCollection.py
│ ├── Private.py 👈 Contains Gemini API key (see below)
├── LoansService/
│ ├── app.py
│ ├── Dockerfile
│ ├── requirements.txt
│ └── loans/
│ ├── Loan.py
│ └── LoansCollection.py
├── nginx/
│ └── nginx.conf
├── docker-compose.yml
```
---

## 🔧 Setup Instructions

### 1. Requirements

- Docker
- Docker Compose
- Internet access to use the Google Books API

### 2. Gemini API Key (BooksService only)

Although the Gemini API is no longer required in Assignment 2, the project structure expects the following file to exist:

Create `BooksService/libraryFunctions/Private.py` with the following content:

```python
# BooksService/libraryFunctions/Private.py
genai_key = "<your-gemini-api-key>"
⚠️ Do NOT upload this file to a public repository.
```
## 🧱 Services Overview

### 📘 BooksService

- **Endpoints**:
  - `POST /books`
  - `GET /books`, `GET /books/{id}`
  - `PUT /books/{id}`
  - `DELETE /books/{id}`
  - `GET /ratings`, `GET /ratings/{id}`
  - `POST /ratings/{id}/values`
  - `GET /top`
- **Responsibilities**:
  - Accepts book submissions with minimal fields and enriches them using the Google Books API.
  - Automatically creates and maintains ratings records for each book.

### 🔄 LoansService

- **Endpoints**:
  - `POST /loans`
  - `GET /loans`, `GET /loans/{loanID}`
  - `DELETE /loans/{loanID}`
- **Responsibilities**:
  - Validates borrowing rules:
    - A member may have up to 2 books on loan.
    - A book already on loan cannot be borrowed again.
  - Fetches book metadata from the BooksService.

---

## 🌐 Reverse Proxy (NGINX)

NGINX is used as a reverse proxy and load balancer:
- Public users can only perform `GET` and limited `POST` operations.
- Routes requests to BooksService and two instances of LoansService.
- Load balances between loans services using **weighted round-robin (3:1)**.

Configuration file is located at `nginx/nginx.conf`.

---

## ▶️ Running the System

Use Docker Compose to start all services:

```bash
docker-compose up --build
```
### Service access:
- Librarian interface: http://localhost:5001 – full access to BooksService
- Member interface: http://localhost:5002 – full access to LoansService
- Public interface: http://localhost/ – routed via NGINX (read-only access)

---

## 💾 Data Persistence & Reliability

- MongoDB is used for persistent storage across BooksService and LoansService.
- Each service persists its data in separate collections: `books`, `ratings`, and `loans`.
- No explicit volume is defined for MongoDB in `docker-compose.yml`. Docker will automatically assign a volume for each run, preventing interference between different student submissions.
- Services are configured with `restart: always` to automatically recover from failures.
- After recovery, services must behave as if no failure occurred.

---

## 🧪 Testing Tips

Use tools like `curl`, `Postman`, or automated tests to verify the system:

- `POST` requests should return status code `201 Created` along with the assigned ID.
- `GET`, `DELETE`, and `PUT` requests should return the appropriate status code and data.
- Error handling should return:
  - `404 Not Found` – when a resource is missing
  - `415 Unsupported Media Type` – when content type is incorrect
  - `422 Unprocessable Entity` – when request data is invalid
- Every response should include a brief success or error message in addition to the status code.

---

## 📌 Notes

- Each book and loan has a globally unique ID that is never reused, even if the original resource is deleted.
- Rating entries are automatically created and deleted with their corresponding book resources.
- The `/top` resource dynamically returns all books with the highest average ratings (min 3 ratings per book).
- The public API (via NGINX) allows only safe operations like `GET`, while internal ports (5001, 5002) offer full CRUD capabilities for librarians and members.
- Load balancing across LoansService instances is configured using NGINX with weighted round-robin (3:1 ratio).

---

## ✅ Summary

This project showcases:
- RESTful microservices design
- Inter-service communication using Docker networks
- API gateway pattern with NGINX (routing + access control)
- Load balancing and service replication
- Failure recovery and data persistence using MongoDB
- Deployment using Docker Compose for reproducibility and testing

Each microservice is independently testable and can scale horizontally for improved reliability and performance.

## Contact
For inquiries or feedback, feel free to reach out via the website's contact section.

[Live Website](www.itsrazo.com) | [GitHub Repo](https://github.com/razoDegree/Library-Cloud)
