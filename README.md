# Powershell-CheatSheet

```
**Used for CTF
```
File Hunting in web directories (or user but not as administrator):

```Powershell
$last=""; Get-ChildItem -Path (Get-Location) -Recurse -Force -File -ErrorAction SilentlyContinue | Where-Object { $_.Attributes -match "Hidden" -or ($_.FullName -notmatch '\\(examples|vendor|bin|src|node_modules|test|Crpto|Local|USOPrivate|LocalLow|Vmware|tests|UEV|MF|IdentityCRL|docs|documentation|tmp|templogs|USOshared|libraries|themes|properties|plugins|Windows\sDefender|Windows\sSecurity|Health|NetFramework|DiagnosticLogCSP|device|storage|Package\sCache|Start\sMenu|classes|php|pear|web-inf|host-manager|doc\.svn|\.hg)\\' -and $_.Name -notmatch '^(Thumbs\.db|desktop\.ini|\.DS_Store)$' -and $_.Extension -notmatch '(\.log|\.tmp|\.bak|\.dll|\.pm|\.pod|\.twig|\.java|\.class|\.properties|\.html|\.jsp|\.gif|\.png|\.svg|\.woff|\.jar|\.xsd|\.mo|\.js|\.xsl|\.css|\.packlist|\.pl|\.ico|\.scss|\.h)$') } | Sort-Object DirectoryName | ForEach-Object { if ($last -ne $_.DirectoryName) { "`n===================== [ $($_.DirectoryName) ] ====================="; $last=$_.DirectoryName }; " - $($_.Name) [$([math]::Round($_.Length/1KB,2)) KB]" }
```

### **File Hunting in user directory**

```Powershell
Get-ChildItem -Path . -Recurse -Force -File -ErrorAction SilentlyContinue | Where-Object { $_.FullName -notmatch '\\(AppData\\Local|AppData\\LocalLow|Local Settings|Start Menu|Application Data)(\\|$)' }
```

### **History hunting 

```Powershell
(Get-PSReadlineOption).HistorySavePath
```

```Powershell
Get-ChildItem -Path 'C:\Users' -Recurse -Force -Filter "ConsoleHost_history.txt" -ErrorAction SilentlyContinue | ForEach-Object { "`n==== $($_.FullName) ===="; Get-Content $_.FullName -ErrorAction SilentlyContinue }
```

##  **look for deleted files in Recycle Bin

```powershell 
Get-ChildItem 'C:\$Recycle.Bin' -Directory -Force -ErrorAction SilentlyContinue | % { $sid=$_.Name; Get-ChildItem $_.FullName -Force -Filter '$I*' -ErrorAction SilentlyContinue | % { try { $fs=[System.IO.File]::Open($_.FullName,'Open','Read'); $br=New-Object System.IO.BinaryReader($fs); $br.BaseStream.Seek(0x14,'Begin')|Out-Null; $len=$br.ReadInt32(); $name=[System.Text.Encoding]::Unicode.GetString($br.ReadBytes($len*2)); Write-Output "$sid >> $name"; $br.Close(); $fs.Close() } catch {} } }
```

##  **Restore files to from recycle bin to desktop

```powershell
$r=(New-Object -ComObject Shell.Application).Namespace(0xA);$i=$r.Items()|?{$_.Name -eq "<name>"};$d=(New-Object -ComObject Shell.Application).Namespace([Environment]::GetFolderPath("Desktop"));if($i){$d.MoveHere($i);"Restored wapt-backup-sunday.7z to Desktop."}else{"File not found in Recycle Bin."}
```
##  **Installed Apps


``` Powershell
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher, InstallDate

Get-ItemProperty HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher, InstallDate
```
