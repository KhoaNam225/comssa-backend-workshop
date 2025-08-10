# Integrating FastAPI with MongoDB Atlas: A Beginner's Adventure

Welcome, brave coder! Today we're going to connect your shiny FastAPI app to MongoDB Atlas, the cloud-based MongoDB service. Think of it as giving your app a brain to remember things (data) even after it goes to sleep.

Don't worry if you've never touched a database before – we'll walk through everything step by step. By the end of this guide, you'll be storing and retrieving data like a pro!

## What We're Building

We're going to transform your simple FastAPI app into a data-powered machine that can:

- Store user information
- Retrieve data from the cloud
- Perform all the basic database operations (CRUD - Create, Read, Update, Delete)

## Prerequisites

Before we dive in, make sure you have:

- Your FastAPI project (which you already have!)
- Python installed on your computer
- A text editor or IDE (VS Code is great for beginners)
- An internet connection (we're going to the cloud!)
- A cup of coffee or your favorite beverage (optional but recommended)

## Part 1: Setting Up MongoDB Atlas (The Cloud Database)

**NOTE**: If you don't like to read the instructions, you can watch the tutorial here for the same guide on creating a new MongoDB database: [https://www.youtube.com/watch?v=Lbhwz_niCq0](https://www.youtube.com/watch?v=Lbhwz_niCq0)

### Step 1: Create Your Free MongoDB Atlas Account

MongoDB Atlas is like having a database butler in the cloud – it handles all the heavy lifting for you, and the best part? It's free for small projects!

To start using MogoDB Atlas, register an account follow the instructions here: [https://www.mongodb.com/docs/guides/atlas/account/](https://www.mongodb.com/docs/guides/atlas/account/)

### Step 2: Create a free cluster

Next, you will need to create a cluster for your database. Follow instructions here to create one: [https://www.mongodb.com/docs/guides/atlas/cluster/](https://www.mongodb.com/docs/guides/atlas/cluster/)

### Step 3: Security set-up

Security first! We need to make sure only you can access your database.

1. **Create a Database User**:

Follow the instructions on this link: [https://www.mongodb.com/docs/guides/atlas/db-user/](https://www.mongodb.com/docs/guides/atlas/db-user/)

2. **Set Up Network Access**:

Follow the instructions on this link to allow connection from your computer: [https://www.mongodb.com/docs/guides/atlas/network-connections/](https://www.mongodb.com/docs/guides/atlas/network-connections/)

### Step 3: Get Your Connection String

Follow this page to get your connection string, choose "Drivers" option when prompted: [https://www.mongodb.com/docs/guides/atlas/connection-string/](https://www.mongodb.com/docs/guides/atlas/connection-string/)

<<INSERT IMAGE HERE!>>

**NOTE**: Copy and paste this connection string somewhere on your computer. You will need this for later step. The connection string should have the format

## Part 2: Installing the Required Packages

Now let's get the tools we need to talk to MongoDB from Python.

### What We're Installing and Why

- **pymongo**: The official MongoDB driver for Python. Think of it as a translator between Python and MongoDB.
- **python-dotenv**: Helps us keep our database credentials safe and secret.
- **pydantic**: Already included with FastAPI, but we'll use it more extensively for data validation.

### Installation Steps

Open your terminal (Command Prompt on Windows, Terminal on Mac) and navigate to your FastAPI project folder.

Activate the virtual environment if you haven't:

**On Windows (Command Prompt):**

```cmd
.venv\Scripts\activate
```

**On Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1
```

**On Mac/Linux (Terminal):**

```bash
source .venv/bin/activate
```

Now let's install the packages. Since you're using `uv` (which is awesome for Python package management), run:

```bash
uv add pymongo python-dotenv
```

**What just happened?**

- `pymongo`: Installed the MongoDB driver
- `python-dotenv`: Added support for environment variables (keeping secrets safe)

## Part 3: Setting Up Environment Variables

We never, ever, EVER put database credentials directly in our code. That's like leaving your house key under the welcome mat with a sign saying "Key here!"

### Step 1: Create a .env File

In your project root (same folder as `main.py`), create a file called `.env` using your text editor.

### Step 2: Add Your MongoDB Connection String

Open the `.env` file in your text editor and add:

```env
MONGODB_USERNAME=<your_mongodb_username>
MONGODB_PASSWORD=<your_mongodb_password>
DATABASE_NAME=comssa-backend-workshop. # Or whatever name you like here
```

Replace the connection string with the one you copied from MongoDB Atlas.

### Step 3: Protect Your Secrets

Add `.env` to your `.gitignore` file so you don't accidentally share your database credentials:

Create or update your `.gitignore` file:

```gitignore
.env   # Add this line here!!!
__pycache__/
*.pyc
.venv/
```

## Part 4: Creating the Database Connection

Time to write some code! Let's create a file to handle our database connection.

### Step 1: Create a Database Module

Create a new file called `database.py` in your project root:

```python
import os
from pymongo import MongoClient
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Get the MongoDB connection string from environment variables
MONGODB_USERNAME = os.getenv("MONGODB_USERNAME", "")
MONGODB_PASSWORD = os.getenv("MONGODB_PASSWORD", "")
DATABASE_NAME = os.getenv("DATABASE_NAME", "")

# Construct the URL to our mongodb, obtain the template from MongoDB Atlas portal
MONGODB_URL = f"mongodb+srv://{quote_plus(MONGODB_USERNAME)}:{quote_plus(MONGODB_PASSWORD)}@cluster0.ssj5clj.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0"

# Create the MongoDB client
client = MongoClient(MONGODB_URL)

# Get the database
database = client[DATABASE_NAME]

# Test the connection (optional but helpful for debugging)
def ping_database():
    try:
        # The ping command is a simple way to test if the database is accessible
        client.admin.command('ping')
        print("Successfully connected to MongoDB!")
        return True
    except Exception as e:
        print(f"Error connecting to MongoDB: {e}")
        return False
```

**What's happening here?**

- We're loading our environment variables (the secret stuff from `.env`)
- Creating a MongoDB client using pymongo
- Setting up our database connection
- Adding a ping function to test if everything works

### Step 2: Create Data Models

Let's create some Pydantic models to define what our data looks like. Create a file called `models.py`:

```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

# The parent class that contains all required information of a user: name, email and age
class UserBase(BaseModel):
    name: str = Field(..., description="User's full name")
    email: str = Field(..., description="User's email address")
    age: int = Field(..., ge=0, le=150, description="User's age")

# This class represents an act of creating a new user in the database
# To create a new user, we basically need just name, email and age. Hence, we simply inherit this class from the base class
class UserCreate(UserBase):
    pass

# This is the actual user object stored in the database, apart from the required fields from the base class
# we also need an id to determine each users and created_at for auditing purpose
class User(UserBase):
    id: str = Field(..., alias="_id")
    created_at: datetime = Field(default_factory=datetime.utcnow)

    class Config:
        populate_by_name = True
        json_encoders = {
            datetime: lambda dt: dt.isoformat()
        }

# This class represents the act of updating an existing user
# When updating a user, we may not update all of the user's information (e.g. we might just need to change the email)
# Therefore, all of these fields are optional and we only need to provide required fields that need to be changed
class UserUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[str] = None
    age: Optional[int] = Field(None, ge=0, le=150)
```

**Breaking this down:**

- `UserBase`: The core user information
- `UserCreate`: Used when creating a new user
- `User`: The complete user model with ID and timestamps
- `UserUpdate`: Used when updating a user (all fields optional)

## Part 5: Understanding Database Operations (CRUD)

Now let's create functions to actually work with our database. Create a file called `crud.py` (CRUD stands for Create, Read, Update, Delete). We'll break down each operation so you understand exactly what's happening.

First, let's set up our imports and collection:

```python
from datetime import datetime, timezone
from typing import List, Optional

from bson import ObjectId
from pymongo.collection import Collection

from database import database
from models import User, UserCreate, UserUpdate

# Get the users collection - think of this as a table in traditional databases
users_collection: Collection = database.get_collection("users")
```

### 5.1 Create Operation - Adding New Users

This function adds a new user to our database:

```python
def create_user(user: UserCreate) -> User:
    """Create a new user in the database"""
    # Convert the Pydantic model to a dictionary that MongoDB can understand
    user_dict = user.model_dump()

    # Add a timestamp when the user was created
    user_dict["created_at"] = datetime.now(timezone.utc)

    # Insert the user into the database - MongoDB automatically creates an _id
    result = users_collection.insert_one(user_dict)

    # Get the newly created user from the database using the generated ID
    created_user = users_collection.find_one({"_id": result.inserted_id})

    if created_user is None:
        raise ValueError("User creation failed, user not found in database.")

    # MongoDB ObjectIds are not JSON serializable, so convert to string
    created_user["_id"] = str(created_user["_id"])

    # Return the user as a Pydantic model for validation and response
    return User(**created_user)
```

**What's happening here step by step:**

1. We take the user data and convert it to a dictionary
2. Add a timestamp so we know when the user was created
3. Insert the data into MongoDB (MongoDB automatically adds a unique `_id`)
4. Retrieve the newly created user to get the complete data including the ID
5. Convert the MongoDB ObjectId to a string so it can be sent as JSON
6. Return a proper User model

### 5.2 Read Operations - Finding Users

#### Getting a Single User by ID

```python
def get_user_by_id(user_id: str) -> Optional[User]:
    """Get a user by their ID"""
    # First, check if the provided ID is a valid MongoDB ObjectId
    if not ObjectId.is_valid(user_id):
        return None

    # Search for the user in the database using their ID
    user = users_collection.find_one({"_id": ObjectId(user_id)})

    # If we found a user, process and return it
    if user:
        # Convert ObjectId to string for JSON serialization
        user["_id"] = str(user["_id"])
        return User(**user)

    # Return None if no user was found
    return None
```

**What this does:**

- Takes a user ID as a string
- Validates that it's a proper MongoDB ObjectId format
- Searches the database for a user with that ID
- Returns the user if found, or None if not found

#### Getting All Users

```python
def get_all_users() -> List[User]:
    """Get all users from the database"""
    # Create a cursor to iterate through all users
    cursor = users_collection.find()
    users = []

    # Loop through each user document in the database
    for user in cursor:
        # Convert ObjectId to string
        user["_id"] = str(user["_id"])
        # Add the user to our list
        users.append(User(**user))

    return users
```

**What this does:**

- Uses `find()` without parameters to get ALL users
- Loops through each user document
- Converts each user to our User model and adds to a list
- Returns the complete list of users

### 5.3 Update Operation - Modifying Existing Users

```python
def update_user(user_id: str, user_update: UserUpdate) -> Optional[User]:
    """Update a user's information"""
    # Validate the user ID format
    if not ObjectId.is_valid(user_id):
        return None

    # Only update fields that were actually provided (not None)
    update_data = {k: v for k, v in user_update.dict().items() if v is not None}

    # If no data to update, just return the current user
    if not update_data:
        return get_user_by_id(user_id)

    # Update the user in the database
    users_collection.update_one(
        {"_id": ObjectId(user_id)},  # Find the user by ID
        {"$set": update_data}        # Set the new values
    )

    # Return the updated user
    return get_user_by_id(user_id)
```

**What this does:**

- Validates the user ID
- Creates a dictionary with only the fields that need updating
- Uses MongoDB's `$set` operator to update only specific fields
- Returns the updated user data

**The `$set` operator:** This is a MongoDB operator that updates specific fields without affecting others. If you only want to change a user's age, it won't touch their name or email.

### 5.4 Delete Operation - Removing Users

```python
def delete_user(user_id: str) -> bool:
    """Delete a user from the database"""
    # Validate the user ID format
    if not ObjectId.is_valid(user_id):
        return False

    # Delete the user from the database
    result = users_collection.delete_one({"_id": ObjectId(user_id)})

    # Return True if a user was actually deleted, False if not found
    return result.deleted_count > 0
```

**What this does:**

- Validates the user ID
- Attempts to delete the user from the database
- Returns `True` if a user was actually deleted, `False` if no user was found

**Why return a boolean?** This tells us whether the operation was successful. If `deleted_count` is 0, it means no user was found with that ID.

## Part 6: Understanding FastAPI Endpoints and HTTP Methods

Now let's understand how to create API endpoints that use our database operations. Each endpoint serves a specific purpose and uses different HTTP methods.

### 6.1 Understanding HTTP Methods

Before we dive into the code, let's understand what each HTTP method means:

- **GET**: Retrieve data (like reading a book)
- **POST**: Create new data (like adding a new entry)
- **PUT**: Update existing data (like editing an entry)
- **DELETE**: Remove data (like erasing an entry)

### 6.2 Setting Up the FastAPI Application

```python
from fastapi import FastAPI, HTTPException, Query
from typing import List
from datetime import datetime
from contextlib import asynccontextmanager

# Import our database and models
from database import ping_database
from models import User, UserCreate, UserUpdate
import crud


# This runs when the app starts
# Main purpose is to check database connectivity
# For the scope of the workshop, you can ignore this part
@asynccontextmanager
async def lifespan(app: FastAPI):
    """Test database connection when the app starts"""
    ping_database()
    yield


# Create FastAPI instance
app = FastAPI(
    title="My Awesome FastAPI App with MongoDB",
    description="This FastAPI application now has a brain (database)!",
    version="2.0.0",
    lifespan=lifespan,
)
```

### 6.3 GET Endpoints - Retrieving Data

#### Root Endpoint (Simple GET)

```python
@app.get("/")
def read_root():
    return {
        "message": "Hello World! Now with MongoDB power!",
        "database": "Connected to MongoDB Atlas",
        "version": "2.0.0"
    }
```

**What this does:**

- `@app.get("/")` means this endpoint responds to GET requests at the root URL
- Returns a simple JSON object with information about your app
- No parameters needed

#### Get All Users (GET)

```python
@app.get("/users/", response_model=List[User])
def get_users():
    """Get all users from the database"""
    return crud.get_all_users()
```

**Understanding this endpoint:**

- `@app.get("/users/")` creates a GET endpoint at `/users/`
- `response_model=List[User]` tells FastAPI this returns a list of User objects
- The function simply calls our CRUD function and returns the result
- FastAPI automatically converts our User objects to JSON

#### Get Single User (GET with Path Parameters)

```python
@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: str):
    """Get a specific user by their ID"""
    user = crud.get_user_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

**Understanding path parameters:**

- `{user_id}` in the path is a **path parameter** - it captures part of the URL
- If someone visits `/users/123`, then `user_id` will be `"123"`
- We use this ID to look up the specific user
- If no user is found, we return a 404 error (standard for "not found")

### 6.4 POST Endpoint - Creating Data

```python
@app.post("/users/", response_model=User)
def create_user(user: UserCreate):
    """Create a new user in the database"""
    try:
        return crud.create_user(user)
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))
```

**Understanding POST and Request Bodies:**

- `@app.post("/users/")` creates a POST endpoint
- `user: UserCreate` is the **request body** - data sent in the request
- When someone makes a POST request, they send JSON data in the body
- FastAPI automatically converts that JSON to a UserCreate object
- We try to create the user and handle any errors with a 400 status code

**Example request body:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 25
}
```

### 6.5 PUT Endpoint - Updating Data

```python
@app.put("/users/{user_id}", response_model=User)
def update_user(user_id: str, user_update: UserUpdate):
    """Update a user's information"""
    user = crud.update_user(user_id, user_update)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

**Understanding PUT:**

- Combines both path parameter (`user_id`) and request body (`user_update`)
- The path parameter tells us WHICH user to update
- The request body tells us WHAT to update
- PUT is typically used for updating existing resources

**Example:** To update user `123`'s age to `26`:

- URL: `/users/123`
- Request body: `{"age": 26}`

### 6.6 DELETE Endpoint - Removing Data

```python
@app.delete("/users/{user_id}")
def delete_user(user_id: str):
    """Delete a user from the database"""
    success = crud.delete_user(user_id)
    if not success:
        raise HTTPException(status_code=404, detail="User not found")
    return {"message": "User deleted successfully"}
```

**Understanding DELETE:**

- Uses only a path parameter to identify which user to delete
- No request body needed - we're just removing something
- Returns a success message instead of the deleted data
- Returns 404 if the user doesn't exist

### 6.7 Understanding HTTP Status Codes

In our endpoints, we use different status codes:

- **200**: Success (automatically returned by FastAPI)
- **400**: Bad Request (when something is wrong with the request)
- **404**: Not Found (when a user doesn't exist)

### 6.8 Complete FastAPI Application Code

Here's how all the pieces fit together:

```python
from fastapi import FastAPI, HTTPException
from typing import List
from datetime import datetime

# Import our database and models
from database import ping_database
from models import User, UserCreate, UserUpdate
import crud

# Create FastAPI instance
app = FastAPI(
    title="My Awesome FastAPI App with MongoDB",
    description="This FastAPI application now has a brain (database)!",
    version="2.0.0",
)

@app.on_event("startup")
def startup_event():
    """Test database connection when the app starts"""
    ping_database()

# Root endpoint
@app.get("/")
def read_root():
    return {
        "message": "Hello World! Now with MongoDB power!",
        "database": "Connected to MongoDB Atlas",
        "version": "2.0.0"
    }

# Create a new user (POST)
@app.post("/users/", response_model=User)
def create_user(user: UserCreate):
    """Create a new user in the database"""
    try:
        return crud.create_user(user)
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

# Get all users (GET)
@app.get("/users/", response_model=List[User])
def get_users():
    """Get all users from the database"""
    return crud.get_all_users()

# Get a specific user (GET with path parameter)
@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: str):
    """Get a specific user by their ID"""
    user = crud.get_user_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Update a user (PUT with path parameter and request body)
@app.put("/users/{user_id}", response_model=User)
def update_user(user_id: str, user_update: UserUpdate):
    """Update a user's information"""
    user = crud.update_user(user_id, user_update)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Delete a user (DELETE with path parameter)
@app.delete("/users/{user_id}")
def delete_user(user_id: str):
    """Delete a user from the database"""
    success = crud.delete_user(user_id)
    if not success:
        raise HTTPException(status_code=404, detail="User not found")
    return {"message": "User deleted successfully"}

# Health check endpoint
@app.get("/health")
def health_check():
    """Check if the app and database are healthy"""
    db_healthy = ping_database()
    return {
        "status": "healthy" if db_healthy else "unhealthy",
        "database": "connected" if db_healthy else "disconnected",
        "timestamp": datetime.utcnow().isoformat()
    }
```

### 6.9 URL Patterns Summary

Here's what each endpoint looks like in action:

- `GET /` - Get basic app info
- `POST /users/` - Create a new user (send user data in request body)
- `GET /users/` - Get all users
- `GET /users/123` - Get user with ID "123"
- `PUT /users/123` - Update user with ID "123" (send update data in request body)
- `DELETE /users/123` - Delete user with ID "123"
- `GET /health` - Check if app and database are working

## Part 7: Testing Your Application

Time for the moment of truth! Let's see if everything works.

### Step 1: Start Your Application

In your terminal, run:

```bash
uv run fastapi dev main.py
```

### Step 2: Test the Connection

1. Open your browser and go to `http://localhost:8000/health`
2. You should see something like:
   ```json
   {
     "status": "healthy",
     "database": "connected",
     "timestamp": "2024-01-15T10:30:00.000Z"
   }
   ```

If you see "connected", congratulations! Your app is talking to MongoDB Atlas!

### Step 3: Explore the API Documentation

FastAPI automatically creates beautiful API documentation for you:

1. Go to `http://localhost:8000/docs`
2. You'll see all your endpoints with the ability to test them directly!

### Step 4: Create Your First User

In the API docs:

1. Click on "POST /users/"
2. Click "Try it out"
3. Enter some test data:
   ```json
   {
     "name": "John Doe",
     "email": "john@example.com",
     "age": 25
   }
   ```
4. Click "Execute"

If everything works, you'll get back a user object with an ID and timestamp!

## Part 8: Common Issues and Troubleshooting

### Problem: "Can't connect to MongoDB"

**Solution:**

1. Check your `.env` file – make sure the MongoDB URL is correct
2. Verify your MongoDB Atlas IP whitelist includes your current IP
3. Double-check your database username and password

### Problem: "ModuleNotFoundError"

**Solution:**
Make sure you installed all the packages:

```bash
uv add pymongo python-dotenv
```

### Problem: "Invalid ObjectId"

**Solution:**
MongoDB uses special IDs. Make sure you're passing valid ID strings to your endpoints.

### Problem: Database connection times out

**Solution:**

1. Check your internet connection
2. Verify your MongoDB Atlas cluster is running
3. Make sure you're using the correct region

## More exercises

Now that we have the basic operations (create, delete, update, read) done. Could you implements some extra endpoints to support these queries:

1. Get all users that are more than X years of age (define your own X).
2. Delete all users that are created more than 5 minutes ago.
3. Adjusting the current create user operation so that if an email has already existed in the database, return an error saying that email already exists.
4. For the get all users endpoint, at the moment, it will return all users in the database regardless how many we have. That will be a bit troublesome if we have many users. For that reason, adjust the endpoint so that now it accepts a new parameter called `limit` which is an integer. The endpoint will return the number of users equal to `limit` from the database.

## Conclusion

You did it! You've successfully connected your FastAPI application to MongoDB Atlas. Your app can now:

- Store user data in the cloud
- Retrieve and update information
- Handle all basic database operations

Remember, this is just the beginning. You can now build all sorts of amazing applications with persistent data storage. Whether it's a blog, an e-commerce site, or the next big social media platform, you have the foundation you need.

## Additional Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [MongoDB Atlas Documentation](https://docs.atlas.mongodb.com/)
- [PyMongo Documentation](https://pymongo.readthedocs.io/)
- [Pydantic Documentation](https://pydantic-docs.helpmanual.io/)

Happy coding, and welcome to the world of database-powered applications!

---

_Made with ❤️ for Curtin students learning web development_
