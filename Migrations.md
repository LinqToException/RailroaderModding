# Strange Custom's Migrations

Migrations operate on a separate system and allow you to write migrations from "old" content to "new" content.
They are called before the save is loaded, and similar to JSON patches work on a mixinto system.

These migrations allow the same JSON patching rules as the normal JSON patches. Therefore, this document will skip over
the basics and is more "straight to the point".


## Definition.json

```json
{
	"manifestVersion": 6,
	"id": "Berk2Geep",
	"name": "Berk -> Geep",
	"version": "1.0",
	"requires": [ "Zamu.StrangeCustoms" ],

	"mixintos": {
		"game-migrations": "file(migrations.json)"
	}
}
```

We're following normal mixinto rules here.

## Migrations JSON

An example JSON would look like this:

```json
{
	"waybillDestinations": {
		"whittier_sawmill.s12": "sylva.interchange"
	},
	"properties": {
		"my_industry.my_component": "Zamu.MyIndustry_MyComponent"
	},
	"carTypes": {
		"ls-284-b65": "ld-gp9"
	}
}
```

- **waybillDestinations** is a mapping between full identifiers (`[industry identifier].[component identifier]`) of industry components. You can use this to "redirect" trains in case you've re-named an industry. Or want to hijack some cars.
- **properties** is a bit more special and deals with the properties system in-game. If you rename an industry, or component, and want the state (storage, contract, etc.) to persist, you'll have to rename those as well.
- **carTypes** is a mapping between car types and can be used to "transform" cars from one type to another. This is useful if the ID, for whatever reason, changes, or you're abusing some spawn mechanic (like e.g. cloning a locomotive, then setting a different `tenderIdentifier` to have a "tender swap" mod).