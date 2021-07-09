⚠️ **EXPERIMENTAL**: this project is unpublished and available only via Github.
Please feel free to submit issues but I can't guarantee a response.

# federation-converter

Utility for converting GraphQL schemas intended for [Apollo Federation][fed]
into valid GraphQL documents.

## Motivation

When creating federated graphs, you end up writing invalid GraphQL schemas:

- directives like `@key` and `@provides` are not defined.
- `extend type` extensions reference undefined types.

This precludes the use of tools like [graphql-schema-linter][lint]. Use this
utility to convert the schema before linting:

```sh
npx github:apollosolutions/federation-converter myschema.graphql | npx \
  graphql-schema-linter --stdin
```

You can also convert a valid GraphQL schema into a federated schema for use
with Apollo tools like [rover][rover]:

```sh
npx github:apollosolutions/federation-converter --from-valid accounts.graphql | \
  rover subgraph publish mygraph@current --name accounts --schema -
```

You can ignore common lint failures on federation elements like `_Entity` by
adding a [configuration file][config] with these settings:

```json
{
  "ignore": {
    "arguments-have-descriptions": ["Query._entities.representations"],
    "fields-are-camel-cased": ["Query._entities", "Query._service"],
    "fields-have-descriptions": ["Query._entities", "Query._service"],
    "types-have-descriptions": ["Query", "_Entity", "_Any", "_Service"],
    "types-are-capitalized": ["_Service"]
  }
}
```

[fed]: https://www.apollographql.com/docs/federation/
[lint]: https://github.com/cjoudrey/graphql-schema-linter
[rover]: https://www.apollographql.com/docs/rover/
[config]: https://github.com/cjoudrey/graphql-schema-linter#configuration-file

## Caveats

- The transformations are not symmetrical: some information like the `extend`
  keyword is lost, so you can't transform your schema then reverse the
  transformation and expect the same output.
- Line numbers change completely, so the schema linting report won't accurately
  reference lines and columns in the original file.
- Applied directives are stripped when converting a valid schema to a federated
  one with `--from-valid`. This is currently the expected behavior with federated
  graphs.
- Currently published only to GitHub and GitHub's NPM Registry. Installing NPM
  packages from GitHub requires a authenticating NPM with a Personal Access Token,
  [as documented here](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry#authenticating-to-github-packages).

```sh
echo "//npm.pkg.github.com/:_authToken=$GITHUB_PAT" >> ~/.npmrc
echo "@apollosolutions:registry=https://npm.pkg.github.com" >> .npmrc
npm i @apollosolutions/federation-converter
```
