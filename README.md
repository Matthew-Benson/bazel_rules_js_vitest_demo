# Overview

Forked from https://github.com/aspect-build/bazel-examples/tree/main/vue.
Follow up to https://github.com/bazelbuild/rules_nodejs/issues/3528.

This is a minimal issue reproduction for Vitest with Bazel's `rules_js` that
highlights an issue where Vitest imports fail in tests without global imports
enabled.

This feature works when running Vitest directly with pnpm, but fails when
running through Bazel, so it seems to either be an import resolution problem
within Vitest in this edge-case of running with Bazel, or an issue with npm
packages with `rules_js`, though it should be noted that it fails with
`rules_nodejs` too which you can see in
[this example repo](https://github.com/Matthew-Benson/bazel_vitest_error_demo).

## Error Case

To see the failure case, run

```sh
bazel test :static_vitest
```

which will give the error:

```
Error: [vite-node] Failed to load vitest
 ❯ static_import.test.ts:1:31
      1| import { assert, expect, test } from 'vitest'
       |                               ^
      2|
      3| test('Math.sqrt()', () => {
```

or

```sh
bazel test :static_extra_import_vitest
```

which is the same build rule as the above, but with an explicit import
for `:node_modules/vitest` in the rule and has a different error:

```
FAIL  static_import.test.ts [ static_import.test.ts ]
Error: No test suite found in file /root/.cache/bazel/_bazel_root/b59cc6b229e9d59ee04afe00c3117d2b/sandbox/processwrapper-sandbox/170/execroot/__main__/bazel-out/k8-fastbuild/bin/static_extra_import_vitest.sh.runfiles/__main__/static_import.test.ts
```

## Passing Cases

### Bazel

Interestingly, global imports configured in the Vitest config do not have the
same problem.

```sh
bazel test :global_vitest
```

This test will pass without errors.

Naturally, global imports make the code a bit harder to read and understand,
so we want to avoid this if at-all possible.

### pnpm

Running either of these tests with `pnpm` will pass, though you may see some
test failures in packages under `bazel-` directories, which can be ignored
in this example.

```sh
function pnpm { bazel run -- @pnpm//:pnpm --dir $PWD "$@" }
pnpm install
pnpm run test:unit run --config=$PWD/static_vite.config.ts static_import.test.ts
pnpm run test:unit run --config=$PWD/global_vite.config.ts global_import.test.ts
rm -rf node_modules/
```
