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

## 5. Adding Swagger OpenAPI support to the above application

To add **Swagger OpenAPI** documentation support to your existing **Flask** application, you can use the **flask-restx** library, which integrates well with Flask applications and provides built-in support for **Swagger** documentation generation

Below is a modified version of your application with OpenAPI documentation added

**Step-by-Step Instructions**

**Install flask-restx**: If you haven't already installed flask-restx, you can do so using **pip**:

```
pip install flask-restx
```

**Modify the Flask Application**: Update your application to use **flask-restx** which includes namespaces, resource classes, and documentation automatically generated from your routes and models

Here's how you can modify your application:

```python
from flask import Flask, request
from flask_restx import Api, Resource, fields
from flask_pymongo import PyMongo
from bson import ObjectId, json_util
import json

app = Flask(__name__)

# Setup MongoDB connection
app.config["MONGO_URI"] = "mongodb://localhost:27017/mydatabase"
mongo = PyMongo(app)

# Initialize API
api = Api(app, version='1.0', title='MongoDB Flask API',
          description='A simple API to interact with MongoDB')

# Namespace
ns = api.namespace('myCollection', description='My Collection operations')

# Model for the MongoDB Document (used for Swagger documentation)
document_model = api.model('Document', {
    'name': fields.String(required=True, description='Name of the person'),
    'age': fields.Integer(description='Age of the person'),
})

# Convert MongoDB response to JSON
def parse_json(data):
    return json.loads(json_util.dumps(data))

@ns.route('/create')
class Create(Resource):
    @ns.expect(document_model)
    @ns.response(201, 'Document created successfully.')
    def post(self):
        # Create a new document
        _id = mongo.db.myCollection.insert_one(request.json).inserted_id
        return {'id': str(_id)}, 201

@ns.route('/read/<string:id>')
@ns.param('id', 'The Document identifier')
class Read(Resource):
    @ns.response(200, 'Document found.')
    @ns.response(404, 'Document not found.')
    def get(self, id):
        # Read a document
        document = mongo.db.myCollection.find_one({"_id": ObjectId(id)})
        if document:
            return parse_json(document), 200
        else:
            return {'error': 'Document not found'}, 404

@ns.route('/update/<string:id>')
@ns.param('id', 'The Document identifier')
class Update(Resource):
    @ns.expect(document_model)
    @ns.response(200, 'Document updated successfully.')
    @ns.response(400, 'Nothing to update.')
    def put(self, id):
        # Update a document
        result = mongo.db.myCollection.update_one({"_id": ObjectId(id)}, {"$set": request.json})
        if result.modified_count:
            return {'success': True}, 200
        else:
            return {'error': 'Nothing to update'}, 400

@ns.route('/delete/<string:id>')
@ns.param('id', 'The Document identifier')
class Delete(Resource):
    @ns.response(200, 'Document deleted successfully.')
    @ns.response(404, 'Document not found.')
    def delete(self, id):
        # Delete a document
        result = mongo.db.myCollection.delete_one({"_id": ObjectId(id)})
        if result.deleted_count:
            return {'success': True}, 200
        else:
            return {'error': 'Document not found'}, 404

if __name__ == '__main__':
    app.run(debug=True)
```

***Explanation**:

**API Initialization**: Api(app) creates a main entry point for the API which will include auto-generated Swagger documentation

**Namespace**: Used to organize and group operations. Each namespace can be considered a single resource or a group of related resources

**Resource Classes**: These replace your regular Flask view functions. Methods of these classes correspond to HTTP methods

**Model Definitions**: Used for request validation and also help generate structured documentation

**Responses and Expectations**: Define possible HTTP responses and set expectations for input models which are reflected in the Swagger documentation

This setup will provide a **Swagger UI** at the root URL of your **Flask** application (e.g., **http://localhost:5000/**) when you run the app, allowing you to interact with your API through a web interface.
