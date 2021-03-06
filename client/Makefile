#-----------------------------------------------------------------------------
# This code is licensed to you under the terms of the GNU GPL, version 2 or,
# at your option, any later version. See the LICENSE.txt file for the text of
# the license.
#-----------------------------------------------------------------------------
include ../common/Makefile.common


CC=gcc
CXX=g++
#COMMON_FLAGS = -m32
VPATH = ../common ../zlib
OBJDIR = obj

LDLIBS = -L/opt/local/lib -L/usr/local/lib -lreadline -lpthread -lm
LUALIB = ../liblua/liblua.a
LDFLAGS = $(COMMON_FLAGS)
CFLAGS = -std=c99 -D_ISOC99_SOURCE -I. -I../include -I../common -I../tools -I../zlib -I/opt/local/include -I../liblua -Wall $(COMMON_FLAGS) -g -O4
LUAPLATFORM = generic

ifneq (,$(findstring MINGW,$(platform)))
    CXXFLAGS = -I$(QTDIR)/include -I$(QTDIR)/include/QtCore -I$(QTDIR)/include/QtGui
    MOC = $(QTDIR)/bin/moc
    LUAPLATFORM = mingw
    ifneq ($(wildcard $(QTDIR)/include/QtWidgets),)
        CXXFLAGS += -I$(QTDIR)/include/QtWidgets
        QTLDLIBS = -L$(QTDIR)/lib -lQt5Core -lQt5Gui -lQt5Widgets
    else
        QTLDLIBS = -L$(QTDIR)/lib -lQtCore4 -lQtGui4
    endif
else ifeq ($(platform),Darwin)
    CXXFLAGS = $(shell pkg-config --cflags QtCore QtGui 2>/dev/null) -Wall -O4
    QTLDLIBS = $(shell pkg-config --libs QtCore QtGui 2>/dev/null)
    MOC = $(shell pkg-config --variable=moc_location QtCore)
    LUAPLATFORM = macosx
else
    CXXFLAGS = $(shell pkg-config --cflags QtCore QtGui 2>/dev/null) -Wall -O4
    QTLDLIBS = $(shell pkg-config --libs QtCore QtGui 2>/dev/null)
    LUALIB +=  -ldl
    LDLIBS +=  -ltermcap -lncurses
    MOC = $(shell pkg-config --variable=moc_location QtCore)
    # Below is a variant you can use if you have problems compiling with QT5 on ubuntu. see http://www.proxmark.org/forum/viewtopic.php?id=1661 for more info. 
    #MOC = /usr/lib/x86_64-linux-gnu/qt4/bin/moc
    LUAPLATFORM = linux
endif

ifneq ($(QTLDLIBS),)
    QTGUI = $(OBJDIR)/proxgui.o $(OBJDIR)/proxguiqt.o $(OBJDIR)/proxguiqt.moc.o
    CFLAGS += -DHAVE_GUI
    LINK.o = $(LINK.cpp)
else
    QTGUI = guidummy.o
endif

CORESRCS = 	uart.c \
		util.c \


