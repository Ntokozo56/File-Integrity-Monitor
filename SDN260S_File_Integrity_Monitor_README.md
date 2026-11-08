# File Integrity Monitor (FIM)

A Java desktop application that detects changes to files in a selected directory by comparing their current SHA-256 hashes against a saved baseline. The project provides both a command-line interface (CLI) and a Java Swing graphical user interface (GUI).

## Overview

File integrity monitoring helps identify unexpected changes to files. This application scans a directory, records file information and SHA-256 hashes, and later compares a fresh scan with the original baseline to identify **modified, new, deleted, and unchanged files**.

## Features

- **SHA-256 hashing:** Generates content hashes for scanned files.
- **Recursive directory scanning:** Processes files within the selected directory and its subdirectories.
- **Baseline management:** Creates, saves, and loads a reference snapshot of file records.
- **Integrity checking:** Identifies added, modified, deleted, and unchanged files.
- **Parallel baseline scanning:** Uses concurrent processing when creating a baseline.
- **Two interfaces:** Choose between an interactive console menu and a Swing desktop GUI.
- **Reports:** Displays a summary and detailed lists of detected changes, with an option to save a text report.
- **Application logging:** Records application activity and errors.

## Technologies

- Java (NetBeans project configuration targets **Java 27**)
- Java Swing for the GUI
- Java standard library for file I/O, SHA-256 hashing, serialization, and concurrency
- Apache Ant / NetBeans build system

## Requirements

- A JDK compatible with the project's configured source level (**JDK 27**, or adjust the source/target settings for an older compatible JDK).
- Apache NetBeans IDE for the easiest setup, or Apache Ant for command-line builds.
- A desktop environment to use the GUI; the CLI can be used in a terminal.

No external third-party runtime libraries are declared in the project.

## Getting Started

### Run with NetBeans

1. Extract the project archive.
2. Open **Apache NetBeans**.
3. Select **File → Open Project** and choose the `SDN_P` directory.
4. Configure a compatible JDK if prompted.
5. Run the project. The entry point is `fim.Main`.
6. At the startup prompt, enter `1` for the CLI or `2` for the GUI.

### Build and run from the terminal

From the `SDN_P` project directory:

```bash
ant clean jar
java -jar dist/SDN_P.jar
```

The included archive also contains a prebuilt `dist/SDN_P.jar`, which you can try running directly with a compatible Java runtime:

```bash
java -jar dist/SDN_P.jar
```

**Tip:** Run the application from the project directory so its relative `data/`, `logs/`, and `reports/` paths resolve as expected.

## How to Use

### Command-Line Interface (CLI)

After choosing `1` at startup, the application displays these options:

| Option | Action |
|---|---|
| 1 | Select directory |
| 2 | Create baseline |
| 3 | Load baseline |
| 4 | Check integrity |
| 5 | View last report |
| 6 | Save report |
| 7 | Exit |

**Typical workflow:**

1. Select a directory to monitor.
2. Create a baseline before making changes. It is saved to `data/baseline.dat`.
3. Add, edit, or delete files in the monitored directory.
4. Select **Check integrity** to compare the current files with the baseline.
5. View the report and optionally save it to `reports/integrity-report.txt`.

If you restart the application, select the same directory and use **Load baseline** before checking integrity.

### Graphical User Interface (GUI)

After choosing `2` at startup:

1. Use **Browse** to select a directory.
2. Click **Create Baseline** to record the initial state.
3. Make changes to files in the directory.
4. Click **Check Integrity** to see the results.
5. Click **Save Report** to export the report, or **Clear** to clear the displayed output.

The GUI creates a baseline in the current session; unlike the CLI, it does not expose a separate **Load Baseline** button.

## Suggested Demonstration and Verification

To verify the program during assessment, create a separate **test folder** containing a few disposable text files. Create a baseline, then perform these tests one at a time, checking the resulting status:

