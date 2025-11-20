# Example Debugging Session

This document shows a complete debugging session using the MLE-Bench Debugging Assistant.

---

## Session 1: KeyError in Data Loading (Fixable)

### INPUT

**Competition ID:** siim-covid19-detection

**Debug Step:** 0

#### Buggy Code

```python
import pandas as pd
import torch
from torch.utils.data import Dataset

class StudyClsDataset(Dataset):
    def __init__(self, image_csv_path, study_csv_path):
        self.image_df = pd.read_csv(image_csv_path)
        self.study_df = pd.read_csv(study_csv_path)

        # Create dictionary mapping study_id to labels
        self.study_labels = {}
        for _, row in self.study_df.iterrows():
            study_id = row['study_id']
            labels = [
                row['Negative for Pneumonia'],
                row['Typical Appearance'],
                row['Indeterminate Appearance'],
                row['Atypical Appearance']
            ]
            self.study_labels[study_id] = labels

    def __len__(self):
        return len(self.image_df)

    def __getitem__(self, idx):
        row = self.image_df.iloc[idx]

        # Get image path
        image_id = row['image_id']
        image_path = f"./data/train/{image_id}.png"

        # Get study-level labels
        study_id = row['study_id']
        target_vec = self.study_labels[study_id]  # LINE 89
        target = torch.tensor(target_vec, dtype=torch.float32)

        return image_path, target

# Create dataset
dataset = StudyClsDataset('./data/train_image_level.csv', './data/train_study_level.csv')
print(f"Dataset size: {len(dataset)}")

# Test loading first item
item = dataset[0]
print("Successfully loaded first item")
```

#### Error Output

```
Traceback (most recent call last):
  File "train.py", line 45, in <module>
    item = dataset[0]
  File "train.py", line 89, in __getitem__
    target_vec = self.study_labels[study_id]
KeyError: 'f6293b1c49e2'
```

#### Client Analysis

The code crashes when trying to access the first item. The error suggests that the study_id from the image CSV doesn't exist in the study CSV mapping dictionary.

#### Client Plan

Need to handle missing study IDs somehow - maybe skip them or use default values.

---

### OUTPUT

| Field | Value |
|-------|-------|
| **bug_confirmed** | TRUE |
| **proposed_debug_analysis_accurate** | 2 |
| **initial_bug_reproducible** | TRUE |
| **bug_fixed** | TRUE |
| **all_bugs_fixed** | FALSE |

**revised_analysis:**
The code throws a KeyError at line 89 in StudyClsDataset.__getitem__ when attempting to look up study_id 'f6293b1c49e2' in the self.study_labels dictionary. The error occurs because not all study_ids referenced in the image-level CSV (train_image_level.csv) have corresponding entries in the study-level CSV (train_study_level.csv), causing dictionary lookup failures when the dataset attempts to retrieve labels for images whose studies lack entries. The code lacks defensive checks for missing keys, assuming perfect one-to-one correspondence between image studies and labeled studies, which results in immediate crashes when encountering orphaned images.

**revised_plan:**
**Bug Fix Plan**
1. Modify the __getitem__ method at line 89 to check if study_id exists in self.study_labels before attempting the dictionary lookup
2. Store row['study_id'] in the study_id variable, then use the 'in' operator to check membership: `if study_id in self.study_labels`
3. If the study_id is found, use the original logic to retrieve target_vec and create the tensor
4. If the study_id is not found, return a default tensor of zeros: `torch.zeros(4, dtype=torch.float32)` to represent unknown/missing labels
5. Keep all other code unchanged including the dictionary creation logic, CSV loading, and dataset length calculation

**Fixed Code:**

```python
    def __getitem__(self, idx):
        row = self.image_df.iloc[idx]

        # Get image path
        image_id = row['image_id']
        image_path = f"./data/train/{image_id}.png"

        # Get study-level labels with safety check
        study_id = row['study_id']

        # CHANGED: Add defensive check for missing study_id
        if study_id in self.study_labels:
            target_vec = self.study_labels[study_id]
        else:
            # Default to zeros for missing study labels
            target_vec = [0.0, 0.0, 0.0, 0.0]

        target = torch.tensor(target_vec, dtype=torch.float32)

        return image_path, target
```

**Time to fix:** 2-3 minutes

---

