# CodeCraftHub

CodeCraftHub is a beginner-friendly REST API for tracking programming courses you want to learn.

The project uses:

- Python
- Flask
- A JSON file for storage
- No database
- No authentication or user management

It is designed to help beginners learn:

- REST API endpoints
- HTTP methods
- JSON requests and responses
- HTTP status codes
- Input validation
- File-based data storage

---

## Features

- Create courses
- List all courses
- View one course
- Update a course
- Delete a course
- Automatically generate numeric course IDs
- Automatically record a `created_at` timestamp
- Store data in `courses.json`
- Automatically create `courses.json` if it does not exist
- Validate required fields
- Validate dates using `YYYY-MM-DD`
- Validate allowed course statuses
- Reject unknown fields
- Handle malformed JSON
- Handle missing or corrupted storage files
- Use atomic writes to reduce the risk of partially written files
- Return JSON error messages with appropriate HTTP status codes

---

## Requirements

Install the following software before starting:

- Python 3.9 or newer
- `pip`
- A terminal:
  - Windows PowerShell 7
  - macOS Terminal
  - Linux shell

Check your Python installation.

### Windows PowerShell

~~~powershell
python --version
~~~

### macOS/Linux

~~~bash
python3 --version
~~~

---

## Project structure

~~~text
codecrafthub/
├── app.py
├── courses.json
├── requirements.txt
├── .gitignore
└── README.md
~~~

### `app.py`

Contains the complete Flask application, including:

- API routes
- Input validation
- JSON file reading
- JSON file writing
- Error handling
- ID generation

### `courses.json`

Stores the course data.

Example:

~~~json
[
  {
    "id": 1,
    "name": "Flask REST APIs",
    "description": "Learn how to build REST APIs with Flask.",
    "target_date": "2026-12-31",
    "status": "Not Started",
    "created_at": "2026-10-13T12:00:00+00:00"
  }
]
~~~

If the file does not exist, the application creates it automatically with:

~~~json
[]
~~~

### `requirements.txt`

Lists the Python packages required by the project.

Example contents:

~~~text
Flask
~~~

Installing from this file ensures that all project dependencies are installed with one command.

### `.gitignore`

Lists files and directories that Git should not track.

Example contents:

~~~gitignore
.venv/
__pycache__/
~~~

The `.venv/` directory contains the local virtual environment, and `__pycache__/` contains Python-generated cache files. These files do not need to be committed to the project repository.

### `README.md`

Contains project documentation and usage instructions.

---

## Installation

### Windows PowerShell

Open PowerShell and clone the repository:

~~~powershell
git clone https://github.com/lgiacobazzi/codecrafthub.git
cd codecrafthub
~~~

Create a virtual environment:

~~~powershell
python -m venv .venv
~~~

Activate the virtual environment:

~~~powershell
.\.venv\Scripts\Activate.ps1
~~~

Install the project dependencies from `requirements.txt`:

~~~powershell
python -m pip install -r requirements.txt
~~~

You can optionally create an empty data file:

~~~powershell
Set-Content courses.json "[]"
~~~

The application will also create `courses.json` automatically if it does not exist.

---

### macOS/Linux

Open a terminal and clone the repository:

~~~bash
git clone https://github.com/lgiacobazzi/codecrafthub.git
cd codecrafthub
~~~

Create a virtual environment:

~~~bash
python3 -m venv .venv
~~~

Activate the virtual environment:

~~~bash
source .venv/bin/activate
~~~

Install the project dependencies from `requirements.txt`:

~~~bash
python3 -m pip install -r requirements.txt
~~~

You can optionally create an empty data file:

~~~bash
printf '[]\n' > courses.json
~~~

The application will also create `courses.json` automatically if it does not exist.

---

### Verify Flask installation

With the virtual environment activated, run:

#### Windows PowerShell

~~~powershell
python -m flask --version
~~~

#### macOS/Linux

~~~bash
python3 -m flask --version
~~~

---

## Running the application

### Windows PowerShell

~~~powershell
python app.py
~~~

### macOS/Linux

~~~bash
python3 app.py
~~~

The development server runs at:

~~~text
http://127.0.0.1:5000
~~~

The application uses a single-threaded development server:

~~~python
app.run(
    debug=True,
    threaded=False,
)
~~~

Keep the terminal running while testing the API.

To stop the application, press:

~~~text
Ctrl+C
~~~

---

## Course data model

A course contains the following fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | Integer | No | Automatically generated, starting at `1` |
| `name` | String | Yes | Course name |
| `description` | String | Yes | Course description |
| `target_date` | String | Yes | Date in `YYYY-MM-DD` format |
| `status` | String | Yes | Must be an allowed status |
| `created_at` | String | No | Automatically generated UTC timestamp |