| Test | Action | Expected outcome |
|---|---|---|
| Unchanged file | Run an integrity check without changing the files | Unchanged |
| Modified file | Edit and save an existing text file | Modified |
| New file | Add another file to the monitored folder | New |
| Deleted file | Delete a file that was present when the baseline was created | Deleted |
| Nested folder | Add or modify a file in a subfolder | The change is included in the scan |
| Missing baseline | Attempt to check integrity before creating or loading a baseline | An error explaining that a baseline is needed |
| Invalid directory | Supply a nonexistent or non-directory path in the CLI | The application should reject the selection |

These are **recommended tests**, not a claim that the group has already executed or passed them. Record actual results and screenshots in the separate system design report. For consistent comparisons, do not recreate the baseline between change tests unless intentionally starting a new test cycle.

## Understanding the Results

| Status | Meaning |
|---|---|
| Unchanged | The file matches the saved baseline. |
| Modified | An existing file's hash differs from its baseline hash. |
| New | A file exists now but was not in the baseline. |
| Deleted | A baseline file is no longer present in the current scan. |

The report includes totals and detailed information for modified, new, and deleted files.

## Project Structure

```text
SDN_P/
├── src/
│   └── fim/
│       ├── Main.java                  # Startup and interface selection
│       ├── cli/ConsoleMenu.java       # Command-line menu
│       ├── gui/FileIntegrityGUI.java  # Swing interface
│       ├── hashing/                   # SHA-256 hashing implementation
│       ├── scanner/FileScanner.java   # Directory scanning
│       ├── comparison/FileComparator.java
│       ├── model/                     # File records and scan results
│       ├── service/IntegrityService.java
│       ├── storage/BaselineManager.java
│       ├── reporting/ReportGenerator.java
│       └── logging/ApplicationLogger.java
├── data/                              # Serialized baseline
├── logs/                              # Application log
├── build/                             # Generated build files
├── dist/                              # Built JAR and distribution notes
├── nbproject/                         # NetBeans configuration
├── build.xml                          # Ant build script
└── manifest.mf
```

The `reports/` directory is created when a report is saved.

## How It Works

1. **Scan:** Read the files in the selected directory.
2. **Hash:** Calculate each file's SHA-256 digest.
3. **Baseline:** Store the reference file records in a serialized baseline file.
4. **Compare:** Rescan the directory and compare current records against the saved baseline.
5. **Report:** Summarize and display the changes; optionally write them to a text file.

## Limitations and Security Notes

- This is a **change-detection** tool; it does not prevent file modification or automatically restore files.
- Scans are initiated by the user rather than performed continuously in the background.
- A trustworthy baseline should be created when the monitored files are known to be in a good state.
- Protect `data/baseline.dat` from unauthorized changes; tampering with the baseline can undermine detection.
- The baseline is stored using Java object serialization. Only load baseline files from trusted sources.
- The application stores its baseline at a shared relative path, so creating another baseline can replace the previous one.
- A matching hash is evidence that the file content matches the baseline; it does not establish who made a change or why.

## Coursework Alignment

This implementation addresses the assigned **File Integrity Monitor** topic by recording hashes and comparing later scans. The source is organized into separate classes for scanning, hashing, comparison, persistence, reporting, logging, and user interfaces. It demonstrates recursion for directory traversal, Java collections for file records, serialization for baseline storage, and an executor-based parallel baseline scan.

**Separate submission requirements:** The project brief also requires a **5–10-page system design document**, including a UML class diagram, design and algorithms, testing evidence, GA2 and GA7 reflections, and a team contribution breakdown. The group must submit the source project as a ZIP named `GroupNumber_ProjectTitle` and present a **7–10-minute demo**. This README does not replace those deliverables.

## Potential Future Improvements

- Automatic scheduled or real-time monitoring
- Baseline selection and loading through the GUI
- Multiple named baselines for different directories
- CSV or JSON report export
- Notifications when unexpected changes are detected
- Automated tests and packaged releases
