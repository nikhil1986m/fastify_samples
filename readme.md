# Fastify Day -3 workshop 

 1. ```npm run init -y```
 2. ```npm install fastify```
 3. ```npm install -D typescript @types/node ts-node```
 4. ```npx tsc --init``` #initialize typescript with basic config.
 5. Update tsconfig.json ensuring latest target to es version (EsNext)

```
{
  "compilerOptions": 
  {
    "target": "ESNext",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "rootDir": "./src",
    "outDir": "./build",
  }
}
```
6. Install NodeMon for setup watch/testing continuosly ( You can setup locally or gloabally ) For more details: https://www.npmjs.com/package/nodemon 
    - ```npm install -g nodemon``` #globally
    - ```npm install --save-dev nodemon``` #locally

7. Update scripts to setup watch 
    <br>
     > **ⓘ Note:** <br>
     Keeps wathcing the typescript chnages and nodemon keeps on comiling and starting restart the server from index.js
     <br>
    - ```tsc -p tsconfig.json -w```    #build script
    <br>
    > **ⓘ Note:** 
    <br>Keeps building the typescript chnages modify index.js
    - ``` tsc -w & nodemon index.js ``` #start script
8. Let's test by creating a ts file and test the  basic setup
    - touch src/0_setup.ts
    - console.log('Day 3 - fastify lab');
9. Run command ```node build/0_setup.js```

## Lab 01 - Setup Fastify App
```
import Fastify from "fastify";

const Port = Number(process.env.PORT) || 5001;


const server = Fastify(
    {
        logger: true
    }
);



server.get("/", async (request, reply) => {
    return { lab1: "node JS session with fastify" }
});






const start = async () => {
    await server.listen({ port: Port }, (err, _address) => {
        if (err) {
            server.log.error(err);
            process.exit(1);
        }
    })
};

start();

```

## Lab02 - Setup Basic Routing

- Create Index.ts

 ```
 import buildServer from './server'

const fastify = buildServer();

const start = async function () {
    try {
        await fastify.listen({ port: 3000 })
    } catch (err) {
        fastify.log.error(err)
        process.exit(1)
    }
}

start();
```

- Create server.ts
```

import Fastify from 'fastify'

import { apiRouter } from './routes/api.router'

import { appRouter } from './routes/app.router';

function buildServer() {
    const fastify = Fastify({
        logger: true
    })

    fastify.register(apiRouter);
    fastify.register(appRouter);

    return fastify;
}

export default buildServer;

```

- Create a folder named routes

  ![alt text](./images/image_l2.png)

  - Create a file named ```api.router.ts```
  ```
  export async function apiRouter(fastify: any) {

    fastify.get('/api/v1/users', {}, async () =>
        [
            { name: 'alice' },
            { name: 'bob' },
        ]);
    }

  ```
  
  - Create a file named ```app.router.ts```
  ```
  export async function appRouter(fastify: any) {
    fastify.get('/', {}, async () =>
        "Welcome to the Fastify API");
  }

  ```

- Run the app

  ``` npm run build ```

  ``` node build/index.js ``` 



## Lab03 - Setup Router Versioning


- Create Index.ts

 ```
 import buildServer from './server'

const fastify = buildServer();

const start = async function () {
    try {
        await fastify.listen({ port: 3000 })
    } catch (err) {
        fastify.log.error(err)
        process.exit(1)
    }
}

start();
```

- Create server.ts
```

iimport Fastify from 'fastify'

import { appRouter } from './routes/app.router';

import appRouterV1 from './routes/api/v1.router';

import appRouterV2 from './routes/api/v2.router';



function buildServer() {
    const fastify = Fastify({
        logger: true
    })

    fastify.register(appRouterV1, { prefix: '/api/v1' });

    fastify.register(appRouterV2, { prefix: '/api/v2' });
    
    fastify.register(appRouter);

    return fastify;
}

export default buildServer;

```

