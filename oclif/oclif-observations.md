# Observations on how `oclif` works
Including OCLIF internls

https://github.com/oclif/oclif

Note: `XXXXX` and `xxxxx` are the names of your command.

## Main concepts:
### plugins
There are three types of plugins
* UserPlugins
* DevPlugins
* CorePlugins

### hook-cmmand-?
* Hooks, commands, plugins
* Each command is like a plugin.
* Each plugin is like a commmand.
* Each plugin (command?) has a set of hooks
* Each command has a set of hooks

### Hook
What is a hook?
* Definition of Hook in [sfcli: hooks](https://developer.salesforce.com/docs/atlas.en-us.228.0.sfdx_cli_plugins.meta/sfdx_cli_plugins/cli_plugins_customize_hooks.htm)
* [oclif hooks](https://oclif.io/docs/hooks)

* Two types of hooks:  Lifecycle hooks & Custom hooks
* Two types of hooks:  Lifecycle events & Custom events
* Lifecycle Events: `init`, `prerun`, `postrun`, `command_not_found`.
`@oclif/plugin-plugins/lib/hooks/update.js` is outside the update plugin.
* Custom Events need to be called explicitly using `this.config.runHook()`
* Lifecycle Events: are called implicitly/automatically (?).
* Each hook hasa a separate js/ts file.
* Order of execution:
  1. `init`
  2. Find the command (means?). (If not found, go to step 7) After command was found:
  3. `prerun`
  4. run(): the main command is run
  5. `postrun`
  6. done.
  7. if command not found: `command_not_found`

* The structure of a plugin is very similar to the structure of commands: both have `run()`. [See](https://github.com/oclif/plugin-update/tree/main/src).

### init hook
* The `init` hook kof the `update` plugin,  Sets important config/values:
   * `binPath` -- ? `this.config.binPath || this.config.bin;`
   * `lastrunfile` filename
   * `autoupdatefile`
   * `autoupdatelogfile` -- do we have such thing?
   * `clientRoot` -- (See below)
   * `autoupdateEnv` no idea.
   * `clientDir` -- ... (bsed no `clientRoot` and current version = config.version )
   * 
### Manifest
Which places can contain manifest?

* Where is manifest file (is supposed to be) read from: ?
* Where does function `fetchManifest()` read from? s3url!

* Where manifest is used? (fetched)
  * check update (or skip)
  * actual update

[Invariance]: this.config.version === manifest.version

### scoped Vars
* How to set scoped vars?
* What are some scoped vars?
  * `XXXXX_UPDATE_INSTRUCTIONS` -- Used at onset of update. ...

  * `XXXXX_CACHE_DIR`
  * `XXXXX_CONFIG_DIR`
  * `XXXXX_DATA_DIR`
  * `XXXXX_BINPATH`
  * `XXXXX_NPM_REGISTRY`
  * `XXXXX_S3_BUCKET`
  * `XXXXX_OCLIF_CLIENT_HOME`
  * `XXXXX_UPDATE_INSTRUCTIONS`

### Other minor concepts
#### channel
`this.channel`. Usually = `stable`
#### tidy, touch
what does it do? `this.tidy();`

* `tidy` and `touch` are relted tools.
* `.tidy()` removes files older than a certain time.
* `.touch()` updates the timestamp of the file, so that it is not deleted in the next `tidy()`

### Internal configs
The `config` & The `pjson`
* The config fields
  * `config.binPath` -- If undefined, it does skipUpdate => `'not updatable'`. How is it set?
  * `this.config.version` Updating CLI from `this.config.version` to `manifest.(version, channel)`

* The `pjson` fields

### local cache
* The `.local/share` is used for what?
   * `~/.local/share/xxxxx/client/6.0.7` is the output of update fter downloading.

### client dir
* The clientDir or `clientRoot` is `~/.local/share/xxxxx/client`.
* It is cleaned (wiped) before an update. (by `ensureClientDir()`)
* Output of the extraction stream is `.clientRoot`/`maifest.version` for example `~/.local/share/xxxxx/client/6.0.7`

### Stream
* It uses `stream` for downloading the update. Not promises, etc.
* Strems can be `pause()`ed or `stream.resume()`ed from downstream. (downstrem controls the dataflow)
* Streams can be redirected to `extract()`.
* You can `await` the downstream that you attached (eg the `extract`).

### update instructions
Should be set in env variable `XXXXX_UPDATE_INSTRUCTIONS`.

### more minor
#### createBin and the bash script
`this.createBin(version);` does what?
`version` is the update-to (read from manifest)

The createBin creates a bash file for the new version (updated).
The reason for bash script is to set env varibles so tht it can run as `reexec`.
also (needs confirmation).
(Why reexec reinstalls?)
The envs includes `XXXXX_BINPATH`, etc.
where `XXXXX` is the name of your command.

* `XXXXX_BINPATH`
* `XXXXX_REDIRECTED`

* The bash file is usually in `~/.local/share/xxxxx/client/bin/xxxxx` (.clientBin)
* `~/.local/share/xxxxx/client/current` (.clientRoot + '/current')
* Aa symbolic link `~/.local/share/xxxxx/client/current` points to ->   `./9.0.0`
#### Bash script
See [The createBin](#createBin)

#### Templates
See [Templates](#templates-1)

### The `baseDir`
* `baseDir` aka `basename`
* Used during extraction.

### s3Key
The `this.config.s3Key()` provides importanat key ids where info is taken fromm `this`, and customised (from manifest, etc)
* `.s3Key('baseDir')` provides the baseDir.  Is overriden by `manifest`.baseDir (directly ready as a field from fetched manifest file).
If not in manifest file, it is reconstructed form other fields of manifest such as: (`.version`, `.channel`) (and bin(why?)platform,arch).
* In generaal, its intpus are: `type` ( = the main `key`), then from the goven object: `.platform`, the templtes['target'][key].
* See templates
* s3Key(key) uses the template from the given key: templates[key].
### reexec
The reexec, spawns the `xxxxx update` after the bash script is created (using `createBin()`).

## Templates
* Templtes is a set of templates (strings)
* Can be customised in package.json: ...`.s3.templates`
* Two sets of templtes exist: `target` and `vanilla`.
* The default tempaltes are the following:

  *  baseDir: `'<%- bin %>'`
  *  unversioned: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- bin %>-<%- platform %>-<%- arch %><%- ext %>"`
  *  versioned: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- bin %>-v<%- version %>/<%- bin %>-v<%- version %>-<%- platform %>-<%- arch %><%- ext %>"`
  *  manifest: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- platform %>-<%- arch %>"`

##### Individual templates:
* `baseDir`: -- see baseDir
*  `manifest`: -- see manifest
*  `versioned`: -- is the URL for the updated tarball. Bypassed by manifest's `.gz` field.
*  `unversioned`: ?
#### target template
#### vanilla template
#### target


### Is update needed?
is cheches in maany places:
* The plugin-update / init hook: -- `autoupdateNeeded()` ius called plugins/update/hook/inside init.
* ...

## Requirements for update
* In `package.json`
```
       .version = \"$FLVER\"
       |
       .oclif.npmRegistry = \"http://127.0.0.1:3000/oclifRegistry\"
       |
       .oclif.update.s3.host = \"http://127.0.0.1:3000/upd\"
       |
       .oclif[\"warn-if-update-available\"].timeoutInDays = -1
       |
       .oclif[\"warn-if-update-available\"].registry = \"http://127.0.0.1:3000/wiua\"
       |
       .engines.node = \">=8.16.0\"
```
* A manifest file, usually called `darwin-x64`:
```
{
      version: "${VERSION_UPDATE_TO}"
      gz: "http://127.0.0.1:3000/upd/xxxxx-v${VERSION_UPDATE_TO}/xxxxx-v${VERSION_UPDATE_TO}-darwin-x64.tar.gz"
      baseDir: "xxxxx"
      sha256gz: "$(cat ....tar.gz | openssl dgst -sha256)"
}
```
* Manifest file to be served in the URL specified by `package.json` `.oclif.update.s3.host` (?). e.g. `http://127.0.0.1:3000/upd/darwin-x64`

## File structure
### File structure created by oclif-dev
The clif-dev itself is at: `"node_modules/@oclif/dev-cli/bin/run`

Creates 
* `./tmp`
* `./dist`
#### In `tmp`:
   * `xxxxx-v6.0.7` -- a folder
   * `tmp/xxxxx/oclif.manifest.json` -- a manifest file:
#### In `dist`:
```text
├── xxxxx-v6.0.7
│   ├── xxxxx-v6.0.7-darwin-x64.tar.gz
│   ├── xxxxx-v6.0.7-linux-arm.tar.gz
│   ├── xxxxx-v6.0.7-linux-x64.tar.gz
│   ├── xxxxx-v6.0.7-win32-x64.tar.gz
│   ├── xxxxx-v6.0.7-win32-x86.tar.gz
│   └── xxxxx-v6.0.7.tar.gz
├── darwin-x64
├── linux-arm
├── linux-x64
├── version
├── win32-x64
└── win32-x86
```
#### Target suffixes:
* `-darwin-x64.tar.gz`
* `-linux-arm.tar.gz`
* `-linux-x64.tar.gz`
* `-win32-x64.tar.gz`
* `-win32-x86.tar.gz`
* `.tar.gz`  -- for node

### File structure created by installer (linux)
