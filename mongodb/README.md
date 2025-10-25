# MongoDB: From Relational Rows to Flexible Documents

This is my field guide to MongoDB — a NoSQL database that trades rigid tables for flexible JSON-like documents. It's part of my Curiosity Documentation series: short notes I write to learn tools from the ground up.

MongoDB's documentation is genuinely excellent at making complex database concepts feel approachable. These are my notes from exploring it.

---

## 🧩 What Is MongoDB?

MongoDB is a NoSQL, document-oriented database that stores data in flexible JSON-like documents instead of rigid tables. It's designed for scalability, high availability, and developer-friendly iteration.

### The Filing Cabinet Metaphor

**SQL databases** are like filing cabinets with rigid folder structures:
- Every folder must have the same fields (columns)
- Adding a new field means restructuring every folder
- Related data lives in separate cabinets (tables) connected by ID numbers

**MongoDB** is like storage bins with labels:
- Each bin (document) can have different contents
- You can add new fields to any document without affecting others
- Related data can live together in the same bin (embedded documents)
- You organize bins into labeled shelves (collections)

**Example:**

SQL might store a user like this across multiple tables:
```sql
-- users table
| id | name | email |
| 1  | Alex | alex@example.com |

-- addresses table
| user_id | street | city |
| 1       | 123 Main | Portland |
```

MongoDB stores it as one document:
```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Alex",
  email: "alex@example.com",
  address: {
    street: "123 Main",
    city: "Portland"
  }
}
```

---

## 🚀 Getting Started

### Installation (local)
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

### Connect via CLI
```bash
mongosh
```

### Basic Commands
```bash
show dbs                  # List databases
use myDatabase            # Switch or create a database
show collections          # List collections
```

---

## 🗂️ CRUD Operations

### Create
```javascript
db.users.insertOne({ name: "Alex", role: "Engineer", active: true })
```

### Read
```javascript
db.users.find({ active: true })
```

### Update
```javascript
db.users.updateOne({ name: "Alex" }, { $set: { role: "Tech Lead" } })
```

### Delete
```javascript
db.users.deleteOne({ name: "Alex" })
```

---

## 🔍 Query Operators

Query operators let you filter documents with conditions beyond simple equality.

### Comparison Operators
```javascript
// Find users older than 25
db.users.find({ age: { $gt: 25 } })

// Find users between 18 and 65
db.users.find({ age: { $gte: 18, $lte: 65 } })

// Find users in specific cities
db.users.find({ city: { $in: ["Portland", "Seattle", "Austin"] } })

// Find users NOT in certain roles
db.users.find({ role: { $ne: "admin" } })
```

### Logical Operators
```javascript
// Find active engineers
db.users.find({
  $and: [
    { role: "engineer" },
    { active: true }
  ]
})

// Find users who are either admins OR engineers
db.users.find({
  $or: [
    { role: "admin" },
    { role: "engineer" }
  ]
})
```

### Element Operators
```javascript
// Find documents where 'phone' field exists
db.users.find({ phone: { $exists: true } })

// Find documents where 'tags' is an array
db.users.find({ tags: { $type: "array" } })
```

### String Pattern Matching
```javascript
// Find users with email ending in @gmail.com
db.users.find({ email: { $regex: /@gmail\.com$/ } })

// Case-insensitive search for "alex"
db.users.find({ name: { $regex: /alex/i } })
```

---

## ⚙️ Indexes

Indexes make queries faster by creating a sorted reference structure — like an index at the back of a textbook that lets you jump directly to a topic instead of scanning every page.

**Without an index:** MongoDB scans every document in a collection (slow for large datasets)
**With an index:** MongoDB looks up the value in the index and jumps directly to matching documents

### Creating Indexes
```javascript
// Single field index (1 = ascending, -1 = descending)
db.users.createIndex({ email: 1 })

// Compound index (multiple fields)
db.users.createIndex({ lastName: 1, firstName: 1 })

// Unique index (enforces uniqueness)
db.users.createIndex({ email: 1 }, { unique: true })

// Text index for full-text search
db.articles.createIndex({ content: "text" })
```

