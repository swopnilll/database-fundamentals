## **Indexes on Different Data Types**

### a. **Numeric Fields**

- Indexes on numeric fields (e.g., `age`) are common and useful.
- Queries filtering on numeric fields can be efficiently sped up by indexes.

### b. **Text Fields**

- You can create indexes on string fields (e.g., `gender`, `name`).
- Since text fields are sortable, indexing them speeds up text-based queries.
- However, if the field only has a small set of distinct values (low cardinality), the index may not provide a significant performance boost.

### c. **Boolean Fields**

- Indexing boolean fields (`true` or `false`) is generally not beneficial.
- Because boolean fields have only two possible values, queries using these indexes tend to return a large portion of the collection.
- The index scan cost might outweigh the benefit compared to a collection scan.

---

## 3\. **Single-field Index Example**

- Creating an index on `gender` ascending:

  `db.contacts.createIndex({ gender: 1 });`

- Query filtering on `gender: "male"` will use this index.
- But if the data is evenly distributed (e.g., half male, half female), the index might not speed up queries much since many documents are returned.
- MongoDB may not consider a collection scan as an alternative in query plan candidates until you explicitly check plans.
