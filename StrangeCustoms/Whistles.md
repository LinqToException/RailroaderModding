# Strange Customs' Whistles

The game knows two different systems: Whistles and horns. Strange Customs adds the ability to add custom whistles by using the file system, rather than bundled files.

# For Users

Whistle packs are identical to Railloader mods. They are installed into their own folder inside /Mods/ and follow Railloader's general mod loading rules,
including dependencies and inverse-dependencies.


# For Developers

## Definition.json
As mentioned before, horns patches are normal Railloader mods, and as such, are defined like them. An example, minimal Definition.json for the mod could look like this:

```json
{
  "manifestVersion": 5,
  "id": "MyMod.CustomWhistle",
  "name": "My Custom Whistle",
  "version": "1.0",
  "requires": [ "Zamu.StrangeCustoms" ],

  "mixintos": {
    "whistles": "file(my-whistles.json)"
  }
}
```

Things to note:

1. The `manifestVersion` should be 5 or higher, to make sure it's a recently enough Railloader version. Earlier versions of Railloader will load the mod, but not
   really process it.
1. The `requires` makes sure that this mod can only be enabled if Strange Customs is also available.
1. The `mixintos` statement is where you can define what file you wish to use to patch what other file. This uses the normal Railloader mixinto system.

For this purpose, Strange Customs defines an alias which is called "whistles".

Currently, it is required to specify the value of the mixinto in the `file()` pattern, where the inside of the "function" defines the path, relative to the Definition.json.
A `file()` may not lead to an outside folder; it's therefore not allowed to load anything outside of the mods' own folder.

## The whistles JSON
The file referenced in the manifest is a whistles container file, which contains the list of whistles to be added. It is a JSON array which contains objects and looks like this:

```json
[
  {
    "name": "Example Whiste",
    "clip": "file(horn.ogg)",
    "model": {
        "assetPackIdentifier": "audio.whistles01",
        "assetIdentifier": "5ChimeA"
    }
  }
]
```

The array contains objects which have the following structure:

- **name** is the name of the whistle. This is displayed in the UI and used to synchronize the whistles across the network.
- **clip** is a file reference to use as audio clip.
- **model** is an asset reference that defines which horn model will be used. For best results, take a look at the existing horns and copy one of them over. Custom
  model loading is currently not supported by Strange Customs and likely won't be.

The above example defines therefore a whistle called "Example Whistle", which uses "horn.ogg" for the audio and uses the 5ChimeA's model from the base game.

## Supported Audio Formats

The list of supported formats is a bit opaque and I don't think Unity is giving you a list. [There's this enum of supported formats](https://docs.unity3d.com/ScriptReference/AudioType.html), but from my experience it's a best-case scenario.

What definitely does work, however, are OGG/Vorbis files. They achieve decent compression while being loadable on almost all platforms. Your mileage may vary, feel free to experiment.

## Console Commands
Because restarting is a bit of a pain, it's possible to re-load all whistle definitions (client-sided) with `/sc-whistles-reload`. This will reload the whistle selection, but not apply changes you may have made. In order to see the effect
on existing locomotives, switch to another whistle then back again.
