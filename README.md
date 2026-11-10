# freebsd-asterisk20-appkonference

FreeBSD port **`net/asterisk20-appkonference`**: the high-performance,
channel-independent conferencing module `app_konference.so`
([pjalbrecht/appkonference](https://github.com/pjalbrecht/appkonference))
built against the official **Asterisk 20 LTS** port (`net/asterisk20`).

The old `net/appkonference` port was removed from the FreeBSD ports tree on
2021-12-31 (it depended on the EOL `net/asterisk13`). This port brings
appkonference back for current FreeBSD and Asterisk 20.

Українською: [README.uk.md](README.uk.md).

Tested on FreeBSD 15.1-RELEASE amd64 with `asterisk20-20.22.0` (clang).

## Layout

```
net/asterisk20-appkonference/
    Makefile  distinfo  pkg-descr  pkg-message
```

## Quick start

```sh
cp -R net/asterisk20-appkonference /usr/ports/net/
make -C /usr/ports/net/asterisk20-appkonference install clean
```

Then in `/usr/local/etc/asterisk/modules.conf`:

```
load => app_konference.so
```

and check:

```sh
asterisk -rx "module show like konference"
asterisk -rx "konference list"
```

## What the port does

- Only the module `app_konference.so`; Asterisk itself comes from the
  official `net/asterisk20` port (no custom Asterisk port).
- Source: GitHub `pjalbrecht/appkonference`, pinned commit
  `519a70d9303c3fcd3645719e2895a8e0b8cdefca` (the repo has no release tags).
- **No changes to the module source code.** The upstream `Makefile` is adapted
  in `post-patch` (`REINPLACE_CMD`):
  - `ASTERISK_SRC_DIR ?=`, `ASTERISK_SRC_VERSION ?= 2000`,
    `INSTALL_MODULES_DIR ?=` (values come from the port's `MAKE_ARGS`);
  - `CFLAGS +=` / `CPPFLAGS +=` so ports flags are kept;
  - enables `-DKQUEUE` (BSD timing) and `-DAC_USE_ALAW` (ULAW + ALAW);
  - the hard-coded `-O3 -ffast-math -funroll-all-loops ...` are controlled by
    the `OPTIMIZED_CFLAGS` option (on by default);
  - links with `$(LDFLAGS)`.
- `-fcommon` for GCC 10+/clang, `USE_CSTD=gnu89`.
- Built with `SPEAKER_SCOREBOARD=0` by default, so voice activity detection
  (libspeex) reports speaker state as the AMI event `ConferenceState`
  (`speaking`/`silent`) — useful for "who is talking" in web panels. The
  upstream default writes it only to an mmap scoreboard file; the port option
  `SCOREBOARD` restores that. Detection works for members joined with the
  `a` (or `T`/`V`) flag in `Konference()`.
- Own `do-install` (the upstream `install` target ignores `DESTDIR`).

## chan_sip note

Many appkonference setups use the legacy `chan_sip`. In `net/asterisk20` it is
the **SIP** option and it is **off by default** (chan_sip was removed in
Asterisk 21). Build Asterisk with it if you need it:

```sh
make -C /usr/ports/net/asterisk20 OPTIONS_SET="SIP PGSQL PJSIP SRTP CURL" \
     OPTIONS_UNSET="MYSQL" config-recursive
make -C /usr/ports/net/asterisk20 reinstall clean
make -C /usr/ports/net/asterisk20-appkonference reinstall clean
```

## Maintainer

Mambursoft — admin@voip.net.ua

## License

appkonference is GPLv2 (see `LICENSE` upstream). The port files are provided
under the same terms.
