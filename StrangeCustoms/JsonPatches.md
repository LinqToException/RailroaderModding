# Strange Custom's JSON Patches

Strange Customs 1.3 introduced _JSON patching_ or _container patching_, allowing you to patch the Definitions.json without having to overwrite the entire file.
This feature uses Railloader's new _mixinto_ definition, which allows mods to define entries, or files, that can be queried by other mods as a "feature" of some sort.

# For Users

JSON patches are identical to Railloader mods. They are installed into their own folder inside /Mods/ and followed Railloader's general mod loading rules,
including dependencies and inverse-dependencies.

Please note that faulty content mods can cause the game to behave erratically. The faulty mod will not be marked as faulted by Railloader, and information about
what went wrong and where may only be found in the railloader.log.

# For Developers

There is a quick-and-somewhat-dirty console command for developers to verify and/or "reload" their patches:

```
/sc-patches <mixinto_name> [print|reload]
```

The first parameter is the name of the mixinto to reload, e.g. `container:ls-260-g16`. The second parameter is optional and one of three possibilities:

- (none): Will re-parse the patch files for this mixinto, but do nothing with it. If there is an error, it will be written to the game log.
- `print`: Will re-parse the patch files for this mixinto, and print the final JSON into the game log. You can use this to verify whether your changes are applied correctly.
- `reload`: Will re-parse the patch files for this mixinto, and attempt to "hot reload" the locomotive. Note that this is very limited in its functionality; while components usually 
  work fine, there are things that cannot be reloaded without re-starting the game. Use on your own risk. If the reload works, it will only affect newly spawned locomotives; not existing ones.

## Definition.json
As mentioned before, JSON patches are normal Railloader mods, and as such, are defined like them. An example, minimal Definition.json for the mod could look like this:

```json
{
	"manifestVersion": 3,
	"id": "CheapFlatcars",
	"name": "Cheap Flatcars",
	"version": "1.0",
	"requires": [ "Zamu.StrangeCustoms" ],

	"mixintos": {
		"container:fl-skeleton01": "file(skeleton.json)"
	}
}
```

Things to note:

1. The `manifestVersion` should be 3 or higher, to make sure it's a recently enough Railloader version. Earlier versions of Railloader will load the mod, but not
   really process it.
1. The `requires` makes sure that this mod can only be enabled if Strange Customs is also available.
1. The `mixintos` statement is where you can define what file you wish to use to patch what other file.

For this purpose, Strange Customs defines an alias which maps to the resource name and is prefixed with `container:`. The resource name corresponds to the folder name
that the asset was found in, e.g. `/Railroader_Data/StreamingAssets/AssetPacks/ls-060-s23/Definitions.json` maps to `container:ls-060-s23`.

Currently, it is required to specify the value of the mixinto in the `file()` pattern, where the inside of the "function" defines the path, relative to the Definition.json.
A `file()` may not lead to an outside folder; it's therefore not allowed to load anything outside of the mods' own folder.

Important to note is that Railroader's Definitions.json (as the name implies) can contain multiple definitions. For example, steam engines and their tenders are usually
kept in the same container and therefore the same Definitions.json. Targetting these with patches will require a `$find` instruction.

## Patch file
Patch files are relatively simple and allow for most modifications. Generally speaking, they mimick the game's Definitions.json. The rules by which they are applied
are fairly simple.

### Normal properties and assignments
Properties are followed through and will set the values as you would expect:

```json
{
	"definition": {
		"basePrice": 1500
	}
}
```

This code snippet will set `Definition.BasePrice` to `1500`. If definition, or base price, does not exist they will be created.

### Patch Instruction Object

Strange Customs supports some "magic" properties, which are prefixed with a dollar sign (`$`), and act as instructions for it to do something other than just assignments.
All these instructions must be placed inside an object. In JSON, they look like the following:

```json
{
	"$find": [ { "path": ..., "value": ... } ],
	"$clone": false,
	"$add": ...,
	"$remove": true,
	"$replace": true,
	"$optional": false
}
```

#### `$find`

`$find` can only be used in array elements and can be used to specify for which element the contained instructions should be used for. You will usually find one
at the very beginning of a patch to find the right definition:

```json
{
	"objects": [
		{
			"$find": [ 
				{ 
					"path": "identifier",
					"value": "pb-osgbrad-lightweight-steel-1915",
					"comp": "equals"
				}
			],

			"definition": {
				"basePrice": 125000
			}
		}
	]
}
```

The snippet above finds the entry whose `identifier` property is equal to `"pb-osgbrad-lightweight-steel-1915"`. A few key points:

