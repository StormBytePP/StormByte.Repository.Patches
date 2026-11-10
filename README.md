# StormByte.Repository.Patches
StormByte's custom patches to allow gentoo system to be built with clang

## mysql-connector-c 26.10.0 on musl

[Resolver compatibility patch](dev-db/mysql-connector-c-26.10.0/0001-detect-reentrant-resolver-for-musl.patch)
detects the complete `res_n*` API and uses the existing `res_search` fallback
when it is unavailable, including on Gentoo musl (not just Alpine).
Copy it into `/etc/portage/patches/dev-db/mysql-connector-c-26.10.0/`
and rebuild the package with `emerge -1av =dev-db/mysql-connector-c-26.10.0`.
