# FastAPI ToDo App Project

## Overview

This is a hands-on project built using **FastAPI**, a modern, high-performance Python framework for building APIs. The app is a minimal **ToDo API** designed to demonstrate the key advantages of using FastAPI over traditional frameworks like Flask or Django.

This README follows the tutorial by Travis Media titled *"Why You NEED To Learn FastAPI | Hands-On Project"*, where we build a ToDo application while showcasing 7 core reasons to learn and use FastAPI.

---

## Why FastAPI?

Modern software often requires APIs that are flexible, scalable, and easy to integrate with multiple platforms (web, mobile, desktop). FastAPI provides a clean solution:

* ✅ Maintain a **single backend** that supports multiple frontends
* ✅ Promote clean code separation (routes, models, logic)
* ✅ Boost productivity with built-in developer tools

> "FastAPI is like the USB port for your backend — once it's in place, anything can plug in and work."

---

## The 7 Advantages of FastAPI (Demonstrated in this Project)

1. **Plain Python Syntax**

   * FastAPI is intuitive and readable.
   * Route definitions are clean and require no boilerplate.

2. **Async Support Built-in**

   * FastAPI uses ASGI for asynchronous request handling out of the box.
   * Great for performance when handling concurrent I/O.

3. **Data Validation with Pydantic**

   * Define data models using `pydantic.BaseModel`
   * Automatically validate and parse input data

4. **Typed Python**

   * Use standard Python type hints for inputs and outputs
   * Improves readability, autocomplete, and documentation

5. **JSON Error Responses**

   * All errors return structured, developer-friendly JSON

6. **Built-in Authentication Support**

   * HTTP Basic Auth, OAuth2, JWT and API Keys

7. **Auto-generated Documentation**

   * Swagger UI: `/docs`
   * ReDoc: `/redoc`

---

##  Project Features

* Create, Read, Update, and Delete ToDo items
* All data is handled in-memory (for simplicity)
* Typed request/response models with `pydantic`
* Uses `uvicorn` as ASGI server

## Technologies Used

| Component       | Tech                                             |
| --------------- | ------------------------------------------------ |
| API Framework   | [FastAPI](https://fastapi.tiangolo.com/)         |
| Server          | [Uvicorn](https://www.uvicorn.org/)              |
| Data Validation | [Pydantic](https://pydantic-docs.helpmanual.io/) |
| Language        | Python 3.10+                                     |

---

## Folder Structure

```
fastapi-project/
├── app/
│   ├── main.py
│   └── models.py
└── README.md

```

---

## How to Run the Project

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install fastapi uvicorn

# Run the app
uvicorn main:app --reload
```

Visit [http://localhost:8000/docs](http://localhost:8000/docs) to access the Swagger UI.