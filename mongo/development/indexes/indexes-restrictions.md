## Explanation of the Concept

Indexes in databases like MongoDB are designed to speed up queries by allowing the database to quickly locate a small subset of documents matching specific criteria without scanning the entire collection.

- When a query targets a **small fraction** of the dataset (e.g., "people older than 60"), the index efficiently narrows down to those few documents, making the query very fast.
- However, if the query matches a **large fraction** of the dataset (e.g., "people older than 20" when almost all people are older than 20), the index becomes less useful. The database ends up scanning almost the entire index and then fetching almost all documents, which adds overhead.
- In such cases, running the query **without an index** (a full collection scan) can be faster, because the database reads all documents directly, skipping the extra step of using the index.
- Therefore, **indexes improve query performance when queries return a small subset of documents**. But if queries return the majority or all documents, indexes might actually slow down the query.

---

## 📘 Learning Note: When to Use Indexes in MongoDB

- **Indexes speed up queries that return a small portion of documents** (e.g., < 20-30% of the collection).
- For queries that return **most or all documents**, indexes may introduce extra overhead and slow down performance.
- A **full collection scan** can be more efficient when the majority of documents match the query.
- Always analyze your query patterns and data distribution before creating indexes.
- Use indexes to target **narrow, selective queries** rather than broad queries.

| Command                                    | Purpose                                        |
| ------------------------------------------ | ---------------------------------------------- |
| `db.collection.createIndex({ age: 1 })`    | Creates an ascending index on the `age` field  |
| `db.collection.dropIndex({ age: 1 })`      | Drops the index on the `age` field             |
| `db.collection.find({ age: { $gt: 60 } })` | Finds documents where `age` is greater than 60 |
| `db.collection.find({ age: { $gt: 20 } })` | Finds documents where `age` is greater than 20 |
