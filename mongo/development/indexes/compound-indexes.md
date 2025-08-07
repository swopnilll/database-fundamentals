## **Compound Indexes**

### a. **What is a Compound Index?**

- A compound index contains more than one field in a single index structure.
- Example: Index on `{ dob.age: 1, gender: 1 }` creates a combined index of pairs (age, gender).
- The order of fields in the compound index **matters** — the index sorts first by the first field, then by the second, and so on.

### b. **How Compound Index Works**

- The index entries are tuples of values: `(age, gender)`, `(31, "male")`, `(31, "female")`, `(32, "male")`, etc.
- The index is sorted first by `age`, then by `gender` within each age.

### c. **Query Performance with Compound Index**

- Queries filtering by both fields (e.g., `age: 35` and `gender: "male"`) can **fully utilize** the compound index.
- Queries filtering only by the **leftmost field(s)** of the index (here, `age`) can **partially use** the compound index.
- Queries filtering only by the **rightmost field(s)** (here, `gender`) **do not** use the compound index and fall back to a collection scan.
- MongoDB can utilize the index for prefixes of the compound key (from left to right), but not for fields that are not the first.

### d. **Example commands**

- Create compound index:

  `db.contacts.createIndex({ "dob.age": 1, gender: 1 });`

- Query using both fields:

  `db.contacts.find({ "dob.age": 35, gender: "male" }).explain("executionStats");`

- Query using only leftmost field:

  `db.contacts.find({ "dob.age": 35 }).explain("executionStats");`

- Query using only rightmost field (gender only) will not use this index:

  `db.contacts.find({ gender: "male" }).explain("executionStats");`

---

## 5\. **Key Points About Compound Indexes**

- Compound indexes store combined field values as **ordered tuples**.
- Only the **leftmost prefix** of the compound index can be used for query filtering.
- You can create compound indexes with multiple fields (3, 4, or more).
- Queries can benefit from:
  - Using the first field alone
  - The first two fields together
  - All fields together
- But not fields that are not at the start (e.g., 3rd field alone in a 4-field compound index).

---

## 6\. **Why Compound Indexes Matter**

- Compound indexes allow MongoDB to speed up queries filtering on **multiple criteria**.
- More selective queries can benefit the most.
- They reduce the need for multiple single-field indexes and improve query efficiency.

---

## 7\. **Summary Table**

| Index Type             | Best Use Case                         | Speed Up Query Filtering?       | Notes                                   |
| ---------------------- | ------------------------------------- | ------------------------------- | --------------------------------------- |
| Single-field (numeric) | Numeric fields like `age`             | Yes                             | Very effective                          |
| Single-field (text)    | Text fields with many distinct values | Yes, but depends on cardinality | Useful if values are selective          |
| Single-field (boolean) | Boolean fields (`true`/`false`)       | Usually No                      | Low cardinality limits usefulness       |
| Compound Index         | Multiple fields in filter             | Yes, but only leftmost prefix   | Order of fields matters for query usage |
