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

ERROR_COUNT=$(grep -Ei 'ERROR|Failed' "$LOG_PATH" | wc -l)

echo "The total number of error count is: $ERROR_COUNT"

```
<img width="1191" height="107" alt="image" src="https://github.com/user-attachments/assets/2eb3c8bf-5442-400c-9281-b19c67761112" />


Task 3: Critical Events

```
Search for lines containing the keyword CRITICAL
Print those lines along with their line number
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

ERROR_COUNT=$(grep -Ei 'ERROR|Failed' "$LOG_PATH" | wc -l)

echo "The total number of error count is: $ERROR_COUNT"

echo "searching for CRITICAL"

grep -n "CRITICAL" "$LOG_PATH" | while IFS=: read -r line_num line_content
do
        echo "Line $line_num: $line_content"
done

```
Example output:

--- Critical Events ---
Line 84: 2025-07-29 10:15:23 CRITICAL Disk space below threshold
Line 217: 2025-07-29 14:32:01 CRITICAL Database connection lost

<img width="1191" height="145" alt="image" src="https://github.com/user-attachments/assets/d86a9810-0edd-4d1b-b41f-543270e8fccb" />

Task 4: Top Error Messages

```
Extract all lines containing ERROR
Identify the top 5 most common error messages
Display them with their occurrence count, sorted in descending order

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

#Task 2

ERROR_COUNT=$(grep -Ei 'ERROR|Failed' "$LOG_PATH" | wc -l)

echo "The total number of error count is: $ERROR_COUNT"

#Task 3

echo "searching for CRITICAL"

grep -n "CRITICAL" "$LOG_PATH" | while IFS=: read -r line_num line_content
do
        echo "Line $line_num: $line_content"
done

#Task 4

echo "--- Top 5 Error Messages ---"
grep "ERROR" "$LOG_PATH" \
    | sed 's/.*ERROR[[:space:]]*//' \
    | sort \
    | uniq -c \
    | sort -nr \
    | head -5

```
sed 's/.*ERROR[[:space:]]*//' → removes everything before the actual error message

sort | uniq -c → groups and counts identical messages

Example output:

--- Top 5 Error Messages ---
45 Connection timed out
32 File not found
28 Permission denied
15 Disk I/O error
9  Out of memory


Task 5: Summary Report
Generate a summary report to a text file named log_report_<date>.txt (e.g., log_report_2026-02-11.txt). The report should include:

```
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Error: No such file exists"
    echo "Please enter ./log_analyser.sh <log-file-path>"
    exit 1
fi    

LOG_PATH="$1"

if [ ! -f "$LOG_PATH" ]; then
    echo "No such file please enter correct file path"
    exit 1
fi

DATE=$(date +"%Y-%m-%d")
TIME=$(date +%H-%M)
REPORT_FILE="log_report_${DATE}_${TIME}.txt"

TOTAL_LINES=$(wc -l < "$LOG_PATH")

# Task 2: Error Count
ERROR_COUNT=$(grep -Ei 'ERROR|Failed' "$LOG_PATH" | wc -l)

# Task 3: Critical Events
CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_PATH" | while IFS=: read -r line_num line_content
do
    echo "Line $line_num: $line_content"
done)

# Task 4: Top Error Messages
TOP_ERRORS=$(grep "ERROR" "$LOG_PATH" \
    | sed 's/.*ERROR[[:space:]]*//' \
    | sort \
    | uniq -c \
    | sort -nr \
    | head -5)

# Task 5: Generate Report
{
echo "===== Log Analysis Report ====="
echo "Date of Analysis: $DATE"
echo "Log File: $LOG_PATH"
echo "Total Lines Processed: $TOTAL_LINES"
echo "Total Error Count: $ERROR_COUNT"
echo ""

echo "--- Top 5 Error Messages ---"
echo "$TOP_ERRORS"
echo ""

echo "--- Critical Events ---"
echo "$CRITICAL_EVENTS"

} > "$REPORT_FILE"

echo "Report generated: $REPORT_FILE"

```

Date of analysis
Log file name
Total lines processed
Total error count
Top 5 error messages with their occurrence count
List of critical events with line numbers

<img width="1191" height="79" alt="image" src="https://github.com/user-attachments/assets/c868631c-c2f0-4538-b417-7f70197b3ac4" />
<img width="1191" height="386" alt="image" src="https://github.com/user-attachments/assets/8620da7d-c233-4c01-a564-0ba63d1f6a2e" />


Task 6 (Optional): Archive Processed Logs
```
Add a feature to:

Create an archive/ directory if it doesn't exist
Move the processed log file into archive/ after analysis
Print a confirmation message

#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Error: No such file exists"
    echo "Please enter ./log_analyser.sh <log-file-path>"
    exit 1
fi    

LOG_PATH="$1"

if [ ! -f "$LOG_PATH" ]; then
    echo "No such file please enter correct file path"
    exit 1
fi

DATE=$(date +"%Y-%m-%d")
TIME=$(date +%H-%M)
REPORT_FILE="log_report_${DATE}_${TIME}.txt"
ARCHIVE_DIR="archive"

TOTAL_LINES=$(wc -l < "$LOG_PATH")

# Task 2: Error Count
ERROR_COUNT=$(grep -Ei 'ERROR|Failed' "$LOG_PATH" | wc -l)

# Task 3: Critical Events
CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_PATH" | while IFS=: read -r line_num line_content
do
    echo "Line $line_num: $line_content"
done)

# Task 4: Top Error Messages
TOP_ERRORS=$(grep "ERROR" "$LOG_PATH" \
    | sed 's/.*ERROR[[:space:]]*//' \
    | sort \
    | uniq -c \
    | sort -nr \
    | head -5)

# Task 5: Generate Report
{
echo "===== Log Analysis Report ====="
echo "Date of Analysis: $DATE"
echo "Log File: $LOG_PATH"
echo "Total Lines Processed: $TOTAL_LINES"
echo "Total Error Count: $ERROR_COUNT"
echo ""

echo "--- Top 5 Error Messages ---"
echo "$TOP_ERRORS"
echo ""

echo "--- Critical Events ---"
echo "$CRITICAL_EVENTS"

} > "$REPORT_FILE"

echo "Report generated: $REPORT_FILE"

# Task 6: Archive Processed Logs
if [ ! -d "$ARCHIVE_DIR" ]; then
    mkdir "$ARCHIVE_DIR"
fi

mv "$REPORT_FILE" "$ARCHIVE_DIR"/

echo "Log file moved to $ARCHIVE_DIR/"


```
<img width="1191" height="302" alt="image" src="https://github.com/user-attachments/assets/9d56d224-ed5e-4058-b17f-4056e96c1ca1" />

