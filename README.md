# MCVE: devcontainers/ci post hook pushes wrong image tag through nested composites

Minimal reproduction of [actions/runner#3514](https://github.com/actions/runner/issues/3514) / [actions/runner#2030](https://github.com/actions/runner/issues/2030) as it affects [devcontainers/ci](https://github.com/devcontainers/ci).

## Structure

```text
workflow
└── outer-composite                   (no inputs)
    └── inner-composite               (input: image-tag=foo)
        └── devcontainers/ci          (node action, imageTag: ${{ inputs.image-tag }})
            ├── main: builds :foo
            └── post: pushes ???
```

## Observed behaviour

```text
[main] INPUT_IMAGETAG=foo  → builds ghcr.io/.../devcontainer:foo  ✓
[post] INPUT_IMAGETAG=""   → defaults to "latest"
                           → tries to push ghcr.io/.../devcontainer:latest  ✗ (never built)
```

## Expected behaviour

```text
[post] INPUT_IMAGETAG=foo  → pushes ghcr.io/.../devcontainer:foo  ✓
```

## Why it happens

Because of [actions/runner#2030](https://github.com/actions/runner/issues/2030), the inner composite receives the wrong context during its post hook. The action then attempts to default to "latest" instead of using the value from the main step, but no manifest with this tag exists.

## Workaround

Persist inputs in the main step using `core.saveState`, then read from `STATE_*` in the post step instead of `INPUT_*`. See [github/codeql-action#2557](https://github.com/github/codeql-action/pull/2557) for a real-world example.