- Create a folder named ```routes/api```

  ![alt text](./images/image_l3.png)

  <br/>

  - Create a file named ```app.router.ts```
  ```
  
  export async function appRouter(fastify: any) {
    fastify.get('/', {}, async () =>
        "Welcome to the Fastify API");


    //// script to test via cUrl +ve tests
    //// curl -X GET -H "Content-Type: application/json" -H "Accept-Version: 1.0.0" http://localhost:3000/teams
    //// script to test via cUrl -ve tests
    //// curl -X GET -H "Content-Type: application/json" -H "Accept-Version: 2.0.0" http://localhost:3000/teams

    fastify.route({
        method: 'GET',
        url: '/teams',
        constraints: { version: '1.0.0' },
        handler: async (_req: any, _reply: any) => {
            return [
                { name: 'India' },
                { name: 'Australia' },
            ];
        },
    });
  }


  ```


  - Create a file named ```v1.router.ts``` under ```api``` folder

   ```
   export default async function apiRouterV1(fastify: any) {
    fastify.get('/users', {}, async () =>
        [
            { name: 'alice' },
            { name: 'bob' },
        ]);
    }
   ```

  
  - Create a file named ```v2.router.ts``` under ```api``` folder

   ```
   export default async function apiRouterV2(fastify: any) {
    fastify.get('/users', {}, async () =>
        [
            { name: 'cody' },
            { name: 'den' },
        ]);
    }
   ```

- Run the app

  ``` npm run build ```

  ``` node build/index.js ``` 


## Lab04 - Unit testing with fastify

 - Update ```package.json``` with ```test``` command
   ```
    "scripts": {
     ...

      "test": "node --test",
     
     ...
    }
   ```
 - Create a file named ```index.tests.ts``` under ```tests```
 ```

import { test } from "node:test"
import assert from "node:assert"
import buildServer from "../server"


test('GET /api/v1/users', async t => {
    await t.test('returns users', async t => {
        const fastify = buildServer()
        const res = await fastify.inject('/api/v1/users')
        assert.equal(res.statusCode, 200)

        assert.equal(JSON.stringify(res.json()), JSON.stringify([
            { name: 'alice' },
            { name: 'bob' },
        ]));
    })
})

test('GET /', async t => {
    await t.test('returns app home', async t => {
        const fastify = buildServer()
        const res = await fastify.inject('/')
        assert.equal(res.statusCode, 200)
        console.log(res.body);
        assert.deepStrictEqual(res.json(),
            { welcomeMessage: "Welcome to the Fastify API" });
    });
})

```


## Lab05 - Decorators

- Create ```authentication.decorator.ts``` under ```decorators``` folder

```

export async function authenticationDecorator(fastify: any) {
    fastify.decorate('authenticate', (name: string) => {
        return `authentication is successful. Hello, ${name}!! `;
    });
}

```
- Register Decorator on the fastify server

 ```

 import Fastify from 'fastify'
import { apiRouter } from './routes/api.router'
import { appRouter } from './routes/app.router';
import { authenticationDecorator } from './decorators/authentication.decorator';

function buildServer() {
    const fastify = Fastify({
        logger: true
    })


    //// Register Authentication Decorator
    authenticationDecorator(fastify);
    
    
    
    
    fastify.register(apiRouter);
    fastify.register(appRouter);

    return fastify;
}

export default buildServer;

 ```

- Update the ```app.router.ts``` to use the ```authenticate``` decorator

```

import { FastifyReply, FastifyRequest } from "fastify"

export async function appRouter(fastify: any) {
    fastify.get('/', async (_req: FastifyRequest, reply: FastifyReply) => {
    
        //// Use authenticate decorator before sendig reply
        const result = fastify.authenticate("alice");
    
    
        return reply.send({ welcomeMessage: result });
    });
}


```

- Run the app

  ``` npm run build ```

  ``` node build/index.js ``` 


## Lab06 - Hooks

- Create file named ```request.hooks.ts``` under ```hooks```

```

import { FastifyRequest, FastifyReply, HookHandlerDoneFunction } from 'fastify';

export const onRequestHook = (request: FastifyRequest, _reply: FastifyReply, done: HookHandlerDoneFunction) => {
    request.log.info(`Received request: ${request.method} ${request.url}`);
    done();
};

export const preHandlerHook = (request: any, reply: FastifyReply, done: HookHandlerDoneFunction) => {
    // Check if the request has a header parameter `trace-id`
    if (!request.headers['trace-id'] as unknown as string) {
        return reply.status(400).send({ error: 'trace-id is required before displaying the result' });
    }
    done();
};


```

