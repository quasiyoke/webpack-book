# webpack-dev-server

Tools, such as [LiveReload](http://livereload.com/) or [Browsersync](http://www.browsersync.io/), allow to refresh the browser as you develop the application and avoid a refresh for CSS changes. It's possible to setup Browsersync to work with webpack through [browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin), but webpack has more tricks in store.

## Webpack `watch` Mode and *webpack-dev-server*

A good first step towards a better development environment is to use webpack in its **watch** mode. You can activate it through `webpack --watch`. Once enabled, it detects changes made to your files and recompiles automatically. *webpack-dev-server* (WDS) builds on top of the watch mode and goes even further.

WDS is a development server running **in-memory**, meaning the bundle contents aren't written out to files, but stored in memory. This is an important distinction when trying to debug code and styles.

By default WDS refreshes content automatically in the browser while you develop your application so you don't have to do it yourself. However it also supports an advanced webpack feature, **Hot Module Replacement** (HMR).

HMR allows patching the browser state without a full refresh making it particularly handy with libraries like React where a refresh blows away the application state. The *Hot Module Replacement* appendix covers the feature in detail.

WDS provides an interface that makes it possible to patch code on the fly, however for this to work effectively, you have to implement this interface for the client-side code. It's trivial for something like CSS because it's stateless, but the problem is harder with JavaScript frameworks and libraries.

## Emitting Files from WDS

Even though it's good that WDS operates in-memory by default for performance reasons, sometimes it can be good to emit files to the file system. This applies in particular, if you are integrating with another server that expects to find the files. [webpack-disk-plugin](https://www.npmjs.com/package/webpack-disk-plugin), [write-file-webpack-plugin](https://www.npmjs.com/package/write-file-webpack-plugin), and more specifically [html-webpack-harddisk-plugin](https://www.npmjs.com/package/html-webpack-harddisk-plugin) can achieve this.

W> You should use WDS strictly for development. If you want to host your application, consider other standard solutions, such as Apache or Nginx.

## Getting Started with WDS

To get started with WDS, install it first:

```bash
npm install webpack-dev-server --save-dev
```

As before, this command generates a command below the `npm bin` directory and you could run *webpack-dev-server* from there. After running the WDS, you have a development server running at `http://localhost:8080`. Automatic browser refresh is in place now, although at a basic level.

{pagebreak}

## Attaching WDS to the Project

To integrate WDS to the project, define an npm script for launching it. To follow npm conventions, call it as *start*. To tell the targets apart, pass information about the environment to webpack configuration so you can specialize as needed:

**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --env development",
  "build": "webpack --env production"
leanpub-end-insert
leanpub-start-delete
  "build": "webpack"
leanpub-end-delete
},
```

T> WDS picks up configuration like webpack itself. The same rules apply.

If you execute either *npm run start* or *npm start* now, you should see something in the terminal:

```bash
> webpack-dev-server --env development

Project is running at http://localhost:8080/
webpack output is served from /
Hash: 2bf6813b1d90a3b0653a
Version: webpack 3.8.1
Time: 689ms
     Asset       Size  Chunks                    Chunk Names
    app.js     328 kB       0  [emitted]  [big]  app
index.html  180 bytes          [emitted]
...
webpack: bundle is now VALID.
```

{pagebreak}

The server is running and if you open `http://localhost:8080/` at your browser, you should see something familiar:

![Hello world](images/hello_01.png)

If you try modifying the code, you should see output in your terminal. The browser should also perform a hard refresh on change.

WDS tries to run in another port in case the default one is being used. The terminal output tells you where it ends up running. You can debug the situation with a command like `netstat -na | grep 8080`. If something is running on the port 8080, it should display a message on Unix.

## Configuring WDS Through Webpack Configuration

To customize WDS functionality it's possible to define a `devServer` field at webpack configuration. You can set most of these options through the CLI as well, but managing them through webpack is a good approach.

{pagebreak}

Enable additional functionality as below:

**webpack.config.js**

