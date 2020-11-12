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

### What's all this then?

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

### Add a new Mutation

There aren't any mutations that allow us to change the data... yet. Let's add one.

First we have to create our root `MutationType` to add to the schema, like there is a root `QueryType`. So in the `/types` folder create a new file called `MutationType.js` and put the following in it:

```
import { GraphQLObjectType } from "graphql";

const MutationType = new GraphQLObjectType({
  name: "Mutation",
  description: "Change info for any Pokémon by number or name",
  fields: () => ({
    mutation: {
      type: MutationType,
      resolve: (_, ...args) => args,
    },
  }),
});

export default MutationType;
```

First we define our root `MutationType` as a new `GraphQLObjectType`, just like the root query; we name our object `Mutation`, give it a description (a generalized description since this is describing all mutations), and we write our mutation resolver. It's similar to the `QueryType` resolver in that, since we're at the highest level in our schema, we don't pass the `object` argument. All our resolver does is take the `args` passed into the mutation and passes them down to the next level, the actual named mutation we write next, for use.

Now we can create our mutation! Looking at our JSON file, one thing we notice is that there isn't any info about what color the Pokemon are. We can add that data. Let's call our mutation `addPokemonColor`.

```
import { GraphQLObjectType } from "graphql";

import PokemonType from "./PokemonType";

const MutationType = new GraphQLObjectType({
  name: "Mutation",
  description: "Change info for any Pokémon by number or name",
  fields: () => ({
    mutation: {
      type: MutationType,
      resolve: (_, ...args) => args,
    },
    updatePokemonColor: {
      type: PokemonType,
      description: "Add or update the color of a Pokemon",
      args: {

      },
      resolve: async (_, args) => {

      },
    },
  }),
});

export default MutationType;
```

