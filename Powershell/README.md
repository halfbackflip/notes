# Powershell

## Definition

"PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework. PowerShell runs on Windows, Linux, and macOS." - [Microsoft Overlords](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.2)

## Execution policy

The execution policy is a security setting; a machine-wide setting which determines the scripts that PowerShell will execute. 
The command `Get-ExecutionPolicy` will list the current execution policy. 

Policy types

| **Policy** | **Description** |
| --------------|-------------------|
| Restricted | Default. Scripts are not executed |
| AllSigned | Powershell will execute anything signed by a CA |
| Unrestricted | All scripts will run |
| RemoteSigned | PowerShell will execute any local script and will execute remote scripts if they’ve been digitally signed by a trusted CA |
| Bypass | Ignores the configured execution policy. Special settings used primarily by app developers embedding powershell within an app. |

## Powershell Providers
A powershell provider is similar to an adapter. It makes a form of data storage on your system appear similar to a drive for manipulation. Each provider has different capabilities.
Providers allow access to objects that would normally not be easily accessible via the command line, presenting it similar to a file system drive. 
[https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-7.2)

## Objects
An object is a structured collection of information with certain properties, methods, aliases, etc. 
To get the properties of a given object, run the Get-Member command, for example, `Get-ChildItem | Get-Member`
[https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-7.2)

## Pipeline Parameter Binding
The process by which powershell will determine which parameter of a command will accept the output of a preceding command in a pipeline.
There are two types of pipeline parameter binding:

1. ByValue: Attempts to see if any paramter of the second command can accept the incoming object as input. The first method powershell tries. 
2. ByPropertyName: Powershell looks for property names which match parameter names. This is the second method powershell will try if ByValue fails.

[https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.2)

## Parenthetical Commands
Powershell runs any command in ()'s first. This will work as long as the parathetical command generates the type of object the preceding parameter expects.

For example, if you have a text file with several module names inside it:
```
Get-Command -Module (Get-Content ./modules.txt)
```

## Formatting 
Powershell has several formatting options available.
| **Name** | **Description** | **Example Command** |
| --------------|-------------------|-------------------| 
| Format-Table | Creates a table-like output. Use properties to select column headers. | Get-Process \| Format-Table -Property ID,Name,Responding | 
| Format-List | Generates series of horizontal rows. Use properties to select included rows. | Get-Process \| Format-List -Property ID,Name,Responding | 
| Format-Wide | Displays a wider, multicolum list. Can only display the values of a single property. The numbe of columns to display can be selected. | Get-Process \| Format-Wide name -col 4 | 
| Out-GridView | Generates a GUI viewer. | Get-Process \| Out-GridView |

## Filtering
When filtering command output, always filter early, as close to the beginning command as possible. 

Comparrison operators are often used with a Where-Object expression following a pipeline. 
For example:
```
 Get-Process | Where-Object {$_.CPU -gt 50 -and $_.ProcessName -like 'Code'}
```

List of comparrison operators:
| **Name** | **Description** |
| --------------|-------------------|
| -eq | equal |
| -ne | not equal |
| -ge | greater than or equal to |
| -le | less than or equal to |
| -gt | greater than |
| -lt | less than |
| -eq | equal |
| -or | logical or |
| -and | logical and |
| -and | logical not |
| -like | checks if two expressions are similar |
| -like | checks if two expressions are not similar |
| -match | compares a string with a regex |

* The command `get-help about_Comparison_Operators` will display the helpfile for all comparrison operators.
** String comparisons, you can use a case-sensitive operators such as: -ceq, -cne, -cgt, -clt, -cge, -cle.

## Powershell Remoting

Method to run powershell commands on remote computers. 

Powershell on windows uses a communications protocol called Web Services for Management (WSMan). WSMan runs entirely over HTTP or HTTPS (HTTP by default). 
WsMan exists as a background service which is disabled by default, Windows Remote Management (WinRM). 
Remoting on Linux and MacOS devices can be performed over Secure Shell, SSH. 

