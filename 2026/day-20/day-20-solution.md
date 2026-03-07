# Day 20 task
```bash
#!/bin/bash

# --- Task 1: Input and Validation ---
if [ -z "$1" ]; then
    echo "Usage: $0 <path_to_log_file>"
    exit 1
fi

LOG_FILE=$1

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' not found."
    exit 1
fi

# Variables for the report
REPORT_DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_$REPORT_DATE.txt"

# --- Task 2: Error Count ---
# Counting lines with ERROR or Failed (case-insensitive for robustness)
ERROR_COUNT=$(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)
echo "Total Errors/Failures found: $ERROR_COUNT"

# --- Task 3: Critical Events ---
echo "--- Critical Events ---"
# -n prints line numbers
CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_FILE")
echo "$CRITICAL_EVENTS"

# --- Task 4: Top 5 Error Messages ---
# We use awk to strip the timestamp/date (assuming first 3 columns) to group similar messages
TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" | awk '{ $1=$2=$3=""; print $0 }' | sed 's/^[ \t]*//' | sort | uniq -c | sort -rn | head -5)

# --- Task 5: Generate Summary Report ---
{
    echo "==============================================="
    echo "LOG ANALYSIS REPORT - $REPORT_DATE"
    echo "==============================================="
    echo "Log File: $LOG_FILE"
    echo "Total Lines Processed: $(wc -l < "$LOG_FILE")"
    echo "Total Error/Failed Count: $ERROR_COUNT"
    echo ""
    echo "--- Top 5 Error Messages ---"
    echo "$TOP_ERRORS"
    echo ""
    echo "--- Critical Events (Line Numbers) ---"
    if [ -z "$CRITICAL_EVENTS" ]; then
        echo "No critical events found."
    else
        echo "$CRITICAL_EVENTS"
    fi
} > "$REPORT_FILE"

echo "Report generated: $REPORT_FILE"

# --- Task 6: Archive Processed Logs ---
ARCHIVE_DIR="archive"
mkdir -p "$ARCHIVE_DIR"
mv "$LOG_FILE" "$ARCHIVE_DIR/"
echo "File '$LOG_FILE' has been moved to $ARCHIVE_DIR/"
```


