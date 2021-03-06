
sbin_PROGRAMS = forked-daapd

if COND_ITUNES
ITUNES_SRC=library/filescanner_itunes.c
endif

if COND_SPOTIFY
SPOTIFY_SRC=spotify.c spotify.h spotify_webapi.c spotify_webapi.h inputs/spotify.c
endif

if COND_LASTFM
LASTFM_SRC=lastfm.c lastfm.h
endif

if COND_CHROMECAST
if COND_PROTOBUF_OLD
CHROMECAST_SRC=outputs/cast.c outputs/cast_channel.v0.pb-c.h outputs/cast_channel.v0.pb-c.c
else
CHROMECAST_SRC=outputs/cast.c outputs/cast_channel.pb-c.h outputs/cast_channel.pb-c.c
endif
endif

if COND_MPD
MPD_SRC=mpd.c mpd.h
endif

if COND_RAOP_VERIFICATION
RAOP_VERIFICATION_SRC=outputs/raop_verification.c outputs/raop_verification.h
endif

if COND_ALSA
ALSA_SRC=outputs/alsa.c
endif

if COND_PULSEAUDIO
PULSEAUDIO_SRC=outputs/pulse.c
endif

if COND_AVAHI
MDNS_SRC=mdns_avahi.c
else
MDNS_SRC=mdns_dnssd.c
endif

if COND_FFMPEG_LEGACY
FFMPEG_SRC=transcode_legacy.c artwork_legacy.c ffmpeg-compat.h
else
FFMPEG_SRC=transcode.c artwork.c
endif

GPERF_FILES = \
	daap_query.gperf \
	rsp_query.gperf \
	dacp_prop.gperf \
	dmap_fields.gperf

GPERF_SRC = $(GPERF_FILES:.gperf=_hash.h)

ANTLR_GRAMMARS = \
	RSP.g RSP2SQL.g \
	DAAP.g DAAP2SQL.g \
	SMARTPL.g SMARTPL2SQL.g 

ANTLR_TOKENS = $(ANTLR_GRAMMARS:.g=.tokens)

ANTLR_DEPS = $(ANTLR_GRAMMARS:%.g=$(srcdir)/%.u)

ANTLR_SRC = \
	RSPLexer.c RSPLexer.h RSPParser.c RSPParser.h \
	RSP2SQL.c RSP2SQL.h \
	DAAPLexer.c DAAPLexer.h DAAPParser.c DAAPParser.h \
	DAAP2SQL.c DAAP2SQL.h \
	SMARTPLLexer.c SMARTPLLexer.h SMARTPLParser.c SMARTPLParser.h  \
	SMARTPL2SQL.c SMARTPL2SQL.h

AM_CPPFLAGS += \
	$(FORKED_CPPFLAGS) \
	$(FORKED_OPTS_CPPFLAGS) \
	$(COMMON_CPPFLAGS) \
	\
	-D_GNU_SOURCE \
	-DDATADIR=\"$(pkgdatadir)\" \
	-DCONFDIR=\"$(sysconfdir)\" \
	-DSTATEDIR=\"$(localstatedir)\" \
	-DPKGLIBDIR=\"$(pkglibdir)\"

forked_daapd_LDADD = \
	$(FORKED_LIBS) \
	$(FORKED_OPTS_LIBS) \
	$(COMMON_LIBS)

forked_daapd_SOURCES = main.c \
	db.c db.h \
	db_init.c db_init.h \
	db_upgrade.c db_upgrade.h \
	logger.c logger.h \
	conffile.c conffile.h \
	cache.c cache.h \
	library/filescanner.c library/filescanner.h \
	library/filescanner_ffmpeg.c library/filescanner_playlist.c \
	library/filescanner_smartpl.c $(ITUNES_SRC) \
	library.c library.h \
	$(MDNS_SRC) mdns.h \
	remote_pairing.c remote_pairing.h \
	avio_evbuffer.c avio_evbuffer.h \
	httpd.c httpd.h \
	httpd_rsp.c httpd_rsp.h \
	httpd_daap.c httpd_daap.h \
	httpd_dacp.c httpd_dacp.h \
	httpd_streaming.c httpd_streaming.h \
	http.c http.h \
	dmap_common.c dmap_common.h \
	$(FFMPEG_SRC) \
	misc.c misc.h \
	rng.c rng.h \
	rsp_query.c rsp_query.h \
	daap_query.c daap_query.h \
	player.c player.h \
	worker.c worker.h \
	input.h input.c \
	inputs/file_http.c inputs/pipe.c \
	outputs.h outputs.c \
	outputs/raop.c $(RAOP_VERIFICATION_SRC) \
	outputs/streaming.c outputs/dummy.c outputs/fifo.c \
	$(ALSA_SRC) $(PULSEAUDIO_SRC) $(CHROMECAST_SRC) \
	evrtsp/rtsp.c evrtsp/evrtsp.h evrtsp/rtsp-internal.h evrtsp/log.h \
	$(SPOTIFY_SRC) \
	$(LASTFM_SRC) \
	$(MPD_SRC) \
	listener.c listener.h \
	commands.c commands.h \
	mxml-compat.h \
	$(GPERF_SRC) \
	$(ANTLR_SRC)

# built by maintainers, and distributed. Clean with maintainer-clean
BUILT_SOURCES = \
	$(GPERF_SRC) \
	$(ANTLR_SRC) \
	$(ANTLR_TOKENS) \
	$(ANTLR_DEPS)

EXTRA_DIST = \
	$(GPERF_FILES) \
	$(ANTLR_GRAMMARS) \
	$(ANTLR_TOKENS) \
	$(ANTLR_DEPS)

# gperf construction rules
%_hash.h: %.gperf
	$(AM_V_GEN)$(GPERF) --output-file=$@ $<

# silent rules for antlr
antlr_verbose = $(antlr_verbose_@AM_V@)
antlr_verbose_ = $(antlr_verbose_@AM_DEFAULT_V@)
antlr_verbose_0 = @echo "  GEN     " $< "products";

# ANTLR grammar products
%.tokens %.c %Lexer.c %Parser.c %Lexer.h %Parser.h %.h: %.g
	$(antlr_verbose)$(ANTLR) -Xconversiontimeout 30000 $(ANTLR_OPTIONS) -fo . $<

# ANTLR dependency files (bypass circular dependency of .g on .tokens)
%.u: %.g
	$(AM_V_GEN)$(ANTLR) -depend -fo . $< > $@
	$(AM_V_at)$(SED) -n -e '/^.*\.g[ ]*:\(.*\)/ { s//\1/;h;d; }' -e '/\.tokens.*:/ { p;d; }' -e '/:/ { G;s/\n/ /;p; }' $@ > $@-t
	$(AM_V_at)mv $@-t $@

-include $(ANTLR_DEPS)
