exports.createReportMasters = (req, res) => {
  try {
    const { app_id, app_name, report_id, report_name, report_desc, frequency } =
      req.body;
    const file = req.file;

    // file operations

    if (!file) {
      return res.status(400).json({ error: "No Excel file uploaded" });
    } else {
      const fileName = `${Date.now()}-${file.originalname}--v-1`;
      const upload = getMulter(fileName);
      upload.single("file");

      //   // Read Excel content (optional)
      //   const workbook = xlsx.readFile(file.path);
      //   const sheet = workbook.Sheets[workbook.SheetNames[0]];
      //   const data = xlsx.utils.sheet_to_json(sheet);
    }

    const validfrequencies = readFile(FREQUENCY_FILE);

    if (!validfrequencies.includes(frequency)) {
      return res.status(400).json({
        message: `Invalid frequencies. Allowed: ${validfrequencies.join(", ")}`,
      });
    }

    //file name
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
      version: 1, //todo
      is_active: true,
      uploadedFileName: `${Date.now()}-${file.originalname}--v-1`,
      created_by: req.user_id,
      modified_by: req.user_id,
      created_at: timestamp,
      modified_at: timestamp,
    };

    parsedFullReportMaster.reports_master.push(newReportMaster);

    writeFile(REPORT_MASTER_FILE, parsedFullReportMaster);
    res.status(201).json({ message: "report master mapped successfully" });
  } catch (error) {
    return res.status(400).json({
      message: error,
    });
  }
};
