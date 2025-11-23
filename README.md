# RuneScape 2 revision #225 (18 May 2004) C99 port
Portable single-threaded C client for early RS2, the last update before a new cache format and ondemand protocol.

Compatible with [2004Scape](https://github.com/2004Scape/Server), the most accurate runescape remake

Features:
- should work on any 32 bit system with 64 MB of RAM on lowmem, networking and a (read-only) filesystem.
- webassembly build to avoid javascript code being optimized out by the browser.
- WIP ports for most game consoles from 1998 until 2013! See [docs](/docs) for images.
- optional [config.ini](example.ini) file to change client behaviour. Create an empty config.ini to avoid passing cli args.

### Disclaimer
> This project is an **educational prototype** created to study C game server–client architecture. It is **not affiliated with, endorsed by, or sponsored by Jagex Ltd.** “RuneScape®” and “Old School RuneScape®” are **registered trademarks of Jagex Ltd.**

## quickstart for windows
All you need to build for 32 bit windows is included:
* tinycc (C compiler, built with `TCC_C=..\tcc.c` env var and removed bcheck lib)
* all 32 bit SDL dlls, only SDL1 works prior to windows XP and is always 32 bit unlike the others

To build simply run `build.bat` in cmd to get the client.exe, optionally set SDL ver `-v 1|2|3` and C compiler `-c tcc|gcc|emcc`.

SDL1 is default for tcc and old mingw-gcc to target windows 9x, but only SDL2/3 have sfx right now. This (unofficial) release doesn't require msys install: https://github.com/fsb4000/gcc-for-Windows98/releases. mingw-gcc 11 optimizations seem to only be slightly faster than tcc though.

If the client fails to start you either aren't passing cli args and don't have a config.ini OR you are using a SDL dll for the wrong architecture. Delete it and it'll be copied during next build

## Platforms and Compilers
To move the executable you have to take the correct `SDL.dll`, `config.ini`, and the `rom/` directory along with it. The consoles will load it from sdcard if they don't embed the files already.

type `::perf` command ingame to see fps and lrucache size

all home consoles (wii, dreamcast, xbox) should be able to run the game at higher res or even full res on PAL TVs so you don't have to pan, but this isn't set up and emulators don't support many video modes.

When adding a new platform also add system ttf font closest to helvetica in gameshell_draw_string when available to avoid Roboto dependency.

To be able to run some emulators on WSL2 you may need to prefix `MESA_GL_VERSION_OVERRIDE=4.6 MESA_GLSL_VERSION_OVERRIDE=460`.

If tcc from your package manager isn't working you should build latest [tcc](#tools) from source

[v86](#tools) is a x86 PC emulator running in the browser, including older windows.

### Windows 95 to Windows 11
build.bat(32 bit): tcc (included), mingw-gcc, emcc

run.ps1: cl, clang, tcc, mingw-gcc, emcc

You might want the updated [PowerShell](#tools) for run.ps1

```
TODO: add wav sfx to complete SDL1 platform for win9x
TODO: make win9x compatible batch file (no delayed expansion?) right now needs to build from more modern system
TODO: clean up ps1 script so it doesn't need to be modified

NOTE: on v86 PC emulator the cursor flickers on win95, and colours on win9x are wrong? win2k is fine
```

### Linux GNU or musl
Makefile: gcc, clang, tcc, mingw-gcc, emcc

arm+musl platforms like postmarketOS can use tcc but it requires some small tweaks:
- comment out wchar_t in include/stddef.h
- build with: https://lists.gnu.org/archive/html/tinycc-devel/2022-02/msg00038.html
TODO SDL3 fails to init?

### FreeBSD
Install sdl1/sdl2 or sdl3+pkgconf and run `gmake SDL=1/2/3`

### MacOS
TODO

### Web (clang)
Install clang and get [wasmlite](#tools) (you need the libc and generated index.html)
then run `make -f wasm.mk run DEBUG=0` with correct sysroot path.

You must add `?client` to the URL and optionally append `&arg 1&arg 2&arg 3&arg 4`.

You can configure ip and port in config.ini.

The only needed files are the index.html + client.wasm and optionally the soundfont/config.ini relative to it.

enable cors in server web.ts with `res.setHeader('Access-Control-Allow-Origin', '*');`

```
TODO fwrite maps like emscripten
TODO add to build.bat/ps1 to replace emscripten
```

### Web (emscripten)
Install [emsdk](#tools)
run `emmake make`/`make CC=emcc` or `build.bat -c emcc` for windows

For make you can append `run` to start a http server and `DEBUG=0` to optimize. Then go to `ip:port/client.html` (or another entrypoint)

Pass 4 args in shell.html to use the ip + port from URL instead of config, otherwise set http_port to 8888 in config for linux servers.

The only needed files are the index.`html,js,wasm` and optionally the soundfont/config.ini relative to it.

enable cors in server web.ts with `res.setHeader('Access-Control-Allow-Origin', '*');`

```
TODO: audio stream is pushed to on same thread causing scape_main stutters, and lowmem w/o audio speeds up (typescript client uses absolute time for idlecycles)
TODO: use indexeddb (add cacheload and cachesave), and maybe add [web worker clientstream](https://emscripten.org/docs/api_reference/wasm_workers.html)
TODO: mobile controls: touch on release + touch to rotate + osk + mouse+kbd, PWA manifest

NOTE: could replace sdl3 audio with https://emscripten.org/docs/api_reference/wasm_audio_worklets.html and decodeAudioData for wavs
NOTE: JSPI decreases output size a lot, but asyncify can be used for older browser compatibility. Windows and Linux output size might differ and sigint on Windows will cause terminate batch job message if using emrun.
NOTE: unused old worldlist code: [shell.html](https://github.com/lesleyrs/Client3/commit/5da924b9f766005e82163d899e52a5df2f771584#diff-c878553ed816480a5e85ff602ff3c5d38788ca1d21095cd8f8ebc36a4dbc07ee)
```


## libraries
* [micro-bunzip](https://landley.net/code/) | https://landley.net/code/bunzip-4.1.c
* [isaac](https://burtleburtle.net/bob/rand/isaacafa.html) | https://burtleburtle.net/bob/c/readable.c
* [TinySoundFont](https://github.com/schellingb/TinySoundFont) - with fix for attack1.mid by skipping RIFF header and android support
* [tiny-bignum-c](https://github.com/kokke/tiny-bignum-c) - prefer libtom/openssl/bigint, but works fine with smaller exponent
* [LibTomMath](https://github.com/libtom/libtommath) | mpi.c is from gen.pl script in [releases](https://github.com/libtom/libtommath/releases/latest). with added ifdefs to fix non-gcc builds.
* [ini](https://github.com/rxi/ini)
* [stb_image and stb_truetype](https://github.com/nothings/stb)

## optional libraries
* [OpenSSL](https://github.com/openssl/openssl) | https://wiki.openssl.org/index.php/Binaries

* [SDL-1.2](https://github.com/libsdl-org/SDL-1.2) | [SDL-2/SDL-3](https://github.com/libsdl-org/SDL) | https://libsdl.org/release/

Using prebuilt SDL but removed tests, SDL1 mingw dotfiles + SDL1 tcc fixes in VC (fixed upstream but no new releases since 2012)

## tools
* [tcc](https://github.com/TinyCC/tinycc) | https://bellard.org/tcc/
* [emsdk](https://github.com/emscripten-core/emsdk) | https://emscripten.org/docs/getting_started/downloads.html
* [devkitpro](https://github.com/devkitPro) | https://devkitpro.org/
* [pspdev](https://github.com/pspdev/pspdev) | https://pspdev.github.io/
* [vitasdk](https://github.com/vitasdk/vdpm) | https://vitasdk.org/
* [kallistios](https://github.com/KallistiOS/KallistiOS) | [mkdcdisc](https://gitlab.com/simulant/mkdcdisc)
* [nxdk](https://github.com/XboxDev/nxdk)
* [android command line tools](https://developer.android.com/studio)
* [powershell](https://github.com/PowerShell/PowerShell)
* [v86](https://github.com/copy/v86.git) | https://copy.sh/v86/
* [wasmlite](https://github.com/lesleyrs/wasmlite)
