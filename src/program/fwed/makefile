#*********************************************************************************
#      Copyright:  (C) 2012 CoherentPlus Sdn. Bhd.
#                  All rights reserved.
#
#       Filename:  Makefile
#    Description:  This is the common top Makefile which used to call function to 
#                  Cross compile the subdir source code and link all the libs to 
#                  excutable binary file.
#    
#        Version:  1.0.0(10/08/2011~)
#         Author:  Guo Wenxue <guowenxue@gmail.com>
#      ChangeLog:  1, Release initial version on "10/08/2011 01:29:33 AM"
#                 
#********************************************************************************/

APP_BINARY_NAME = fwed
INST_PATH = /tftp
TOP_PATH = $(shell pwd)
LIBS_PATH=${TOP_PATH}/.libs
COMPILE_DATE=$(shell date  +"%Y-%m-%d %H:%M:%S %Z")

#Excutable binary file runs on platform
ARCH?=arm920t
#ARCH?=i386

#Static linked or Dynamic link
LINK_MODE=STATIC
MODE=PRODUCTION
DEBUG=1

CFLAGS+=-Werror -Wall
CFLAGS+=-DLINUX -DIS_LITTLE_ENDIAN -D_GNU_SOURCE -D_REENTRANT -D_LARGE_THREADS -D_THREAD_SAFE
CFLAGS+=-DAPP_BINARY_NAME="${APP_BINARY_NAME}"

#Compile Modules
CFLAGS+=-DSWE_GSMD
CFLAGS+=-DSWE_ROUTED

CFLAGS+=-DCONFIG_PPP

ifeq ("${MODE}", "PRODUCTION")
	CFLAGS+=-DPRODUCTION_MODE
endif
ifdef DEBUG
	CFLAGS+=-g -DDEBUG
endif

ifeq ("${LINK_MODE}", "STATIC")
	CFLAGS+=--static
	LDFLAGS+=-static
else
	LDFLAGS+=-ldl
endif

# ==================================
# **   Cross Compiler Setting     **
# ==================================

TMP=$(shell echo $(ARCH) | tr "[A-Z]" "[a-z]")
ifneq (,$(filter i386,$(TMP)))
	CROSS_COMPILE=
else
	CROSS_COMPILE=/opt/buildroot-2012.08/$(ARCH)/usr/bin/arm-linux-
endif
TOP_COMPILE=YES


# Get alll the source code folder, all the source code in the subdir will be compiled and
# archive to a static library, named lib${subdir_name}.a
OBJFILES = $(patsubst %.c,%,$(wildcard *.c))
SUBSRCS = $(shell find . -follow  -maxdepth 1 -type d|sed -n 's/.\///p'|grep -v 'libs' \
		  |grep -v 'etc' | grep -v '.svn'|grep -v '.git'|grep -v 'include'|grep -v 'bin')

# Add the subdir compiled static library into LDFLAGS
#LDFLAGS+=$(patsubst %,-L%,$(SUBSRCS)) 
#LDFLAGS+=$(patsubst %,-l%,$(SUBSRCS)) 

LDFLAGS+=-L${LIBS_PATH}
LDFLAGS+=$(patsubst %,-l%,$(SUBSRCS)) 
LIBS=$(patsubst %,${LIBS_PATH}/lib%.a,$(SUBSRCS))
LDFLAGS+=-Xlinker "-(" $(LIBS) -Xlinker "-)"

CFLAGS+=$(patsubst %,-I${TOP_PATH}/%,$(SUBSRCS))
CFLAGS+=-I$(TOP_PATH) 
LDFLAGS+=-lpthread

export CC=${CROSS_COMPILE}gcc
export CXX=${CROSS_COMPILE}gcc
export AR=${CROSS_COMPILE}ar
export AS=${CROSS_COMPILE}as
export LD=${CROSS_COMPILE}ld
export RANLIB=${CROSS_COMPILE}ranlib
export STRIP=${CROSS_COMPILE}strip
export CFLAGS
export LDFLAGS
export ARCH
export LINK_MODE
export LIBS_PATH

