<# :
:: Many many many thanks and credits to:
::  - original idea and first script by Sajjo for ATI 2020 v24.x.x.x
::  - complete script redesign and most work on it by ZORAX
::  - new patches for ATI 2021 by Ash P and KleineZiege
::
:: 08/26/2020, skyteddy: initial script for ATI 2021 v25.4.1.30290, https://forums.mydigitallife.net/posts/1614754
::   - script based on the last script by ZORAX for ATI 24.6.1.25700, https://forums.mydigitallife.net/posts/1602801
::   - patches by Ash P and KleineZiege
::   - added 2 more files for patching, stopping processes and firewall rules
::   - registry entries modified
::
:: 08/27/2020, skyteddy: added ATI 2021 v25.4.1.30480, https://forums.mydigitallife.net/posts/1615012
::
:: 10/07/2020, skyteddy: added ATI 2021 v25.5.1.32010, https://forums.mydigitallife.net/posts/1622182
::
:: 11/24/2020, skyteddy: added ATI 2021 v25.6.1.34340, https://forums.mydigitallife.net/posts/1632490
::
:: 12/22/2020, skyteddy: added ATI 2021 v25.6.1.35860, https://forums.mydigitallife.net/posts/1637025
::
:: 03/11/2021, skyteddy: added ATI 2021 v25.7.1.39184, https://forums.mydigitallife.net/posts/1649328
::
:: 03/30/2021, skyteddy: added ATI 2021 v25.8.1.39216, https://forums.mydigitallife.net/posts/1652708
::
:: 01/27/2022, skyteddy: added ATI 2021 v25.10.1.39287, https://forums.mydigitallife.net/posts/1719701


@set "SUPPORTED_VERSIONS=25.4.30290;25.4.30480;25.5.32010;25.6.34340;25.6.35860;25.7.39184;25.8.39216;25.10.39287"

@goto :SET_VARIABLES
:START
echo %_GREEN2%=============================================================================
echo =  %_CYAN2%Hybrid script for patching and activation of %_GREEN2%                            =
echo =                                                                           =
echo =                         %_MAGENTA2% Acronis True Image 2021 %_GREEN2%                         =
echo =                                                                           =
echo =  Supported versions:  %_CYAN2%25.4.1.30290%_GREEN2%                                        =
echo =                       %_CYAN2%25.4.1.30480%_GREEN2%                                        =
echo =                       %_CYAN2%25.5.1.32010%_GREEN2%                                        =
echo =                       %_CYAN2%25.6.1.34340%_GREEN2%                                        =
echo =                       %_CYAN2%25.6.1.35860%_GREEN2%                                        =
echo =                       %_CYAN2%25.7.1.39184%_GREEN2%                                        =
echo =                       %_CYAN2%25.8.1.39216%_GREEN2%                                        =
echo =                       %_CYAN2%25.10.1.39287%_GREEN2%                                       =
echo =                                                                           =
if not defined _HELP echo =  Run with%_WHITE2% /h%_GREEN2% parameter for usage instructions.                            =
echo =============================================================================%_NOCOLOR%
echo.
if %_VERBOSE% "VERBOSE mode is set"
if defined _HELP (
	call :HELP
	goto :EXIT
)
if defined _YES if %_VERBOSE% "Unattended installation is set"
net session %_nul3% || goto :ELEVATE
if %_VERBOSE% "Test for Administrator rights OK"
if %_VERBOSE% "Operating system build number is %_OSBUILD%"

