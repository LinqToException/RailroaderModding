# Troubleshooting

This file is supposed to act as some kind of "entry guide" for troubleshooting issues.

# Basic Guide to Reporting Issues (BGRI)

If you want to report an issue with a mod (_any_ mod), there are a few things that make it more likely that people are willing to help you:

1. **RTFM** (Read The Friendly Manual). Most mods come with documentation, either on their download page, or as some kind of text file inside the download. Try to fight the urge to click on the big, shiny download button first and
   read (and follow!) the instructions that are provided to you. You'll save a lot of time for yourself and others.
1. **Be polite.** This should be obvious, but most mod authors work for free in their spare time to provide something for you to use, for free. You're not their customer, and they are under no obligation to assist you in any way.
  It's a free service they can provide if they want to.
2. **Be efficient.** Error reports should be concise but conclusive. Avoid things like just saying "can someone help me" (_don't ask to ask_), "it doesn't work" or "it breaks". Try to follow this pattern:
   - **What were you trying to do?** For example, "I've tried starting a new game, "
   - **What went wrong exactly?** "but the industry mod I've installed does not show up."
   - **What have you tried so far?** "I've re-run RailLoader and re-installed the mod, which was successful, but didn't work either."
   - **What have you checked so far?** "In the mod menu, the mod displays as loaded."
   - **Have a log file ready.** It might be necessary to have a log file, which is overwritten in semi-regular intervals. If you encounter an issue, putting it aside could help helping you in the future.
3. **Have a plan to provide everything we need.** Often, more information might be necessary, such as a savefile, a logfile, or just a general action ("Have you tried restarting Steam completely?" as a common example).
   It's possible that these may _sound_ like they don't make much sense, but there might be a reason for those things to be asked. Providing the necessary information is key to having your issues resolved.

   Keep in mind that files can contain identifying information about you (such as installation directory, user name, or Steam ID). Don't share (log) files publicly unless you're comfortable with that, or make sure
   to strip this personal information before uploading it.

Always remember: There's a person on the other end that likely had to deal with all kinds of support questions before, and a good chunk of those were probably simple mistakes made by the user. You're very likely
not the first person to need help, you're probably not the first person to have this problem, and following these steps will make it more likely for someone to (be able to) help you.

Thank you for helping us help you help us all.

# Log Files And You

The game keeps its log files in a hidden directory, namely `%USERPROFILE%/AppData/LocalLow/Giraffe Lab LLC/Railroader` in a file called `player.log`. This is Unity's main log file; in case something goes horribly haywire and it crashes,
this file will (usually) contain the last breath. The player-previous.log is (as the name implies) the previous log, i.e. the second-to-last play session.

RailLoader keeps separate files in the game's directory, as railloader.log. Similar to Unity's previous log, Railloader keeps three more logs, labelled railloader-X.log, where X is the how many last-th session it is
(i.e. railloader-3.log was the 4th last session, because 3+1).

Both files have in common that they can be somewhat technical in nature, and hard to decode if you're not familiar with the logging used. So here's a short introduction:

Railroader uses [Serilog](https://serilog.net/), a .NET library for structured logging, sprinkled with "normal" Unity messages. Railloader expands this by re-directing all messages into a railloader.log, which allows the format to be unified across
Unity and Serilog messages (i.e. both have the same timestamp, and context messages).

## Log Format

The log has a simple format:

```
[TIMESTAMP LOG_LEVEL SOURCE] MESSAGE
EXCEPTION
```

- **Timestamp** is the time at which the message was logged. This usually has a sub-second resolution that allows roughly seeing which frame something occurred in.
- **LOG_LEVEL** is the log level of the message. This is used for filtering out "usually unnecessary" messages, and to flag the... importantness of messages to the reader. The following log levels exist, in ascending severity,
  with a very personal opinion of what they mean:
   - **VRB**: Verbose; usually very spammy and containing in-depth information that is rarely used. By default turned off, but can be turned on with `--verbose` as start argument when using Railloader.
   - **DBG:** Debug, usually somewhat spammy and containing additional debug information. By default turned off, but can be turned on with `--verbose` as start argument when using Railloader.
   - **INF**: Information, the basic log level for things. This is the first log level that is enabled by default, and contains messages you want the user to usually see.
   - **WRN**: Warning, the first "this is probably not very good" level. Warnings usually mean that something isn't quite as expected, but not horribly gone wrong. This _can_ mean that something is up, but doesn't have to.
   - **ERR**: The first really-not-so-good level. Errors should usually not be happening, and their presence might indicate that something is going wrong.
   - **FTL:** Fatal, a level you shouldn't really see in a game log, but it's technically there!
- **SOURCE**: Serilog allows you to specify a "source context" (technically, a magic property that an be attached by using `.ForContext<T>()` or `.ForContext(typeof(T))` on a `ILogger`). This can be useful to indicate _what_ logged
  this message. In Railroader, it's often missing, resulting in an empty string. For Unity log calls (`UnityEngine.Debug.*`), Railloader will try to guess the source based on the assemply and fill in "Unity" if it's probably something Unity-ish.
- **MESSAGE**: The message to be logged. Internally, this consists of a message template (`"Hello {Name}!"`) and a set of properties (`{ Name = "World", SourceContext = "Greeter" }`), but it's rendered to a string in the log files. This message can,
  but usually does not, span multiple lines.
- **EXCEPTION**: Starting on a new line, log messages can have an attached exception (by passing it as first parameter to the logging call). This is optional, and usually missing.

## Key points to read and understand log files

Reading the log files is an art, and usually only the people that have written the log lines really understand what they mean and in what context they are written exactly. However, there are some rough points you can look at:

- **Make sure the log file is correct.** One annoying issue can be stale log files, where people swear up and down that they have given you the correct log file, but it's not the right one. Railloader has started to log the current local time,
  as ISO-8601 string, at the very beginning of the log. Check the date and time to see whether this log file is really "fresh" (or relevant to whenever the issue happened). Usually, you also want a `railloader.log`, and not an e.g. `railloader-3.log`.
- **Ignore unrelated errors.** The base game may have some exceptions already out-of-the-box (currently, for example, fairly early in the log, it will complain about a `System.NullReferenceException` with `TextMeshProUGUI.Rebuild` a few times).
  It might be hard to figure out what error is related to an issue, and when in doubt, you should take it into consideration (unless somebody more knowledgeable tells you that it's unrelated).
- **The more severe, the more interesting.** For a first rough overview over what is going on, you can ignore `VRR`, `DBG`, and often `INF`, and CTRL-F straight for `ERR`. If that yields nothing interesting, `WRN`. Only if that also doesn't work should
  you try to go "deeper" into the log.
- **Try to limit the search area.** The log file, especially with `--verbose`, contains many things you can search for - like car numbers for example. If you have a car that doesn't want to load at an industry for whatever reason, try searching for its
  number (i.e. designation without the railroad prefix). You might find an entry where its ID is referenced, which you can then search for again, and so forth.

  At other times, you might know that it happened at a specific time, or after a specific event - try to see if there's anything in the log around that time, or announcing that event.
- **It's a matter of practice.** The more you look at (and work with) the log files, the more familiar you will be with what should be there, or shouldn't.

Generally speaking, log files contain a lot of information, and usually can be used to identify, or even solve, most issues.

# Strange Customs Issues

Strange Customs is trying to validate some, but not all, input given to it. It is, however, very noisy, especially with `--verbose`, and most issues can be solved with that.

## After updating to SC 1.9+, I get a weird window and my tracks aren't loading properly.

1. Read the window.
2. Read the window again if you are still following this checklist and have not figured out the issue by now.
3. One of the mods you're using is invalid and causes problems. Figuring out _which_ mod causes problems is not trivial; you can use the graph-dump.zip and give it to someone else to help them figure it out. If you want to go on your own, the message likely includes which segment/span/industry is affected; in the `changes-final.json` and `graph-final.json` you can then take a look at the final state of the game, and who probably caused the errors. It's not always obvious, however, and requires some knowledge about map modding.

## Why does my mod not work with 1.9+ when it worked fine before?!

Short answer: It didn't.

Long answer: It never really did. SC 1.9 added more strict validation, which should have happened earlier. So far, it either silently ignored errors (only logging them in the log, which was apparently ignored by everyone), or allowed the creation of invalid segments/spans/industries. This then led to other issues later on, such as interchanges spawning endless cars, industries not loading, or tracks not being visible.

SC 1.9 and later will have more strict validation, meaning mods that were previously built in an "unclean" way will no longer work. You cannot turn this behaviour off and you shouldn't want to, either. It's for your own good - you don't want to play with a half-patched game.

## "My Interchange spawns duplicate cars" / "My interchange does not spawn cars" / "My interchange spawns way too many of the same cars"

This is potentially an issue with a modded industry, which is _not_ the one with the many cars. Check your log file for a message like `No car for filter `.

If you find this message, identify the offending industry by

1. Open the Railloader Settings
2. Select Strange Customs
3. Check "Show Dev Settings"
4. Check "Dump Final Graph"
5. Restart the game or load another save
6. Open the `graph-modded.json` in your game directory
7. Search for the filter that didn't match, enclosed in quotes, that you read in the log message (e.g. `No car for filter: HAT` => `"HAT"`).
8. You should find an industry component/industry with the offending filter.

You have various options to fix this issue:

1. Change the filter to point to a valid car (either by editing the JSON of the mod, or writing a patch mod)
2. Remove the offending mod
3. Download/Fix the missing cars that the industry requests

## My `Model.OpsNew.TeleportLoadingIndustry` does not load cars.

The most common mistake is to not set up the input and output spans properly; although it is called a "teleport", it's really following the tracks, and either setting the spans wrong or not providing
a proper path will lead to odd/non-functional behaviour.

0. Check the log file if there's any obvious issue related to your industry, industry component, input span IDs, output span IDs, or cars that are being loaded.
1. Make sure that the `inputSpans` and the `trackSpans` are both set (and ideally, identical).
2. Make sure that there exists a valid track connection (it cannot reverse, but it can have switches) from every `inputSpans` to every `outputSpans`.
3. Make sure that both spans are valid. SC _should_ complain about invalid spans; but check the log file for the span IDs to see if anything pops up.
4. Spans are directional. If it's not working, try reversing your `inputSpan` (i.e. switching `upper` and `lower` around).
5. Check with For Your Convenience whether the storage of the industry component is actually filling up.

## My JSON is invalid

Throw your JSON into a validator like for example [JSONLint](https://jsonlint.com/) to try to figure out what the mistake is. Common ones include:

- Not separating properties or items (rule of thumb: Unless an `{`, `[`, `}` or `]` follows, you probably want to put an `,` at the end of the line)
- Not ending quotes (`"` get lonely easily; it is required to alway sprovide them in pairs. Unless you escape them with `\`, but that's something that's usually not required.)
- Mixing up arrays (`[]`) and objects (`{}`). Arrays are "Here's a bunch of various things"; whereas objects are "Here's a thing with a bunch of properties."
- Using the wrong special characters (the ones you need are `"{}[]:.,`)

The Newtonsoft.Json.JsonException _tries_ to be helpful, but sometimes errors can "cascade" down: The place where the error occurred, and the place where the parser figured out that something wasn't right, are not necessarily the same.
You can compare this to a pigeon having an eventful, but unfortunate, meeting with an airplane jet engine. When the airplane crashes much farther away, the thing you see is an airplane-shaped crater, not a bird-shaped hole in the engine. 
It can be similar in JSON: You may forget a character on one line, but it takes a few lines until the parser actually figures out something is wrong.

Some recommendations:

- **Proper Indentation:** JSON ignores all whitespace outside of strings. You can put as many as you want wherever you want, and you should use that. Whenever you go "one level in" (i.e. start an object `{`, or an array `[`), indent
  the content of the object/array by a fixed amount (e.g. 3 whitespaces, or 1 tab). Keep this consistent: It makes it much easier to see where an object ends, and you can easier figure out if you're closing everything you're opening properly.
- **Automatically close things:** Whenever you open something (an object, array, or string), automatically write the corresponding closing character right away, then work away in the middle.
- **Use a proper editor to your advantage:** Some editors offer you JSON validation (and sometimes formatting) built-in. Make use of that to track brackets and format the document.
- **It's usually a comma.** Perhaps I'm personally biased, but that's what it is for me.

## I've edited some track, but the switches/junctions are not spawning

The game's mesh generator cannot create switches/junctions for segments that do not intersect properly, and this can cause the game to refuse to consider it a track - i.e. the switch is not just invisible, the segments do not exist at all. Generally speaking, the following rules apply:

- The segments must diverge. If you have two (almost) straight pieces that never really separate, then the switch cannot be generated.
- The segments need to be planar-ish. If you don't want to fiddle with trigonometry to figure out whether it's planar or not, for all nodes involved in the segment set the Y position to the same value, and make sure that the rotation's X and Z are both 0. If you do want to do trigonometry, make sure that ||(root.pos-branch1.pos)|| and ||(root.pos-branch2.pos)|| both have the same inclination (i.e. the y-incline is the same for both), and that the rotation on _all_ nodes is correct. If you're using Unity, `Quaternion.LookAtRotation` is your best friend for these scenarios - plop in the directional vector and voilà.
- When live-editing track, it's possible that the switch was just not re-generated properly after you've adjusted it. Exit to the main menu and back again to make sure you have a decent, recent state.
- Try to keep the segments somewhat short-ish. Running switches with exceedingly long or exotic splines can be problematic.