#A function used to goes into the sub-folder to compile
MAKEFUNC = @MakeSubDir() \
    { \
        for DIR in ${SUBSRCS}; do \
            if [ -d $${DIR} ] ; then \
                cd $${DIR}; \
                MakeSubDir; \
                if [ -f makefile -o -f Makefile ]; then \
                    pwd; \
                    make $(1); \
                    if [ "$$?" != "0" ]; then \
                        exit 1; \
                    fi; \
                fi; \
                cd ..; \
            fi; \
        done; \
        if [ -f makefile -o -f Makefile ]; then \
            make $(1); \
            if [ "$$?" != "0" ]; then \
                exit 1; \
            fi; \
        fi; \
    }; \

MAKEME = cd $(2); MakeSubDir $(1); cd ..;
LOOPMAKEFUNC = $(MAKEFUNC) $(foreach dir,$(SUBSRCS),$(call MAKEME,$(1),$(dir)))

.PHONY: all

all: entry version prepare binary install

prepare:
	@if [ ! -L cp_library  ] ; then \
		ln -s ../cp_library; \
	fi

entry:
	@echo $(SUBSRCS)
	@echo " ";
	@echo " =========================================================";
	@echo " ** Come into compile ${APP_BINARY_NAME} for ARCH $(ARCH) ";
	@echo " =========================================================";
	@echo " ";
	@make clean

version:
	@echo "/* Generated by makefile, don't Edit it by hand */" > version.h
	@echo '#define DATE "$(COMPILE_DATE)"' >> version.h
	@echo "#define MAJOR 1" >>version.h
	@echo "#define MINOR 0" >>version.h
	@echo "#define REVER 0" >>version.h
	@if [ -f .svn/entries ] ; then \
		echo "#define SVNVER `sed -n -e 4p .svn/entries`" >>version.h; \
	else \
		echo "#define SVNVER 0" >>version.h; \
	fi;
	@echo "" >> version.h
	@echo '#define version(progname) printf("%s Version %d.%d.%d Build @%05d (%s)\n", progname, MAJOR, MINOR, REVER,SVNVER, DATE)' >> version.h
	@echo '#define copyright() printf("Copyright (C) 2012 CoherentPlus Sdn. Bhd.\n")' >>version.h
	@echo '#define banner(progname) {version(progname); copyright(); printf("\n");}' >>version.h
	@echo "" >> version.h

subdir:
	@$(call LOOPMAKEFUNC,all)
	@echo " ";
	@echo " =========================================================";
	@echo " ** Compile and linking ${APP_BINARY_NAME} for ${ARCH} now";
	@echo " =========================================================";

%: %.c
	$(CC) $< $(CFLAGS) -c -o $@.o

binary: subdir $(OBJFILES)
	$(CC) $(CFLAGS) -o $(APP_BINARY_NAME) *.o $(LDFLAGS) 
	$(STRIP) $(APP_BINARY_NAME)
	
tag:
	@cscope -Rbq
	@ctags --c-kinds=+defglmnstuvx --langmap=c:.c.h.ho.hem.het.hec.hev.him.hit.hic.hiv -R .

install:
	cp $(APP_BINARY_NAME) ${INST_PATH}

uninstall:
	rm -f ${INST_PATH}/$(APP_BINARY_NAME)

clean:
	@$(call LOOPMAKEFUNC,clean)
	@rm -rf ${LIBS_PATH}
	@rm -f version.h
	@rm -f *.o $(APP_BINARY_NAME)
	@rm -f *.elf*
	@rm -f *.gdb
	@rm -f cp_library

distclean: clean
	@$(call LOOPMAKEFUNC,distclean)

clear: distclean
	@rm -f tags  cscope*

