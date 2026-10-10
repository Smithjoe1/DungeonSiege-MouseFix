Dungeon Siege 1 / LOA – Invisible UI Cursor Fix

An issue exists where the mouse cursor in Dungeon Siege and Legends of Aranna disappears during the game when hovered over any 2D element, such as inventory, menus & tooltips. The 3d world cursor still works. 

The two files are patched at the below locations 
File	File offset	Original bytes	Patched bytes
DSLOA.exe	0x1526E	0F 8E B6 00 00 00	E9 B7 00 00 00 90
DungeonSiege.exe	0x14CAF	0F 8E B6 00 00 00	E9 B7 00 00 00 90

When the mouse is over the game window, the game tells Windows to show no cursor (SetCursor(NULL) in its WM_SETCURSOR handler) and draws its own instead. That self-drawn UI cursor fails. This patch skips the SetCursor(NULL) call, so Windows shows its normal cursor. It only works in windowed mode.

Instructions
Back up DSLOA.exe and/or DungeonSiege.exe before replacing with patched files.

Edit the config files below and add "fullscreen = false" to the top line of each:
LOA: Documents\Dungeon Siege LOA\DungeonSiege.ini
Base game: Documents\Dungeon Siege\DungeonSiege.ini
Launch the game. 

You'll see the normal Windows arrow over the UI. The game's sword cursor still shows in the 3D world, and the arrow hides briefly while you drag an item.

Offsets were taken from the original CD release for DungeonSiege.exe with Kallah's DSLOA.exe. 

If your bytes don't match or the game crashes, your exe is a different build. Search for 39 7B 78 0F 8E and patch the JLE that follows to a JMP to E9 B7 00 00 00, it will leave an empty byte afterwards, set to NOP 90 to resolve.

