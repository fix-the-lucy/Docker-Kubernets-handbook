# 🐳 Developing with Docker – JS App + MongoDB + Docker Network

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Project Overview — What We're Building |
| 2 | Docker Network — How Containers Talk |
| 3 | Pull MongoDB + Mongo Express Images |
| 4 | Run MongoDB Container |
| 5 | Run Mongo Express Container |
| 6 | Connect Node.js App to MongoDB |
| 7 | Hands-On: Full Node.js App Code |

---

## 🗺️ 1. Project Overview — What We're Building

A **User Profile** app with:
- **Node.js** backend (runs on port `3000`)
- **MongoDB** container (database, port `27017`)
- **Mongo Express** container (visual DB UI, port `8081`)
- All containers talk through an **isolated Docker Network**

```
Browser → localhost:3000
              │
        ┌─────▼──────┐      Docker Network (mongo-network)
        │  Node.js   │ ──► ┌────────────┐   ┌──────────────────┐
        │  App :3000 │     │  MongoDB   │◄──│  Mongo Express   │
        └────────────┘     │  :27017    │   │  UI  :8081       │
                           └────────────┘   └──────────────────┘
```

> 💡 JS App connects to MongoDB via `localhost:27017` (or container name inside Docker network).  
> Mongo Express connects to MongoDB to give a visual UI at `localhost:8081`.

---

## 🌐 2. Docker Network — How Containers Talk

Containers in the **same Docker network** can talk to each other using their **container name** as hostname.

```bash
# List all existing networks
docker network ls

# Create a new custom network
docker network create mongo-network

# Verify it was created
docker network ls
```

**Output:**
```
NETWORK ID     NAME             DRIVER    SCOPE
ef8884144b3e9  bridge           bridge    local
484f006515ef   docker_default   bridge    local
70eb8e9f1c1e   mongo-network    bridge    local   ← our new network
96afbbc49358   none             null      local
```

> 🔑 Containers in `mongo-network` can reach each other by **name** (e.g., `mongodb`) instead of IP address.

---

## 📥 3. Pull MongoDB + Mongo Express Images

```bash
# Pull MongoDB (latest)
docker pull mongo

# Pull Mongo Express (UI for MongoDB)
docker pull mongo-express

# Verify images are downloaded
docker images
```

**Expected output:**
```
REPOSITORY      TAG      IMAGE ID       SIZE
mongo           latest   965553e202a4   363 MB
mongo-express   latest   597a2912329c   97.6 MB
redis           4.0      e187e861db44   89.2 MB
```

---

## ▶️ 4. Run MongoDB Container

```bash
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --net mongo-network \
  mongo
```

**Flags explained:**

| Flag | Value | Purpose |
|---|---|---|
| `-d` | — | Run in background (detached) |
| `-p` | `27017:27017` | Expose MongoDB port to host |
| `-e` | `MONGO_INITDB_ROOT_USERNAME` | Set admin username |
| `-e` | `MONGO_INITDB_ROOT_PASSWORD` | Set admin password |
| `--name` | `mongodb` | Container name (used by other containers) |
| `--net` | `mongo-network` | Join our custom network |

---

## ▶️ 5. Run Mongo Express Container

```bash
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  --net mongo-network \
  --name mongo-express \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  mongo-express
```

**Flags explained:**

| Flag | Value | Purpose |
|---|---|---|
| `-p` | `8081:8081` | Expose Mongo Express UI port |
| `-e` | `ME_CONFIG_MONGODB_ADMINUSERNAME` | Match MongoDB admin username |
| `-e` | `ME_CONFIG_MONGODB_ADMINPASSWORD` | Match MongoDB admin password |
| `--net` | `mongo-network` | Same network as MongoDB |
| `-e` | `ME_CONFIG_MONGODB_SERVER=mongodb` | Connect to container named `mongodb` |

**Check logs to confirm connection:**
```bash
docker logs <mongo-express-container-id>
```

**Expected log output:**
```
Waiting for mongodb:27017...
Welcome to mongo-express
Mongo Express server listening at http://0.0.0.0:8081
Database connected
Admin Database connected
```

> ✅ Open `http://localhost:8081` in browser → You'll see the Mongo Express UI!

---

## 🔗 6. Connect Node.js App to MongoDB

The Node.js app connects to MongoDB using the connection string:

```
mongodb://admin:password@localhost:27017
```

**In `server.js`:**
```javascript
MongoClient.connect('mongodb://admin:password@localhost:27017', function(err, client) {
  if (err) throw err;

  var db = client.db('user-account');
  // ... rest of app logic
});
```

> 💡 When running the JS app **outside Docker** (directly on host), use `localhost:27017`.  
> When running the JS app **inside Docker**, use the container name `mongodb:27017`.

