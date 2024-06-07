# DbAdmin:UX Project Guide

This guide will walk you through creating a DbAdmin UX for SQLite processes, which involves designing an intuitive user interface for database administration tasks. This tool will facilitate easy management of databases, including operations like creating, updating, and deleting records, running queries, and visualizing data.

## Prerequisites

- Basic knowledge of HTML, CSS, JavaScript, and Lua.
- Node.js and npm installed.
- AOS terminal access.

## Project Structure

Here's the structure of our project:

dbadmin-ux/

├── backend/

│ ├── main.lua

├── frontend/

│ ├── index.html

│ ├── styles.css

│ ├── script.js

├── README.md


## Step 1: Setting Up the Frontend

Create a new directory named `frontend` and create the following files:

### 1. index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DbAdmin UX</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>DbAdmin UX for SQLite</h1>
    <div class="container">
        <div class="form-group">
            <h2>Create Table</h2>
            <input type="text" id="tableName" placeholder="Table Name">
            <input type="text" id="columns" placeholder="Columns (e.g. id INTEGER PRIMARY KEY, name TEXT)">
            <button onclick="createTable()">Create Table</button>
        </div>

        <div class="form-group">
            <h2>Insert Record</h2>
            <input type="text" id="insertTableName" placeholder="Table Name">
            <input type="text" id="values" placeholder="Values (e.g. 1, 'John Doe')">
            <button onclick="insertRecord()">Insert Record</button>
        </div>

        <div class="form-group">
            <h2>Update Record</h2>
            <input type="text" id="updateQuery" placeholder="Update Query">
            <button onclick="updateRecord()">Update Record</button>
        </div>

        <div class="form-group">
            <h2>Delete Record</h2>
            <input type="text" id="deleteQuery" placeholder="Delete Query">
            <button onclick="deleteRecord()">Delete Record</button>
        </div>

        <div class="form-group">
            <h2>Run Select Query</h2>
            <input type="text" id="selectQuery" placeholder="Select Query">
            <button onclick="runSelectQuery()">Run Query</button>
            <div id="queryResult"></div>
        </div>
    </div>
    <script src="script.js"></script>
</body>
</html>
```
### 2. styles.css
```css
    body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 20px;
  }

  h1 {
    text-align: center;
  }

  .container {
    max-width: 600px;
    margin: 0 auto;
  }

  .form-group {
    background-color: #fff;
    padding: 20px;
    margin-bottom: 20px;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }

  .form-group h2 {
    margin-top: 0;
  }

  .form-group input {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    border: 1px solid #ddd;
    border-radius: 5px;
}

.form-group button {
    padding: 10px 20px;
    background-color: #28a745;
    border: none;
    color: white;
    border-radius: 5px;
    cursor: pointer;
  }

  .form-group button:hover {
    background-color: #218838;
  }
```
### 3. script.js
```js
   async function sendMessage(action, tags) {
    const message = {
        Target: 'your-process-id',
        Tags: {
            Action: action,
            ...tags
        }
    };

    const response = await fetch('https://your-aos-endpoint/send', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(message)
    });

    return await response.json();
}

function createTable() {
    const tableName = document.getElementById('tableName').value;
    const columns = document.getElementById('columns').value;

    sendMessage('CreateTable', { TableName: tableName, Columns: columns })
        .then(response => alert(response))
        .catch(error => console.error(error));
}

function insertRecord() {
    const tableName = document.getElementById('insertTableName').value;
    const values = document.getElementById('values').value;

    sendMessage('InsertRecord', { TableName: tableName, Values: values })
        .then(response => alert(response))
        .catch(error => console.error(error));
}

function updateRecord() {
    const query = document.getElementById('updateQuery').value;

    sendMessage('UpdateRecord', { Query: query })
        .then(response => alert(response))
        .catch(error => console.error(error));
}

function deleteRecord() {
    const query = document.getElementById('deleteQuery').value;

    sendMessage('DeleteRecord', { Query: query })
        .then(response => alert(response))
        .catch(error => console.error(error));
}

function runSelectQuery() {
    const query = document.getElementById('selectQuery').value;

    sendMessage('SelectQuery', { Query: query })
        .then(response => {
            const queryResult = document.getElementById('queryResult');
            queryResult.innerHTML = JSON.stringify(response, null, 2);
        })
        .catch(error => console.error(error));
}

```
## Step 2: Setting Up the Backend

Create a new directory named backend and create the following file:

main.lua
```lua
local Handlers = require('handlers')
local sqlite3 = require('sqlite3')

-- Create an in-memory SQLite database
local db = sqlite3.open_memory()

-- Handler to create a table
Handlers.add('CreateTable', Handlers.utils.hasMatchingTag('Action', 'CreateTable'), function(msg, env, response)
    local tableName = msg.Tags.TableName
    local columns = msg.Tags.Columns
    local createStmt = 'CREATE TABLE ' .. tableName .. ' (' .. columns .. ')'
    db:exec(createStmt)
    Handlers.utils.reply('Table created successfully')(msg)
end)

-- Handler to insert a record
Handlers.add('InsertRecord', Handlers.utils.hasMatchingTag('Action', 'InsertRecord'), function(msg, env, response)
    local tableName = msg.Tags.TableName
    local values = msg.Tags.Values
    local insertStmt = 'INSERT INTO ' .. tableName .. ' VALUES (' .. values .. ')'
    db:exec(insertStmt)
    Handlers.utils.reply('Record inserted successfully')(msg)
end)

-- Handler to run a select query
Handlers.add('SelectQuery', Handlers.utils.hasMatchingTag('Action', 'SelectQuery'), function(msg, env, response)
    local query = msg.Tags.Query
    local result = {}
    for row in db:nrows(query) do
        table.insert(result, row)
    end
    Handlers.utils.reply(result)(msg)
end)

-- Handler to update a record
Handlers.add('UpdateRecord', Handlers.utils.hasMatchingTag('Action', 'UpdateRecord'), function(msg, env, response)
    local query = msg.Tags.Query
    db:exec(query)
    Handlers.utils.reply('Record updated successfully')(msg)
end)

-- Handler to delete a record
Handlers.add('DeleteRecord', Handlers.utils.hasMatchingTag('Action', 'DeleteRecord'), function(msg, env, response)
    local query = msg.Tags.Query
    db:exec(query)
    Handlers.utils.reply('Record deleted successfully')(msg)
end)


```
## Step 3: Loading the Backend into AOS


```sh
  .load main.lua

```

## Step 4: Testing the User Interface and Backend

1.Open the index.html file in your browser.

2.Use the forms to create tables, insert records, update records, delete records, and run select queries.

3.Ensure that the operations are performed correctly and the results are displayed as expected.



