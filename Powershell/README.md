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
| Continue | (Default) Display the error message and keep on going |
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

## Toolmaking
When creating powershell scripts, focus on creating single-task tools. These tools are then activated or controlled by a controller script. 
Do not create tool that perform multiple tasks or reproduce functionality already avaialble in powershell.

## Naming Conventions
- Start with a powershell approved verb. Run `Get-Verb` for a list of powershell approved verbs
- Always use a singular noun, i.e. user not users
- Prefix the noun with something related to your company or functions. For example `Get-ECorpUser`

## Naming Parameters
- More important than naming a cmdlet
- When deciding on a parameter name, focus on the core, native powershell commands. What param name would they use?
- Core command param examples:
    * -ComputerName
    * -FilePath
    * -PSSession
- Consistent powershell parameter labelling is imporant as it allows commands to connect in the pipeline.
- If you need to determine a parameter name is a good selection, run Powershell to see if other commands are using the parameter:
```
Get-Command -CommandType Cmdlet -ParameterName ComputerName
Get-Command -CommandType Cmdlet -ParameterName Computer
```

## Creating proper output
- Powershell commands produce objects as output
- Consider an object as a row in spreadsheet, with each column representing a property
- Objects are structured data. An object's data can be retrieved by referring to it's property names
- Objects are output and placed into the powershell pipeline
- This carries the object ot the next command in the pipeline
- When the last command has output its objects in the pipeline, it is then handled by the processing system
- The Formatting system then determines how to display the output 
- Tools should not focus on formatting output, rather getting the correct objects in the pipeline is key.

## Parameters Revisted
- The only way a command can accept data, is through its parameters
- When you design a command, and design its parameters... you decide how a commmand will accept information
- Each parameter is passed through a pipeline. For example:
```
Get-Service | Where Status -eq "Running" | ConvertTo-HTML | Service-File stats.htm
```

## Pipeline ByValue
Powershell has a preference to pass entire objects from the pipeline into a command. In order to allow this:
1. The recieving command must have a parameter that supports accepting pipeline input ByValue
2. That parameter must be able to accept the incoming object type

Way of showing the help menu in a window
```
Help Convertto-html -ShowWindow
```
In pipelines, System.Object is the mother type for all other objects. Everything else inherits from this Object type. System objects can accepts pipeline input using the ByValue technique.

## Trace Command
Command which allows you to view the passing of objects. 
For example:
```
 Trace-Command -Expression { Get-Process | ConvertTo-Html | out-null } -Name ParameterBinding -PSHost
```
Specifying named or positional paramters always takes precedence. These items are always binded first. In the debug output, the phrase `No Coercion` indicates powershell was able to output as-is without attempting to convert it to something else. 

## ByPropertyName
If ByValue parameter binding fails, powershell will look at the objects in the pipeline to see if they have a property with the same name and bind to that parameter. If two commands have matching propety and parameter names, this can be a useful technique. For example:
```
Import-CSV Users.csv | New-ADUser
```
If "SKIPPED" appears in the debug trace log, this indicates that ByValue did not work, powershell will then attempt ByPropertyName. For example:
```
 Trace-Command -Expression { Get-Process -Name o* | Stop-Job }  -PSHost -Name ParameterBinding
```
If ByValue and ByPropertyName both fail, then the entire pipeline will fail. The pipeline will throw an error message and fail. 

## Planning Parameters
- When you start designing tools: only have 1 parameter designed to accept pipeline input by ByValue. 
- As many parameters as you want can accept input ByPropertyName
- Determining this plan requires understanding the useage pattern:
    1. Will your command/tool be accepting pipeline input?
    2. Will your command/tool be the initial command at the start of the pipeline

## Scripts and Security

### PowerShell's Script Security Goal
Power shell's security mechanisms are not a boundary, rather they are intended to prevent accidental or unintended execution. Anything an attacker could do in powershell, they could also do without it. Locking down or removing powershell, does not get rid of COM, .NET framework or WMI. These underlying items are still there. For example, a user may not restricted by their execution policy to not be able to click and run scripts. However, if they copy and paste the same script into the shell, it will run at their permission level.

