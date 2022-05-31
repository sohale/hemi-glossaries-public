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

### hooks, command, plugins
* Hooks, commands, plugins
* Each command is like a plugin.
* Each plugin is like a commmand.
* Each plugin (command?) has a set of hooks
* Each command has a set of hooks

* The structure of a plugin is very similar to the structure of commands: both have `run()`. [See](https://github.com/oclif/plugin-update/tree/main/src).

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
* See channel. (read from contents of manifest)

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
Channel is a feature of (a potential?) update.

`this.channel`. Usually = `stable`

The channel field is (often) read from the manifest contents.
"channel": "stable",

But it can be 
  * The `channel` of update is from the manifest:  `manifest.channel`. Is read from contents of manifest.
  * Can be the argument of update command: `xxxxx update stable`
  * Where is it read when "check for update" (`warn-if-update-available`)

Values of channel
* `stable`
* ...

#### tidy, touch
what does it do? `this.tidy();`

* `tidy` and `touch` are relted tools.
* `.tidy()` removes files older than a certain time.
* `.touch()` updates the timestamp of the file, so that it is not deleted in the next `tidy()`

### Internal configs
The `config` & The `pjson`
* The config fields
  * `config.binPath` -- If undefined, it does skipUpdate => `'not updatable'`. How is it set?
  * `this.config.version` The "from" version: Updating CLI from `this.config.version` to `manifest.{version, channel}`
  * `this.config.name` is the application name, here: `xxxxx`
  * The `channel` of update is from the manifest:  `manifest.channel`. See section for channel
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

## Update
1. Preparing files for installer
2. Prepared files for uploading into updater
3. Check if update is available (`warn-if-update-available`)
4. Update command (default: channel=`stable`)
5. Update command from a specific channel

### Is update needed?
is checks in many places:
* The plugin-update / init hook: -- `autoupdateNeeded()` ius called plugins/update/hook/inside init.
* ...

### Requirements for update
* In `package.json` (use jq)
```
       .version = "6.0.7"
       .oclif.npmRegistry = "http://127.0.0.1:3000/oclifRegistry"
       .oclif.update.s3.host = "http://127.0.0.1:3000/upd"
       .oclif["warn-if-update-available"].timeoutInDays = -1
       .oclif["warn-if-update-available"].registry = "http://127.0.0.1:3000/wiua"
       .engines.node = ">=8.16.0"
```
* A manifest file, usually called `darwin-x64`:
```
{
      version: "${VERSION_UPDATE_TO}"
      gz: "http://127.0.0.1:3000/upd/xxxxx-v${VERSION_UPDATE_TO}/xxxxx-v${VERSION_UPDATE_TO}-darwin-x64.tar.gz"
      baseDir: "xxxxx"
      sha256gz: "$(cat ....tar.gz | openssl dgst -sha256)"
      
      "channel": "stable"        /* must have */
}
```
* Manifest file to be served in the URL specified by `package.json` `.oclif.update.s3.host` (?). e.g. `http://127.0.0.1:3000/upd/darwin-x64`

## File structure
### File structure created by oclif-dev
The clif-dev itself is at: `"node_modules/@oclif/dev-cli/bin/run`

Creates 
* `./tmp`
* `./dist`
#### In `tmp`
* `xxxxx-v6.0.7` -- a folder
* `tmp/xxxxx/oclif.manifest.json` -- a manifest file:

```text
tmp
├── xxxxx
│   ├── oclif.manifest.json
│   ├── package-lock.json
│   ├── package.json
│   └── src  ...
├── darwin-x64 /xxxxx /...
├── linux-arm /xxxxx /...
├── linux-x64 /xxxxx ...
├── win32-x64 /xxxxx / ...
├── win32-x86 /xxxxx / ...
│
│
├── cache
│   ├── 12.22.12 / SHASUMS256.txt.asc
│   ├── node-v12.22.12-darwin-x64             ?
│   ├── node-v12.22.12-linux-armv7l
│   ├── node-v12.22.12-linux-x64
│   ├── node-v12.22.12-win32-x64.exe
│   └── node-v12.22.12-win32-x86.exe
├── node
│   ├── node-v12.22.12-darwin-x64 / ... (node, npm, npx)
│   ├── node-v12.22.12-linux-armv7l / ... (node, npm, npx)
│   ├── node-v12.22.12-linux-x64 / ... (node, npm, npx)
│   ├── node-v12.22.12-win-x64 / ... (node, npm, npx)
│   ├── node-v12.22.12-win-x86 / .... (node, npm, npx)
│   ├── node-v12.22.12-darwin-x64.tar.xz
│   ├── node-v12.22.12-linux-armv7l.tar.xz
│   ├── node-v12.22.12-linux-x64.tar.xz
│   ├── node-v12.22.12-win-x64.7z
│   └── node-v12.22.12-win-x86.7z
└─.
```
* 12.22.12 is the node version
* v6.0.7 is your xxxxx applcation version
* As of olcif version `"@oclif/dev-cli": "^1.22.2"`, `"@oclif/command": "^1.5.19"`, `"@oclif/plugin-update": "1.3.9"`.
* The file `oclif.manifest.json` (in the root) is ?
#### In `dist`
Manifest files in its root and `.tar.gz` files in a folder:
```text
dist
├── xxxxx-v6.0.7
│   ├── xxxxx-v6.0.7-darwin-x64.tar.gz
│   ├── xxxxx-v6.0.7-linux-arm.tar.gz
│   ├── xxxxx-v6.0.7-linux-x64.tar.gz
│   ├── xxxxx-v6.0.7-win32-x64.tar.gz
│   ├── xxxxx-v6.0.7-win32-x86.tar.gz
│   └── xxxxx-v6.0.7.tar.gz
│
│        /* manifest files: */
│
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

## oclif-dev
### Updating readme using `oclif-dev`
Readme can be generated using `oclif-dev readme` in the folder.
It also seeks the["warn-if-update-available"].registry URL on server wiua/xxxxx
The `README.md` should contian   `<!-- usage -->` and `<!-- commands -->` oethrwise nothing is generated.
See [doc1](https://github.com/oclif/dev-cli/blob/main/README.md#oclif-dev-readme) and [doc2](https://github.com/oclif/dev-cli/blob/47fb493c523bd417d39cb83aecd4aac1c6bb505b/src/commands/readme.ts#L19).

### manifest using `oclif-dev`
The command `oclif-dev manifest` can be used.
However, it seems to not to create the manifest file as expected.
The manifest files created by `oclif-dev  pack` are different.

### How to run `oclif-dev`
You can run it using `node_modules/@oclif/dev-cli/bin/run`
