# Strange Customs' Horns

The game knows two different systems: Whistles and horns. Strange Customs adds the ability to add custom horns by making a Railloader mod.

# For Users

Horn packs are identical to Railloader mods. They are installed into their own folder inside /Mods/ and follow Railloader's general mod loading rules,
including dependencies and inverse-dependencies.


# For Developers

## Definition.json
As mentioned before, horns patches are normal Railloader mods, and as such, are defined like them. An example, minimal Definition.json for the mod could look like this:

```json
{
	"manifestVersion": 4,
	"id": "MyMod.CustomHorn",
	"name": "My Custom Horn",
	"version": "1.0",
	"requires": [ "Zamu.StrangeCustoms" ],

	"mixintos": {
		"horns": "file(my-horns.json)"
	}
}
```

Things to note:

1. The `manifestVersion` should be 4 or higher, to make sure it's a recently enough Railloader version. Earlier versions of Railloader will load the mod, but not
   really process it.
1. The `requires` makes sure that this mod can only be enabled if Strange Customs is also available.
1. The `mixintos` statement is where you can define what file you wish to use to patch what other file.

For this purpose, Strange Customs defines an alias which is called "horns".

Currently, it is required to specify the value of the mixinto in the `file()` pattern, where the inside of the "function" defines the path, relative to the Definition.json.
A `file()` may not lead to an outside folder; it's therefore not allowed to load anything outside of the mods' own folder.

## The horns JSON
The file referenced in the manifest is a horns container file, which contains the list of horns to be added. It is a JSON array which contains objects and looks like this:

```json
[
	{
		"name": "Toot Toot",
		"layers": [
			{
				"file": "file(layer0.ogg)",
				"keyframes": [
					{ "t": 0, "value": 1 },
					{ "t": 1, "value": 0 },
				]
			},
			{
				"file": "file(layer1.ogg)",
				"keyframes": [
					{ "t": 1, "value": 0 },
					{ "t": 0, "value": 1 }
				]
			}
		]
	}
]
```

The array contains objects which have the following structure:

- **name** is the name of the horn. This is displayed in the UI and used to synchronize the horns across the network.
- **layers** is an array that must contain exactly 2 layer objects. Each horn can have two layers, for example high horn/low horn, which are then interpolated between when quilling it.
  - **file** is the file reference to use for the horn. This must be a file() reference.
  - **keyframes** is an array that defines a curve to determine the volume of a source according to the "quilling" parameter.
	- **t** is the "quilling factor", which goes from 0 to 1.
	- **value** is the volume, from 0 to 1.

The above example defines therefore a horn called "Toot Toot", which uses layer0.ogg for low-quilling and layer1.ogg for high-quiling.

## Supported Audio Formats

The list of supported formats is a bit opaque and I don't think Unity is giving you a list. [There's this enum of supported formats](https://docs.unity3d.com/ScriptReference/AudioType.html), but from my experience it's a best-case scenario.

What definitely does work, however, are OGG/Vorbis files. They achieve decent compression while being loadable on almost all platforms. Your mileage may vary, feel free to experiment.

## Console Commands
Because restarting is a bit of a pain, it's possible to re-load all horn definitions (client-sided) with `/sc-horns-reload`. This will re-read all JSONs, and apply all horns, making the changes live-ish in your game.
Note that if an error occurs, i.e. an invalid JSON, a file that cannot be loaded or similar, then the only place you can see this kind of information would be the railloader.log and/or the console (when starting the game with `--console`).