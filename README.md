Great! You're aiming for a realistic token-based authentication system using JWT (JSON Web Tokens), where:

After login, the backend issues a JWT token.

The token is passed in the Authorization header like:

Authorization: Bearer <token>

All protected routes (like GET /applications) decode the token to get the user_id.



---

✅ Solution Overview

1. Login → Validates credentials and returns a JWT token.


2. Protected Routes (e.g., GET /applications) → Require the token in header, extract user_id from it.


3. No user_id in query → All secure, token-driven.




---

📦 Install One Extra Package

npm install jsonwebtoken


---

🔐 Secret Key

Add a secret for signing the token. You can hardcode it or put in .env. For now, we’ll use:

const JWT_SECRET = "your-secret-key"; // keep this safe!


---

✅ Full Updated index.js

const express = require("express");
const fs = require("fs");
const { v4: uuidv4 } = require("uuid");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt");

const app = express();
app.use(express.json());

const JWT_SECRET = "your-secret-key";
const USER_FILE = "user.json";
const APP_FILE = "app_master.json";
const COUNTRIES_FILE = "countries.json";
const REPORTING_NATURE_FILE = "reporting_nature.json";

// Utility
const readFile = (file) => JSON.parse(fs.readFileSync(file, "utf8"));
const writeFile = (file, data) => fs.writeFileSync(file, JSON.stringify(data, null, 2));

// ✅ Login (returns token)
app.post("/login", async (req, res) => {
  const { user_id, password } = req.body;
  const users = readFile(USER_FILE);
  const user = users.find(u => u.user_id === user_id);
  
  if (!user) return res.status(404).json({ message: "User not found" });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(401).json({ message: "Invalid password" });

  const token = jwt.sign({ user_id: user.user_id }, JWT_SECRET, { expiresIn: "1h" });
  res.json({ token });
});

// ✅ Middleware to verify JWT token
function authenticateToken(req, res, next) {
  const authHeader = req.headers["authorization"];
  const token = authHeader && authHeader.split(" ")[1];

  if (!token) return res.status(401).json({ message: "No token provided" });

  jwt.verify(token, JWT_SECRET, (err, decoded) => {
    if (err) return res.status(403).json({ message: "Invalid token" });
    req.user_id = decoded.user_id;
    next();
  });
}

// ✅ GET apps created by logged-in user
app.get("/applications", authenticateToken, (req, res) => {
  const apps = readFile(APP_FILE);
  const userApps = apps.filter(app => app.created_by === req.user_id);
  res.json(userApps);
});

// ✅ POST create new app
app.post("/applications", authenticateToken, (req, res) => {
  const {
    application_name,
    impacted_country,
    app_id,
    description,
    no_of_reports,
    reporting_nature,
    is_active
  } = req.body;

  const countries = readFile(COUNTRIES_FILE);
  const validNatures = readFile(REPORTING_NATURE_FILE);

  const invalidCountries = impacted_country.filter(c => !countries.includes(c));
  if (invalidCountries.length > 0) {
    return res.status(400).json({ message: `Invalid countries: ${invalidCountries.join(", ")}` });
  }

  if (!validNatures.includes(reporting_nature)) {
    return res.status(400).json({ message: `Invalid reporting nature. Allowed: ${validNatures.join(", ")}` });
  }

  const apps = readFile(APP_FILE);
  const timestamp = new Date().toISOString();

  const newApp = {
    id: uuidv4(),
    application_name,
    impacted_country,
    app_id,
    description,
    no_of_reports: parseInt(no_of_reports),
    reporting_nature,
    is_active: Boolean(is_active),
    created_by: req.user_id,
    modified_by: req.user_id,
    created_at: timestamp,
    modified_at: timestamp
  };

  apps.push(newApp);
  writeFile(APP_FILE, apps);

  res.status(201).json({ message: "Application created successfully", id: newApp.id });
});

// ✅ Start server
app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});


---

🧪 Example Usage

1. Login to get token

POST /login
{
  "user_id": "user001",
  "password": "Pass@123"
}

Response:

{
  "token": "eyJhbGciOi..."
}

2. Use token in request headers

GET /applications
Authorization: Bearer eyJhbGciOi...

3. POST new app with token

POST /applications
Authorization: Bearer eyJhbGciOi...

{
  "application_name": "Sales Tool",
  "impacted_country": ["India", "USA"],
  "app_id": "APP101",
  "description": "Tracks sales",
  "no_of_reports": 3,
  "reporting_nature": "Monthly",
  "is_active": true
}


---

Let me know if you want:

Token refresh mechanism

User registration API with password hash

Role-based access (admin vs user)

Sure! Here's a complete and secure User Registration API in Node.js + Express, which:

Registers a user with a unique uuid

Hashes the password using bcrypt

Stores the user data in user.json

Prevents duplicate user registration

Sets timestamps for auditing



---

✅ Prerequisites

Make sure you have installed:

npm install express uuid bcrypt fs


---

🧾 user.json (initial empty array):

[]


---

✅ Registration API Code (POST /register)

Add this to your existing index.js:

const bcrypt = require("bcrypt");
const { v4: uuidv4 } = require("uuid");

const USER_FILE = "user.json";

// Utility to read/write users
const readUsers = () => JSON.parse(fs.readFileSync(USER_FILE, "utf-8"));
const writeUsers = (users) => fs.writeFileSync(USER_FILE, JSON.stringify(users, null, 2));

// ✅ POST /register - Register a new user
app.post("/register", async (req, res) => {
  const { user_id, password, role, created_by } = req.body;

  if (!user_id || !password || !role) {
    return res.status(400).json({ message: "user_id, password, and role are required" });
  }

  const users = readUsers();

  // Check if user_id already exists
  if (users.find(user => user.user_id === user_id)) {
    return res.status(409).json({ message: "User ID already exists" });
  }

  const hashedPassword = await bcrypt.hash(password, 10);
  const timestamp = new Date().toISOString();

  const newUser = {
    uuid: uuidv4(),
    user_id,
    password: hashedPassword,
    role,
    is_active: true,
    created_by: created_by || "system",
    modified_by: created_by || "system",
    created_at: timestamp,
    modified_at: timestamp
  };

  users.push(newUser);
  writeUsers(users);

  res.status(201).json({ message: "User registered successfully", user_id });
});


---

✅ Sample Request

POST /register
Content-Type: application/json

{
  "user_id": "user001",
  "password": "Secret@123",
  "role": "admin",
  "created_by": "system"
}


---

✅ Success Response

{
  "message": "User registered successfully",
  "user_id": "user001"
}


---

🔐 Important Notes

Passwords are hashed with bcrypt before storing.

Duplicate user IDs are prevented.

The uuid, created_at, and modified_at are automatically handled.



---

Would you like me to:

Add role validation from a master list?

Add email/mobile support?

Implement token generation right after registration (auto-login)?