```javascript
...

leanpub-start-delete
module.exports = {
  // Entries have to resolve to files! It relies on Node.js
  // convention by default so if a directory contains *index.js*,
  // it resolves to that.
leanpub-end-delete
leanpub-start-insert
const commonConfig = {
leanpub-end-insert
  ...
};

leanpub-start-insert
const productionConfig = () => commonConfig;

const developmentConfig = () => {
  const config = {
    devServer: {
      // Display only errors to reduce the amount of output.
      stats: "errors-only",

      // Parse host and port from env to allow customization.
      //
      // If you use Docker, Vagrant or Cloud9, set
      // host: options.host || "0.0.0.0";
      //
      // 0.0.0.0 is available to all network devices
      // unlike default `localhost`.
      host: process.env.HOST, // Defaults to `localhost`
      port: process.env.PORT, // Defaults to 8080
    },
  };

  return Object.assign({}, commonConfig, config);
};

module.exports = env => {
  if (env === "production") {
    return productionConfig();
  }

  return developmentConfig();
};
leanpub-end-insert
```

After this change, you can configure the server host and port options through environment parameters. The merging portion of the code (`Object.assign`) is tricky (shallow merge) and it will be fixed in the *Splitting Configuration* chapter.

If you access through `http://localhost:8080/webpack-dev-server/`, WDS provides status information at the top. If your application relies on WebSockets and you use WDS proxying, you need to use this particular url as otherwise WDS logic interferes.

![Status information](images/status-information.png)