- Register hooks in the ```app.router.ts``` routes
 ```
  import { FastifyReply, FastifyRequest } from "fastify"
  import { onRequestHook, preHandlerHook } from "../hooks/request.hooks";

  export async function appRouter(fastify: any) {
      fastify.get('/', async (_req: FastifyRequest, reply: FastifyReply) => {
          const result = "Welcome to the Fastify API";
          return reply.send({ welcomeMessage: result });
      });

      //// script to test via cUrl +ve tests
      //// curl - X GET - H "Content-Type: application/json" - H "Accept-Version: 1.0.0" - H "trace-id: xf8u7-7868-8987-9908" http://localhost:3000/teams
      //// script to test via cUrl -ve tests
      //// curl - X GET - H "Content-Type: application/json" - H "Accept-Version: 1.0.0" http://localhost:3000/teams


      fastify.route({
          method: 'GET',
          url: '/teams',
          
          //// Register request hooks 
          onRequest: onRequestHook,

          //// Register prehandler hooks 
          preHandler: preHandlerHook,
          
          
          handler: async (_req: any, _reply: any) => {
              return [
                  { name: 'India' },
                  { name: 'Australia' },
              ];
          },
      });
  }

    
  ```

  ## Lab07 - Plugins & Miscellaneous

  - Fastify Session - Plugin
      - ```npm install @fastify/session```
    
    ```
      // server.js
      import Fastify from "fastify";
      import fastifySession from '@fastify/session';
      import fastifyCookie from '@fastify/cookie';

      const server = Fastify({ logger: true });


      server.register(fastifyCookie);
      server.register(fastifySession, {
          secret: 'the-secret-identifier-key-of-the-application', // Change this to a more secure secret
          cookie: {
              secure: false, // Set to true in production with HTTPS
              httpOnly: true,
              maxAge: 1000 * 60 * 60 * 24 // 1 day
          }
      });

      // Route to set a session variable
      server.get('/setcount', (request: any, reply) => {

          if (!request.session.count) {
              request.session.count = 1;
          }
          reply.send({ message: `Session variable count set to ${request.session.count} !!` });
      });

      // Route to get session variable
      server.get('/getcount', (request: any, reply) => {
          const count = request.session.count++;
          reply.send({ counter: count });
      });

      // Route to destroy the session
      server.get('/logout', (request: any, reply) => {
          request.session.delete(); // Clear the session
          reply.send({ message: 'Logged out!' });
      });


      // Start the server
      const start = async () => {
          try {
              await server.listen({ port: 3000 });
              server.log.info(`Server listening on http://localhost:3000`);
          } catch (err) {
              server.log.error(err);
              process.exit(1);
          }
      };

      start();

    ```

    - Fastify/compress (Plugin)
      - ```npm install @fastify/compress```

      ```
      import fastifyCompress from '@fastify/compress';
      import fastify from 'fastify';
      import zlib from 'zlib';

      const app = fastify({ logger: true });

      app.register(fastifyCompress, {
            global: true,
            threshold: 5120,
            encodings: ['br', 'gzip'],
            brotliOptions: {
                params: {
                    [zlib.constants.BROTLI_PARAM_MODE]: zlib.constants.BROTLI_MODE_GENERIC,
                    [zlib.constants.BROTLI_PARAM_QUALITY]: 6
                }
            },
            zlibOptions: {
                level: 6
            }
        });


        // User interface
        interface User {
            id: number;
            name: string;
        }

        // In-memory "database"
        const users: User[] = [];

        app.get<{ Reply: User[] | { message: string } }>('/users', async (request, reply) => {
            return reply.send(users);
        });


        app.post<{ Body: User[]; Reply: User[] }>('/bulk-users', async (request, reply) => {
            const userList: User[] = request.body;
            console.log(userList);
            let currentId: number = users.length ? Math.max(...users.map(user => user.id)) : 0;
            userList.forEach(element => {
                currentId += 1; // Increment the ID for each new user
                element.id = currentId; // Assign the new ID
            });

            console.log(JSON.stringify(userList));
            users.push(...userList);
            return reply.status(201).send(userList);
        });

        //Request body: { "id":  1,  "name": "nikhil" }
        // Create User
        app.post<{ Body: User; Reply: User }>('/users', async (request, reply) => {
            const user: User = { ...request.body, id: users.length + 1 };
            users.push(user);
            return reply.status(201).send(user);
        });

        // Read User by ID
        app.get<{ Params: { id: number }; Reply: User | { message: string } }>('/users/:id', async (request, reply) => {
            const userId = Number(request.params.id);
            console.log(userId);
            const user = users.find(u => u.id == userId);
            if (!user) {
                return reply.status(404).send({ message: 'User not found' });
            }
            return reply.send(user);
        });

        // Update User
        app.put<{ Params: { id: number }; Body: Partial<User>; Reply: User | { message: string } }>('/users/:id', async (request, reply) => {
            const userIndex = users.findIndex(u => u.id === request.params.id);
            if (userIndex === -1) {
                return reply.status(404).send({ message: 'User not found' });
            }
            users[userIndex] = { ...users[userIndex], ...request.body };
            return reply.send(users[userIndex]);
        });

        // Delete User
        app.delete<{ Params: { id: number }; Reply: { message: string } }>('/users/:id', async (request, reply) => {
            const userIndex = users.findIndex(u => u.id === request.params.id);
            if (userIndex === -1) {
                return reply.status(404).send({ message: 'User not found' });
            }
            users.splice(userIndex, 1);
            return reply.send({ message: 'User deleted' });
        });

        // Start the server
        app.listen({ port: 3000 }, err => {
            if (err) {
                console.error(err);
                process.exit(1);
            }
            console.log('Server is running on http://localhost:3000');
        });



      ```

 - Fastify with Generics

   ```

    import fastify from 'fastify';

    const app = fastify({ logger: true });

    // User interface
    interface User {
        id: number;
        name: string;
    }

    // In-memory "database"
    const users: User[] = [];

    app.get<{ Reply: User[] | { message: string } }>('/users', async (request, reply) => {
        return reply.send(users);
    });


    app.post<{ Body: User[]; Reply: User[] }>('/bulk-users', async (request, reply) => {
        const userList: User[] = request.body;
        console.log(userList);
        let currentId: number = users.length ? Math.max(...users.map(user => user.id)) : 0;
        userList.forEach(element => {
            currentId += 1; // Increment the ID for each new user
            element.id = currentId; // Assign the new ID
        });

        console.log(JSON.stringify(userList));
        users.push(...userList);
        return reply.status(201).send(userList);
    });

    //Request body: { "id":  1,  "name": "nikhil" }
    // Create User
    app.post<{ Body: User; Reply: User }>('/users', async (request, reply) => {
        const user: User = { ...request.body, id: users.length + 1 };
        users.push(user);
        return reply.status(201).send(user);
    });

    // Read User by ID
    app.get<{ Params: { id: number }; Reply: User | { message: string } }>('/users/:id', async (request, reply) => {
        const userId = Number(request.params.id);
        console.log(userId);
        const user = users.find(u => u.id == userId);
        if (!user) {
            return reply.status(404).send({ message: 'User not found' });
        }
        return reply.send(user);
    });

    // Update User
    app.put<{ Params: { id: number }; Body: Partial<User>; Reply: User | { message: string } }>('/users/:id', async (request, reply) => {
        const userIndex = users.findIndex(u => u.id === request.params.id);
        if (userIndex === -1) {
            return reply.status(404).send({ message: 'User not found' });
        }
        users[userIndex] = { ...users[userIndex], ...request.body };
        return reply.send(users[userIndex]);
    });

    // Delete User
    app.delete<{ Params: { id: number }; Reply: { message: string } }>('/users/:id', async (request, reply) => {
        const userIndex = users.findIndex(u => u.id === request.params.id);
        if (userIndex === -1) {
            return reply.status(404).send({ message: 'User not found' });
        }
        users.splice(userIndex, 1);
        return reply.send({ message: 'User deleted' });
    });

    // Start the server
    app.listen({ port: 3000 }, err => {
        if (err) {
            console.error(err);
            process.exit(1);
        }
        console.log('Server is running on http://localhost:3000');
    });

   ```