We know we have to define the type it returns - in this case, we can use the `PokemonType` that already exists. We know we'll have to pass it some arguments, and we know we'll have to write a resolver. So what should we pass as arguments? [Best practice](https://www.apollographql.com/blog/designing-graphql-mutations-e09de826ed97) tells us that:

* mutations should have one argument called `input`, and it should be required
* the `type` of the input should be unique and relate to what the mutation is doing
* for example if we have a mutation `updatePokemonColor` the input type could be `PokemonColorInputType` and include the fields `name` (to find the Pokemon you want to update) and `color` (the color you want to update its data with); and a mutation `updatePokemonWeight`, would have a `PokemonWeightInputType` with the fields `name`, `maximumWeight`, and `minimumWeight`
* some APIs share input types - for these two mutations we could potentially have a `PokemonInfoInputType` and have the fields `name`, `maximumWeight`, `minimumWeight`, `color`, and whatever else we might want to change - but best practices say to be specific in your schema design

Having one argument called `input` means the client is only required to send one variable with per mutation instead of one for every argument on the mutation. It also means we can add/remove fields in the `input` and don't have to go changing that in lots of places client-side - our code is more resilient.

So we want our argument `input` to be of a type that matches what our mutation is doing. Let's create `PokemonColorInputType.js` in our `/types` folder:

```
import { GraphQLNonNull, GraphQLInputObjectType, GraphQLString } from "graphql";

const PokemonColorInputType = new GraphQLInputObjectType({
  name: "PokemonColorInput",
  description: "Represents a Pokémon input for mutation updatePokemonColor",
  fields: () => ({
    number: {
      type: GraphQLString,
      description: "The identifier of this Pokémon",
    },
    name: {
      type: GraphQLString,
      description: "The name of this Pokémon",
    },
    color: {
      type: new GraphQLNonNull(GraphQLString),
      description: "The color of this Pokémon",
    },
  }),
});

export default PokemonColorInputType;
```

We require the `color` argument because we can't do anything with our mutation without it, whereas `name` and `id` are optional because we don't know which one the user will pick. So back to our `MutationType.js` file, we'll import our new type and use it:

```
import { GraphQLObjectType, GraphQLNonNull } from "graphql";

import PokemonColorInputType from "./PokemonColorInputType";
import PokemonType from "./PokemonType";

const MutationType = new GraphQLObjectType({
  name: "Mutation",
  description: "Change info for any Pokémon by number or name",
  fields: () => ({
    mutation: {
      type: MutationType,
      resolve: (_, ...args) => args,
    },
    updatePokemonColor: {
	  type: PokemonType,
	  description: "Add or update the color of a Pokemon",
      args: {
        input: {
          type: new GraphQLNonNull(PokemonColorInputType),
        },
      },
      resolve: async (_, args) => {},
    },
  }),
});

export default MutationType;
```

Now we have to write our resolver. This time our resolver won't just be fetching data, it will be changing it. Here we'll write the resolver inline, but if we wanted we could make a function in our `service/Pokemon.js` file if we wanted.

```
import { GraphQLObjectType, GraphQLNonNull } from "graphql";
import { fromGlobalId } from "graphql-relay";

import PokemonColorInputType from "./PokemonColorInputType";
import PokemonType from "./PokemonType";

import { getPokemonById, getPokemonByName } from "../service/Pokemon";

const MutationType = new GraphQLObjectType({
  name: "Mutation",
  description: "Change info for any Pokémon by number or name",
  fields: () => ({
    mutation: {
      type: MutationType,
      resolve: (_, ...args) => args,
    },
    updatePokemonColor: {
      type: PokemonType,
      description: "Add or update the color of a Pokemon",
      args: {
        input: {
          type: new GraphQLNonNull(PokemonColorInputType),
        },
      },
      resolve: async (_, { input }) => {
        // destructure our arguments
        const { id, name, color } = input;
        let pokemon;

        // if we don't have an id or name we can't find the Pokemon
        if (!id && !name) {
          throw new Error(
            "You need to specify either the ID or name of the Pokémon"
          );
        }

        // fetch the Pokemon
        if (id) {
          pokemon = await getPokemonById(fromGlobalId(id).id);
        } else if (name) {
          pokemon = await getPokemonByName(name);
        }

        // add or update the property in the object
        pokemon.color = color;

        return pokemon;
      },
    },
  }),
});

export default MutationType;

```

Now we have to add the `MutationType` to the schema back in `schema.js`:

```
import { GraphQLSchema } from "graphql";

import MutationType from "./type/MutationType";
import QueryType from "./type/QueryType";

const schema = new GraphQLSchema({
  query: QueryType,
  mutation: MutationType,
});

export default schema;
```

And run our build script `npm run build-schema`. Now let's try mutating the color of a Pokemon. In GraphiQL we'll write:

```
mutation changePokemonColor($input: PokemonColorInput!) {
  updatePokemonColor(input: $input) {
    name
    color
  }
}
```

with the query variables

```
{
  "input": {
    "name": "Pikachu",
    "color": "yellow"
  }
}
```

But it doesn't work. Why? We've forgotten to update the `PokemonType` that already existed with our new field. In `types/PokemonType.js` let's add one more field:

```
color: {
  type: GraphQLString,
  description: "The color of this Pokémon",
  resolve: (pokemon) => pokemon.color,
},
```

We want to make sure it's nullable because the field doesn't exist on any of our Pokemon yet (something to consider when adding new fields to your GraphQL schema).  If we run the build schema script and try the mutation again now, it works!

```
{
  "data": {
    "updatePokemonColor": {
      "id": "UG9rZW1vbjowMjU=",
      "name": "Pikachu",
      "color": "yellow"
    }
  }
}
```

### Add a new Enum type

[Enums](https://blog.logrocket.com/what-you-need-to-know-about-graphql-enums/), or "enumeration", types allow you to define a restricted set of values that are expected in a field (letting the client know via the type system they can always expect one of a limited set of values to be returned for the field) or argument (letting the client know they can only pass one of these values as an argument to a query or mutation).

For example, let's look our newly written `getPokemonsByType` query. What if we passed the `type` of "Dog"?

```
query {
  getPokemonsByType(type: "Dog") {
    name
    types
    weaknesses
    resistant
  }
}
```

We wouldn't get any data back:

```
{
  "data": {
    "getPokemonsByType": []
  }
}
```

This isn't great because maybe the client thinks there just aren't any Pokemon we've categorised under "Dog", when in reality the type "Dog" doesn't exist. If we had an enum type for this field, instead of a `GraphQLString`, then we would be able to validate the query input and in our schema, the client would also be able to see every type of `type` they could query.

In our `/types` folder let's create a file called `ElementalEnumType.js`:

```
import { GraphQLEnumType } from "graphql";

const ElementalEnumType = new GraphQLEnumType({
  name: "ElementalEnum",
  description: "All possible elemental types",
  values: {
    BUG: { value: "Bug" },
    DARK: { value: "Dark" },
    DRAGON: { value: "Dragon" },
    ELECTRIC: { value: "Electric" },
    FAIRY: { value: "Fairy" },
    FIGHTING: { value: "Fighting" },
    FIRE: { value: "Fire" },
    FLYING: { value: "Flying" },
    GHOST: { value: "Ghost" },
    GRASS: { value: "Grass" },
    GROUND: { value: "Ground" },
    ICE: { value: "Ice" },
    NORMAL: { value: "Normal" },
    POISON: { value: "Poison" },
    PSYCHIC: { value: "Psychic" },
    ROCK: { value: "Rock" },
    STEEL: { value: "Steel" },
    WATER: { value: "Water" },
  },
});

export default ElementalEnumType;
```

GraphQL gives us the `new GraphQLEnumType` constructor where we add the usual `name` and `description`, but this time instead of a resolver or `type` field, we'll add `values`, which are the possible values of the enum. For the key-value pairs of each item in the enum, the key must be written exactly how it appears in our data. Enums are always written in all caps. But in our data, the types are written "Fire" not "FIRE", and we have to match the value of the enum to how the data actually looks. So in the value object for each enum, we add how the data actually looks. If we were using a SQL database and the data we were fetching was from a column of type ENUM then we wouldn't need to specify these alternative values, and could just write `DRAGON: { }`.

Make sure to import the `ElementalEnumType` file into `PokemonType.js` and we'll update the fields `types`, `resistant`, and `weaknesses` to have a type of `new GraphQLList(ElementalEnumType)` instead of `new GraphQLList(GraphQLString)`.

Then in `QueryType.js` we want to import our enum and update the `args.type` type to `new GraphQLNonNull(ElementalEnumType)`, like so:

```
    getPokemonsByType: {
      type: new GraphQLList(PokemonType),
      args: {
        type: {
          type: new GraphQLNonNull(ElementalEnumType),
        },
      },
      resolve: async (_, { type }) => {
        return await getPokemonsByType(type);
      },
    },
```

Now if we run our build schema script and go back to the IDE, we can try to query for type "Dog" again. We receive back the error message _'Argument \"type\" has invalid value \"Dog\".\nExpected type \"ElementalEnum\", found \"Dog\".'_ If we look in the docs, we can see that the query has been updated to define the new expected arguments: `getPokemonsByType(type: ElementalEnum!): [Pokemon]`. We can also click on `ElementalEnum` and it will list all the possible values.

It's important to note that enums are always written in all caps and without quotes (strings have quotes). So our query argument couldn't be updated to `getPokemonsByType(type: "DRAGON")` - it would have to be `getPokemonsByType(type: DRAGON)`. Then we get back the data we expect, and can see that enums are now returned as well:

```
{
  "data": {
    "getPokemonsByType": [
      {
        "name": "Dratini",
        "types": [
          "DRAGON"
        ],
        "weaknesses": [
          "ICE",
          "DRAGON",
          "FAIRY"
        ],
        "resistant": [
          "FIRE",
          "WATER",
          "ELECTRIC",
          "GRASS"
        ]
      },
      ...
    ]
  }
}
```

### Add a new filter on a query

Now let's say we want to add some filters to our queries. What is a filter? Take for example the [Countries API](https://countries-274616.ew.r.appspot.com/):

```
query getCountry {
  Country(name: "Germany") {
    name
    nameTranslations(first: 10, filter: {OR: [{languageCode_contains: "e"}, {languageCode_not_starts_with: "d"}]}) {
      languageCode
      value
    }
  }
}
```

The filters `languageCode_contains`, and `languageCode_not_starts_with` are provided by the library `neo4j-graphql-js` - in GraphQL APIs generally this is the case that the library connecting the database (SQL, MongoDB, Neo4J, etc) to the GraphQL API adds some filtering capabilities. Then you can use the operators `AND` and `OR` to combine the filters. So in the example above we want translations of "Germany" in languages whose language codes contain the letter "e" OR 
