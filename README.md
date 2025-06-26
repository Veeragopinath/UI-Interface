function versionedFileName(originalName, version = 1) {
  const dotIndex = originalName.lastIndexOf(".");
  const name = originalName.substring(0, dotIndex);
  const ext = originalName.substring(dotIndex); // includes the dot

  return `${name}--v-${version}${ext}`;
}

const original = "Veera.xlsx";
const versioned = versionedFileName(original); // default version 1
console.log(versioned); // Output: Veera--v-1.xlsx