# HighWire

A web browser for **Atari 68k** computers — TOS, FreeMiNT and MagiC.

HighWire renders today's web on hardware from the late 1980s: TLS 1.3, CSS,
downloadable web fonts and modern image formats, on a 68030 with a few megabytes
of RAM. It has been in development, on and off, since the 1990s.

**Current release: v0.5.0 Beta 2.**

This repository holds the project's landing page, published at
<https://highwire-browser.github.io>. The page is a placeholder for now.

## What's in Beta 2

- Tabs and a new navigation toolbar, drawn with VDI primitives so they scale
  with the screen rather than being fixed bitmaps
- HTTPS via Mbed TLS — TLS 1.2 and 1.3, with connection pooling, because on a
  68k the handshake costs more than the transfer
- CSS layout: floats, tables, inline-block, pseudo-classes and pseudo-elements
- Downloadable `@font-face` web fonts
- PNG, JPEG, GIF, WebP and SVG images
- True colour where the hardware provides it
- `about:settings`, `about:fonts` and `about:cache`

JavaScript is present but switched off in this release. It is not yet reliable
enough on real sites to ship enabled by default.

## Requirements

| | |
|---|---|
| CPU | 68030 minimum; **68060 strongly suggested** |
| Memory | 8 MB workable, more is better |
| System | TOS, FreeMiNT or MagiC |
| Graphics | NVDI, SpeedoGDOS or fVDI |
| Network | MiNTnet, STiK2, STinG or IConnect, loaded as a runtime module |

## Why this organisation exists

Building HighWire means building for a platform nobody ships binaries for any
more, against libraries that mostly assume a little-endian machine with a modern
libc. Several of those libraries need patching before they work at all on a
68k — and those patches lived, for years, scattered across one developer's disk.

This organisation exists so that everything needed to build HighWire is in one
place, at the versions it was actually built against. If you want to change the
browser and rebuild it, you should not first have to rediscover which libpng
release we used, or work out for yourself why AES-GCM fails on big-endian.

## The forks, and what we changed

Both are published as forks so the diff against upstream stays visible. Neither
has been submitted upstream.

### [`mbedtls`](https://github.com/Highwire-Browser/mbedtls) — branch `highwire-m68k-bigendian`

**Mbed TLS does not work correctly on big-endian platforms.** Three separate
bugs, each breaking a different stage of a TLS 1.3 handshake, all still present
in upstream 3.6.6:

| file | function | symptom |
|---|---|---|
| `library/bignum_core.c` | `mbedtls_mpi_core_read_le()` | `MBEDTLS_ERR_ECP_INVALID_KEY` — X25519 key exchange fails |
| `library/gcm.c` | `gcm_mult_smalltable()` | `MBEDTLS_ERR_SSL_INVALID_MAC` — every AES-GCM suite fails |
| `library/bignum_core.h` | `GET_BYTE()` | RSA certificate signatures verify as garbage |

Two of those break the *default* modern TLS 1.3 configuration, so this affects
any big-endian target — PowerPC, SPARC, s390x, MIPS BE — not only m68k.

There is also a trap that is not a patch: 3.6.6 changed the default of
`MBEDTLS_PLATFORM_DEV_RANDOM` from `/dev/urandom` to `/dev/random`. On FreeMiNT
`/dev/random` blocks until the entropy pool fills, and a machine idling at a
splash screen never fills it — so the application hangs inside
`psa_crypto_init()` with no error and no log line. See `HIGHWIRE-BIGENDIAN.md`
on that branch.

### [`nanosvg`](https://github.com/Highwire-Browser/nanosvg)

Altered source version, marked as such per the zlib licence. Adds `<clipPath>`
and `clip-path="url(#id)"` support carried through to the rasteriser, which
previously ignored clipping entirely; nested `<svg>` viewports, so sprite sheets
render as intended; and a fix for `objectBoundingBox` gradient coordinates,
which were resolved against the wrong box and came out flat.

## Building

You need the **m68k-atari-mintelf** cross toolchain — GCC targeting ELF for
Atari. Thorsten Otto builds and hosts it:

- Cross compilers: <https://tho-otto.de/crossmint.php>
- Snapshots and library packages: <https://tho-otto.de/snapshots/crossmint/>

[gemlib](https://github.com/freemint/gemlib) and
[cflib](https://github.com/freemint/cflib) come from the FreeMiNT project and are
installed separately rather than mirrored here.

The browser source is not yet public.

## Credits

HighWire is the work of **The HighWire Project**.

The browser engine — parser, scanner, renderer, scheduler and layout — and the
network overlay modules are substantially the work of **Ralph ("AltF4")** and
**Dan ("Baldrick")**, built on the original Project HighWire source release by
**Robert Goldsmith**.

## Licence

Released under the zlib licence. Copyright in each contributor's work remains
with its author. This work is based, in part, on the Project HighWire source
release by Robert Goldsmith.
