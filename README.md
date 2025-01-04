# MGE-VScript
A fully vanilla compatible rewrite of the MGEMod plugin.  No sourcemod plugins required.

## Features & Progress

| Feature | Status |
|---------|--------|
| [Legacy map support](https://github.com/sapphonie/MGEMod/blob/master/addons/sourcemod/configs/mgemod_spawns.cfg) | ✅ |
| Quake-style announcer lines (toggleable) | ✅ |
| Endif | ✅ |
| Ammomod | ✅ |
| Infammo | ✅ |
| Turris | ✅ |
| BBall | ✅ |
| Plain text ELO/stat tracking | ✅ |
| Database stat tracking support | ❌ |
| 4Player | ❌ |
| Koth | ❌ |
| Arena leader system for custom rulesets | ❌ |
| Arbitrary team sizes | ❌ |
| Player-configurable spawn ordering | ❌ |
| In-Game map configuration tool | ❌ |

## Installation
- Drop the mapspawn.nut file and mge folder in your tf/scripts/vscripts directory.
- If you know github/git, I recommend cloning the repository to this directory so you're always up to date.

## Converting your map configs
- Open a copy of `mgemod_spawns.cfg` in VSCode/NP++/any text editor that supports regex search/replace, enable regex
- If you're confused, Google/ask your favorite AI chat bot how to enable regex search/replace in your text editor

    - Find pattern: `(\s*)"([^"]+)"\s*\n\s*\{`
    - Replace pattern: `$1"$2": {`
    - Replace All
- Then:
    - Find pattern: `(\s*)"([^"]+)"\s+"([^"]+)"`
    - Replace pattern: `$1"$2": "$3"`
    - Replace All

- **CUSTOM MAPS NEED TO BE INDEXED MANUALLY!** See the `mgemod_spawns.cfg` file for an example of how to index your map
    - Failing to index your maps will result in !add being unordered, rendering everyone's !add binds useless

## ELO/Stat Tracking
### SECURITY WARNING
Support [This github issue](https://github.com/ValveSoftware/Source-1-Games/issues/6356) if you want this to be fixed.
- While most existing MGE maps are safe to use, **DO NOT ENABLE ANY STAT TRACKING ON UNTRUSTED MAPS!**
    - Any MGE maps created after the VScript update can pack a mapspawn.nut file that will override ours
    - Not only will this break the gamemode, but malicious maps can target either the database or filesystem and manipulate player stats
    - This cannot be fixed unless Valve implements another reserved file (i.e. init.nut) that runs before mapspawn.nut and can only run from the server filesystem
    - **How to check if a map is safe:**
        - Open the bsp using GCFScape, open the .zip file, and check the tf/scripts/vscripts directory in this zip file
        - If you see a mapspawn.nut file, the gamemode will either not load correctly or this map is unsafe
        - Ctrl+F and search for `StringToFile` or `FileToString` in every script file, if you see any of these, the map is probably unsafe
        - Search for `__MGE__VPI`.  If this shows up anywhere, the map is attempting to tamper with the database 

### Plain Text
- perfect option for MGE servers running on a single physical server
- player stats are tracked in the `tf/scriptdata/mge_playerdata` directory indexed by steamid.

### Database
- Database tracking uses [VScript-Python Interface](https://github.com/potato-tf/VPI) to send data from vscript to python through the filesystem.
    - Install Python 3.10 or newer if you don't already have it
    - Install the `aiomysql` module
    - Add your database credentials to `tf/scripts/mge_python/vpi.py` and run this script constantly in the background, this is your database connection
        - You should create a systemd service for this on linux, or whatever the windows equivalent is
    - Check server console for any VPI related errors when you join/leave the server.
    - This will automatically create the `mge_playerdata` table in your database

## NavMesh generation

Included is a tool to generate a navmesh for every arena on a given map.  Load any map you want to generate a navmesh for in singleplayer, enable cheats, and paste this into console

`ent_fire bignet callscriptfunction "MGE_CreateNav"`

Or for only one arena:

```ent_fire bignet RunScriptCode "MGE_CreateNav(`Badlands Middle`)"```

### WARNING:
- This is very slow and will freeze your game for every arena
- More "abstract" arenas (such as the ones on oihguy or chillypunch) will generate nav squares where you may not want them, and will take forever to generate.  Both oihguy and triumph take 30+ mins for every arena.  You have been warned.