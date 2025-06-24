Great! Based on your requirements, here’s how to create GET and POST APIs using Node.js + Express with storage in a file named app_master.json (no database used).


---

✅ Schema:

{
  "id": "uuid",
  "application_name": "string",
  "impacted_country": ["country1", "country2"],
  "app_id": "string",
  "description": "string",
  "no_of_reports": 3,
  "reporting_nature": "string",    // (taken from a master list)
  "is_active": true,
  "created_by": "string",
  "modified_by": "string",
  "created_at": "ISO8601 timestamp",
  "modified_at": "ISO8601 timestamp"
}


---

📁 File Structure

project/
│
├── app_master.json          <-- Data storage file
├── reporting_nature.json    <-- Master values for reporting nature
├── countries.json           <-- Master list of countries
├── index.js                 <-- Express server
└── package.json


---

📦 Install Packages

npm init -y
npm install express uuid fs


---

🧾 Create app_master.json file with empty array

[]


---

🧾 Create reporting_nature.json

["Daily", "Weekly", "Monthly", "Quarterly"]


---

🧾 Create countries.json

["India", "USA", "UK", "Germany", "France"]


---

📄 index.js

const express = require("express");
const fs = require("fs");
const { v4: uuidv4 } = require("uuid");

const app = express();
app.use(express.json());

const APP_FILE = "app_master.json";
const COUNTRIES_FILE = "countries.json";
const REPORTING_NATURE_FILE = "reporting_nature.json";

// Utility: Read JSON file
const readFile = (file) => JSON.parse(fs.readFileSync(file, "utf8"));

// Utility: Write to file
const writeFile = (file, data) => {
  fs.writeFileSync(file, JSON.stringify(data, null, 2));
};

// ✅ GET all applications
app.get("/applications", (req, res) => {
  const apps = readFile(APP_FILE);
  res.json(apps);
});

// ✅ POST new application
app.post("/applications", (req, res) => {
  const {
    application_name,
    impacted_country,
    app_id,
    description,
    no_of_reports,
    reporting_nature,
    is_active,
    created_by
  } = req.body;

  const countries = readFile(COUNTRIES_FILE);
  const validNatures = readFile(REPORTING_NATURE_FILE);

  // Validation
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
    created_by,
    modified_by: created_by,
    created_at: timestamp,
    modified_at: timestamp
  };

  apps.push(newApp);
  writeFile(APP_FILE, apps);

  res.status(201).json({ message: "Application created successfully", id: newApp.id });
});

// 🔁 Start Server
app.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});


---

🧪 Sample POST Request (/applications)

{
  "application_name": "Sales Dashboard",
  "impacted_country": ["India", "USA"],
  "app_id": "APP12345",
  "description": "Tracks sales metrics",
  "no_of_reports": 5,
  "reporting_nature": "Monthly",
  "is_active": true,
  "created_by": "admin"
}


---

Let me know if you want to add update/delete APIs or apply filtering/search on GET.