### Viewing and Managing Indexes
```javascript
// See all indexes on a collection
db.users.getIndexes()

// Drop an index
db.users.dropIndex("email_1")

// Analyze query performance
db.users.find({ email: "alex@example.com" }).explain("executionStats")
```

**When to add indexes:**
- Fields you frequently query or sort by
- Fields used in joins/lookups
- Unique identifiers beyond `_id`

**Trade-off:** Indexes speed up reads but slow down writes (inserts/updates), so don't over-index.

---

## 📦 Aggregation Pipelines

Aggregation pipelines transform data through a series of stages — like an assembly line where each stage processes documents and passes them to the next stage.

### Common Pipeline Stages

**$match** — Filter documents (like `find()`)
```javascript
{ $match: { status: "completed" } }
```

**$group** — Group documents and calculate aggregates
```javascript
{ $group: { _id: "$customerId", totalSpent: { $sum: "$amount" } } }
```

**$project** — Reshape documents (include/exclude fields, compute new fields)
```javascript
{ $project: { name: 1, email: 1, fullAddress: { $concat: ["$street", ", ", "$city"] } } }
```

**$sort** — Order documents
```javascript
{ $sort: { createdAt: -1 } }  // newest first
```

**$limit** and **$skip** — Pagination
```javascript
{ $skip: 20 },
{ $limit: 10 }  // skip first 20, return next 10
```

**$lookup** — Join with another collection (like SQL JOIN)
```javascript
{
  $lookup: {
    from: "orders",
    localField: "_id",
    foreignField: "customerId",
    as: "customerOrders"
  }
}
```

### Real-World Example: Customer Analytics

Calculate total revenue per customer, sorted by spend:

```javascript
db.orders.aggregate([
  // Stage 1: Only include completed orders
  { $match: { status: "completed" } },

  // Stage 2: Group by customer and sum amounts
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      orderCount: { $sum: 1 }
    }
  },

  // Stage 3: Sort by total spent (highest first)
  { $sort: { totalSpent: -1 } },

  // Stage 4: Limit to top 10 customers
  { $limit: 10 },

  // Stage 5: Look up customer details
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "customer"
    }
  },

  // Stage 6: Reshape the output
  {
    $project: {
      customerName: { $arrayElemAt: ["$customer.name", 0] },
      totalSpent: 1,
      orderCount: 1
    }
  }
])
```

**Output:**
```javascript
[
  { _id: ObjectId("..."), customerName: "Alex", totalSpent: 5420, orderCount: 12 },
  { _id: ObjectId("..."), customerName: "Jordan", totalSpent: 4890, orderCount: 8 },
  // ...
]
```

---

## 📐 Schema Design: Embed vs Reference

One of the biggest questions in MongoDB: should you embed related data or reference it?

### Embed When:
- Data is always accessed together
- One-to-few relationships (e.g., user has 1-5 addresses)
- Child data doesn't change often
- You want atomic updates

**Example: Blog post with comments**
```javascript
{
  _id: ObjectId("..."),
  title: "My First Post",
  content: "...",
  comments: [
    { author: "Alex", text: "Great post!", createdAt: ISODate("2024-01-15") },
    { author: "Jordan", text: "Thanks for sharing", createdAt: ISODate("2024-01-16") }
  ]
}
```

### Reference When:
- Data is accessed independently
- One-to-many or many-to-many relationships
- Child data grows unbounded
- You need to avoid document size limits (16MB)

**Example: Blog post referencing author**
```javascript
// posts collection
{
  _id: ObjectId("..."),
  title: "My First Post",
  authorId: ObjectId("507f1f77bcf86cd799439011")
}

// users collection
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Alex",
  email: "alex@example.com"
}
```

**Rule of thumb:** Embed what you read together. Reference what you query separately.

---

