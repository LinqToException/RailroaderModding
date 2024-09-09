# Strange Customs (SC)

Some frequently asked questions regarding SC, and possible solutions

# Can I edit passenger stops?

Currently, no.

# Can I edit _<anything related to CTC>_?

No, and this functionality isn't planned for SC either. Someone could give it a shot using e.g. Splineys, however. Just make it a fix-named Spliney that is then populated/parsed by your mod and can be changed by others I guess, like a mini-tracks-section.

# How can I get a list of all nodes/segments/spans/industries/loads?

In the Railloader settings for Strange Customs, check the _Dev Settings_ checkbox, then enable 

# How can I change the size of a loading/service area?

Adjust the track span. Do not just move the node, as this is more likely to conflict with other mods and is generally not a very good idea.

# Why is moving nodes not a good idea?

While technically absolutely fine, care must be taken when moving nodes (and therefore changing segments). The game references positions on tracks relative to a node. If you save your game, move a node and therefore change a segment's length, and had
a car parked halfways parked on that segment, after loading the save again, it's possible that the trucks on the edited segment may now be at a different location - therefore stretching or squeezing the car, which it likely won't appreciate completely.

# What can I do if my switches/points/junctions are not showing up?

See [/Troubleshooting.md](/Troubleshooting.md#ive-edited-some-track-but-the-switchesjunctions-are-not-spawning)

# What is the proper order to build tracks (in the JSON)?

1. Nodes
2. Segments
3. TrackSpans

# What do I need to make an industry?

- Probably some new tracks (see above)
- Create an industry in an existing area
- Add components to the industry

For more general ideas, see [Graph.md](Graph.md).

# Can I use multiple files for my game-graph updates?

Yes. Simply justify them as an array in the `Definition.json` of your mod:

```json
{
  "mixintos": {
    "game-graph": [ "file(tracks.json)", "file(industries.json)" ]
  }
}
```

SC will apply patches first in the order that Railloader defines (by using the mod loading priority), then by the order in the array (e.g. in the example, it first applies `tracks.json`, then `industries.json`).

# How does SC's patching internally work?

1. SC serializes the base-game state into JSON (which can be dumped as graph.json)
2. SC asks Railloader for a list of mixintos for `game-graph`. For each mixinto, it will

   1. Load the JSON
   2. Apply the JSON to the current base game state
   3. If an error occurs at any point in this stage, abort applying the current patch (but already made changes will continue to be present)
3. Allow mods via event callback to add their own, dynamic data to the graph
4. Take this new, modded graph (which can be dumped into the graph-modded.json) and deserialize it again
5. Creates, in the following order (and if an exception occurs at any point here, stops the entire patching process where it currently was and displays an error):
   1. Nodes
   2. Segments
   3. Spans
   4. Areas and Industries
   5. Signs
   6. Scenery
   7. Splineys
  
# What are splineys?
Splineys were originally meant as some kind of "blob" data type, where SC does not (need to) know what exactly it is, but other mods can read and parse that data. Originally made for trestles, then rivers/roads, but nowadays abused for other mods
that just want to have some kind of data format that is patchable by others.

# How can I build trestle bridges?
By using splineys in a graph patch file:

```json
{
  "tracks": { ... },
  "splineys": {
    "MyMod.MyTrestle": {
      "handler": "StrangeCustoms.AutoTrestleBuilder",
      "headStyle": "Block",
      "tailStyle": "Bent",
      "points": [
        {
          "position": { "x": 152, "y": 495, "z": 1520 },
          "rotation": { "x": 0, "y": 220, "z": 0 }
        },
        {
          "position": { "x": 180, "y": 495, "z": 1541 },
          "rotation": { "x": 0, "y": 230, "z": 0 }
        }
      ]
    }
  }
}
```

For `headStyle` and `tailStyle`, you can use either `"Block"` or `"Bent"`. This changes the way the bridge is generated slightly.

You must specify at least two points, but can define as many as you want. In order to be in sync with the tracks, it's recommended to use exactly the same positions as the nodes that make up the track.
The game's Bezier curves are dependent on the length of the segment, meaning that if you start a trestle in the middle of a track segment, it will have a different curvature than if it were identical.

# Can I edit roads/rivers?

Editing the existing roads and rivers is possible with Strange Customs 1.8 and later. In earlier versions, you can add new roads/rivers and edit those added by other mods by using splineys:

```json
{
  "splineys": {
    "MyMod.MyRiverOrRoad": {
      "handler": "StrangeCustoms.Tracks.FlowyThingBuilder",
      "profile": "R2_Profile_River_Mountain",
      "style": "River",
      "offsetY": -4,
			"points": [
				{
					"position": { "x": -27080, "y": 563, "z": -20567 },
					"rotation": { "x": 0, "y": 0, "z": 0 },
					"width": 4
				},
				{
					"position": { "x": -27080, "y": 563, "z": -20627 },
					"rotation": { "x": 0, "y": 0, "z": 0 },
					"width": 4
				}
			]
    }
  }
}
```

- For the `profile`, possible choices are "Railroader Paved Road (SplineProfile)", "R2_Profile_River_Mountain (SplineProfile)", "RAM Road profile (SplineProfile)"
- For the style, it's "River" or "Road"
- The `offsetY` defines how deep below/above the terrain the riverr/road should be carved/elevated into. It's a bit fiddly.
- Width defines the width at this point, and can be tweaked per-point.
- You can add as many points as you wish.

Earlier versions of SC had issues with reloading rivers; those should be fixed in SC 1.8 and later.

# Can I ship AssetPacks with Strange Customs?

Strange Customs automatically loads all asset packs inside all enabled, valid Railloader mods that are inside that folder called `SCAssetPacks`, e.g. 
- Railroader
  - Mods
    - MyMod
      - Definition.json
      - SCAssetPacks
        - Steve
          - Bundle
          - Catalog.json
          - Definitions.json

You can fit as many asset packs inside SCAssetPacks as you wish. If you need to reference them in e.g. scenery, currently it's necessary to use `zsc://<mod_id>/SCAssetPacks/<AssetPackName>` as asset pack name instead of "just" `<AssetPackName>`.

The packs are added to the game's normal loading system, after the base game assets, in the order that they are defined by Railloader.