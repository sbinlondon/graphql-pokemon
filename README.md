<h1 align="center">GraphQL Pokémon</h1>
<p align="center">
  Get information of a Pokémon with GraphQL!<br />
</p>

## FAC20 Info

Forked from [lucasbento](https://github.com/lucasbento/graphql-pokemon) and updated slightly for Founders & Coders cohort 20 to use in their GraphQL workshop!

## How to use

Get Pokémon's information through queries in GraphQL (based off the [PokeAPI](https://pokeapi.co/)).

```graphql
query {
  pokemon(name: "Pikachu") {
    id
    number
    name
    attacks {
      special {
        name
        type
        damage
      }
    }
    evolutions {
      id
      number
      name
      weight {
        minimum
        maximum
      }
      attacks {
        fast {
          name
          type
          damage
        }
      }
    }
  }
}
```

## Running

### Development

1. Fork and/or clone the repo.
2. Run `npm i` to install packages.
3. Run `npm run watch` to start the server
4. Open `http://localhost:5000/` to see th GraphQL playground!
5. Try [the query above](http://localhost:5000/?query=query%20%7B%0A%20%20pokemons(first%3A10)%20%7B%0A%20%20%20%20id%0A%20%20%20%20number%0A%20%20%20%20name%0A%20%20%20%20attacks%20%7B%0A%20%20%20%20%20%20special%20%7B%0A%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20type%0A%20%20%20%20%20%20%20%20damage%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20evolutions%20%7B%0A%20%20%20%20%20%20id%0A%20%20%20%20%20%20number%0A%20%20%20%20%20%20name%0A%20%20%20%20%20%20weight%20%7B%0A%20%20%20%20%20%20%20%20minimum%0A%20%20%20%20%20%20%20%20maximum%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20attacks%20%7B%0A%20%20%20%20%20%20%20%20fast%20%7B%0A%20%20%20%20%20%20%20%20%20%20name%0A%20%20%20%20%20%20%20%20%20%20type%0A%20%20%20%20%20%20%20%20%20%20damage%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D)


## Let's add some stuff!

## What's all this then?

Let's have a look around the repo.

In the `/schemas` folder we have our auto-generated schemas. Yep, you don't have to hand-build your schemas!

This is done in this repo with `scripts/buildSchema.js` which you can run by typing `npm run build-schema` in your terminal. We'll get to that later. Just have a quick look at the script though, and see that we are using `introspectionQuery` and `printSchema` from the official npm `graphql` package. Introspection is when we query the GraphQL server for information about itself. It's like asking, "Hey GraphQL, what are all the available fields on type GuineaPig?" and GraphQL will tell us. Through these [introspection queries](https://moonhighway.com/five-introspection-queries), the `printSchema` package creates a map of the schema for us based on the Query, Mutation, and other types we've defined.

So where do we define them?

In our root folder with `app.js` and `index.js` we have `schema.js`. Like everything else in GraphQL, the schema is a type (`SchemaType`). We use the `new GraphQLSchema({...})` constructor to create the schema object, and inside we define two main types: `QueryType` and `MutationType`. These types will hold all the information about how people can query our API and how they can change the data.

In the `/types` folder we have all our defined types. When we create types, we usually use `new GraphQLObjectType` (like in `QueryType.js`). At the top level, most custom created types ought to have: `name`, `description`, and `fields`.

For the main `QueryType`, these `fields` are going to be the actual queries we can run, like `getPokemons` and `getPokemon`. For the `fields` in a type, we have to define what type the field is (it's types all the way down), arguments (if it has any), and its _resolver_, or the function that runs to return the data.

Looking at the query `getPokemons` we see it does take a required argument (`type: new GraphQLNonNull(GraphQLInt),`) of an integer, or the number of Pokemon we want to fetch info about. Then the resolver is a function called `getPokemons` in `service/Pokemon.js`. If we open that, we see that `getPokemons` takes the argument, goes to our data (`pokemons/pokemons.json`, a big array of objects with data about each Pokemon) and filters that object based on our argument.

In GraphiQL, if we pass this query:

```
query {
  getPokemons(first:3) {
    number
    name
  }
}
```

Then we get the first 3 Pokemon in the JSON file - Bulbasaur, Ivysaur, and Venusaur.

Notice that the argument `first` is required in `getPokemons` - and if we forget it, GraphQL will automatically throw an error for us telling us that. But no arguments are required for `getPokemon`. However, in the resolver for `getPokemon`, we can also define our own errors. Here we can't require either `name` or `id` because we're not sure which one people will search by. But if they forget both, then we throw an error telling them that.

One more thing before we try adding our own query: when we define our resolvers, these generally take 4 arguments, which have to be in the proper order. We are only going to worry about the first two: the first is generally called `object` or `obj` in documentation (though like the variable `i` in loops, you can name it anything that makes sense in the context), and the second is `args`. You might notice for our `Query` type, the resolver arguments are `(_, args)`. This is because when we define our schema, `Query` and `Mutation` types are known as root types. They are the very top level of our GraphQL server. So they aren't being passed any data objects yet. However, if you look at `PokemonType` (which is the object where we define all the fields that are returned from our `getPokemon` and `getPokemons` queries and what they are allowed to be like `string`, `int`, `boolean`, etc) we see that every resolver for the `PokemonType` has the argument `(pokemon)`. This is because in our top level Query resolver for `getPokemon` we fetch the info about a Pokemon from our JSON file, but then we have to make this data match the shape defined in `PokemonType`. So we pass all that raw Pokemon data down to `PokemonType` and into the resolver for each field.

### Add a new Query

We'll begin by adding a new Query type to the schema.

If you open the GraphiQL IDE, we have two queries available: 

* `getPokemons` which returns us an array of Pokemon (the number is specified by the required argument `first`)
* `getPokemon` which requires us to pass an argument of the `id` of the Pokemon if we know it, or the `name`

But say we want to be able to search Pokemon by their elemental type (Grass, Fire, Electric, etc). We can write a query to take care of that.

Looking at our data in `pokemons/pokemons.json`, we can see that in each Pokemon object, there is a field called `types` that has an array of the elemental types associated with that Pokemon, so we know we're working with arrays. 

In `service/Pokemon.js` we have our functions that take care of the data wrangling/resolving for the queries; let's add one called `getPokemonsByType`:

```
export async function getPokemonsByType(typeName) {
  const type = typeName.toLowerCase().trim();

  const typeCapitalised = type.charAt(0).toUpperCase() + type.slice(1);

  const pokemon = pokemons.filter(({ types }) =>
    types.includes(typeCapitalised)
  );

  return pokemon || null;
}
```

And finally in our `QueryType.js` file let's add a new object under `getPokemon` and `getPokemons`:

```
    getPokemonsByType: {
      type: new GraphQLList(PokemonType),
      args: {
        type: {
          type: new GraphQLNonNull(GraphQLString),
        },
      },
      resolve: async (_, { type }) => {
        return await getPokemonsByType(type);
      },
    },
```

Our new `getPokemonsByType` query takes an argument that is a `type` like "Grass", the resolver uses our function we just wrote that returns the array of Pokemon that match the type, and we return from the query a `GraphQLList` (which is the fancy GraphQL name for an array) of `PokemonType`-shaped objects.

Let's try it out in the GraphiQL IDE:

```
query {
  getPokemonsByType(type: "Grass") {
    number
    name
  }
}
```

But something's gone wrong. _'Cannot query field \"getPokemonsByType\" on type \"Query\". Did you mean \"getPokemons\" or \"getPokemon\"?'_

This is because when we add a new type, field, etc to our schema that changes it, we need to update the schema. The schema is the end all, be all, one stop source of truth for your GraphQL API. If it doesn't exist in the schema, we can't use it. Run `npm run build-schema` and now try the query again. We should get an array of Pokemon back! 🎉