:QUERY
set ATIH=
if exist "%ProgramFiles%\Acronis\TrueImageHome\TrueImage.exe" (
	set ATIH=x86
	set "PFDIR=%ProgramFiles%"
	set "REG_UNINSTALL=HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"
)
if exist "%ProgramFiles(x86)%\Acronis\TrueImageHome\TrueImage.exe" (
	set ATIH=x64
	set "PFDIR=%ProgramFiles(x86)%"
	set "REG_UNINSTALL=HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall"
)
if %_VERBOSE% "Program files path for Acronis set to %PFDIR%"
if not defined ATIH (
	call :ERROR "TrueImage.exe not found in default installation paths!" || exit /b 1
)
set KEY_UNINSTALL=
if %_VERBOSE% "Looking for uninstall info in %REG_UNINSTALL%"
for /f %%A in ('reg.exe query %REG_UNINSTALL% /f {*') do (
	reg.exe query %%A /v DisplayName %_nul2% | find "Acronis True Image" %_nul1% && set "KEY_UNINSTALL=%%A"
)
if not defined KEY_UNINSTALL (
	call :ERROR "Cannot find Acronis uninstall registry keys!" || exit /b 1
)
if %_VERBOSE% "Found Acronis uninstall info in %KEY_UNINSTALL%"
set VERSION=
for /f "tokens=3" %%A in ('reg.exe query %KEY_UNINSTALL% /v DisplayVersion') do (
	set "VERSION=%%A"
)
if not defined VERSION (
	call :ERROR "Cannot find installed version in registry keys!" || exit /b 1
)
if %_VERBOSE% "Value found in DisplayVersion is %VERSION%"
set BUILD=
for %%A in (%SUPPORTED_VERSIONS%) do (
	if "%%A"=="%VERSION%" set BUILD=%VERSION:~-5%
)
if not defined BUILD (
	call :ERROR "Installed version %VERSION% is not supported by this script!" || exit /b 1
)

set PATCHED_BUILD=
if exist "%PFDIR%\Acronis\TrueImageHome\patched.ver" set /P PATCHED_BUILD= <"%PFDIR%\Acronis\TrueImageHome\patched.ver"
set _SAMEVER=
if %BUILD% EQU %PATCHED_BUILD% set _SAMEVER=1

if %_VERBOSE% line
echo Installed version %_CYAN2%%VERSION%%_NOCOLOR% found in a %_CYAN2%%ATIH%%_NOCOLOR% system.
if defined PATCHED_BUILD if %_VERBOSE% "Patch was previously run for build %PATCHED_BUILD%"
echo.
if defined _SAMEVER (
	echo %_YELLOW2%It seems the script has already been applied to the version above.%_NOCOLOR%
	echo.
	echo You may run the script again in an already patched system to verify the files
	echo and update system settings. Backup files will not be changed.
	goto :CHOICE
)
echo %_YELLOW2%The following files will be patched:%_NOCOLOR%
echo.
echo %_WHITE2%%HOSTS%
echo %PFDIR%\Acronis\TrueImageHome\ga_service.exe
echo %PFDIR%\Acronis\TrueImageHome\ti_managers.dll
echo %PFDIR%\Acronis\TrueImageHome\TrueImageTools.exe
echo %PFDIR%\Acronis\TrueImageHome\acronis_drive.exe
echo %PFDIR%\Acronis\TrueImageHome\mobile_backup_status_server.exe
echo %PFDIR%\Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe%_NOCOLOR%
echo.
echo Backups (.bak) will be created before patching and saved in same location.
echo A checksum routine prevents patching wrong files.
:CHOICE
if defined _YES goto :SERVICES
echo.
echo %_WHITE2%Do you want to proceed? [Y/N] %_NOCOLOR%
choice /c yn /n >nul
if %errorlevel% NEQ 1 goto :EXIT

:SERVICES
echo.
echo %_CYAN2%Stopping Acronis services and processes...%_NOCOLOR%
net stop afcdpsrv %_nul3%
net stop mmsminisrv %_nul3%
net stop AcrSch2Svc %_nul3%
net stop syncagentsrv %_nul3%
wmic process where name='tib_mounter_monitor.exe' delete %_nul3%
wmic process where name='TrueImageMonitor.exe' delete %_nul3%
wmic process where name='TrueImageTools.exe' delete %_nul3%
wmic process where name='anti_ransomware_service.exe' delete %_nul3%
wmic process where name='ga_service.exe' delete %_nul3%

