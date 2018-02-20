# User Guide
## Configuration

The CLI has 3 methods of configuration:

1. `$FOSSA_API_KEY` and `$FOSSA_ENDPOINT` can be set by environment variable.
2. `.fossa.yaml` contains most of the configurable settings, and override environment variables.
3. Flags can be passed directly to commands, and will override both environment variables and the configuration file.

### Modules and the module spec

The CLI must be configured to build specific modules as entry points. 

Providing entry points helps reduce false positives. For example, a Go project may have a `docs/` folder that provides documentation using a Ruby static site generator. If the static site generator is not distributed to users, it's generally not necessary to analyze it for license obligations.

When used with the CLI flag, the module spec is defined as:

```
module1,module2,module3
```

where a module is defined as:

```
module_type:module_path
```

The characters `:` and `,` are regarded as special characters. They may be escaped by a preceding backslash (`\`). <!-- TODO: this escaping is not actually implemented. -->

Some examples:

- `commonjs:.,rubygem:./docs`
- `golang:github.com/fossas/fossa-cli/cmd/fossa`

### `.fossa.yaml`

The CLI searches for a `.fossa.yaml` in the working directory.

#### v1 (current)

```yaml
version: 1

cli:
  # # Defaults to https://app.fossa.io
  # server: https://fossa.on-prem
  api_key: some-key-here
  # # If `project` or `locator` are unset, infer locator from VCS.
  # project: git+github.com/fossas/fossa-cli
  # locator: git+github.com/fossas/fossa-cli$revision

analyze:
  modules:
    - name: fossa-cli
      path: github.com/fossas/fossa-cli/cmd/fossa
      type: gopackage
```

#### v2 (preview)

```yaml
version: 2

endpoint: https://fossa.on-prem # Defaults to https://app.fossa.io
api_key: some-key-here

project:
  # If `project` or `revision` are unset, infer locator from VCS.
  name: github.com/fossas/fossa-cli
  revision: 9ad32d41ed38b5952b40af5c67185ed97c86f31a

modules:
  - path: github.com/fossas/fossa-cli/cmd/fossa
    type: gopackage
```

## CLI Reference

All flags should be passed to the invoked subcommand. Global flags are currently NOT supported.

### `fossa`
Runs an analysis, uploading the results to FOSSA. Can optionally build if required.

#### Example
```bash
# Runs an analysis and uploads the results. Builds if required.
# This is probably the command you want.
FOSSA_API_KEY=YOUR_API_KEY fossa -m go:./cmd/fossa
```

#### Flags
##### `-c, --config file_name`
Path to a configuration file. Defaults to `.fossa.yaml`.

##### `-p, --project project_name`
Sets the configuration value for the FOSSA project.

##### `-r, --revision revision_hash`
Sets the configuration value for the FOSSA project's revision.

##### `-e, --endpoint url`
Sets the endpoint of the FOSSA server. Useful for on-premises deployments.

##### `-m, --modules module_spec`
Passed to `fossa build` and `fossa analyze`.

<!-- ##### `-i, --ignore ignore_spec`
Passed to `fossa analyze`. -->

##### `-o, --output`
Passed to `fossa analyze`.

##### `-b, --build`
Runs a build if required.

##### `-f, --force`
Passed to `fossa build`.

##### `--debug`
Print debugging information to `stderr`.

##### `-v, --version`
Print the version, then exit.

##### `-h, --help`
Print a help message, then exit.

<!-- ### `fossa init`

Makes a best-effort attempt at inferring the correct configuration, then outputs the configuration to `stdout`.

Configuration inference is done on a language-by-language basis. For example, `commonjs` packages are inferred by checking for `package.json`s that are not within `node_modules`. -->

### `fossa build`

Makes a best-effort attempt at building the project using default build conventions.

#### Example
```bash
# No API key is required for builds
fossa build -m go:./cmd/fossa
```

#### Flags
##### `-c, --config file_name`
Path to a configuration file. Defaults to `.fossa.yaml`.

##### `-m, --modules module_spec`
Sets the modules to use as entry points for building the project.

##### `-f, --force`
Clear cached build artifacts (such as a `node_modules` folder), and run the build from scratch.

**WARNING:** This command will delete cached build artifacts! This is generally not what you want, unless you intend to use FOSSA to build your production binary. If you build your production binary ahead of time, you should NOT delete your build artifacts. Deleting build artifacts and re-running the build may cause some build systems to non-deterministically resolve a different set of dependencies, which will make your FOSSA analysis less accurate.

##### `--debug`
Print debugging information to `stderr`.

##### `-h, --help`
Print a help message, then exit.

### `fossa analyze`

Analyzes the project for a list of its dependencies, optionally uploading the results to FOSSA.

#### Example
```bash
# Show analysis results
FOSSA_API_KEY=YOUR_API_KEY fossa analyze --output
```

#### Flags
##### `-c, --config file_name`
Path to a configuration file. Defaults to `.fossa.yaml`.

##### `-p, --project project_name`
Sets the configuration value for the FOSSA project.

##### `-r, --revision revision_hash`
Sets the configuration value for the FOSSA project's revision.

##### `-e, --endpoint url`
Sets the endpoint of the FOSSA server. Useful for on-premises deployments.

##### `-m, --modules module_spec`
Sets the modules to use as entry points when analyzing dependencies.

<!-- ##### `-i, --ignore ignore_spec`
Sets the modules and paths to ignore when analyzing dependencies. -->

##### `-o, --output`
Prints analysis results to `stdout`. When this flag is set, `fossa analyze` provides interactive output on `stderr`.

##### `--allow-unresolved`
Do not fail on unresolved dependencies.

For some languages, `fossa analyze` does import path tracing to determine dependencies. If these the dependencies at the import paths cannot be found, the dependency is _unresolved_.

Unresolved dependencies generally indicate an incomplete build or some other kind of build error. For highly custom build systems, this may not be the case.

##### `--no-upload`
Do not upload analysis results.

##### `--debug`
Print debugging information to `stderr`.

##### `-h, --help`
Print a help message, then exit.

### `fossa upload`

Uploads user-provided build data to FOSSA. This allows users to manually provide a dependency list if their build system is too complex for FOSSA's default analyzer.

#### Example
```bash
# You can manually override the project and revision name.
# This is useful when `fossa` can't manually infer them from `git`.
FOSSA_API_KEY=YOUR_API_KEY_HERE fossa upload --project=PROJECT_NAME --revision=SOME_HASH --data=$(fossa analyze --no-upload --output)
```

#### Flags
##### `-c, --config file_name`
Path to a configuration file. Defaults to `.fossa.yaml`.

##### `-p, --project project_name`
Sets the configuration value for the FOSSA project.

##### `-r, --revision revision_hash`
Sets the configuration value for the FOSSA project's revision.

##### `-e, --endpoint url`
Sets the endpoint of the FOSSA server. Useful for on-premises deployments.

##### `--data`
The user-provided build data. See `fossa analyze --output` for an example of the correct build data format.

##### `--debug`
Print debugging information to `stderr`.

##### `-h, --help`
Print a help message, then exit.

<!-- ### `fossa report`

Print the project's license report.

#### Flags
##### `-t, --type report_type`
Print a specific type of license report for automatically creating attribution files. Possible report types include:
- `NOTICE`: generate a `NOTICE` file for your dependencies
- `ATTRIBUTION`: generate an `ATTRIBUTION` file for your dependencies

##### `-j, --json`
Print the report data in JSON format.
-->
### `fossa test`

Checks whether the project has licensing issues, as configured by its policy within FOSSA. If there are issues, it prints them on `stdout` and exits with code 1. If there are not issues, it exits with code 0.

#### Flags
<!-- ##### `-j, --json`
Print issues in JSON format. -->

##### `-c, --config file_name`
Path to a configuration file. Defaults to `.fossa.yaml`.

##### `-p, --project project_name`
Sets the configuration value for the FOSSA project.

##### `-r, --revision revision_hash`
Sets the configuration value for the FOSSA project's revision.

##### `-e, --endpoint url`
Sets the endpoint of the FOSSA server. Useful for on-premises deployments.

##### `-t, --timeout`
The amount of time to wait for an issue report (in seconds). Defaults to 10 minutes.