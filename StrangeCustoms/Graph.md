# Strange Customs' Graph

**This is a work in progress documentation.**

Strange Customs, using the same idea as for car JSON patches (see JsonPatches.md). They currently allow modders to (**C**reate, **R**ead, **U**pdate, **D**elete):

- Tracks
  - Nodes [CRUD]
  - Segments [CRUD]
  - Spans [CRUD] (do NOT delete spans that are used by non-SC-managed resources, such as team tracks/progressions/passenger stops)
- Areas [CRU\_]
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
  - TeamTrack [CRUD] (from 2024.6.5 onwards)
  - Generic IndustryComponents that implement `StrangeCustoms.Tracks.Industries.ICustomIndustryComponent` [CRUD]
- Loads [CRU\_]
- Scenery [CRUD] (UD only for specifically asset-based scenery and mod-spawned scenery)
- Labels on the map and signposts [\_RU\_]

As a rule of thumb: Strange Customs supports _at most_ whatever it dumps (when enabled in the options). If it isn't dumped, it's not recognized or supported.

As another rule of thumb, Strange Customs will _not_ validate your JSON or modifications you make. If something goes wrong, the graph can end up in an invalid state, which can lead to non-patched, partially-patched or badly-patched game states. It is usually not possible to figure out which mod is responsible for said bad patch.

## General

Uses the "game-graph" mixinto and Strange Customs' JSON patching approach to modify the graph.

```jsonc
// Definition.json
{
  "mixintos": {
    "game-graph": "file(my-changes.json)"
}
```

You can specify multiple JSONs; they will be loaded in the order they're defined as:

```jsonc
// Definition.json
{
  "mixintos": {
    "game-graph": [ "file(tracks.json)", "file(areas.json"), "file(scenery.json)" ]
}
```

In addition:
- You can dump the current graph by enabling it in the options, then reloading a save-game.
- Also available is an experimental auto-reload, which will cause any changes to any map mixinto to force the game to reload all map mixintos.
- All positions are metric. x/z are the plane (right/forward); y is up. They are defined as objects with x/y/z attributes.
- All rotations are vectors of euler angles in local space (x is pitch, y is yaw, z is roll). You want x for uppy-downy, y for lefty-righty, and z to do a barrel roll. This is different from the game's Definitions.json, which seems to use Quaternions (arrays with 4 numbers) instead.
- Whenever an object is used, the key is usually the identifier.
- Whenever a property is set to `null`, it is assumed that this identifier should be deleted, if it exists.
- Usually, SC will not allow mixing between arrays, objects, and values. An exception is null, which can be assigned to and from every object.

## Tracks

The game's tracks are represented as a graph of Bézier curves, which consists of **Nodes**, **Segments** and **TrackSpans**, where **Locations** are also relevant.

The game has some implicit rules:

- The end node's rotation defines the segment's curve. If you are familiar with Beziers: The rotation of the end nodes is used to determine the position of the control points (by projecting forwards from the rotation; the length is defined by the distance of the curve itself).
- At most two segments can leave in the same direction, and a node may have at most three segments attached to it. It's not really possible, with the base game, to have three-way switches, double-slip switches or similar. Turntables (not supported) work because they work their own logic into the network.
- Switches must be planar to work. If the mesh generator can't figure out the intersection between two rails to generate the junction mesh, it will not render the junction.
- Every node that has only one segment attached to it implicitly gets a buffer stop. The exception are nodes that have are part of a turntable; the turntable will override this behaviour.

To change them, simply access the corresponding object.

- Property changes can simply be done by mimicking the path to be adjusted.
- The key is the ID of the node/segment/span. Those MUST be unique across the game and all mods. The game itself auto-generates those, where the first character usually is the type (N/S/P). You can name them whatever you want to, recommended would be to prefix them with the character, then your mod name, then some unique name in your mod.
- Setting a property to null will delete this node/segment. This will later mods that also wish to modify this path to error out.
- 
### Nodes
**Nodes** are the foundation of the network. They have an ID, a position and a rotation. They mark a point in the world through which a piece of track can go.

```jsonc
{
  "tracks": {
    "nodes": {
      // The ID of the node, as referenced in the segments
      // The game usually prefixes those with 'N'. It's recommended to add an unique identiifer, starting with your name/mod name as abbreviation.
      "NSCExample_Sqf1": {
        // The position, in 3D space, absolute
        "position": { "x": 120, "y": 572, "z": -12000 },
        // The rotation, in Euler degrees. X is upsy-downsy; Y is lefty-righty; Z is doing a barrel roll.
        // You want some X for inclines; Y is your main man; and Z is if you want to have some super-elevation or banking or whatever.
        // Unity, by default, likes to have these values between -180 and 180, but it does not really matter that much.
        "rotation": { "x": 0, "y": 45, "z": 0 },
        // If this node is part of a switch, and the location of the switch stand is unfavorable, you can switch it here.
        "flipSwitchStand": false
      }
  }
}
```