:HOSTS
echo.
echo %_CYAN2%Patching HOSTS file...%_NOCOLOR%
attrib -R -H -S "%HOSTS%" %_nul2%
if exist "%HOSTS%.bak" (
	echo %_YELLOW2%WARNING: File HOSTS.bak already exists! A new backup will not be created.%_NOCOLOR%
) else (
	copy /Y "%HOSTS%" "%HOSTS%.bak" %_nul3%
)
echo. >>"%HOSTS%" || (
	echo %_YELLOW2%WARNING: Error attempting to write to HOSTS file! %_NOCOLOR%
	echo You might have to edit it manually.
	goto :FIREWALL
)
call :ADDHOST liveupdate.acronis.com
call :ADDHOST activation.acronis.com
call :ADDHOST web-api-tih.acronis.com
call :ADDHOST download.acronis.com
call :ADDHOST orders.acronis.com
call :ADDHOST ns1.acronis.com
call :ADDHOST ns2.acronis.com
call :ADDHOST ns3.acronis.com
call :ADDHOST account.acronis.com
call :ADDHOST gateway.acronis.com
echo Flushing DNS cache...
ipconfig.exe /flushdns %_nul1%

:FIREWALL
echo.
echo %_CYAN2%Deleting Acronis firewall permissions...%_NOCOLOR%
for %%A in (
"Acronis Drive"
"Acronis Managed Machine Service Mini"
"Acronis Media Builder"
"Acronis Mobile Backup Server"
"Acronis Mobile Backup Status Server"
"Acronis Sync Agent Service"
"Acronis System Report"
"Acronis True Image 2021"
"acronis_drive.exe"
"ga_service"
"ga_service.exe"
"LicenseActivator.exe"
"mobile_backup_status_server.exe"
"report_sender.exe"
"TrueImageHomeService.exe"
"TrueImageMonitor.exe"
"TrueImageTools.exe"
"AcronisTrueImage"
"AcronisLicenseActivator"
"AcronisSyncAgent"
) do (
	netsh advfirewall firewall show rule name=%%A %_nul3% && (
		if %_VERBOSE% "Deleting rule name %%~A"
		netsh advfirewall firewall delete rule name=%%A %_nul3%
	)
)
echo.
echo %_CYAN2%Creating new firewall restrictions...%_NOCOLOR%
if %_VERBOSE% "Creating BLOCK rule for %PFDIR%\Acronis\TrueImageHome\LicenseActivator.exe"
netsh advfirewall firewall add rule name="AcronisLicenseActivator" dir=out action=block profile=any program="%PFDIR%\Acronis\TrueImageHome\LicenseActivator.exe" %_nul3%
if %_VERBOSE% "Creating BLOCK rule for %PFDIR%\Common Files\Acronis\SyncAgent\syncagentsrv.exe"
netsh advfirewall firewall add rule name="AcronisSyncAgent" dir=out action=block profile=any program="%PFDIR%\Common Files\Acronis\SyncAgent\syncagentsrv.exe" %_nul3%

:OS_CONDITION
echo.
if "%ATIH%"=="x64" goto :REG_X64

:REG_X86
echo %_CYAN2%Adding registry keys for x86 system...%_NOCOLOR%
reg add "HKLM\SOFTWARE\Acronis\TrueImage" /v "standard" /t REG_SZ /d " 12108  0 31109 12 24  0120  3  7 31 98 23  0 18 13120 18  6 97 27  2  1 16109120  5109 17 17 31 29 98  3120 97  4  7 13 12  3 98 30120 18  1  2 31 20  0 97 23120 16 17102 30  3  6 18  5120  2  0 25 96  2103 13 97" /f
reg add "HKLM\SOFTWARE\Acronis\TrueImageHome\Settings" /v "ActivatoionBlob" /t REG_SZ /d "{ \"Error\" : 0, \"Status\" : \"\", \"Value\" : \"1234\", \"Version\" : 1}" /f
goto :PATCH

