FROM mcr.microsoft.com/windows/servercore:ltsc2022-amd64

# Download Links:
ENV sql_express_download_url "https://download.microsoft.com/download/5/1/4/5145fe04-4d30-4b85-b0d1-39533663a2f1/SQL2022-SSEI-Expr.exe"

ENV sa_password="_" \
    attach_dbs="[]" \
    ACCEPT_EULA="_" \
    sa_password_path="C:\ProgramData\Docker\secrets\sa-password"

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]


COPY start.ps1 /

WORKDIR /


RUN Invoke-WebRequest -Uri $env:sql_express_download_url -OutFile sqlexpress.exe
RUN Start-Process -Wait -FilePath .\sqlexpress.exe -ArgumentList /MEDIATYPE=Core, /Action=Download, /Q, /Language=en-US, /HIDEPROGRESSBAR, /MEDIAPATH="C:\installer";
RUN Start-Process -Wait -FilePath .\installer\SQLEXPR_x64_ENU.exe -ArgumentList /qs, /x:setup ;
RUN .\setup\setup.exe /q /ACTION=Install /INSTANCENAME=SQLEXPRESS /FEATURES=SQLEngine /UPDATEENABLED=0 /SQLSVCACCOUNT='NT AUTHORITY\System' /SQLSYSADMINACCOUNTS='BUILTIN\ADMINISTRATORS' /TCPENABLED=1 /NPENABLED=0 /IACCEPTSQLSERVERLICENSETERMS ;

RUN Remove-Item -Recurse -Force sqlexpress.exe, setup

RUN stop-service MSSQL`$SQLEXPRESS ; \
    set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql16.SQLEXPRESS\mssqlserver\supersocketnetlib\tcp\ipall' -name tcpdynamicports -value '' ; \
    set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql16.SQLEXPRESS\mssqlserver\supersocketnetlib\tcp\ipall' -name tcpport -value 1433 ; \
    set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql16.SQLEXPRESS\mssqlserver\' -name LoginMode -value 2 ;

CMD .\start -sa_password $env:sa_password -ACCEPT_EULA $env:ACCEPT_EULA -attach_dbs \"$env:attach_dbs\" -Verbose