### **FARM Stack Tutorial with Semantic Seed Studio TDD Standards**

This tutorial follows the Semantic Seed TDD (Test-Driven Development) standards, integrating FastAPI for the backend, React for the frontend, and MongoDB for the database. You'll start with test-driven development (TDD), following a cycle of writing tests, making them pass, and refactoring code for clarity and performance.

### **1. Project Setup and File Structure**

1. **Create Project Directories**:
   Run these bash commands to create the project structure and necessary files.

```bash
# Create the project root directory and navigate into it
mkdir farm-stack-todo
cd farm-stack-todo

# Create subdirectories for backend, frontend, and nginx
mkdir backend frontend nginx

# Create Docker and environment files in the project root
touch docker-compose.yml

# Backend setup: Create the backend directory structure
cd backend
python -m venv venv  # Create a virtual environment
source venv/bin/activate  # Activate the virtual environment
mkdir src  # Create a directory for source code

# Create backend-specific files
touch Dockerfile requirements.txt pyproject.toml
cd src
touch server.py dal.py

# Frontend setup: Navigate back to root and initialize React app
cd ../../frontend
npx create-react-app .

# Nginx setup: Create an nginx configuration file
cd ../nginx
touch nginx.conf

# Final directory structure should look like this:
# farm-stack-todo/
# ├── backend/
# │   ├── venv/
# │   ├── src/
# │   │   ├── server.py
# │   │   ├── dal.py
# │   ├── Dockerfile
# │   ├── pyproject.toml
# │   └── requirements.txt
# ├── frontend/
# ├── nginx/
# │   └── nginx.conf
# └── docker-compose.yml
```

2. **Set up Backend Environment**:
   Now activate your Python virtual environment and install dependencies:

```bash
# Install FastAPI, Motor, and other backend dependencies
pip install "fastapi[all]" "motor[srv]" beanie aiostream

# Generate the requirements.txt file
pip freeze > requirements.txt
```

---

### **2. Test-Driven Development (TDD) Approach for the Backend**

We'll follow the TDD process by first writing tests (Red), then implementing code to make the tests pass (Green), and finally refactoring for quality (Refactor).

---

#### **Step 1: Write Failing Tests for the Data Access Layer (DAL) - Red Tests**

Create the `test_dal.py` file in the `src` directory to define tests for the `dal.py` module:

```python
import pytest
from dal import ToDoDAL, ToDoList

@pytest.fixture
def mock_collection():
    # Create a mock MongoDB collection here
    pass

@pytest.fixture
def todo_dal(mock_collection):
    return ToDoDAL(mock_collection)

def test_list_todo_lists_empty(todo_dal):
    # Arrange
    mock_collection.find.return_value = []
    
    # Act
    result = list(todo_dal.list_todo_lists())

    # Assert
    assert result == []

def test_create_todo_list(todo_dal):
    # Arrange
    mock_collection.insert_one.return_value.inserted_id = "mock_id"

    # Act
    result = todo_dal.create_todo_list("New List")

    # Assert
    assert result == "mock_id"
```

Run the tests using `pytest`, and they will fail, confirming the Red stage:

```bash
pytest
```

Commit the failing tests:
```bash
git commit -m "WIP: Red tests for DAL"
```

---

#### **Step 2: Implement the DAL to Pass Tests - Green Tests**

Now that the tests are failing, implement the DAL to make them pass. Update `dal.py` with the following content:

```python
from bson import ObjectId
from motor.motor_asyncio import AsyncIOMotorCollection
from pymongo import ReturnDocument
from pydantic import BaseModel
from uuid import uuid4

class ToDoDAL:
    def __init__(self, todo_collection: AsyncIOMotorCollection):
        self._todo_collection = todo_collection

    async def list_todo_lists(self):
        async for doc in self._todo_collection.find(
            {}, projection={"name": 1, "item_count": {"$size": "$items"}}
        ):
            yield ListSummary.from_doc(doc)

    async def create_todo_list(self, name: str) -> str:
        response = await self._todo_collection.insert_one({"name": name, "items": []})
        return str(response.inserted_id)
```