Example course:

~~~json
{
  "id": 1,
  "name": "Flask REST APIs",
  "description": "Learn how to build REST APIs with Flask.",
  "target_date": "2026-12-31",
  "status": "Not Started",
  "created_at": "2026-10-13T12:00:00+00:00"
}
~~~

The `id` and `created_at` fields are generated by the server. Do not include them when creating or updating a course.

---

## Validation rules

### Required fields

Requests that create or update courses must include:

~~~text
name
description
target_date
status
~~~

`PUT` requires all four fields because it replaces the editable course data.

### Name

- Must be a string
- Must not be empty or contain only spaces

### Description

- Must be a string
- Must not be empty or contain only spaces

### Target date

- Must be a string
- Must be a real calendar date
- Must use the exact `YYYY-MM-DD` format

Valid:

~~~text
2026-12-31
~~~

Invalid:

~~~text
31-12-2026
2026-2-3
2026-02-30
~~~

### Status

The value must be exactly one of:

~~~text
Not Started
In Progress
Completed
~~~

These values are case-sensitive.

For example, this is invalid:

~~~text
started
~~~

### Unknown fields

Unknown fields are rejected.

For example, this request is invalid because `priority` is not supported:

~~~json
{
  "name": "Flask",
  "description": "Learn Flask.",
  "target_date": "2026-12-31",
  "status": "Not Started",
  "priority": "High"
}
~~~

---

## HTTP status codes

| Status | Meaning |
|---|---|
| `200 OK` | Successful GET or PUT request |
| `201 Created` | Course successfully created |
| `204 No Content` | Course successfully deleted |
| `400 Bad Request` | Invalid input, malformed JSON, missing fields, or wrong content type |
| `404 Not Found` | Course does not exist |
| `500 Internal Server Error` | Storage file cannot be read or written |

Successful responses return the course object or list directly.

They are not wrapped in a property such as:

~~~json
{
  "success": true,
  "course": {}
}
~~~

Errors use this format:

~~~json
{
  "error": "Description of the problem"
}
~~~

---

## API endpoints

The base URL is:

~~~text
http://127.0.0.1:5000
~~~

### Create a course

#### Request

~~~text
POST /api/courses
~~~

#### Windows PowerShell

Use `curl.exe`, not PowerShell's `curl` alias:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask REST APIs","description":"Learn how to build REST APIs with Flask.","target_date":"2026-12-31","status":"Not Started"}'
~~~

#### macOS/Linux

~~~bash
curl -i -X POST http://127.0.0.1:5000/api/courses \
  -H "Content-Type: application/json" \
  -d '{"name":"Flask REST APIs","description":"Learn how to build REST APIs with Flask.","target_date":"2026-12-31","status":"Not Started"}'
~~~

#### Successful response

Status:

~~~text
201 Created
~~~

Example:

~~~json
{
  "created_at": "2026-10-13T12:00:00+00:00",
  "description": "Learn how to build REST APIs with Flask.",
  "id": 1,
  "name": "Flask REST APIs",
  "status": "Not Started",
  "target_date": "2026-12-31"
}
~~~

The `created_at` value is generated dynamically and will be different on each request.

The response also includes a `Location` header:

~~~text
Location: /api/courses/1
~~~

---

### Get all courses

#### Request

~~~text
GET /api/courses
~~~

#### Windows PowerShell

~~~powershell
curl.exe -i -X GET http://127.0.0.1:5000/api/courses
~~~

#### macOS/Linux

~~~bash
curl -i -X GET http://127.0.0.1:5000/api/courses
~~~

#### Successful response

Status:

~~~text
200 OK
~~~

The response is a list directly:

~~~json
[
  {
    "created_at": "2026-10-13T12:00:00+00:00",
    "description": "Learn how to build REST APIs with Flask.",
    "id": 1,
    "name": "Flask REST APIs",
    "status": "Not Started",
    "target_date": "2026-12-31"
  }
]
~~~

If there are no courses:

~~~json
[]
~~~

---

### Get one course

#### Request

~~~text
GET /api/courses/<id>
~~~

#### Windows PowerShell

~~~powershell
curl.exe -i -X GET http://127.0.0.1:5000/api/courses/1
~~~

#### macOS/Linux

~~~bash
curl -i -X GET http://127.0.0.1:5000/api/courses/1
~~~

#### Successful response

Status:

~~~text
200 OK
~~~

~~~json
{
  "created_at": "2026-10-13T12:00:00+00:00",
  "description": "Learn how to build REST APIs with Flask.",
  "id": 1,
  "name": "Flask REST APIs",
  "status": "Not Started",
  "target_date": "2026-12-31"
}
~~~