### Execution Policy
Run the command `Get-ExecutionPolicy` to see the current execution policy. An execution policy can be set with `Set-ExecutionPolicy`
The command `help about_Execution_Policies` will display details about execution policies
These policies are intended to prevent accidenal execution only and are not considered a true security measure.

| **Policy** | **Description** |
| --------------|-------------------|
| Restricted | Default. Scripts are not executed |
| AllSigned | Powershell will execute anything signed by a CA |
| Unrestricted | All scripts will run |
| RemoteSigned | PowerShell will execute any local script and will execute remote scripts if they’ve been digitally signed by a trusted CA |
| Bypass | Ignores the configured execution policy. Special settings used primarily by app developers embedding powershell within an app. |
| Undefined | No policy found. Powershell will move down the scope list to user the first effective policy |

### Execution Scope
You can view your current scope by running `Get-ExecutionPolicy -List`. 
Items in the policy scope are set via Group Policy
The execution policy can be set at one of the following scopes:

1. LocalMachine - Applies to the entire machine. The top level policy, outranks all others. 
HKLM:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell

2. CurrentUser - Applies to the current user
HKCU:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell

3. Process - Applies to the current powershell session set in the `$env:PSExecutionPolicyPreference` variable

### Default Application
By default, a .ps1 script is open in notepad, it is not run by powershell

### Recommendations
Use AllSigned when certs will be used to control script releases.
AllSigned is useful as a procedure, declaring a script ready for production and useage.
Not useful as a security measure. A user running a script cannot do anything beyond their current permissions

___

## Scripting in Powershell

### Comparrison Operators
Powershell's core comparrison operators are:

| Name | Description |
| ---- | ---- |  
| -eq | Equal to |
| -ne | Not equal to |
| -gt | Greather than |
| -ge | Greater than or equal to |
| -lt | Less than |
| -le | Less than or equal to |

* If you need a case-sensitive comparison, add a 'c' in front of the name, for example 'ceq' or 'cgt'

### Wildcards
Common wildcard comparrison can be done with `-like` and `-notlike`. * is used to match zero or more characters while ? is used to make single character comparrisons.
For example:
```
'bill' -eq 'will'
'bill' -like '*will'
'don' -notlike 'don*'
'ron' -like 'r?n'
'donald' -like 'd?n'
```
### In vs Contains
`-contains` and `-in`  are both used to check an array for a certain value. 
However, note how the order they appear in is important. 
For example:
```
$array = @("one","two","three")
$array -contains "one"
"one" -in $array
```

### If Constructs
If is used to make logical decisions in code. At each stage and expression is evaluated and 
returns true or false. If the expression evaluates to true, the code below will execute. The basic syntax is:
```
If (<expression>) {
    # code
} ElseIf (<expression>) {
    # code
} ElseIf (<expression>) {
    # code
} Else {

    # code
}
```
Here is an example of using If to perform an action and stop a process
```
$proc = Get-Process -Name "notepad"

If ($proc.cpu -gt 1) {
    Stop-Process -Name $proc
}
```
You can create a between situation by using gt and lt together with -or. For example:
```
If ($proc.vm -gt 4 -or $proc.vm -lt 2) {
  # take some action
}
```

### Foreach
foreach is designed to take a collection or an array of objects and go through them one at a time. 
During each turn, the object is placed in a sperate variable to refer to it. 
```
ForEach ($item in $collection) {
    # run code on $item
}
```

Best Practice:
Use ForEach over ForEach-Object in a script. You can name your single item variable, it executes faster and often consumes less memory.

### Limitations
foreach does not write directly to the pipeline. The below will not work as the pipe is considered empty once the loop completes.
```
$numbers = 1..100
foreach ($n in $numbers) {
    $n * 3
} | Out-File num.txt
```
Instead, capture the loop in a variable. Then pipe the variable to the outfile. 
```
$numbers = 1..100
$num = foreach ($n in $numbers) {
    $n * 3
} 
$num | Out-File num.txt
```
You do not always need to use a looping construct. For example:
```
$services = Get-Service -name bits,lanmanserver,spooler
Foreach ($service in $services) {
  Restart-service $service -passthru
}
```
Versus this:
```
$services = Get-Service -name bits,lanmanserver,spooler
$services | restart-service -passthru
```

