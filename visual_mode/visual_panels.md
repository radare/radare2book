# Visual Panels

## Concept

Visual Panels is characterized by the following core functionalities:

1. Split Screen
2. Display multiple screens such as Symbols, Registers, Stack, as well as customed panels
3. Menu will cover all those commonly used commands for you so that you don't have to memorize any of them

CUI met some useful GUI as the menu, that is Visual Panels.

## Overview

![Panels Overview](panels_overview.png)

## Commands
```
| Panels commands:
| !                       run r2048 game
| .                       seek to PC or entrypoint
| :                       run r2 command in prompt
| _                       start the hud input mode
| |                       split the current panel vertically
| -                       split the current panel horizontally
| w                       change the current layout of the panels
| ?                       show this help
| X                       close current panel
| m                       move to the menu
| V                       go to the graph mode
| b                       browse symbols, flags, configurations, classes, etc
| c                       toggle cursor
| C                       toggle color
| d                       define in the current address. Same as Vd
| e                       change title and command of current panel
| D                       show disassembly in the current panel
| g                       show graph in the current panel
| *                       show pseudo code/r2dec in the current panel\n
| i                       insert hex
| M                       open new custom frame
| hl                      scroll panels horizontally
| HL                      resize panels horizontally
| jk                      scroll panels vertically
| JK                      scroll panels vertically by page
| sS                      step in / step over
| uU                      undo / redo seek
| pP                      seek to next or previous scr.nkey
| nN                      create new panel with given command
| q                       quit, back to visual mode
```

## Basic Usage

Use `tab` to move around the panels until you get to the targeted panel. Then, use `hjkl`, just like in vim, to scroll the panel you are currently on.
Use `S` and `s` to step over/in, and all the panels should be updated dynamically while you are debugging.
Either in the Registers or Stack panels, you can edit the values by inserting hex. This will be explained later.
While hitting `tab` can help you moving between panels, it is highly recommended to use `m` to open the menu.
As usual, you can use `hjkl` to move around the menu and will find tons of useful stuff there.

## Split Screen

`|` is for the vertical and `-` is for the horizontal split. You can delete any panel by pressing `X`.

## Edit Values

Either in the Register or Stack panel, you can edit the values. Use `c` to activate cursor mode and you can move the cursor by pressing `hjkl`, as usual. Then, hit `i`, just like the insert mode of vim, to insert a value.
