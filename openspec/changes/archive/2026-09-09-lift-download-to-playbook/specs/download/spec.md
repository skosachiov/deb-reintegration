## Purpose

Shared Debian source download logic that fetches a DSC file, extracts source file references, and downloads all source tarballs from any Debian/Ubuntu source package URL.

## ADDED Requirements

### Requirement: Download DSC file from URL
The shared download tasks SHALL accept a DSC file URL as input and download the DSC file to the build directory.

#### Scenario: DSC file downloaded
- **WHEN** the download tasks are invoked with a valid `dsc_url` and `build_dir`
- **THEN** the DSC file is saved at `build_dir/<dsc_filename>`

### Requirement: Derive package metadata from DSC URL
The shared download tasks SHALL compute `package_name`, `build_dir`, `dsc_dir`, `source_dirname`, and `source_dir` from the DSC URL and build root, without requiring caller-provided values for these derived variables.

#### Scenario: Metadata derived from DSC URL
- **WHEN** the download tasks are invoked with `dsc_url` and `build_root`
- **THEN** `package_name` is the DSC basename split on `_` and taking the first field, `build_dir` is `build_root/package_name`, `dsc_dir` is the dirname of `dsc_url`, `source_dirname` is the DSC basename without `.dsc` extension, and `source_dir` is `build_dir/source_dirname`

### Requirement: Download additional source files
The shared download tasks SHALL download an optional list of additional source files from a base URL when `source_files` is provided and non-empty.

#### Scenario: Additional source files downloaded
- **WHEN** `base_url` and `source_files` are provided and `source_files` is non-empty
- **THEN** each file in `source_files` is downloaded from `<base_url>/<filename>` to `build_dir/<filename>`

#### Scenario: No additional source files
- **WHEN** `source_files` is not provided or is empty
- **THEN** no additional source files are downloaded and the task is skipped

### Requirement: Extract and download DSC-referenced source files
The shared download tasks SHALL parse the DSC file's `Files:` section to extract source filenames, then download each referenced source file from the DSC directory URL.

#### Scenario: DSC source files downloaded
- **WHEN** the DSC file has been downloaded and contains a `Files:` section
- **THEN** all source filenames listed in the `Files:` section are downloaded from `<dsc_dir>/<filename>` to `build_dir/<filename>`

#### Scenario: DSC file has no Files section
- **WHEN** the DSC file does not contain a `Files:` section
- **THEN** no additional files are downloaded and a debug message is displayed

### Requirement: Ensure build directory exists
The shared download tasks SHALL create the build directory if it does not already exist.

#### Scenario: Build directory created
- **WHEN** `build_dir` does not exist
- **THEN** the directory is created with mode 0755

### Requirement: Display download summary
The shared download tasks SHALL display a debug message summarizing the build directory, package name, DSC filename, and total number of source files downloaded.

#### Scenario: Summary displayed
- **WHEN** download completes
- **THEN** a debug message shows build directory, package name, DSC filename, and count of downloaded source files

### Required inputs
The shared download tasks SHALL require the following variables at include time:
- `dsc_url` — URL of the DSC file
- `build_root` — root build directory

Optional variables:
- `base_url` — base URL for additional source files (default: derived from DSC URL dirname)
- `source_files` — list of additional source filenames to download (default: empty)
