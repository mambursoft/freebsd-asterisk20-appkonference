# Порт: appkonference (app_konference.so) для Asterisk 20 LTS
# Mambursoft · admin@voip.net.ua
#
# Високопродуктивний плагін голосових/відео конференцій для Asterisk.
# Збирається проти ВСТАНОВЛЕНОГО Asterisk 20 (${LOCALBASE}/include/asterisk).
# Джерело: pjalbrecht/appkonference (GitHub), пінований коміт заради
# відтворюваності збірки на різних хостах.
#
# Історія: походить від net/appkonference 2.8 (SourceForge, для asterisk13).
# Перероблено під Asterisk 20: інше джерело (GitHub), ASTERISK_SRC_VERSION=2000,
# увімкнено ULAW+ALAW (alaw-транки України), -fcommon (GCC 10+/clang).

PORTNAME=	appkonference
DISTVERSIONPREFIX=	g
DISTVERSION=	2.9.20
PORTREVISION=	1
CATEGORIES=	net

MAINTAINER=	admin@voip.net.ua
COMMENT=	High-performance Asterisk 20 voice/video conferencing plugin
WWW=		https://github.com/pjalbrecht/appkonference

LICENSE=	GPLv2
LICENSE_FILE=	${WRKSRC}/LICENSE

ONLY_FOR_ARCHS=		amd64 i386 armv6 armv7 aarch64 powerpc64 powerpc64le
ONLY_FOR_ARCHS_REASON=	uses libwebrtc typedefs (not tested on other archs)

# Збірка проти вихідників/заголовків ВСТАНОВЛЕНОГО Asterisk 20.
# Якщо dialplan використовує канали SIP/ (legacy chan_sip), net/asterisk20
# треба зібрати з опцією SIP — в офіційному порту вона ВИМКНЕНА за
# замовчуванням (див. pkg-message). Опції залежностей ports не перевіряє.
BUILD_DEPENDS=	asterisk:net/asterisk20
RUN_DEPENDS=	asterisk:net/asterisk20

USES=		compiler:features gmake
USE_CSTD=	gnu89

# Джерело — GitHub, пінований коміт HEAD master (2022-03-31, "Video mode for
# conference leave"). Теги у репозиторії відсутні, тож фіксуємо SHA.
USE_GITHUB=	yes
GH_ACCOUNT=	pjalbrecht
GH_PROJECT=	appkonference
GH_TAGNAME=	519a70d9303c3fcd3645719e2895a8e0b8cdefca

# Makefile та вихідники — у КОРЕНІ репозиторію (не в підкаталозі konference/,
# як було у тарболі SourceForge 2.8). Тому WRKSRC_SUBDIR не задаємо.

# Asterisk 20 → версія 2000 (Makefile appkonference кодує як %01d%02d).
MAKE_ARGS=	ASTERISK_SRC_VERSION=2000 \
		ASTERISK_SRC_DIR=${LOCALBASE} \
		INSTALL_MODULES_DIR=${PREFIX}/lib/asterisk/modules

# Детекція мовлення (libspeex, SILDET=2) у upstream увімкнена завжди, але
# куди йде результат, визначає SPEAKER_SCOREBOARD:
#   вимкнено (типово в порту) — AMI-подія ConferenceState (speaking/silent),
#     її слухають веб-інтерфейси через AMI/rawman («хто говорить»);
#   увімкнено (опція SCOREBOARD) — лише mmap-файл scoreboard, подій немає.
# Детекція працює для учасників з прапорцем 'a' (або 'T'/'V') у Konference().

# -fcommon: без нього GCC 10+/сучасний clang падають на "multiple definition"
# (глобалі конференції визначені в кількох .o). Паритет з Docker-патчем 0001.
CFLAGS+=	-fcommon

PLIST_FILES=	lib/asterisk/modules/app_konference.so

# Правки upstream Makefile через REINPLACE (а не files/patch-*): стійко до
# дрібних змін контексту. ?= дає змогу передати значення з MAKE_ARGS, += не
# затирає CFLAGS/CPPFLAGS із ports; вмикаємо KQUEUE (таймінг *BSD) і ALAW;
# прибираємо жорсткі -O3/-funroll (керуються опцією OPTIMIZED_CFLAGS).
post-patch:
	@${REINPLACE_CMD} \
		-e 's|^ASTERISK_SRC_DIR =|ASTERISK_SRC_DIR ?=|' \
		-e 's|^ASTERISK_SRC_VERSION = .*|ASTERISK_SRC_VERSION ?= 2000|' \
		-e 's|^INSTALL_MODULES_DIR = .*|INSTALL_MODULES_DIR ?= /usr/lib/asterisk/modules|' \
		-e 's|^REVISION = .*|REVISION = ${DISTVERSION}-g519a70d|' \
		-e 's|^CFLAGS = -pipe|CFLAGS += -pipe|' \
		-e 's|^CFLAGS += -O3 -ffast-math -funroll-all-loops -fsingle-precision-constant|#&|' \
		-e 's|^CPPFLAGS = \$$(INCLUDE)|CPPFLAGS += $$(INCLUDE)|' \
		-e 's|^# CPPFLAGS += -DKQUEUE|CPPFLAGS += -DKQUEUE|' \
		-e 's|^# CPPFLAGS += -DAC_USE_ALAW|CPPFLAGS += -DAC_USE_ALAW|' \
		-e 's|\$$(CC) \$$(SOLINK)|$$(CC) $$(LDFLAGS) $$(SOLINK)|' \
		${WRKSRC}/Makefile

# upstream install пише одразу в INSTALL_MODULES_DIR без DESTDIR -> власний do-install
do-install:
	${MKDIR} ${STAGEDIR}${PREFIX}/lib/asterisk/modules
	${INSTALL_LIB} ${WRKSRC}/app_konference.so ${STAGEDIR}${PREFIX}/lib/asterisk/modules/

OPTIONS_DEFINE=		OPTIMIZED_CFLAGS SCOREBOARD
OPTIONS_DEFAULT=	OPTIMIZED_CFLAGS
OPTIMIZED_CFLAGS_CFLAGS=	-O3 -ffast-math
SCOREBOARD_DESC=	Speaker state to mmap scoreboard instead of AMI ConferenceState events
SCOREBOARD_MAKE_ARGS=	SPEAKER_SCOREBOARD=1
SCOREBOARD_MAKE_ARGS_OFF=	SPEAKER_SCOREBOARD=0

.include <bsd.port.pre.mk>

.if ${PORT_OPTIONS:MOPTIMIZED_CFLAGS} && ${CHOSEN_COMPILER_TYPE} == gcc
CFLAGS+=	-funroll-all-loops -fsingle-precision-constant
.endif

.if ${CHOSEN_COMPILER_TYPE} == clang
CFLAGS+=	-fblocks
.endif

.include <bsd.port.post.mk>
