---
Tags:
  - drizzle
  - database
  - nodejs
---
 # Drizzle ORM
 


## Install
```sh
npm i drizzle-orm postgres && npm i -D drizzle-kit
```

## Drizzle Kit Setup
Create a `drizzle.config.js` by executing `touch drizzle.config.js`
```js
if (!process.env.DATABASE_URL) {
console.error("Database String not found...")
process.exit(1)
}

/** @type { import("drizzle-kit").Config } */
export default {
dialect: "postgresql",
schema: "./src/schemas/*",
out: "./drizzle",
dbCredentials: {
  url: process.env.DATABASE_URL,
},
verbose: true,
strict: true,
}
```

## Drizzle Kit Scripts
```json
"scripts": {
	// Other Node Scripts
    "drizzle:gen": "npx --env-file=.env drizzle-kit generate --config=drizzle.config.js",
    "drizzle:push": "npx --env-file=.env drizzle-kit push --config=drizzle.config.js",
    "drizzle:studio": "npx --env-file=.env drizzle-kit studio --config=drizzle.config.js"
  },
```

## Drizzle Connector Function
```js
/* Drizzle Postgres Connection */
export const pgDrizzle = drizzle(postgres(env.DATABASE_URL), {
  logger: true,
})
```

## A Simple Drizzle Schema
```ts
import {
  integer,
  pgEnum,
  pgTable,
  text,
  uniqueIndex,
  uuid,
} from "drizzle-orm/pg-core"

export const CourseType = pgEnum("courseType", ["Crash Course", "Series"])

export const Courses = pgTable(
  "course",
  {
    id: uuid("id").primaryKey().defaultRandom().unique(),
    slug: text("slug").notNull(),
    title: text("title").notNull(),
    subTitle: text("subTitle"),
    description: text("description"),
    courseFee: integer("courseFee"),
    courseType: CourseType("courseType").default("Crash Course"),
  },
  (table) => {
    return {
      slugIndex: uniqueIndex("slugIndex").on(table.slug),
      title: uniqueIndex("titleIndex").on(table.title),
    }
  },
)
```

## Drizzle PG Data Types
