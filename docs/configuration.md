---
prev: /getting-started.html
next: false
sidebar: auto
otherTopics: true
meta:
 - name: description
   content: Documentation over the server configuration. Ts.ED is built on top of Express and use TypeScript language.
 - name: keywords
   content: configuration ts.ed express typescript node.js javascript decorators mvc class models
---

# Configuration

@@ServerSettings@@ let you to configure quickly your server via decorator. This decorator take your configuration and merge it with the default server configuration.

The default configuration is as follow:
```json
{
  "rootDir": "path/to/root/project",
  "env": "development",
  "port": 8080,
  "debug": false,
  "httpsPort": 8000,
  "uploadDir": "${rootDir}/uploads",
  "mount": {
    "/rest": "${rootDir}/controllers/**/*.ts" // support ts with ts-node then fallback to js
  },
  "componentsScan": [
    "${rootDir}/middlewares/**/*.ts",
    "${rootDir}/services/**/*.ts",
    "${rootDir}/converters/**/*.ts"
  ],
  "routers": {
    "mergeParams": false,
    "strict": false,
    "caseSensitive": false
  }
}
```

You can customize your configuration as follow:

```typescript
// server.ts
import {ServerLoader, ServerSettings} from "@tsed/common";
import Path = require("path");

@ServerSettings({
   rootDir: Path.resolve(__dirname), //optional. By default it's equal to process.cwd()
   mount: {
     "/rest": "${rootDir}/controllers/current/**/*.js",
     "/rest/v1": [
        "${rootDir}/controllers/v1/users/*.js", 
        "${rootDir}/controllers/v1/groups/**/*.ts", // support ts entry
        "!${rootDir}/controllers/v1/groups/old/*.ts", // support ts entry
        MyController // support manual import
     ]
   }
})
export class Server extends ServerLoader {}

// app.ts
import {$log, ServerLoader} from "@tsed/common";
import {Server} from "./Server";

async function bootstrap() {
  try {
    $log.debug("Start server...");
    const server = await ServerLoader.bootstrap(Server);

    await server.listen();
    $log.debug("Server initialized");
  } catch (er) {
    $log.error(er);
  }
}

bootstrap();
```
> Ts.ED support [ts-node](https://github.com/TypeStrong/ts-node). Ts extension will be replaced by a Js extension if 
ts-node isn't the runtime.

## Options

* `rootDir` &lt;string&gt;: The root directory where you build run project. By default, it's equal to `process.cwd()`.
* `env` &lt;Env&gt;: The environment profile. By default the environment profile is equals to `NODE_ENV`.
* `port` &lt;string | number&gt;: Port number for the [HTTP.Server](https://nodejs.org/api/http.html#http_class_http_server).
* `httpsPort` &lt;string | number&gt;: Port number for the [HTTPs.Server](https://nodejs.org/api/https.html#https_class_https_server).
* `httpsOptions` &lt;[Https.ServerOptions](https://nodejs.org/api/tls.html#tls_tls_createserver_options_secureconnectionlistener))&gt;:
  * `key` &lt;string&gt; | &lt;string[]&gt; | [&lt;Buffer&gt;](https://nodejs.org/api/buffer.html#buffer_class_buffer) | &lt;Object[]&gt;: The private key of the server in PEM format. To support multiple keys using different algorithms an array can be provided either as a plain array of key strings or an array of objects in the format `{pem: key, passphrase: passphrase}`. This option is required for ciphers that make use of private keys.
  * `passphrase` &lt;string&gt; A string containing the passphrase for the private key or pfx.
  * `cert` &lt;string&gt; | &lt;string[]&gt; | [&lt;Buffer&gt;](https://nodejs.org/api/buffer.html#buffer_class_buffer) | [&lt;Buffer[]&gt;](https://nodejs.org/api/buffer.html#buffer_class_buffer): A string, Buffer, array of strings, or array of Buffers containing the certificate key of the server in PEM format. (Required)
  * `ca` &lt;string&gt; | &lt;string[]&gt; | [&lt;Buffer&gt;](https://nodejs.org/api/buffer.html#buffer_class_buffer) | [&lt;Buffer[]&gt;](https://nodejs.org/api/buffer.html#buffer_class_buffer): A string, Buffer, array of strings, or array of Buffers of trusted certificates in PEM format. If this is omitted several well known "root" CAs (like VeriSign) will be used. These are used to authorize connections.
* `uploadDir` &lt;string&gt;: The temporary directory to upload the documents. See more on [Upload file with Multer](/tutorials/multer.md).
* `mount` &lt;[IServerMountDirectories](/api/common/config/interfaces/IServerMountDirectories.md)&gt;: Mount all controllers under a directories to an endpoint.
* `componentsScan` &lt;string[]&gt;: List of directories to scan [Services](/docs/services.md), [Middlewares](/docs/middlewares.md) or [Converters](/docs/converters.md).
* `exclude` &lt;string[]&gt;: List of glob patterns. Exclude all files which matching with this list when ServerLoader scan all components with the `mount` or `scanComponents` options.
* `statics` &lt;[IServerMountDirectories](/api/common/config/interfaces/IServerMountdirectories.md)&gt;: Object to mount all directories under to his endpoints. See more on [Serve Static](/tutorials/serve-static-files.md).
* `swagger` &lt;Object&gt;: Object configure swagger. See more on [Swagger](/tutorials/swagger.md).
* `routers` &lt;object&gt;: Global configuration for the Express.Router. See express [documentation](http://expressjs.com/en/api.html#express.router).
* `validationModelStrict` &lt;boolean&gt;: Use a strict validation when a model is used by the converter. When a property is unknown, it throw a `BadRequest` (see [Converters](/docs/converters.md)). By default true.
* `logger` &lt;[ILoggerSettings](/api/common/config/interfaces/ILoggerSettings.md)&gt;: Logger configuration.
* `controllerScope` &lt;`request`|`singleton`&gt;: Configure the default scope of the controllers. Default: `singleton`. See [Scope](/docs/injection-scopes.md).
* `acceptMimes` &lt;string[]&gt;: Configure the mimes accepted by default by the server.
* `errors` &lt;[IErrorsSettings](/api/common/config/interfaces/IErrorsSettings.md)&gt;: Errors configuration (see [Throw Http exceptions](/tutorials/throw-http-exceptions.md)).

## HTTP & HTTPs server
### Change address

It's possible to change the HTTP and HTTPS server address as follows:

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   httpPort: "127.0.0.1:8081",
   httpsPort: "127.0.0.2:8082",
})
export class Server extends ServerLoader {}
```

### Random port

Random port assignment can be enable with the value `0`. The port assignment will be delegate to the OS.

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   httpPort: "127.0.0.1:0",
   httpsPort: "127.0.0.2:0",
})
export class Server extends ServerLoader {}
```

Or: 

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   httpPort: 0,
   httpsPort: 0,
})
export class Server extends ServerLoader {}
```

### Disable HTTP

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   httpPort: false
})
export class Server extends ServerLoader {}
```

### Disable HTTPS

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   httpsPort: false,
})
export class Server extends ServerLoader {}
```

### HTTPs configuration

You see the example [projet HTTPs](https://github.com/TypedProject/example-ts-express-decorator/tree/2.0.0/example-https)


## Logger
### Default logger

Default logger use by Ts.ED is [ts-log-debug](https://TypedProject.github.io/ts-log-debug/). 

 - [Configuration](https://TypedProject.github.io/ts-log-debug#/getting-started?id=installation),
 - [Customize appender (chanel)](https://TypedProject.github.io/ts-log-debug#/appenders/custom),
 - [Customize layout](https://TypedProject.github.io/ts-log-debug#/layouts/custom)

### Configuration

Some options is provided:

- `logger.level`: Change the default log level displayed in the terminal. Values: `debug`, `info`, `warn` or `error`. By default: `info`. 
- `logger.logRequest`: Log all incoming request. By default is true and print the configured `logger.requestFields`.
- `logger.requestFields`: Fields displayed when a request is logged. Possible values: `reqId`, `method`, `url`, `headers`, `body`, `query`,`params`, `duration`.
- `logger.reqIdBuilder`: A function called for each incoming request to create a request id.
- `logger.jsonIndentation`: The number of space characters to use as white space in JSON output. Default is 2 (0 in production).
- `logger.disableRoutesSummary`: Disable routes table displayed in the logger. By default debug is `false`.
- `logger.format`: Specify log format. Example: `%[%d{[yyyy-MM-dd hh:mm:ss,SSS}] %p%] %m`. See [ts-log-debug configuration](https://TypedProject.github.io/ts-log-debug/).
- `logger.ignoreUrlPatterns` (`String` or `RegExp`): List of pattern to ignore logged request according to the `request.url`.

> It's recommended to disable logRequest in production. Logger have a cost on the performance.

### Request logger

For each Express.Request, a logger will be attached and can be used like here:

```typescript
request.log.info({customData: "test"}) // parameter is optional
request.log.debug({customData: "test"})
request.log.warn({customData: "test"})
request.log.error({customData: "test"})
request.log.trace({customData: "test"})
```

A call with once of this method will generate a log according to the `logger.requestFields` configuration:

```bash
[2017-09-01 11:12:46.994] [INFO ] [TSED] - {
  "status": 200,
  "reqId": 1,
  "method": "GET",
  "url": "/api-doc/swagger.json",
  "duration": 92,
  "headers": {
    "host": "0.0.0.0:8001",
    "connection": "keep-alive",
    "upgrade-insecure-requests": "1",
    "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.101 Safari/537.36",
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "accept-encoding": "gzip, deflate",
    "accept-language": "fr-FR,fr;q=0.8,en-US;q=0.6,en;q=0.4"
  },
  "body": {},
  "query": {},
  "customData": "test"
}
```

You can configure this output from configuration:

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";

@ServerSettings({
   logger: {
       requestFields: ["reqId", "method", "url", "headers", "body", "query","params", "duration"]
   }
})
export class Server extends ServerLoader {

}
```

or you can override the middleware with @@OverrideMiddleware@@.

Example: 

```typescript
import {ServerLoader, ServerSettings, OverrideMiddleware, LogIncomingRequestMiddleware, Res, Req} from "@tsed/common";

@OverrideMiddleware(LogIncomingRequestMiddleware)
export class CustomLogIncomingRequestMiddleware extends LogIncomingRequestMiddleware {
 
    public use(@Req() request: any, @Res() response: any) {
    
        // you can set a custom ID with another lib
        request.id = require('uuid').v4()
        
        return super.use(request, response); // required 
    }
    
    protected requestToObject(request) {
        return {
           reqId: request.id,
           method: request.method,
           url: request.originalUrl || request.url,
           duration: new Date().getTime() - request.tsedReqStart.getTime(),
           headers: request.headers,
           body: request.body,
           query: request.query,
           params: request.params
        }
    }
}
```

### Shutdown logger

Shutdown return a Promise that will be resolved when ts-log-debug has closed all appenders and finished writing log events. 
Use this when your program exits to make sure all your logs are written to files, sockets are closed, etc.

```typescript
import {$log} from "ts-log-debug";

$log
  .shutdown()
  .then(() => {
     console.log("Complete")
  }); 
```

## Get configuration

The configuration can be reused throughout your application in different ways. 

- With dependency injection in [Service](/docs/services.md), [Controller](/docs/controllers.md), [Middleware](/docs/middlewares.md), [Filter](/docs/filters.md) or [Converter](/docs/converters.md).
- With the decorators @@Constant@@ and @@Value@@.

### From service (DI)

```typescript
import {ServerSettingsService, Service} from "@tsed/common";

@Service() // or Controller or Middleware
export class MyService {
  constructor(serverSettingsService: ServerSettingsService) {}
}
```

### From decorators

Decorators @@Constant@@ and @@Value@@ can be used in all classes
including: 
 
 - [Service](/docs/services.md),
 - [Controller](/docs/controllers.md),
 - [Middleware](/docs/middlewares.md),
 - [Filter](/docs/filters.md)
 - [Converter](/docs/converters.md).
 
@@Constant@@ and @@Value@@ accept an expression as parameters to
inspect the configuration object and return the value.

<<< @/docs/docs/snippets/providers/binding-configuration.ts

::: warning
@@Constant@@ return an Object.freeze() value.
:::

::: tip NOTE
The values for the decorated properties aren't available on constructor. Use $onInit() hook to use the value.
:::