## Session 2: ZIP Extraction to Read-Only Directory (Fixable)

### INPUT

**Competition ID:** rsna-pneumonia-detection

**Debug Step:** 0

#### Buggy Code

```python
import zipfile
import pandas as pd
import os

def prepare_data():
    print("Extracting training data...")

    # Extract train and test zips
    with zipfile.ZipFile('./data/train.zip', 'r') as zip_ref:
        zip_ref.extractall('./data/train/')  # LINE 12

    with zipfile.ZipFile('./data/test.zip', 'r') as zip_ref:
        zip_ref.extractall('./data/test/')

    print("Data extraction complete")

    # Load CSV
    train_df = pd.read_csv('./data/train.csv')
    print(f"Loaded {len(train_df)} training samples")

if __name__ == "__main__":
    prepare_data()
```

#### Error Output

```
Extracting training data...
Traceback (most recent call last):
  File "prepare_data.py", line 23, in <module>
    prepare_data()
  File "prepare_data.py", line 12, in prepare_data
    zip_ref.extractall('./data/train/')
  File "/usr/lib/python3.8/zipfile.py", line 1634, in extractall
    self._extract_member(zipinfo, path, pwd)
  File "/usr/lib/python3.8/zipfile.py", line 1687, in _extract_member
    return self._extract_member_to_path(zipinfo, targetpath, pwd)
  File "/usr/lib/python3.8/zipfile.py", line 1718, in _extract_member_to_path
    with open(targetpath, "wb") as target:
OSError: [Errno 30] Read-only file system: './data/train/sample_001.dcm'
```

#### Client Analysis

The extraction is failing because the ./data/ directory appears to be read-only in the container environment.

#### Client Plan

Extract to a different directory that has write permissions, like the current directory or /tmp/.

---

### OUTPUT

| Field | Value |
|-------|-------|
| **bug_confirmed** | TRUE |
| **proposed_debug_analysis_accurate** | 2 |
| **initial_bug_reproducible** | TRUE |
| **bug_fixed** | TRUE |
| **all_bugs_fixed** | TRUE |

**revised_analysis:**
The code throws an OSError at line 12 in prepare_data() when attempting to extract train.zip to the './data/train/' directory. The error occurs because the ./data/ directory is mounted as read-only in the container environment, preventing any write operations including file extraction. The code assumes write permissions to the data directory, but competition environments typically provide read-only access to original data files while requiring extracted or generated files to be written to the working directory or other writable locations.

**revised_plan:**
**Bug Fix Plan**
1. Change the extractall() destination at line 12 from './data/train/' to './train/' (current directory)
2. Change the extractall() destination at line 14 from './data/test/' to './test/' (current directory)
3. This allows extracted files to be written to the writable working directory while still reading the original ZIPs from read-only ./data/
4. Keep all other code unchanged including CSV loading logic (CSVs can be read directly from read-only ./data/)

**Fixed Code:**

```python
def prepare_data():
    print("Extracting training data...")

    # Extract train and test zips to writable directory
    with zipfile.ZipFile('./data/train.zip', 'r') as zip_ref:
        zip_ref.extractall('./train/')  # CHANGED: Extract to current dir

    with zipfile.ZipFile('./data/test.zip', 'r') as zip_ref:
        zip_ref.extractall('./test/')  # CHANGED: Extract to current dir

    print("Data extraction complete")

    # Load CSV
    train_df = pd.read_csv('./data/train.csv')
    print(f"Loaded {len(train_df)} training samples")
```

**Time to fix:** 1-2 minutes

---

## Session 3: Missing Mask Directory - Architecture Issue (Unfixable)

### INPUT

**Competition ID:** organ-segmentation-2023

**Debug Step:** 0

#### Buggy Code