Points to keep in mind:
- In a remoting session, Powershell serializes the output objects into XML
- Then deserializes them back into XML on the recieving endpoint
- The recieved XML represents point-in-time snapshots, not actual live data

### Remoting & Variables
Sessions can be created and stored in variables to ensure they are immediately accessible.
```
$db_server = New-PSSession -ComputerName db01 -Credential DbAdmin
```
Storing sessions in variables also allows several computers to be stored in a single variable
```
$db_servers = New-PSSession -ComputerName db01, db02, db03 -Credential DbAdmin
```
Multiple sessions can then be closed using a single command
```
$db_servers = Remove-PSSession
```
When using mulitple variables to store sessions, individual sessions can be accessed by using index numbers.
```
Enter-PSSession -session $db_servers[1]
```
Multiple sessions can also be retrieved by chaining commands together
```
Get-PSSession -ComputerName db01 | Enter-PSSession
```

### Invoke-Command with Multiple Sessions Objects
Using `Invoke-command` along with mutliple session objects we target multiple computers
at once. 
```
Invoke-Command -ComputerName { Get-Process } -Session (Get-PSSession -ComputerName db01, db02, db03)
```

## Jobs

### Background Jobs
Background jobs allow you to run commands without interrupting or interacting with the current shell session. 
These jobs run in a completely different powershell process.
The output of background jobs can be retrieved once they complete running. 
Once a job has been retrieved, it will be removed from the job cache unless the `-keep` parameter is specified.

### Thread Jobs
Creates a thread job. Instead of creating a seperate powershell process like background jobs, a thread job spins up another thread within the same process.
Thread jobs are fast to start and are useful for short-term scripts or commands. However, powershell has a "throttle limit" of 10 threads at a time. 
This limit can be increased by adjusting the `-ThrottleLimit` parameter. Increasing the throttle limit beyond a certain point may result in diminishing returns. 

### -AsJob
Many modern commands may offer the `-AsJob` parameter. This parameter creast a job for the specified command. 

## Powershell CIM 

CIM refers to "Common Information Model." CIM is designed to describe the structure and behavior of common managed resources such as storage, network, or software components. Before CIM there were different cmdlets to manage windows and non-windows. WMI cmdlets were used to manage Windows, while
WsMAN cmdlets were aimed at non-windows implementations. CIM cmdlets were designed for windows or non-windows, allowing powershell to adapt to a cloud-focused, vendor neutral environment.

Get-CimInstance is often used in combination with Invoke-CimMethod in a pipeline to execute CIM methods. For example, this will enable DHCP on all intel adapters.
```
Get-CimInstance -ClassName Win32_NetworkAdapterConfiguration -filter
"description like '%intel%'" | Invoke-CimMethod -MethodName EnableDHCP
```


| **Cmdlet** | **Description** |
| --------------|-------------------|
| Get-CimAssociatedInstance | Gets all the associated instances for a particular instance. |
| Get-CimClass | Gets class schema of a CIM class. |
| Get-CimInstance | Gets instances of a class. |
| Get-CimSession | Gets a list of CIM Sessions that have been made. |
| Invoke-CimMethod | Invokes instance or static method of a class.|
| New-CimInstance | Creates a new instance of a class. |
| New-CimSession | Creates a CIM Session with local or a remote machine |
| New-CimSessionOption | Creates a set of options that can be used while creating a CIM session. |
| Register-CimIndicationEvent | Helps subscribe to events. |
| Remove-CimInstance | Removes one of more instances of a class. |
| Remove-CimSession | Removes CimSessions that are there on a machine. |
| Set-CimInstance | Modifies one or more instances of a class. |

