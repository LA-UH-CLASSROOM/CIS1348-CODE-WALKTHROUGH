# Code Walkthrough: Project 1 Solution

The full solution is 76 lines. It is read top to bottom as a sequence of clearly labeled steps.

## Step 1 — Get Collector Name
```python
spotifyfile = open("SPOTIFY_PRIVATE_PLAYLIST_BEST_OF_BEETLES_SONGS.txt", "r")
collectorname = spotifyfile.readline().strip()
spotifyfile.close()
```
Read one line, strip whitespace. Don't paste the name in — read it from the file so the code works for any dataset.

**Verify:** `print(collectorname)` — confirm you see the right name before continuing.

---

## Step 2 — Load All Member Names into a Lookup Table
```python
names = [""] * 100
for i in range(100):
    memberfile = open(f"members/member_ID{i}.txt", "r")
    names[i] = memberfile.readline().strip()
    memberfile.close()
```
Pre-allocate 100 slots. Assign each name at its corresponding index `i`. The index is the ID — this is what makes it a lookup table.

**Verify:** `print(i, names[i])` inside the loop. Spot-check a few random entries against the actual files.

---

## Step 3 — Find Collector ID
```python
collectorid = -1
for i in range(100):
    if names[i] == collectorname:
        collectorid = i
        break
```
Scan the lookup table for the collector's name. When found, record the index and stop. Starting with `-1` is intentional: if the name is never found, `collectorid` stays `-1`, which is an obviously wrong value that signals a problem.

**Verify:** `print(collectorid)` — confirm it matches the file you expect.

---

## Step 4 — Load and Parse All Transactions
```python
transactionsfile = open("transactions.txt", "r")
transactionlines = transactionsfile.readlines()
transactionsfile.close()

fromlist = []
tolist = []
amountlist = []
for line in transactionlines:
    line = line.strip()
    parts = line.split("|")
    fromid = int(parts[0].split(":")[1])
    toid = int(parts[1].split(":")[1])
    amount = int(parts[2].split(":")[1])
    fromlist.append(fromid)
    tolist.append(toid)
    amountlist.append(amount)

numberoftransactions = len(fromlist)
```
Three parallel lists: `fromlist[i]`, `tolist[i]`, `amountlist[i]` all refer to the same transaction at index `i`.

The format of each transaction line is `from:ID|to:ID|amount:N`. Split by `|` to get the three fields; split each field by `:` and take index `[1]` to get the value; convert to `int`.

**Verify:** Print a few parsed transactions and cross-check them visually against `transactions.txt`.

---

## Step 5 — Calculate Total Received from Collector
```python
total_amount_received_from_the_collector = [0] * 100

for i in range(numberoftransactions):
    if fromlist[i] == collectorid:
        total_amount_received_from_the_collector[tolist[i]] += amountlist[i]
```
For every transaction where the sender is the collector, add the amount to the receiver's running total.

**Verify:** Print non-zero entries. You should see a small number of people who received money from the collector.

---

## Step 6 — Find the Boss
```python
bossid = -1
maxreceived = 0
for i in range(100):
    if total_amount_received_from_the_collector[i] > maxreceived:
        maxreceived = total_amount_received_from_the_collector[i]
        bossid = i
```
Scan all totals. Keep track of the maximum seen so far and the person who achieved it. At the end, `bossid` holds the winner.

**Verify:** `print(bossid)` — then manually search the transactions file for transactions from the collector to this ID to sanity-check the total.

---

## Step 7 — Mark Criminals
```python
iscriminal = [False] * 100
iscriminal[bossid] = True
for i in range(numberoftransactions):
    if tolist[i] == bossid:
        iscriminal[fromlist[i]] = True
```
Everyone who sent money to the boss is a criminal. The boss is also marked — explicitly, because the boss did not send money to himself.

**Verify:** Print IDs and names of all criminals. Confirm the boss is in the list.

---

## Step 8 — Calculate Balances
```python
balance = [0] * 100
for i in range(numberoftransactions):
    balance[tolist[i]] += amountlist[i]
    balance[fromlist[i]] -= amountlist[i]
```
For every transaction: add to the receiver's balance, subtract from the sender's balance. This is done for everyone — no need to filter by criminal status here.

---

## Step 9 — Write the Report
```python
report = open("analysis.txt", "w")
report.write("id\tname\tbalance\n")
for i in range(100):
    if iscriminal[i] == True:
        report.write(f"{i}\t{names[i]}\t{balance[i]}\n")
report.close()
```
Loop from 0 to 99. Print only criminals. Because we loop in index order, the output is naturally sorted by ID — no sort function needed.

---

*© 2026 Levent Albayrak. Distributed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).*
