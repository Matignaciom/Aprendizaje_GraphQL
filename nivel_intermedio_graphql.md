En el nivel intermedio de GraphQL, nos enfocaremos en conceptos más avanzados como validación, relaciones entre tipos, filtros y argumentos, manejo de errores, autenticación y autorización.
Aquí hay una guía detallada con ejemplos de código:

### 6. **Validación y Tipado:**

GraphQL realiza automáticamente la validación y el tipado de las consultas. Asegurémonos de que nuestro esquema sea robusto y tenga tipos definidos correctamente.

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Definir el esquema con tipos y validaciones
const typeDefs = gql`
  type Query {
    getUser(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
    age: Int!
  }
`;

const users = [
  { id: '1', name: 'John', age: 25 },
  { id: '2', name: 'Jane', age: 30 }
];

const resolvers = {
  Query: {
    getUser: (_, { id }) => users.find(user => user.id === id)
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL con errores:

```graphql
query {
  getUser {
    name
  }
}
```

Respuesta esperada con error:

```json
{
  "errors": [
    {
      "message": "Argument \"id\" of required type \"ID!\" was not provided.",
      "locations": [
        {
          "line": 2,
          "column": 11
        }
      ],
      "path": ["getUser"]
    }
  ],
  "data": null
}
```

### 7. **Relaciones entre Tipos:**

Ahora, exploremos cómo manejar relaciones entre tipos en el esquema.

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Definir el esquema con relación entre tipos
const typeDefs = gql`
  type Query {
    getUser(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
    age: Int!
    posts: [Post]
  }

  type Post {
    id: ID!
    title: String!
    content: String!
  }
`;

const users = [
  { id: '1', name: 'John', age: 25, posts: ['101'] },
  { id: '2', name: 'Jane', age: 30, posts: ['102', '103'] }
];

const posts = [
  { id: '101', title: 'GraphQL Basics', content: 'An introduction to GraphQL' },
  { id: '102', title: 'Advanced GraphQL', content: 'Deep dive into GraphQL concepts' },
  { id: '103', title: 'GraphQL Best Practices', content: 'Tips and tricks for GraphQL development' }
];

const resolvers = {
  Query: {
    getUser: (_, { id }) => users.find(user => user.id === id)
  },
  User: {
    posts: user => posts.filter(post => user.posts.includes(post.id))
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL para obtener un usuario con sus publicaciones:

```graphql
query {
  getUser(id: "1") {
    name
    posts {
      title
      content
    }
  }
}
```

Respuesta esperada:

```json
{
  "data": {
    "getUser": {
      "name": "John",
      "posts": [
        {
          "title": "GraphQL Basics",
          "content": "An introduction to GraphQL"
        }
      ]
    }
  }
}
```

### 8. **Filtros y Argumentos:**

Vamos a agregar argumentos a nuestras consultas para permitir filtros y ordenamientos.

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Definir el esquema con argumentos
const typeDefs = gql`
  type Query {
    getUsers(age: Int): [User]
  }

  type User {
    id: ID!
    name: String!
    age: Int!
  }
`;

const users = [
  { id: '1', name: 'John', age: 25 },
  { id: '2', name: 'Jane', age: 30 },
  { id: '3', name: 'Doe', age: 25 }
];

const resolvers = {
  Query: {
    getUsers: (_, { age }) => (age ? users.filter(user => user.age === age) : users)
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL con argumento para filtrar por edad:

```graphql
query {
  getUsers(age: 25) {
    name
  }
}
```

Respuesta esperada:

```json
{
  "data": {
    "getUsers": [
      {"name": "John"},
      {"name": "Doe"}
    ]
  }
}
```

### 9. **Manejo de Errores:**

Vamos a simular un escenario de error y manejarlo adecuadamente.

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Definir el esquema con manejo de errores
const typeDefs = gql`
  type Query {
    getUser(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
    age: Int!
  }
`;

const users = [
  { id: '1', name: 'John', age: 25 },
  { id: '2', name: 'Jane', age: 30 }
];

const resolvers = {
  Query: {
    getUser: (_, { id }) => {
      const user = users.find(user => user.id === id);
      if (!user) {
        throw new Error('Usuario no encontrado');
      }
      return user;
    }
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL para un usuario inexistente:

```graphql
query {
  getUser(id: "3") {
    name
  }
}
```

Respuesta esperada con error:

```json
{
  "errors": [
    {
      "message": "Usuario no encontrado",
      "locations": [
        {
          "line": 2,
          "column": 7
        }
      ],
      "path": ["getUser"]
    }
  ],
  "data": null
}
```

### 10. **Autenticación y Autorización:**

Añadamos autenticación y autorización a nuestro servidor GraphQL.

```javascript
const { GraphQLServer,

 gql } = require('graphql-yoga');

// Ejemplo básico de autenticación y autorización
const typeDefs = gql`
  type Query {
    currentUser: User
  }

  type User {
    id: ID!
    name: String!
    role: String!
  }
`;

const users = [
  { id: '1', name: 'John', role: 'admin' },
  { id: '2', name: 'Jane', role: 'user' }
];

const resolvers = {
  Query: {
    currentUser: (_, __, { currentUser }) => {
      if (!currentUser) {
        throw new Error('No autenticado');
      }
      return currentUser;
    }
  }
};

const server = new GraphQLServer({
  typeDefs,
  resolvers,
  context: req => ({
    currentUser: users.find(user => user.id === req.request.headers.userid)
  })
});

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL para obtener el usuario actual:

```graphql
query {
  currentUser {
    name
    role
  }
}
```

Respuesta esperada:

```json
{
  "data": {
    "currentUser": {
      "name": "John",
      "role": "admin"
    }
  }
}
```

Estos conceptos intermedios te permitirán construir APIs GraphQL más complejas y robustas.