---

## 💻 7. Hands-On: Full Node.js App Code

> Try it yourself! Follow these steps:

### Step 1 — Project Structure

```
simple-js-app/
├── server.js
├── index.html
├── profile-1.jpg
└── package.json
```

### Step 2 — `package.json`

```json
{
  "name": "simple-js-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.17.1",
    "mongodb": "^3.3.3"
  }
}
```

### Step 3 — `server.js` (Full Code)

```javascript
const express = require('express');
const path = require('path');
const fs = require('fs');
const MongoClient = require('mongodb').MongoClient;

const app = express();
app.use(express.json());

// Serve the HTML page
app.get('/', function (req, res) {
  res.sendFile(path.join(__dirname, 'index.html'));
});

// Serve profile picture
app.get('/profile-picture', function (req, res) {
  var img = fs.readFileSync(path.join(__dirname, 'profile-1.jpg'));
  res.writeHead(200, { 'Content-Type': 'image/jpg' });
  res.end(img, 'binary');
});

// GET user profile from MongoDB
app.get('/get-profile', function (req, res) {
  MongoClient.connect('mongodb://admin:password@localhost:27017', function (err, client) {
    if (err) throw err;

    var db = client.db('user-account');
    var query = { userid: 1 };

    db.collection('users').findOne(query, function (err, result) {
      if (err) throw err;
      client.close();
      res.send(result);
    });
  });
});

// POST - Update user profile in MongoDB
app.post('/update-profile', function (req, res) {
  var userObj = req.body;
  var response = res;

  console.log('connecting to the db...');

  MongoClient.connect('mongodb://admin:password@localhost:27017', function (err, client) {
    if (err) throw err;

    var db = client.db('user-account');
    userObj['userid'] = 1;

    var query = { userid: 1 };
    var newValues = { $set: userObj };

    db.collection('users').updateOne(query, newValues, { upsert: true }, function (err, res) {
      if (err) throw err;
      console.log('successfully updated or inserted');
      client.close();
      response.send(userObj);
    });
  });
});

// Start server
app.listen(3000, function () {
  console.log('App listening on port 3000!');
});
```

### Step 4 — `index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>User Profile</title>
</head>
<body>
  <h1>User Profile</h1>
  <img id="profile-img" src="/profile-picture" width="150" /><br><br>

  Name: <b id="name"></b><br>
  Email: <input id="email" type="text" /><br>
  Interests: <input id="interests" type="text" /><br><br>

  <button onclick="saveProfile()">Save Profile</button>
  <button onclick="loadProfile()">Edit Profile</button>

  <script>
    // Load profile on page open
    window.onload = function() { loadProfile(); };

    function loadProfile() {
      fetch('/get-profile')
        .then(res => res.json())
        .then(data => {
          if (data) {
            document.getElementById('name').innerHTML = data.name || '';
            document.getElementById('email').value = data.email || '';
            document.getElementById('interests').value = data.interests || '';
          }
        })
        .catch(err => console.log('No profile yet'));
    }

    function saveProfile() {
      var payload = {
        name: document.getElementById('email').value,
        email: document.getElementById('email').value,
        interests: document.getElementById('interests').value
      };

      fetch('/update-profile', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      })
      .then(res => res.json())
      .then(data => { loadProfile(); })
      .catch(err => console.log(err));
    }
  </script>
</body>
</html>
```

### Step 5 — Run Everything

```bash
# 1. Create Docker network
docker network create mongo-network

# 2. Start MongoDB
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --net mongo-network \
  mongo

# 3. Start Mongo Express
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  --net mongo-network \
  --name mongo-express \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  mongo-express

# 4. Install Node.js dependencies
npm install

# 5. Start the Node app
node server.js
```

### Step 6 — Open in Browser

| URL | What you see |
|---|---|
| `http://localhost:3000` | User Profile App |
| `http://localhost:8081` | Mongo Express DB UI |

---

## 📝 Key Takeaways

- Use `docker network create` to let containers talk by **name** instead of IP
- Pass **environment variables** with `-e` flag for credentials
- Both containers must be on the **same network** to communicate
- Node.js app connects via `mongodb://admin:password@localhost:27017`
- Mongo Express gives a **visual browser UI** to inspect your MongoDB data
- `docker logs <container>` is your best friend for debugging connection issues

---

## 🔗 Resources

- 📺 [Docker Tutorial – Developing with Containers](https://youtu.be/6YisG2GcXaw?si=4tUiOjKUEdyCSUPH)
- 📖 [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
- 📖 [Mongo Express Docker Hub](https://hub.docker.com/_/mongo-express)
- 📖 [Docker Networking Docs](https://docs.docker.com/network/)
