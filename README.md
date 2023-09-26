const axios = require('axios');
const cheerio = require('cheerio');
const commander = require('commander');
const path = require('path');
const fs = require('fs');
const ExifParser = require('exif-parser');
const ExifWriter = require('exif-writer');

// Define the path to the image file
const imagePath = 'Dobe.jpeg'; // Replace with the path to your image file

// Define a function to edit or delete EXIF metadata
function editOrDeleteMetadata(imagePath, options) {
  // Read the image file
  fs.readFile(imagePath, (err, imageBuffer) => {
    if (err) {
      console.error('Error reading image file:', err);
      return;
    }

    // Create an ExifParser instance and parse the image data
    const exifParser = ExifParser.create(imageBuffer);
    const result = exifParser.parse();

    // Edit or delete metadata based on options
    if (options.edit) {
      // Edit metadata by updating specific fields
      if (options.title) {
        result.image.ImageDescription = options.title;
      }
      // Add more fields as needed

      // Write the updated EXIF data back to the image
      const exifWriter = ExifWriter.create(result.image);
      const updatedBuffer = exifWriter.encode();
      fs.writeFile(imagePath, updatedBuffer, (err) => {
        if (err) {
          console.error('Error writing updated image:', err);
        } else {
          console.log('Metadata edited successfully.');
        }
      });
    } else if (options.delete) {
      // Delete specific metadata fields
      if (options.fieldToDelete) {
        delete result.image[options.fieldToDelete];
      }
      // Add more fields to delete as needed

      // Write the updated EXIF data back to the image
      const exifWriter = ExifWriter.create(result.image);
      const updatedBuffer = exifWriter.encode();
      fs.writeFile(imagePath, updatedBuffer, (err) => {
        if (err) {
          console.error('Error writing updated image:', err);
        } else {
          console.log('Metadata deleted successfully.');
        }
      });
    }
  });
}

// Define command-line options
commander
  .option('-e, --edit', 'Edit metadata')
  .option('-d, --delete', 'Delete metadata')
  .option('-t, --title <title>', 'Set image title')
  .option('-f, --fieldToDelete <field>', 'Delete specific metadata field')
  .parse(process.argv);

// Call the function with the provided options
if (commander.edit || commander.delete) {
  editOrDeleteMetadata(imagePath, {
    edit: commander.edit,
    delete: commander.delete,
    title: commander.title,
    fieldToDelete: commander.fieldToDelete,
  });
} else {
  console.log('No action specified. Use --edit or --delete.');
}
