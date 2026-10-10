# Dungeon Siege 1 / LOA – Invisible UI Cursor Fix

An issue exists where the mouse cursor in Dungeon Siege and Legends of Aranna disappears during the game when hovered over any 2D element, such as inventory, menus & tooltips. The 3D world cursor still works.

This fix stops the game engine from hiding the Windows mouse cursor, and removes the game's own cursor.

## Instructions

1. Back up `DSLOA.exe` and/or `DungeonSiege.exe` before replacing with patched files.
2. Edit the config files below and add `fullscreen = false` to the top line of each:
   - LOA: `Documents\Dungeon Siege LOA\DungeonSiege.ini`
   - Base game: `Documents\Dungeon Siege\DungeonSiege.ini`
3. Launch the game.

You'll see the normal Windows arrow over the whole game. The game's own sword cursor is removed. The arrow hides while you drag an item, but the item icon still follows the mouse.

## Details

Offsets were taken from the original CD release for `DungeonSiege.exe` with Kallah's `DSLOA.exe`.

The two files are patched at the below locations:

| File | File offset | Original bytes | Patched bytes |
|---|---|---|---|
| DSLOA.exe | 0x1526E | 0F 8E B6 00 00 00 | E9 B7 00 00 00 90 |
| DSLOA.exe | 0x2629BA | 55 8B EC | C2 04 00 |
| DungeonSiege.exe | 0x14CAF | 0F 8E B6 00 00 00 | E9 B7 00 00 00 90 |
| DungeonSiege.exe | 0x249FDD | 55 8B EC | C2 04 00 |

If your bytes don't match or the game crashes, your exe is a different build:

- **Cursor fix:** search for `39 7B 78 0F 8E` and patch the `JLE` that follows to a `JMP` (`E9 B7 00 00 00`). It will leave a spare byte afterwards; set it to `NOP` (`90`).
- **Sword removal:** search for `55 8B EC 81 EC 9C 00 00 00 53 56 8B D9 8D B3 44` and change the first three bytes to `C2 04 00`.

When the mouse is over the game window, the game tells Windows to show no cursor (`SetCursor(NULL)` in its `WM_SETCURSOR` handler) and draws its own instead. That self-drawn UI cursor fails. This patch skips the `SetCursor(NULL)` call, so Windows shows its normal cursor. It only works in windowed mode.

The second patch makes the game's own cursor-drawing routine return immediately, so the sword cursor is never drawn. Dragged item icons are drawn separately, so they still follow the mouse.
