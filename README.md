# Set up Graphql Node Server
<div align="center">
<img src="https://uvn8ig.am.files.1drv.com/y4mwu_wfrCp2wTn8NHUh7DMjG5DyR8Ch6vvlamnRtwGzp5gwoCyTsd-PSpU-Klkk6bEarxLT6djH9Z-uvNVgTOd78i5Hdxi_eLNkLLxs1Cd2xiDeXHxerA890VqW4Wglik5bh24TmwxcyU7emnPdaLLOkVbOCd_DPcjQGFMe4dkH7w8T96cnv2G3a6TEkKevWh_BxXt_sB-J7yfqFbFq9p6lA?width=660&height=440&cropmode=none" width="660" height="440" />
</div>

1. In index.js/server.js add an end point for graphql

```javascript
const graphqlHTTP = require('express-graphql');
const schema = require('./schema/schema');

// bind express with graphql
app.use('/graphql', graphqlHTTP({
    schema,
    graphiql: true
}));

```
## Create some Models in MongoDB

### Book

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const bookSchema = new Schema({
    name: String,
    genre: String,
    authorId: String
});

module.exports = mongoose.model('Book', bookSchema);

```

### Author

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const authorSchema = new Schema({
    name: String,
    age: Number
});

module.exports = mongoose.model('Author', authorSchema);

```



## GraphQLObjectType


### Create a schema folder

> In the schema folder we create GraphQLObjectType's which contain

####  > name

#### > fields : we describe to graphql what a book looks like by describing the model schema 

```javascript
const graphql = require('graphql');
const Book = require('../to/your/model');

// create a model type e.g book / author

const BookType = new GraphQLObjectType({
    name: 'Book',
    fields: ( ) => ({
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        genre: { type: GraphQLString },
        author: {
            type: AuthorType,
            resolve(parent, args){
                return Author.findById(parent.authorId);
            }
        }
    })
});

const AuthorType = new GraphQLObjectType({
    name: 'Author',
    fields: ( ) => ({
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        age: { type: GraphQLInt },
        books: {
            type: new GraphQLList(BookType),
            resolve(parent, args){
                return Book.find({ authorId: parent.id });
            }
        }
    })
});

```
## RootQuery
Each schema must contain a root query on for example how to get a book author or say a list of books or authors.

```javascript

const RootQuery = new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
        book: {
            type: BookType,
            args: { id: { type: GraphQLID } },
            resolve(parent, args){
                return Book.findById(args.id);
            }
        },
        author: {
            type: AuthorType,
            args: { id: { type: GraphQLID } },
            resolve(parent, args){
                return Author.findById(args.id);
            }
        },
        books: {
            type: new GraphQLList(BookType),
            resolve(parent, args){
                return Book.find({});
            }
        },
        authors: {
            type: new GraphQLList(AuthorType),
            resolve(parent, args){
                return Author.find({});
            }
        }
    }
});

```
## Mutations

To change or add data in the database we must create a Mutation examples below...

```javascript
const Mutation = new GraphQLObjectType({
    name: 'Mutation',
    fields: {
        addAuthor: {
            type: AuthorType,
            args: {
                name: { type: GraphQLString },
                age: { type: GraphQLInt }
            },
            resolve(parent, args){
                let author = new Author({
                    name: args.name,
                    age: args.age
                });
                return author.save();
            }
        },
        addBook: {
            type: BookType,
            args: {
                name: { type: new GraphQLNonNull(GraphQLString) },
                genre: { type: new GraphQLNonNull(GraphQLString) },
                authorId: { type: new GraphQLNonNull(GraphQLID) }
            },
            resolve(parent, args){
                let book = new Book({
                    name: args.name,
                    genre: args.genre,
                    authorId: args.authorId
                });
                return book.save();
            }
        }
    }
});

```

## Types

There are many types you can use when creating these qraphql schemas. [Link graphql/types](https://graphql.org/graphql-js/type/)

> Common ones used

```javascript
const {
    GraphQLObjectType,
    GraphQLString,
    GraphQLSchema,
    GraphQLID,
    GraphQLInt,
    GraphQLList,
    GraphQLNonNull
} = graphql;

```

# Finally

At the end of your qraphql Schema expose those methods

```javascript
module.exports = new GraphQLSchema({
    query: RootQuery,
    mutation: Mutation
});

```
Acknowledgement and Adapted from tutorial by
  [The Net Ninja](https://www.thenetninja.co.uk/)


