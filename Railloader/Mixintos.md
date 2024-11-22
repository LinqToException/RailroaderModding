# Mixintos

Mixintos are Railloader's system to allow mods to provide data for other mods to consume. Mixintos are defined in the _Definition.json_ and can be either references to files, or references to directories.

Railloader performs validation on whether the entries exist before returning them to a requester, but will not verify the contents (because they could be anything and RL doesn't really care).

## Providing Mixintos
Mixintos can either be defined statically in the _Definition.json_ (`file()`, `dir()`) or dynamically from within a script mod (`IMixintoDefinitionProvider`).

For static mixintos, two options are available: The "simple" mixintos, and the "advanced" ones:

```json
{
  "mixintos": {
    "some-mixinto-type": [
      // Simple mixintos: Simply defining them as a string. No configuration possible.
      "file(my_file.json)",
      "dir(my_dir)",
      // Advanced mixintos: Including configuration
      {
        // Type is determined automatically based on file()/dir() usage
        "mixinto": "file(my_file.json)",
        // "requires" can be used to emit this entry only if *all* conditions are available.
        "requires": [ "SomeMod", { "id": "SomeOtherMod", "notBefore": "2.0" }],
        // "conflictsWith" defines that this mixinto should not be returned if *any* match
        "conflictsWith": [ "SomeBadMod", { "id": "SomeOutdatedMod", "notAfter": "3.5" }]
      }
    ]
  }
}
```

`requires` and `conflictsWith` are similar to the mod loading, with the exception of order: While `requires` changes the mod loading order when used in the _Definition.json_, it does only change whether the mixinto is returned or not when used in the mixintos block. Similarly, `conflictsWith` does not cause an error, but merely ignores the mixinto.

This can be used to provide conditional mixintos (i.e. "if that other mod is available, use this mixinto too in order to fix some issues that may arise from both mods being loaded simultaneously").

### `file(...)`
In order to provide a file as mixinto, use `file(path_to_file_relative_to_mod's_directory)`. Railloader will make sure that the file exists and is not outside the mod's directory (for safety reasons).

### `dir(...)`
Similar to `file()`, except it refers to a directory (relative to the mod's directory) instead. Railloader will make sure that the directory exists and is not outside the mod's directory (for safety reasons).

### `IMixintoDefinitionProvider`
Types that implement `PluginBase` and `IMixintoDefinitionProvider` and have been defined in an assembly that has been loaded by RL by definining it with `"assemblies": [ ... ]` in the _Definition.json_ can provide mixintos dynamically on-demand as an enumerator. Each time a mixinto is requested, the method will be called.

If `Requires` or `ConflictsWith` is set, Railloader will perform the necessary validation and filter out results as-needed. Therefore, it is not necessary to perform dependency checks in `IMixintoDefinitionProvider` - simply return properly configured `MixintoDefinition`s instead.

## Consuming Mixintos

On the `IModdingContext` that is injectable in your `PluginBase` inherited class, calling `GetMixintos(string)` will on-demand evaluate all Definition.jsons and `IMixintoDefinitionProvider`s and return them in the mod loading order. It is up to the caller to further verify whether the mixintos are good (e.g. by checking the type, file extension, or further validation). Railloader will guarantee that entries returned are existing files/directories, are inside the respective mod's folder, have been loaded and do not conflict/have all requirements met (for both the mod and the mixinto).