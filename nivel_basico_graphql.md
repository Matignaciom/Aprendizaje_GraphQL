En este nivel, nos centraremos en entender los fundamentos esenciales de GraphQL.

### 1. **Conceptos Fundamentales:**

GraphQL es un lenguaje de consulta para tu API y un entorno de ejecución del lado del servidor para ejecutar esas consultas con tus datos existentes. Aquí hay algunos términos clave:

- **Schema (Esquema):** Define la estructura de tus datos y las operaciones que se pueden realizar.
- **Query (Consulta):** Solicita datos al servidor.
- **Mutation (Mutación):** Modifica datos en el servidor.
- **Subscription (Suscripción):** Permite al servidor enviar datos en tiempo real al cliente cuando ocurren eventos.

### 2. **Instalación y Configuración:**

En este ejemplo, utilizaremos `graphql-yoga`, que es una biblioteca fácil de usar para comenzar rápidamente.

```bash
# Instala graphql-yoga
npm install graphql-yoga
```

### 3. **Creación de un Esquema Simple:**

Crea un archivo `index.js`:

```javascript
const { GraphQLServer } = require('graphql-yoga');

// Definir el esquema básico
const typeDefs = `
  type Query {
    hello: String!
  }
`;

// Implementar resolvers para manejar las consultas
const resolvers = {
  Query: {
    hello: () => '¡Hola, GraphQL!'
  }
};

// Configurar el servidor GraphQL
const server = new GraphQLServer({ typeDefs, resolvers });

// Iniciar el servidor en el puerto 4000
server.start(() => {
  console.log('Servidor GraphQL iniciado en http://localhost:4000');
});
```

Ejecuta tu servidor:

```bash
node index.js
```

Abre tu navegador y visita [http://localhost:4000](http://localhost:4000). Puedes probar la consulta `hello` en la interfaz de GraphQL Playground.

### 4. **Consultas Básicas:**

Ahora, veamos cómo realizar consultas básicas. Puedes usar la herramienta GraphQL Playground o cualquier cliente GraphQL para enviar consultas.

Consulta GraphQL:

```graphql
query {
  hello
}
```

Respuesta esperada:

```json
{
  "data": {
    "hello": "¡Hola, GraphQL!"
  }
}
```

### 5. **Mutaciones:**

Agreguemos una mutación para modificar datos:

```javascript
// Agregar una mutación al esquema
const typeDefs = `
  type Query {
    hello: String!
  }
  
  type Mutation {
    changeGreeting(newGreeting: String!): String!
  }
`;

// Implementar resolvers para la mutación
const resolvers = {
  Query: {
    hello: () => '¡Hola, GraphQL!'
  },
  Mutation: {
    changeGreeting: (_, { newGreeting }) => {
      // Puedes hacer algo con newGreeting, pero por ahora solo lo devolveremos
      return `Saludo cambiado a: ${newGreeting}`;
    }
  }
};
```

Consulta GraphQL para la mutación:

```graphql
mutation {
  changeGreeting(newGreeting: "¡Hola, Nuevo GraphQL!")
}
```

Respuesta esperada:

```json
{
  "data": {
    "changeGreeting": "Saludo cambiado a: ¡Hola, Nuevo GraphQL!"
  }
}
```
