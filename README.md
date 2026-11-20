# StormByte.Repository.Patches
StormByte's custom patches to allow gentoo system to be built with clang

## FFmpeg 9.1

The patches in [media-video/ffmpeg](media-video/ffmpeg/) are applied in filename
order. For 9.1, `0008` has updated API-test Makefile context; `0007`, `0010`
and `0011` are still needed and apply without changes.

Removed `0009` (Opus active-channel delay pointers) and `0012` (x265 generated
Dolby Vision RPU cleanup): both fixes are already present in the pristine
9.1 release. Remove these obsolete files from
`/etc/portage/patches/media-video/ffmpeg/` as well when deploying this series.

Validated against the signature-verified 9.1 release: all four remaining
patches apply in order with zero fuzz, and the affected codec objects compile.
`fate-api-seek-byte-mkv` passes with `0007` and fails without it (zero packets
after the byte seek instead of four), confirming that fix is still needed.

## mysql-connector-c 26.10.0 on musl

[Resolver compatibility patch](dev-db/mysql-connector-c-26.10.0/0001-detect-reentrant-resolver-for-musl.patch)
detects the complete `res_n*` API and uses the existing `res_search` fallback
when it is unavailable, including on Gentoo musl (not just Alpine).
Copy it into `/etc/portage/patches/dev-db/mysql-connector-c-26.10.0/`
and rebuild the package with `emerge -1av =dev-db/mysql-connector-c-26.10.0`.
