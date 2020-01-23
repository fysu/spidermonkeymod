* Build original source with nmake    
$ cd src   
$ nmake -f js.mak  

------------------
some mod for compiling success: 

#ifdef XP_WIN
#include <windef.h>
#include <winbase.h>
#endif
->
#ifdef XP_WIN
#include <windows.h>
#endif

