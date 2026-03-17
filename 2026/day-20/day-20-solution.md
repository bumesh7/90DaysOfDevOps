Task 1: Input and Validation

```
Accept the path to a log file as a command-line argument
Exit with a clear error message if no argument is provided
Exit with a clear error message if the file doesn't exist

#!/bin/bash

if [ $# -eq 0  ]; then
	echo "Error: No such file exists"
	echo "Please enter ./log_analyser.sh <log-file-path>"
	exit 1
fi	

LOG_PATH="$1"

if [ ! -f "$LOG_PATH" ]; then
	echo "No such file please enter correct file path"
        exit 1
fi

```
<img width="783" height="106" alt="image" src="https://github.com/user-attachments/assets/60b5e18b-d8df-4300-9982-842b6122b10d" />
<img width="853" height="106" alt="image" src="https://github.com/user-attachments/assets/0bd80c8f-dd9d-42d5-8055-e14bbc45f7db" />


Task 2: Error Count

```
Count the total number of lines containing the keyword ERROR or Failed
Print the total error count to the console
```

Task 3: Critical Events
Search for lines containing the keyword CRITICAL
Print those lines along with their line number
Example output:

--- Critical Events ---
Line 84: 2025-07-29 10:15:23 CRITICAL Disk space below threshold
Line 217: 2025-07-29 14:32:01 CRITICAL Database connection lost
Task 4: Top Error Messages
Extract all lines containing ERROR
Identify the top 5 most common error messages
Display them with their occurrence count, sorted in descending order
Example output:

--- Top 5 Error Messages ---
45 Connection timed out
32 File not found
28 Permission denied
15 Disk I/O error
9  Out of memory
Task 5: Summary Report
Generate a summary report to a text file named log_report_<date>.txt (e.g., log_report_2026-02-11.txt). The report should include:

Date of analysis
Log file name
Total lines processed
Total error count
Top 5 error messages with their occurrence count
List of critical events with line numbers
Task 6 (Optional): Archive Processed Logs
Add a feature to:

Create an archive/ directory if it doesn't exist
Move the processed log file into archive/ after analysis
Print a confirmation message
