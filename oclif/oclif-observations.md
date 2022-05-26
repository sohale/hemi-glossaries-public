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

### Internal configs
The `config` & The `pjson`
* The config fields
  * `config.binPath` -- If undefined, it does skipUpdate => `'not updatable'`. How is it set?

* The `pjson` fields
