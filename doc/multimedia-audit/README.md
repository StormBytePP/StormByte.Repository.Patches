# Multimedia audit patches

Exported on 2026-11-02 from the Multimedia audit's
`doc/logs/reproduce/patches.tsv` and `doc/PR-DRAFTS.md`, plus the two
Matroska seek patches created earlier in the same audit.

There are **66 distinct applicable fixes for ten packages**: all 64 fixes in
the upstream-submission manifest and the Matroska fix/regression-test pair.
The package directories contain 67 patch files because OpenH264 2.7.0 needs
one version-specific replacement. Nine additional patch files below this
documentation directory preserve originals or a non-applicable build fix.
No existing repository files were replaced.

`inventory.tsv` records every exported file, its source path, base revision,
upstream commit where available, and source/export SHA-256 checksums. Paths
in its `export_path` column are relative to this repository root; source paths
are relative to the Multimedia repository root.

## Gentoo package mapping

Names, installed versions, source origins, and preparation roots were checked
against `/var/db/pkg` and the local Gentoo repository at
`/var/db/gentoo/repos/gentoo`. No third-party fork package was substituted.

| Gentoo package | Fixes | Installed version | Audited base |
|---|---:|---|---|
| `media-libs/fdk-aac` | 20 | 2.0.3 | `716f4394641d53f0d79c9ddac3fa93b03a49f278` (v2.0.3) |
| `media-libs/kvazaar` | 10 | 2.3.2 | `6040962bed5cc68c5ad01234c38c08b8b2822068` (v2.3.2) |
| `media-libs/libvmaf` | 2 | 3.2.1 | `74f930d17de8c58e7c2ce55e2b897b0030ebd8ef` |
| `media-libs/libvorbis` | 5 | 1.3.7-r2 | `0657aee69dec8508a0011f47f3b69d7538e9d262` (v1.3.7) |
| `media-libs/openh264` | 2 | 2.7.0 | `652bdb7719f30b52b08e506645a7322ff1b2cc6f` (v2.6.0) |
| `media-libs/svt-av1` | 10 | 4.1.0 | `9292ec8e32bce26f781f277ec8739b53426c4300` (v4.2.0) |
| `media-libs/x264` | 1 | 0.0.20240513-r1 | `00852f599572d10798b2dafd2c5286213d0ce795` |
| `media-libs/x265` | 8 | 4.3 | `e444744c03978c1fb4e037168967020cf2648427` (4.2) |
| `media-sound/lame` | 2 | 3.100-r3 | `1f5cc9487284d5950343aa5d4f70de433468070a` |
| `media-video/ffmpeg` | 6 | 9.0.2 | `5a215d7ce4de21ec5c34214994eea0ac13dc9054` |

### Application order and Gentoo adaptations

Preserve the numeric filename prefixes. Gentoo's `eapply_user` combines
matching package/version directories and applies patch basenames in POSIX
sort order, with version-specific files overriding identically named
package-wide files. Unrelated lower-numbered bundled compatibility patches
are not prerequisites for these fixes.

The local Portage implementation uses GNU `patch -p1`; email headers from
`git format-patch` are accepted without converting the patches to plain
diffs. Apply these as Gentoo user patches, not as `git am` commits.

* **x265:** Gentoo's official ebuild sets `S="${S}/source"` and prepares
  patches there. The eight active copies remove `source/` only from
  `diff --git`, `---`, and `+++` file paths so `-p1` addresses the correct
  files. Commit metadata, hunk content, and original email footer whitespace
  are preserved. Exact original upstream-format emails are retained under
  `upstream/x265/`. The bundled origin is Multicoreware's Bitbucket repository;
  Gentoo 4.3 uses the official Multicoreware GitHub release, not an unrelated
  x265 fork.
* **OpenH264 2.7.0:** upstream already has the `BsFlush` guard from part of
  `0001-golomb-avoid-empty-word-shifts.patch`, but still needs its
  `BsWriteBits` guard. The original complete patch remains package-wide.
  `media-libs/openh264-2.7.0/` supplies the same-basename override containing
  only the missing hunk. Gentoo automatically selects this override for
  2.7.0, including ebuild revisions; `0002` still comes from the package-wide
  directory. Do not manually apply both versions of `0001`.
* **Meson-only FFmpeg fix:** the audit also created
  `0006-meson-vaapi-check-name.patch`. Gentoo's official FFmpeg ebuild uses
  upstream `configure`/Make, not Multimedia's Meson port, so this patch is
  preserved under `non-applicable/` and deliberately not activated.
* **Other bundled patches:** older Windows/compiler, CMake/pkg-config,
  shallow-version, MOV metadata/subtitle, and similar patches were not
  created by this audit and were not copied indiscriminately.

## Verification and version caveats

All 66 active fixes apply in order to their documented source bases using
GNU `patch -p1 --fuzz=0 --batch --forward --no-backup-if-mismatch`. This
includes the email patches and the x265 source-directory adjustment.
Unadapted copies and archived originals match their source bytes exactly;
x265 active copies differ only in the documented path metadata.

Additional patch-application checks, without fetching or building:

| Installed source | Result |
|---|---|
| FFmpeg 9.0.2 cached release archive | All 6 patches apply without fuzz |
| FDK-AAC 2.0.3, Kvazaar 2.3.2, Vorbis 1.3.7 | Installed upstream releases match the audited bases |
| SVT-AV1 v4.1.0 local Git tag | All 10 patches apply without fuzz |
| VMAF v3.2.1 local Git tag | Both patches apply without fuzz |
| OpenH264 v2.7.0 local Git tag | Both selected patches apply without fuzz, using the version-specific override |
| x264 installed commit `4613ac3c15fd75cebc4b9f65b7fb95e70a3acce1` | Patch applies without fuzz |
| x265 4.3 | Release sources unavailable locally; only audited 4.2 patch application checked |
| LAME 3.100 release archive | Unavailable locally; only audited Git snapshot patch application checked |

These are source-patch checks, not Gentoo builds or tests. Gentoo's own
ebuild patches, selected USE flags, newer versions, and different source
snapshots can still require rebasing. In particular, do not assume x265 4.3
is source-compatible merely because the package mapping is correct.
Future OpenH264 versions also need a fresh check: the 2.7.0 override is
intentionally not applied indiscriminately to future releases.

## Rebuild libraries before FFmpeg

This checkout is **not** the live `/etc/portage/patches` tree on the inspected
system: the directories have different identities and the new patches are
not yet visible there. Deploy through the existing patches-repository
workflow first, preserving any live user patches and copying the OpenH264
version-specific directory too. Merely running `emerge` before deployment
will not activate these fixes.

After deployment, the following commands deliberately finish the patched
libraries before rebuilding FFmpeg. VMAF is first because Gentoo x265 can
depend on it through `USE=vmaf`. Run the FFmpeg command only after both
library commands succeed and any version-specific patch failures are resolved.
The library phases explicitly exclude FFmpeg, preventing an incidental
dependency/rebuild selection from moving its rebuild ahead of the libraries.

```sh
emerge -1 --exclude media-video/ffmpeg media-libs/libvmaf &&
emerge -1 --exclude media-video/ffmpeg \
    media-libs/fdk-aac \
    media-libs/kvazaar \
    media-libs/libvorbis \
    media-libs/openh264 \
    media-libs/svt-av1 \
    media-libs/x264 \
    media-libs/x265 \
    media-sound/lame

# Separate, final step:
emerge -1 media-video/ffmpeg
```

No system patch tree was modified; no `emerge`, build, push, tag, or commit
was run during export.