:REG_X64
echo %_CYAN2%Adding registry keys for x64 system...%_NOCOLOR%
reg add "HKLM\SOFTWARE\WOW6432Node\Acronis\TrueImage" /v "standard" /t REG_SZ /d " 12108  0 31109 12 24  0120  3  7 31 98 23  0 18 13120 18  6 97 27  2  1 16109120  5109 17 17 31 29 98  3120 97  4  7 13 12  3 98 30120 18  1  2 31 20  0 97 23120 16 17102 30  3  6 18  5120  2  0 25 96  2103 13 97" /f
reg add "HKLM\SOFTWARE\WOW6432Node\Acronis\TrueImageHome\Settings" /v "ActivatoionBlob" /t REG_SZ /d "{ \"Error\" : 0, \"Status\" : \"\", \"Value\" : \"1234\", \"Version\" : 1}" /f

:PATCH
echo.
if not defined _SAMEVER if exist "%PFDIR%\Acronis\TrueImageHome\ga_service.exe.bak" (
	if %_VERBOSE% "Deleting old ga_service.exe.bak"
	del /F "%PFDIR%\Acronis\TrueImageHome\ga_service.exe.bak"
)
if not exist "%PFDIR%\Acronis\TrueImageHome\ga_service.exe.bak" (
	if %_VERBOSE% "Backing up ga_service.exe"
	copy /Y "%PFDIR%\Acronis\TrueImageHome\ga_service.exe" "%PFDIR%\Acronis\TrueImageHome\ga_service.exe.bak" %_nul1%
)
echo %_CYAN2%Patching %PFDIR%\Acronis\TrueImageHome\ga_service.exe%_NOCOLOR%
type NUL >"%PFDIR%\Acronis\TrueImageHome\ga_service.exe"
echo.
powershell.exe -c "Invoke-Expression $([System.IO.File]::ReadAllText('%~f0'))"
echo.


echo %_GREEN2%Activation complete! %_NOCOLOR%
echo %BUILD% >"%PFDIR%\Acronis\TrueImageHome\patched.ver"
goto :EXIT

