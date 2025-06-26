You're currently trying to handle file uploads using express-fileupload, but your code still includes logic for multer. These two libraries are completely different and should not be mixed.


---

✅ Goal:

You want to upload a file using express-fileupload, and save it with a custom file name (Date.now() + file.originalname) and process it inside your controller.


---

🔧 Step-by-Step Conversion from Multer to express-fileupload

📦 1. Install express-fileupload if not already:

npm install express-fileupload


---

🛠️ 2. Setup Middleware in your app.js or main server file:

const express = require('express');
const fileUpload = require('express-fileupload');
const app = express();

app.use(express.json());
app.use(fileUpload()); // <-- Enable file upload


---

✨ 3. Modify Your Controller to Use express-fileupload

I'll walk through the changes needed in your createReportMasters function:

✅ Updated createReportMasters Function Using express-fileupload

exports.createReportMasters = (req, res) => {
  try {
    const { app_id, app_name, report_id, report_name, report_desc, frequency } = req.body;

    // file is accessed via req.files (as per express-fileupload)
    if (!req.files || !req.files.file) {
      return res.status(400).json({ error: "No Excel file uploaded" });
    }

    const file = req.files.file;

    // Create custom file name
    const fileName = `${Date.now()}-${file.name}--v-1`;

    // Path to save file
    const filePath = path.join(__dirname, "../../uploads", fileName); // or any folder

    // Move the file
    file.mv(filePath, (err) => {
      if (err) {
        return res.status(500).json({ error: "Failed to save file" });
      }

      // Frequency validation
      const validfrequencies = readFile(FREQUENCY_FILE);
      if (!validfrequencies.includes(frequency)) {
        return res.status(400).json({
          message: `Invalid frequency. Allowed: ${validfrequencies.join(", ")}`,
        });
      }

      // Define master file path
      const REPORT_MASTER_FILE = path.join(
        REPORT_MASTER_FOLDER_PATH,
        `${app_id}--${app_name}.json`
      );

      const parsedFullReportMaster = readFile(REPORT_MASTER_FILE);
      const timestamp = new Date().toISOString();

      const newReportMaster = {
        id: uuidv4(),
        report_id,
        report_name,
        report_desc,
        frequency,
        version: 1,
        is_active: true,
        uploadedFileName: fileName,
        created_by: req.user_id,
        modified_by: req.user_id,
        created_at: timestamp,
        modified_at: timestamp,
      };

      parsedFullReportMaster.reports_master.push(newReportMaster);

      writeFile(REPORT_MASTER_FILE, parsedFullReportMaster);
      return res.status(201).json({ message: "Report master mapped successfully" });
    });
  } catch (error) {
    return res.status(500).json({ message: error.message || error });
  }
};


---

📁 Folder Setup Recommendation

Ensure this directory exists (or create it):

/project-root
 └── data/
 └── uploads/     <-- save files here


---

🧪 Frontend (for testing in Postman or React)

Set body to form-data:

file → (select file)

app_id → example123

app_name → Test App

other fields...



---

📝 Summary of Key Changes

Feature	Before (multer)	After (express-fileupload)

Middleware	multer({ storage })	app.use(fileUpload())
File field	req.file	req.files.file
Save file manually	Handled by multer	Use file.mv()
Custom filename	Via filename callback	Manual filename with Date.now()



---

If you also want me to convert your updateReportMasters function to use express-fileupload, just say so and I’ll provide that too.

