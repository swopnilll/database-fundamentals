# 📘 MongoDB Compound Index Explained with Real-World Example

## Context: Real Database Scenario — A Social Media App's User Profiles

Imagine a **social media app** where users have profiles stored in a MongoDB collection called `users`. Each user document looks like this:

```json
{   "_id": ObjectId("..."),   "username": "johndoe",   "age": 29,   "gender": "male",   "location": "New York",   "interests": ["sports", "music", "coding"],   "isActive": true }
```

## Use Case

You want to quickly find users who are:

- **Active users** (`isActive: true`)
- Of a specific **gender** (e.g., `gender: "female"`)
- Within a certain **age range** (e.g., 25 to 35)

This is a common filtering scenario — to find a targeted audience segment.

## Why MongoDB and Compound Index?

- MongoDB allows **fast queries on nested and flexible schemas**.
- Data access patterns often filter on **multiple fields** combined, making **compound indexes** ideal.
- SQL can do the same with composite indexes, but MongoDB offers flexible schema and indexing with easier scaling horizontally.

## Step 1: Create Compound Index on the fields you query together

You create a compound index on `isActive`, `gender`, and `age` fields like this:

```js
db.users.createIndex({ isActive: 1, gender: 1, age: 1 });
```

- Order is important:
  `isActive` first because it filters active users,
  `gender` second, and
  `age` third.

## Step 2: Run a query filtering on all these fields

Example query:

`db.users.find({   isActive: true,   gender: "female",   age: { $gte: 25, $lte: 35 } })`

## Step 3: How MongoDB uses the compound index

- The index entries look like this (simplified):
  `(true, "female", 25)`, `(true, "female", 26)`, `(true, "male", 25)`, `(false, "female", 30)`, etc.

- MongoDB uses this index to quickly jump to the `isActive: true` entries, then narrow down to `gender: female`, then filter by age range efficiently.
- Without the compound index, MongoDB might scan the whole collection or multiple indexes, which is slower.

## Step 4: What if you query only some fields?

- Query only on `isActive` and `gender`:
  This query **can still use** the compound index since `isActive` and `gender` are the **leftmost prefix** of the index.

- Query only on `gender` and `age`:
  MongoDB **cannot use** this compound index efficiently because the first field in the index (`isActive`) is not specified in the query — the index is ordered by `isActive` first.

- Query only on `age`:
  Also no efficient use of this index because age is the **third** field.

## Why MongoDB over SQL here?

- MongoDB supports **dynamic, nested, and schema-flexible documents**; adding or removing fields is easier.
- Queries on nested arrays (`interests`), which are common in social apps, are simpler in MongoDB.
- Horizontal scaling (sharding) and flexible indexing in MongoDB handle huge, unstructured data well.
- Compound indexes work similarly to SQL composite indexes but adapt well to flexible document structures.

## Example Explain Plan (MongoDB shell)

`db.users.find({   isActive: true,   gender: "female",   age: { $gte: 25, $lte: 35 } }).explain("executionStats")`

- You will see `"stage": "IXSCAN"` showing the compound index is used.
- `"indexName"` will be `"isActive_1_gender_1_age_1"` (auto-generated).
- `"totalKeysExamined"` will be much smaller than `"totalDocsExamined"` in a collection scan.
- Query is **efficient** because the compound index narrows down results fast.

# Summary

| Scenario                            | Compound Index Use?            | Effectiveness                     |
| ----------------------------------- | ------------------------------ | --------------------------------- |
| Filter by all three fields          | ✔ Fully used                   | Fastest query time                |
| Filter by `isActive`, `gender` only | ✔ Partially used (left prefix) | Still uses index efficiently      |
| Filter by `gender`, `age` only      | ✘ Not used                     | Collection scan or less efficient |
| Filter by `age` only                | ✘ Not used                     | Collection scan or less efficient |

# Visualizing a Compound Index Table Example

### Context: Compound index on `{ isActive: 1, gender: 1, age: 1 }`

| Index Entry # | isActive | gender | age | Pointer to Document (simplified) |
| ------------- | -------- | ------ | --- | -------------------------------- |
| 1             | true     | female | 25  | → Document A                     |
| 2             | true     | female | 28  | → Document B                     |
| 3             | true     | female | 30  | → Document C                     |
| 4             | true     | male   | 22  | → Document D                     |
| 5             | true     | male   | 35  | → Document E                     |
| 6             | false    | female | 27  | → Document F                     |
| 7             | false    | male   | 31  | → Document G                     |

---

### Key points about this table:

- **Sorted by `isActive` first**, then by `gender`, then by `age`:
  - All `true` entries come before `false` entries.
  - Within `true`, `female` comes before `male` (alphabetically ascending).
  - Within each `(isActive, gender)` group, sorted by age ascending.
- Each row stores a **tuple** of these three values plus a pointer to the actual document in the collection.
- When MongoDB searches with a query like:
  `{ isActive: true, gender: "female", age: { $gte: 26 } }`
  It quickly finds the first row matching `(true, female, 26)` or next bigger, then scans sequentially forward through the index entries, stopping when conditions fail.

---

### Why this structure speeds up queries:

- Instead of scanning **all documents**, it scans **only the matching tuples** in this index.
- Because it's sorted, queries using left-most fields (`isActive` and `gender`) can jump directly to the right "block" of the index.
- Queries specifying only `isActive` can scan all entries with `isActive: true` efficiently.
- Queries on just `gender` or `age` can't directly jump in because the index is sorted by `isActive` first.
