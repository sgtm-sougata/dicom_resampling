To enhance the script to also remove blank contours (i.e., contours with no actual contour data) along with duplicate ROIs and contours, you need to check if a contour has no data. 

Here’s how you can modify the script to handle the removal of both duplicate and blank contours:

### Modified Python Script

```python
import pydicom
from pydicom.dataset import Dataset

def remove_duplicate_and_blank_rois_and_contours(rtstruct_path, output_path):
    # Load the existing RTSTRUCT file
    ds = pydicom.dcmread(rtstruct_path)

    if 'StructureSetROISequence' not in ds:
        raise ValueError("No StructureSetROISequence found in the DICOM file.")

    if 'ROIContourSequence' not in ds:
        raise ValueError("No ROIContourSequence found in the DICOM file.")
    
    # Create dictionaries to track unique ROIs and contours
    roi_dict = {}
    unique_rois = []
    unique_contours = []
    
    # Process ROIs
    for roi in ds.StructureSetROISequence:
        roi_name = roi.ROIName
        roi_number = roi.ROINumber
        
        if roi_name not in roi_dict:
            roi_dict[roi_name] = roi_number
            unique_rois.append(roi)

    # Track used ROI numbers
    roi_numbers = {roi.ROINumber for roi in unique_rois}

    # Process contours
    contour_dict = {}
    for contour in ds.ROIContourSequence:
        referenced_roi_number = contour.ReferencedROINumber

        # Check if the referenced ROI number is valid
        if referenced_roi_number in roi_numbers:
            # Check if the contour data is not empty
            if hasattr(contour, 'ContourSequence') and len(contour.ContourSequence) > 0:
                roi_name = None
                for roi in unique_rois:
                    if roi.ROINumber == referenced_roi_number:
                        roi_name = roi.ROIName
                        break
                
                if roi_name:
                    if (roi_name, referenced_roi_number) not in contour_dict:
                        contour_dict[(roi_name, referenced_roi_number)] = contour
                        unique_contours.append(contour)

    # Update the dataset with unique ROIs and contours
    ds.StructureSetROISequence = unique_rois
    ds.ROIContourSequence = unique_contours
    
    # Save the modified RTSTRUCT file
    ds.save_as(output_path)

# Path to your existing RTSTRUCT file
rtstruct_path = 'path/to/your/existing_rtstruct.dcm'

# Path to save the modified RTSTRUCT file
output_path = 'path/to/your/cleaned_rtstruct.dcm'

# Remove duplicate and blank ROIs and contours
remove_duplicate_and_blank_rois_and_contours(rtstruct_path, output_path)
```

### Explanation

1. **Load the Existing RTSTRUCT File**: The DICOM file is loaded using `pydicom.dcmread()`.

2. **Track Unique ROIs**:
   - Maintain a dictionary (`roi_dict`) to track ROIs by name.
   - Store unique ROIs in the `unique_rois` list.

3. **Track Unique and Non-Blank Contours**:
   - Maintain a dictionary (`contour_dict`) to track contours by a combination of ROI name and referenced ROI number.
   - Only add contours to `unique_contours` if they have non-empty contour data. This is checked using the presence and length of `ContourSequence` in the contour dataset.

4. **Update and Save Dataset**:
   - Replace the existing sequences with the filtered lists of unique ROIs and contours.
   - Save the updated DICOM file to the specified output path.

### Additional Considerations

- **Contour Data Check**: The script checks if the `ContourSequence` attribute is present and not empty. If your DICOM files use a different attribute or format, adjust the check accordingly.

- **DICOM Compliance**: Ensure that the modified file maintains DICOM compliance.

- **File Validation**: After running the script, use a DICOM viewer or validator to confirm that the modifications are correct and the file remains valid.

This script provides a comprehensive approach to handling duplicates and blank contours in RTSTRUCT files. Adjust the script as needed to fit specific requirements or file formats in your use case.
