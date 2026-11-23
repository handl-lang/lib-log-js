# lib-log-js

JavaScript backend of the `log` contract — leveled records to the JS
console via the host-injected `js.call` marshaller
(`console.debug`/`info`/`warn`/`error`; `trace`->`debug`,
`fatal`->`error`).

`src/index.hnl` implements the full `lib-log` surface as a single
module: the same pure logger algebra and canonical `format` renderer
as `log-c` (backend-portable `LEVEL ts msg k=v ...` text), plus the
effectful emission path.

Call shape: `console.<method>(line[, attrs])` — the text line carries
level + ISO-8601-UTC timestamp (`Date.now()` -> the same
civil-from-days algorithm as `log-c`) + message, and the merged attrs
record rides as a real JS object (`js_object` descriptor — the
`lib-json-js`/`lib-fetch-js` choice) so devtools shows structured,
inspectable data. The attrs argument is omitted when empty.

Unlike `log-c`/`log-js`'s contract siblings this backend's emission
tests need a JS host: on a host without the `js` natives the `js.call`
sites surface as `call_host` failures, which `scripts/hnl-gate.sh`
tolerates (the `lib-json-js` convention). The 37 pure-surface tests
(levels, logger algebra, `format`) run anywhere.

One implementation note: the module exports an arrow named `error`
(the error-level convenience), which shadows the host `error` module
inside the file — so `error.raise` is imported as `raise` up front.

## Layout

```text
src/index.hnl       the .hnl sources (all contract arrows)
src/index.test.hnl  contract tests — 44 cases (37 host-free, 7 js.call)
pkg.hnl             package manifest — modules, deps, description, license
```

Dependencies: log (contract), js

## Gate

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.
