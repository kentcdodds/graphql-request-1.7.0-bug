# graphql-request-1.7.0-bug

This is a reproduction of a bug that came up in 1.7.0.

There are actually 2 bugs IMO.

## Bundle Size

In 1.7.0, `graphql-tag` was added as a dependency and `graphql` was added to
`peerDependencies`. One of the reasons we use `graphql-request` is because it's
so small.

Here's the webpack output for `graphql-request@1.6.0`:

```
Hash: 00ae46853441bb94f0bc
Version: webpack 4.16.2
Time: 637ms
Built at: 2018-07-24 16:55:50
  Asset      Size  Chunks             Chunk Names
main.js  12.6 KiB       0  [emitted]  main
Entrypoint main = main.js
[1] ./src/index.js 25 bytes {0} [built]
    + 3 hidden modules
```

Here's the webpack output for `graphql-request@1.7.0`:

```
Hash: 7489bc885808f855eac6
Version: webpack 4.16.2
Time: 2910ms
Built at: 2018-07-24 16:54:42
  Asset     Size  Chunks             Chunk Names
main.js  156 KiB       0  [emitted]  main
Entrypoint main = main.js
[2] ./src/index.js 25 bytes {0} [built]
[5] ./node_modules/graphql/index.mjs + 99 modules 474 KiB {0} [built]
    |    100 modules
    + 4 hidden modules
```

Notice the difference in the `main.js` filesize: 12.6 KiB before, 156 KiB after.
And this is using `webpack --mode production`. This is definitely a bug.

## webpack not handling a require statement

I'm honestly not sure what's up here, but if you open up the `index.html` in
`1.6.0` and open the console there's nothing in there and the code in
`graphql-request` runs fine.

If you open up the `index.html` in `1.7.0` there's the following error in the
console:

```
main.js:1 Uncaught ReferenceError: require is not defined
    at Object.<anonymous> (main.js:1)
    at t (main.js:1)
    at Module.<anonymous> (main.js:1)
    at t (main.js:1)
    at Object.<anonymous> (main.js:1)
    at t (main.js:1)
    at Module.<anonymous> (main.js:1)
    at t (main.js:1)
    at main.js:1
    at main.js:1
```

I'm not 100% certain why that's happening, but it's this statement:

```javascript
require('./../../process/browser.js')
```

I think it's in the `graphql/jsutils/instanceOf.mjs` file where it's using
`process`. I'm not sure why that's causing issues here...

\*\*That said, if the first bug is fixed (remove the dependency on `graphql`),
then this bug will also be fixed.

Thanks!
