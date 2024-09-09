# Strange Customs' Graph

**This is a work in progress documentation.**

Strange Customs, using the same idea as for car JSON patches (see JsonPatches.md). They currently allow modders to (**C**reate, **R**ead, **U**pdate, **D**elete):

- Track Layouts (Nodes, Segments, Spans) [CRUD]
- Areas [\_RU\_]
- Industries [CRUD]
- Industry Components [CRUD] with some limitations:
	- IndustryLoaderBase [CRUD]
		- IndustryLoader [CRUD]
		- TeleportLoadingIndustry [CRUD]
	- Interchange [CRUD]
	- InterchangedIndustryLoader [CRUD]
	- FormulaicIndustry [CRUD]
	- IndustryUnloader [CRUD]
	- ProgressionIndustry [\_\_\_\_] (no special configuration)
	- TeamTrack [\_\_\_\_] (not implemented yet)
- Loads [CRU\_]
- Scenery [CRUD] (UD only for specifically asset-based scenery and mod-spawned scenery)
- Labels on the map and signposts [\_RU\_]

As a rule of thumb: Strange Customs supports _at most_ whatever it dumps (when enabled in the options). If it isn't dumped, it's not recognized or supported.

As another rule of thumb, Strange Customs will _not_ validate your JSON or modifications you make. If something goes wrong, the graph can end up in an invalid state, which can lead to non-patched, partially-patched or badly-patched game states. It is usually not possible to figure out which mod is responsible for said bad patch.

## General

- Uses the "game-graph" mixinto and Strange Customs' JSON patching approach to modify the graph.
- You can dump the current graph by enabling it in the options, then reloading a save-game.
- Also available is an experimental auto-reload, which will cause any changes to any map mixinto to force the game to reload all map mixintos.
- All positions are metric. x/z are the plane (right/forward); y is up. They are defined as objects with x/y/z attributes.
- All rotations are vectors of euler angles in local space (x is pitch, y is yaw, z is roll). You want x for uppy-downy, y for lefty-righty, and z to a barrel roll. This is different from the game's Definitions.json, which seems to use Quaternions instead.
- Whenever an object is used, the key is usually the identifier.
- Whenever a property is set to `null`, it is assumed that this identifier should be deleted, if it exists.
- 
## Tracks

The game's tracks are represented as a graph of Bézier curves, which consists of **Nodes**, **Segments** and **TrackSpans**, where **Locations** are also relevant.

- **Nodes** are the foundation of the network. They have an ID, a position and a rotation.
- **Segments** represent tracks. They have an ID, a start and end node, and some more metadata.
- **TrackSegments** represent a piece of track that can span across multiple segments. If you can highlight something in-game on the tracks (e.g. industry tracks), those are spans. They are defined by an ID and two locations, an upper an lower. Segments must be normalized manually, i.e. the two locations must face in the correct way. **Experimental:** You can define `"normalize": true` to auto-normalize TrackSpans. This may not work with spans on/across added segments.
- **Locations** are both a position on the track and a facing direction. They are defined by a segment, an end that they are relative to, and the distance to that end. While it's possible that two locations would represent the same position, the direction would be different. As example, for a segment of length 100, the locations [start + 25] and [end + 75] would be at the same position, but face in opposite directions.

The game has some implicit rules:

- The end node's rotation defines the segment's curve.
- It does not matter which node is start or end. The game will figure out on its own which way is correct.
- At most two segments can leave in the same direction. It's not really possible, with the base game, to have three-way switches or similar. Turntables (not supported) work because they work their own logic into the network.
- Switches must be planar to work. If the mesh generator can't figure out the intersection between two rails to generate the junction mesh, it will not render the junction.
- Every node that has only one segment attached to it implicitly gets a buffer stop.
- If a segment has a track group, it is considered part of an MapFeature unlock and therefore unavailable by default.
- TrackSpans can be re-used. Keep in mind that another mod may change the span though, which could affect your mod as well.

To change them, simply access the corresponding object.

- Property changes can simply be done by mimicking the path to be adjusted.
- The key is the ID of the node/segment/span. Those MUST be unique across the game and all mods. The game itself auto-generates those, where the first character usually is the type (N/S/P). You can name them whatever you want to, recommended would be to prefix them with the character, then your mod name, then some unique name in your mod.
- Setting a property to null will delete this node/segment. This will later mods that also wish to modify this path to error out.

## Industries

- Industries are compound objects that consist of one or more components.
- Each industry belongs to an area, which may also define when it is unlocked (at the time the area is unlocked, unless a map feature says otherwise).
- Not all components are equally supported by Strange Customs at the moment.
- If an industry is contract-based, values defined by components represent the highest-tier values. Lower tiers are scaled down.
- The property key is the id of the industry and must be unqiue across all areas and mods. 
- To add an industry, simply define a new key. For best effect, copy an existing industry and tweak it.

### General for Components
- The key defines the id of the component. This must be unique within the industry, but is fine to be used within the game.
- `trackSpans` is an array that can be manipulated with operations like `"trackSpans": [ { "$add": "P1234" } ]` or `$find`/`$remove`, or just flat-out `$replace`. Check the JsonPatches.md for further instructions.
- `trackSpans` is usually used to determine some logic such as "what cars are spotted at the component right now", and may affect logistics operations.
- Must have a valid `"type"` property and fitting metadata. Check the dump for advice on how to do so.
- `carTypeFilter` is a wildcard-esque construct to define what cars are accepted for this industry, based on the car type. Check the dump for what is allowed. Usually it's wildcards and comma-separated-values.

### Formulaic Industries
- Usually no `trackSpans` because it does not directly interact with cars.
- Requires other industries to be present.
- `inputTermsPerDay` defines the maximum amount of goods that the factory will accept from attached loaders. This is in units/day, depending on the load.
- `outputTermsPerDay` defines the maximum amount of goods that the factory will throw into attached unloaders. This is in units/day, depending on the load.
- The conversion is capped by the minimum of any input and the minimum available storage of any output.

### Loaders
- `storageChangeRate` defines how many units/day this loader will produce per day "out of thin air". You can use this for resource sources (e.g. Connelly).
- `carTransferRate` is how many units/day the loader will put into car(s).
- `orderAroundEmpties` in this context is "Does it order empties from the interchange?"
- `orderAroundLoaded` in this context is "Does it send away fully loaded cars to the interchange?"

### Unloaders
- `storageChangeRate` defines how many units/day this unloader will void per day "into thin air". You can use this for resource drains (e.g. Bryson Coal & Lumber).
- `carTransferRate` is how many units/day the unloader will remove from car(s).
- `orderAroundEmpties` in this context is "Does it send empties away to the interchange?"
- `orderAroundLoaded` in this context is "Does it order fully loaded cars from the interchange?"

### Teleporters
- `inputSpans` and `outputSpans` are lists of trackspans used for input/output
- Similar to Robinson/Alarka's mines, cars will be teleported from one span to the other if a valid, non-occupied path can be found.
- `carLoadPeriod` defines the interval between two attempts at loading a car, in seconds.
- `carLengthFeet` is a bit weird, but used to find open space on the output spans.