# Confusing Supplements

Confusing Supplements is a companion mod for Strange Customs 1.9+ and adds some (example) components. While Strange Customs offers some components out of the box related to car and map patches, Confusing Supplements adds more useful components that go beyond the basic functionality.

From an update/stability point of view, Strange Customs is trying to be as backwards compatible as possible, without losing functionality unless necessary. Confusing Supplements is less "safe" in this regard; as it's a hotchpotch of assorted things, it's possible that updates may take longer, or certain components disappear because they have been made obsolete.

# Car Components

## Refiller

Turns a car into a refiller, which will try to find an directly adjacent locomotive or tender, then transfer matching fuel.

**Requirements:**
- The target must be a SteamLocomotive, DieselLocomotive, or Tender
- The refiller must define LoadSlots, and those MUST have a required load identifier
- Only fuel will be transferred if the identifiers match
- If both ends contain a target, a random one is selected and kept until it is decoupled

Component JSON:
```json
{
  "kind": "ConfusingSupplements.Refiller",
  "name": "Refiller",

  // Optional: Rate (in units/h) that this car should load off to neighboring car(s)
  "transferRate": 36000
}
```

## DestinationSign

Turns an asset into a swappable sign, which can be adjusted with LMB/RMB. This can be used if you want
to quickly adjust text on a locomotive between several, pre-defined values.

**Requirements:**

An AssetPack asset which has the sign prefab.
- The prefab MUST contain a child called "Sign", which is the actual sign model. This is hidden/shown depending on the text.
- Any other children are not affected and will stay where they are.
- You can specify as many texts as you would like.
- The size must match the text area to be occupied; x and z are the dimensions the text can take (in meters); y is how far it is projected. Keep y to something like 0.15.
- The text is projected at the position of the component. Move the actual model/signs respectively.
- Use the shared car shader for the mesh where the text should be projected; otherwise it will not show up.
- There must be a collider somewhere which has the Pickable layer set. This will be used to determine the clicking area.

