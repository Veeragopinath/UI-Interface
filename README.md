Great idea! Separating Multer config (file upload logic) into a service file and keeping the API logic clean is a best practice. Below is a clean and modular structure for your Express app.


---

✅ Goal: Modular Express File Upload

Structure:

project-root/
├── uploads/                  ← Excel files saved here
├── services/
│   └── multerService.js      ← Multer config (file upload logic)
├── routes/
│   └── uploadRoute.js        ← Express route (API logic)
├── server.js                 ← Main entry file
├── package.json
└── .gitignore


---

📦 Step-by-Step Setup


---

📁 services/multerService.js

const multer = require('multer');
const path = require('path');
const fs = require('fs');

// Create uploads folder if it doesn't exist
const uploadDir = path.join(__dirname, '../uploads');
if (!fs.existsSync(uploadDir)) {
  fs.mkdirSync(uploadDir);
}

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, uploadDir);
  },
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  }
});

const upload = multer({ storage });

module.exports = upload;


---

📁 routes/uploadRoute.js

const express = require('express');
const router = express.Router();
const upload = require('../services/multerService');
const xlsx = require('xlsx');
const path = require('path');

router.post('/upload', upload.single('file'), (req, res) => {
  try {
    const file = req.file;
    const createdBy = req.body.createdBy;

    if (!file) {
      return res.status(400).json({ error: 'No Excel file uploaded' });
    }

    // Read Excel content (optional)
    const workbook = xlsx.readFile(file.path);
    const sheet = workbook.Sheets[workbook.SheetNames[0]];
    const data = xlsx.utils.sheet_to_json(sheet);

    res.json({
      message: 'File uploaded successfully',
      uploadedBy: createdBy,
      filePath: file.filename,
      excelData: data,
    });
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({ error: 'Server error' });
  }
});

module.exports = router;


---

📁 server.js

const express = require('express');
const uploadRoutes = require('./routes/uploadRoute');
const app = express();
const PORT = 3000;

// Middleware to parse form data (text fields)
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Route
app.use('/api', uploadRoutes);

// Start server
app.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});


---

🧪 How to Test

1. Run the server:



node server.js

2. Open Postman or frontend form:

POST http://localhost:3000/api/upload

form-data:

file: Excel file

createdBy: Your name or ID






---

✅ Result

Excel file is stored in /uploads

createdBy is read from form field

Excel content is parsed and returned



---

Let me know if you want to:

Save the file or parsed data to a database

Support multiple files

Add file validation (like .xlsx only)


I'll help you extend this further.

