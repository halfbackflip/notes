# Powershell

## Definition

"PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework. PowerShell runs on Windows, Linux, and macOS." - [Microsoft Overlords](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.2)

## Execution policy

The execution policy is a security setting; a machine-wide setting which determines the scripts that PowerShell will execute. 
The command 'Get-ExecutionPolicy' will list the current execution policy. 

Policy types

| **Policy** | **Description** |
| --------------|-------------------|
| Restricted | Default. Scripts are not executed |
| AllSigned | Powershell will execute anything which has been signed by a CA |
| Unrestricted | All scripts will run |
| RemoteSigned | PowerShell will execute any local script and will execute remote scripts if they’ve been digitally signed by a trusted CA |
| Bypass | Ignores the conifgured execution policy. Special setting used primarily by app developers embedding powershell within an app. |

## Powershell Providers
A powershell provider is similar to an adapter. It makes a form of data storage on your system appear similar to a drive for manipulation. Each provider has different capabilities.
Providers provide access to objects that would normally not be easily accessible via the command line. This data is presented similar to a file system drive. 
[https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-7.2)

## Objects
An object is a structured collection of information with certain properties, methods, aliases, etc. 
To get the properties of a given object, run the Get-Member command, for example, `Get-ChildItem | Get-Member`
[https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-7.2)
___

## Powershell Commands

| **Command** | **Description** |
| --------------|-------------------|
|`$PSVersionTable`| Lists the current version of powershell |
|`Help *object*`| Search help doco for items with the word object |
|`Help Get-Item -Full`| Retrieve full help for Get-Item |
|`Help Get-Item -Examples`| Retrieve examples for Get-Item |
|`Help Get-Item -Online`| Check the online help documentation |
|`Help about_*`| List all about pages, i.e. internal documentation |
|`Help about_Functions*`| Read the about page for functions |
|`Update-Help`| Update the Help Doco to the latest |
|`Get-verb`| List approved verbs for useage in powershell |
|`Get-Alias -Definition "Get-Process"`| List the Aias, i.e. a shortened nickname for the get-process command |
|`New-Alias -Name "ntst" -Value netstat`| Create a new alias to run netats with the alias ntst |
|`Test-NetConnection google.com`| Test the network connection to google.com |
|`New-Item -ItemType Directory -Name MyFolder1`| Create a new folder |
|`Remove-Item MyFolder1`| Remove a folder |
|`Get-PSDrive`| Lists currently connected powershell providers |
|`Set-Location`| Same as linux cd |
|`Set-location -Path hkcu:`| Change the path to a PS Provider |
|`Get-ChildItem \| Where-Object Name -eq 'scripts'`| Search for a childitem with a name of "scripts |
|`Get-Process \| Export-Csv procs.csv`| Export processes to csv |
|`Get-Process pwsh \| Select-Object Threads`| Select only the thread objects from processes |
|`Get-Process \| ConvertTo-Json \| Out-File procs.json`| Export as JSON to show nested threats |
|`Get-Content ./procs.json \| ConvertFrom-Json`| Read the above JSON file |
|`Get-Process \| ConvertTo-Json \| Out-File procs.json`| Export as JSON to show nested threats |
|`Get-Process \| ConvertTo-Json \| Out-File procs.json`| Export as JSON to show nested threats |
|`Get-Process \| ConvertTo-Json \| Out-File procs.json`| Export as JSON to show nested threats |
|`Get-Command \| Export-Clixml .\cmd.xml`| Export Get-Command to clixml. Clixml is unique to PowerShell. Best if the output is being re-used by powershell. Has both import/export commands available. |
|`Get-Process \| Select-Object Name \| Out-File process.txt`| Using Out-file to generate a flat, text file or process names|
|`Compare-Object -Reference (Import-Clixml reference.xml) -Difference (Get-Process) -Property Name`| Compare two objects, a reference and difference with one another |
|`Get-Process -Name firefox \| Stop-Process -Confirm`| Add a confirm prompt before killing firefox procesds |
|`Get-Command -All -Type Alias \| Export-Csv -IncludeType -Delimiter'\|' alias_commands.csv`| Create a csv with a custom delimeter and include the type|
|`Get-Command \| Export-CSV services.CSV –NoClobber`| Export a csv and prevent overwriting an existing file |
|`Get-Module az -ListAvailable`| List all available modules |
|`Find-Module Az`| Look for modules with the name of Az |
|`Update-Module`| Updates a module |
|`Get-command -Module Microsoft.PowerShell.Archive`| Lists all the command for a given module |
|`Install-Module az -Scope CurrentUser`| Install a module for the current user |
|`Compress-Archive -Path .\chapter7.txt -DestinationPath .\chapter7.zip -Force -Verbose`| Force compress a file, the -Verbose switch shows an activity progress bar |
|`Get-Process \| Sort-Object -Property CPU -Descending`| Sort CPU usage, descending |
|`Get-Date \| Select-Object -Property DayofWeek `| Use the Get-Date module to display the day of the week |
|`Get-ChildItem \| Select-Object -Property Name,CreationTime`| Display the names and creation date of files in the current working directory |
|`Get-ChildItem \| Select-Object -Property Name,CreationTime,LastWriteTime \| Sort-Object LastWriteTime \| Export-Csv files.csv`| Export to csv the Name,CreationTime,LastWriteTime of files in the current working directory |
|`Get-ChildItem \| Select-Object -Property Name,CreationTime,LastWriteTime \| Sort-Object LastWriteTime \| Out-file files.html`| Export to html the Name,CreationTime,LastWriteTime of files in the current working directory |