### Switch
The Switch construct is used to replace a huge if block which contains multiple ElseIf sections. 

Syntax is:
```
switch (<principal>) {
  <candidate> { <script block> }
  <candidate> { <script block> }
  <candidate> { <script block> }
  default { <script block {
}
```
- The priniciple is a variable with a single value or object. It cannot contain collections or arrays.
- Each candidate is a boolean or value the principal might contain
- The script block will execute if the candidate is true
- The default block executes if nothing matches.

Here is an example:
```
$hero = "Batman"

switch -wildcard($hero)
{
    "*Joker*"{"Contains Joker"}
    "*Clayface*"{"Contains Clayface"}
    "*man*"{"Contains man"}
    default {"Nada"}
}
```

### Do/While
While executes a specific block of statements. This will execute while some condition is true. There are two basic variations

While: Executes as long as the condition is true.
```
While (<condition>) {
 # code
}
```

Do While: Executes as least once then as long as the condition is true.
```
Do {
 # code
} While (<condition>)
```

### Break
The Break keyword exits any construct. 
Some exceptions:
- If it is a For, Foreach, While or Switch construct, Break will immediately exit. 
- In a script, but outside a loop -- Break will exit the entire script.
- In an IF construct Break will exit whatever contains the If construct. 

Break is useful for aborting an operation. For example if you have a list of computers or IPs you want to ping, go through them one-by-one but stop immediately if one does not responsd. 
```
$ips = '8.8.8.8', '9.9.9.9', '192.168.10.1'
foreach ($ip in $ips) {
    If (-not (Test-NetConnection $ip)) {
        Break
    }
}
``` 
Here is an example of a while loop using break to end an infinite loop. This can be useful for requesting input indefinetly, etc. Be careful with this approach as it best to write loops with a natural end point. If Break is needed, surround it by blank lines and comments to indicate the intention.
```
While ($true){
    $n = Read-Host "Enter a Number"
    If ($n -eq 0) { break }
    else { Write-Host 'Enter another: '}
}
```
___

# Script Design

## Design Principles
Tools do one thing. Do not create your own tool if the functionality already exists in an existing powershell cmdlet.
Design tools that are testable, as you will need to create automated unit tests for your tools.
Single purpose tools are easily testable. The fewer functionalities and logic branches the tool has, the easier it will be to use and maintain. 

Single purpose tools require designing the tools to perform pipeline input.
Write examples and possible use cases of your tool(s) to consider how you will code them. 
For example, if you had the program `Set-MachineStatus`, the following could be some example use cases:
```
Set-MachineStatus
Get-Content names.txt | Set-MachineStatus
Get-ADComputer -filter * | Select -Expand Name | Set-MachineStatus
Get-ADComputer -filter * | Set-MachineStatus
Set-MachineStatus -ComputerName (Get-Content names.txt)
```
Be careful with tool names and parameter names. Tools should always adopt the standard verb-noun pattern in powershell. 
Only use an approved verb from the list `Get-Verb`. 

Create a short prefix on your command's noun to make the tool distinguishable from other powershell cmdlets.

Parameter names should always follow Powershell Parameters. Whenever you need a parameter, examine how native powershell cmdlets 
name and use their parameters. For example, if you need a path it will usually be -FilePath or -Path on most native commands.

## Design and context
Not every business need can be solved with a single tool or command. Sometimes a suite of tools or a module is needed. 
Always stay focused on the tool/controller design pattern: Tools do one thing. Controllers manage tools. 

There is an approach which states you start tool design by writing the help file. Then code to the help file. 
TDD is test-drive development, where you write automated tests first to specify how your code should work. 

Always run powershell commands in the shell to practice and experiment with. 

___

# Building a Module

## Start with a function
Keep your function tightly scoped and self-contained. Functions should perform one function only. 
General function best practices:

