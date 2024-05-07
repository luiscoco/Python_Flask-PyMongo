# Python a CRUD API with Flask-PyMongo

Creating a simple **Python CRUD (Create, Read, Update, Delete) API** to connect to **MongoDB** involves several steps

Below, I'll outline a basic **Flask** application that uses Flask and PyMongo

Before starting, make sure you have MongoDB installed and running on your local machine or have access to a MongoDB instance

## 1. Install Required Packages

First, you need to install the necessary **Python packages**

You can do this using **pip**

The primary packages are **Flask** for the web framework and **Flask-PyMongo** for interacting with MongoDB

```
pip install Flask Flask-PyMongo
```

## 2. Python Code for the CRUD API

Here is a simple Flask application that provides basic CRUD functionality:

```python
from flask import Flask, jsonify, request
from flask_pymongo import PyMongo
from bson import ObjectId

app = Flask(__name__)

# Setup MongoDB connection
app.config["MONGO_URI"] = "mongodb://localhost:27017/mydatabase"
mongo = PyMongo(app)

# Collection
db = mongo.db.myCollection

@app.route('/create', methods=['POST'])
def create():
    # Create a new document
    _id = db.insert_one(request.json).inserted_id
    return jsonify(str(_id)), 201

@app.route('/read/<id>', methods=['GET'])
def read(id):
    # Read a document
    document = db.find_one({"_id": ObjectId(id)})
    if document:
        return jsonify(str(document)), 200
    else:
        return jsonify({"error": "Document not found"}), 404

@app.route('/update/<id>', methods=['PUT'])
def update(id):
    # Update a document
    result = db.update_one({"_id": ObjectId(id)}, {"$set": request.json})
    if result.modified_count:
        return jsonify({"success": True}), 200
    else:
        return jsonify({"error": "Nothing to update"}), 400

@app.route('/delete/<id>', methods=['DELETE'])
def delete(id):
    # Delete a document
    result = db.delete_one({"_id": ObjectId(id)})
    if result.deleted_count:
        return jsonify({"success": True}), 200
    else:
        return jsonify({"error": "Document not found"}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

**Explanation of the Code**

**Connection Setup**: The MongoDB connection is configured via the MONGO_URI, which connects to a database named mydatabase

**CRUD Functions**:

**Create**: Inserts a new document into the collection based on the provided JSON data

**Read**: Fetches a document by its _id

**Update**: Modifies an existing document specified by its _id with new data

**Delete**: Removes the document with the specified _id

## 3. Running Your Application

To run your Flask application:

Save the script in a file, for example, **app.py**

Run the script using Python: **python app.py**

The Flask app will start on **localhost** on **port 5000**

## 4. Testing the API

You can test the API using tools like Postman or Curl:

**Create**: Send a POST request to http://localhost:5000/create with JSON data

**Read**: Send a GET request to http://localhost:5000/read/<id>

**Update**: Send a PUT request to http://localhost:5000/update/<id> with JSON data

**Delete**: Send a DELETE request to http://localhost:5000/delete/<id>

This will provide you with a basic but functional **CRUD API** for **MongoDB** using **Flask**

Adjust the structure and complexity based on your specific requirements and **MongoDB** setup
