---
title: 我的Makefile文件
Status: public
url: my_makefile
date: 2014-11-23
---

最近学习了《GNU Make项目管理》，改进了我之前一直在用的Makefile文件，解决我之前的Makefile中一直存在的修改依赖头文件后不能自动编译cpp文件的问题。本文列举了我常用的两个Makefile文件，其中第一个为我常用的Makefile，第二个为从网上找到的其他Makefile文件。

# 第一个Makefile

```makefile
all:
INCLUDE = -I./

FLAGS = -g -Wall $(INCLUDE)
FLAGS += -fPIC

LIBDIR = -lz -lm -lcrypto

LINK = $(LIBDIR) -lpthread

GCC = g++

# for C++ language
CODE.cpp = main.cpp \
			trim.cpp

CPP.o = $(CODE.cpp:.cpp=.o)
OBJS.d = $(CODE.cpp:.cpp=.d)

OBJS.o = $(CPP.o)

# 解决头文件依赖
-include $(subst .cpp,.d,$(CODE.cpp))

%.d: %.cpp
	$(GCC) -M $(FLAGS) $< > $@.$$$$;		\
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@;	\
	rm -f $@.$$$$

# rule for C++ language
%.o : %.cpp	
	$(GCC) $(FLAGS) -o $@ -c $<	
	@echo $*.o build successfully!......

TARGET = main
	
$(TARGET) : $(OBJS.o) 
	$(GCC) $(OBJS.o) -o $(TARGET) $(LINK)
	@echo $(TARGET) BUILD OK!.........

all : $(TARGET)

.PHONY:
clean:
	rm -rf $(TARGET)
	rm -rf $(OBJS.o)
	rm -rf $(OBJS.d)
	rm -rf *.d
```

该文件特点为需要手工将需要编译的源文件手动添加到Makefile中，可能比较麻烦，但是编译时比较灵活。可以随意修改需要编译源文件的顺序和是否需要编译源文件。

# 第二个Makefile

```makefile
###########################################################
#
# KEFILE FOR C/C++ PROJECT
# Author: swm8023 <swm8023@gmail.com>
# Date:   2014/01/30
#
###########################################################

.PHONY: all clean
all: 

# annotation when release version
DEBUG       := y
TARGET_PROG := main

# project directory	
DEBUG_DIR   := ./Debug
RELEASE_DIR := ./Release
BIN_DIR     := $(if $(DEBUG), $(DEBUG_DIR), $(RELEASE_DIR))

# shell command
CC    := gcc
CXX   := g++
RM    := rm -rf
MKDIR := mkdir -p
SED   := sed
MV    := mv

# init sources & objects & depends
sources_all := $(shell find . -name "*.c" -o -name "*.cpp" -o -name "*.h")
sources_c   := $(filter %.c, $(sources_all))
sources_cpp := $(filter %.cpp, $(sources_all))
sources_h   := $(filter %.h, $(sources_all))
objs        := $(addprefix $(BIN_DIR)/,$(strip $(sources_cpp:.cpp=.o) $(sources_c:.c=.o)))
deps        := $(addprefix $(BIN_DIR)/,$(strip $(sources_cpp:.cpp=.d) $(sources_c:.c=.d)))

# create directory
$(foreach dirname,$(sort $(dir $(sources_c) $(sources_cpp))),\
  $(shell $(MKDIR) $(BIN_DIR)/$(dirname)))

# complie & link variable
CFLAGS     := $(if $(DEBUG),-g -O, -O2)
CFLAGS     += $(addprefix -I ,$(sort $(dir $(sources_h))))
CXXFLAGS    = $(CFLAGS)
LDFLAGS    := 
LOADLIBES  += #-L/usr/include/mysql
LDLIBS     += #-lpthread -lmysqlclient

# add vpath
vpath %.h $(sort $(dir $(sources_h)))
vpath %.c $(sort $(dir $(sources_c)))
vpath %.cpp $(sort $(dir $(sources_cpp)))

# generate depend files
# actually generate after object generated, beacasue it only used when next make)
ifneq "$(MAKECMDGOALS)" "clean"
sinclude $(deps)
endif

# make-depend(depend-file,source-file,object-file,cc)
define make-depend
  $(RM) $1;                                     \
  $4 $(CFLAGS) -MM $2 |                         \
  $(SED) 's,\($(notdir $3)\): ,$3: ,' > $1.tmp; \
  $(SED) -e 's/#.*//'                           \
         -e 's/^[^:]*: *//'                     \
         -e 's/ *\\$$//'                        \
         -e '/^$$/ d'                           \
         -e 's/$$/ :/' < $1.tmp >> $1.tmp;      \
  $(MV) $1.tmp $1;
endef

# rules to generate objects file
$(BIN_DIR)/%.o: %.c
	@$(call make-depend,$(patsubst %.o,%.d,$@),$<,$@,$(CC))
	$(CC) $(CFLAGS) -o $@ -c $<

$(BIN_DIR)/%.o: %.cpp
	@$(call make-depend,$(patsubst %.o,%.d,$@),$<,$@,$(CXX))
	$(CXX) $(CXXFLAGS) -o $@ -c $<

# add-target(target,objs,cc)
define add-target
  REAL_TARGET += $(BIN_DIR)/$1
  $(BIN_DIR)/$1: $2
	$3 $(LDFLAGS) $$^ $(LOADLIBES) $(LDLIBS) -o $$@
endef

# call add-target
$(foreach targ,$(TARGET_PROG),$(eval $(call add-target,$(targ),$(objs),$(CXX))))

all: $(REAL_TARGET) $(TARGET_LIBS)

clean: 
	$(RM) $(BIN_DIR)
```

该Makefile为从[一个通用的C/C++ Makefile](http://c4fun.cn/blog/2014/01/30/common-makefile/)中直接获得的，为了避免原博客以后不能访问的情况，这里备份一下。

该Makefile可以动检测Makefile所在目录及其子目录中的.c和.cpp文件，并进行编译，不需要手动修改Makefile来填写需要编译的源文件，比较自动化。

# 相关参考

第二个Makefile文件的作者博客中的两篇文章：[GNU Make学习总结（一）](http://c4fun.cn/blog/2014/01/13/gnu-make-study01/)和[GNU Make学习总结（二）](http://c4fun.cn/blog/2014/01/13/gnu-make-study02/)

# 相关下载

[一个包含上述两个Makefile的例子](http://pan.baidu.com/s/1jGDo5ls)