1. Info to be used by the function should only come from parameters which feed into the function.
Functions should not rely on external variables outside the function itself. 

2. Output from a function should be to the Powershell pipeline only. 

## Designing Input Parameters
Any info which will need to be fed to the function should be created in parameters.
- Data types are enclosed in square brackets
- Common data types include [string],[int],[datetime]
- Parameters become variables inside the function
- Seperate sections with commas
- The [string[]] denotes an array of values.


## Creating a script module
Script modules are supported by Powershell v2 and later. Generally these modules should be stored in a path specified by the PSModulePath environment variable, `$env:psmodulepath`. A script module requires a `.psm1` extention and to be placed in a subfolder here. The subfolder and module name must match. To temporarilyload a module, you will need to manually run `Import-Module` and provide the full path to the module being loaded. The `-Force` parameter will allow you to overwrite any existing versions of the module. Modules can then be removed by running `Remove-Module`

## Advanced Functions
In order to create an "Advanced Function" add the `[CmdletBinding()]` parameter to the function.
For example:
```
function test {
    [CmdletBinding()]
    Param(
        [string]$ComputerName
    )
}
```
Adding this will enabled access to common parameters, decorators for parameters and additional help options. The help files `about_CommonParameters` amd `about_Functions_Advanced` provide additional details. 

Some useful common parameters are:
| **Command** | **Description** |
| --------------|-------------------|
|`-Verbose`| Enables useage of Write-Verbose in your function. |
|`-Debug`| |`-Verbose`| Enables useage of Write-Debug in your function. |
|`-ErrorAction`| Create custom actions in the event of an error. |
|`-ErrorVariable`| Specify an error variable to capture errors. |
|`-InformationAction`| Overrides the global $InformationPreference variable, enabls Write-Information in the function. |
|`-InformationVariable`| Create a variable for Write-Information to hold info. |
|`-OutVariable`| Specifies a variable in which PowerShell will place copies of your function's output |
|`-PipelineVariable`| Specifies a variable for powershell to store a copy of the current pipeline element |

### Parameter Decorators
Advanced functions allow you to add parameter decorators.
Parameter dectorators can enable accepting pipeline input by value by using the `ValueFromPipeline=$True` and 
pipeline input by property using the `ValueFromPipelineByPropertyName=$True` parameter decorator.
`Mandatory=$True` will make the parameter mandatory. The `[ValidateSet()]` enables input validation. Aliases can also be added. 
Adding `SupportsShouldProcess=$True` will enable the `-WhatIf` and `-Confirm` parameters for the function. 

For example:
```
function test {
    [CmdletBinding()]
    Param(
        [Parameter(ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$True, Mandatory=$True)]
        [Alias('CN','Hostname','MachineName')]
        [string]$ComputerName
    )
}
```
___

# Objects and output

## Splatting
Splatting allows you to pass parameters using a hash table. The keys need to be parameter names and the values are the corresponding parameter values.The hashtable is then assigned to a variable. For example:
```
$params = @{'Name'='example.com'
            'Server'='1.1.1.1'}

Resolve-DnsName @params
``` 
## Object output
Never use `Write-Host` to create output. `Write-Host` will only draw the output directly on to the screen. That output cannot be fed to the pipeline, maniupulated or re-used. Forllowing the previous example, you can create an object by running `New-Object -TypeName PSObject` like the following:
```
$params = @{'Name'='example.com'
            'Server'='1.1.1.1'}

$dns = Resolve-DnsName @params | Select-Object -Property Name, Type, IPAddress

$props = @{'Name'= $dns.Name
            'Type'=$dns.Type
            'IP'=$dns.IPAddress}

$obj = New-Object -TypeName PSObject -Property $props
```

