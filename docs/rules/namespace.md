# namespace

Enforces names exist at the time they are dereferenced, when imported as a full namespace (i.e. `import * as foo from './foo'; foo.bar();` will report if `bar` is not exported by `./foo`.).

Will report at the import declaration if there are _no_ exported names found.

Also, will report for computed references (i.e. `foo["bar"]()`).

Reports on assignment to a member of an imported namespace.

Note: for modules, the plugin will find exported names from [`jsnext:main`], if present in `package.json`.
Redux's npm module includes this key, and thereby is lintable, for example. Otherwise,
the whole `node_modules` folder is ignored by default ([`import/ignore`]) as most published modules are
formatted in CommonJS, which [at time of this writing](https://github.com/benmosher/eslint-plugin-import/issues/13)
is not able to be analyzed for exports.

## Rule Details

Currently, this rule does not check for possible
redefinition of the namespace in an intermediate scope. Adherence to the ESLint
`no-shadow` rule for namespaces will prevent this from being a problem.

For [ES7], reports if an exported namespace would be empty (no names exported from the referenced module.)

Given:
```js
// @module ./named-exports
export const a = 1
const b = 2
export { b }

const c = 3
export { c as d }

export class ExportedClass { }

// ES7
export * as deep from './deep'
```
and:
```js
// @module ./deep
export const e = "MC2"
```

See what is valid and reported:

```js
// @module ./foo
import * as names from './named-exports'

function great() {
  return names.a + names.b  // so great https://youtu.be/ei7mb8UxEl8
}

function notGreat() {
  doSomethingWith(names.c) // Reported: 'c' not found in imported namespace 'names'.

  const { a, b, c } = names // also reported, only for 'c'
}

// also tunnels through re-exported namespaces!
function deepTrouble() {
  doSomethingWith(names.deep.e) // fine
  doSomethingWith(names.deep.f) // Reported: 'f' not found in deeply imported namespace 'names.deep'.
}

```

## Further Reading

- Lee Byron's [ES7] export proposal
- [`import/ignore`] setting
- [`jsnext:main`](Rollup)

[ES7]: https://github.com/leebyron/ecmascript-more-export-from
[`import/ignore`]: ../../README.md#importignore
[`jsnext:main`]: https://github.com/rollup/rollup/wiki/jsnext:main