- `$find` expects an array-of-instructions. An item is only found if **all** instructions match. This can be used to further fine-tune searches by homing into an explicit part.
- `path` is a [JSON.NET path](https://www.newtonsoft.com/json/help/html/SelectToken.htm) and can deal with nested properties. For example, `"definition.kind"` would target the definition's kind property.
- `value` is compared by value or, if they are enumerable, by sequence. It therefore works for simple values and arrays, but probably not for objects.
- `comp` can be used to define the kind of operation to perform. Currently supported are `equals`, `notEquals` for all types including collections; and `startsWith`, `endsWith`, and `contains` for strings. If not specified, `equals` is assumed.
- If the element cannot be found, an exception is thrown and the rest of the patch is skipped.
- The `$optional` instruction can be used to make find failures acceptable. If the item cannot be found, a warning is logged, but the patch itself is applied.

#### `$clone`

`$clone` is only used together with `$find` in array elements. It creates a complete copy of the found array element, adds it to the end of the array, and then applies
whatever is inside the object to the newly created object. This can be used if you want to copy an existing (more complex) structure, but only want to modify a few things.

```json
{
    "objects": [
		{
			"$find": [ { "path": "identifier", "value": "ls-260-g16" } ],
			"$clone": true,
			"identifier": "ls-260-g16-rtx",
			"metadata": {
				"name": "G-16 RTX"
			},
			"definition": {
				"tenderIdentifier": "lt-260-g16-rtx",
			}
		},
		{
			"$find": [ { "path": "identifier", "value": "lt-260-g16" } ],
			"$clone": true,
			"identifier": "lt-260-g16-rtx",
			"metadata": {
				"name": "G-16 RTX"
			},
			"definition": {
				"loadSlots": [
					{
						"$find": [ { "path": "requiredLoadIdentifier", "value": "coal" } ],
						"maximumCapacity": 24000
					},
					{
						"$find": [ { "path": "requiredLoadIdentifier", "value": "water" }],
						"maximumCapacity": 8000
					}
				]
			}
		}
	]
}
```

This example does the following:

1. Finds the definition for the locomotive, clones it, changes the identifier, name and used tender on the newly duplicated item.
2. Finds the definition for the tender, clones it, changes the identifier, name, and capacity to be twice the default values.


#### `$remove`

`$remove` can be used to specify that the element should be removed. When used for array elements, it must be used with `$find`. It can also be used for normal
properties, however. Keep in mind that removing elements is usually a bad idea, and should only be used if you are sure what you are doing. The game does not take
kindly to expected, but missing data.

```json
{
	"objects": [
		{
			"$find": [ { "path": "identifier", "value": "ls-280-c25" } ],

			"definition": {
				"airhosePosition": { "$remove": true },
				"wheelsets": [
					{
						"$find": [ { "path": "animation.clipName", "value": "Pilot" } ],
						"$remove": true
					}
				]
			}
		}
	]
}
```

Removes the wheelsets and airhosePosition whose animation's clipName is "Pilot".

#### `$replace`
`$replace` will replace the current object with the value of the property. This can be used to quickly replace more complex structures.
`$replace` should therefore be used with care and, if possible, not at all.

```json
{
	"objects": [
		{
			"$find": [ { "path": "identifier", "value": "ls-280-c25" } ],

			"definition": {
				"airHosePosition": {
					"$replace": [ 1, 2, -1 ]
				},
			},

			"components": [
				{ 
					"$find": [ { "path": "kind", "value": "ToggleAnimation" }, { "path": "name", "value": "Engineer Door" } ],
					"transform": {
						"$replace": {
							"position": [ -1, 2, 3 ],
							"rotation": [ 0, 0, 0, 0 ],
							"scale": [ 1, 1, 2 ]
						}
					}
				}
			]
		}
	]
}
```

Replaces the airhosePosition of the locomotive, and also replaces the entire transform of the Engineer Door component.

#### `$add`
`$add` can only be used in arrays, and will add the content of the property to the array at the end.

```json
{
	"objects": [
		{
			"$find": [ { "path": "identifier", "value": "ls-280-c25" } ],
			"components": [
				{ 
					"$add": {
						"kind": "ToggleAnimation",
						"name": "New Animation",
						"transform": {
							"position": [ -1, 2, 3 ],
							"rotation": [ 0, 0, 0, 0 ],
							"scale": [ 1, 1, 2 ]
						}
					}
				}
			]
		}
	]
}
```


#### `$optional`
`$optional` is a bool that is only evaluated in conjunction with `$find`. When it is defined and `true`, then the patch will not fail if a `$find` instruction could not be found.
This is useful if you want to, for example, share a patch across multiple files, where it may not exist in one form or another.

```json
{
	"objects": [
		{
			"$find": [ { "path": "identifier", "value": "ls-404-unavailable" } ],
			"$optional": true,
			"metadata": {
				"name": "But I still haven't found what I'm looking for"
			}
		}
	]
}
```

#### Arrays
Arrays are handled a bit special: Each element of an array must be an object, and each of those objects must be a special instruction. The following are some often
used combinations:

- `$find` and `$replace` or `$remove`: Find an element from the array and remove or replace it. Using `$optional`, you can write patches that _may_ fail. In case the element isn't found, the instruction is silently skipped.
- `$add`: Add a new element to the array.