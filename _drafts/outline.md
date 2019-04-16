- module vs chunk vs bundle

  - [What are module, chunk and bundle in webpack?](<https://stackoverflow.com/questions/42523436/what-are-module-chunk-and-bundle-in-webpack>): 

    A **bundle** is some related code packaged into a single file.

    If you don't want all of your code be put into a single huge bundle you will split it into multiple bundles which are called **chunks** in webpack terminology. In some cases you will define how your code is split chunks yourself (with an `entry` that points to multiple files and an output file template like `[name].[chunkhash].js`), in other cases webpack will do it for you (e.g. with `CommonsChunkPlugin`).

  - [Concepts - Bundle vs Chunk](<https://github.com/webpack/webpack.js.org/issues/970>): Yes a chunk is a bundle. All in all, a chunk is the abstraction in the source code that encapsulates and defines how groups of modules will be written to file.

  - [SurviveJS: Glossary](<https://survivejs.com/webpack/appendices/glossary/>): **Chunk** is a webpack-specific term that is used internally to manage the bundling process. Webpack composes bundles out of chunks, and there are several types of those.

    

- devOps

- `contenthash`

- 

- `optimization.splitChunks.chunks = 'all'`

- vendors 分离

- vendors 单个包分离

- 分离应用层代码

