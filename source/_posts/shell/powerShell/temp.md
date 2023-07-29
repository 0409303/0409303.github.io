


ExecutionPolicy


Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope MachinePolicy
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope UserPolicy
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope LocalMachine


Get-ExecutionPolicy -List


Set-ExecutionPolicy Bypass -Scope Process -Force; Copy-Item \\cosbj-01\public\CosmosPowerShell\builds\install.ps1 $HOME\Downloads\install.ps1; & $HOME\Downloads\install.ps1 -Force