CMDSRCS = 	crapto1/crapto1.c\
			crapto1/crypto1.c\
			mfkey.c\
			loclass/cipher.c \
			loclass/cipherutils.c \
			loclass/des.c \
			loclass/ikeys.c \
			loclass/elite_crack.c\
			loclass/fileutils.c\
			whereami.c\
			mifarehost.c\
			parity.c\
			crc.c \
			crc16.c \
			crc64.c \
			iso14443crc.c \
			iso15693tools.c \
			data.c \
			graph.c \
			ui.c \
			cmddata.c \
			lfdemod.c \
			cmdhf.c \
			cmdhf14a.c \
			cmdhf14b.c \
			cmdhf15.c \
			cmdhfepa.c \
			cmdhflegic.c \
			cmdhficlass.c \
			cmdhfmf.c \
			cmdhfmfu.c \
			cmdhftopaz.c \
			cmdhw.c \
			cmdlf.c \
			cmdlfawid.c \
			cmdlfcotag.c\
			cmdlfem4x.c \
			cmdlffdx.c \
			cmdlfgproxii.c \
			cmdlfhid.c \
			cmdlfhitag.c \
			cmdlfio.c \
			cmdlfindala.c \
			cmdlfpcf7931.c\
			cmdlfpresco.c\
			cmdlfpyramid.c\
			cmdlft55xx.c \
			cmdlfti.c \
			cmdlfviking.c\
			cmdlfvisa2000.c\
			cmdparser.c \
			cmdmain.c \
			scripting.c\
			cmdscript.c\
			pm3_binlib.c\
			pm3_bitlib.c\
			aes.c\
			protocols.c\
			sha1.c\
			cmdcrc.c\
			reveng/reveng.c\
			reveng/cli.c\
			reveng/bmpbit.c\
			reveng/model.c\
			reveng/poly.c\
			reveng/getopt.c\

ZLIBSRCS = deflate.c adler32.c trees.c zutil.c inflate.c inffast.c inftrees.c
ZLIB_FLAGS = -DZ_SOLO -DZ_PREFIX -DNO_GZIP -DZLIB_PM3_TUNED 
#-DDEBUG -Dverbose=1


COREOBJS = $(CORESRCS:%.c=$(OBJDIR)/%.o)
CMDOBJS = $(CMDSRCS:%.c=$(OBJDIR)/%.o)
ZLIBOBJS = $(ZLIBSRCS:%.c=$(OBJDIR)/%.o)

RM = rm -f
BINS = proxmark3 flasher fpga_compress #snooper cli
CLEAN = cli cli.exe flasher flasher.exe proxmark3 proxmark3.exe fpga_compress fpga_compress.exe snooper snooper.exe $(CMDOBJS) $(OBJDIR)/*.o *.o *.moc.cpp

all: lua_build $(BINS) 

all-static: LDLIBS:=-static $(LDLIBS)
all-static: snooper cli flasher fpga_compress

proxmark3: LDLIBS+=$(LUALIB) $(QTLDLIBS)
proxmark3: $(OBJDIR)/proxmark3.o $(COREOBJS) $(CMDOBJS) $(QTGUI)
	$(CXX) $(CXXFLAGS) $^ $(LDLIBS) -o $@

snooper: $(OBJDIR)/snooper.o $(COREOBJS) $(CMDOBJS) $(OBJDIR)/guidummy.o
	$(CXX) $(CXXFLAGS) $^ $(LDLIBS) -o $@

cli: $(OBJDIR)/cli.o $(COREOBJS) $(CMDOBJS) $(OBJDIR)/guidummy.o
	$(CXX) $(CXXFLAGS) $^ $(LDLIBS) -o $@

flasher: $(OBJDIR)/flash.o $(OBJDIR)/flasher.o $(COREOBJS)
	$(CXX) $(CXXFLAGS) $^ $(LDLIBS) -o $@

fpga_compress: $(OBJDIR)/fpga_compress.o $(ZLIBOBJS)
	$(CXX) $(CXXFLAGS) $(ZLIB_FLAGS) $^ $(LDLIBS) -o $@

$(OBJDIR)/%.o: %.c
	$(CC) $(CFLAGS) $(ZLIB_FLAGS) -c -o $@ $<

$(OBJDIR)/%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c -o $@ $<

proxguiqt.moc.cpp: proxguiqt.h
	$(MOC) -o$@ $^

clean:
	$(RM) $(CLEAN)
	cd ../liblua && make clean

tarbin: $(BINS)
	$(TAR) $(TARFLAGS) ../proxmark3-$(platform)-bin.tar $(BINS:%=client/%)

lua_build:
	@echo Compiling liblua, using platform $(LUAPLATFORM)
	cd ../liblua && make $(LUAPLATFORM)

.PHONY: all clean
