# net/asterisk20-appkonference — порт FreeBSD

Високопродуктивний модуль конференцій `app_konference.so`, незалежний від
типу каналу, для **Asterisk 20 LTS** у вигляді порту FreeBSD.

Супровід: **Mambursoft** (admin@voip.net.ua).
Upstream модуля: https://github.com/pjalbrecht/appkonference

Старий порт `net/appkonference` видалено з дерева ports 2021-12-31 (залежав
від EOL `net/asterisk13`). Цей порт повертає appkonference для сучасної
FreeBSD і Asterisk 20.

## Що це (і чим НЕ є)

- **Це** лише модуль конференцій `app_konference.so`, що збирається проти вже
  встановленого офіційного порту `net/asterisk20`.
- **Це НЕ** копія чи форк Asterisk. Сам Asterisk береться з офіційного порту
  `net/asterisk20`.

Перевірено: FreeBSD 15.1-RELEASE amd64, `asterisk20-20.22.0`, clang.

## Джерело

- Upstream: `pjalbrecht/appkonference` (GitHub).
- Пінований коміт `519a70d9303c3fcd3645719e2895a8e0b8cdefca` (2022-03-31).
  Тегів релізів у репозиторії немає, тож SHA фіксується заради відтворюваної
  збірки.
- Вихідний код модуля НЕ змінюється. Порт лише адаптує upstream `Makefile`
  через `post-patch` і має власний `do-install`.

## Що змінює post-patch

- `ASTERISK_SRC_DIR ?=`, `ASTERISK_SRC_VERSION ?= 2000`,
  `INSTALL_MODULES_DIR ?=` — значення приходять з `MAKE_ARGS` порту;
- `CFLAGS +=` / `CPPFLAGS +=` — прапорці з ports не затираються;
- вмикає `-DKQUEUE` (таймінг *BSD) і `-DAC_USE_ALAW` (ULAW + ALAW);
- жорсткі `-O3 -ffast-math -funroll-all-loops ...` керуються опцією
  `OPTIMIZED_CFLAGS` (типово увімкнена);
- лінкування з `$(LDFLAGS)`.

Також `-fcommon` (GCC 10+/clang) і `USE_CSTD=gnu89`.

Типово модуль збирається з `SPEAKER_SCOREBOARD=0`: детекція мовлення
(libspeex) повідомляє стан учасника AMI-подією `ConferenceState`
(`speaking`/`silent`), тож веб-панель може показувати, хто говорить. Upstream
за замовчуванням пише стан лише в mmap-файл scoreboard; опція порту
`SCOREBOARD` повертає таку поведінку. Детекція працює для учасників з
прапорцем `a` (або `T`/`V`) у `Konference()`. `do-install` кладе
`app_konference.so` у stage-каталог, бо upstream-ціль `install` ігнорує
`DESTDIR`.

## chan_sip

Якщо dialplan використовує канали `SIP/` (застарілий `chan_sip`), зважте: в
офіційному `net/asterisk20` це опція **SIP**, і вона **вимкнена за
замовчуванням** (в Asterisk 21 chan_sip видалено). Ports не перевіряють опції
залежностей, тож Asterisk треба зібрати з нею самостійно.

WebRTC (браузерний софтфон) потребує `PJSIP` (res_pjsip, транспорт `wss`),
`SRTP` (DTLS-SRTP), `res_http_websocket` та OpenSSL (збираються типово).

## Встановлення

```sh
cp -R net/asterisk20-appkonference /usr/ports/net/
make -C /usr/ports/net/asterisk20-appkonference install clean
```

Відтворювана збірка пари (без діалогів):

```sh
make -C /usr/ports/net/asterisk20 OPTIONS_SET="SIP PGSQL PJSIP SRTP CURL" \
     OPTIONS_UNSET="MYSQL" config-recursive
make -C /usr/ports/net/asterisk20 reinstall clean
make -C /usr/ports/net/asterisk20-appkonference reinstall clean
```

## Після встановлення

У `/usr/local/etc/asterisk/modules.conf`:

```
load => app_konference.so
```

Перевірка:

```sh
asterisk -rx "module show like konference"
asterisk -rx "konference list"
```

## Ліцензія

Модуль appkonference — GPLv2 (файл `LICENSE` в upstream). Файли порту —
на тих самих умовах.