---

### Update a course

#### Request

~~~text
PUT /api/courses/<id>
~~~

`PUT` requires all four editable fields:

- `name`
- `description`
- `target_date`
- `status`

#### Windows PowerShell

~~~powershell
curl.exe -i -X PUT http://127.0.0.1:5000/api/courses/1 -H "Content-Type: application/json" -d '{"name":"Advanced Flask REST APIs","description":"Build production-ready REST APIs with Flask.","target_date":"2027-01-31","status":"In Progress"}'
~~~

#### macOS/Linux

~~~bash
curl -i -X PUT http://127.0.0.1:5000/api/courses/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Advanced Flask REST APIs","description":"Build production-ready REST APIs with Flask.","target_date":"2027-01-31","status":"In Progress"}'
~~~

#### Successful response

Status:

~~~text
200 OK
~~~

~~~json
{
  "created_at": "2026-10-13T12:00:00+00:00",
  "description": "Build production-ready REST APIs with Flask.",
  "id": 1,
  "name": "Advanced Flask REST APIs",
  "status": "In Progress",
  "target_date": "2027-01-31"
}
~~~

The `id` and `created_at` values are preserved by the server.

---

### Delete a course

#### Request

~~~text
DELETE /api/courses/<id>
~~~

#### Windows PowerShell

~~~powershell
curl.exe -i -X DELETE http://127.0.0.1:5000/api/courses/1
~~~

#### macOS/Linux

~~~bash
curl -i -X DELETE http://127.0.0.1:5000/api/courses/1
~~~

#### Successful response

Status:

~~~text
204 No Content
~~~

There is no response body.

---

## Error examples

### Missing required fields

#### Windows PowerShell

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Incomplete Course"}'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "Missing required fields: description, status, target_date"
}
~~~

---

### Invalid status

#### Windows PowerShell

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask Course","description":"Learn Flask.","target_date":"2026-12-31","status":"Started"}'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "status must be one of: \"Not Started\", \"In Progress\", \"Completed\""
}
~~~

---

### Invalid target date

#### Windows PowerShell

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask Course","description":"Learn Flask.","target_date":"31-12-2026","status":"Not Started"}'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "target_date must be a valid date in YYYY-MM-DD format"
}
~~~

---

### Unknown field

#### Windows PowerShell

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask Course","description":"Learn Flask.","target_date":"2026-12-31","status":"Not Started","priority":"High"}'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "Unknown fields: priority"
}
~~~

---

### Malformed JSON

The closing `}` is intentionally missing:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask Course","description":"Learn Flask.","target_date":"2026-12-31","status":"Not Started"'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "Request body contains invalid JSON"
}
~~~

---

### Missing `Content-Type` header

#### Windows PowerShell

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -d '{"name":"Flask Course","description":"Learn Flask.","target_date":"2026-12-31","status":"Not Started"}'
~~~

#### Response

Status:

~~~text
400 Bad Request
~~~

~~~json
{
  "error": "Content-Type must be application/json"
}
~~~

---

### Course not found

#### Windows PowerShell

~~~powershell
curl.exe -i -X GET http://127.0.0.1:5000/api/courses/9999
~~~

#### Response

Status:

~~~text
404 Not Found
~~~

~~~json
{
  "error": "Course not found"
}
~~~

The same `404` response is returned when trying to update or delete a course that does not exist.

---

## Testing instructions

### Manual testing order

For an easy first test, run the commands in this order:

1. Start the server.
2. Create a course.
3. Get all courses.
4. Get the course by ID.
5. Update the course.
6. Delete the course.
7. Try to get the deleted course.
8. Run the error examples.

Start the server:

#### Windows PowerShell

~~~powershell
python app.py
~~~

#### macOS/Linux

~~~bash
python3 app.py
~~~

Then use the API commands from the previous sections.

### Reset the test data

To start again with no courses:

#### Windows PowerShell

~~~powershell
Set-Content courses.json "[]"
~~~

#### macOS/Linux

~~~bash
printf '[]\n' > courses.json
~~~

Restarting the server is not normally required after resetting the file.

### Verify a successful create

A successful create should return:

~~~text
201 CREATED
~~~

and a course object containing:

- `id`
- `name`
- `description`
- `target_date`
- `status`
- `created_at`

### Verify a successful delete

A successful delete should return:

~~~text
204 NO CONTENT
~~~

There should be no JSON response body.

---

## Troubleshooting

### Flask not found

#### Problem

You see an error such as:

~~~text
ModuleNotFoundError: No module named 'flask'
~~~

#### Solution

Activate the virtual environment first.

Windows PowerShell:

~~~powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
~~~

