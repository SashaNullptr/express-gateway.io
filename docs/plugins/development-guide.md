---
layout: doc-section
title:  "Development guide"
doc-order: 25.2
---
## Development guide 

### Example

Here is [Example Plugin](https://github.com/ExpressGateway/express-gateway-plugin-example) with all extension points utilized. 

#### Installing Example plugin 

[Install][install] Express Gateway

run CLI command `eg gateway create` to create Express Gateway instance 

``` 
> eg gateway create
? What's the name of your Express Gateway? example-gateway
? Where would you like to install your Express Gateway? example-gateway
? What type of Express Gateway do you want to create? Basic (default pipeline with proxy)
```
While using the CLI command (`eg plugin install express-gateway-plugin-example`) can speedup installation of the plugin, let's do it the manual way to understand mechanics: 
- go to instance folder
- npm install the example package
- edit `./config/system.config.yml` file and enable plugin

```
cd example-gateway
npm i --save express-gateway-plugin-example
```

Now edit the `./config/system.config.yml` file

You should find the following section: 
```yml
# plugins:
  # express-gateway-plugin-example:
  #   param1: 'param from system.config' 
```
Uncomment the `express-gateway-plugin-example` plugin declaration

```yml
plugins:
    express-gateway-plugin-example:
        param1: 'param from system.config' 
```
Equivalent JSON configuration is: 
```json
"plugins": {
    "express-gateway-plugin-example": {
        "param1": "param from system.config"
    }
} 
```

#### Run with Example plugin 

Run Express Gateway 
```
LOG_LEVEL=debug npm start
```

Here are some of output lines we expect to see if everything is ok. 

```
Loading plugins. Plugin engine version: 1.2.0
...
Loaded plugin express-gateway-plugin-example using from package express-gateway-plugin-example
...
registering policy example
...
registering gatewayExtension
...
registering condition url-match
...

```
### Package overview
The `express-gateway-plugin-example` plugin is an npm package.

Main components are:
manifest.js file - contains and exports plugin definition 
package.json - contains plugin name and dependencies 

All the rest is completely optional. Still, some structure may help. That is why the example plugin contains individual folders for each extension type

Note: `manifest.js` naming is just a convention. The name of this file is configured in `main` property of `package.json`. Node.JS standard naming is index.js  

#### Manifest.js File overview (Plugin Manifest)
In a nutshell the plugin interface\manifest is straightforward 
```js
module.exports = {
  version: '1.2.0',
  init: function (pluginContext) {
    // pluginContext.registerX calls 
  },
  policies:['example'], 
  options:{
    param1: {
      type: 'string',
      required: true
    }
  }
}
```
- version - Hint for the Plugin System how to process plugin, '1.2.0' only at this point
- init - Function that will be called right after Express Gateway will `require` the plugin package
- policies - list of policies to be added to the whitelist (requires confirmation from user)
- options - JSON schema for support plugin options. Will be used for prompting during CLI execution. Note: at this point only simple types: `boolean`, `string` and `number` are supported. Full featured JSON Schema validation is planned for future releases

[Policy Development guide]({{ site.baseurl}} {% link docs/plugins/policy-development.md %})
[Condition Development guide]({{ site.baseurl}} {% link docs/plugins/condition-development.md %})
[Gateway and Admin Route Development guide]({{ site.baseurl}} {% link docs/plugins/route-development.md %})

### Events 
Express Gateway exposes several events. 
This is how to subscribe: 

```js
module.exports = {
  version: '1.2.0',
  init: function (pluginContext) {
    pluginContext.eventBus.on('hot-reload', function ({ type, newConfig }) {
      // "type" is gateway or system
      // depends on what file was changed 
      // newConfig - is newly loaded configuration of ExpressGateway  
      console.log('hot-reload', type, newConfig);
    });
    pluginContext.eventBus.on('http-ready', function ({ httpServer }) {
      console.log('http server is ready', httpServer.address());
    });
    pluginContext.eventBus.on('https-ready', function ({ httpsServer }) {
      console.log('https server is ready', httpsServer.address());
    });
    pluginContext.eventBus.on('admin-ready', function ({ adminServer }) {
      console.log('admin server is ready', adminServer.address());
    });
  }
}
``` 

[install]: {{ site.baseurl}} {% link docs/installation.md %}