:HELP
if %_VERBOSE% line
echo %_WHITE2%Usage instructions:%_NOCOLOR%
echo    1) Install one of the supported versions of Acronis True Image 2021
echo    2) Run the script
echo    3) Have fun ;-)
echo    4) Remember to always backup your important data!
echo.
echo %_WHITE2%Command line parameters:%_NOCOLOR%
echo(   /h ^| /?   Show these instructions
echo    /y        Run the script completely unattended
echo    /v        Enable verbose mode
goto :EOF

:SET_VARIABLES
@echo off
title ATI 2021 Activation Script
setlocal
set "_nul1=>nul" && set "_nul2=2>nul" && set "_nul3=>nul 2>nul"
chcp 1252 %_nul1%
set _YES=
set _HELP=
set "_VERBOSE=defined _VERBOSE rem"
set PS_VERBOSE=

:PARAMETERS
if /I '%1'=='/v' set "_VERBOSE=defined _VERBOSE call :VERBOSE" && set "PS_VERBOSE=$true"
if /I '%1'=='-v' set "_VERBOSE=defined _VERBOSE call :VERBOSE" && set "PS_VERBOSE=$true"
if /I '%1'=='/y' set _YES=1
if /I '%1'=='-y' set _YES=1
if /I '%1'=='-h' set _HELP=1
if /I '%1'=='/h' set _HELP=1
if /I '%1'=='-?' set _HELP=1
if /I '%1'=='/?' set _HELP=1
if not '%1'=='' shift /1 && goto :PARAMETERS

set "_OSBUILD="
for /f %%A in ('WMIC OS Get BuildNumber 2^>nul ^| findstr /r "[0-9]"') do set "_OSBUILD=%%A"
if not defined _YES if %_OSBUILD% GEQ 10586 (
	set "_NOCOLOR=[0m" && set "_BRIGHT=[1m"
	set "_RED1=[31m" && set "_GREEN1=[32m" &&	set "_YELLOW1=[33m" && set "_BLUE1=[34m" && set "_MAGENTA1=[35m" && set "_CYAN1=[36m"
	set "_RED2=[91m" && set "_GREEN2=[92m" &&	set "_YELLOW2=[93m" && set "_BLUE2=[94m" && set "_MAGENTA2=[95m" && set "_CYAN2=[96m" && set "_WHITE2=[97m"
)
set "HOSTS=%WINDIR%\system32\drivers\etc\hosts"
echo.
if not defined _YES cls
goto :START

:ELEVATE
if defined _YES goto :NOELEVATE
if %_VERBOSE% "Requesting elevation..."
powershell.exe -C "Start-Process cmd.exe -ArgumentList '/c %~fs0 %*' -Verb RunAs" %_nul3%
if not ERRORLEVEL 1 endlocal && exit /b 0
if %_VERBOSE% "Elevation denied"
call :HELP
:NOELEVATE
call :ERROR "Administrator rights required!"
exit /b 255

:ADDHOST
find /I "%1" "%HOSTS%" %_nul3%% || (
	if %_VERBOSE% "Blocking access to %1"
	echo.127.0.0.1 %1>>"%HOSTS%"
)
goto :EOF

:EXIT
if not defined _YES (
	echo.
	echo Press any key to exit...
	pause %_nul1%
)
endlocal
exit /b 0

:VERBOSE
if "%~1"=="line" echo. && goto :EOF
echo %_CYAN1%%~1%_NOCOLOR%
goto :EOF

:ERROR
echo.
echo %_RED2%ERROR: %~1%_NOCOLOR%
if not defined _YES (
	echo.
	echo Press any key to exit...
	pause %_nul1%
)
endlocal
exit /b 1

## POWERSHELL SCRIPT #>
## VALUES
$version30290 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"9894640d385f595e656284649ccca659";
	patched	=	"184e4845d07fc934912076fd3a75c0fb";
	changes	=	@((1272916,144),(1272917,144),(8823122,144),(8823123,144),(8823125,233),(8823126,156),(8823127,0),(8823130,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"f1b199c1c967bdbec4b263ac176692e0";
	patched	=	"752337b4eb284f0a6b31333d52bfb62e";
	changes	=	@((11032418,144),(11032419,144),(11032421,233),(11032422,156),(11032423,0),(11032426,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"7d51f8e34172d8509a3628f695715668";
	patched	=	"58b8916436ec5b511ca0e57124d074b8";
	changes	=	@((1457394,144),(1457395,144),(1457397,233),(1457398,156),(1457399,0),(1457402,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"a1b5dca9666ea28078205777a5b6d362";
	patched	=	"c3c9d151c3c1adf4798dca083c47d06a";
	changes	=	@((529362,144),(529363,144),(529365,233),(529366,156),(529367,0),(529370,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"1e8c4ddac978af59d4bca33a019a6c85";
	patched	=	"35b42d8e4343d45a4d33c834ceb2982f";
	changes	=	@((9263426,144),(9263427,144),(9263429,233),(9263430,156),(9263431,0),(9263434,144));
})

$version30480 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"fb291f5a505b01897a2e4d3ebca58638";
	patched	=	"d0591a9f2b85ac02d416a9816b044cc7";
	changes	=	@((1272772,144),(1272773,144),(8823122,144),(8823123,144),(8823125,233),(8823126,156),(8823127,0),(8823130,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"adeec764835b546213a21a8064ded9d1";
	patched	=	"35942dddf97091f1dfda33459dbc923b";
	changes	=	@((11033458,144),(11033459,144),(11033461,233),(11033462,156),(11033463,0),(11033466,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"ee631b6152384881c9ba5639a9503db1";
	patched	=	"afee61edc858514b46e7c307d1321609";
	changes	=	@((1456834,144),(1456835,144),(1456837,233),(1456838,156),(1456839,0),(1456842,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"a45a408f1bfd001b754a7e83fa70bdfe";
	patched	=	"934b07f56c028b083a89331195fc2bb6";
	changes	=	@((529378,144),(529379,144),(529381,233),(529382,156),(529383,0),(529386,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"4ce360cac1cdbc39c6104e550dda4731";
	patched	=	"9617b2cba2a7fa1324a3f40b23c833bb";
	changes	=	@((9263890,144),(9263891,144),(9263893,233),(9263894,156),(9263895,0),(9263898,144));
})

$version32010 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"7ac8d207fc260bcee9530893265d5cff";
	patched	=	"959e09c8ffa9b334b737c1b138fa974a";
	changes	=	@((1273732,144),(1273733,144),(8834786,144),(8834787,144),(8834789,233),(8834790,156),(8834791,0),(8834794,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"c3aaaf927e0a0d1cc49a43a896621710";
	patched	=	"8258ddf89f1e09d717abbc269be2cd91";
	changes	=	@((11051730,144),(11051731,144),(11051733,233),(11051734,156),(11051735,0),(11051738,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"9e3056eaaf1af3aeb4093d9865bcefd3";
	patched	=	"6ea8db7f17f44d57390e7578a49823a2";
	changes	=	@((1456290,144),(1456291,144),(1456293,233),(1456294,156),(1456295,0),(1456298,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"b4af581072fe6e90b82441cb9bb17bd3";
	patched	=	"6bce76c23462e70c7b3b81c1cb2bc02e";
	changes	=	@((529058,144),(529059,144),(529061,233),(529062,156),(529063,0),(529066,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"b815b8b409532005db13a95528925674";
	patched	=	"67a3cb1dd8a10176ace5f9c9738d9d0f";
	changes	=	@((9262034,144),(9262035,144),(9262037,233),(9262038,156),(9262039,0),(9262042,144));
})

$version34340 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"6564c66d190950d61a5b662c78b58d39";
	patched	=	"f7ac7e96a3e414211215a4882c10b1bf";
	changes	=	@((1302884,144),(1302885,144),(9653922,144),(9653923,144),(9653925,233),(9653926,156),(9653927,0),(9653930,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"b067e7da2eea6394347c316a77b27dc5";
	patched	=	"c8ebf69b55024a2091e61e0c94466837";
	changes	=	@((11066658,144),(11066659,144),(11066661,233),(11066662,156),(11066663,0),(11066666,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"b9b395ccb383b7bddd545b4bbca77ceb";
	patched	=	"f4fe5fc6b713e42babf78b61fa84672b";
	changes	=	@((1466018,144),(1466019,144),(1466021,233),(1466022,156),(1466023,0),(1466026,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"94d92356f2166968a3b35d50cae5ce9c";
	patched	=	"a2a4747043fbeccbf649b663dd15e5f5";
	changes	=	@((532034,144),(532035,144),(532037,233),(532038,156),(532039,0),(532042,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"173ab48e1f55653da286c42de7d46b36";
	patched	=	"47d9946d9323be2d3b65bbe73c403dbb";
	changes	=	@((9286434,144),(9286435,144),(9286437,233),(9286438,156),(9286439,0),(9286442,144));
})

$version35860 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"fc165021d17d15412c40c8e2e613b5f1";
	patched	=	"b0e2f447a59755b0ce03bbf8ea8c5353";
	changes	=	@((1300836,144),(1300837,144),(9651602,144),(9651603,144),(9651605,233),(9651606,156),(9651607,0),(9651610,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"2bf8c1b9f4aa6953182c63e66c6c1a60";
	patched	=	"a79df8c1520f5b9dd0e975c56ed064d8";
	changes	=	@((11068290,144),(11068291,144),(11068293,233),(11068294,156),(11068295,0),(11068298,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"6ef218ec51d8a1daa7f98cfdbb3016b7";
	patched	=	"4f063c055f17b85d4addb010056d6b6a";
	changes	=	@((1465378,144),(1465379,144),(1465381,233),(1465382,156),(1465383,0),(1465386,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"e0820dda826df9f12bfe914d98f07870";
	patched	=	"7930f310ad3c5a37240be182e411beec";
	changes	=	@((532530,144),(532531,144),(532533,233),(532534,156),(532535,0),(532538,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"5b3cf8070172314db3c595c3688bb7e9";
	patched	=	"55fc579e009813bbaae3d2cc572b5bec";
	changes	=	@((9285874,144),(9285875,144),(9285877,233),(9285878,156),(9285879,0),(9285882,144));
})

$version39184 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"5a61e42dac1ecf3d9b9d83c3f96ed19e";
	patched	=	"e901ff762f994d29932728385f1b9716";
	changes	=	@((1496948,144),(1496949,144),(9818722,144),(9818723,144),(9818725,233),(9818726,156),(9818727,0),(9818730,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"a2837b4f8f40e7d632b805b50b55df90";
	patched	=	"3254dc4e00e969977579b21ee5136aba";
	changes	=	@((11497586,144),(11497587,144),(11497589,233),(11497590,156),(11497591,0),(11497594,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"305198c92ee83aa78959018fa7e6ab57";
	patched	=	"e2b09e217cacf1b211c2d893662abdfa";
	changes	=	@((1476034,144),(1476035,144),(1476037,233),(1476038,156),(1476039,0),(1476042,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"c4ca8ac0563e6f43ca34d556e36fc6eb";
	patched	=	"87334afc8c98cade4a2e9feab2612141";
	changes	=	@((534290,144),(534291,144),(534293,233),(534294,156),(534295,0),(534298,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"803c8bba5257a893a702e998ec6146cc";
	patched	=	"9cb7d0ec3dd6980aecd6ef9ef4a1cf11";
	changes	=	@((9385234,144),(9385235,144),(9385237,233),(9385238,156),(9385239,0),(9385242,144));
})

$version39216 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"8d33bbbecac6ecf3cb83821a968534df";
	patched	=	"4882bdbd43fca35080c0aaa1b31e87f5";
	changes	=	@((1495748,144),(1495749,144),(9827490,144),(9827491,144),(9827493,233),(9827494,156),(9827495,0),(9827498,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"96bc5bca94f5d040a3921abf9c8016d3";
	patched	=	"9dd7b1b57146cf98bb9e302506c3c3f2";
	changes	=	@((11506706,144),(11506707,144),(11506709,233),(11506710,156),(11506711,0),(11506714,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"75b13df4fa5cfbbc8265bb0dda489bc6";
	patched	=	"1c557f218da1355241c5ebfa0714f70d";
	changes	=	@((1480946,144),(1480947,144),(1480949,233),(1480950,156),(1480951,0),(1480954,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"72cb913fc0b05fb2f4cdc6186ef9d71c";
	patched	=	"584d0ddc5a8f3b798bb337d081767c3b";
	changes	=	@((533906,144),(533907,144),(533909,233),(533910,156),(533911,0),(533914,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"4312a4850d335603118036caffa6524b";
	patched	=	"2d892576b614d6c10ec6d6fccd8d81b2";
	changes	=	@((9398882,144),(9398883,144),(9398885,233),(9398886,156),(9398887,0),(9398890,144));
})

$version39287 = @(
@{
	name	=	"Acronis\TrueImageHome\ti_managers.dll";
	md5		=	"4badfe88903aede931ef7a307d117e29";
	patched	=	"18bbd26c82cf69689fe5323b8595e179";
	changes	=	@((1493924,144),(1493925,144),(9826722,144),(9826723,144),(9826725,233),(9826726,156),(9826727,0),(9826730,144));
},
@{
	name	=	"Acronis\TrueImageHome\TrueImageTools.exe";
	md5		=	"4abe3a979b43c509b79fb0f631b7a953";
	patched	=	"c6d0ba251a51252518fc1f5c9bb34d0e";
	changes	=	@((11509202,144),(11509203,144),(11509205,233),(11509206,156),(11509207,0),(11509210,144));
},
@{
	name	=	"Acronis\TrueImageHome\acronis_drive.exe";
	md5		=	"071b5eb9fcceec16d2a863e0ae312a79";
	patched	=	"ea3de1db31ac659afabe945e56c61570";
	changes	=	@((1482562,144),(1482563,144),(1482565,233),(1482566,156),(1482567,0),(1482570,144));
},
@{
	name	=	"Acronis\TrueImageHome\mobile_backup_status_server.exe";
	md5		=	"8b1486c75e5c0164091b6f0ac9d8ae80";
	patched	=	"959a17bdd1df7bbb7fc245161eca5ae6";
	changes	=	@((532434,144),(532435,144),(532437,233),(532438,156),(532439,0),(532442,144));
},
@{
	name	=	"Common Files\Acronis\TrueImageHome\TrueImageHomeService.exe";
	md5		=	"20053b97d1e44af846951dd4b4651196";
	patched	=	"63de06ab077f09f3cda0ffbf3802ff99";
	changes	=	@((9396450,144),(9396451,144),(9396453,233),(9396454,156),(9396455,0),(9396458,144));
})

Write-Host -ForegroundColor Magenta "POWERSHELL: Starting binary patching script"
Try {
$files		= Get-Variable $('version' + $env:BUILD) -ValueOnly
$hasher		= New-Object -TypeName System.Security.Cryptography.MD5CryptoServiceProvider;
foreach ($file in $files) {
	## MAKE SURE FILE EXISTS
	$fileName	= Join-Path $env:PFDIR $file.name
	if (Test-Path $fileName) {
		## READ FILE
		Write-Host -ForegroundColor Cyan "`nReading file $fileName"
		$binary = [System.IO.File]::ReadAllBytes($fileName);
		
		## CHECKSUM VERIFY
		$fileHash = [System.BitConverter]::ToString($hasher.ComputeHash($binary)) -replace ('\-','');
		if ($env:PS_VERBOSE) { Write-Host -ForegroundColor DarkCyan "MD5 checksum is $fileHash" }
		
		if ($file.patched -eq $fileHash) {
			Write-Host -ForegroundColor Green "File has already been patched!"
		} elseif ($file.md5 -eq $fileHash) {
			## BACKUP FILE
			Copy-Item -Path $fileName -Destination ($fileName+".bak") -Force;
			## PATCH
			if ($env:PS_VERBOSE) { Write-Host -ForegroundColor White "Address      patching to" }
			foreach ($change in $file.changes) {
				if ($env:PS_VERBOSE) { Write-Host "$('0x{0:X8}' -f $change[0])   $('{0:X2}' -f $binary[$change[0]]) ----> $('{0:X2}' -f $change[1])" }
				$binary[$change[0]] = $change[1];
			}
			## WRITE FILE
			if ($env:PS_VERBOSE) { Write-Host -ForegroundColor DarkCyan "Writing patched file back..." }
			[System.IO.File]::WriteAllBytes($fileName, $binary);
			Write-Host "Patching done!";
		} else {
			Write-Host -ForegroundColor Red "Wrong checksum! Expected: $($file.md5)`nFile will not be patched!";
		}
	} else {
		Write-Host -ForegroundColor Red "$($fileName) not found!`nFile cannot be patched!";
	}	
}
} Finally {
Remove-Variable binary -ErrorAction SilentlyContinue
}
