# **Local MongoDB Setup on Windows (CMD + DBeaver + Express)**
---

This guide walks through **running MongoDB locally using Docker**, managing it optionally via **DBeaver**, and connecting it to a **Node.js/Express application**.

---

## **1. Prerequisites**

* Docker Desktop installed and running
* Node.js installed (for Express backend)
* Optional: [DBeaver Community Edition](https://dbeaver.io/download/) for GUI

---

## **2. Create a Data Folder**

Create a folder to persist MongoDB data:

```cmd
mkdir "C:\Users\Alvin Kayi\Desktop\MongoDB"
```

This ensures your database will retain data even if the container stops or is removed.

---

## **3. Run MongoDB via Docker**

Run the MongoDB container:

```cmd
docker run -d --name mongodb-local -p 27017:27017 -v "C:\Users\Alvin Kayi\Desktop\MongoDB:/data/db" mongo:6
```

**Explanation:**

* `mongodb-local` → container name
* `27017` → default MongoDB port
* Volume mapping persists database in `"C:\Users\Alvin Kayi\Desktop\MongoDB"`

Verify it’s running:

```cmd
docker ps
```

---

## **4. Access MongoDB via CMD**

To use the MongoDB shell:

```cmd
docker exec -it mongodb-local mongo
```

Example commands in Mongo shell:

```js
use testDB
db.users.insertOne({ name: "Alvin", role: "Admin" })
db.users.find()
```

Exit Mongo shell with:

```js
exit
```

---

## **5. Connect MongoDB via DBeaver (Optional GUI)**

1. Download and open **DBeaver**: [https://dbeaver.io/download/](https://dbeaver.io/download/)
2. Create a new connection → Select **MongoDB**
3. Enter connection details:

| Field          | Value                    |
| -------------- | ------------------------ |
| Host           | localhost                |
| Port           | 27017                    |
| Database       | testDB (or leave blank)  |
| Authentication | None (default container) |

4. Click **Test Connection → Finish**
5. Browse collections, run queries, and manage your database visually.

---

## **6. Connect MongoDB to Node.js / Express**

1. **Install dependencies**:

```cmd
npm install express mongodb
```

2. **Create `db-mongo.js`**:

```js
const { MongoClient } = require("mongodb");

const url = "mongodb://localhost:27017";
const dbName = "testDB"; // Database name

const client = new MongoClient(url);

async function connectMongo() {
  try {
    await client.connect();
    console.log("Connected to MongoDB!");
    return client.db(dbName);
  } catch (err) {
    console.error("MongoDB connection failed", err);
  }
}

module.exports = connectMongo;
```

3. **Create `app.js`**:

```js
const express = require("express");
const connectMongo = require("./db-mongo");

const app = express();
const port = 3000;

app.use(express.json());

let db;

// Connect to MongoDB before starting the server
connectMongo().then(database => {
  db = database;
  app.listen(port, () => console.log(`Server running on http://localhost:${port}`));
});

// Sample route: insert a user
app.post("/users", async (req, res) => {
  try {
    const user = req.body;
    const result = await db.collection("users").insertOne(user);
    res.json({ insertedId: result.insertedId });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Failed to insert user" });
  }
});

// Sample route: get all users
app.get("/users", async (req, res) => {
  try {
    const users = await db.collection("users").find({}).toArray();
    res.json(users);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Failed to fetch users" });
  }
});
```

4. **Run your Express server**:

```cmd
node app.js
```

* **Test routes**:

  * `POST http://localhost:3000/users` → Add a user
  * `GET http://localhost:3000/users` → Fetch all users

---

## **7. Stop / Start / Remove MongoDB Container**

Stop container:

```cmd
docker stop mongodb-local
```

Start container:

```cmd
docker start mongodb-local
```

Remove container:

```cmd
docker rm -f mongodb-local
```

> Data persists in `"C:\Users\Alvin Kayi\Desktop\MongoDB"` even after removing the container.

---

## ✅ **Notes**

* MongoDB runs locally in Docker; no GUI is required.
* DBeaver is optional but useful for visually managing databases and collections.
* Node.js/Express connects to MongoDB using the official MongoDB driver.

---


