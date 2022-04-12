# Building a GraphQL API using Prisma

If you are planning on building a GraphQL API, read on to find out how you can build one effortlessly using Prisma.

## Foreword

Traditional APIs focused on REST services with standard endpoints and HTTP verbs like GET, POST, PUT, and DELETE. With the advent of GraphQL, REST now has an alternative that is just as strong and viable for any project.

GraphQL was developed internally by Facebook back in 2012 and publicly released in 2015. Instead of building traditional request/response models, it enables you to interact with your application’s data through specific queries, fetching the exact data that you need. GraphQL provides a powerful way for web and mobile apps to fetch data from an API and Prisma is an excellent tool that greatly simplifies interacting with your database via a GraphQL API.

## What is Prisma?

Prisma is an open-source database toolkit segregated into three main tools, namely:
 - **Prisma Client**: A query builder that gets auto-generated from the Prisma schema with types tailored to your application.
 - **Prisma Migrate**: A service that automatically generates SQL database migrations, that are fully customizable. For quick data model prototyping in development, `prisma db push` lets you make changes to the database without generating migration files.
 - **Prisma Studio**: A visual database browser that opens in your browser to graphically view and edit the data.

Check out Prisma [here](https://www.prisma.io/).

## Why use Prisma

GraphQL is not a programming language capable of arbitrary computation, but is instead a language used to make requests to application services that have capabilities defined in this specification.

Prisma’s main goal is to make application developers more productive when working with databases. It is an excellent tool that greatly simplifies interacting with your database via a GraphQL API.

With this, let's start building our API.

## Requirements

To kickstart development, you'd need the following in your system:
 - [Node.js >= v10](https://nodejs.org/en/)
 - [Git](https://git-scm.com/) (Optional, in case you want to test and deploy your API)

We'll download the rest of the dependencies as we navigate through this tutorial. To create a GraphQL API with Prisma, we'll also be using Apollo-Server.

You might need a [GitHub](https://github.com/) account as well.

![Image](repo-assets/README.png)

## Creating the GraphQL API

In this tutorial, we'll be creating a GraphQL API for a College. The API will have multiple entities inclusive of *student*, *teacher*, *department* and *course*.

We will be defining new models in the Prisma schema, adapting the GraphQL schema to make use of the models.

There's going to be one-to-many relation from Department to the Student model as well as betwwen a Teacher, to a Course. This will allow us to represent the Teacher of Course and associate multiple Courses to each Teacher for instance.

### Step 1: Creating the Node.js Project

We'll start of by making the directory. The following commands will create that, navigate to the folder and initialize NPM in the project.

```bash
mkdir graphql-prisma
cd graphql-prisma
npm init --yes
```

The last command generates a `package.json` file which looks similar to the file below.

```json
{
  "name": "graphql-prisma",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

`package.json` will be used as the configuration file for the project. Now we are ready to install other dependencies. Execute the following command.

```bash
npm install apollo-server graphql --save
```

With the aforementioned command, we have installed two packages, namely:
1. **apollo-server:** A community-maintained open-source GraphQL server that's compatible with any GraphQL client.
2. **graphql:** The library we'll be using to build the GraphQL schema.

### Step 2: Setting Up Prisma with PostgreSQL

We'll install Prisma CLI in this step, alongside creating the initial Prisma Schema. We'll also setup PostgreSQL that we'll be running locally with Docker and connect Prisma to it.

The Prisma schema is the main configuration file for our Prisma setup and contains the database schema.

We'll start off by installing the Prisma CLI using the command below:

```bash
npm install prisma -D
```

The -D flag is the shortcut for `--save-dev`, which records the package as a development dependency under the `devDependencies` property in the `package.json` file.

The Prisma CLI will assist with database workflows such as running database migrations and generating Prisma Client.

Now let's set up PostgreSQL database using Docker. We need to create a new Docker Compose file using:

```bash
nano docker-compose.yml
```

Now paste the following lines in the Docker Compose Configuration File.

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:13
    restart: always
    environment:
      - POSTGRES_USER=db_user
      - POSTGRES_PASSWORD=db_password
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'
volumes:
  postgres:
```

This file is responsible for starting the official PostgreSQL Docker image on your machine.

The `POSTGRES_USER` and `POSTGRES_PASSWORD` environment variables set the credentials for the superuser. We will use these credentials to connect Prisma to the database. With the aforementioned lines of code, we define a volume where PostgreSQL will store its data, and bind the 5432 port on the machine to the same port in the Docker container.

With this, let's launch the PostgreSQL database server with the following:

```bash
docker-compose up -d
```

To verify whether the database is up and running, execute this command:

```bash
docker ps
```

The output will share the status of the container running PostgreSQL. With all of this in place, let's initiate the setup for Prisma. Since all invocations of Prisma CLI should be prefixed with `npx` to ensure local installation is used, execute the following:

```bash
npx prisma init
```

The Prisma CLI creates a new folder named `prisma` within the project directory. The folder consists of `schema.prisma` which is the main configuration file for the Prisma project including the data model, and a `.env` file which will carry the databse connection URL.

To update the location of the database, modify the `DATABASE_URL` environment variable so it looks like this:

```env
DATABASE_URL="postgresql://db-user:db-password@localhost:5432/my-blog?schema=public"
```

As you can see here, we are using the database credentials we have already specified in the Docker Compose File.

### Step 3: Defining the Data Model with Prisma Migrate
In this step, we are simply going to define the data model for our API in the Prisma Schema that we generated in the previous step. The data model will then be mapped to the databse in Prisma Migrate, which will generate and send the SQL statements for creating the tables that correspond to your data model.

Like we discussed before, we are going to have `student`, `teacher`, `department` and `course` entities.

Open the `schema.prisma` file housed inside the `prisma` folder in the project directory and paste the following model definitions:

```javascript
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Student {
  id        Int        @id @default(autoincrement())
  email     String     @unique @db.VarChar(255)
  fullName  String?    @db.VarChar(255)
  enrolled  Boolean    @default(false)
  dept      Department @relation(fields: [deptId], references: [id])
  deptId    Int
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@map(name: "student")
}

model Department {
  id          Int       @id @default(autoincrement())
  name        String    @unique
  description String?   @db.VarChar(500)
  students    Student[]
  courses     Course[]
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@map(name: "department")
}

model Teacher {
  id        Int         @id @default(autoincrement())
  email     String      @unique @db.VarChar(255)
  fullName  String?     @db.VarChar(255)
  courses   Course[]
  type      TeacherType @default(FULLTIME)
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt

  @@map(name: "teacher")
}

model Course {
  id          Int         @id @default(autoincrement())
  code        String      @unique
  title       String      @db.VarChar(255)
  description String?     @db.VarChar(500)
  teacher     Teacher?    @relation(fields: [teacherId], references: [id])
  teacherId   Int?
  dept        Department? @relation(fields: [deptId], references: [id])
  deptId      Int?
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  @@map(name: "course")
}

enum TeacherType {
  FULLTIME
  PARTTIME
}
```

We have added the following models to the Schema:
 - **Department**: Represents course Specialties.
 - **Teacher**: Represents course Instructors/Facilitators.
 - **Course**: Represents subject matters.
 - **Student**: Represents our main entity.

Next, let's create and apply the migration locally with the following command while creating the corresponding table in the database:

```bash
npx prisma migrate dev --name "init" 
```

This command creates a new migration on your file system and runs it against the database to create the database schema. Now let's install Prisma Client using:

```bash
npm install @prisma/client
```

We will now use Prisma Client in the GraphQL resolvers to read and write data in the database. Create a new directory named `src` and create `database.js` file. Paste the following lines of code inside the same.

```javascript
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient();

module.exports = {
  prisma,
}
```

Now we'll start working on the GraphQL Schema.

### Step 4: Defining the GraphQL Schema

We'll start off by creating a `schema.js` file inside the `src` directory, which will convert the GraphQL schema string into the format that Apollo expects. Paste the following in the file:

```sql
const { gql } = require("apollo-server")

const typeDefs = gql `

 type Student {
    id: ID!
    email: String!
    fullName: String!
    dept: Department!
    enrolled: Boolean
    updatedAt: String
    createdAt: String
  }

  type Department {
    id: ID!
    name: String!
    description: String
    students: [Student]
    courses: [Course]
    updatedAt: String
    createdAt: String
  }

  type Teacher {
    id: ID!
    email: String!
    fullName: String!
    courses: [Course]
    type: TeacherType
    updatedAt: String
    createdAt: String
  }

  type Course {
    id: ID!
    code: String!
    title: String!
    description: String
    teacher: Teacher
    dept: Department
    updatedAt: String
    createdAt: String
  }

  input TeacherCreateInput {
    email: String!
    fullName: String!
    courses: [CourseCreateWithoutTeacherInput!]
  }

  input CourseCreateWithoutTeacherInput {
    code: String!
    title: String!
    description: String
  }

  type Query {
    enrollment: [Student!]
    students: [Student!]
    student(id: ID!): Student
    departments: [Department!]!
    department(id: ID!): Department
    courses: [Course!]!
    course(id: ID!): Course
    teachers: [Teacher!]!
    teacher(id: ID!): Teacher
  }

  type Mutation {
    registerStudent(email: String!, fullName: String!, deptId: Int!): Student!
    enroll(id: ID!): Student
    createTeacher(data: TeacherCreateInput!): Teacher!
    createCourse(code: String!, title: String!, teacherEmail: String): Course!
    createDepartment(name: String!, description: String): Department!
  }

enum TeacherType {
  FULLTIME
  PARTTIME
}
`

module.exports = {
    typeDefs,
  }
```

With the aforementioned lines of code, we are bringing about quite a few changes to the GraphQL schema. Let's take a look at them.

 - `teacher` now returns an array of `course`.
 - `department` returns an array of `student`.
 - The `createTeacher` mutation expects the `TeacherCreateInput` as its input type.
 - The `teacherEmail` field is an optional argument to the `createCourse` mutation.

To match the schema we created, we need to create resolvers.

### Step 5: Defining the GraphQL Resolvers

In order to work with resolvers, we first need to create a sub-directory called `resolvers` inside the `src` directory. In this sub-directory, we need to create three files, namely `index.js`, `query.js`, and `mutation.js`.

Copy the lines of code below and paste them in the `mutation.js` file:

```javascript
const { prisma } = require("../database.js");

const Mutation = {
    registerStudent: (parent, args) => {
      return prisma.student.create({
        data: {
          email: args.email,
          fullName: args.fullName,
          dept: args.deptId && {
            connect: { id: args.deptId },
          },
        },
      });
    },
    enroll: (parent, args) => {
      return prisma.student.update({
        where: { id: Number(args.id) },
        data: {
          enrolled: true,
        },
      });
    },

    createTeacher: (parent, args) => {
      return prisma.teacher.create({
        data: {
          email: args.data.email,
          fullName: args.data.fullName,
          courses: {
            create: args.data.courses,
          },
        },
      });
    },

    createCourse: (parent, args) => {
      console.log(parent, args)
      return prisma.course.create({
        data: {
          code: args.code,
          title: args.title,
          teacher: args.teacherEmail && {
            connect: { email: args.teacherEmail },
          },
        },
      });
    },

    createDepartment: (parent, args) => {
      return prisma.department.create({
        data: {
          name: args.name,
          description: args.description,
        },
      });
    },
  };

  module.exports = {
    Mutation,
  }
```

The `createCourse` mutation resolver after these changes uses the `teacherEmail` argument (if passed) to create a relation between the created `course` and an existing `teacher`.

Once you are through, paste the following lines of code in the `query.js` file:

```javascript
const { prisma } = require("../database.js");

const Query = {
    enrollment: (parent, args) => {
      return prisma.student.findMany({
        where: { enrolled: true },
      });
    },
    student: (parent, args) => {
      return prisma.student.findFirst({
        where: { id: Number(args.id) },
      });
    },

    students: (parent, args) => {
      return prisma.student.findMany({});
    },

    departments: (parent, args) => {
      return prisma.department.findMany({});
    },

    department: (parent, args) => {
      return prisma.department.findFirst({
        where: { id: Number(args.id) },
      });
    },

    courses: (parent, args) => {
      return prisma.course.findMany({});
    },

    course: (parent, args) => {
      return prisma.course.findFirst({
        where: { id: Number(args.id) },
      });
    },

    teachers: (parent, args) => {
      return prisma.teacher.findMany({});
    },

    teacher: (parent, args) => {
      return prisma.teacher.findFirst({
        where: { id: Number(args.id) },
      });
    },
  };

  module.exports = {
    Query,
  }
```

Lastly, copy the code below into the `index.js` file:

```javascript
const { prisma } = require("../database.js");
const { Query } = require("./query.js");
const { Mutation } = require("./mutation.js");

const Student = {
  id: (parent, args, context, info) => parent.id,
  email: (parent) => parent.email,
  fullName: (parent) => parent.fullName,
  enrolled: (parent) => parent.enrolled,
  dept: (parent, args) => {
    return prisma.department.findFirst({
      where: { id: parent.dept },
    });
  },
};

const Department = {
  id: (parent) => parent.id,
  name: (parent) => parent.name,
  description: (parent) => parent.description,
  students: (parent, args) => {
    return prisma.department.findUnique({
        where: { id: parent.id },
      }).students();
  },
  courses: (parent, args) => {
    return prisma.department.findUnique({
        where: { id: parent.id },
      }).courses();
  },
};

const Teacher = {
  id: (parent) => parent.id,
  email: (parent) => parent.email,
  fullName: (parent) => parent.fullName,
  courses: (parent, args) => {
    return prisma.teacher.findUnique({
        where: { id: parent.id },
      }).courses();
  },
};

const Course = {
  id: (parent) => parent.id,
  code: (parent) => parent.code,
  title: (parent) => parent.title,
  description: (parent) => parent.description,
  teacher: (parent, args) => {
    return prisma.course.findUnique({
        where: { id: parent.id },
      }).teacher();
  },
  dept: (parent, args) => {
    return prisma.course.findUnique({
      where: { id: parent.id },
    }).dept();
  },
};

const resolvers = {
  Student,
  Department,
  Teacher,
  Course,
  Query,
  Mutation,
};

module.exports = {
  resolvers,
};
```

What's eventually happening now is that the new `createTeacher` mutation resolver creates a `teacher` and related `courses` using nested writes.

Additionally, the `Teacher.courses` and `Post.teacher` resolvers define how to resolve the courses and teacher fields when the `Teacher` or `Post` are queried. These use Prisma’s Fluent API to fetch the relations. You can read more about it [here](https://www.prisma.io/docs/concepts/components/prisma-client/relation-queries#fluent-api)

Let's dive into building the GraphQL Server finally.

### Step 6: Creating the GraphQL Server

This is the final step towards the development of our API. We'll create the GraphQL server with Apollo Server and bind it to a port so that the server can accept connections.

To do so, we need to create a file `index.js` inside the `src` folder and paste the following lines of code:

```javascript
const { ApolloServer } = require('apollo-server')
const { typeDefs } = require('./schema')
const { resolvers } = require('./resolvers')

const port = process.env.PORT || 9090;

const server = new ApolloServer({ resolvers, typeDefs });

server.listen({ port }, () => console.log(`Server runs at: http://localhost:${port}`));
```

Let's start the server now to test the GraphQL API. Execute the following command:

```bash
npm install nodemon -D
npm start
```

That's it. You should now be greeted by the GraphQL Playground, which looks similar to this:

![GraphQL Playground](repo-assets/graphql.png)

### Step 7: Testing and Deployment

We can test the backend of our GraphQL API by executing the following GraphQL queries and mutations in the Playground:

**Create Departments**

```
mutation {
  createDepartment(name: "Backend Engineering", description: "Express, ApolloServer, Prisma, Docker, Postgres") {
    id
    name
    description

  }
}
```

**Create Course**

```
mutation CreateCourseMutation($createCourseCode: String!, $createCourseTitle: String!) {
  createCourse(code: $createCourseCode, title: $createCourseTitle) {
    id
    code
    title
    description
    teacher {
      id
      fullName
    }
  }
}
```

**Create Teacher**

```
mutation CreateTeacherMutation($createTeacherData: TeacherCreateInput!) {
  createTeacher(data: $createTeacherData) {
    id
    fullName
    createdAt
    courses {
      id
      code
      title
    }
  }
}
```

We can also fetch the `teacher` whenever the return value of a query is `Course`. In this example, the `Course.teacher` resolver will be called.

## Ending Notes

We have successfully evolved our database schema with Prisma Migrate and exposed the new model in the GraphQL API, harnessing the power of GraphQL and Apollo as well. 

Take a look at the official documentation of Prisma [here](https://www.prisma.io/docs) to get a better perspective about the services offered.

GraphQL fetches data with a single call, provisioning tailored requests. It reduces bandwidth by not over-fetching data and it's suitable for smaller networks and microservices. Make sure you take a look at this resource for more insights into GraphQL, available [here](https://www.howtographql.com/graphql-js/0-introduction/).

## References

 - [How To Build a GraphQL API with Prisma and Deploy to DigitalOcean's App Platform](https://www.digitalocean.com/community/tutorials/how-to-build-a-graphql-api-with-prisma-and-deploy-to-digitalocean-s-app-platform)
 - [How To Build a GraphQL API with Apollo-Server and Prisma](https://dev.to/nditah/how-to-build-a-graphql-api-with-apollo-server-and-prisma-1bfj)
 - [Basic GraphQL API with Node, Prisma and Apollo](https://www.viget.com/articles/getting-started-with-node-prisma-and-apollo/)