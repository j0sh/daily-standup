# Daily Standup

### 5 May 2023

**Good News**: Making solid progress on the wasm + deno build

**Past Week**: WASM build for Exiv2 and friends complete.

**Today**: Updated Midjourney metadata [project repo](https://github.com/j0sh/midjourney-metadata) with work so far. Parameterize image metadata writer to take a list of new XMP keys + values.

### Notes

It's been too long since the last update; I am failing at this daily standup thing.

In any case, the WASM stuff is in pretty good shape. It's taken me longer than I care to admit - all week, basically - but I have a pretty comfortable grasp of things at this point:

* Getting things even to *build* in emscripten, including exiv2 dependencies zlib and expat. [Work log](https://gist.github.com/j0sh/ddcd773f2d5066b2511096620702ece6)
* [Running](https://github.com/j0sh/midjourney-metadata/blob/02e969f58526eb34e53a26f791bb410aa5145c39/reader.ts) emscripten-compiled stuff with Deno. This [Github comment](https://github.com/emscripten-core/emscripten/issues/13190) in particular turned out to be a lifesaver.
* Passing in large blobs from JS to wasm. This is just using the built-in [Uint8Array-to-string](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html?highlight=memory#built-in-type-conversions) conversion. Presumably this involves copying data under the hood; this can probably be optimized out later.
* Returning large blobs from wasm to JS. This [SO question](https://stackoverflow.com/questions/65566923/is-there-a-more-efficient-way-to-return-arrays-from-c-to-javascript) helped a lot, via [this Github issue](https://github.com/emscripten-core/emscripten/issues/5519).
* [Full reproductibility](https://github.com/j0sh/midjourney-metadata/tree/02e969f58526eb34e53a26f791bb410aa5145c39/nixpkgs) via [Nixpkgs](https://github.com/NixOS/nixpkgs) and a [Nix shell](https://github.com/j0sh/midjourney-metadata/blob/02e969f/shell.nix)  (I am not cool enough yet to be using [Flakes](https://nixos.wiki/wiki/Flakes).)
* Understanding *which* metadata format I should be using. [XMP](https://en.wikipedia.org/wiki/Extensible_Metadata_Platform) seems to be the way to go here, since Exif typically has a [fixed set](https://exiv2.org/tags.html) of tags.
* [Reading](https://github.com/j0sh/midjourney-metadata/blob/02e969f58526eb34e53a26f791bb410aa5145c39/reader.cpp) Exif + XMP image metadata via Exiv2 API
* [Writing](https://github.com/j0sh/midjourney-metadata/blob/02e969f58526eb34e53a26f791bb410aa5145c39/writer.cpp) XMP image metadata via Exiv2 API

Functionally, what's left is:
* Update metadata writer to take a list of keys + values to write
* Wire in Midjourney, starting with the community showcase (Deno script)
* Port stuff to a Chrome extension

The emscripten compilation process is pretty slow (especially configure steps) but it's all tolerable for now. A bigger concern is size of the generated JS; right now it is a whopping 3.3 MB, and 1.2 MB gzipped. There are probably ways to cut this down - disabling unneeded features, compiler optimization flags, maybe [closure compiler](https://developers.google.com/closure/compiler) - but that's for later. And since this will ultimately be packaged as a self-contained browser extension, maybe it doesn't matter so much.

### 1 May 2023


**Good News**: First Monday solo!

**Last Friday**: Got the [project page](https://github.com/j0sh/midjourney-metadata) up for the Midjourney image metadata embed. Figured out a few more internal APIs that the MJ front-end uses. Didn't have a chance to actually dig into the WASM build.

**Today**: Finish up the WASM build for the [exiv2](https://exiv2.org) image metadata tool. See the [April 28 notes](https://github.com/j0sh/daily-standup#28-april-2023) for more detail on that.

**Blockers** Still the wasm build. Can't achieve the goal of embedding image metadata completely in-browser, without a metadata tool that also works in the browser.

### Notes

The Midjourney internal API is pretty easy to figure out, so I won't get into too much detail here; it's a good exercise for aspiring scrapers. But as is the case with all APIs, especially internal-facing ones, it is pretty interesting to examine just what is on display: upcoming features and experiments (video?!?), discarded features, internal structure, accumulated cruft.

At a quick glance, the design of the API itself is actually pretty nice. The "latest updates" feed is heavily parameterized (resembling GraphQL but with a more sensible query string based approach), and there is an easy way query parameters for individual jobs in a batch. It would be interesting to know the data structures that are powering the feed on the backend, since indexing over arbitrary fields can get expensive. The only odd appendage I've found so far is that the job seed is found in a separate endpoint, which also returns a subset of job parameters that are available elsewhere... you can almost see the evolution of the API here.

MJ could have made this a *lot* harder to deal with - see some of the heroic efforts in [TikTok reversing](https://www.nullpt.rs/reverse-engineering-tiktok-vm-1) - but I am hoping they won't feel the need to, partly to encourage the community to build neat things on top of it. In all likelihood, the team is probably slammed with scaling the service, and this stuff is the least of their concerns.


## 28 April 2023

**Good News**: [Day One](https://www.aboutamazon.com/news/company-news/2016-letter-to-shareholders)

**Yesterday**: Yak shaving around getting exiv2 to cross-compile to wasm.

**Today**: Get a basic project page up for the midjourney metadata project. Hopefully nail down the wasm build.

**Blockers**: Wasm build!

### Notes

For background, here is the project I have in mind:

**Embed job parameters from Midjourney as image metadata**

Wherever the image goes, the prompt can go with it. Fewer cases of people asking, "hey share prompt plz" This would also be friendly to tooling - eg, extracting alt tags automatically.

Random thought: is there a standard metadata for alt tags? XMP Description? What about exif?

Would like this to run 100% in browser as a Chrome extension, but there doesn't seem to be good options for writing image metadata in JS. So that means WebAssembly.

Not gonna lie, it gives me an excuse to use this WASM thing too. Reflecting back on yesterday ... I should have known better.

#### Image metadata writer evaluation

1. ffmpeg: image metadata support is not amazing. There is something, but unclear the full extent; in any case it would be cumbersome.
2. exiftool: this is what everyone seems to use for metadata, but it's in perl. Not sure how I feel about shipping a [wasm perl interpreter](https://webperl.zero-g.net)
3. exiv2: C++ so probably doable. Got it to compile on my box without too much fuss.

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
