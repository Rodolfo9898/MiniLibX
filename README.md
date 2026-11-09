*This project has been created as part of the 42 curriculum by rperez-t.*

# MiniLibX

## Description

The MiniLibX given by 42 for the graphic projects (fract-ol, so_long, cub3d, ...), for
Linux and for macOS, kept together in one repository. It is not my code: these are the
sources as distributed by 42, unchanged, so that a project can pull both versions with a
single `git submodule` and pick the right one for the machine at build time.

| Folder | What it is |
|---|---|
| `linux/` | The X11 version, [42Paris/minilibx-linux](https://github.com/42Paris/minilibx-linux). It has its own `README.md`, man pages in `man/` and a test program in `test/` |
| `mac/` | The native macOS version (OpenGL and AppKit) that 42 provides for its Macs |

Both build a static `libmlx.a` and expose the same `mlx.h` API (`mlx_init`,
`mlx_new_window`, `mlx_new_image`, `mlx_hook`, `mlx_loop`, ...). The two versions do not
behave exactly the same: key codes differ between X11 and macOS, and some functions exist
on only one side (`mlx_destroy_display` is Linux only, for example). Check the `mlx.h` of
the folder you build.

## Instructions

### Requirements

- **Linux**: `gcc` or `cc`, `make`, and the X11 development files. On Debian/Ubuntu:
  `sudo apt-get install gcc make xorg libxext-dev libbsd-dev`
- **macOS**: the Xcode command line tools (`xcode-select --install`). No X11 needed: the
  `mac/` version uses OpenGL and AppKit directly.

### Using it as a submodule

Add it to your project, for example under `libs/MiniLibX`:

```sh
git submodule add https://github.com/Rodolfo9898/MiniLibX.git libs/MiniLibX
```

And when cloning a project that already has it:

```sh
git clone --recurse-submodules <project-url>
# or, in an existing clone
git submodule update --init --recursive
```

### Building

On Linux:

```sh
make -C linux    # runs ./configure, writes Makefile.gen, builds libmlx.a and libmlx_Linux.a
```

On macOS:

```sh
make -C mac      # builds libmlx.a
```

### Picking the version in a Makefile

Select the folder and the link flags from the system:

```make
OS = $(shell uname)

ifeq ($(OS), Linux)
    MLX     = $(LIBS_DIR)/MiniLibX/linux/
    MLX_LNK = -lXext -lX11 -lm
else ifeq ($(OS), Darwin)
    MLX     = $(LIBS_DIR)/MiniLibX/mac/
    MLX_LNK = -framework OpenGL -framework AppKit
endif

MLX_LIB = $(MLX)libmlx.a
MLX_INC = -I$(MLX)
```

Compile with `$(MLX_INC)` so `#include "mlx.h"` finds the header of the current system,
and link with `$(MLX_LIB) $(MLX_LNK)`.

### Cleaning

- `make -C linux clean` runs `./configure clean`, which needs `Makefile.gen` to exist (it
  is created by the first build). It removes `obj/`, `libmlx.a` and `libmlx_Linux.a`, but
  leaves `Makefile.gen` and `test/Makefile.gen` behind: remove them yourself if you want a
  clean tree.
- `make -C mac clean` removes `libmlx.a` and the objects.

Build output is ignored by the `.gitignore` of each folder, so a build does not dirty the
submodule.

### Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `X11/Xlib.h: No such file or directory` | X11 headers missing: `sudo apt-get install xorg libxext-dev` |
| `undefined reference to XShmCreateImage` (or other `X...`) | Link flags missing or in the wrong order: put `-lXext -lX11 -lm` after `libmlx.a` |
| `make: *** No rule to make target 'clean'` or `Makefile.gen: No such file` on Linux | `clean` was run before any build: nothing to clean |
| `Can't open display` | No X server reachable (SSH without `-X`, a container without the host display) |
| Empty `libs/MiniLibX` folder in a project | The submodule was not fetched: `git submodule update --init` |

## Resources

- The `linux/README.md` and the man pages: `man ./linux/man/man3/mlx.3`,
  `mlx_new_window.3`, `mlx_new_image.3`, `mlx_pixel_put.3`, `mlx_loop.3`
- minilibx-linux upstream - https://github.com/42Paris/minilibx-linux
- 42 Docs, MiniLibX - https://harm-smits.github.io/42docs/libs/minilibx
- `linux/LICENSE`: BSD 2-Clause, Copyright (c) 2021, Ecole 42