T> [dotenv](https://www.npmjs.com/package/dotenv) allows you to define environment variables through a *.env* file. *dotenv* allows you to control the host and port setting of the setup quickly.

T> Enable `devServer.historyApiFallback` if you are using HTML5 History API based routing.

{pagebreak}

### Understanding `--env`

Even though `--env` allows to pass strings to the configuration, it can do a bit more. Consider the following example:

**package.json**

```json
"scripts": {
  "start": "webpack-dev-server --env development",
  "build": "webpack --env.target production"
},
```

Instead of a string, you should receive an object `{ target: "production" }` at configuration now. You could pass more key-value pairs, and they would go to the `env` object. If you set `--env foo` while setting `--env.target`, the string wins. Webpack relies on [yargs](http://yargs.js.org/docs/#parsing-tricks-dot-notation) for parsing underneath.

## Enabling Error Overlay

WDS provides an overlay for capturing warnings and errors:

**webpack.config.js**

```javascript
const config = {
  devServer: {
    ...
leanpub-start-insert
    // overlay: true is equivalent
    overlay: {
      errors: true,
      warnings: true,
    },
leanpub-end-insert
  },
};
```

{pagebreak}

Run the server now (`npm start`) and break the code to see an overlay in the browser:

![Error overlay](images/error-overlay.png)

## Enabling Hot Module Replacement

Hot Module Replacement is one of those features that sets webpack apart. Implementing it requires additional effort on both server and client-side. The *Hot Module Replacement* appendix discusses the topic in greater detail. If you want to integrate HMR to your project, give it a look. It won't be needed to complete the tutorial, though.

## Accessing the Development Server from Network

It's possible to customize host and port settings through the environment in the setup (i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows). The default settings are enough on most platforms.

To access your server, you need to figure out the ip of your machine. On Unix, this can be achieved using `ifconfig | grep inet`. On Windows, `ipconfig` can be utilized. An npm package, such as [node-ip](https://www.npmjs.com/package/node-ip) come in handy as well. Especially on Windows, you need to set your `HOST` to match your ip to make it accessible.

## Making It Faster to Develop Configuration

WDS will handle restarting the server when you change a bundled file, but what about when you edit the webpack config? Restarting the development server each time you make a change tends to get boring after a while. This can be automated as [discussed in GitHub](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892) by using [nodemon](https://www.npmjs.com/package/nodemon) monitoring tool.

To get it to work, you have to install it first through `npm install nodemon --save-dev`. After that, you can make it watch webpack config and restart WDS on change. Here's the script if you want to give it a go:

**package.json**

```json
"scripts": {
  "start": "nodemon --watch webpack.config.js --exec \"webpack-dev-server --env development\"",
  "build": "webpack --env production"
},
```

It's possible WDS [will support the functionality](https://github.com/webpack/webpack/issues/3153) itself in the future. If you want to make it reload itself on change, you should implement this workaround for now.

{pagebreak}

## Polling Instead of Watching Files

Sometimes the file watching setup provided by WDS won't work on your system. It can be problematic on older versions of Windows, Ubuntu, Vagrant, and Docker. Enabling polling is a good option then:

**webpack.config.js**

```javascript
const developmentConfig = merge([
leanpub-start-insert
  {
    devServer: {
      watchOptions: {
        // Delay the rebuild after the first change
        aggregateTimeout: 300,

        // Poll using interval (in ms, accepts boolean too)
        poll: 1000,
      },
    },
    plugins: [
      // Ignore node_modules so CPU usage with poll
      // watching drops significantly.
      new webpack.WatchIgnorePlugin([
        path.join(__dirname, "node_modules")
      ]),
    ],
leanpub-end-insert
  },
  ...
]);
```

The setup is more resource intensive than the default but it's worth trying out.

T> There are more details in *webpack-dev-server* issue [#155](https://github.com/webpack/webpack-dev-server/issues/155).

## Alternate Ways to Use *webpack-dev-server*

You could have passed the WDS options through a terminal. It's clearer to manage the options within webpack configuration as that helps to keep *package.json* nice and tidy. It's also easier to understand what's going on as you don't need to dig out the answers from the webpack source.

Alternately, you could have set up an Express server and use a middleware. There are a couple of options:

* [The official WDS middleware](https://webpack.js.org/guides/development/#webpack-dev-middleware)
* [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)
* [webpack-universal-middleware](https://www.npmjs.com/package/webpack-universal-middleware)
* [webpack-isomorphic-dev-middleware](https://www.npmjs.com/package/webpack-isomorphic-dev-middleware)

There's also a [Node.js API](https://webpack.js.org/configuration/dev-server/) if you want more control and flexibility.

W> There are [slight differences](https://github.com/webpack/webpack-dev-server/issues/106) between the CLI and the Node API.

## Other Features of *webpack-dev-server*

WDS provides functionality beyond what was covered above. There are a couple of relevant fields that you should be aware of:

* `devServer.contentBase` - Assuming you don't generate *index.html* dynamically and prefer to maintain it yourself in a specific directory, you need to point WDS to it. `contentBase` accepts either a path (e.g., `"build"`) or an array of paths (e.g., `["build", "images"]`). This defaults to the project root.
* `devServer.proxy` - If you are using multiple servers, you have to proxy WDS to them. The proxy setting accepts an object of proxy mappings (e.g., `{ "/api": "http://localhost:3000/api" }`) that resolve matching queries to another server. Proxy settings are disabled by default.
* `devServer.headers` - Attach custom headers to your requests here.

T> [The official documentation](https://webpack.js.org/configuration/dev-server/) covers more options.

## Development Plugins

The webpack plugin ecosystem is diverse and there are a lot of plugins that can help specifically with development:

* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin) can be handy when you are developing on a case-insensitive environments like macOS or Windows but using case-sensitive environment like Linux for production.
* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin) allows webpack to install and wire the installed packages with your *package.json* as you import new packages to your project.
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils) contains webpack utilities developed for [Create React App](https://www.npmjs.com/package/create-react-app). Despite its name, they can find use beyond React. If you want only webpack message formatting, consider [webpack-format-messages](https://www.npmjs.com/package/webpack-format-messages).

{pagebreak}

## Output Plugins

There are also plugins that make webpack output easier to notice and understand:

* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin) rings the system bell on failure instead of letting webpack fail silently.
* [webpack-notifier](https://www.npmjs.com/package/webpack-notifier) uses system notifications to let you know of webpack status.
* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) can be used to get tidier output during the build process. Take care if you are using Continuous Integration (CI) systems like Travis as they can clobber the output. Webpack provides `ProgressPlugin` for the same purpose. No nyan there, though.
* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin) improves on error reporting of webpack. It captures common errors and displays them in a friendlier manner.
* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard) gives an entire terminal based dashboard over the standard webpack output. If you prefer clear visual output, this one comes in handy.

{pagebreak}

## Conclusion

WDS complements webpack and makes it more friendly by developers by providing development oriented functionality.

To recap:

* Webpack's `watch` mode is the first step towards a better development experience. You can have webpack compile bundles as you edit your source.
* Webpack's `--env` parameter allows you to control configuration target through terminal. You receive the passed `env` through a function interface.
* WDS can refresh the browser on change. It also implements **Hot Module Replacement**.
* The default WDS setup can be problematic on certain systems. For this reason, more resource intensive polling is an alternative.
* WDS can be integrated to an existing Node server using a middleware. This gives you more control than relying on the command line interface.
* WDS does far more than refreshing and HMR. For example proxying allows you to connect it with other servers.

In the next chapter, you learn to compose configuration so that it can be developer further later in the book.
