# ForgeKit Games Repository

> [!CAUTION]
> This is an automated repository, only very technical users should interact with settings and files.

This is a repository used for storing data and game files for the ForgeKit launcher.

These files will be loaded by the launcher at any given moment.

## Repo Structure

Each game in this repository should have it's own dedicated GitHub branch.

The GitHub branch's name will be used as the game's identifier going forward. (Cannot contain underscores!)

## GitHub Branch Structure

GitHub branches must contain the following folders and files:

- `settings.json` (The main settings for the game, described further down)
- A folder named `art` with the following images (must be `.png`):
    - `icon.png` (Icon of the game, preferrably 1:1 aspect ratio)
    - `logo.png` (Main logo of the game, any aspect ratio)
    - `cover.png` (Cover art for the game, used in thumbnails, preferrably 16:9)
    - `background.png` (Background art for the game, used in the game launch screen, 16:9)
    - `patch.png` (Cover art for the patch note section, only needed for the latest patch, 16:9)
- A folder named `notes` with the patch notes for the game, containing the following files:
    - `titles.json` (Explained further down below)
    - `vX-Y-Z.html` (An HTML file for each patch note, named after the version number)

> Patch note HTML files must be electron friendly!

## Versioning

Each release of the game should have a version number and a patch note (optional)

Version numbers are structured with the following method: `vX.Y.Z`.
- `X`: The major number, representing any massive changes in the game.
- `Y`: The minor number, representing substaintial changes without too many overarching implications.
- `Z`: The patch number, representing small fixes and features.

> Do note that in `.` is replaced with `-` only when absolutely necessary, and `v` is always lowercase!

## Releases

Each game release should have its tag structured in the format of `vX.Y.Z-gameid`.

As releases aren't specific to any GitHub branch in the API, each release should have a unique tag.

The tag should have the version number appended with `-gameid`, where the game ID is the name of the GitHub branch for your game in the repository.

The release title should be `gameid - vX.Y.Z` for convention purposes.

The assets in the release should just be `platform.zip` for each platform.

For example, if the release has Windows and MacOS versions, the asset files should `windows.zip` and `macos.zip`.

When the .zip file is extracted, it should contain a file called `launcher.json` which provides all relevant information to the launcher.

## JSON File Structure

### `settings.json`

This file contains all relevant information about the game. The structure is as follows:

```json
{
    "name" : "Game Name",
    "authors" : ["Dev 1", "Dev 2"],
    "enabled_global" : true,
    "game_branches" : {
        "alpha" : {
            "name" : "Stable",
            "latest_version" : "v1.0.0",
            "enabled" : true,
            "platforms" : ["windows"],
            "requirements" : null,
            "requires_online": false
        },
        "beta" : {
            "name" : "Beta",
            "latest_version" : "v1.1.0", 
            "enabled" : true,
            "platforms" : ["windows", "linux"],
            "requirements" : ["steam"],
            "requires_online": false
        },
        "gamma" : {
            "name" : "Development",
            "latest_version" : "v1.1.1",
            "enabled" : false,
            "platforms" : ["windows", "linux"],
            "requirements" : ["steam"],
            "requires_online": true
        }
    }
}
```

Explanations:

- `name`: The full game name, such as Rainbow Six Siege.
- `authors` : The people working on this game. Can be an array of strings or a singular string.
- `enabled_global` : Whether the game is accessible globally, regardless of the game branch.

`game_branches` is an object, containing game branch objects, each describing a version of the game, where the key of the object is the ID of the branch (i.e. alpha or delta). Game branches (not to be confused with GitHub branches) are used to ship different versions of the game to different playtesters. The IDs of the branches can be anything, as long as they are unique, and do not change, as they are used internally for things such as file management. Each child object of `game_branches` must have it's key as the name for the game branch.

The following is the explanation for each `game_branches` child object:
- `name`: This the is the name for the branch that is displayed to users instead of the ID. If you wish to change the purpose of a branch, change this property and not the ID.
- `latest_version`: This is the version of the game in the releases section that players in this game branch will load.
- `enabled`: Whether this game branch is accessible to playtesters.
- `platforms`: An array of string deciding which platforms are currently available. The current options are _windows_, _linux_, and _macos_. 
- `requirements`: An array containing which third party softwares are required to run the game. Options so far are _steam_ and _xbox_.
- `requires_online`: Whether the game needs an internet connection to run.

##
### `titles.json`

This file contians any information about patch notes.

```json
{
    "v1.0.0" : {
        "title" : "Welcome to the Arena!",
        "type" : "Major Update"
    },
    "v1.0.1" : {
        "title" : "Addressing some issues!",
        "type" : "Bug Fixes"
    }
}
```

Each patch note in the `notes` folder should be mentioned in this JSON file by mentioning the version as a property in the JSON object. The value should be another object with the following properties;

- `title`: The title of the patch note
- `type`: The type of patch note informing players of the amount of changes in the launcher.

##
### `launcher.json`

This file contians any information about how the launcher should interact with the game files.

```json
{
    "core_exec_command" : "path/to/game/file.exe",
    "relative_dependency_commands" : [
        {
            "command" : "relativeDependencyCommand1",
            "critical": true
        },
        {
            "command" : "relativeDependencyCommand2",
            "critical": false
        }
    ],
    "system_dependency_commands" : [
        {
            "command" : "systemDependencyCommand1",
            "critical": true
        },
        {
            "command" : "systemDependencyCommand1",
            "critical": false
        }
    ]
}
```

Explanations:

- `core_exec_command`: The command used to start the game (must be for the specific platform).
- `relative_dependency_commands`: An array containing all commands (as `command` objects) that are to be executed before the game launch, in the context of the extracted zip folder.
- `system_dependency_commands`: An array containing all commands (as `command` objects) that are to be executed before the game launch, in the context of the user root folder.

The following is the explanation for each `dependency_commands` child object (or a `command` object):
- `command`: The command to be executed (must be for the specific platform)
- `critical`: Whether this command is required to finish successfully for the game to be launched.

The order of command execution is `system_dependency_commands`, then `relative_dependency_commands`. Only after the successful finish of dependency commands being executed is the `core_exec_command` executed.

The definition of a successful execution of dependency commands is no dependency command labelled as critical fails to be executed successfully.

> In the current state, `launcher.json` files do pose massive security risks by executing commands directly. For the time being, please be careful with the commands that are in the `launcher.json` file!

## Patch Notes

Patch notes are always delivered in a `vX-Y-Z.html` file as stated before, and should contain no styling. The supported tags in the HTML so far are `h1`, `h2`, `p`, `ul`, and `li`. Items such as `img` could be used at your own discretion. The patch note window is always in a scrollable 1000x600px window. A preview for how these will look can be done by opening the `patchNoteTemplate.html` file and entering the contents of your patch notes in the `<div class="notes">` container. A preview website of some form may be added in the near future

> In the current state, patch note `.html` files pose security risks due to injecting raw HTML into the container, and malicious JS could lead to unintended behaviour. Please be catious with the contents of patch note `.html` files.

## Access Tokens

To be completed/reworked