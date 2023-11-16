Desarrollar proyectos prácticos es una excelente manera de consolidar tus conocimientos de GraphQL.

Aquí te proporcionaré un ejemplo de proyecto práctico paso a paso: una aplicación simple de blog con GraphQL.
Este proyecto incluirá la creación de un servidor GraphQL, la conexión a una base de datos, la implementación de consultas y mutaciones, y finalmente, la construcción de un cliente GraphQL para interactuar con el servidor.

### **Proyecto Práctico: Aplicación de Blog con GraphQL**

#### **1. Configuración del Servidor GraphQL:**

Primero, crea un nuevo proyecto de Node.js e instala las dependencias necesarias:

```bash
mkdir blog-app
cd blog-app
npm init -y
npm install express graphql express-graphql mongoose
```

Luego, crea un archivo `server.js` para el servidor GraphQL:

```javascript
// server.js

const express = require('express');
const { graphqlHTTP } = require('express-graphql');
const mongoose = require('mongoose');
const { buildSchema } = require('graphql');

// Conectar a la base de datos MongoDB
mongoose.connect('mongodb://localhost:27017/blog', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'Error de conexión a MongoDB:'));
db.once('open', () => console.log('Conexión exitosa a MongoDB'));

// Definir el esquema GraphQL
const schema = buildSchema(`
  type Post {
    id: ID!
    title: String!
    content: String!
  }

  type Query {
    getPosts: [Post]
  }

  type Mutation {
    createPost(title: String!, content: String!): Post
  }
`);

// Definir resolvers
const resolvers = {
  getPosts: async () => {
    // Obtener todos los posts desde la base de datos
    return await Post.find();
  },
  createPost: async ({ title, content }) => {
    // Crear un nuevo post y guardarlo en la base de datos
    const post = new Post({ title, content });
    return await post.save();
  },
};

// Crear el modelo de Post
const Post = mongoose.model('Post', {
  title: String,
  content: String,
});

// Crear la aplicación Express y configurar el endpoint GraphQL
const app = express();
app.use('/graphql', graphqlHTTP({ schema, rootValue: resolvers, graphiql: true }));

// Iniciar el servidor en el puerto 4000
const PORT = 4000;
app.listen(PORT, () => console.log(`Servidor GraphQL iniciado en http://localhost:${PORT}/graphql`));
```

#### **2. Configuración del Cliente GraphQL:**

A continuación, crearemos un cliente simple para interactuar con nuestro servidor GraphQL. Puedes usar cualquier biblioteca GraphQL en el lado del cliente, pero en este ejemplo, utilizaremos `@apollo/client`:

```bash
npm install react react-dom @apollo/client graphql
```

Luego, crea un directorio `client` y dentro de él, crea un nuevo proyecto de React:

```bash
mkdir client
cd client
npx create-react-app .
```

Modifica `src/App.js` para incluir una interfaz de usuario simple para la aplicación de blog:

```javascript
// src/App.js

import React, { useState } from 'react';
import { useQuery, useMutation, gql } from '@apollo/client';

const GET_POSTS = gql`
  query {
    getPosts {
      id
      title
      content
    }
  }
`;

const CREATE_POST = gql`
  mutation CreatePost($title: String!, $content: String!) {
    createPost(title: $title, content: $content) {
      id
      title
      content
    }
  }
`;

function App() {
  const { loading, error, data } = useQuery(GET_POSTS);
  const [createPost] = useMutation(CREATE_POST);

  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const handleCreatePost = async () => {
    await createPost({ variables: { title, content }, refetchQueries: [{ query: GET_POSTS }] });
    setTitle('');
    setContent('');
  };

  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>Blog App</h1>
      <div>
        <h2>Crear Nuevo Post</h2>
        <input type="text" placeholder="Título" value={title} onChange={(e) => setTitle(e.target.value)} />
        <textarea placeholder="Contenido" value={content} onChange={(e) => setContent(e.target.value)} />
        <button onClick={handleCreatePost}>Crear Post</button>
      </div>
      <h2>Posts</h2>
      <ul>
        {data.getPosts.map((post) => (
          <li key={post.id}>
            <strong>{post.title}</strong>
            <p>{post.content}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

#### **3. Ejecutar la Aplicación:**

Ahora, ejecuta tanto el servidor GraphQL como el cliente React. Abre dos terminales y ejecuta los siguientes comandos:

```bash
# Terminal 1: Iniciar el servidor GraphQL
cd blog-app
node server.js
```

```bash
# Terminal 2: Iniciar el cliente React
cd blog-app/client
npm start
```

Visita [http://localhost:3000](http://localhost:3000) en tu navegador para interactuar con la aplicación de blog. Puedes crear nuevos posts y ver la lista de posts existentes.

Este proyecto práctico cubre la creación de un servidor GraphQL, la conexión a una base de datos, la implementación de consultas y mutaciones, y la construcción de un cliente GraphQL en el lado del cliente.
Puedes expandir este proyecto agregando características adicionales, como autenticación, comentarios en los posts, o cualquier otra funcionalidad que desees explorar. ¡Espero que encuentres útil este ejemplo!
