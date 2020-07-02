---
layout: post
title:  "MineCraft Startup Scripts"
subtitle: "The Ultimate IPtables Guide"
date:   2020-06-13 13:33:37
tags: [minecraft,minecraft java,minecraft java edition,minecraft bedrock,minecraft bedrock edition,startup script,script, start.bat, run.bat]
categories: [scripts]
---

There are many subpar startup scripts on the market. Most of them have only a MineCraft Java edition support. What is more, languages like C# and Java are used, which
often makes them pretty resource-heavy. This is why this server startup solution is written in BATCH - the scripting language of  [CMD](https://en.wikipedia.org/wiki/Cmd.exe "CMD on Wikipedia").

**NOTE:** This script works ONLY on Windows OS and has been tested (and developed) on Windows 10.


**The MC:Java Edition Startup Script**

This edition of the script allows server owners to start  **ANY**  server jarfile. It can also auto-update (and redownload) the latest PaperSpigot build for the version chosen.
The list of available commands is:

* /launch - start the server
* /jarupdate - update or download the server jarfile
* /exit - gracefully shut the script down
* /logclean or /logclear or /clearlogs - clear the logs
* /clear - clear the console window
* /credits - info about the script author
* /help - shows a list of these commands

And now - onto the scipt:

This codeblock shows an example app banner. This can be freely changed to anything you like.
```markdown
#Example banner.cfg
    __  __________       __                                    ______            _          
   /  |/  / ____/ |     / /________ _____  ____  ___  _____   / ____/___  ____ _(_)___  ___ 
  / /|_/ / /    | | /| / / ___/ __ `/ __ \/ __ \/ _ \/ ___/  / __/ / __ \/ __ `/ / __ \/ _ \
 / /  / / /___  | |/ |/ / /  / /_/ / /_/ / /_/ /  __/ /     / /___/ / / / /_/ / / / / /  __/
/_/  /_/\____/  |__/|__/_/   \__,_/ .___/ .___/\___/_/     /_____/_/ /_/\__, /_/_/ /_/\___/ 
                                 /_/   /_/                             /____/               
```

This codeblock shows the script itself. If you do not have any  [BATCH](https://en.wikipedia.org/wiki/Batch_file "Batch file on Wikipedia")  knowledge you should
only edit the configurable variables between  ``REM ## START OF CONFIGURABLE VARIABLES``  and  ``REM ## END OF CONFIGURABLE VARIABLES`` . Otherwise, yo uare free to edit and
redistribute my work, but only under the  [Mozilla Public License](https://en.wikipedia.org/wiki/Mozilla_Public_License "Mozilla Public License on Wikipedia").
```batch
@echo off
REM ## START OF CONFIGURABLE VARIABLES - DO NOT LEAVE ANY OF THEM EMPTY
SET APPNAME=MCWrapper
SET SERVERNAME=YourServerName
SET SYSTEMBITS=x64
SET MAXRAM=1G
SET CORECOUNT=8
SET JARFILENAME=custom.jar
SET SERVERVERSION=1.15.2
REM ## END OF CONFIGURABLE VARIABLES

REM ## MAIN SCRIPT PESUDOCODE
title %APPNAME% // %SERVERNAME%
cls
IF EXIST banner.cfg (type banner.cfg && echo[) ELSE (echo [ ! ] %APPNAME%'s banner file,banner.cfg, could not be found. Please, create one.)
echo [ + ] Starting initialization of %APPNAME%'s engine!     
echo [ + ] %APPNAME% started in 100ms.
:INPUTAWAIT
set /p INPUT=:::: 
IF "%INPUT%"=="/launch" GOTO :SRV_LAUNCH 
IF "%INPUT%"=="/jarupdate" GOTO :JARUPDATE 
IF "%INPUT%"=="/exit" GOTO :EXIT 
IF "%INPUT%"=="/logclean" GOTO :LOGCLEAN 
IF "%INPUT%"=="/logclear" GOTO :LOGCLEAN 
IF "%INPUT%"=="/clearlogs" GOTO :LOGCLEAN 
IF "%INPUT%"=="/clear" GOTO :CLEAR 
IF "%INPUT%"=="/credits" GOTO :CREDITS 
IF "%INPUT%"=="/help" GOTO :HELP 

echo [ ! ] Unknown command! For more information use /help.
GOTO INPUTAWAIT
exit

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 1 - Checking if jarfile is present
:SRV_LAUNCH
echo [ + ] Setting XMX to %MAXRAM%.
echo [ + ] Setting GCThreads to %CORECOUNT%.
echo [ + ] Launching server.
IF EXIST %JARFILENAME% GOTO :SRV_LAUNCHSEQ 
echo [ ! ] Server launch failed, as %JARFILENAME% is missing. You add it and try again.
GOTO INPUTAWAIT

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 2 - Check system bits
:SRV_LAUNCHSEQ
IF "%SYSTEMBITS%"=="x64" GOTO :64BIT_LAUNCH 
IF "%SYSTEMBITS%"=="x86" GOTO :32BIT_LAUNCH 

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 3 - x64 variant
:64BIT_LAUNCH 
echo [ + ] Server staring in 64bit mode.
java -d64 -Xmx%MAXRAM% -XX:+IgnoreUnrecognizedVMOptions -XX:+IdleTuningGcOnIdle -Xjit:disableGuardedStaticFinalFieldFolding -Xtune:virtualized -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch  -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=8 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=true -Daikars.new.flags=true -jar %JARFILENAME% nogui
echo[
echo [ - ] Server has shut down. You can close this window by typing /exit, or start the server again by typing /launch.
GOTO INPUTAWAIT

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 3  - x32 variant
:32BIT_LAUNCH 
echo [ + ] Server staring in 32bit mode.
java -Xmx%MAXRAM% -XX:+IgnoreUnrecognizedVMOptions -XX:+IdleTuningGcOnIdle -Xjit:disableGuardedStaticFinalFieldFolding -Xtune:virtualized -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch  -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=8 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=true -Daikars.new.flags=true -jar %JARFILENAME% nogui
echo[
echo [ - ] Server has shut down. You can close this window by typing /exit, or start the server again by typing /launch.
GOTO INPUTAWAIT

REM ## JARUPDATE
:JARUPDATE
curl https://papermc.io/api/v1/paper/%SERVERVERSION%/latest/download > custom.jar
GOTO INPUTAWAIT

REM ## EXIT
:EXIT
exit

REM ## LOGCLEAN
:LOGCLEAN
IF EXIST logs (rmdir /Q /S logs) ELSE (echo [ - ] There are no logs to clear.)
GOTO INPUTAWAIT

REM ## CLEAR
:CLEAR
cls
GOTO INPUTAWAIT

REM ## CREDITS
:CREDITS 
echo %APPNAME% v2.0-MERCURY by CalifaxYT
echo ~ WEBSITE: https://califax.host
echo ~ DISCORD: Califax#3071
echo ~ Long live the internet.
GOTO INPUTAWAIT

REM ## HELP
:HELP
echo %APPNAME% v2.0-MERCURY by CalifaxYT
echo ~ This is very simple and powerful standalone tool
echo ~ for atomization and improvement of your server management.
echo[
echo Existing commands:
echo  '/launch' - start the server
echo  '/jarupdate' - update or download the server jarfile
echo  '/exit' - gracefully shut the script down
echo  '/logclean or /logclear or /clearlogs' - clear the logs
echo  '/clear' - clear the console window
echo  '/credits' - info about the script author
echo  '/help' - this page
GOTO INPUTAWAIT
```

**The MC:Bedrock Edition Startup Script**

This edition of the script allows server owners to start  **ANY**  server .exe. It *CAN NOT* auto-update (and redownload) the latest server build.
The list of available commands is:

/launch - start the server
/exit - gracefully shut the script down
/logclean or /logclear or /clearlogs - clear the logs
/clear - clear the console window
/credits - info about the script author
/help - shows a list of these commands

And now - onto the scipt:

This codeblock shows an example app banner. This can be freely changed to anything you like.
```markdown
#Example banner.cfg
    __  __________       __                                    ______            _          
   /  |/  / ____/ |     / /________ _____  ____  ___  _____   / ____/___  ____ _(_)___  ___ 
  / /|_/ / /    | | /| / / ___/ __ `/ __ \/ __ \/ _ \/ ___/  / __/ / __ \/ __ `/ / __ \/ _ \
 / /  / / /___  | |/ |/ / /  / /_/ / /_/ / /_/ /  __/ /     / /___/ / / / /_/ / / / / /  __/
/_/  /_/\____/  |__/|__/_/   \__,_/ .___/ .___/\___/_/     /_____/_/ /_/\__, /_/_/ /_/\___/ 
                                 /_/   /_/                             /____/               
```

This codeblock shows the script itself. If you do not have any  [BATCH](https://en.wikipedia.org/wiki/Batch_file "Batch file on Wikipedia")  knowledge you should
only edit the configurable variables between  ``REM ## START OF CONFIGURABLE VARIABLES``  and  ``REM ## END OF CONFIGURABLE VARIABLES``. Otherwise, yo uare free to edit and
redistribute my work, but only under the  [Mozilla Public License](https://en.wikipedia.org/wiki/Mozilla_Public_License "Mozilla Public License on Wikipedia").
```batch
@echo off
REM ## START OF CONFIGURABLE VARIABLES - DO NOT LEAVE ANY OF THEM EMPTY
SET APPNAME=MCWrapper
SET SERVERNAME=MagmaCraft
SET EXEFILENAME=bedrock_server.exe
REM ## END OF CONFIGURABLE VARIABLES

REM ## MAIN SCRIPT PESUDOCODE
title %APPNAME% :::: %SERVERNAME%
cls
IF EXIST banner.cfg (type banner.cfg && echo[) ELSE (echo [ ! ] %APPNAME%'s banner file,banner.cfg, could not be found. Please, create one.)
echo [ + ] Starting initialization of %APPNAME%'s engine!     
echo [ + ] %APPNAME% started in 100ms.
:INPUTAWAIT
set /p INPUT=:::: 
IF "%INPUT%"=="/launch" GOTO :SRV_LAUNCH 
IF "%INPUT%"=="/exit" GOTO :EXIT 
IF "%INPUT%"=="/logclean" GOTO :LOGCLEAN 
IF "%INPUT%"=="/logclear" GOTO :LOGCLEAN 
IF "%INPUT%"=="/clearlogs" GOTO :LOGCLEAN 
IF "%INPUT%"=="/clear" GOTO :CLEAR 
IF "%INPUT%"=="/credits" GOTO :CREDITS 
IF "%INPUT%"=="/help" GOTO :HELP 

echo [ ! ] Unknown command! For more information use /help.
GOTO INPUTAWAIT
exit

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 1 - Checking if jarfile is present
:SRV_LAUNCH
echo [ + ] Launching server.
IF EXIST %EXEFILENAME% GOTO :SRV_LAUNCHSEQ 
echo [ ! ] Server launch failed, as %EXEFILENAME% is missing. You add it and try again.
GOTO INPUTAWAIT

REM ## SERVER LAUNCH & SHUTDOWN SEQUENCE PART 2 - Run the .EXE
:SRV_LAUNCHSEQ
echo [ + ] Server will start in a moment...
%EXEFILENAME%
echo[
echo [ - ] Server has shut down. You can close this window by typing /exit, or start the server again by typing /launch.
GOTO INPUTAWAIT


REM ## EXIT
:EXIT
exit

REM ## LOGCLEAN
:LOGCLEAN
IF EXIST internalStorage (rmdir /Q /S internalStorage) ELSE (echo [ - ] There are no logs to clear.)
GOTO INPUTAWAIT

REM ## CLEAR
:CLEAR
cls
GOTO INPUTAWAIT

REM ## CREDITS
:CREDITS 
echo %APPNAME% v2.0-MERCURY by CalifaxYT
echo ~ WEBSITE: https://califax.host
echo ~ DISCORD: Califax#3071
echo ~ Long live the internet.
GOTO INPUTAWAIT

REM ## HELP
:HELP
echo %APPNAME% v2.0-MERCURY by CalifaxYT
echo ~ This is very simple and powerful standalone tool
echo ~ for atomization and improvement of your server management.
echo[
echo Existing commands:
echo  '/launch' - start the server
echo  '/exit' - gracefully shut the script down
echo  '/logclean or /logclear or /clearlogs' - clear the logs
echo  '/clear' - clear the console window
echo  '/credits' - info about the script author
echo  '/help' - this page
GOTO INPUTAWAIT
```

These scripts will be updated from time to time. Thanks for using them.


```

[CalifaxYT Discord](https://discord.gg/WufbYbv "Discord Invite URL")

---

CalifaxYT - [https://twitter.com/ytcalifax](https://twitter.com/ytcalifax "Twitter Profile")
