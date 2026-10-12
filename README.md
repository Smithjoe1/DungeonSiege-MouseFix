# Dungeon Siege 1 / LOA – Cursor & Fullscreen Fix

An issue exists where the mouse cursor in Dungeon Siege and Legends of Aranna disappears during the game when hovered over any 2D element, such as inventory, menus & tooltips. The 3D world cursor still works.

These patched executables make the game use the normal Windows cursor everywhere, in windowed or fullscreen mode. They also remove the game's own sword cursor, and stop the cursor jumping or hiding when you click.

The patched files are based on the original CD release of `DungeonSiege.exe` and Kallah's `DSLOA.exe`.

## Instructions

1. Back up your original `DSLOA.exe` and/or `DungeonSiege.exe`.
2. Replace them with the patched files.
3. Choose windowed or fullscreen in the config files by adding `fullscreen = false` or `fullscreen = true` to the top line:
   - LOA: `Documents\Dungeon Siege LOA\DungeonSiege.ini`
   - Base game: `Documents\Dungeon Siege\DungeonSiege.ini`
4. Launch the game.

You'll see the normal Windows arrow over the whole game. Left-clicks and item drags never move or hide it. During middle/right-button camera drags it stays visible and pinned where you clicked, and the camera keeps turning without stopping at the screen edge.

## Patches

Each patch is listed as the bytes to search for and what they become. `xx` bytes are addresses that differ between the two files and are left unchanged (or, for `CALL` instructions, adjusted to point at the same function from the new location).

### DSLOA.exe & DungeonSiege.exe

| Purpose | Original bytes | Patched bytes |
|---|---|---|
| Show the Windows cursor over the game (skip `SetCursor(NULL)`) | `39 7B 78 0F 8E B6 00 00 00` | `39 7B 78 E9 B7 00 00 00 90` |
| Remove the game's sword cursor | `55 8B EC 81 EC 9C 00 00 00 53 56 8B D9 8D B3 44` | `C2 04 00 81 EC 9C 00 00 00 53 56 8B D9 8D B3 44` |
| Don't hide the cursor at startup in fullscreen | `80 7D 74 00 74 07 57 FF 15` | `80 7D 74 00 EB 07 57 FF 15` |
| Re-show the cursor when the window activates in fullscreen | `85 C0 7D F8 80 7B 74 00 0F 85 xx xx xx xx 6A 01 FF D6` | `85 C0 7D F8 80 7B 74 00 90 90 90 90 90 90 6A 01 FF D6` |
| Don't capture the mouse just because the game is fullscreen | `38 41 74 74 06 F6 41 4C 01` | `38 41 74 EB 06 F6 41 4C 01` |
| Don't warp the cursor to the screen centre in fullscreen | `80 79 74 00 74 4A 8B 81 C8 00 00 00` | `80 79 74 00 EB 4A 8B 81 C8 00 00 00` |
| Left button no longer grabs the mouse (button handler) | `8B BE 0C 01 00 00 B8 80 0F 00 00 23 F8` | `8B BE 0C 01 00 00 B8 00 0F 00 00 23 F8` |
| Left button no longer grabs the mouse (capture check) | `66 F7 81 0C 01 00 00 80 0F 75 0B` | `66 F7 81 0C 01 00 00 00 0F 75 0B` |
| Middle/right drags hold the cursor at the click position instead of the screen centre | Block A (below) | Block A (below) |
| Middle/right drags store the click position, and never hide the cursor | Block B (below) | Block B (below) |

### Block A – capture warp

Original (74 bytes):
```
8B 81 C8 00 00 00 2B 81 C0 00 00 00 99 2B C2 D1 F8 89 81 34 01 00 00 8B
81 CC 00 00 00 2B 81 C4 00 00 00 99 2B C2 8B 91 C4 00 00 00 D1 F8 03 D0
89 81 38 01 00 00 8B 81 34 01 00 00 03 81 C0 00 00 00 52 50 FF 15 xx xx
xx xx
```
Patched:
```
8B 81 34 01 00 00 8B 91 38 01 00 00 03 81 C0 00 00 00 03 91 C4 00 00 00
52 50 FF 15 xx xx xx xx EB 28 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90
```

### Block B – button press/release

Search for `3B C1 74 44` followed by the original bytes below. The block is 45 bytes, plus the following 6-byte show check.

Original:
```
6A 01 8B CE E8 xx xx xx xx 85 FF 74 20 84 C0 75 1C 8B 86 C4 00 00 00 03
44 24 14 50 8B 86 C0 00 00 00 03 44 24 14 50 FF 15 xx xx xx xx
80 7E 74 00 75 11
```
Patched (`CALL` target adjusted to the same function):
```
85 FF 75 14 8B 44 24 10 89 86 34 01 00 00 8B 44 24 14 89 86 38 01 00 00
6A 01 8B CE E8 xx xx xx xx EB 0A 90 90 90 90 90 90 90 90 90 90
80 7E 74 00 EB 11
```

## How it works

When the mouse is over the game window, the game tells Windows to show no cursor (`SetCursor(NULL)` in its `WM_SETCURSOR` handler) and draws its own instead. That self-drawn UI cursor fails. The cursor fix skips the `SetCursor(NULL)` call, so Windows shows its normal cursor.

The sword removal makes the game's own cursor-drawing routine return immediately, so the sword cursor is never drawn. Dragged item icons are drawn separately, so they still follow the mouse.

In fullscreen the game also hides the Windows cursor on startup and on window activation, and grabs the mouse by warping it to the screen centre every frame. These patches remove that, so fullscreen handles the mouse the same way windowed mode does.

While a mouse button is held, the game grabs the mouse so camera drags don't stop at the screen edge. The left button no longer triggers this. Middle/right drags hold the cursor at the click position instead of the screen centre, and the cursor is never hidden.
