---
layout: post
title: Batch script to install Generic Volume License Keys during Specialize phase of Sysprep.
date: 2016-08-09
category: ScriptSamples
---

Recently I had to prepare VMs with different operating systems for sysrep. I wanted to install the KMS client keys on the VM after sysprep was done and the VM was being Specialized. So I wrote a script and called it in the Specialization pass of the answer file. 

Below is the script in case someone else needs it.
Currently I don’t do specific scans for Education and the “N” edition, but its easy to put that in – by adding additional findstr clauses. Hope this helps someone else

{% highlight bash %}
    @ECHO off
    setlocal enableextensions
    SET me=%~n0
    SET parent=%~dp0
    SET GLOBAL_PRPLOG=C:\logfolder\GlobalActions.log
    SET PROD_KEY=NOT_FOUND
    SET CASECHOICE=CASE_DEFAULT
    SET OS=X
    
    REM read version x.y.z.p
    for /f "usebackq tokens=*" %%n in (`WMIC.exe os get Version ^| findstr /v /r "^$"`) do (
    SET OSVersion=%%n )
    
    REM read Description : Ex: Windows 10 Professional N
    for /f "usebackq tokens=*" %%n in (`WMIC.exe os get Caption ^| findstr /v /r "^$"`) do (
    SET OSName=%%n )
    
    REM read type... 1=> workstation, 2=> domain controller , 3=> server
    for /f "usebackq tokens=*" %%n in (`WMIC.exe os get ProductType ^| findstr /v /r "^$"`) do (
    SET OSType=%%n )
    
    REM read architecture. Can be 32-bit or 64-bit
    for /f "usebackq tokens=*" %%n in (`WMIC.exe os get OSArchitecture ^| findstr /v /r "^$"`) do (
    SET OSArch=%%n )
    
    CALL :WriteLog %me%: Detected OSVersion:%OSVersion%; OSName:%OSName%; OSType=%OSType%; OSArch=%OSArch%
    
    
    REM ************************************************
    ECHO.%OSVersion% | findstr /C:"6.0" >nul
    IF %ERRORLEVEL% == 0 ( 
    IF %OSType% == 1 (
    SET OS=WVista
    ) ELSE ( 
    SET OS=WS2008
    ) 
    goto AppendSKU
    )
    
    ECHO.%OSVersion% | findstr /C:"6.1" >nul
    IF %ERRORLEVEL% == 0 (
    IF %OSType% == 1 (
    SET OS=W7
    ) ELSE ( 
    SET OS=WS2008R2
    ) 
    goto AppendSKU
    )
    
    ECHO.%OSVersion% | findstr /C:"6.2" >nul
    IF %ERRORLEVEL% == 0 ( 
    IF %OSType% == 1 (
    SET OS=W8
    ) ELSE ( 
    SET OS=WS2012
    ) 
    goto AppendSKU
    )
    
    ECHO.%OSVersion% | findstr /C:"6.3" >nul
    IF %ERRORLEVEL% == 0 (
    IF %OSType% == 1 (
    SET OS=W8.1
    ) ELSE ( 
    SET OS=WS2012R2
    ) 
    goto AppendSKU
    )
    
    ECHO.%OSVersion% | findstr /C:"10.0" >nul
    IF %ERRORLEVEL% == 0 (
    IF %OSType% == 1 (
    SET OS=W10
    ) ELSE ( 
    SET OS=WS2016
    ) 
    goto AppendSKU
    )
    
    ECHO.%OSVersion% | findstr /C:"5.2" >nul
    IF %ERRORLEVEL% == 0 (
    goto ExitCleanup
    )
    
    goto ExitCleanup
    
    REM ************************************************
    :AppendSKU
    ECHO.%OSName% | findstr /C:"Pro" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Pro
    )
    
    ECHO.%OSName% | findstr /C:"Enterprise" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Ent
    )
    
    ECHO.%OSName% | findstr /C:"Education" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Edu
    )
    
    ECHO.%OSName% | findstr /C:"Business" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Biz
    )
    
    ECHO.%OSName% | findstr /C:"Standard" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Std
    )
    
    ECHO.%OSName% | findstr /C:"Datacenter" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Dtc
    )
    
    ECHO.%OSName% | findstr /C:"Essential" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Ess
    )
    
    ECHO.%OSName% | findstr /C:"Web" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%Web
    )
    
    ECHO.%OSName% | findstr /C:"HPC" >nul
    IF %ERRORLEVEL% == 0 (
    SET CASECHOICE=CASE_%OS%HPC
    )
    
    2>NUL CALL :%CASECHOICE%
    goto ExitCleanup
    
    REM ************************************************
    :CASE_DEFAULT
    SET PROD_KEY=NOT_FOUND
    goto END_CASE
    :CASE_WS2016Dtc
    SET PROD_KEY=CB7KF-BWN84-R7R2Y-793K2-8XDDG
    goto END_CASE
    :CASE_WS2016Std 
    SET PROD_KEY=WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY
    goto END_CASE
    :CASE_WS2016Ess
    SET PROD_KEY=JCKRF-N37P4-C2D82-9YXRT-4M63B
    goto END_CASE
    :CASE_W10Pro
    SET PROD_KEY=W269N-WFGWX-YVC9B-4J6C9-T83GX
    goto END_CASE
    :CASE_W10ProN
    SET PROD_KEY=MH37W-N47XK-V7XM9-C7227-GCQG9
    goto END_CASE
    :CASE_W10Ent
    SET PROD_KEY=NPPR9-FWDCX-D2C8J-H872K-2YT43
    goto END_CASE
    :CASE_W10EntN
    SET PROD_KEY=DPH2V-TTNVB-4X9Q3-TJR4H-KHJW4
    goto END_CASE
    :CASE_W10Edu
    SET PROD_KEY=NW6C2-QMPVW-D7KKK-3GKT6-VCFB2
    goto END_CASE
    :CASE_W10EduN 
    SET PROD_KEY=2WH4N-8QGBV-H22JP-CT43Q-MDWWJ
    goto END_CASE
    :CASE_W10Ent2015LTSB
    SET PROD_KEY=WNMTR-4C88C-JK8YV-HQ7T2-76DF9
    goto END_CASE
    :CASE_W10Ent2015LTSBN
    SET PROD_KEY=2F77B-TNFGY-69QQF-B8YKP-D69TJ
    goto END_CASE
    :CASE_W10Ent2016LTSB
    SET PROD_KEY=DCPHK-NFMTC-H88MJ-PFHPY-QJ4BJ
    goto END_CASE
    :CASE_W10Ent2016LTSBN
    SET PROD_KEY=QFFDN-GRT3P-VKWWX-X7T3R-8B639
    goto END_CASE
    :CASE_W8.1Pro
    SET PROD_KEY=GCRJD-8NW9H-F2CDX-CCM8D-9D6T9
    goto END_CASE
    :CASE_W8.1ProN
    SET PROD_KEY=HMCNV-VVBFX-7HMBH-CTY9B-B4FXY
    goto END_CASE
    :CASE_W8.1Ent
    SET PROD_KEY=MHF9N-XY6XB-WVXMC-BTDCT-MKKG7
    goto END_CASE
    :CASE_W8.1EntN
    SET PROD_KEY=TT4HM-HN7YT-62K67-RGRQJ-JFFXW
    goto END_CASE
    :CASE_WS2012R2Std
    SET PROD_KEY=D2N9P-3P6X9-2R39C-7RTCD-MDVJX
    goto END_CASE
    :CASE_WS2012R2Dtc
    SET PROD_KEY=W3GGN-FT8W3-Y4M27-J84CP-Q3VJ9
    goto END_CASE
    :CASE_WS2012R2Ess
    SET PROD_KEY=KNC87-3J2TX-XB4WP-VCPJV-M4FWM
    goto END_CASE
    :CASE_W8Pro
    SET PROD_KEY=NG4HW-VH26C-733KW-K6F98-J8CK4
    goto END_CASE
    :CASE_W8ProN
    SET PROD_KEY=XCVCF-2NXM9-723PB-MHCB7-2RYQQ
    goto END_CASE
    :CASE_W8Ent
    SET PROD_KEY=32JNW-9KQ84-P47T8-D8GGY-CWCK7
    goto END_CASE
    :CASE_W8EntN
    SET PROD_KEY=JMNMF-RHW7P-DMY6X-RF3DR-X2BQT
    goto END_CASE
    :CASE_WS2012 
    SET PROD_KEY=BN3D2-R7TKB-3YPBD-8DRP2-27GG4
    goto END_CASE
    :CASE_WS2012N
    SET PROD_KEY=8N2M2-HWPGY-7PGT9-HGDD8-GVGGY
    goto END_CASE
    :CASE_WS2012SingleLanguage
    SET PROD_KEY=2WN2H-YGCQR-KFX6K-CD6TF-84YXQ
    goto END_CASE
    :CASE_WS2012CountrySpecIFic
    SET PROD_KEY=4K36P-JN4VD-GDC6V-KDT89-DYFKP
    goto END_CASE
    :CASE_WS2012ServerStd
    SET PROD_KEY=XC9B7-NBPP2-83J2H-RHMBY-92BT4
    goto END_CASE
    :CASE_WS2012MultiPointStd
    SET PROD_KEY=HM7DN-YVMH3-46JC3-XYTG7-CYQJJ
    goto END_CASE
    :CASE_WS2012MultiPointPremium
    SET PROD_KEY=XNH6W-2V9GX-RGJ4K-Y8X6F-QGJ2G
    goto END_CASE
    :CASE_WS2012Dtc
    SET PROD_KEY=48HP8-DN98B-MYWDG-T2DCC-8W83P
    goto END_CASE
    :CASE_W7Pro 
    SET PROD_KEY=FJ82H-XT6CR-J8D7P-XQJJ2-GPDD4
    goto END_CASE
    :CASE_W7ProN
    SET PROD_KEY=MRPKT-YTG23-K7D7T-X2JMM-QY7MG
    goto END_CASE
    :CASE_W7ProE
    SET PROD_KEY=W82YF-2Q76Y-63HXB-FGJG9-GF7QX
    goto END_CASE
    :CASE_W7Ent
    SET PROD_KEY=33PXH-7Y6KF-2VJC9-XBBR8-HVTHH
    goto END_CASE
    :CASE_W7EntN
    SET PROD_KEY=YDRBP-3D83W-TY26F-D46B2-XCKRJ
    goto END_CASE
    :CASE_W7EntE
    SET PROD_KEY=C29WB-22CC8-VJ326-GHFJW-H9DH4
    goto END_CASE
    :CASE_WS2008R2Web
    SET PROD_KEY=6TPJF-RBVHG-WBW2R-86QPH-6RTM4
    goto END_CASE
    :CASE_WS2008R2HPC
    SET PROD_KEY=TT8MH-CG224-D3D7Q-498W2-9QCTX
    goto END_CASE
    :CASE_WS2008R2Std
    SET PROD_KEY=YC6KT-GKW9T-YTKYR-T4X34-R7VHC
    goto END_CASE
    :CASE_WS2008R2Ent
    SET PROD_KEY=489J6-VHDMP-X63PK-3K798-CPX3Y
    goto END_CASE
    :CASE_WS2008R2Dtc
    SET PROD_KEY=74YFP-3QFB3-KQT8W-PMXWJ-7M648
    goto END_CASE
    :CASE_WVistaBiz
    SET PROD_KEY=YFKBB-PQJJV-G996G-VWGXY-2V3X8
    goto END_CASE
    :CASE_WVistaBizN
    SET PROD_KEY=HMBQG-8H2RH-C77VX-27R82-VMQBT
    goto END_CASE
    :CASE_WVistaEnt
    SET PROD_KEY=VKK3X-68KWM-X2YGT-QR4M6-4BWMV
    goto END_CASE
    :CASE_WVistaEntN
    SET PROD_KEY=VTC42-BM838-43QHV-84HX6-XJXKV
    goto END_CASE
    :CASE_WS2008Web
    SET PROD_KEY=WYR28-R7TFJ-3X2YQ-YCY4H-M249D
    goto END_CASE
    :CASE_WS2008Std
    SET PROD_KEY=TM24T-X9RMF-VWXK6-X8JC9-BFGM2
    goto END_CASE
    :CASE_WS2008StdwithoutHyper-V
    SET PROD_KEY=W7VD6-7JFBR-RX26B-YKQ3Y-6FFFJ
    goto END_CASE
    :CASE_WS2008Ent
    SET PROD_KEY=YQGMW-MPWTJ-34KDK-48M3W-X4Q6V
    goto END_CASE
    :CASE_W2008EntwithoutHyper-V
    SET PROD_KEY=39BXF-X8Q23-P2WWT-38T2F-G3FPG
    goto END_CASE
    :CASE_WS2008HPC
    SET PROD_KEY=RCTX3-KWVHP-BR6TB-RB6DM-6X7HP
    goto END_CASE
    :CASE_WS2008Dtc
    SET PROD_KEY=7M67G-PC374-GR742-YH8V4-TCBY3
    goto END_CASE
    :CASE_WS2008DtcwithoutHyper-V
    SET PROD_KEY=22XQ2-VRXRG-P8D42-K34TD-G3QQC
    goto END_CASE
    :END_CASE 
    GOTO :EOF
    
    :ExitCleanup 
    IF /I "%PROD_KEY%" == "NOT_FOUND" (
    CALL :WriteLog %me%: ProductKey:%PROD_KEY%
    SET ERRORLEVEL=2
    ) ELSE (
        CALL :WriteLog %me%: Installing ProductKey:%PROD_KEY%
        cscript.exe //B "C:WindowsSystem32slmgr.vbs" -ipk %PROD_KEY%
        IF %ERRORLEVEL% == 0 (
        CALL :WriteLog %me%: successfully installed key
        ) ELSE (
        CALL :WriteLog %me%: Failed to install key : %ERRORLEVEL%
        )
    )
    endlocal & exit /b %ERRORLEVEL%
    
    :WriteLog
    ECHO %* >> "%GLOBAL_PRPLOG%"
    ECHO %* 
    EXIT /B 0
{% endhighlight %}