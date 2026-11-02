# Multimedia audit patches

Exported on 2026-11-02 from the Multimedia audit's
`doc/logs/reproduce/patches.tsv` and `doc/PR-DRAFTS.md`, plus the two
Matroska seek patches created earlier in the same audit.

There are **66 distinct applicable fixes for ten packages**: all 64 fixes in
the upstream-submission manifest and the Matroska fix/regression-test pair.
The package directories contain 77 patch files, including one OpenH264 2.7.0
replacement and eight x265 4.3 overrides (seven intentional empty masks).
Two more intentional empty masks handle Gentoo libvorbis 1.3.7-r2 overlap.
Nine additional patch files below this
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
* **x265 4.3:** seven fixes are already present upstream in the release,
  before Gentoo's own patches. Applying the older-base patches again fails.
  The same-basename empty files in `media-libs/x265-4.3/` intentionally mask
  `0003` through `0009` using Portage's documented empty-file override
  mechanism. They are required files, not accidentally empty patches.
  The `0010` override retains both RPU ownership changes; it omits only
  an unrelated trailing-blank-line deletion already inapplicable to 4.3.
  All generic patches and archived originals remain unchanged.

  | Masked patch | Equivalent correction already in x265 4.3 |
  |---|---|
  | `0003` NAL buffer | `if (m_occupancy)` guards the copy |
  | `0004` bitstream | `topword < 32` guards the shift |
  | `0005` lookahead | temporal estimator reference is acquired inside the enabled-filter branch |
  | `0006` signed vectors | shifts operate on `uint32_t`, not negative signed operands |
  | `0007` rate control | both residual divisions use nonzero-denominator guards |
  | `0008` coefficient groups | two successive shifts, by `cgBlkPos` then 1, avoid shifting by 64 |
  | `0009` diamond directions | unsigned shifts precede the signed directional decode |
* **Vorbis 1.3.7-r2:** Gentoo already applies upstream `psy-bounds.patch`
  (commit `315da9cc9d30484c802b2e2ea150df39e060e2b9`) and
  `ubsan-shift.patch` (commit `bb4047de4c05712bf1fd49b9584c360b8e4e0adf`).
  They supply the same interpolation-index clamp and unsigned search-hint
  shift as audit patches `0004` and `0007`. Same-basename zero-byte masks
  in `media-libs/libvorbis-1.3.7-r2/` suppress only those redundant patches
  for that exact Gentoo revision. The interpolation condition uses
  `>=P_BANDS-2` rather than `>=P_BANDS-1`, which has the same effective
  integer clamp: assigning `P_BANDS-2` when already equal changes nothing.
  The remaining `0003`, `0005`, and `0006` apply unchanged, retaining all
  five meaningful corrections. Generic originals remain untouched.
  Full ordered application and checksum evidence is in
  `libvorbis-1.3.7-r2-application.log`.
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
| Gentoo Vorbis 1.3.7-r2 | All four official patches, then the three unmasked audit patches apply without fuzz; both masked fixes verified already present |
| SVT-AV1 v4.1.0 local Git tag | All 10 patches apply without fuzz |
| VMAF v3.2.1 local Git tag | Both patches apply without fuzz |
| OpenH264 v2.7.0 local Git tag | Both selected patches apply without fuzz, using the version-specific override |
| x264 installed commit `4613ac3c15fd75cebc4b9f65b7fb95e70a3acce1` | Patch applies without fuzz |
| x265 4.3 | Cached release plus all nine official Gentoo patches checked; seven upstream-equivalent masks and the adapted RPU patch select/apply correctly without fuzz |
| LAME 3.100 release archive | Unavailable locally; only audited Git snapshot patch application checked |

These are source-patch checks, not Gentoo builds or tests. Gentoo's own
ebuild patches, selected USE flags, newer versions, and different source
snapshots can still require rebasing. x265 4.3 now has a version-specific
selection verified after Gentoo's own patches; future releases need another check.
Future OpenH264 versions also need a fresh check: the 2.7.0 override is
intentionally not applied indiscriminately to future releases.

## Rebuild libraries before FFmpeg

This checkout is **not** the live `/etc/portage/patches` tree on the inspected
system: the directories have different identities and the new patches are
not yet visible there at initial export. Continue using the existing
patches-repository workflow; no deployment tool is required. The user has
already deployed through that mechanism. Deploy subsequent corrections the
same way, preserving live user patches and including both the OpenH264 and
x265 and Vorbis version-specific directories. In particular, retain all zero-byte
mask files. Merely rerunning `emerge` before those corrections reach the
live tree will still use the old failing selection.

The 66 fixes comprise the manifest's 64 plus
`0007-avformat-matroska-reset-state-after-generic-or-byte-.patch` and
`0008-tests-api-cover-Matroska-byte-seek-after-EOF.patch`.
Do not activate the archived Meson-only
`doc/multimedia-audit/non-applicable/0006-meson-vaapi-check-name.patch`
or the eight archived x265 originals.

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