Nodes themselves have no graphical presentation, and are usually not referenced outside of segments.

### Segments
**Segments** represent tracks. They have an ID, a start and end node, and some more metadata. They represent the actual track as a Bezier spline on which the trains will run. The mesh generated from this spline has a only visual thing; it has no influence on the train physics or similar.

```jsonc
{
  "tracks": {
    "segments": {
      // The ID of the segment, as referenced in the spans
      // The game usually prefixes those with 'S'. It's recommended to add an unique identiifer, starting with your name/mod name as abbreviation.
      "SSCExample_mLo15": {
        // The IDs of the nodes that define this segments. Segments do not truly have a start or end (and the game calls them A/B instead),
        // but for linearity's sake we'll pretend they start at one point and go the other way.
        "startId": "NSCExample_Sqf1",
        "endId": "NSCExample_51P2r",
        // The priority defines which segment should be the "normal" one if this segment is part of a switch. This field is optional and defaults to 0.
        "priority": 0,
        // The speed limit (in mph) can artificially lower a segment's speed. Note that the curvature is still evaluated by the AI/game, and may still be lower.
        "speedLimit": 40,
        // MapFeatures (part of progressions) can turn on/turn off track groups by name. If not set/null, the track will be available from the start; otherwise,
        // it will be hidden until a map feature enables it.
        // Defaults to null
        "groupId": "s2",
        // Defines the color on the map; in 3 shades of gray. Values are "mainline", "branch", and "industrial". Defaults to "mainline".
        "trackClass": "mainline",
        // The style influences the mesh/map generator:
        // - "standard" is the default, and will cause the generator to emit rails, ties, gravel-bits and adjust the terrain accordingly
        // - "bridge" will just emit rails and ties, without any terrain adjustments. This allows for "floating" track.
        // - "tunnel" will dig a tunnel, with a tunnel portal on each end of the spline (i.e. cannot be used for >1 lane/>1 consecutive spline without graphic glitches).
        // - "yard" is similar to standard, except a bit muddier.
        "style": "tunnel"
      }
    }
  }
}
```

### TrackSpans
**Spans** represent a piece of track that can span across multiple segments. If you can highlight something in-game on the tracks (e.g. industry tracks), those are spans. They are defined by an ID and two locations, an upper an lower. Segments must be normalized manually, i.e. the two locations must face in the correct way. You can define `"normalize": true` to auto-normalize TrackSpans. This may not work with spans on/across added segments. Note that a span has a purpose which defines which way its locations should face, which may differ between applications (i.e. one application may expects the locations to look outwards from the center of the span, while others expect them to point towards the center).

**Locations** are both a position on the track and a facing direction. They are defined by a segment, an end that they are relative to, and the distance to that end. While it's possible that two locations would represent the same position, the direction would be different. As example, for a segment of length 100, the locations [start + 25] and [end + 75] would be at the same position, but face in opposite directions.

```jsonc
{
  "tracks": {
    "spans": {
      // The ID of the span, as referenced elsewhere
      // The game usually prefixes those with 'P'. It's recommended to add an unique identiifer, starting with your name/mod name as abbreviation.
      "PSCExample_5sZz2": {
        // upper and lower define the two ends of the span and are identical, therefore only one will be documented
        "upper": {
          // ID of the segment that the upper is on
          "segmentId": "SSCExample_mLo15",
          // "end" and "distance" go together: If we are on "segmentId", and we're starting at "end", and we're going "distance" meters along the segment, this is where we will end up.
          // as mentioned before: The direction matters, because the location "faces" a certain way. The distance may also not be longer than the segment, because it cannot cross to another.
          "end": "start",
          "distance": 15
        },
        "lower": {
          "segmentId": "SCExample_k91P25":,
          "end": "end",
          "distance": 0
        }
      }
    }
  }
}
```

## Industries

- Industries are compound objects that consist of one or more components.
- Each industry belongs to an area, which may also define when it is unlocked (at the time the area is unlocked, unless a map feature says otherwise).
- Not all components are equally supported by Strange Customs at the moment.
- If an industry is contract-based, values defined by components represent the highest-tier values. Lower tiers are scaled down.
- The property key is the id of the industry and must be unqiue across all areas and mods. 
- To add an industry, simply define a new key. For best effect, copy an existing industry and tweak it.

### General for Components (IndustryComponent)
- The key defines the id of the component. This must be unique within the industry, but is fine to be used within the game. The complete identifier of the component will be concatenated with the industry's identifier.
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
