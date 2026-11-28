# lib-log-js

JavaScript backend of the `log` contract — leveled records to the JS
console via the host-injected `js.call` marshaller
(`console.debug`/`info`/`warn`/`error`; `trace`→`debug`,
`fatal`→`error`). Pure `.hnl`.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the full `lib-log` surface as a single module |
| `src/index.test.hnl` | the .hnl test suite |
| `pkg.hnl` | manifest — dep `log` only (`js` is a host/builtin contract, NOT a dep — see Contract) |
| `scripts/hnl-gate.sh`, `workflows/gate.yaml` | the gate / bupha entry |

## Contract

- Same pure logger algebra and canonical `format` renderer as
  `log-c`; emission calls `console.<method>(line[, attrs])` — the
  text line carries level + ISO-8601-UTC timestamp + message, and the
  merged attrs ride as a real JS object (`js_object` descriptor, the
  `lib-json-js`/`lib-fetch-js` choice); omitted when empty.
- `js` stays out of `pkg.hnl` `deps` and `.handl.toml`
  `[dependencies]`: the `js` contract is a host/builtin surface
  (compile-js binds `from "js"` to `__handl_rt` shims regardless of
  dep tables). Declaring the dep makes `hnl run`/`hnl test` resolve
  `js` to `lib-js`'s signature-only exports — `call` erases to `null`
  and every call dies with "callee is not an arrow-value". Undeclared,
  the import lowers to the ambient `"js.call"` callee and fails
  honestly as `call_host: function 'js.call' not found` on hosts
  without JS natives.
- Emission tests need a JS host: without `js` natives the `js.call`
  sites surface as `call_host` failures, which `hnl-gate.sh` tolerates
  (the `lib-json-js` convention). The 37 pure-surface tests run
  anywhere.
- Gotcha: the module exports an arrow named `error` which shadows the
  host `error` module — `error.raise` is imported as `raise` up front.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