An alternative to performing this can be done using `[pscustomobject]@{` For example:
```
$params = @{'Name'='example.com'
            'Server'='1.1.1.1'}

$dns = Resolve-DnsName @params | Select-Object -Property Name, Type, IPAddress

$obj = [pscustomobject]@{
Name = $dns.Name
Type = $dns.Type
IP = $dns.IPAddress
}

```
Either approach is fine, with `[pscustomobject]@{` being a little newer. 
Lastly `Add-Member` will allow you to add additional info on to an object after it has been created. For example:
```
$params = @{'Name'='example.com'
            'Server'='1.1.1.1'}

$dns = Resolve-DnsName @params | Select-Object -Property Name, Type, IPAddress

$obj = [pscustomobject]@{
Name = $dns.Name
Type = $dns.Type
IP = $dns.IPAddress
}

$obj | Add-Member -MemberType NoteProperty `
            -Name Protocol `
            -Value 'DNS'

```
___

# Pipelines

Powershell has 6 channels or pipelines. The sucess pipeline is used to pass commands from object to object.
This is the pipeline you know. It takes objects and outputs them to the screen. 

All the pipeline types are:
1. Success
2. Error
3. Warning
4. Verbose
5. Debug
6. Information

Each pipeline has a specific method needed to pass information to it, i.e.
the verbose pipeline displays items in bright yellow text.

There are also several preference variables for each pipeline which control the output. 
For example, `$VerbosePreference` controls the verbose pipeline.

## Verbose Output
Verbose output is disabled by default.
Warning output is enabled. 

Verbose is often used to write explanatory items in the output.
For example:
```
Write-Verbose "Connecting to $computer over $protocol"         1
        $session = New-CimSession -ComputerName $computer '
                                  -SessionOption $option

        Write-Verbose "Querying from $computer"
        $os_params = @{'ClassName'='Win32_OperatingSystem'
```
Do not wait until after you have finished scripting to add vervose statements in, add them as you script. Insert verbose messages throughout your script. Adding messages helps you troubleshoot errors as they arise. Verbose messages can also serve as internal documentation. 

You can even add a prefix to each verbose message to indicate what script block is being called. 
```
Function TryMe {
[cmdletbinding()]
Param(
[string]$Computername
)

Begin {
    Write-Verbose "[BEGIN  ] Starting: $($MyInvocation.Mycommand)"
    Write-Verbose "[BEGIN  ] Initializing array"
    $a = @()

} #begin

Process {
    Write-Verbose "[PROCESS] Processing $Computername"
    # code goes here
} #process

End {
    Write-Verbose "[END    ] Ending: $($MyInvocation.Mycommand)"

} #end

} #function
```

## Information Output
Similar to verbose output, using the Information channel requires planning. 
You need to determine what needs to be logged and how it might be used. 

Here is an example of recording variables using an information variable
```
Function Test-Me {
[cmdletbinding()]
Param()

Write-Information "Starting $($MyInvocation.MyCommand) " -Tags Process
Write-Information "PSVersion = $($PSVersionTable.PSVersion)" -Tags Meta
Write-Information "OS = $((Get-CimInstance Win32_operatingsystem).Caption)"'
-Tags Meta

Write-Verbose "Getting top 5 processes by WorkingSet"
Get-process | sort WS -Descending | select -first 5 -OutVariable s

Write-Information ($s[0] | Out-String) -Tags Data

Write-Information "Ending $($MyInvocation.MyCommand) " -Tags Process

}
```

If you run this with `test-me -InformationAction Continue` it will display each of the informational 
actions and `test-me -InformationVariable inf` will store the output in an inf variable. 

The variable storing information output also contains a range of actions. 
For example run `$inf | gm`

___

# Manifests

When powershell starts up, it enumerates all the folders listed in `$PSModulePath` environment variable to load modules.
Every folder under this path is a potential module to be loaded. 

PowerShell considers the following potential modules:

- .psd1 file having the same filename as the module's folder name. This is a module manifest and tells the shell what else needs to be loaded.

- .dll file having the same filename as the module's folder name. This is a compiled or binary module, usually written in C#.

- .psm1 file having the same filename as the module's folder name. This is a script module.

Script modules can be ineffecient since they require loading everytime the shell starts. If a manifest is specified it will greatly speed up load time.
To create a manifest, you will first need a script module prepared. Run the following command:
```
New-ModuleManifest –Path <ModuleName>.psd1 –Root ./<ModuleName>.psm1
```

