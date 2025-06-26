const express = require("express");
const fs = require("fs");
const { v4: uuidv4 } = require("uuid");
const path = require("path");
const getMulter = require("../services/multerServices");

const APP_FILE = path.join(__dirname, "../../data/app_master.json");
const COUNTRIES_FILE = path.join(__dirname, "../../data/countries.json");
const FREQUENCY_FILE = path.join(__dirname, "../../data/frequencies.json");
const REPORTING_NATURE_FILE = path.join(
  __dirname,
  "../../data/reporting_nature.json"
);
const REPORT_MASTER_FOLDER_PATH = path.join(
  __dirname,
  "../../data/report-master"
);

// Utility: Read JSON file
const readFile = (file) => JSON.parse(fs.readFileSync(file, "utf8"));

// Utility: Write to file
const writeFile = (file, data) => {
  fs.writeFileSync(file, JSON.stringify(data, null, 2));
};

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

exports.getReportMasters = (req, res) => {
  try {
    const app_id = req.query.app_id;
    const app_name = req.query.app_name;
    //file name
    const REPORT_MASTER_FILE = path.join(
      REPORT_MASTER_FOLDER_PATH,
      `${app_id}--${app_name}.json`
    );
    const parsedFullReportMaster = readFile(REPORT_MASTER_FILE);

    res.json(parsedFullReportMaster);
  } catch (error) {
    return res.status(400).json({
      message: error,
    });
  }
};

exports.updateReportMasters = (req, res) => {
  try {
    const {
      app_id,
      app_name,
      id,
      report_id,
      report_name,
      report_desc,
      frequency,
    } = req.body;
    const file = req.file;

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
    if (parsedFullReportMaster.reportMaster) {
      // finding index of to be edited rm
      const reportMasterIndex = reportMaster.findIndex((obj) => obj.id === id);
      const existingReportMaster = reportMaster.find((obj) => obj.id === id);

      //file operation
      let newFileName;
      if (file && existingReportMaster.uploadedFileName) {
        newFileName = `${Date.now()}-${file.originalname}--v-${
          existingReportMaster.uploadedFileName[-1] + 1
        } `;
        const upload = getMulter(newFileName);
        upload.single("file");
      } else {
        //   // Read Excel content (optional)
        //   const workbook = xlsx.readFile(file.path);
        //   const sheet = workbook.Sheets[workbook.SheetNames[0]];
        //   const data = xlsx.utils.sheet_to_json(sheet);
      }
      parsedFullReportMaster.reportMaster[reportMasterIndex] = {
        id,
        report_id,
        report_name,
        report_desc,
        frequency,
        version: existingReportMaster.version + 1,
        is_active: true,
        uploadedFileName: req.file
          ? newFileName
          : existingReportMaster.uploadedFileName,
        created_by: existingReportMaster.created_by,
        modified_by: req.user_id,
        created_at: existingReportMaster.created_at,
        modified_at: timestamp,
      };

      writeFile(REPORT_MASTER_FILE, parsedFullReportMaster);
      res.status(201).json({ message: "report master mapped successfully" });
    }

    return res.status(400).json({
      message: "report master does not exist",
    });
  } catch (error) {
    return res.status(400).json({
      message: error,
    });
  }
};
