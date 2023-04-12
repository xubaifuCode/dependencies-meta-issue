**Basic Information:**

- There are three workspaces: `app1`, `app2`, and `lib1`.
- `app1` and `app2` both depend on `lib1` and use the `workspace:` protocol.
- `app1` and `app2` both set `dependenciesMeta.*.injected` to `true` for `lib1`.

---

- The `package.json` file of `lib1`:

```json
{
  "name": "lib1",
  "version": "1.0.1",
  "dependencies": {
    "b": "^2.0.1"
  },
  "peerDependencies": {
    "d": "*",
    "e": "*"
  }
}
```

- The `package.json` file of `app1`:

```json
{
  "name": "app1",
  "version": "1.0.0",
  "dependencies": {
    "lib1": "workspace: *"
  },
  "dependenciesMeta": {
    "lib1": {
      "injected": true
    }
  }
}
```

- The `package.json` file of `app2`:
 
```json
{
  "name": "app2",
  "version": "1.0.0",
  "dependencies": {
    "lib1": "workspace: *"
  },
  "dependenciesMeta": {
    "lib1": {
      "injected": true
    }
  }
}
```

- `pnpm-lock.yaml`

```yaml
lockfileVersion: '6.0'

importers:

  apps/app1:
    dependencies:
      lib1:
        specifier: 'workspace: *'
        version: file:packages/lib1(d@1.0.1)(e@0.2.33)
    dependenciesMeta:
      lib1:
        injected: true

  apps/app2:
    dependencies:
      d:
        specifier: ^1.0.1
        version: 1.0.1
      lib1:
        specifier: 'workspace: *'
        version: file:packages/lib1(d@1.0.1)(e@0.2.33)
    dependenciesMeta:
      lib1:
        injected: true

  packages/lib1:
    dependencies:
      b:
        specifier: ^2.0.1
        version: 2.0.1
      d:
        specifier: '*'
        version: 1.0.1
      e:
        specifier: '*'
        version: 0.0.1

packages:
  
  # 忽略其他的 package ...

  file:packages/lib1(d@1.0.1)(e@0.2.33):
    resolution: {directory: packages/lib1, type: directory}
    id: file:packages/lib1
    name: lib1
    version: 1.0.1
    peerDependencies:
      d: '*'
      e: '*'
    dependencies:
      b: 2.0.1
      d: 1.0.1
      e: 0.2.33
    dev: false
```

---

**Problem:**

When modifying the `package.json` file of `lib1` by adding a dependency `a` in `dependencies`, and then running `pnpm i --filter lib1`, the `dependencies` of `packages/lib1` in `pnpm-lock.yaml` has already added `a`.

```yaml
importers:
  packages/lib1:
    dependencies:
      # new dependency
      a:
        specifier: ^3.0.1
        version: 3.0.1
      # ignore others...
```


**However,** the information in `file:packages/lib1(d@1.0.1)(e@0.2.33)` is not updated accordingly, which may cause `app1` and `app2` unable to find the dependency `a`.

This problem can be confusing when using the `workspace:` protocol, as inconsistent declarations are made. Shouldn't it be updated automatically?

```yaml
packages:
  file:packages/lib1(d@1.0.1)(e@0.2.33):
    resolution: {directory: packages/lib1, type: directory}
    id: file:packages/lib1
    name: lib1
    version: 1.0.1
    peerDependencies:
      d: '*'
      e: '*'
    dependencies:
      b: 2.0.1
      d: 1.0.1
      e: 0.2.33
    dev: false
```

### pnpm version: 8.2.0

### Code to reproduce the issue: 

https://github.com/xubaifuCode/dependencies-meta-issue

### Expected behavior:

**For the situation where** `injected` is set to `true`, `app1` and `app2`, which depend on `lib1` using the `workspace:` protocol, should synchronize the dependency declarations of `lib1` and install dependencies correctly.

### Actual behavior:

**For the situation where** `injected` is set to `true`, `app1` and `app2`, which depend on `lib1` using the `workspace:` protocol, did not synchronize the dependency declarations of `lib1` and install dependencies correctly. If `injected` is canceled, the dependencies will be installed correctly.

### Additional information:

- `node -v` prints: v18.6.0
- Windows, macOS, or Linux?: macOS