## Manifest Sections
Some metadata or data about the module itself includes:

- `ModuleVersion` Module version using the standard Microsoft w.x.y.z version notation. Mandatory if you submit modules to PowerShellGallery.com.
- A globally unique identifier (GUID) is a requirement and is generated automatically.
- Author;  Mandatory if you submit modules to PowerShellGallery.com.
- Copyright and Description are optional, but you should include Description for PowerShellGallery submissions (it may become mandatory at some point).
- `ModuleList` a list of all submodules that your module includes, i.e. the names of any .psm1 files. Does not perform any actions and is rarely seen
- `FileList` is similar to ModuleList; A method to document all the files included in the module.

## Supporting elements
You can specify several supporting elements which are loaded and unloaded anlong with the module. Each of these elements is an array. 
- ScriptsToProcess lists PowerShell scripts (.ps1 files) to be run before the module is loaded. 
- TypesToProcess; a list of PowerShell Extensible Type System (ETS) extensions—usually .ps1xml files.
- FormatsToProcess; a list of PowerShell formatting view files—usually .ps1xml files that your module needs to load. 
Usually these items are placed in each supporting module's folder and specified using `./filename` in the array instead of loading the entire module. 

## Exporting Members
You can declare certain functions as being exported from the module, i.e. available. 
Any functions which are not explicitly exported become private to the module and unavailable outside the module. 
This is useful for creating internal helper modules.

Five types of items can be exported:

1. FunctionsToExport holds functions you want users to use.
2. CmdletsToExport equivalent of FunctionsToExport when publishing a compiled module, not used in a script module.
3. VariablesToExport holds module-level variables to add to the global scope. Good way to publish variables that set things like log filenames, database connection strings, and so on.
4. AliasesToExport holds aliases you define in your module (using New-Alias)
5. DscResourcesToExport is a special list related to building Desired State Configuration (DSC) resource modules.
___

# Scripting Tips
The folllowing are some tips to improve your scripting.
- Use Source Control. Source Control advantages include easier collaboration with teams, revisit earlier versions of your code, backup your code, share code and control community input. 
- Spelling it Out. When sharing your script avoid aliases. Spell everything out fully. 
- Use comments to explain your intention as you progress, not what a cmdlet does. `Write-Verbose` is used for explaining the progression of the script, not what an individual command does. 
- Formatting. Focus on spacing for readability, keep hash tables tightly structured, and include comments for closing braces. Comment the end of constructs `{`.
- Variable names should provide a clear idea of whats in them. Parameter variables should always be singular. Avoid Hungarian style variable naming, i.e. placing the data type in front of the variable name such as `strComputer="blah"`
- Scripts are structured, permanent artifacts. In contrast, the console is best for one-off comands to get something done quickly and forget about it. Make sure your scripts are clearly structured. 
- Learn how to use `PlatyPS` an open source project used by the powershell team to generate external, i.e. not comment-based help. 
- Don't use `Write-Host` for output. Alternatives can be `Write-Information` to write to the information channel. If you need to show the command progress to the user, learn how to use the Write-Progress cmdlet instead of `Write-Host`.
- Use single quotes to delmit strings. Double quotes are only needed to call variables or subexpressions inside the string. 
For example:
```
$message = "The computer name is $computername"
$message = "Yesterday was $( (Get-Date).AddDays(-1) )"
```
- Don't mess with the Global Scope; Do not push your own variables into the global scope. 
- Maintain Flexiblity. Do not hardcode values. Use Parameters.
- Never hardcode credentials into your code. No usernames. API keys. Plaintext passwords, etc. Learn how to use the [pscredentials] object as a parameter.

___

# Testubg

## Automated Unit Tests
A general process to follow is:
1. Write some code or modify code
2. Check the code into source control
3. Repo triggers a continous integration pipeline. 
4. The pipeline builds out a vm to test your script. The pipeline copies the script into a vm and runs several tests. 
5. If any tests fail, you get an email with the details.
6. If the tests pass, code is deployed to a repository, making it available for production useage. 

