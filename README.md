# Daily Standup

## 28 April 2023

**Good News**: Day One

**Yesterday**: Yak shaving around getting exiv2 to cross-compile to wasm.

**Today**: Get a basic project page up for the midjourney metadata embed. Hopefully nail down the wasm build.

**Blockers**: Wasm build!

### Notes

For background, here is the project I have in mind:

Embed job parameters from Midjourney as image metadata.

Wherever the image goes, the prompt can go with it. Fewer cases of people asking, "hey share prompt plz" This would also be friendly to tooling - eg, extracting alt tags.

Random thought: is there a standard metadata for alt tags? XMP Description? What about exif?

Would like this to run 100% in browser as a Chrome extension, but there doesn't seem to be good options for writing image metadata in JS. So that means WebAssembly.

Not gonna lie, it gives me an excuse to use this WASM thing too. Reflecting back on yesterday ... I should have known better.

#### Image metadata writer evaluation

Looked at ffmpeg: image metadata support is not amazing. There is something, but unclear the full extent; in any case it would be cumbersome.
exiftool: this is what everyone seems to use for metadata, but it's in perl. Not sure how I feel about shipping a [wasm perl interpreter](https://webperl.zero-g.net)
exiv2: C++ so probably doable. Got it to compile on my box without too much fuss.

Okay, now on to cross-compiling for wasm...

#### Wasm yak shaving

Meta note: I try to use nixpkgs based environments whenever possible for reproductibility.
Yesterday was probabably the first time I've tried packaging CMake and meson based projects as a Nix derivation and everything worked out of the box with the default mkDerivation build configuration; just add `cmake` or `meson ninja` to the nativeBuildInputs. See [this gist](https://gist.github.com/j0sh/78eadeb628956de3f09b9ea28ea6fa8d) for an example.

Of course, I had to dig into _how_ that worked and naturally it is a [bit kludgy](https://github.com/NixOS/nixpkgs/issues/18678#issuecomment-569477884) under the hood, but makes for quite a smooth DX when it works.

"When it works" is carrying a lot of weight herre... hit a few roadblocks in getting exiv2 actually cross-compile for wasm. This is partly due to the maturity of the tooling, but also due to my general unfamiliarity with the space. I've done a bunch of cross-compiling over the years, but the wasm landscape right now seems uniquely fragmented:

1. whether llvm is [good enough](https://github.com/ern0/howto-wasm-minimal) out of the box, eg via the `--target wasm32` flag? (with nix shell, I hit roadblocks with the `wasm-ld` linker not being found, althogh clang seems to have picked up on the target correctly. missing linker flags?)
1. what about the [tooling](https://github.com/WebAssembly/wasi-sdk) distributed by the webassembly project itself? I believe this would be an ideal use of nixpkg's [global stdenv replacement](https://nixos.wiki/wiki/Using_Clang_instead_of_GCC) capability; I just haven't figured out how to actually use it yet. 
1. or just use [emscripten](https://emscripten.org/docs/compiling/Building-Projects.html)? This has composability issues with nixpkgs and with out-of-the-box cmake / clang derivations (although I could just override the configure / build phases for now).
1. how to get all this to actually using [Nix](https://github.com/NixOS/nixpkgs). Nix is me doing things the hard way; I'll have lots more to say about this another day.
1. oh, first I gotta compile all the exiv2 dependencies too... anyone have tips for getting a [wasm cross-build](https://github.com/mesonbuild/meson/blob/master/cross/wasm.txt) for meson? (I am not excited about figuring out yet another damn build system that I will otherwise never have to touch.) What about with nixpkgs? Luckily the meson-using library is an [ini parser](https://github.com/benhoyt/inih), which appears to only be used by exiv2 for a configuration file, which I don't need support for... and looks like this can be ifdef'd out so fingers crossed the definition is surfaced via cmake.
1. Nixpkgs has semblance of [wasm packaging via emscripten](https://github.com/NixOS/nixpkgs/blob/06567334beec3fe0f33ed8f91b33a4a195a3b9ba/doc/languages-frameworks/emscripten.section.md) (with zlib! png support!) but the overall approach seems to be in [flux](https://github.com/NixOS/nixpkgs/pull/217428).
