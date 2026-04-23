<h1 align="center">Prisma Notes</h1>

- [Setup and Installation:](#setup-and-installation)
- [Introduction:](#introduction)
  - [What is Prisma:](#what-is-prisma)
  - [What is ORM and ODM:](#what-is-orm-and-odm)
  - [Difference between Prisma and Mongoose:](#difference-between-prisma-and-mongoose)
- [Schema:](#schema)
  - [Common Data Types:](#common-data-types)
  - [Common Column Constraints:](#common-column-constraints)
- [CRUD Operations:](#crud-operations)
  - [Create(POST):](#createpost)
    - [create():](#create)
    - [createMany():](#createmany)
    - [createManyAndReturn():](#createmanyandreturn)
    - [Create Options:](#create-options)
      - [Select:](#select)
      - [include:](#include)
      - [skipDuplicates (Only for createMany):](#skipduplicates-only-for-createmany)
  - [READ(GET):](#readget)
    - [findMany():](#findmany)
    - [findUnique():](#findunique)
    - [findUniqueOrThrow():](#finduniqueorthrow)
    - [findFirst():](#findfirst)
    - [findFirstOrThrow():](#findfirstorthrow)
    - [count():](#count)
    - [Common read options:](#common-read-options)
      - [WHERE:](#where)
      - [orderBy:](#orderby)
      - [take and skip:](#take-and-skip)
      - [DISTINCT:](#distinct)
    - [aggregate():](#aggregate)

# Setup and Installation: 
- Step 1: Install dependencies

```bash
npm init -y
npm i express pg dotenv @prisma/client @prisma/adapter-pg
npm i -D typescript tsx prisma @types/node @types/express @types/pg 
npx tsc --init
```

here, 
  - prisma - The Prisma CLI for running commands like prisma init, prisma migrate, and prisma generate
  - @prisma/client - The Prisma Client library for querying your database
  - @prisma/adapter-pg - The node-postgres driver adapter that connects Prisma Client to your database


- Step 2: Configure ESM support: 

```json
// tsconfig.json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2023",
    "strict": true,
    "esModuleInterop": true,
    "ignoreDeprecations": "6.0"
  }
}
```

```json
// package.json
{
  "type": "module"
}
"scripts": {
    "dev": "tsx watch ./index.ts",
    "build": "tsc",
    "start": "node ./dist/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
},
```

- Step 3:  Initialize Prisma ORM: 

```bash
npx prisma init --datasource-provider postgresql --output ../generated/prisma
```

This command does a few things:
  - Creates a prisma/ directory with a schema.prisma file containing your database connection and schema models
  - Creates a .env file in the root directory for environment variables
  - Creates a prisma.config.ts file for Prisma configuration 

- Step 4: update env file with database connection string: 

```js
DATABASE_URL="postgresql://username:password@localhost:5432/mydb?schema=public"
```

- Step 5: Define data model: 

Open prisma/schema.prisma and add the following models:

```js
// prisma/schema.prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User { 
  id    Int     @id @default(autoincrement()) 
  name  String?
  email String  @unique
} 
```

- step 6: Create and apply your first migration: 

Create and apply your first migration

```bash
npx prisma migrate dev --name init
```

This command creates the database tables based on your schema. Now run the following command to generate the Prisma Client:

```bash
npx prisma generate
```

- Step 7: Instantiate Prisma Client: 

Now that you have all the dependencies installed, you can instantiate Prisma Client. You need to pass an instance of the Prisma ORM driver adapter adapter to the PrismaClient constructor:

```js
// lib/prisma.ts
import "dotenv/config";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

const connectionString = `${process.env.DATABASE_URL}`;

const adapter = new PrismaPg({ connectionString });
const prisma = new PrismaClient({ adapter });

export { prisma };
```

- step 8: use this boilerplate code to test setup: 

```ts
import express, { Request, Response } from "express";
import dotenv from "dotenv";
import path from "path";
import { prisma } from "./lib/prisma";

dotenv.config({ path: path.join(process.cwd(), ".env") });

const app = express();
app.use(express.json());

const port = process.env.PORT || 3000;


app.get("/", (req: Request, res: Response) => {
    res.send("Prisma + PostgreSQL + TypeScript API is running!");
});



/*
add all crud routes here
*/



app.use((req: Request, res: Response) => {
    res.status(404).send({
        error: "Route not found",
        path: req.path,
    });
});

app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```

step 9: We can explore our data with Prisma Studio

```bash
npx prisma studio
```

# Introduction: 
## What is Prisma:
Prisma is a Node.js and TypeScript ORM (Object-Relational Mapping) that provides type-safe database access, migrations, and a visual data editor. It's consists of three main components:
- Prisma Client: For type-safe database queries
- Prisma Migrate: For managing database schema changes
- Prisma Studio: A GUI for viewing and editing data in our database

## What is ORM and ODM:
ORM is a technique used to interact with relational databases (like PostgreSQL, MySQL) using programming language objects instead of writing raw SQL queries.

It maps:
- Tables → Models
- Rows → Objects
- Columns → Fields


ODM is a technique used to interact with NoSQL/document-based databases (such as MongoDB) using programming language objects instead of writing raw NOSQL queries.

It maps:

- Collections → Models
- Documents → Objects (JSON-like)
- Fields → Properties

Note:  Prisma also supports NoSQL databases like MongoDB but Mongoose ODM is only for MongoDB, while Prisma provides both ORM and ODM capabilities depending on the database we choose to use.


## Difference between Prisma and Mongoose:

| Feature         | Prisma (ORM)                                   | Mongoose (ODM)                             |
| --------------- | ---------------------------------------------- | ------------------------------------------ |
| Database Type   | Relational (PostgreSQL, MySQL, etc.) + MongoDB | NoSQL (MongoDB only)                       |
| Data Model      | Table-based (rows & columns)                   | Document-based (JSON/BSON)                 |
| Schema Approach | Schema-first (defined in Prisma schema file)   | Schema-on-top (defined in code)            |
| Type Safety     | Strong (auto-generated TypeScript types)       | Limited (requires manual TypeScript setup) |
| Migrations      | Built-in (Prisma Migrate)                      | Not built-in                               |

# Schema: 

## Common Data Types: 
| Prisma Type | Description                   | Example Use Case      | Notes                      |
| ----------- | ----------------------------- | --------------------- | -------------------------- |
| `String`    | Text data                     | name, email           | Maps to VARCHAR/TEXT       |
| `Int`       | Integer numbers               | age, count            | 32-bit                     |
| `BigInt`    | Large integers                | large counters, IDs   | Use for high-scale systems |
| `Float`     | Decimal numbers (approximate) | ratings, percentages  | ⚠️ Not for money            |
| `Decimal`   | Precise decimal               | price, financial data | ✅ Best for money           |
| `Boolean`   | true/false                    | isActive              | Simple flag                |
| `DateTime`  | Date + time                   | createdAt             | ISO format                 |
| `Json`      | JSON object                   | metadata, configs     | Flexible but risky         |
| `Bytes`     | Binary data                   | file storage          | Rare use                   |
| `Enum`      | Fixed values                  | role, status          | Safer than String          |

```sql
enum ProductStatus {
  AVAILABLE
  OUT_OF_STOCK
  DISCONTINUED
}

model Product {
  id          String   @id @default(uuid())
  name        String
  description String?

  stock       Int
  bigCounter  BigInt

  price       Decimal
  rating      Float

  isActive    Boolean  @default(true)

  metadata    Json

  image       Bytes?

  status      ProductStatus

  createdAt   DateTime @default(now())
}
```


## Common Column Constraints: 

| Constraint           | Syntax         | Purpose                   |
| -------------------- | -------------- | ------------------------- |
| Primary Key          | `@id`          | Uniquely identifies a row |
| Default Value        | `@default()`   | Sets default value        |
| Unique               | `@unique`      | Prevent duplicate values  |
| Optional (NULL)      | `?`            | Allows null               |
| Updated At           | `@updatedAt`   | Auto-update timestamp     |
| Relation             | `@relation()`  | Foreign key mapping       |
| Map (DB column name) | `@map()`       | Rename column in DB       |
| Native Type          | `@db.*`        | DB-specific type          |
| Composite ID         | `@@id([])`     | Multi-column primary key  |
| Composite Unique     | `@@unique([])` | Multi-column unique       |
| Index                | `@@index([])`  | Improve query performance |
| Map Table Name       | `@@map()`      | Rename table in DB        |

```sql
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  username  String   @unique

  age       Int      @default(18)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  posts     Post[]

  @@index([email])
}

model Post {
  id        String   @id @default(uuid())
  title     String

  authorId  String
  author    User     @relation(fields: [authorId], references: [id])

  createdAt DateTime @default(now())

  @@index([authorId])
}
```

# CRUD Operations: 

## Create(POST):

### create():

```js
app.post("/users", async (req: Request, res: Response) => {
    try {
        const { name, email } = req.body;

        const user = await prisma.user.create({
            data: {
                name,
                email,
            },
        });

        res.send({
            success: true,
            message: "User created successfully",
            data: user,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

### createMany():

```js
app.post("/users/bulk", async (req: Request, res: Response) => {
    try {
        const users = req.body.users;

        const result = await prisma.user.createMany({
            data: users,
            skipDuplicates: true,
        });

        res.send({
            success: true,
            message: "Users created successfully",
            data: result,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

Note: skipDuplicates is not supported on MongoDB, SQLServer, or SQLite.

### createManyAndReturn():

```js
app.post("/users/bulk-return", async (req: Request, res: Response) => {
    try {
        const users = req.body.users;

        const result = await prisma.user.createManyAndReturn({
            data: users,
            skipDuplicates: true,
        });

        res.send({
            success: true,
            message: "Users created and returned successfully",
            data: result,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

Note: Supported only PostgreSQL, CockroachDB, and SQLite.

### Create Options: 
#### Select: 
Controls what fields we get back from DB

```js
await prisma.user.create({
    data: {
        name,
        email
    },
    select: {
        id: true,
        name: true,
    },
});
```

#### include: 
Controls what related records we get back from DB

```js
await prisma.user.create({
    data: {
        name, 
        email
    },
    include: {
        posts: true,
    },
});
```

```js
await prisma.user.create({
    data: {
        name,
        email,
    },
    include: {
        posts: {
            select: {
                id: true,
                title: true,
            },
        },
    },
});
```

#### skipDuplicates (Only for createMany):
Skip records with duplicate unique fields

```js
await prisma.user.createMany({
  data: [
    { name: "Bob", email: "bob@prisma.io" },
    { name: "Yewande", email: "bob@prisma.io" },
  ],
  skipDuplicates: true,
});
```

here, only one data will be created

## READ(GET):
### findMany():
Fetch multiple records (most used query in Prisma)

```js
app.get("/users", async (_req: Request, res: Response) => {
    try {
        const users = await prisma.user.findMany();

        res.send({
            success: true,
            message: "Users fetched",
            data: users,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

### findUnique():

```js
app.get("/users/:id", async (req: Request, res: Response) => {
    try {
        const id = Number(req.params.id);

        const user = await prisma.user.findUnique({
            where: {
                id,
            },
        });

        res.send({
            success: true,
            message: "User fetched",
            data: user,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

### findUniqueOrThrow():

Same as findUnique(), but throws error if not found

```js
app.get("/users/:id", async (req: Request, res: Response) => {
    try {
        const id = Number(req.params.id);

        const user = await prisma.user.findUniqueOrThrow({
            where: {
                id,
            },
        });

        res.send({
            success: true,
            message: "User fetched",
            data: user,
        });

    } catch (error: any) {
        res.status(404).send({
            success: false,
            message: "User not found",
        });
    }
});
```

### findFirst():
Fetch first matching record based on filter

```js
app.get("/users", async (_req: Request, res: Response) => {
    try {
        const user = await prisma.user.findFirst({
            where: {
                name: "Tamim",
            },
        });

        res.send({
            success: true,
            message: "First matching user",
            data: user,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

### findFirstOrThrow():
Same as findFirst() but throws error if no match found

```js
app.get("/users/first", async (_req: Request, res: Response) => {
    try {
        const user = await prisma.user.findFirstOrThrow({
            where: {
                name: "Tamim",
            },
        });

        res.send({
            success: true,
            message: "User found",
            data: user,
        });

    } catch (error: any) {
        res.status(404).send({
            success: false,
            message: "No user found",
        });
    }
});
```

### count():
Returns number of records

```js
app.get("/users/count", async (_req: Request, res: Response) => {
    try {
        const count = await prisma.user.count();

        res.send({
            success: true,
            message: "User count fetched",
            data: count,
        });

    } catch (error: any) {
        res.status(500).send({
            success: false,
            message: error.message,
        });
    }
});
```

### Common read options:
#### WHERE: 
| SQL Meaning | Prisma Operator | Example                |
| ----------- | --------------- | ---------------------- |
| =           | direct value    | `where: { age: 20 }`   |
| !=          | `not`           | `{ age: { not: 20 } }` |
| >           | `gt`            | `{ age: { gt: 18 } }`  |
| <           | `lt`            | `{ age: { lt: 30 } }`  |
| >=          | `gte`           | `{ age: { gte: 18 } }` |
| <=          | `lte`           | `{ age: { lte: 30 } }` |

| SQL Meaning | Prisma Operator | Example                         |
| ----------- | --------------- | ------------------------------- |
| IN          | `in`            | `{ id: { in: [1,2,3] } }`       |
| NOT IN      | `notIn`         | `{ id: { notIn: [1,2] } }`      |
| BETWEEN     | `gte + lte`     | `{ age: { gte: 10, lte: 20 } }` |


| SQL Meaning              | Prisma Operator       | Example                                              |
| ------------------------ | --------------------- | ---------------------------------------------------- |
| LIKE %text%              | `contains`            | `{ name: { contains: "tam" } }`                      |
| LIKE text%               | `startsWith`          | `{ name: { startsWith: "Ta" } }`                     |
| LIKE %text               | `endsWith`            | `{ name: { endsWith: "im" } }`                       |
| ILIKE (case-insensitive) | `mode: "insensitive"` | `{ name: { contains: "tam", mode: "insensitive" } }` |


| Meaning     | Prisma Operator | Example                    |
| ----------- | --------------- | -------------------------- |
| IS NULL     | `equals: null`  | `{ email: null }`          |
| IS NOT NULL | `not: null`     | `{ email: { not: null } }` |

| Meaning           | Prisma Operator                | Example                               |
| ----------------- | ------------------------------ | ------------------------------------- |
| Has value in list | `has` / `hasEvery` / `hasSome` | `{ tags: { has: "js" } }`             |
| Contains all      | `hasEvery`                     | `{ tags: { hasEvery: ["js","ts"] } }` |
| Contains any      | `hasSome`                      | `{ tags: { hasSome: ["js","ts"] } }`  |


- Comparison Filters: 
```sql
where: {
    age: {
        gt: 18,
        lt: 30,
        gte: 18,
        lte: 30,
        not: 25
    }
}
```

- Multiple conditions (AND / OR / NOT): 
```sql
where: {
    AND: [
        { age: { gte: 18 } },
        { name: { contains: "a" } }
    ]
}
```
```sql
where: {
    OR: [
        { name: "Tamim" },
        { name: "John" }
    ]
}
```
```sql
where: {
    NOT: {
        name: "Tamim"
    }
}
```
- IN / NOT IN: 

```sql
where: {
    name: {
        in: ["A", "B", "C"],
        notIn: ["X", "Y"]
    }
}
```

- Pattern Matching: 

```sql
where: {
    name: {
        contains: "tam",
        startsWith: "Ta",
        endsWith: "im",
        mode: "insensitive"
    }
}
```

#### orderBy: 
```sql
orderBy: {
    age: "asc"
}
```

```sql
orderBy: {
    age: "desc"
}
```

```sql
orderBy: [
    { age: "desc" },
    { name: "asc" }
]
```

#### take and skip: 

```sql
const page = 2;
const limit = 10;

await prisma.user.findMany({
    skip: (page - 1) * limit,
    take: limit
});
```
#### DISTINCT: 

```sql
distinct: ["name"]
```

### aggregate():
In Prisma, aggregation is done using:
- aggregate()
- groupBy()
- _count, _sum, _avg, _min, _max


```sql
const result = await prisma.user.aggregate({
  _count: true,
  _avg: {
    age: true,
    salary: true,
  },
  _sum: {
    salary: true,
  },
  _min: {
    age: true,
  },
  _max: {
    age: true,
  },
});

-- output 
{
  _count: 10,
  _avg: { age: 25, salary: 50000 },
  _sum: { salary: 500000 },
  _min: { age: 18 },
  _max: { age: 40 }
}
```

```sql
const result = await prisma.user.aggregate({
  where: {
    age: {
      gt: 20,
    },
  },
  _avg: {
    salary: true,
  },
});
```

- groupBy(): 
  
```sql
const result = await prisma.user.groupBy({
  by: ['age'],
  _count: {
    _all: true,
  },
  _avg: {
    salary: true,
  },
});

-- output
[
  {
    age: 20,
    _count: { _all: 3 },
    _avg: { salary: 40000 },
  },
  {
    age: 25,
    _count: { _all: 5 },
    _avg: { salary: 60000 },
  },
]
```

with having: 

```sql
const result = await prisma.user.groupBy({
  by: ['age'],
  _count: {
    _all: true,
  },
  having: {
    age: {
      gt: 20,
    },
  },
});

```

```sql
const result = await prisma.user.groupBy({
  by: ['age'],
  _count: {
    _all: true,
  },
  orderBy: {
    age: 'asc',
  },
});
```