# Observations on how `oclif` works
Including OCLIF internls

https://github.com/oclif/oclif

## Main concepts:
### plugins
There are three types of plugins
* UserPlugins
* DevPlugins
* CorePlugins

### Hook
What is a hook?

`@oclif/plugin-plugins/lib/hooks/update.js` is outside the update plugin.

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
  * `FLASH_UPDATE_INSTRUCTIONS` -- Used at onset of update. ...

  * `FLASH_CACHE_DIR`
  * `FLASH_CONFIG_DIR`
  * `FLASH_DATA_DIR`
  * `FLASH_BINPATH`
  * `FLASH_NPM_REGISTRY`
  * `FLASH_S3_BUCKET`
  * `FLASH_OCLIF_CLIENT_HOME`
  * `FLASH_UPDATE_INSTRUCTIONS`

### Other minor concepts
#### channel
`this.channel`. Usually = `stable`
#### tidy
what does it do? `this.tidy();`

### Internal configs
The `config` & The `pjson`
* The config fields
  * `config.binPath` -- If undefined, it does skipUpdate => `'not updatable'`. How is it set?
  * `this.config.version` Updating CLI from `this.config.version` to `manifest.(version, channel)`

* The `pjson` fields

### local cache
* The `.local/share` is used for what?
   * `~/.local/share/flash/client/6.0.7` is the output of update fter downloading.

### Stream
* It uses `stream` for downloading the update. Not promises, etc.
* Strems can be `pause()`ed.
* Streams can be resirected to `extract()`.
