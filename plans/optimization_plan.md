# Optimization Plan for ComfyS3

## Problem
The `LoadImageS3` and `SaveImageS3` nodes are causing high latency because they perform full bucket scans (listing thousands of files) during initialization and save operations.

## Proposed Changes

### 1. Optimize `LoadImageS3` Node
**File:** `src/nodes/load_image_s3.py`
- **Current Behavior:** `INPUT_TYPES` calls `S3_INSTANCE.get_files()` which lists all files in the S3 bucket to populate a dropdown menu.
- **Change:** Replace the dropdown menu with a simple text input (`STRING`) for the image path.
- **Benefit:** Eliminates the initial bucket scan entirely.

### 2. Optimize `SaveImageS3` Node
**File:** `src/nodes/save_image_s3.py`
- **Current Behavior:** `save_images` calls `S3_INSTANCE.get_save_path` which scans the output folder to calculate the next file counter (e.g., `Image_00001_.png`).
- **Change:** Add an `overwrite_existing` boolean input.
- **Benefit:** When enabled, this will bypass the counter calculation and file scanning, allowing direct file saving.

### 3. Optimize `S3 Client` Logic
**File:** `src/client_s3.py`
- **Change 1 (`does_folder_exist`):** Update to use `MaxKeys=1`. The current implementation iterates through results; limiting to 1 key makes the check instant if any file exists.
- **Change 2 (`get_save_path`):** Update to accept an `overwrite` flag. If `overwrite` is True, skip the file listing and counter calculation, and just return the base filename.

## Verification
- Confirm that `LoadImageS3` no longer triggers S3 list operations on startup.
- Confirm that `SaveImageS3` with `overwrite_existing=True` does not list files.
- Confirm `does_folder_exist` is faster.
