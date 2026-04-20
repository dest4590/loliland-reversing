# [LoliLand](https://loliland.ru/) Reversing

### [README-RU](README.md)

### Analysis of the [LoliLand](https://loliland.ru/) project for Minecraft

## Project Structure

The LoliLand project uses a custom launcher to run modified Minecraft builds and includes protection against installing third-party modifications.

Each build contains `LoliMod`, which provides its functionality. `LoliMod` is common to all builds.

## Launcher Analysis

The launcher is a JAR file (`launcher.jar`) located at `%appdata%\.loliland\launcher.jar`.

It is obfuscated, but code strings remain readable.

When launched, the launcher sends PC configuration data to the API server [(server list)](servers-en.md).

```json
{"operatingSystem":{"name":"Windows 11","arch":"amd64","isX64":true},"processor":{"id":"*********","name":"AMD Ryzen 5 5600X 6-Core Processor ","physicalCount":6,"logicalCount":12,"freq":3701000000},"baseboard":{"manufacturer":"********","name":"Micro-Star International Co., Ltd.","serialNumber":"**************","hardwareUUID":"**************"},"graphicsCards":[{"name":"Meta Virtual Monitor","vendor":"Meta Inc.","virtualMemory":0},{"name":"NVIDIA GeForce GTX 1080 Ti","vendor":"NVIDIA","virtualMemory":4293918720}],"displays":["24G2WG3-: 52 x 29 cm (20,5 x 11,4 in)","HP ZR22w: 48 x 27 cm (18,9 x 10,6 in)"],"totalMemory":34282242048}
```

After authorization, the launcher performs the following requests:

-   Authorization: `https://api.loliland.ru/launcher.auth` Result: `{"data":{"votes":0},"type":"success"}`
-   Launch parameters: `https://launcher.loliland.pro/sync` Result: [JSON format](api/sync-output.json)
-   Server IPs: `https://api.loliland.ru/launcher.address` Result: [JSON format](api/address-output.json)

Client launch process by the launcher:

1.  **File verification:** `updateVerify: [libraries, natives, mods, coremods, client.jar]` - the launcher determines a list of files and their hashes to check for updates.
2.  **Configuration version check:** `updateVersion: {config/_=3.1, optionsof.txt=2.0, lang/_=1.1, options.txt=2.0}` - configuration file versions are checked.
3.  **Local hashes:** `updateVersion local hashes: {}` - output of local file hashes.
4.  **File removal:** `toRemove: []` - definition of files for removal (used for protection).

Source code for file verification stages is located in the file [launcher/file-security.java](launcher/file-security.java).

## Client Analysis (LoliMod)

`LoliMod` is responsible for the main client functions, including:

-   Store
-   Custom chat
-   GUI
-   Server network packet handling

When launched, `LoliMod` loads server packet processing modules:

```
[18:49:43] [Client thread/INFO] [LoliLand]: On Server PreInit load module: l0lIIOLAND
[18:49:43] [Client thread/INFO] [LoliLand]: On Server PreInit loaded module: l0lIIOLAND
```

## Client Analysis (client.jar)

The client (`client.jar`), when launched, loads mods and checks their hashes (if this file is missing, the mod is not considered and not loaded).

Each mod contains a file `META-INF\mc_class_data_hash.bin` with a list of class hashes. Example content:

```
00000000 E1 88 43 78:68 2F 09 A7|84 1C 8F 9D:50 2F 28 D3
00000010 A1 E4 E1 BB:18 13 B5 76|45 15 4E 11:E3 99 84 B4
00000020 8B BF 00 7E:D6 AD 87 F9|90 EC AE E9:D1 D7 4A CE
00000030 C2 40 3E 05:FC 8A 05 92|51 19 00 56:BA 7F 7C EE
```

## Bypassing Launcher and Client Protection

**Bypassing launcher hash checks:**

1.  Open `launcher.jar`.
2.  Find the string `toRemove: ` in the code.
3.  Delete the code responsible for this stage of file verification.

**Bypassing client mod verification:**

1.  Open `%userprofile%\loliland\updates\clients\techno_magic\client.jar` (path depends on the build).
2.  Find the string `Examining file %s for potential mods`.
3.  Replace the condition `if (ClassDataDumpParser.HAS_DUMP) {` with `if (false) {` to force load mods without hashes.

**Launching the client without the launcher (recommended):**

Use the [start.py](scripts/start.py) script.

**Conclusion:**

The launcher and client use simple protection methods that can be bypassed. However, using cheats on the LoliLand project is unlikely to bring the desired result.

## Bypasses

### Vanish bypass (display all players in the TAB list, including those in vanish):

1.  Find the string `loliland/gui/modules/tablist/vanish.png`.
2.  Delete the condition in which this string is located.
3.  Repeat for all icons (`fly.png`, `god.png`, `vanish.png`).
