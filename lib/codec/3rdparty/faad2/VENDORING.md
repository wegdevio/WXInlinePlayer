# Vendoring faad2

The wasm build now decodes AAC in-process with faad2 instead of delegating to
the browser's `AudioContext.decodeAudioData`. The decoder source is not checked
in here — drop it in manually before running `lib/codec/build.sh`.

Layout expected by `CMakeLists.txt`:

```
lib/codec/3rdparty/faad2/
├── CMakeLists.txt          # already in repo
├── VENDORING.md            # this file
├── include/                # ← you add this (knik0/upstream layout)
│   └── neaacdec.h          #   public API header
└── libfaad/                # ← you add this
    ├── decoder.c
    ├── syntax.c
    └── ... (all *.c / *.h from upstream libfaad/)
```

Older faad2 forks ship `neaacdec.h` inside `libfaad/` instead of `include/`.
Both layouts work — the CMake config probes for whichever you have.

## Quick steps (knik0/faad2 fork, recommended)

```bash
cd lib/codec/3rdparty
git clone --depth 1 https://github.com/knik0/faad2 faad2-upstream
cp -R faad2-upstream/libfaad  faad2/libfaad
cp -R faad2-upstream/include  faad2/include
rm -rf faad2-upstream
```

If you see `fatal error: 'neaacdec.h' file not found`, you forgot to copy
`include/` — either copy it as above, or move `neaacdec.h` into `libfaad/`.

Any reasonably modern faad2 fork works (Mediainfo's, upstream sf.net, knik0's).
The `libfaad/` source layout has been stable since faad2 2.7.

## License

faad2 is GPLv2-only. Vendoring it makes the whole resulting `*.wasm` GPLv2 —
sources that get statically linked in inherit the copyleft. If that doesn't fit
your distribution model, switch decoder (e.g. fdk-aac, which has a separate
Fraunhofer license) or keep AAC decoding on the browser side as before.