Re-run the tests to ensure they pass, confirming the Green stage:

```bash
pytest
```

Commit the changes:
```bash
git commit -m "WIP: Green tests for DAL"
```

---

#### **Step 3: Refactor the DAL for Quality**

Refactor the code to improve readability and performance. Ensure all tests still pass after refactoring:

```bash
pytest
```

Commit the refactor:
```bash
git commit -m "Refactor complete: DAL implementation"
```

---

### **3. Implementing the FastAPI Server with TDD**

#### **Step 4: Write Failing Tests for FastAPI - Red Tests**

Before implementing the FastAPI server, write failing tests for the API. Create `test_server.py`:

```python
from fastapi.testclient import TestClient
from server import app

client = TestClient(app)

def test_get_all_lists_empty():
    # Act
    response = client.get("/api/lists")

    # Assert
    assert response.status_code == 200
    assert response.json() == []

def test_create_todo_list():
    # Act
    response = client.post("/api/lists", json={"name": "New List"})

    # Assert
    assert response.status_code == 201
    assert response.json()["name"] == "New List"
```

Run the tests and observe that they fail:

```bash
pytest
```

Commit the failing tests:
```bash
git commit -m "WIP: Red tests for FastAPI server"
```

---

#### **Step 5: Implement the FastAPI Server to Pass Tests - Green Tests**

Implement the FastAPI server in `server.py`:

```python
from fastapi import FastAPI, status
from dal import ToDoDAL, ListSummary, ToDoList

app = FastAPI()

@app.get("/api/lists")
async def get_all_lists():
    return [i async for i in app.todo_dal.list_todo_lists()]

@app.post("/api/lists", status_code=status.HTTP_201_CREATED)
async def create_todo_list(new_list: dict):
    return {"id": await app.todo_dal.create_todo_list(new_list['name']), "name": new_list['name']}
```

Re-run the tests and ensure they pass:

```bash
pytest
```

Commit the passing tests:
```bash
git commit -m "WIP: Green tests for FastAPI server"
```

---

#### **Step 6: Refactor the FastAPI Server for Quality**

Refactor the FastAPI server code for clarity and efficiency, then run the tests to ensure they still pass:

```bash
pytest
```

Commit the refactor:
```bash
git commit -m "Refactor complete: FastAPI server"
```

---

### **4. Integration Testing for Full Stack**

Write integration tests that ensure the backend and frontend interact correctly. Create `test_integration.py`:

```python
from fastapi.testclient import TestClient
from server import app

client = TestClient(app)

def test_create_list_and_item():
    # Arrange
    list_name = "Groceries"
    
    # Act - Create a list
    list_response = client.post("/api/lists", json={"name": list_name})
    list_id = list_response.json()["id"]

    # Act - Add an item to the list
    item_response = client.post(f"/api/lists/{list_id}/items/", json={"label": "Buy Milk"})
    
    # Assert
    assert list_response.status_code == 201
    assert item_response.status_code == 201
```

Run the integration tests:
```bash
pytest
```

---

### **5. Running the Application and Testing**

To build and run the application using Docker, execute the following commands:

```bash
docker-compose up --build
```

Visit `http://localhost:8000` in your browser to interact with the application.

---

### **6. Stopping the Application**

To stop the application:

- If running without Docker: press `Ctrl+C` in each terminal.
- If using Docker, press `Ctrl+C` in the terminal running `docker-compose up`, and stop the containers with:

```bash
docker-compose down
```

---

By following the Semantic Seed TDD workflow, you can ensure that every feature is developed with test coverage. The process of writing Red tests, passing them (Green), and refactoring for quality ensures that your code remains clean, maintainable, and scalable.
