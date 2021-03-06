!include params.mak

!if "$(CPU)" == "AMD64"
!if "$(PLATFORM)" != "x64"
!error "$(PLATFORM) != $(CPU)"
!endif
!else
!if "$(PLATFORM)" != "x86"
!error "$(PLATFORM) != $(CPU)"
!endif
!endif

!if !defined(VC) && defined(_NMAKE_VER)

!if "$(_NMAKE_VER)">"14"
VC=14
!elseif "$(_NMAKE_VER)">"11"
VC=11
!else
#default
VC=10
!endif
!endif


CPPFLAGS = -nologo -D_WIN32_WINNT=0x0500 -DUNICODE -D_UNICODE
LINKFLAGS = -nologo -incremental:no -machine:$(PLATFORM)
CANDLEFLAGS = -nologo -dSourceDir="$(SOURCE_DIR)" -dPlatform=$(PLATFORM) -dVerMajor=$(VER_MAJOR) -dVerMinor=$(VER_MINOR) -dVerBuild=$(VER_BUILD)
LIGHTFLAGS = -nologo -spdb -sval
ASFLAGS = -nologo -c

!if $(VC) >= 11 && "$(PLATFORM)" == "x86"
CPPFLAGS=$(CPPFLAGS) /arch:IA32
!endif

!if $(VC) >= 14
CPPFLAGS = $(CPPFLAGS) /Zc:threadSafeInit-
!endif

!if "$(PLATFORM)" == "x86"
LINKFLAGS = $(LINKFLAGS) -osversion:5.00
CONSUBSYSTEM = -subsystem:console,5.00
WINSUBSYSTEM = -subsystem:windows,5.00
!else
CONSUBSYSTEM = -subsystem:console
WINSUBSYSTEM = -subsystem:windows
!endif

!ifdef RELEASE
OUTDIR = Release
CPPFLAGS = $(CPPFLAGS) -O1 -DNDEBUG
LIGHTFLAGS = $(LIGHTFLAGS) -dcl:high
!else
OUTDIR = Debug
CPPFLAGS = $(CPPFLAGS) -Fd$(OUTDIR)\ -Zi
LINKFLAGS = $(LINKFLAGS) -debug
LIGHTFLAGS = $(LIGHTFLAGS) -cc $(OUTDIR) -reusecab
!endif

OUTDIR = $(OUTDIR).$(PLATFORM)
CPPFLAGS = -Fo$(OUTDIR)\ $(CPPFLAGS)
CANDLEFLAGS = -out $(OUTDIR)\ $(CANDLEFLAGS)
ASFLAGS = $(ASFLAGS) -Fo$(OUTDIR)\\

WIXOBJ = $(OUTDIR)\installer.wixobj $(OUTDIR)\files.wixobj $(OUTDIR)\features.wixobj $(OUTDIR)\shortcuts.wixobj $(OUTDIR)\ui.wixobj
!ifdef NIGHTLY
MSI_NAME = $(SOURCE_DIR)\final.msi
LIGHTFLAGS = $(LIGHTFLAGS) -sh
!else
MSI_NAME = $(OUTDIR)\far.msi
!endif

!if "$(PLATFORM)" == "x86"
VC10OBJ = $(OUTDIR)\vc_crt_fix.obj
VC10CPP = ..\..\plugins\common\vc_crt_fix_impl.cpp
!endif

!ifdef NIGHTLY
$(MSI_NAME): $(OUTDIR) $(WIXOBJ) en-us.wxl ui_en-us.wxl $(OUTDIR)\customact.dll
  light $(LIGHTFLAGS) -cultures:en-us -loc en-us.wxl -loc ui_en-us.wxl -out $@ $(WIXOBJ)
!else
$(MSI_NAME): $(OUTDIR) $(OUTDIR)\1033.msi
  copy /y $(OUTDIR)\1033.msi $(MSI_NAME)

$(OUTDIR)\1033.msi: $(WIXOBJ) en-us.wxl ui_en-us.wxl $(OUTDIR)\customact.dll
  light $(LIGHTFLAGS) -cultures:en-us -loc en-us.wxl -loc ui_en-us.wxl -out $@ $(WIXOBJ)

$(OUTDIR)\1049.msi: $(WIXOBJ) ru-ru.wxl ui_ru-ru.wxl $(OUTDIR)\customact.dll
  light $(LIGHTFLAGS) -cultures:ru-ru -loc ru-ru.wxl -loc ui_ru-ru.wxl -out $@ $(WIXOBJ)

$(OUTDIR)\1049: $(OUTDIR)\1033.msi $(OUTDIR)\1049.msi
  torch -nologo -t language $(OUTDIR)\1033.msi $(OUTDIR)\1049.msi -out $@
!endif

$(OUTDIR)\customact.dll: customact.cpp $(VC10CPP) customact.def $(OUTDIR)\customact.res $(VC10OBJ)
  cl $(CPPFLAGS) -EHsc customact.cpp $(VC10CPP) -link $(LINKFLAGS) $(WINSUBSYSTEM) -dll -out:$@ -def:customact.def $(OUTDIR)\customact.res $(VC10OBJ) msi.lib shell32.lib rpcrt4.lib

$(OUTDIR)\customact.res: customact.rc $(OUTDIR)\consize.exe
  rc -nologo -DOUTDIR=$(OUTDIR) -fo$@ customact.rc

$(OUTDIR)\consize.exe: consize.cpp
  cl $(CPPFLAGS) -GS- consize.cpp -link $(LINKFLAGS) $(CONSUBSYSTEM) -nodefaultlib -entry:main -out:$@ kernel32.lib

$(WIXOBJ): params.mak defines.wxi guids_$(PLATFORM).wxi

.wxs{$(OUTDIR)}.wixobj::
  candle $(CANDLEFLAGS) $<

$(OUTDIR)\vc_crt_fix.obj: ..\..\plugins\common\vc_crt_fix.asm
  $(AS) $(ASFLAGS) $**

$(OUTDIR):
  if not exist $(OUTDIR) mkdir $(OUTDIR)


clean:
  if exist $(OUTDIR) rd /s /q $(OUTDIR)


.SUFFIXES: .wxs
.PHONY: clean
