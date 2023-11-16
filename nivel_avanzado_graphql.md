En el nivel avanzado de GraphQL, exploraremos temas más avanzados como la optimización de consultas, el uso de suscripciones, la integración con bases de datos, clientes GraphQL en el lado del cliente, herramientas y middleware, seguridad y mejores prácticas para implementar GraphQL en producción.

### 11. **Optimización de Consultas:**

Optimizar consultas en GraphQL es crucial para mejorar el rendimiento. Utilizaremos fragmentos y paginación como ejemplos.

#### Uso de Fragmentos:

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Utilizando fragmentos para optimizar la consulta
const typeDefs = gql`
  type Query {
    getUser(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
    posts: [Post]
  }

  type Post {
    id: ID!
    title: String!
  }

  fragment PostDetails on Post {
    id
    title
  }
`;

const users = [
  { id: '1', name: 'John', posts: [{ id: '101', title: 'First Post' }] },
  { id: '2', name: 'Jane', posts: [] }
];

const resolvers = {
  Query: {
    getUser: (_, { id }) => users.find(user => user.id === id)
  },
  User: {
    posts: user => user.posts
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL con fragmento:

```graphql
query {
  getUser(id: "1") {
    name
    posts {
      ...PostDetails
    }
  }
}
```

#### Paginación:

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');

// Implementando paginación para obtener solo una cantidad específica de resultados
const typeDefs = gql`
  type Query {
    getPosts(page: Int, pageSize: Int): [Post]
  }

  type Post {
    id: ID!
    title: String!
  }
`;

const allPosts = Array.from({ length: 50 }, (_, index) => ({ id: `${index + 1}`, title: `Post ${index + 1}` }));

const resolvers = {
  Query: {
    getPosts: (_, { page = 1, pageSize = 10 }) => {
      const start = (page - 1) * pageSize;
      const end = start + pageSize;
      return allPosts.slice(start, end);
    }
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL con paginación:

```graphql
query {
  getPosts(page: 2, pageSize: 5) {
    id
    title
  }
}
```

### 12. **Subscriptions:**

Las suscripciones permiten que el servidor envíe datos en tiempo real al cliente cuando ocurren eventos. Usaremos `graphql-yoga` para implementar suscripciones.

```javascript
const { GraphQLServer, gql, PubSub } = require('graphql-yoga');

const typeDefs = gql`
  type Query {
    messages: [String]
  }

  type Mutation {
    postMessage(user: String, message: String): ID!
  }

  type Subscription {
    messages: [String]
  }
`;

const messages = [];
const pubsub = new PubSub();

const resolvers = {
  Query: {
    messages: () => messages
  },
  Mutation: {
    postMessage: (_, { user, message }) => {
      const id = messages.length.toString();
      messages.push(`${user}: ${message}`);
      pubsub.publish('messages', { messages });
      return id;
    }
  },
  Subscription: {
    messages: {
      subscribe: () => pubsub.asyncIterator('messages')
    }
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(({ port }) => {
  console.log(`Servidor GraphQL iniciado en http://localhost:${port}`);
});
```

Consulta GraphQL con suscripción:

```graphql
subscription {
  messages
}
```

### 13. **Integración con Bases de Datos:**

Vamos a conectar nuestro servidor GraphQL a una base de datos. En este ejemplo, usaremos MongoDB y el paquete `mongoose`.

```bash
# Instala mongoose
npm install mongoose
```

```javascript
const { GraphQLServer, gql } = require('graphql-yoga');
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/graphql_example', { useNewUrlParser: true });

const Post = mongoose.model('Post', {
  title: String,
  content: String
});

const typeDefs = gql`
  type Query {
    getPosts: [Post]
  }

  type Post {
    id: ID!
    title: String!
    content: String!
  }
`;

const resolvers = {
  Query: {
    getPosts: async () => await Post.find()
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Consulta GraphQL para obtener todos los posts:

```graphql
query {
  getPosts {
    id
    title
    content
  }
}
```

### 14. **Apollo Client (o Cliente GraphQL de tu elección):**

Vamos a utilizar Apollo Client para realizar consultas GraphQL desde el lado del cliente.

```bash
# Instala apollo-client y apollo-link-http
npm install @apollo/client apollo-link-http
```

```javascript
// index.js (lado del cliente)
import { ApolloClient, InMemoryCache, ApolloProvider, useQuery, gql } from '@apollo/client';
import React from 'react';
import ReactDOM from 'react-dom';

const client = new ApolloClient({
  uri: 'http://localhost:4000', // URL del servidor GraphQL
  cache: new InMemoryCache()
});

const GET_POSTS = gql`
  query {
    getPosts {
      id
      title
      content
    }
  }
`;

const App = () => {
  const { loading, error, data } = useQuery(GET_POSTS);

  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>Posts</h1>
      {data.getPosts.map(post => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
};

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById('root')
);
```

### 15. **Herramientas y Middleware:**

Exploraremos herramientas y middleware comunes utilizados con GraphQL.

#### Apollo Server y GraphQL Playground:

```javascript
const { ApolloServer, gql } = require('apollo

-server');

const typeDefs = gql`
  type Query {
    hello: String
  }
`;

const resolvers = {
  Query: {
    hello: () => '¡Hola, GraphQL!'
  }
};

const server = new ApolloServer({ typeDefs, resolvers, introspection: true, playground: true });

server.listen().then(({ url }) => {
  console.log(`Servidor GraphQL iniciado en ${url}`);
});
```

#### GraphQL Middleware (Ejemplo con Express):

```javascript
const express = require('express');
const { ApolloServer, gql } = require('apollo-server-express');

const typeDefs = gql`
  type Query {
    hello: String
  }
`;

const resolvers = {
  Query: {
    hello: () => '¡Hola, GraphQL!'
  }
};

const server = new ApolloServer({ typeDefs, resolvers });

const app = express();
server.applyMiddleware({ app });

app.listen({ port: 4000 }, () =>
  console.log(`Servidor GraphQL iniciado en http://localhost:4000${server.graphqlPath}`)
);
```

### 16. **Seguridad:**

Asegurar tu servidor GraphQL es esencial. Aquí hay algunas consideraciones de seguridad:

- **Validación de Entrada:** Utiliza herramientas como `express-validator` para validar y sanitizar datos de entrada.
- **Autenticación y Autorización:** Implementa un sistema robusto de autenticación y autorización.
- **Limitación de Consultas:** Usa herramientas como `graphql-depth-limit` para evitar consultas profundas y costosas.
- **Seguridad en Producción:** Configura encabezados de seguridad como `helmet` y considera el uso de herramientas como `express-rate-limit` para proteger contra ataques de denegación de servicio.

### 17. **GraphQL en Producción:**

Algunas consideraciones para implementar GraphQL en producción:

- **Escalabilidad:** Asegúrate de que tu servidor sea escalable y pueda manejar el aumento de tráfico.
- **Monitoreo:** Implementa herramientas de monitoreo como `Apollo Engine` o `Prometheus` para supervisar el rendimiento de tu servidor GraphQL.
- **Caching:** Usa estrategias de caché eficientes, ya sea a nivel de servidor (por ejemplo, `apollo-server` con `apollo-datasource`) o a nivel de cliente (por ejemplo, Apollo Client con `ApolloLink`).
