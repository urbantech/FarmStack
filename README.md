# FarmStack
1. **TDD Workflow**:
    - Begin with writing tests that demonstrate the feature is not yet implemented (Red tests).
    - Implement code to make tests pass (Green tests).
    - Refactor code to improve quality (Refactor).
    - Commit changes as you progress (WIP commits for Red, Green, and Refactor stages).

2. **Test Case Structure**:
    - Follow the Arrange, Act, Assert (AAA) pattern in all test cases.
    - Group related tests using `describe` blocks for clarity.
    - Write clear, descriptive test names that specify the expected behavior.
    - Organize tests into suites (unit tests, integration tests, etc.).

---

### Updated Tutorial with Semantic Seed TDD Standard Formatting

---

### Project Setup and Backend Development with TDD

**Step 1: Set up the project structure** (Same as original)

---

**Step 2: Implement Data Access Layer (DAL)**

Before implementing the DAL, write failing tests to demonstrate that the desired functionality does not exist yet (Red tests).

#### Red Tests for `dal.py`

Create a test file `test_dal.py` in the `src` directory:

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

This tests two functions in the DAL (`list_todo_lists` and `create_todo_list`), ensuring that the initial implementation will fail. Commit this as a WIP (Red tests):

```bash
git commit -m "WIP: Red tests for DAL"
```

---

**Step 3: Implement DAL**

Now, write the code in `dal.py` to make the tests pass:

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

Run the tests and ensure they pass (Green tests). Commit the passing tests:

```bash
git commit -m "WIP: Green tests for DAL"
```

---

**Step 4: Refactor DAL**

Refactor the DAL to improve code quality, ensuring the tests still pass. Commit the refactored code:

```bash
git commit -m "Refactor complete: DAL implementation"
```

---

### Implementing the FastAPI Server with TDD

**Step 5: Write Tests for FastAPI Server**

Before implementing the FastAPI server, write tests to ensure the API endpoints don’t work yet.

Create a test file `test_server.py` in the `src` directory:

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

These tests will fail as the FastAPI server hasn’t been implemented yet. Commit this as a WIP (Red tests):

```bash
git commit -m "WIP: Red tests for FastAPI server"
```

---

**Step 6: Implement the FastAPI Server**

Now, implement the FastAPI server in `server.py` to make the tests pass:

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

Run the tests again, ensuring they pass (Green tests). Commit the passing tests:

```bash
git commit -m "WIP: Green tests for FastAPI server"
```

---

**Step 7: Refactor FastAPI Server**

Refactor the FastAPI server code to improve readability and performance, then commit the changes:

```bash
git commit -m "Refactor complete: FastAPI server"
```

---

### Integration Tests for the Full Stack

Write integration tests that simulate real-world usage of the application. These tests will involve running the backend and frontend together and verifying that the entire system works as expected.

Create `test_integration.py`:

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

These integration tests will verify that different components (DAL, FastAPI) interact correctly.

---

### Running and Testing the Application

At this point, you can run all tests (unit, integration, etc.) using the following command:

```bash
pytest
```

This command will run the tests in separate suites and ensure the correctness of all components.

---

By incorporating the Semantic Seed TDD workflow, we ensure that every feature is backed by tests from the very beginning. The process emphasizes writing tests first (Red), making them pass (Green), and refactoring for clarity and quality (Refactor).
