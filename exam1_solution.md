# Code Walkthrough: Exam 1 Solution

The task: parse an Apache web server log, find all 404 errors, count how often each referrer URL caused them, and output the results sorted by frequency.

The full solution is 32 lines.

## Step 1 — Accept Arguments
```python
import sys

inputfilename = sys.argv[1]
outputfilename = sys.argv[2]
```
The script is called as `python solution.py apache_logs.txt result.txt`. Never hard-code filenames — the grader runs the script with different filenames.

---

## Step 2 — Read the Log
```python
lines = open(inputfilename, "r").readlines()
```
One line of code. All log lines are now in a list.

---

## Step 3 — Parse Each Line
```python
uniquereferrers = {}

for line in lines:
    parts = line.strip().split('"')
    returncode = parts[2].strip().split()[0]
    referrer = parts[3]
```
Apache log lines look like this:
```
1.2.3.4 - - [01/Jan/2024] "GET /page HTTP/1.1" 200 1234 "https://referrer.com" "Mozilla/..."
```
Splitting on `"` (the quote character) separates the variable-length string fields cleanly:
- `parts[1]` = the request (`GET /page HTTP/1.1`)
- `parts[2]` = ` 200 1234 ` (status code and byte count, space-separated)
- `parts[3]` = the referrer URL
- `parts[5]` = the user agent string

Take `parts[2]`, strip it, split on space, take index `[0]` to get just the status code (`"200"`, `"404"`, etc.).

**Why not split on spaces directly?** The request, date, and user-agent fields contain spaces. Space-splitting would break them into many fragments. Quotes are a safe boundary.

---

## Step 4 — Filter
```python
    if returncode != "404":
        continue
    if referrer == "-":
        continue
```
Skip lines that are not 404 errors. Skip entries with no referrer (a `-` means the request came directly, not from another site).

---

## Step 5 — Count Referrers
```python
    if referrer in uniquereferrers:
        uniquereferrers[referrer] += 1
    else:
        uniquereferrers[referrer] = 1
```
Dictionary accumulates a count per referrer URL.

---

## Step 6 — Build Sortable List
```python
result = []
for referrer in uniquereferrers:
    r = (uniquereferrers[referrer], referrer)
    result.append(r)

result.sort(reverse=True)
```
Each element is `(count, referrer)`. Sorting puts the highest count first. Ties are broken by the second element, so referrers with the same count come out in reverse alphabetical order.

---

## Step 7 — Write Output
```python
f = open(outputfilename, "w")
f.write("referrer\tcount\n")
for r in result:
    f.write(f"{r[1]}\t{r[0]}\n")
f.close()
```
Output: header row, then one line per referrer in descending frequency order.

---
*© 2026 Levent Albayrak. Distributed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).*
