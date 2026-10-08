# OneDrive missing-name crash workaround for libmsgraph 0.2.1

A local patch for a JSON parsing crash affecting OneDrive access through GNOME Files on Ubuntu 24.04.

## Problem

Debugging `gvfsd-onedrive` with GDB traced the crash to
`msg_drive_item_new_from_json()` in `src/drive/msg-drive-item.c`.

In the response I investigated, an item associated with OneDrive’s
Personal Vault had a `remoteItem` object without a `name` field.
Reading the missing field triggered an assertion failure and crashed
the backend. GNOME Files then reported “connection closed.”

## Patch

The patch uses `remoteItem.name` when available and otherwise falls
back to the outer item’s `name`. If neither exists, it returns an
error instead of reading the missing field.

See [onedrive-name-fallback.patch](onedrive-name-fallback.patch).

## Build

From an unmodified libmsgraph 0.2.1 source directory:

```bash
patch -p1 < /path/to/onedrive-name-fallback.patch
meson setup build-local
ninja -C build-local
```

Replace `/path/to/onedrive-name-fallback.patch` with the actual
location of the downloaded patch.

The patched library is built under `build-local/src`. Use
`LD_LIBRARY_PATH` to load it without replacing the system library.
A user-level systemd override for `gvfs-daemon.service` can make
this setting persistent.

## Tested behavior

With the patched library, file browsing, opening files, uploading,
and editing/saving worked in my environment.