```json
{
  "kind": "ConfusingSupplements.DestinationSign",
  "name": "DestinationSign",

  // Identifier of the sign asset to use
  "model": {
    "assetPackIdentifier": "zsc://MyMod/SCAssetPacks/MySign",
    "assetIdentifier": "my-sign"
  },

  // The size of the projected text, in meters
  "size": [0.75, 0.15, 0.25],

  // The list of destinations to flip through when clicking on it
  "destinations": [
    "Guld",
    "Cumbercity",
    "Cannot",
    "Blackier",
    "Grnu"
  ]
}

## Bodygroups

Allows enabling/disabling children of the car/thing in question. Adjusted using the inspector.

**Requirements:**

- In each bodygroup, all GOs except the currently selected one will be disabled. Therefore, it's necessary to structure the prefab accordingly.
- An GO _can_ be specified by multiple bodygroups, but such behaviour is weird and will result in pain and misery. Don't do it.
- Only one BG component per car is allowed.
- The GOs are set active/inactive when they are chosen/unchosen. You can use this with scripts that use OnEnable/OnDisable or similar (e.g. animators, Audio Sources).

```json
{
  "kind": "ConfusingSupplements.Bodygroups",
  "name": "Bodygroups",

  // The key is an ID, must be unique, and is transmitted/saved
  "groups": {
    "lights": {
      // Displayed name
      "name": "Lights",

      // Again, IDs
      "options": {
        "none": {
          // Displayed option
          "name": "None",

          // Empty paths are allowed and mean "nothing is activated"
          "path": null,
        },
        "fancy": {
          "name": "Fancy Lights",
          "path": [ "path", "from", "root", "object" ]
        }
      }
    }
  }
}
```

## LabelPrinters

Labelprinters allow additional text decals to be placed onto a car.
- Components are grouped (by `Group` if not empty; `Name` otherwise).
- Per group, one text input is provided. Changing this text will update all components that have this group/name.
- The name is displayed to the user. Choose wisely.
- Editing it is a bit of a PITA, as it does not show up in the editor. This is based on laziness/some technical constraints.
- Its definition is more or less the same as a Decal-component, with the exception of `group`.

```json
{
  "kind": "ConfusingSupplements.LabelPrinter",
  // The first component's name will be displayed in the customization window.
  "name": "Side",

  // Optional: If set, the group is detached from the name, and the first component's name is displayed to the user
  "group": null,

  // Same as decal components in-game.
  "size": [ 2, 1, 0.05 ],
  "content": "RoadNumber",
  "forceColor": ""
}
```

# IndustryComponents

## Empty

A component that does absolutely nothing, but can be used to "highlight" tracks by supplying spans.

```json
{
  "type": "ConfusingSupplements.IndustryComponents.Empty",
  "name": "Foobar Area",
  "trackSpans": [ "Pfoo", "Pbar" ]
}
```

## CaptiveConversionLoader
Similar to the IndustryLoader, but strictly used for captive freight, and converts `loadId` from the industry storage to `convertedLoadId` when filling cars.

- The units between loadId and convertedLoadId must be identical.
- This component is not interacting with FormulaicIndustryComponents and will be ignored by them.

```json
{
  "type": "ConfusingSupplements.IndustryComponents.CaptiveConversionLoader",
  "name": "Player Coal Load",
  "title": "BUY CHEAP COAL", // Optional: Name to display in the order drop-down; if null the industry name is used
  "trackSpans": [ "Pfoo", "Pbar" ],
  "loadId": "coal", // take "coal" from the industry's storage
  "convertedLoadId": "player-coal", // fill the same amount of "player-coal" into the cars instead,
  "carTypeFilter": "*",
  "carTransferRate": 1000
}
```

## CaptiveConversionUnloader
Similar to IndustryUnloader, but strictly used for captive freight, and converts `convertedLoadId` from the car into `loadId` into the storage when emptying cars.

- The units between loadId and convertedLoadId must be identical.
- This component is not interacting with FormulaicIndustryComponents and will be ignored by them.
- The max storage is defined in the loadId, i.e. the industry storage (not what's in the car).
- If the convertedLoad defines a payout, it will be paid at the end of the day as long as >= 1 units have been unloaded.

```json
{
  "type": "ConfusingSupplements.IndustryComponents.CaptiveConversionUnloader",
  "name": "Player Coal Unload",
  "title": "SELL CHEAP COAL", // Optional: Name to display in the order drop-down; if null the industry name is used
  "trackSpans": [ "Pfoo", "Pbar" ],
  "convertedLoadId": "player-coal", // accept this load on cars...
  "loadId": "coal", // and convert it to this load when put into the industry
  "carTypeFilter": "*",
  "carTransferRate": 1000,
  "maxStorage": 10000
}
```

## Pay4Resource
Similar to a captive IndustryLoader, but will charge the player for purchasing resources. They are taken from the industry's storage, which must therefore exist.

```json
{
  "type": "ConfusingSupplements.IndustryComponents.Pay4Resource",
  "name": "Purchasing Things",
  "title": "SUPER CHEAP COAL", // Optional: Name to display in the order drop-down; if null the industry name is used
  "trackSpans": [ "Pfoo", "Pbar" ],
  "carTypeFilter": "HM,HT",
  "loadId": "coal",
  "carTransferRate": 750000,
  "notAfterHour": 24, // Optional: Do not load cars if the time is after this hour. Default 24
  "notBeforeHour": 0, // Optional: Do not load cars if the time is before this hour. Default 0.
  "costPerUnit": 0.002, // Price per unit loaded; independent of the actual load's price.
  "fillPercentage": 0.9, // Optional: Consider cars "full" and re-waybill them if they are at least this much loaded ([0..1]). Default 1.
  "bookReasons": [ "Foo", "Bar" ] // Optional: Array containing string reasons that will be listed in the accounting book for this purchase. A random one will be picked.
}
```