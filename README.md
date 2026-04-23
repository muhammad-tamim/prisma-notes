<h1 align="center">Prisma Notes</h1>

- [Setup and Installation:](#setup-and-installation)
  - [Express + PostgreSQL + Prisma + TS](#express--postgresql--prisma--ts)
- [Introduction:](#introduction)
  - [What is Prisma:](#what-is-prisma)
  - [What is ORM and ODM:](#what-is-orm-and-odm)
  - [Difference between Prisma and Mongoose:](#difference-between-prisma-and-mongoose)

# Setup and Installation: 
## Express + PostgreSQL + Prisma + TS
- Step 1: Install dependencies
```bash
npm init -y
npm i express pg dotenv @prisma/client @prisma/adapter-pg
npm i -D typescript tsx prisma @types/node @types/express @types/pg 
```

```bash
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
  email String  @unique
  name  String?
  posts Post[]
} 

model Post { 
  id        Int     @id @default(autoincrement()) 
  title     String
  content   String?
  published Boolean @default(false) 
  author    User    @relation(fields: [authorId], references: [id]) 
  authorId  Int
} 
```

step 6: Create and apply your first migration: 

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

- step 8: Write your first query: 

```ts
// script.ts
import { prisma } from "./lib/prisma";

async function main() {
  // Create a new user with a post
  const user = await prisma.user.create({
    data: {
      name: "Alice",
      email: "alice@prisma.io",
      posts: {
        create: {
          title: "Hello World",
          content: "This is my first post!",
          published: true,
        },
      },
    },
    include: {
      posts: true,
    },
  });
  console.log("Created user:", user);

  // Fetch all users with their posts
  const allUsers = await prisma.user.findMany({
    include: {
      posts: true,
    },
  });
  console.log("All users:", JSON.stringify(allUsers, null, 2));
}

main()
  .then(async () => {
    await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });
```


Run the script, You should see the created user and all users printed to the console!

```bash
npx tsx script.ts
```

step 9: Explore your data with Prisma Studio

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