```python
import os
import numpy as np
from PIL import Image
import torch
from torch.utils.data import Dataset

class OrganSegmentationDataset(Dataset):
    def __init__(self, data_dir, split='train'):
        self.data_dir = data_dir
        self.split = split

        # Get list of cases
        self.cases = []
        split_dir = os.path.join(data_dir, split)

        for case in os.listdir(split_dir):
            case_path = os.path.join(split_dir, case)
            if os.path.isdir(case_path):
                # Get all day directories for this case
                for day in os.listdir(case_path):
                    day_path = os.path.join(case_path, day)
                    if os.path.isdir(day_path):
                        self.cases.append((case, day))

        print(f"Found {len(self.cases)} samples")

    def __len__(self):
        return len(self.cases)

    def __getitem__(self, idx):
        case, day = self.cases[idx]

        # Load scan
        scan_dir = os.path.join(self.data_dir, self.split, case, day, 'scans')
        scan_files = sorted(os.listdir(scan_dir))
        scan_path = os.path.join(scan_dir, scan_files[0])
        scan = Image.open(scan_path)
        scan_array = np.array(scan)

        # Load mask - LINE 71
        mask_dir = os.path.join(self.data_dir, self.split, case, day, 'masks')
        mask_files = sorted(os.listdir(mask_dir))
        mask_path = os.path.join(mask_dir, mask_files[0])
        mask = Image.open(mask_path)
        mask_array = np.array(mask)

        return torch.from_numpy(scan_array), torch.from_numpy(mask_array)

# Create dataset
dataset = OrganSegmentationDataset('./data', split='train')
sample = dataset[0]
```

#### Error Output

```
Found 82 samples
Traceback (most recent call last):
  File "train.py", line 50, in <module>
    sample = dataset[0]
  File "train.py", line 71, in __getitem__
    mask_files = sorted(os.listdir(mask_dir))
FileNotFoundError: [Errno 2] No such file or directory: './data/train/case77/case77_day20/masks'
```

#### Client Analysis

The masks directory doesn't exist in the expected location. Maybe masks are stored differently or need to be generated from the CSV file.

#### Client Plan

Check if masks are in the CSV as RLE strings and need to be decoded, or if they're in a different directory structure.

---

### OUTPUT

| Field | Value |
|-------|-------|
| **bug_confirmed** | TRUE |
| **proposed_debug_analysis_accurate** | 1 |
| **initial_bug_reproducible** | TRUE |
| **bug_fixed** | FALSE |
| **all_bugs_fixed** | FALSE |

**revised_analysis:**
The code throws a FileNotFoundError at line 71 in __getitem__ when attempting to list files in the './data/train/case77/case77_day20/masks/' directory, which doesn't exist in the dataset structure. The error occurs because the code's data loading architecture assumes masks are stored as separate image files in a parallel directory structure to scans/, when in fact the competition dataset stores segmentation masks as run-length encoded (RLE) strings in the train.csv file alongside case metadata. The code's design fundamentally conflicts with the actual data format, expecting a filesystem-based mask storage system that doesn't match the competition's CSV-based RLE storage approach.

**revised_plan:**
This bug cannot be fixed with minimal changes. The code's architecture assumes masks exist as PNG/TIFF files in a masks/ subdirectory parallel to scans/, but the actual dataset stores masks as RLE-encoded strings in train.csv indexed by case and day identifiers. Fixing this would require: (1) completely rewriting the __init__ method to parse train.csv instead of using os.listdir() to enumerate cases, (2) implementing RLE-to-binary-mask decoding functionality (typically 20-30 lines of specialized code), (3) redesigning the __getitem__ method to look up masks by case/day ID in the CSV DataFrame rather than loading from filesystem paths, and (4) changing how scan-mask pairs are created from parallel sorted file lists to CSV-based ID matching. These changes constitute a fundamental redesign of the entire dataset class's data loading paradigm, going far beyond a minimal localized bugfix and instead requiring a complete architectural rewrite of how data is accessed and paired.

**Time to fix:** N/A (unfixable with minimal changes)

---

## Key Takeaways

### Fixable Bugs
- **KeyError**: Add defensive checks (2-3 min)
- **Read-only filesystem**: Change extraction paths (1-2 min)
- **Missing libraries**: Add try/except pip install (2-3 min)
- **7z files**: Install py7zr and extract (3-5 min)

### Unfixable Bugs
- Architecture mismatches requiring rewrites
- Missing internet access for model downloads
- System dependencies unavailable in container
- Data format incompatibilities requiring complete redesign

### Best Practices
1. Always provide full tracebacks
2. Include exact line numbers in analysis
3. Explain causal chain from root cause to error
4. Keep fixes minimal (4-8 lines typically)
5. Test that proposed fixes would actually work
6. Be honest about unfixable bugs - don't force solutions

---

**End of Example Session**