[https://devblogs.microsoft.com/powershell/introduction-to-cim-cmdlets/](https://devblogs.microsoft.com/powershell/introduction-to-cim-cmdlets/)

## Run Commands in Parallel 
Running commands in parallel allows your scripts to execute faster than if you ran the commands sequentially. Powershell 7 introduces a new parameter, 
`ForEach-Object -Parallel` Parallel is a powerful parameter, but has limitations and only runs 5 script blocks in parallel by default. This limit cna be changed
by adjusting the `-ThrottleLimit` parameter. The following are examples showing the performance by using Measure-Command. 

Without Parallel, counting 1-20.
```
Measure-Command { 1..20 | ForEach-Object {Write-Output "$_"; Start-Sleep -Seconds 2}}
```

With Parallel, counting 1-20
```
Measure-Command { 1..20 | ForEach-Object -Parallel {Write-Output "$_"; Start-Sleep -Seconds 2}}
```

With Parallel and throttle limit to 10, counting 1-20.
```
Measure-Command { 1..20 | ForEach-Object -ThrottleLimit 10 -Parallel {Write-Output "$_"; Start-Sleep -Seconds 2}}
``` 

## Variables
All items in powershell are treated as an object and have corresponding methods which can be used on them. 
For example, `Get-Member` reveals the methods which can be used on a string or integer assigned to a variable:
```
$a = 1
$a | Get-Member

$a = "One"
$a | Get-Member
```
### Variable Conventions

There are no set variable best practices, but the key points are:
- Start with a letter or underscore
- Variable names can have spaces, but must be enclosed in {}, i.e. ${Space Var}
- Variables do not persist after closing a shell session
- Keep variable names succinct

### Variable Quotes

Single quotation marks make the string a literal string. 
For example:
```
$var = 'batman'
'I am $var'
```
Double quotes will result in the variable value being displayed.
For example:
```
$var = "batman"
"I am $var"
```
Back ticks before a character in double quotes are used as escape characters, removing any special meaning or value associate with a variable.
For example:
```
$var = "batman"
"I am `$var"
```
Back ticks can also add special meaning to a given letter. 
For example n creates a new line:
```
$var = "batman"
"I `nam `n$var"
```

### Collections of objects
Collections of objects, similar to a list a can be declared with the following:
```
$heroes = 'batman','superman','antman','hulk','wolverine','spiderman'
$heroes
````
Individual items in the array, can then be accessed via index:
```
$heroes = 'batman','superman','antman','hulk','wolverine','spiderman'
$heroes[0]
$heroes[3]
```
Replacing an item can be done by referencing the index:
```
$heroes = 'batman','superman','antman','hulk','wolverine','spiderman'
$heroes[2] = $heroes[2].replace('antman','thor')
$heroes
```
Commands can be assigned to variables. Properties of the variable can then be accessed via method.
```
$processes = Get-Process
$processes.Name
$processes.Id
```
A variable's type can be declared. Notice the difference in output when a variable's type is declared and it is not declared. 
```
$n = Read-Host "Enter a number"
$n * 10 

[int]$n = Read-Host "Enter a number"
$n * 10 
```
The following object types are supported in powershell.

- [int]—Integer numbers
- [single] and [double]—Single-precision and double floating point numbers.
- [string]—A string of characters
- [char]—Exactly one character 
- [xml]—An XML document; whatever string you assign to this will be parsed to make sure it contains valid XML markup.

To get the type of a varable, use GetTypeCode():
```
[int]$n = Read-Host "Enter a number"
$n.GetTypeCode()

[string]$n = Read-Host "Enter a number"
$n.GetTypeCode()
``` 

## Input and Output

The Read-Host cmdlet is desiged to display a text prompt and collect input from the user. 
```
$name = Read-Host "Name > "
$name
```
The Write-Host cmdlet can write output and print directly to the host app's screen. This allows for different foreground colors, styling and other effects.
This cmdlet is not designed for error messages, warnings or debugging messages. It does not send objects into the pipeline.
```
$name = Read-Host "Name > "
Write-Host $name -ForegroundColor blue
```
The write-output cmdlet can send objects into the pipeline. Since it does not write directly to the display, alternate colors cannot be specified. 
Once at the end of the pipelin, Write-Output will display the objects. Note the below example, where the output may not display if it is longer than 5 characters.
```
$name = Read-Host "Name > "
Write-Output $name | Where-Object { $_.length -gt 5}
```
### Aditional Write Commands
When the configuration variable is set to `Continue`, the commands below will produce output. 
If the configuration variable is set to `SilentlyContinue`, these commands will not produce output. 

| **Command** | **Description** | **Config Variable** |
| --------------|-------------------|-------------------|
| Write-Warning | Displays warning text in yellow preceded by WARNING | $WarningPreference (Continue by default) |
| Write-Verbose | Displays additional information text; preceded by the label VERBOSE | $VerbosePreference (SilentlyContinue by default) |
| Write-Debug | Displays debugging text preceded by DEBUG| $DebugPreference (SilentlyContinue by default) |
| Write-Error | Produces an error message |	$ErrorActionPreference (Continue by default) |
| Write-Information | Displays info messages that can be written to the information stream | $InformationPreference (SilentlyContinue by default) |

## Parameters
Creating parameters in powershell is done by setting a param() block before the script.
For example below the will specify 'target' as a parameter and '8.8.8.8' as the default parameter.

```
 param ( 
    $target = 8.8.8.8
    ) 
 Test-NetConnection $target
```

## Comment Based Help
Comment based help keywords surrounded by `<# #>` allow you to a add a "Get-Help" entry to a script. This designation must come at the start of the script, right before the comments section. The below is an example template:
```
<#
.SYNOPSIS A brief description of the function or script. This keyword can be used
only once in each topic.

.DESCRIPTION A detailed description of the function or script. This keyword can be used
only once in each topic.

.PARAMETER <Parameter-Name> A detailed description about any parameters in the script.

.EXAMPLE A sample command that uses the function or script. 

.INPUTS Types of objects that can be piped into the function or script

.OUTPUTS Types of objects returned.

.NOTES Additional info about the function/script.

.LINK Name of a related topic. Each link value must be preceded by a #.

.COMPONENT Technology or feature that the function or script uses, or
to which it is related. 

.ROLE Name of the user role for the help topic. 

.FUNCTIONALITY The keywords that describe the intended use of the function.

.FORWARDHELPTARGETNAME Redirects to the help topic for the specified command. 

.FORWARDHELPCATEGORY Specifies the help category of the item in .ForwardHelpTargetName. 

.EXTERNALHELP Specifies an XML-based help file for the script or function.

#>
```

## Scope
Scopes can be considered a container for powershell aliases, variables and functions. The following are types of scopes:

1. Global Scope - The top-level scope of the shell itself. Contains built-in variables, aliases, functions and PSDrives, along with your profile. 
2. Script Scope - Each script that is run executes its own script. Only elements in the script scope exists in the script. Global scopes elements are available too. 
3. Private Scope - Commands that are run as part of a function or loop within a script. Commands, parameters and variables only exist in this function. Items from the Global and Script scopes can be inherited. 


## Advanced Scripting
Advanced Scripting functionality can be enabled by add the `[CmdletBinding()]` directly after the comment-based help. For example:
```
<#
.SYNOPSIS foo bar
#>
[CmdletBinding()] 
```

Adding this line allows adding of parameter decorators. For example, the makes the target parameter mandatory, creates an alias of ip, sets a help message which prompts the user to enter an IP is also added and explicity sets the types as string. 

```
<#
.SYNOPSIS Test the network
#>
[CmdletBinding()] 
 param ( 
    [Parameter(Mandatory=$True HelpMessage="Enter a an IP to test: ")][Alias('ip')]  [string]$target = 8.8.8.8
    ) 
 Test-NetConnection $target
```

The [ValidateSet()] decorator can also be used to validate user input in the script. For example, `[ValidateSet(2,3,4)]` would require entering number 2-4. A full list of advanced help features can be viewed by running `help about_functions_advanced_parameters`

## Regular Expressions
Powershell includes an operation -match or -cmatch for case sensitive matches that work with regular expressions. 
For example the below will match an ip address pattern and return true:
```
"192.168.1.1" -match "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
```
The Select-String command can be used in the pipeline. This may be combined with commands such as Get-Content or Get-ChildItem. 
The below command will search all files in the current working directory for ip addresses. 
```
 Get-ChildItem | Select-String -pattern "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" | Format-Table Filename, LineNumber, Line -wrap
```
The following help documentation contains more information about about regular expressions in powershell. `help about_regular_expressions`

## Loops

### Foeach
Standard form of looping which allows you to iterate through a series of values in a collection of items. For example:
```
$array = 1..100
foreach($a in $array) {Write-Output $a}
```

### Foreach-Object
The Foreach-Object cmdlet performs an action defined in a script block on each item in the input collection of objects. 
As Foreach-Object requires an input of objects, it is most often called via the pipeline. 
For example this takes Get-Process as input and uses foreach to display the id. 
```
Get-Process | ForEach-Object {$_.Id}
```
Foreach-Object normally runs in sequence, however by adding the `-Parallel`operator it will run each action on the loop in parallel. The `-ThrottleLimit` operation also allows you to adjust the number of parallel blocks to run from the default of 10. Note the difference in performance. 
```
# Parallel
measure-command {1..10 | ForEach-Object -parallel {Write-Output $_; start-sleep -Seconds 2}}

# Normal
measure-command {1..10 | ForEach-Object {Write-Output $_; start-sleep -Seconds 2}}
```

### While
A while loop performs an iterating action until the condition to terminate is satisfied. This can include a boolean (true/false) statement or a command. For example, writing 1..10 using a while loop requires an -le, less than or equal to, comparrison operator. Once $n reaches 10, this condition is false and the loop terminates.
```
$n=1
While ($n -le 10){Write-Output $n; $n++}
```

### Do While
Similar to the while loop, however executes at least once -- regardless of wether the condition is true or not. The syntax is a reversal of the normal while loop. For example, this will write a '1' even though the condition is false. 
```
$n=1
do {
   Write-Output $n
} while ($n -ne 1)
```

## Handling Errors

The $Error automation variable holds any error objects which occurr in the current session. Error objects can be accessed by index, for example $Error[0]. 
By default errors will be placed into this variable, although this can be changed by setting the -ErrorAction parameter to ignore. 

The -ErrorVariable parameter allows you to specify a custom variable to send errors to. This will only hold the most recent error unless preceded by a '+' sign. 
For example:
```
Get-Process -ErrorVariable +errors
```
The about page, `get-help about_automatic_variables` contains more info about error variables.

## Errors and Exceptions

When powershell hits an error by default, it will keep going instead of stopping. 
For example, the service "Test" does not exist, but it will keep going. 
```
Get-Service Fax, PlugPlay, Test
```
If you set the error action variable of `$ErrorActionPreference`, you can manually define how powershell should handle this situation. 
Generally commands won’t generate an exception unless you run them with the Stop error action. Setting to Stop may be needed to handle specific errors.

| Value | Definition |
| -------------- |  -------------- |
| Break | Enter the debugger when an error occurrs or an exception is raised |
| Contine | (Default) Display the error message and keep on going |
| Ignore | Suppress the error message and continue to execute the command.  |
| Inquire | Display the error message and ask if you want to continue |
| SilentlyContinue | No effect. Error message isn't displayed. Execution continues without interruption.  |
| Stop | Displays the error message and stops executing.  |
| Suspend | Automatically suspends a workflow job to allow for further investigation |

Re-running the above command with a different -ErrorAction variable.
```
Get-Service Fax, PlugPlay, Test -ErrorAction Stop
Get-Service Fax, PlugPlay, Test -ErrorAction Ignore
```
Instead of changing the `$ErrorActionPreference` globally, it is best practice to specify a behavior on a per-command basis. 
Setting this command globally may result in the accidental suppression of important error messages to help build your script.

Try and catch can statements can also be used to temporarily modify $ErrorActionPreference for the duration of the a command or block to catch an exception. 
For example:
```
try {Get-Service Fax, PlugPlay, Test, WinDefend, SysMain -ErrorAction Stop}
catch {Write-Host "Nope"}
```

## VS Code Debugging
After creating a breakpoint by clicking on the gutter, VS Code includes the following options:

| **Command** | **Description** |
| --------------|-------------------|
|Resume | continue running the script |
|Step Over | Runs the current highlighted line and stops on the line right after |
|Restart | Restart the script from the beginning |
|Stop | Stops running the script and exits the debug experience |

## Custom Profiles
The hosting application, i.e. VS Code, the console or another text editor is responsible for loading and running profile scripts at startup.
Profile scripts can be used to customise the powershell environment by loading modules, changing to a different starting directory, etc.
Details about creating profiles can be viewed at `help about_profiles`. Some profile settings which tweak colors or fonts can cause errors in VS Code or Windows Terminal depending on the setting. The below are the powershell profile files powershell attempts to launch, in order:

1. $pshome\profile.ps1 - Executes for all users of the computer on any powershell host.
2. $pshome\Microsoft.PowerShell_profile.ps1 - Executes for all users if they are using the console host.
3. $pshome/Microsoft.VSCode_profile.ps1 - Executes for all users if they are using VSCode
4. $home\Documents\WindowsPowerShell\profile.ps1 - Executes only for the current user in the home dir, regardless of the shell being used.
5. $home\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 - Executes for the current user if using the console host.

If a given script doesn't exist, the hosting application will skip and move on to the next profile. 

## Custom Prompts
The powershell prompt, i.e. PS C:\> can be customised. The prompt is created by a built-in function called prompt. 
This function can be replaced in a profile to result in a different prompt. For example, adding the below prompt will display `6:07 PM [COMPUTERNAME]:>` as the main prompt.
```
function prompt {
 $time = (Get-Date).ToShortTimeString()
 "$time [$env:COMPUTERNAME]:> "
}
``` 

## Useful Operators

### -as and -is
-as can be used to convert an objects data type, aka casting. For example:
```
$a = 1000 / 5
$a.GetTypeCode()

$b = $a -as [string]
$b.GetTypeCode()

$c = $a -as [int]
$c.GetTypeCode()
```
-is returns a boolean depending on if an object is a certain type or not. For example:
```
"Hello" -is [string]
```

### -replace
The -replace operator uses regex/simple matching and replaces all occurrences of one string within another to replace those occurrences.
For example:
```
"Frodo" -replace "Frod","Bilb"
```
The string replace() method is similar, but uses a static replace. 
```
$name = "Frodo" 
$name.Replace('Frod','Bilb')
```

### -join and -split
-join concerts arrays to delimited lists. 
-split splits up delimited lists. 

For example:
```
$party = "Frodo","Gandalf","Bilbo","Gimili"
$party -join ";"
$party -split ";"
```

### -contains, -like and -in
-contains is used to test if an object exists in an array and return a boolean. For example:
```
$party = "Frodo","Gandalf","Bilbo","Gimili"
$party -contains "Gandalf"
```
-in operator does the same, but it flips the order of the operands.
```
$party = "Frodo","Gandalf","Bilbo","Gimili"
"Bilbo" -in $party
```

-like is designed for wildcard string comparisons. For example:
```
"Frodo" -like "*rodo*"
"Frodo" -like "*nazgul*"
```
## Setting Default Parameter Values
Defaults are stored in the $PSDefaultParameterValues variable. This variable is empty each time a new shell window is opened and is populated with a table as variables are run. 

The command `$PSDefaultParameterValues` will display the values set during the session. 
The help file `help about_Parameters_Default_Values` has further information about this. 

## Script Blocks
The following parameters accept script blocks

- FilterScript parameter of Where-Object takes a script block.
- Process parameter of ForEach-Object takes a script block.
- hash table used to create custom properties with Select-Object, or custom columns with Format-Table, accepts a script block as the value of the E, or Expression, key.
- Remoting and job-related commands, including Invoke-Command and Start-Job, accept script blocks on their -ScriptBlock parameter.

The help file `help about_script_blocks` has further information about this. 
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
| `Get-Process | Format-Table Name,ID,@{l='VirtualMB';e={$_.vm/1MB}},@{l='PhysicalMB';e={$_.workingset/1MB}}`| Run get process and includes custom formatting based on calculated values for Virtual MB and Physical MB
|`Get-Process \| Format-Table Name, CPU, Description, Company -GroupBy Company`| Runs get process, then groups the output results by company name |
|`Get-Process \| Where-Object -FilterScript {$_.WorkingSet -gt 100MB}`| Runs get process, then filters where the working process is greater than 100mb |
|`Get-Process \| Where-Object { $_.Name -notlike 'pwsh*' } \|Sort-Object VM -Descending`| Runs get proces, then filters for all objects which are not powershell |
| `Enter-PSSession` | Start a new remote session |
| `Exit-PSSession` | Exit a remote session |
| `Invoke-Command` | Used to run multiple commands at once |
|`Invoke-Command –scriptblock {Get-Process } -HostName Server01,Server02 -UserName <username> `| Runs get-process to retrieve a list of processes running from multiple remote computers. |
| `Start-Job -ScriptBlock { Get-Process }` | Starts a jobs to retrieve all running processes |
| `Start-ThreadJob -ScriptBlock { Get-Process }` | Starts a thread job to retrieve all running processes |
| `Get-Job` | Retrieves all  jobs |
| `Receive-Job -Id 1` | Retrieves the results of job id 1 |
| `Receive-Job -Id 1 -Keep` | Retrieves the results of job id 1 and keeps the results cached |
|`Get-Job -Id 4 | Select-Object -ExpandProperty ChildJobs`| List all the child jobs of a given job |
| `Stop-Job` | Terminates the job. Results generated up to this point can be retrieved |
| `Remove-Job` | Deletes a job and any output still cached within from memory |
| `Wait-Job` | Forces the shell to stop and wait until the job (or jobs) is completed |
|`Get-CimInstance -ClassName Win32_NetworkAdapterConfiguration`| Get all network adapters |
| `Get-CimInstance -ClassName Win32_NetworkAdapterConfiguration | Invoke-CimMethod -MethodName ReleaseDHCPLease` | Invoke a cim method to release DHCP lease |
|`$db_servers = New-PSSession -ComputerName db01, db02, db03 -Credential DbAdmin`| Create multiple sessions in a variable|
|`Invoke-Command -ComputerName { Get-Process } -Session (Get-PSSession -ComputerName db01, db02, db03)`| Run multiple commands on serveral computers with active sessions at once |
| `Write-Verbose "Connecting to hyperdrive >>>"` | Add the optional verbose output which be viewed by adding the -verbose switch to a command |
|`$pshome`| Built in variable which contains the install folder of Powershell |
|`$home`| Built in variable pointing to the current user's profile folder |
|`$Profile | Format-List -force`| List all current profiles|
|`"String" \| get-member`| List all the vailable string manipulation methods|
|`$username = "  Gandalf   " $username.trim()`| Trims all whitespace|
|`$username.TrimStart()`|Trims whitespace at the start|
|`$username.TrimEnd()`|Trims whitespace at the end|
|`get-date | get-member`| List all the available date manipulation methods |
|`Get-Date`| List today's date|
|`(Get-Date).Day`| List the day |