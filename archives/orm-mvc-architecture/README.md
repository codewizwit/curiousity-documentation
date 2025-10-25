# ORM & MVC: When Objects Met Databases — 2020

_The moment I understood how code talks to databases_

---

## Context

**When:** Early 2020 (Bootcamp Module 13 - Object Relational Mapping)
**What I was learning:** ORMs (Sequelize), MVC architecture, how JavaScript objects map to database tables
**Why it was hard:** I understood databases. I understood JavaScript. But how do you get them to talk to each other? How does `user.save()` become `INSERT INTO users...`?
**Where I was:** Building my first full-stack apps with databases

**The challenge:** Understanding the "magic" layer between object-oriented code and relational databases.

---

## The Notes

These 7 pages capture me learning how ORMs bridge the gap between code and databases, and how MVC architecture organizes everything into a coherent system.

### Page 1: Module 13 - ORM Introduction
![Module 13 ORM Intro](./01-module-13-orm-intro.jpeg)

**What's happening here:**
"MODULE 13 - OBJECT RELATIONAL MAPPING"

Notes on what an ORM is, why it exists, and the problems it solves.

**What I notice now:**
The title says it all: Object-Relational Mapping. Taking objects (JavaScript classes/instances) and mapping them to relational data (SQL tables/rows). The bridge between two worlds.

---

### Page 2: ORM Concepts Continued
![ORM Continued](./02-orm-continued.jpeg)

**What's happening here:**
More detailed notes on ORM concepts, relationships, and how they work.

**What I notice now:**
I was building my mental model step by step - not just "what is an ORM" but "how does it actually work under the hood?"

---

### Page 3: Database Setup & APIs
![Database Setup](./03-database-setup-apis.jpeg)

**What's happening here:**
Notes on setting up databases, API routes, connecting everything together.

Steps and diagrams showing the flow from HTTP request → route → database.

**What I notice now:**
I needed to see the full pipeline: request comes in, hits a route, calls the database through the ORM, sends back a response. Drawing the flow made it concrete.

---

### Page 4: Sequelize Models
![Sequelize Models](./04-sequelize-models.jpeg)

**What's happening here:**
Defining models in Sequelize - how to map JavaScript classes to database tables.

Notes on data types, relationships (foreign keys), and model definitions.

**What I notice now:**
The "aha" was realizing models are like blueprints. You define the structure once in code, and the ORM handles creating the table, validating data, and mapping everything.

---

### Page 5: Models & Database Operations
![Models and Operations](./05-models-operations.jpeg)

**What's happening here:**
How to perform operations using models: create, read, update, delete through the ORM instead of raw SQL.

**What I notice now:**
This is where the magic clicked: `User.create()` instead of writing `INSERT INTO users...`. The ORM translates method calls into SQL behind the scenes.

---

### Page 6: MVC - Class, Instance → Data
![MVC Class Instance Diagram](./06-mvc-class-instance-diagram.jpeg)

**What's happening here:**
Diagram showing MVC flow: "CLASS, INSTANCE → DATA (BACKEND)"

Visual representation of how classes become instances which become database records.

**What I notice now:**
The boxes and arrows again! I needed to visualize the transformation:
- **Class** (User definition) → **Instance** (new User) → **Data** (row in database)

---

### Page 7: MVC Architecture Diagram
![MVC Architecture](./07-mvc-architecture-diagram.jpeg)

**What's happening here:**
"MODEL VIEW CONTROLLER" diagram with boxes showing how each piece connects.

Mapping out the full MVC pattern and how it organizes a web application.

**What I notice now:**
MVC is about separation of concerns:
- **Model** = data and business logic (ORM lives here)
- **View** = presentation (what users see)
- **Controller** = coordination (connects model and view)

Drawing it as boxes helped me understand where each piece of code belongs.

---

## The Aha Moment

**Page 4 — Sequelize Models**

The breakthrough: **Models are blueprints that map 1:1 to database tables.**

You define the structure once in JavaScript:
```javascript
class User {
  name: string
  email: string
}
```

The ORM creates the table:
```sql
CREATE TABLE users (
  name VARCHAR(255),
  email VARCHAR(255)
);
```

And then you work with objects:
```javascript
const user = await User.create({ name: 'Alex', email: 'alex@example.com' });
```

Instead of SQL:
```sql
INSERT INTO users (name, email) VALUES ('Alex', 'alex@example.com');
```

**The ORM is a translator.** You speak JavaScript, it speaks SQL.

---

## How I'd Explain This Now

**Then (2020):**
"An ORM lets you use JavaScript to talk to databases instead of writing SQL."

**Now (2025):**
"Think of an ORM as a bilingual translator between your code and your database. You speak in objects and method calls (`user.save()`), the ORM translates that into SQL (`UPDATE users SET...`), then translates the database response back into objects you can use in your code.

ORMs like Sequelize give you type safety, validation, and relationship management without writing raw SQL. You define models (blueprints), the ORM creates tables, and then you work with JavaScript objects that automatically sync with the database.

The tradeoff: you gain developer experience and safety, but lose some performance and control compared to hand-written SQL. Use ORMs for most CRUD operations, drop down to raw SQL for complex queries."

**The translator metaphor came from these notes** — trying to understand the bridge between code and database.

---

## What This Taught Me

**Abstraction is powerful, but you need to understand what's underneath.**

ORMs feel like magic when they work. But when they break (or when you need to optimize), you need to know the SQL they're generating.

I learned to:
1. **Use the abstraction** (write cleaner code with models)
2. **Understand the translation** (know what SQL is being generated)
3. **Know when to drop down** (use raw SQL for complex joins or performance)

This pattern shows up everywhere in engineering: abstractions are tools, not replacements for understanding.

**Visual thinking reveals structure.**

The MVC diagrams showed me that architecture isn't abstract — it's about organizing code into logical boxes:
- Models = data layer
- Views = presentation layer
- Controllers = coordination layer

When code has a clear home, systems are easier to understand and maintain.

---

## Connection to Today

The ORM concepts from these notes connect to:
- How I structure backend applications (clear separation of concerns)
- When I use ORMs vs raw SQL (developer experience vs performance)
- How I explain system architecture (boxes and arrows, data flow)

The instinct to visualize data flow and understand the layers started here.

---

_From early 2020 bootcamp notes - Module 13. Part of [The Archives](../README.md)._