macOS/Linux:

~~~bash
source .venv/bin/activate
python3 -m pip install -r requirements.txt
~~~

You can verify the installation:

~~~powershell
python -m flask --version
~~~

or:

~~~bash
python3 -m flask --version
~~~

---

### Port 5000 is already in use

#### Problem

Another application is already using port `5000`.

#### Solution 1: Stop the other application

Find and stop the process using port 5000.

On Windows PowerShell:

~~~powershell
Get-NetTCPConnection -LocalPort 5000
~~~

On macOS/Linux:

~~~bash
lsof -i :5000
~~~

#### Solution 2: Use a different port

Temporarily change the final section of `app.py`:

~~~python
app.run(
    debug=True,
    threaded=False,
    port=5001,
)
~~~

Then use:

~~~text
http://127.0.0.1:5001
~~~

in your curl commands.

---

### `curl` behaves unexpectedly in PowerShell

PowerShell may map `curl` to another command depending on the environment.

Use `curl.exe` explicitly:

~~~powershell
curl.exe -i -X GET http://127.0.0.1:5000/api/courses
~~~

For POST and PUT requests, place the JSON body in single quotes:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:5000/api/courses -H "Content-Type: application/json" -d '{"name":"Flask","description":"Learn Flask.","target_date":"2026-12-31","status":"Not Started"}'
~~~

The `-i` option displays response headers, including the HTTP status code.

---

### PowerShell blocks virtual environment activation

#### Problem

You see an error similar to:

~~~text
running scripts is disabled on this system
~~~

#### Solution

Open PowerShell as your normal user and allow locally created scripts for your user account:

~~~powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
~~~

Confirm the change when prompted, then activate the environment again:

~~~powershell
.\.venv\Scripts\Activate.ps1
~~~

If you do not want to change the execution policy, you can run Python directly from the virtual environment:

~~~powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe app.py
~~~

---

### `courses.json` is corrupted

#### Problem

The application returns:

~~~text
500 Internal Server Error
~~~

with an error such as:

~~~json
{
  "error": "Course data is currently unavailable"
}
~~~

This can happen when `courses.json`:

- Contains invalid JSON
- Contains invalid text encoding
- Does not contain a top-level JSON array
- Contains a course without an `id`
- Contains invalid course ID data

The application does not automatically overwrite a corrupted file because doing so could destroy existing course data.

#### Solution

1. Stop the Flask server.
2. Make a backup copy of `courses.json`.
3. Open the file and repair the JSON manually.

For a fresh start, replace its contents with:

~~~json
[]
~~~

On Windows PowerShell:

~~~powershell
Set-Content courses.json "[]"
~~~

On macOS/Linux:

~~~bash
printf '[]\n' > courses.json
~~~

Then restart the application.

---

### A course ID was not found

Make sure you use an ID that exists:

~~~powershell
curl.exe -i -X GET http://127.0.0.1:5000/api/courses
~~~

The response lists all existing course IDs.

---

### PUT returns a missing fields error

`PUT` requires all four fields:

~~~json
{
  "name": "Course name",
  "description": "Course description",
  "target_date": "2026-12-31",
  "status": "Not Started"
}
~~~

Do not send only the field you want to change. The current implementation treats `PUT` as a complete replacement of the editable fields.

---

## How JSON file storage works

When the API needs course data:

1. It reads `courses.json`.
2. It parses the JSON.
3. It verifies that the top-level value is a list.
4. It verifies that each course is an object with an ID.
5. It returns the course data to the route.

When the API saves course data:

1. It creates a temporary file.
2. It writes the complete course list to the temporary file.
3. It flushes the data to disk.
4. It replaces `courses.json` using `os.replace()`.

This is called an atomic write. It helps prevent a partially written JSON file if the application stops during a write.

A corrupted file is not silently replaced. The API returns a `500 Internal Server Error` instead.

---

## Known limitations

This project is intentionally simple for learning REST API fundamentals.

- The application uses Flask's single-threaded development server.
- The JSON file is not suitable for concurrent production use.
- Multiple processes or servers could overwrite each other's changes.
- The JSON file does not provide database features such as transactions, indexes, or advanced queries.
- There is no authentication or user management.
- All users would access the same course list.
- IDs start at `1`.
- IDs are generated as one greater than the highest existing ID.
- If the last course is deleted, a later course can reuse that deleted ID.
- There is no pagination or filtering.
- There is no automated test suite included.
- Debug mode is intended for development only and should not be used in production.

For a production application, consider using:

- A production WSGI server
- A database such as SQLite or PostgreSQL
- Authentication and authorization
- Database transactions
- Concurrency-safe storage
- Automated tests
- Environment-based configuration
