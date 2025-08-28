# Json-Dataset-GroupBy-and-SortBy-Operator

## JSON Dataset Spring Boot Application

### 1. Project Overview
This Spring Boot application allows you to:  
- Insert JSON records into a dataset and persist them in a relational database.  
- Query JSON records with group-by and sort-by operations dynamically.

## Technology Stack
- Java 21  
- Spring Boot 3.x  
- Spring Data JPA  
- MySQL  
- WebClient for external API calls  
- JUnit 5 & Mockito for testing  
- Maven

## Database Configuration
MySQL is used as the relational database. Configure the `application.properties`:

    spring.datasource.url=jdbc:mysql://localhost:3306/json_dataset
    spring.datasource.username=root
    spring.datasource.password=your_password
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

---

## 2. API Endpoints

### 2.1 Insert Record API
**URL:**  
`POST /api/dataset/{datasetName}/record`  

**Path Parameters:**  
- `datasetName` – Name of the dataset (e.g., `employee_dataset`)

**Request Body:**  
Single Record:

    {
      "id": 1,
      "name": "John Doe",
      "age": 30,
      "department": "Engineering"
    }

Bulk Records:

    [
      {"id": 2, "name": "Jane Smith", "age": 25, "department": "Engineering"},
      {"id": 3, "name": "Alice Brown", "age": 28, "department": "Marketing"}
    ]

**Responses:**  
Single Record Insert Success:

    {
      "message": "Record added successfully",
      "dataset": "employee_dataset",
      "recordId": 1
    }

Bulk Insert Success:

    {
      "message": "2 records added successfully",
      "dataset": "employee_dataset"
    }

**Error Examples:**  
Empty JSON:

    {
      "message": "Validation failed",
      "details": "Empty JSON data not allowed"
    }

Malformed JSON:

    {
      "message": "Malformed JSON request",
      "details": "...exception details..."
    }

Invalid type (not Map/List):

    {
      "message": "Validation failed",
      "details": "Invalid request body. Must be a JSON object or an array of JSON objects."
    }

---

### 2.2 Query API
**URL:**  
`GET /api/dataset/{datasetName}/query`

**Query Parameters (optional):**  
- `groupBy` – Field name to group records (e.g., department)  
- `sortBy` – Field name to sort records (e.g., age)  
- `order` – Sort order: `asc` or `desc` (required if sortBy is used)

**Examples:**  
Group by department:

    GET /api/dataset/employee_dataset/query?groupBy=department

Response:

    {
      "groupedRecords": {
        "Engineering": [
          { "id": 1, "name": "John Doe", "age": 30, "department": "Engineering" },
          { "id": 2, "name": "Jane Smith", "age": 25, "department": "Engineering" }
        ],
        "Marketing": [
          { "id": 3, "name": "Alice Brown", "age": 28, "department": "Marketing" }
        ]
      }
    }

Sort by age ascending:

    GET /api/dataset/employee_dataset/query?sortBy=age&order=asc

Response:

    {
      "sortedRecords": [
        { "id": 2, "name": "Jane Smith", "age": 25, "department": "Engineering" },
        { "id": 3, "name": "Alice Brown", "age": 28, "department": "Marketing" },
        { "id": 1, "name": "John Doe", "age": 30, "department": "Engineering" }
      ]
    }

**Errors:**  
Invalid order:

    {
      "message": "Validation failed",
      "details": "Invalid order value. Use 'asc' or 'desc'."
    }

Missing field for groupBy/sortBy:

    {
      "message": "Validation failed",
      "details": "Field 'salary' not found in dataset: employee_dataset"
    }

Missing dataset:

    {
      "message": "Validation failed",
      "details": "No records found for dataset: unknown"
    }

No parameters provided:

    "Please provide either groupBy or sortBy parameter"

---

## Database Table
`dataset_record` is automatically created with fields:  
- `id`  
- `dataset_name`  
- `json_data` (TEXT)

---

## Running the Application

**Option 1: Using Maven**

    mvn clean install
    mvn spring-boot:run

**Option 2: Using IDE (Eclipse/IntelliJ)**  
- Right-click the main class `JsonDatasetApplication.java`  
- Run as Spring Boot App

The application will start at:  
http://localhost:8080

---

## Testing APIs using Postman

**Insert Single Record**  
Method: POST  
URL: `http://localhost:8080/api/dataset/record`  
Body: raw JSON (application/json)

    {"id":1,"name":"John Doe","age":30,"department":"Engineering"}

**Insert Bulk Records**  
Method: POST  
URL: `http://localhost:8080/api/dataset/records`  
Body: raw JSON array

    [
      {"id":2,"name":"Jane Smith","age":25,"department":"Engineering"},
      {"id":3,"name":"Alice Brown","age":28,"department":"Marketing"}
    ]

**Query with Group-By**  
Method: GET  
URL: `http://localhost:8080/api/dataset/query?groupBy=department`

**Query with Sort-By**  
Method: GET  
URL: `http://localhost:8080/api/dataset/query?sortBy=age&order=asc`

**Error Handling**  
- Empty JSON: 400 BAD REQUEST  
- Malformed JSON: 400 BAD REQUEST  
- Missing dataset / field: 400 BAD REQUEST  
- Invalid order parameter: 400 BAD REQUEST  
- Server errors: 500 INTERNAL SERVER ERROR
