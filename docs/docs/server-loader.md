# ServerLoader
## Lifecycle Hooks

[ServerLoader](/api/common/server/components/ServerLoader.md) calls a certain number of methods (hooks) during its initialization
phase (lifecycle). This lifecycle hooks that provide visibility into these key life moments and the ability to act
when they occur.

This schemes resume the order of the server's lifecycle and a service's lifecycle.

![lifecycle-hooks](./../assets/hooks-in-sequence.png)

Hook method | Description
--- | --- | ---
`constructor` | On this phase nothing is constructed. Express app isn't created.
[`$onInit`](#serverloaderoninit-void-promise) | Respond when the server starting his lifecycle. Is good place to initialize Database connection.
[`$beforeRoutesInit`](#serverloaderbeforeroutesinit-void-promise) | This hooks is the right place to configure the middlewares that must be used with your ExpressApplication. At this step, [InjectorService](/api/di/services/InjectorService.md) and [services](/docs/services.md) are ready and can be injected. The [Controllers](/docs/controllers.md) isn't built.
[`$afterRoutesInit`](#serverloaderafterroutesinit-void-promise) | Respond just after all [Controllers](/docs/controllers.md) are built. You can configure the [`serve-static`](https://github.com/expressjs/serve-static) middleware on this phase.
[`$onReady`](#serverloaderonready-void) | Respond when the server is ready. At this step, HttpServer or/and HttpsServer object is available. The server listen the port.
`$onServerInitError`| Respond when an error is triggered on server initialization.

> For more information on Service hooks see [Services lifecycle hooks](/docs/services.md#lifecycle-hook)

### Hooks examples
#### ServerLoader.$onInit(): void | Promise

During this phase you can initialize some stuff before injector loading. This hook accept a promise as return and let you to wait the database connection before run the next lifecycle's phase.

Example with mongoose Api:
```typescript
class Server extends ServerLoader {
  async $onInit(): Promise  {
    return new Promise((resolve, reject) => {
       setTimeout(resolve, 100)
    });
  }
}
```
::: tip Note
Database connection can be performed with Asynchronous Provider since v5.26. See [custom providers](/docs/custom-providers.md)
:::

***

#### ServerLoader.$beforeRoutesInit(): void | Promise

Some middlewares are required to work with all decorators as follow:

* `cookie-parser` are required to use `@CookieParams`,
* `body-parser` are require to use `@BodyParams`,
* [`method-override`](https://github.com/expressjs/method-override).

At this step, [services](/docs/services.md) are built.

Example of middlewares configuration:
```typescript
class Server extends ServerLoader {
    async $beforeRoutesInit(): void | Promise  {

        const cookieParser = require('cookie-parser'),
            bodyParser = require('body-parser'),
            compress = require('compression'),
            methodOverride = require('method-override'),
            session = require('express-session');

        this.use(GlobalAcceptMimesMiddleware)
            .use(bodyParser.json())
            .use(bodyParser.urlencoded({
                extended: true
            }))
            .use(cookieParser())
            .use(compress({}))
            .use(methodOverride());

    }
}
```
> `$beforeRoutesInit` accept a promise to defer the next lifecycle's phase.

***

#### ServerLoader.$afterRoutesInit(): void | Promise

This hook will be called after all the routes are collected by [`ServerLoader.mount()`](/api/common/server/components/ServerLoader.md)
or [`ServerLoader.scan()`](/api/common/server/components/ServerLoader.md).
When all routes are collected, [ServerLoader](/api/common/server/components/ServerLoader.md) build the [controllers](/docs/controllers.md) then [ServerLoader](/api/common/server/components/ServerLoader.md) mount each route to the ExpressApp.

This hook is the right place to add middlewares before the [Global Handlers Error](/docs/middlewares/override/global-error-handler.md).

```typescript
class Server extends ServerLoader {
    public $afterRoutesInit(){

    }
}
```

***

#### ServerLoader.$onReady(): void

On this phase your Express application is ready. All controllers are imported and all services is constructed.
You can initialize other server like a Socket server.

Example:
```typescript
class Server extends ServerLoader {
    public $onReady(): void {
        console.log('Server ready');
        const io = SocketIO(this.httpServer);
        // ...
    }
}
```
> If you want integrate Socket.io, you can see the tutorials ["How to integrate Socket.io"](/tutorials/socket-io.md).

## Versioning REST API

Ts.ED provide the possibility to mount multiple Rest path instead of the default path `/rest`.
You have two methods to configure all global endpoints for each directories scanned by the [ServerLoader](/api/common/server/components/ServerLoader.md).

```typescript
import {ServerLoader, ServerSettings} from "@tsed/common";
import Path = require("path");
const rootDir = Path.resolve(__dirname);

@ServerSettings({
   rootDir,
   mount: {
     "/rest": "${rootDir}/controllers/current/**/*.js",
     "/rest/v1": [
        "${rootDir}/controllers/v1/users/*.js",
        "${rootDir}/controllers/v1/groups/*.js"
     ]
   }
})
export class Server extends ServerLoader {

}
```
> Note: mount attribute accept a list of glob for each endpoint. That lets you declare a resource versioning.



