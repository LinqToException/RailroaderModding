# Custom Liveries

Beginning with 1.9, Strange Customs has the ability to offer livery customizations for rolling stock that has opted in to do so.

**This is an experimental feature, subject to change.**

## Requirements

In order for rolling stock to be eligible for texture swapping, in its Definitions.json entry, the `metadata.tags` array must contain the tag `"sc-livery-swappable"`. This serves as an opt-in mechanism.

## Dumping Textures

To dump the textures of an existing, valid car, follow the following steps:

0. Make sure that the vehicle you want to dump the textures from has the `sc-livery-swappable` tag. If it does not, the following steps will fail.
1. Select the car in-game (CTRL-Shift-Click)
2. Open the console
3. Enter `/sc-livery-dump`
4. The game will freeze for a bit while dumping the textures
5. If succcessful, the console will tell you the path it has dumped the textures into, relative to the game directory. This is usually in a folder called `SC Livery Dumps/$TYPE IDENTIFIER$`.

Please note that the alpha channel is currently not dumped correctly, and therefore some features relying on alpha may not work correctly.

## Creating a Livery Mod

A livery mod is a normal Railloader mod that contains one or more directory mixintos.

0. Make sure that the vehicle you want to apply the livery to has the `sc-livery-swappable` tag.
1. Create a basic Railloader mod by creating a new folder inside `Mods/` named after your mod, with a Definition.json:
   ```json
   {
     "manifestVersion": 6,
     "id": "MyName.MyMod",
     "name": "My Amazing Liveries",
     "version": "1.0",
     "requires": [ { "id": "Zamu.StrangeCustoms", "notBefore": "1.9.0" } ],
     "mixintos": {
     }
   }
   ```
2. Inside your mod folder, create a folder that will house the files for the livery you wish to create. The name of the folder will be the same as the livery name. As convention, it's recommended to create at least a folder
   named like the engine identifier (e.g. `ld-gp9`), then create the livery folders inside that.
3. Inside the newly created livery folder, insert all textures that you wish to override. You do not (and should not) add textures that have not changed. Only textures that are present in the directory will be overwritten; all others will be ignored.
4. In your mod's Definition.json, add mixinto entries for each livery. The mixinto key is `livery:$TYPE_NAME$`, the value should be `dir(path_to_directory)`.

   Assuming you have made a livery for the locomotive with identiifer le-460, and the livery is inside `Mods/MyModName/le-460/Custom Livery`, the mixinto section would look as follows:
   ```json
   "mixintos": {
     "livery:le-460": "dir(le-460/Custom Livery)"
   ```

   If you have multiple liveries, you must specify them as an array:
   ```json
   "mixintos": {
     "livery:le-460": [ "dir(le-460/Custom Livery)", "dir(le-460/Historical Livery)" ]
   }
   ```

## Applying Liveries

1. Make sure that the mod is loaded correctly.
2. Select the vehicle with CTRL-Click.
3. Under _Equipment_, _Customize_, there should be a section called _Liveries_. This may require you to be a trainmaster or above in case it's a multiplayer game.
4. In the drop-down, you can select the new livery.
5. The livery change should be replicated to other clients that have the livery installed as well, assuming the host has SC 1.9 or later installed.
