# Subgraph Template - Apollo Server Typescript

This template repository contains the necessary boiler plate code for you to develop your Subgraph quickly with Apollo Server and Typescript. We recommend using code generation for your types in Typescript, this repository supports code generation for resolver types using [GraphQL Code Generator](https://www.graphql-code-generator.com/) and a REST API datasource using [swagger-typescript-api](https://github.com/acacode/swagger-typescript-api). A rich set of npm scripts  are included in the repository to provide a smoother deverloper experience along with working tests using [`jest`](https://jestjs.io/). Working examples for using GitHub Acitons to integrate with Apollo Studio for registering your schema and running schema validation. 

## Using this template

### Quickstart 

```
git clone https://github.com/apollographql/subgraph-template-typescript-apollo-server .
npm i
npm run start
```

To get running quickly, use the `start:dev-mocked` npm script. This will start the server with `nodemon` and mock any portions of the schema that doesn't have resolvers 

### The GraphQL Folder - your typeDefs and resolvers

By default, the `src/graphql` folder is where your schema modules should live. A schema modules is defined by a set of type definitions and the associated resolvers (which are optional). The `src/utils/schema.js` file contains a helper function `generateSubgraphSchema` to generate a `GraphQLSchema` from the modules you define in the folder. The modules expect the `.graphql` file and associated resolvers to be named the same (i.e. `bar.graphql` and `bar.js`). 

If you define resolvers, they should export `resolvers` and `typeDefs` like below:

```javascript
export const resolvers = {
  ...
}
export const typeDefs = gql`
type Query {
  helloWolrd: String
}
`
```

If you don't define any resolvers, they can be mocked automatically for you by using `start:mocked` or `start:dev-mocked`.

### Schema mocking

By setting `process.env.SHOULD_MOCK=true` any schema that doesn't have defined resolvers will be mocked. 

### Code Generation

**Resolver Types**

GraphQL Code Generator is integrated and exposed in the `generate:resolver-types` npm script. This runs with every `build` of the project prior to typescript compiles the project. The `codegen.yml` at the root of the project has the minimal configuration needed and can be used as is or customized to your needs.

**REST API DataSource** 

This is a simple example of how you could autogenerate a REST API class from a swagger file using `swagger-typescript-api`. This is exposed in the `generate:rest-api` npm script and runs with every `build` of the project prior to typescript compiles the project. The `swagger.yaml` file is located in the `src/datasources` folder and generates the class in the same folder. Looking at the command being run more closely:

```
swagger-typescript-api -p ./src/datasources/swagger.yaml -o ./src/datasources -n BarAPI.ts --axios
```

- `-n` is used to name the generated file, you can change this to be whatever you like
- `--axios` is an option to use `axios`, otherwise the default is `node-fetch` 

### DataSource construction

Apollo Server has a pattern where developers can define `dataSources` that will be available on the `context` in the GraphQL resolvers. In `src/utils/server.ts` there is a helper function `generateDataSources` that takes any datasources defined in `src/datasources` and populates them into the `ApolloServer` instance; this includes the autogenerated REST API class being added under whatever the file is named as. Each file should define a single class that is exported like below:

```typescript
import { DataSource } from 'apollo-datasource';
export class LocationsAPI extends DataSource {
  ...
}
```

### DataSourceContext Type

The shape of the `context` argument in our generated resolver types is based on `src/types/DataSourceContext.ts`. When you add a new datasource into the project, you'll need to add it into this interface so it's available in your resolvers.  

### Testing with Jest

Jest is a testing framework used by most of Apollo's current projects.

To run tests in the repo:
`npm test`

The Jest configuration can be found at `jest.config.ts`. As configured, Jest will run all files named `*.test.ts` found within any `__tests__` folder. This is simply a convention chosen by this repo and can be reconfigured via the `testRegex` configuration option in [`jest.config.ts`](jest.config.ts).

For more information on configuring Jest see the [Jest docs](https://jestjs.io/docs/configuration).

### NPM Script Explained

- `build`: This script is a "preLaunchTask" and does everything needed to have the project ready to run.
- `generate`: Run all `generate` scripts concurrently
- `generate:resolver-types`: Generate resolver types using GraphQL Code Generator
- `generate:rest-api`: Generate a REST API class based on a swagger file using `swagger-typescript-api`
- `postinstall`: Runs the `build` script after installing packages
- `prettier:check`: Check project with prettier
- `prettier:fix`: Fix project files with prettier
- `schema:copy-files`: Copy all `.graphql` files from `src` folder to `dist` folder. This is because typescript only compiles `.ts` files over into the output folder, `.graphql` files have to be manually copied. Some teams place the schema at the root of the project for this reason, but this can make deployment challenging. `shx cp` ensures the copy of those files works on Mac/Windows/Linux.
- `schema:output`: Combine all of the modularized schemas into a single schema and output to a file. This is intended to be used in CI operations for schema validation.
- `start`: Start up the project with mocking disabled
- `start:mocked`: Start up the project with mocking enabled
- `start:dev`: Build and Start up the project using nodemon with mocking disabled. The project will build and restart on any file changes.
- `start:dev-mocked`: Build and Start up the project using nodemon with mocking enabled. The project will build and restart on any file changes.
- `test`: Run local unit tests
- `test:ci`: Run local unit tests with CI options to generate a table

## Prettier

Prettier is an opinionated code formatting tool. 

To check for formatting issues:
`npm run prettier:check`

To auto-fix formatting issues:
`npm run prettier:fix`

This is enforced in CI via the `Prettier` job.

> For additional information on configuring Prettier, [visit the docs](https://prettier.io/docs/en/options).

## Renovate

### Installation

[GitHub app](https://github.com/apps/renovate)

> Note: a GitHub _org_ admin must approve app installations. By adding a GitHub app to your repo, you'll be submitting a request for approval. At the time of writing this, the GitHub UI doesn't make this clear.

Renovate automates dependency updates. The bot will open and merge PRs with updates to a variety of dependencies (including but not limited to npm dependencies). Renovate is _highly_ configurable via the [renovate.json5](renovate.json5) file. Package restrictions and scheduling are just a couple things that we commonly configure.

If you've configured PRs to require approval (mentioned in [GitHub](#github)), you may want to also install [Renovate's Approve bot](https://github.com/apps/renovate-approve). The approve bot will approve all renovate PRs in order to appease the PR approval requirement.

If you're unfamiliar with Renovate, the docs are really worth perusing even if just to get an idea of what kinds of configuration are possible.

> For additional information on configuring Renovate, [visit the docs](https://docs.renovatebot.com/).
