# Strange Customs' Bells

Starting with 1.9.24326, Strange Customs added the ability to add custom bells into the game.

# For Users

Bells are distributed as normal Railloader mods. They are installed into their own folder inside _/Mods/_ and follow Railloader's general mod loading rules, including dependencies and inverse-dependencies.


To install a bell mod, simply drag it onto the Railloader.exe.

# For Developers

## Definition.json
As mentioned before, bells are normal Railloader mods, and as such, are defined like them. An example, minimal Definition.json for the mod could look like this:

```json
{
  "manifestVersion": 5,
  "id": "MyMod.CustomBell",
  "name": "My Custom Bell",
  "version": "1.0",
  "requires": [ { "id": "Zamu.StrangeCustoms", "notBefore": "1.9.24326" } ],

  "mixintos": {
    "bells": "file(my-bells.json)"
  }
}
```

The bells file is listing the available bells, similar to SC's horns:

```json
[
  {
    // The name of the bell; used in-game in the dropdown menu and internally for synchronization
    "name": "More COWBELL",

    // The file path to the bell, relative to the mod's directory
    "file": "file(cowbell.wav)",

    // Optional: An array of numbers defining when to cut the clip.
    "indexTimes": null
  }
]
```

`indexTimes` can be used if a single file contains multiple clips that can be used interchangeably. The array contains the file should be cut (in seconds) to produce new clips. Assuming you have 3 clips in a file, all 2.6 seconds long (the clips should have the same length roughly), then it would look something like `"indexTimes": [ 2.6, 5.2 ]`.