1>  jsstr.cpp
ENABLE_ASSEMBLER
ENABLE_JIT

ENABLE_JIT=0
ENABLE_YARR_JIT=0

--------------------------
define:

XP_WIN
SVR4
SYSV
_BSD_SOURCE
POSIX_SOURCE
X86_LINUX
_CRT_SECURE_NO_WARNINGS
__MWERKS__
ENABLE_JIT=0
ENABLE_YARR_JIT=0

---------------------------

https://blog.csdn.net/sun007700/article/details/100829527

http://www.it1352.com/349390.html

#include "stdafx.h"
#include <time.h>
#include <windows.h> 
 
const __int64 DELTA_EPOCH_IN_MICROSECS= 11644473600000000;
 
/* IN UNIX the use of the timezone struct is obsolete;
 I don't know why you use it. See http://linux.about.com/od/commands/l/blcmdl2_gettime.htm
 But if you want to use this structure to know about GMT(UTC) diffrence from your local time
 it will be next: tz_minuteswest is the real diffrence in minutes from GMT(UTC) and a tz_dsttime is a flag
 indicates whether daylight is now in use
*/
struct timezone2 
{
  __int32  tz_minuteswest; /* minutes W of Greenwich */
  bool  tz_dsttime;     /* type of dst correction */
};
 
struct timeval2 {
__int32    tv_sec;         /* seconds */
__int32    tv_usec;        /* microseconds */
};
 
int gettimeofday(struct timeval2 *tv/*in*/, struct timezone2 *tz/*in*/)
{
  FILETIME ft;
  __int64 tmpres = 0;
  TIME_ZONE_INFORMATION tz_winapi;
  int rez=0;
 
   ZeroMemory(&ft,sizeof(ft));
   ZeroMemory(&tz_winapi,sizeof(tz_winapi));
 
    GetSystemTimeAsFileTime(&ft);
 
    tmpres = ft.dwHighDateTime;
    tmpres <<= 32;
    tmpres |= ft.dwLowDateTime;
 
    /*converting file time to unix epoch*/
    tmpres /= 10;  /*convert into microseconds*/
    tmpres -= DELTA_EPOCH_IN_MICROSECS; 
    tv->tv_sec = (__int32)(tmpres*0.000001);
    tv->tv_usec =(tmpres%1000000);
 
 
    //_tzset(),don't work properly, so we use GetTimeZoneInformation
    rez=GetTimeZoneInformation(&tz_winapi);
    tz->tz_dsttime=(rez==2)?true:false;
    tz->tz_minuteswest = tz_winapi.Bias + ((rez==2)?tz_winapi.DaylightBias:0);
 
  return 0;
}
 
 
int _tmain(int argc, _TCHAR* argv[])
{
struct timeval2 tv;
struct timezone2 tz;
struct tm *tm1; 
time_t time1;
 
ZeroMemory(&tv,sizeof(tv));
ZeroMemory(&tz,sizeof(tz));
 
gettimeofday(&tv, &tz); // call gettimeofday()
time1=tv.tv_sec;
tm1 = localtime(&time1); 
 
 
 
 FILE *f;
 f=fopen("rez.txt","w");
 
 fprintf(f,"%04d.%02d.%02d %02d:%02d:%02d\n",1900+tm1->tm_year,1+tm1->tm_mon,tm1->tm_mday,tm1->tm_hour,tm1->tm_min,tm1->tm_sec);
 fprintf(f,"Diffrence between GMT(UTC) and local time=%d %s\n",tz.tz_minuteswest,"minutes");
 fprintf(f,"Is Daylight now=%s\n",tz.tz_dsttime?"Yes":"No");
 
 fclose(f);
 return 0;
}

