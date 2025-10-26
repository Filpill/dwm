# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About This Project

This is a customized build of dwm (Dynamic Window Manager), forked from Luke Smith's build. dwm is a minimal, tiling window manager for X11 that is configured through editing C code and recompiling.

## Build and Installation

```bash
# Compile dwm
make

# Install dwm system-wide (requires sudo)
sudo make install

# Clean build artifacts
make clean

# Uninstall
sudo make uninstall
```

**Critical:** After making ANY changes to config.h or dwm.c, you MUST:
1. Run `make` to recompile
2. Restart dwm (typically Mod+Shift+Q or by logging out/in)

Changes do not take effect until dwm is recompiled and restarted.

## Configuration Architecture

### config.h - Main Configuration File
This is the primary configuration file. All customization happens here:

- **Appearance:** borders, gaps, bar settings, colors, fonts
- **Rules array:** Window-specific behavior (floating, tags, positioning)
  - Format: `{class, instance, title, tags, isfloating, isterminal, noswallow, monitor, floatx, floaty, floatw, floath}`
  - `floatx` and `floaty` are proportional (0.0-1.0) positions on the monitor
  - `floatw` and `floath` are proportional (0.0-1.0) window dimensions
  - Set to -1 or 0 to use default behavior
- **Keys array:** All keyboard shortcuts
- **Buttons array:** Mouse button bindings
- **Constants:** Terminal, browser, and application definitions at the top

### Core Code Structure

**dwm.c** (main window manager logic)
- Event-driven architecture using X11 event handlers
- `applyrules()` function (around line 400) applies window rules from config.h
- Custom float positioning is applied at dwm.c:424-435
- Clients are organized in a linked list per monitor
- Tags are implemented as bit masks (not workspaces)

**vanitygaps.c** (layout gaps management)
- Included directly into dwm.c
- Implements gap toggle, increment, and layout-specific gap logic
- Provides multiple layout algorithms: tile, bstack, deck, spiral, dwindle, centeredmaster, centeredfloatingmaster
- `getgaps()` calculates gaps based on window count and smartgaps setting

**shiftview.c** (tag navigation utilities)
- `shiftview()`: Cycles through tags left/right
- `shifttag()`: Moves window to adjacent tag

**drw.c/drw.h** (drawing library)
- Abstraction layer for X11 drawing operations
- Handles fonts, colors, and text rendering

## Patches and Features

This build includes several patches:
- **Systray:** System tray support in the bar
- **Vanitygaps:** Configurable gaps between windows
- **Swallowing:** Terminal windows can be "swallowed" by GUI apps launched from them
- **Scratchpads:** Toggle-able floating windows (defined in `scratchpads[]`)
- **Xresources:** Some settings can be loaded from ~/.Xresources
- **Sticky windows:** Windows can stick across all tags
- **Fullscreen:** Toggle fullscreen mode
- **Custom float positioning:** Per-application float window positioning via rules

## Window Rules and Positioning

When modifying window rules (e.g., positioning gsimplecal):
1. Use `xprop` to identify the WM_CLASS (class and instance) of a window
2. Add/modify the rule in config.h rules array
3. For custom positioning:
   - `floatx = 1.0` positions at right edge
   - `floatx = 0.0` positions at left edge
   - `floaty = 0.0` positions at top
   - `floaty = 1.0` positions at bottom
   - Values are proportional to remaining space (monitor size minus window size)
4. Recompile with `make` and restart dwm

## Tag System

dwm uses tags (1-9), not traditional workspaces:
- Windows can belong to multiple tags simultaneously (bitwise operations)
- `SPTAGMASK` and `SPTAG(i)` macros handle scratchpad tags separately
- Tags are defined in the `tags[]` array in config.h

## Development Patterns

- Configuration is done at compile-time (no config files)
- Keep config.h changes separate from dwm.c modifications
- Test changes by recompiling and restarting dwm
- Use `xprop` for debugging window properties
- Monitor Xorg logs for errors: `~/.local/share/xorg/Xorg.0.log`