## Problems with Manual Testing
Manual testing has several issues. These include:
1. Cannot cover every possible situation with manual testing. 
2. Time-consuming. Manual testing takes time and effort. 
3. Manual testing does not auto-change or adapt itself. 

## Benefits of Automated Testing
Benefits of automated testing includes:
1. Automated testing is automatic.
2. Can adapt and learn your code. 
3. Supports TDD, Test Driven Development i.e. designing and writing tests to test your code before actually scripting the code. 

## Pester
Open source project that is bundled with Windows 10. An automated unit-testing framework for powershell. 
The coder writes baseic tests in pester, then pester runs the tests for you. Microsoft uses pester to automate the testing of its own Powershell project.  
Pester Website: [https://pester.dev/](https://pester.dev/)
Pester help file: `about_pester`

## Coding to be tested
To successful use pester or code testing, focus on building self-contained, single-task modular tools. Tools which perfom several different activities are difficult to test. 

Pester is only a powershell testing framework. It does not handle .NET, COM or other stuff. 

## Testing Scenarios

### Integration Tests
Focuses on testing the end state and results of your command. 
If you write a command to create a VM, it tests if this was successful. 
Integration tests do not focus on what is happening inside the code, rather if the goal itself was achieved. 

### Unit Tests
Unit tests are more granular. These tests are designed to check the internal working of the code and logic to ensure the code runs. 
The end goal of the script is not the main concern of the unit test. 

___

# Signing your Script

## Why?
Signing a script authenticates the person who wrote it. This does not mean the script is safe to run. Script signing also allows for the verification of code integrity, confirming the script content has not been changed.

## Certificates
Signing your script requires certificate generation. 

The purpose of a cert is identity. Certificates are issued by Certificate Authorities to validate identities. Your identity is embedded in your Certificate. 

If your code is internal to your organizaition, you may have internal PKI infrastructure and be running your own internal CA. This will alllow internal code signing. A commercial CA or internal PKI is essential for deploying code, either internally or externally. 

Self-signed certificates can also be generated. These certs are only useful when you are running code yourself on a computer under your control. 

Certificates consist of PKI key pairs. Public and private keys. Your private key is kept safe, not shared and used to generate a hash of your code. The public key is shared and can be used to verify this hash. 

## Setting Execution Policy
The first step to test script signing requires configuring your command to require signed scripts. Run the following from an Admin elevated prompt:
```
Set-Executionpolicy AllSigned -force
```

## Code Signing Basics
Certificates issued by a CA must support Microsoft's Authenticode extension. 
The certificate must also be issued by a CA that is trusted by your computer. 
If you have an AD Domain in your organisation, they will likely be using Active Directory Certificate Services to issue code signing certificates. 

To create a self-signed certificate, you will need powershell's Remote Server Administration Tools. This will allow you to run the following:
```
New-SelfSignedCertificate -type CodeSigningCert -Subject "CN=Art
 Deco" -CertStoreLocation Cert:\CurrentUser\My\ -testroot
```
After creating the cert, powershell will throw an "unkown error" message since it cannot yet verify the certificate chain. The certificate creation can then be verified by checking:
```
dir Cert:\CurrentUser\My\ -CodeSigningCert
```
In code signing, a certificate's thumbprint refers to its official, unique name. 
To fully issue a self-signed certificate on a windows computer you will also need to add the certificate in the certificate manager snap-in, i.e. `Certmgr.msc`. 

Sign a script by performing the following:
```
$cert = dir Cert:\CurrentUser\My\ -CodeSigningCert
Set-Alias -Name sign -Value Set-AuthenticodeSignature
sign .\psvm.ps1 -Certificate $cert
```
Note the creation of the alias in line 2. Once this is done, the code signing will add a signature block to your code. In powershell, you can sign .ps1, .psm1, and .ps1xml files.

# Testing script signatures.
The following command can be used to test and verify a script's signature:
```
Get-AuthenticodeSignature .\psvm.ps1
```

___

# Publishing 

## Requirements
Publishing a scripts allows you to share your script with others and track your older versions. 

Before you publish consider: Does this functionality already exist? Check the Powershell Gallery to make sure you are not doubling up here.

Before publishing you will need to update your manifest. 

The following settings are required in a manifest before publishing:
1. ModuleVersion
2. Author
3. Description
4. PrivateData: Tags, LicenseUri - link to a url to your license file
5. ProjectUri - The URL of the github repo or location of your module

You will also need to apply for a powershell gallery API key before publishing. 
Here is a sample command for publishing a script
```
Publish-Module -path <PATH> -repository PSGallery -nugetapikey $api_key
```

Scripts can also be published. Before publishing a script, you will need to create a special type of header that includes all the required tags, versioning and metadata. This is done with the `New-ScriptFileInfo` cmdlet.

Once the proper script requirements are in place, the script can be published to the powershell gallery using an API key:
```
Publish-Script -Path C:\scripts\Check-ModuleUpdate.ps1 -NuGetApiKey
 $psgallerykey -Repository PSGallery
```

## Searching the gallery
You can search the powershell gallery for commands or by tags. For example:
```
find-module *scan* | Select Version,Name,Author, Description,PublishedDate
find-script *scan* | Select Version,Name,Author, Description,PublishedDate | Format-Table
```
Search by tags
```
find-module -tag ad,activedirectory
```
___

# Debugging

## Types of bugs
Here are 3 types of categories of bugs.

1. Syntax bugs: You typed something wrong. Misstyped variables can be particularly difficult to detect.

2. Results Bugs: A command produces results you do not expect.

3. Logic Bugs: The commands run without an explicit error, but an issue with the way your code is written causes an error.

## Dealing with Syntax Bugs
Setting strict mode can help handle bugs. This can be done by typing at top of the begin or other block in a function:
```
Set-StrictMode –Version 2.0
```
Strict mode throws errors when referring to non-existent variables and enforces strict syntax when calling functions. This prevents many user-caused errors. 

## Breakpoints
Allow you to run a script and pause at a specific place. This pause allows you to examine the script, check the contents of variables/properties and even proceed to run the script line by line. 

Setting breakpoints and debuggins requires having a strong idea about what your script is supposed to do -- and where it might need to be fixed. 

## Setting Watches
Once a breakpoint is set and debugging has started, you can set variables to "Watch" in the left column. By adding a variable with a `+` you can watch the variable change between breakpoints. 

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
|`Get-Job -Id 4 \| Select-Object -ExpandProperty ChildJobs` | List all the child jobs of a given job |
| `Stop-Job` | Terminates the job. Results generated up to this point can be retrieved |
| `Remove-Job` | Deletes a job and any output still cached within from memory |
| `Wait-Job` | Forces the shell to stop and wait until the job (or jobs) is completed |
|`Get-CimInstance -ClassName Win32_NetworkAdapterConfiguration` | Get all network adapters |
| `Get-CimInstance -ClassName Win32_NetworkAdapterConfiguration \| Invoke-CimMethod -MethodName ReleaseDHCPLease` | Invoke a cim method to release DHCP lease |
|`$db_servers = New-PSSession -ComputerName db01, db02, db03 -Credential DbAdmin`| Create multiple sessions in a variable|
|`Invoke-Command -ComputerName { Get-Process } -Session (Get-PSSession -ComputerName db01, db02, db03)`| Run multiple commands on serveral computers with active sessions at once |
| `Write-Verbose "Connecting to hyperdrive >>>"` | Add the optional verbose output which be viewed by adding the -verbose switch to a command |
|`$pshome`| Built in variable which contains the install folder of Powershell |
|`$home`| Built in variable pointing to the current user's profile folder |
|`$Profile \| Format-List -force`| List all current profiles|
|`"String" \| get-member`| List all the vailable string manipulation methods|
|`$username = "  Gandalf   " $username.trim()`| Trims all whitespace|
|`$username.TrimStart()`|Trims whitespace at the start|
|`$username.TrimEnd()`|Trims whitespace at the end|
|`get-date \| get-member`| List all the available date manipulation methods |
|`Get-Date`| List today's date|
|`(Get-Date).Day`| List the day |
|`Help Convertto-html -ShowWindow`| Display a window for a given help command|