---
layout: post
title:  "java"
date:   2017-09-04 16:47:53
categories: java
---
### set enverioment

export LANG=C
export ALT_BOOTDIR=/usr/lib/jvm/java-7-openjdk-amd64
export ALT_JDK_IMPORT_PATH=$ALT_BOOTDIR
export ALLOW_DOWNLOADS=true
export HOTSPOT_BUILD_JOBS=4
export ALT_PARALLEL_COMPILE_JOBS=4
export SKIP_COMPARE_IMAGES=true
export USE_PRECOMPILED_HEADER=true
#export BUILD_LANGTOOLS=true
export BUILD_JAXP=false
#export BUILD_JAXWS=false
export BUILD_CORBA=false
#export BUILD_HOTSPOT=true
#export BUILD_JDK=true
#export SKIP_DEBUG_BUILD=false
#export SKIP_FASTDEBUG_BUILD=true
#export DEBUG_NAME=debug
export DISABLE_HOTSPOT_OS_VERSION_CHECK=ok
export BUILD_DEPLOY=false
export BUILD_INSTALL=false
export ALT_OUTPUTDIR=/home/xxxx/jdk7_build
unset JAVA_HOME
unset CLASSPATH
unset LD_LIBRARY_PATH
make sanity


### openjdk7 hotspot build errors

1. 问题1  

>&2 echo "*** This OS is not supported:" `uname -a`; exit 1;
*** This OS is not supported: Linux tms2 4.4.0-53-generic #74~14.04.1-Ubuntu SMP Fri Dec 2 03:43:31 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
make[5]: *** [check_os_version] Error 1

--修复办法，导出如下变量

export DISABLE_HOTSPOT_OS_VERSION_CHECK=ok

2. 问题2 

/home/xxxx/jdk7/hotspot/src/share/vm/runtime/interfaceSupport.hpp:430:0: error: "__LEAF" redefined [-Werror]
 #define __LEAF(result_type, header) 

--修复办法，修改hotspot/src/share/vm/runtime/interfaceSupport.hpp:430
```c
#ifdef __LEAF
#undef __LEAF
#define __LEAF(result_type, header)                                  \
  TRACE_CALL(result_type, header)                                    \
  debug_only(NoHandleMark __hm;)                                     \
  /* begin of body */
#endif
```

3. 问题3

/home/xxxx/jdk7/hotspot/src/share/vm/oops/constantPoolOop.cpp:272:39: error: converting 'false' to pointer type 'methodOop' [-Werror=conversion-null]
   if (cpool->cache() == NULL)  return false;  // nothing to load yet

--修复办法，修改hotspot/src/share/vm/oops/constantPoolOop.cpp:272

```c
if (cpool->cache() == NULL)  return (methodOop)false;  // nothing to load yet
```

4. 问题4 

/home/xxxx/jdk7/hotspot/src/share/vm/opto/loopnode.cpp:896:49: error: converting 'false' to pointer type 'Node*' [-Werror=conversion-null]
   if (expr == NULL || expr->req() != 3)  return false;

--修复办法，修改hotspot/src/share/vm/opto/loopnode.cpp:896

```c
  // Quick cutouts:
  if (expr == NULL || expr->req() != 3)  return NULL;
```

5. 问题5 

cd linux_amd64_compiler2/product && ./test_gamma
java full version "1.7.0_121-b00"
Using java runtime at: /usr/lib/jvm/java-7-openjdk-amd64/jre
Error occurred during initialization of VM
Unable to load native library: /usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/libjava.so: symbol JVM_SetNativeThreadName, version SUNWprivate_1.1 not defined in file libjvm.so with link time reference

--修复办法， hotspot/make/linux/Makefile

```c
$(TARGETS_C2):  $(SUBDIRS_C2)
        cd $(OSNAME)_$(BUILDARCH)_compiler2/$@ && $(MAKE) $(MFLAGS)
        #cd $(OSNAME)_$(BUILDARCH)_compiler2/$@ && ./test_gamma
        cd $(OSNAME)_$(BUILDARCH)_compiler2/$@
```

### openjdk7 jdk build errors

#### 问题1

gcc: error: unrecognized command line option '-mimpure-text'
make[4]: *** [/home/xxxx/jdk7_build/lib/amd64/libverify.so] Error 1
make[4]: Leaving directory `/home/xxxx/jdk7/jdk/make/java/verify'


--修复办法，修改jdk/make/common/shared/Compiler-gcc.gmk，去掉-mimpure-text

#### 问题2

		< ../../../src/share/classes/java/util/CurrencyData.properties
Error: time is more than 10 years from present: 1136059200000
java.lang.RuntimeException: time is more than 10 years from present: 1136059200000
	at build.tools.generatecurrencydata.GenerateCurrencyData.makeSpecialCaseEntry(GenerateCurrencyData.java:285)
	at build.tools.generatecurrencydata.GenerateCurrencyData.buildMainAndSpecialCaseTables(GenerateCurrencyData.java:225)
	at build.tools.generatecurrencydata.GenerateCurrencyData.main(GenerateCurrencyData.java:154)
make[4]: *** [/home/xxxx/jdk7_build/lib/currency.data] Error 1

--修复办法，修改jdk/src/share/classes/java/util/CurrencyData.properties

修改文件内容中时间，使其和当前时间差，小于10年。

#### 问题3

PLATFORM_API_LinuxOS_ALSA_CommonUtils.c:(.text+0x36): undefined reference to `snd_lib_error_set_handler'
/home/xxxx/jdk7_build/tmp/sun/javax.sound/jsoundalsa/obj64/PLATFORM_API_LinuxOS_ALSA_PCM.o: In function `DAUDIO_GetFormats':
PLATFORM_API_LinuxOS_ALSA_PCM.c:(.text+0x1f6): undefined reference to `snd_pcm_format_mask_malloc'
PLATFORM_API_LinuxOS_ALSA_PCM.c:(.text+0x203): undefined reference to `snd_pcm_close'
PLATFORM_API_LinuxOS_ALSA_PCM.c:(.text+0x225): undefined reference to `snd_pcm_hw_params_malloc'
PLATFORM_API_LinuxOS_ALSA_PCM.c:(.text+0x236): undefined reference to `snd_pcm_hw_params_get_format_mask'
PLATFORM_API_LinuxOS_ALSA_PCM.c:(.text+0x246): undefined reference to `snd_pcm_format_mask_free'


--修复办法， 修改 jdk/make/javax/sound/jsoundalsa/Makefile
```c
#LDFLAGS += -lasound
OTHER_LDLIBS += -lasound
```

#### 问题4

In file included from /home/xxxx/jdk7_build/gensrc/sun/awt/X11/generator/sizer.64.c:11:0:
../../../src/solaris/native/sun/awt/awt_p.h:51:36: fatal error: X11/extensions/Xrender.h: No such file or directory
 #include <X11/extensions/Xrender.h>
 
--修复办法， apt-get install libxrender-dev

#### 问题5

../../../src/solaris/native/sun/xawt/XToolkit.c:48:34: fatal error: X11/extensions/XTest.h: No such file or directory
 #include <X11/extensions/XTest.h>
                                  ^
compilation terminated.

--修复办法， apt-get install libxtst-dev
