AVPROGS-$(CONFIG_FFMPEG)   += ffmpeg
AVPROGS-$(CONFIG_FFPLAY)   += ffplay
AVPROGS-$(CONFIG_FFPROBE)  += ffprobe

AVPROGS     := $(AVPROGS-yes:%=%$(PROGSSUF)$(EXESUF))
PROGS       += $(AVPROGS)

AVBASENAMES  = ffmpeg ffplay ffprobe
ALLAVPROGS   = $(AVBASENAMES:%=%$(PROGSSUF)$(EXESUF))
ALLAVPROGS_G = $(AVBASENAMES:%=%$(PROGSSUF)_g$(EXESUF))

OBJS-ffmpeg                        += fftools/ffmpeg_opt.o fftools/ffmpeg_filter.o fftools/ffmpeg_hw.o
OBJS-ffmpeg-$(CONFIG_CUVID)        += fftools/ffmpeg_cuvid.o
OBJS-ffmpeg-$(CONFIG_LIBMFX)       += fftools/ffmpeg_qsv.o
ifndef CONFIG_VIDEOTOOLBOX
OBJS-ffmpeg-$(CONFIG_VDA)          += fftools/ffmpeg_videotoolbox.o
endif
OBJS-ffmpeg-$(CONFIG_VIDEOTOOLBOX) += fftools/ffmpeg_videotoolbox.o

define DOFFTOOL
OBJS-$(1) += fftools/cmdutils.o fftools/$(1).o $(OBJS-$(1)-yes)
$(1)$(PROGSSUF)_g$(EXESUF): $$(OBJS-$(1))
$$(OBJS-$(1)): | fftools
$$(OBJS-$(1)): CFLAGS  += $(CFLAGS-$(1))
$(1)$(PROGSSUF)_g$(EXESUF): LDFLAGS += $(LDFLAGS-$(1))
$(1)$(PROGSSUF)_g$(EXESUF): FF_EXTRALIBS += $(EXTRALIBS-$(1))
-include $$(OBJS-$(1):.o=.d)
endef

OBJS-fftools                       += fftools/cmdutils.o fftools/ffmpeg_filter.o fftools/ffmpeg_hw.o fftools/ffmpeg_lib.o fftools/ffmpeg_opt.o fftools/ffprobe_lib.o
ifdef CONFIG_VIDEOTOOLBOX
OBJS-fftools                       += fftools/ffmpeg_videotoolbox.o
endif

$(foreach P,$(AVPROGS-yes),$(eval $(call DOFFTOOL,$(P))))

libfftools: fftools/ffmpeg_lib.o fftools/ffprobe_lib.o
	ar rcs fftools/libfftools.a $(OBJS-fftools)

all: $(AVPROGS) libfftools

fftools/ffprobe.o fftools/cmdutils.o: libavutil/ffversion.h | fftools
OUTDIRS += fftools

ifdef AVPROGS
install: install-progs install-data
endif

install-progs-yes:
install-progs-$(CONFIG_SHARED): install-libs

install-progs: install-progs-yes $(AVPROGS)
	$(Q)mkdir -p "$(BINDIR)"
	$(INSTALL) -c -m 755 $(AVPROGS) "$(BINDIR)"
	$(Q)mkdir -p "$(LIBDIR)"
	$(INSTALL) -c -m 755 fftools/libfftools.a "$(LIBDIR)"
	$(Q)mkdir -p "$(INCDIR)"
	$(INSTALL) -c -m 755 config.h "$(INCDIR)"
	$(Q)mkdir -p "$(INCDIR)"/libfftools
	$(INSTALL) -c -m 755 fftools/cmdutils.h fftools/ffmpeg.h "$(INCDIR)"/libfftools
	$(Q)mkdir -p "$(INCDIR)"/libavutil
	$(INSTALL) -c -m 755 libavutil/internal.h "$(INCDIR)"/libavutil/
	$(INSTALL) -c -m 755 libavutil/timer.h "$(INCDIR)"/libavutil/
	$(Q)mkdir -p "$(INCDIR)"/libavutil/x86
	$(INSTALL) -c -m 755 libavutil/x86/timer.h "$(INCDIR)"/libavutil/x86/
	$(INSTALL) -c -m 755 libavutil/x86/emms.h "$(INCDIR)"/libavutil/x86/
	$(INSTALL) -c -m 755 libavutil/libm.h "$(INCDIR)"/libavutil/
	$(Q)mkdir -p "$(INCDIR)"/compat
	$(INSTALL) -c -m 755 compat/w32pthreads.h "$(INCDIR)"/compat/

uninstall: uninstall-progs

uninstall-progs:
	$(RM) $(addprefix "$(BINDIR)/", $(ALLAVPROGS))

clean::
	$(RM) $(ALLAVPROGS) $(ALLAVPROGS_G) $(CLEANSUFFIXES:%=fftools/%)
