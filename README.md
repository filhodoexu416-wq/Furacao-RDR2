# Furacao-RDR2
Weather-driven supercells and F0-F5 tornadoes for Red Dead Redemption 2 story mode. Script Hook RDR2 plugin (.asi) with full C++ source and build steps.
# Furacao - Supercells and Tornadoes for Red Dead Redemption 2

A Script Hook RDR2 plugin (`Furacao.asi`) for **story mode** that adds weather-driven supercells and F0-F5 tornadoes (and, as a secondary feature, hurricanes on the Lemoyne coast).

Every in-game minute the mod reads the game's own weather (temperature, weather type, time of day, region). When conditions line up, a supercell forms over the plains; its strength sets the ceiling for the tornado it produces. The funnel descends from a wall cloud, touches down, strengthens, weakens and ropes out.

Nexus Mods page: https://www.nexusmods.com/reddeadredemption2/mods/10937

## Install

1. Install [Script Hook RDR2](http://www.dev-c.com/rdr2/scripthookrdr2/) by Alexander Blade (`ScriptHookRDR2.dll` and `dinput8.dll` in the game folder).
2. Copy `Furacao.asi` and `Furacao.ini` to the game folder, next to `RDR2.exe`.
3. Load story mode. About 12 seconds later a message confirms the mod is loaded.

Story mode only. The code returns immediately when a network session is active.

## What the binary does and does not do

`Furacao.asi` is a Windows x64 DLL loaded by the ASI loader. It is built **without the C runtime** (freestanding, custom entry point), which is why it is small and why some antivirus heuristics flag it.

Its complete import table:

| DLL | Functions | Used for |
| --- | --- | --- |
| ScriptHookRDR2.dll | scriptRegister, scriptUnregister, scriptWait, nativeInit, nativePush64, nativeCall, worldGetAllPeds, worldGetAllVehicles, worldGetAllObjects | Calling game natives |
| KERNEL32.dll | GetModuleFileNameA, CreateFileA, WriteFile, SetFilePointer, CloseHandle, GetPrivateProfileStringA, WritePrivateProfileStringA, GetTickCount, GetCurrentProcessId | Reading `Furacao.ini`, writing `Furacao.log`, seeding the random generator |
| USER32.dll | GetAsyncKeyState, GetForegroundWindow, GetWindowThreadProcessId | Hotkeys (F6-F11), only while the game window is in the foreground |

- No network access of any kind.
- The only files it touches are `Furacao.ini` (read, and three settings saved from in-game hotkeys) and `Furacao.log` (append), both next to the .asi.
- It does not send key presses or messages to any window, does not start processes and does not load other libraries.

## Build

The release binary is built on Linux with LLVM, cross-compiling to Windows x64. No Visual Studio and no Windows SDK are needed.

Requirements:

- clang, lld (`lld-link`), `llvm-dlltool`, `llvm-rc`. The release was built with Ubuntu clang 18.1.3 and LLD 18.1.3.
- `ScriptHookRDR2.lib` from the Script Hook RDR2 SDK (http://www.dev-c.com/rdr2/scripthookrdr2/). It is not redistributed here.

Steps:

```sh
# from the repository root
SHV_LIB=/path/to/ScriptHookRDR2.lib ./build.sh
# output: dist/Furacao.asi
```

`build.sh` generates import libraries for kernel32 and user32 with `llvm-dlltool`, compiles `src/furacao.cpp` and `src/plat_win.cpp` with `-ffreestanding -nostdinc`, compiles the version resource `src/furacao.rc`, and links with `lld-link /dll /entry:DllMain /nodefaultlib /Brepro`.

The link is deterministic (`/Brepro`): with the same compiler version and the same `ScriptHookRDR2.lib`, the output is byte-identical.

SHA-256 of the released `Furacao.asi` (version 2.0):

```
165f2bb1286849d79efdf3b637c4ca39044505632b45bb02323c6a27457c7788
```

A development variant (`CAPTURA=1 ./build.sh`) additionally imports `PostMessageA` and `keybd_event` to press the ReShade screenshot key during test tornadoes. It was used to check the funnel's appearance and is **not** the published binary.

## Source layout

- `src/furacao.cpp` - the whole mod: atmosphere model, supercells, tornadoes, hurricanes, wind forces, HUD.
- `src/nat.h` - wrappers for the game natives used (hash, name and argument types are checked by `tools/check_natives.py` against the public native database).
- `src/plat_win.cpp`, `src/plat.h` - Windows part: DLL entry, .ini, log, keyboard.
- `src/base.h` - math, random numbers and string helpers (no C library).
- `test/sim.cpp` - native Linux simulation: a mock game world answers the natives and the mod logic is checked against it.
- `test/pe_test.cpp` - loads the real `Furacao.asi` into memory on Linux, with stand-ins for Script Hook and the Windows API, and checks that it behaves exactly like the simulated code.

Comments and on-screen text are in Brazilian Portuguese.

## Credits

Requires Script Hook RDR2 and uses its SDK, (c) Alexander Blade.
