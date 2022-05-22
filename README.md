# Create a NestJS Project
### Project setup
<!-- TODO: introduction to NestJS -->
```bash
# Install NestJS
npm i -g @nestjs/cli

# Create a new NetsJS application 'nestjs-graphql-mongodb'
# option: npm
nest new nestjs-graphql-mongodb
cd nestjs-graphql-mongodb

# Install dependencies
npm i @nestjs/graphql graphql-tools graphql @nestjs/apollo apollo-server-express @nestjs/mongoose mongoose class-validator --save
```
Keep only root files `main.ts` and `app.module.ts`

![Alt text](readme/nest-new.png)

# Start a mongo server instance
### Install Docker engine
https://docs.docker.com/engine/install/ubuntu/
### Docker componse
The Compose file defines services, networks, and volumes for a Docker application. A `service` is an abstract concept implemented on platforms by running the same `container image`. Services store and share persistent data into `volumes`.

The top-level `version` property is defined by specification for backward compability but is only informative.

The top-level `name` property is defined by specification as project name to be used if user doesn't set one explicitly.

### MongoDB 
MongoDB is a NoSQL database that uses JSON-like documents. The MongoDB server in the image listens on the standard port, 27017.

When the container is started for the first time, it will execute files with the extension `.sh` and `.js` that are found in `/docker`.

Create a `docker` folder in the root of the NestJS project.

![Alt text](readme/docker-folder.png)

`docker-compose.yml`
```yml
version: '3.9'

services: 
  mongo: 
    image: mongo:latest 
    container_name: my-mongo 
    restart: always 
    environment: 
      MONGO_INITDB_ROOT_USERNAME: rootuser 
      MONGO_INITDB_ROOT_PASSWORD: rootpassword 
    ports: 
      - 27017:27017 
    volumes: 
      - ./docker-entrypoint-initdb.d/mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
```
`mongo-init.js`
```javascript
db = db.getSiblingDB('test');
db.createUser(
    {
        user: "user",
        pwd: "password",
        roles: [
            {
                role: "readWrite",
                db: "test"
            }
        ]
    }
);
db.createCollection('users');
```
Useful docker commands
```bash
# Docker Compose
docker-compose up -d
# List containers
docker ps -a
# Stop and Remove containers
docker stop <container-name> && docker rm <container-name>
# List images
docker images
# Remove images
docker rmi <image-id>
```
# Mongoose setup
Mongoose is a object modeling tool that can be used to integrate Nest with MongoDB database.

```bash
# Create module 'database'
nest g mo database
```
Import the MongooseModule into the DatabaseModule.The forRoot() method accepts the same configuration object as mongoose.connect() from the Mongoose package.

`database.module.ts`
```javascript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [MongooseModule.forRoot('mongodb://user:password@localhost/test')],
})
export class DatabaseModule {}
```
### Model injection
With Mongoose, everything is derived from a Schema. Each schema maps to a MongoDB collection and defines the shape of the documents within that collection. Schemas are used to define Models. Models are responsible for creating and reading documents from the underlying MongoDB database.

`user.schema.ts`
```javascript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

export type UserDocument = User & Document;

@Schema()
export class User {
  @Prop()
  name: string;

  @Prop()
  email: string;

  @Prop()
  age: number;
}

export const UserSchema = SchemaFactory.createForClass(User);
```
# GraphQL setup
GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data which gives clients the power to ask for exactly what they need and nothing more.

Nest offers two ways of building GraphQL applications, the code first and the schema first methods. 

In the `code first` approach, decorators and TypeScript classes generates the corresponding GraphQL schema. This approach works exclusively with TypeScript and avoid context switching between language syntaxes.

In the `schema first` approach, the source of truth is GraphQL SDL (Schema Definition Language) files. SDL is a language-agnostic way to share schema files between different platforms. Nest automatically generates TypeScript definitions (using either classes or interfaces) based on the GraphQL schemas to reduce the need to write redundant boilerplate code.

```bash
# Create resource 'user'
# option: GraphQL (code first), generate CRUD entry points
# avoid creating test files
nest g resource user --no-spec
```
Import the GraphQLModule and configure it with the forRoot() static method.

`app.module.ts`
```javascript
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { DatabaseModule } from './database/database.module';
import { UserModule } from './user/user.module';

@Module({
  imports: [
    DatabaseModule,
    UserModule,
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: 'schema.gql',
    }),
  ],
})
export class AppModule {}
```
## Reference
### NestJS
- NestJS First Steps: https://docs.nestjs.com/first-steps
- NestJS Recipes: https://docs.nestjs.com/recipes/crud-generator
- NestJS Mongo: https://docs.nestjs.com/techniques/mongodb
- NestJS GraphQL: https://docs.nestjs.com/graphql/quick-start
- Tutorial: https://javascript.plainenglish.io/build-a-scalable-graphql-server-with-nestjs-mongodb-typescript-1eeda049f7c8
### Docker 
- DockerHub Mongo: https://hub.docker.com/_/mongo
- MongoDB server with Docker Compose: https://dev.to/sonyarianto/how-to-spin-mongodb-server-with-docker-and-docker-compose-2lef
- MongoDB and Docker: https://www.digitalocean.com/community/tutorials/how-to-set-up-flask-with-mongodb-and-docker

<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="200" alt="Nest Logo" /></a>
</p>

[circleci-image]: https://img.shields.io/circleci/build/github/nestjs/nest/master?token=abc123def456
[circleci-url]: https://circleci.com/gh/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="_blank">Node.js</a> framework for building efficient and scalable server-side applications.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/dm/@nestjs/common.svg" alt="NPM Downloads" /></a>
<a href="https://circleci.com/gh/nestjs/nest" target="_blank"><img src="https://img.shields.io/circleci/build/github/nestjs/nest/master" alt="CircleCI" /></a>
<a href="https://coveralls.io/github/nestjs/nest?branch=master" target="_blank"><img src="https://coveralls.io/repos/github/nestjs/nest/badge.svg?branch=master#9" alt="Coverage" /></a>
<a href="https://discord.gg/G7Qnnhy" target="_blank"><img src="https://img.shields.io/badge/discord-online-brightgreen.svg" alt="Discord"/></a>
<a href="https://opencollective.com/nest#backer" target="_blank"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor" target="_blank"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-ff3f59.svg"/></a>
    <a href="https://opencollective.com/nest#sponsor"  target="_blank"><img src="https://img.shields.io/badge/Support%20us-Open%20Collective-41B883.svg" alt="Support us"></a>
  <a href="https://twitter.com/nestframework" target="_blank"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Installation

```bash
$ npm install
```

## Running the app

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://kamilmysliwiec.com)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](LICENSE).
