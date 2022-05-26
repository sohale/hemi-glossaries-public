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

### client dir
* The clientDir or `clientRoot` is `~/.local/share/flash/client`.
* It is cleaned (wiped) before an update. (by `ensureClientDir()`)
* Output of the extraction stream is `.clientRoot`/`maifest.version` for example `~/.local/share/flash/client/6.0.7`

### Stream
* It uses `stream` for downloading the update. Not promises, etc.
* Strems can be `pause()`ed or `stream.resume()`ed from downstream. (downstrem controls the dataflow)
* Streams can be redirected to `extract()`.
* You can `await` the downstream that you attached (eg the `extract`).


#### The createBin
`this.createBin(version);` does what?
`version` is the update-to (read from manifest)

#### The `baseDir`
* `baseDir` aka `basename`
* Used during extraction.

#### s3Key
The `this.config.s3Key()` provides importanat key ids where info is taken fromm `this`, and customised (from manifest, etc)
* `.s3Key('baseDir')` provides the baseDir.  Is overriden by `manifest`.baseDir (directly ready as a field from fetched manifest file).
If not in manifest file, it is reconstructed form other fields of manifest such as: (`.version`, `.channel`) (and bin(why?)platform,arch).
* In generaal, its intpus are: `type` ( = the main `key`), then from the goven object: `.platform`, the templtes['target'][key].
* See templates
* s3Key(key) uses the template from the given key: templates[key].

#### Templates
* Templtes is a set of templates (strings)
* Can be customised in package.json: ...`.s3.templates`
* Two sets of templtes exist: `target` and `vanilla`.
* The default tempaltes are the following:

  *  baseDir: `'<%- bin %>'`
  *  unversioned: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- bin %>-<%- platform %>-<%- arch %><%- ext %>"`
  *  versioned: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- bin %>-v<%- version %>/<%- bin %>-v<%- version %>-<%- platform %>-<%- arch %><%- ext %>"`
  *  manifest: `"<%- channel === 'stable' ? '' : 'channels/' + channel + '/' %><%- platform %>-<%- arch %>"`

##### Individuaaal templates:
* baseDir -- see baseDir
*  manifest -- see manifest
*  versioned ?
*  unversioned ?
#### target template
#### vanilla template
#### target