## 🔗 Working with Node.js + TypeScript

MongoDB pairs beautifully with Node.js. Here's how to use it with TypeScript for type safety.

### Installation
```bash
npm install mongodb
npm install --save-dev @types/mongodb
```

### Connection Setup
```typescript
import { MongoClient, Db } from 'mongodb';

const uri = "mongodb://localhost:27017";
const client = new MongoClient(uri);

async function connectToDatabase(): Promise<Db> {
  await client.connect();
  console.log("Connected to MongoDB");
  return client.db("myDatabase");
}
```

### Typed Collections
```typescript
interface User {
  _id?: ObjectId;
  name: string;
  email: string;
  role: "engineer" | "designer" | "manager";
  active: boolean;
  createdAt: Date;
}

const db = await connectToDatabase();
const users = db.collection<User>("users");

// TypeScript knows the shape of documents!
const user = await users.findOne({ email: "alex@example.com" });
console.log(user?.name); // Type-safe access
```

### CRUD with Types
```typescript
// Insert
const result = await users.insertOne({
  name: "Alex",
  email: "alex@example.com",
  role: "engineer",
  active: true,
  createdAt: new Date()
});

// Find with filter
const engineers = await users
  .find({ role: "engineer", active: true })
  .toArray();

// Update
await users.updateOne(
  { email: "alex@example.com" },
  { $set: { role: "manager" } }
);

// Delete
await users.deleteOne({ email: "alex@example.com" });
```

---

## 🎯 Common Patterns & Gotchas

### Pattern: Pagination
```javascript
// Skip/Limit (simple but slow for deep pages)
db.users.find().skip(20).limit(10);

// Range-based (faster, but requires sorting by indexed field)
db.users.find({ _id: { $gt: lastSeenId } }).limit(10);
```

### Pattern: Upsert (Update or Insert)
```javascript
// Create if doesn't exist, update if it does
db.users.updateOne(
  { email: "alex@example.com" },
  { $set: { name: "Alex", lastLogin: new Date() } },
  { upsert: true }
);
```

### Gotcha: Array Updates
```javascript
// Wrong: This replaces entire array
db.users.updateOne(
  { _id: userId },
  { $set: { tags: ["new-tag"] } }
);

// Right: This adds to array
db.users.updateOne(
  { _id: userId },
  { $push: { tags: "new-tag" } }
);

// Add only if not already present
db.users.updateOne(
  { _id: userId },
  { $addToSet: { tags: "new-tag" } }
);
```

### Gotcha: The `_id` Field
```javascript
// MongoDB auto-generates _id if you don't provide one
const doc = { name: "Alex" };
await db.users.insertOne(doc);
console.log(doc._id); // Now populated!

// But be careful with manual IDs
await db.users.insertOne({ _id: "custom-id", name: "Alex" });
// This will fail if _id already exists
```

---

## ☁️ MongoDB Atlas

MongoDB Atlas is the managed cloud version of MongoDB — it handles scaling, backups, and monitoring automatically.

Connect to an Atlas cluster using your connection string:
```bash
mongosh "mongodb+srv://<cluster-url>/myDatabase" --apiVersion 1
```

---

## 🧠 Key Takeaways

**What I've learned from MongoDB's docs:**

1. **BSON vs JSON** — MongoDB stores data as BSON (binary JSON), which supports richer types like `Date`, `ObjectId`, and `Binary`.

2. **Schema flexibility is a feature, not a bug** — But that doesn't mean "no schema." Define models early for consistency.

3. **Aggregation pipelines are powerful** — They can replace many SQL JOINs and complex queries, but they take practice to read and write.

4. **Index intentionally** — Every index speeds up reads but slows down writes. Only index fields you actually query.

5. **Embed for performance, reference for flexibility** — Choose based on how you read the data, not just how it's related.

---

**Goal:** Keep exploring how MongoDB's documentation communicates these concepts clearly. The docs are a great example of developer education that feels empowering, not intimidating.

---