```bash
Total Errors/Failures found: 116
--- Critical Events ---
5:2026-03-07 11:51:43 [CRITICAL]  - 32760
6:2026-03-07 11:51:43 [CRITICAL]  - 3764
7:2026-03-07 11:51:43 [CRITICAL]  - 31261
9:2026-03-07 11:51:43 [CRITICAL]  - 29507
12:2026-03-07 11:51:43 [CRITICAL]  - 30573
15:2026-03-07 11:51:43 [CRITICAL]  - 11068
16:2026-03-07 11:51:43 [CRITICAL]  - 14406
19:2026-03-07 11:51:43 [CRITICAL]  - 11558
22:2026-03-07 11:51:43 [CRITICAL]  - 28280
25:2026-03-07 11:51:43 [CRITICAL]  - 31191
28:2026-03-07 11:51:43 [CRITICAL]  - 31634
36:2026-03-07 11:51:43 [CRITICAL]  - 5561
41:2026-03-07 11:51:43 [CRITICAL]  - 398
45:2026-03-07 11:51:43 [CRITICAL]  - 27067
53:2026-03-07 11:51:43 [CRITICAL]  - 24796
57:2026-03-07 11:51:43 [CRITICAL]  - 28743
84:2026-03-07 11:51:43 [CRITICAL]  - 29343
87:2026-03-07 11:51:43 [CRITICAL]  - 23294
94:2026-03-07 11:51:43 [CRITICAL]  - 4156
95:2026-03-07 11:51:43 [CRITICAL]  - 1432
101:2026-03-07 11:51:43 [CRITICAL]  - 11538
104:2026-03-07 11:51:43 [CRITICAL]  - 6760
107:2026-03-07 11:51:43 [CRITICAL]  - 24591
110:2026-03-07 11:51:43 [CRITICAL]  - 4151
112:2026-03-07 11:51:43 [CRITICAL]  - 740
120:2026-03-07 11:51:43 [CRITICAL]  - 2345
122:2026-03-07 11:51:43 [CRITICAL]  - 30725
124:2026-03-07 11:51:43 [CRITICAL]  - 14274
130:2026-03-07 11:51:43 [CRITICAL]  - 9080
131:2026-03-07 11:51:43 [CRITICAL]  - 4701
136:2026-03-07 11:51:43 [CRITICAL]  - 31635
137:2026-03-07 11:51:43 [CRITICAL]  - 30513
143:2026-03-07 11:51:43 [CRITICAL]  - 29014
148:2026-03-07 11:51:43 [CRITICAL]  - 3149
149:2026-03-07 11:51:43 [CRITICAL]  - 5463
152:2026-03-07 11:51:43 [CRITICAL]  - 20343
154:2026-03-07 11:51:43 [CRITICAL]  - 8208
155:2026-03-07 11:51:43 [CRITICAL]  - 14543
156:2026-03-07 11:51:43 [CRITICAL]  - 30775
163:2026-03-07 11:51:43 [CRITICAL]  - 1331
166:2026-03-07 11:51:43 [CRITICAL]  - 30225
173:2026-03-07 11:51:43 [CRITICAL]  - 22762
175:2026-03-07 11:51:43 [CRITICAL]  - 22126
183:2026-03-07 11:51:43 [CRITICAL]  - 10948
190:2026-03-07 11:51:43 [CRITICAL]  - 22506
192:2026-03-07 11:51:43 [CRITICAL]  - 28027
194:2026-03-07 11:51:43 [CRITICAL]  - 30303
197:2026-03-07 11:51:43 [CRITICAL]  - 19946
203:2026-03-07 11:51:43 [CRITICAL]  - 13869
222:2026-03-07 11:51:43 [CRITICAL]  - 2612
232:2026-03-07 11:51:43 [CRITICAL]  - 20326
233:2026-03-07 11:51:43 [CRITICAL]  - 25787
237:2026-03-07 11:51:43 [CRITICAL]  - 18266
248:2026-03-07 11:51:43 [CRITICAL]  - 32462
256:2026-03-07 11:51:43 [CRITICAL]  - 9846
257:2026-03-07 11:51:43 [CRITICAL]  - 3791
261:2026-03-07 11:51:43 [CRITICAL]  - 25478
271:2026-03-07 11:51:43 [CRITICAL]  - 22804
273:2026-03-07 11:51:43 [CRITICAL]  - 7839
274:2026-03-07 11:51:43 [CRITICAL]  - 14137
278:2026-03-07 11:51:43 [CRITICAL]  - 23255
279:2026-03-07 11:51:43 [CRITICAL]  - 5590
283:2026-03-07 11:51:43 [CRITICAL]  - 7700
292:2026-03-07 11:51:43 [CRITICAL]  - 5352
297:2026-03-07 11:51:43 [CRITICAL]  - 26335
307:2026-03-07 11:51:43 [CRITICAL]  - 15986
315:2026-03-07 11:51:43 [CRITICAL]  - 25999
333:2026-03-07 11:51:43 [CRITICAL]  - 31588
335:2026-03-07 11:51:43 [CRITICAL]  - 14590
349:2026-03-07 11:51:43 [CRITICAL]  - 29545
354:2026-03-07 11:51:43 [CRITICAL]  - 12702
360:2026-03-07 11:51:43 [CRITICAL]  - 28137
362:2026-03-07 11:51:43 [CRITICAL]  - 5835
378:2026-03-07 11:51:43 [CRITICAL]  - 15252
383:2026-03-07 11:51:43 [CRITICAL]  - 28482
394:2026-03-07 11:51:43 [CRITICAL]  - 18683
398:2026-03-07 11:51:43 [CRITICAL]  - 21361
406:2026-03-07 11:51:43 [CRITICAL]  - 11865
416:2026-03-07 11:51:43 [CRITICAL]  - 8319
419:2026-03-07 11:51:43 [CRITICAL]  - 1160
420:2026-03-07 11:51:43 [CRITICAL]  - 21975
422:2026-03-07 11:51:43 [CRITICAL]  - 860
424:2026-03-07 11:51:43 [CRITICAL]  - 12920
434:2026-03-07 11:51:43 [CRITICAL]  - 32205
435:2026-03-07 11:51:43 [CRITICAL]  - 16258
443:2026-03-07 11:51:43 [CRITICAL]  - 27430
455:2026-03-07 11:51:43 [CRITICAL]  - 5905
462:2026-03-07 11:51:43 [CRITICAL]  - 9279
471:2026-03-07 11:51:43 [CRITICAL]  - 24528
477:2026-03-07 11:51:43 [CRITICAL]  - 4525
500:2026-03-07 11:51:44 [CRITICAL]  - 8164
Report generated: log_report_2026-03-07.txt
File 'sample_log.log' has been moved to archive/
```

I used grep to filter patterns, awk to strip timestamps, sort and uniq -c to count occurrences, and sed for final formatting.

### Key Learnings:
1. Combining multiple command-line tools can create powerful data processing pipelines.
2. Using associative arrays in bash can help manage complex data structures, but sometimes simple text processing with grep/awk is more efficient for log analysis.
3. Automating log analysis can save time and provide consistent insights, especially when dealing with large log files.