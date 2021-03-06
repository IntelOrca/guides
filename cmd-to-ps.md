# Command Prompt to Powershell
This guide should give you some information on how to switch from Command Prompt to Powershell. It will show you how to create a familiar environment or give you an alternative method for common operations.

I personally use PowerShell over Command Prompt for the following reasons:
* Improved prompt
* Extensible tab completion
* Colour support
* Much easier scripting language with .NET integration

It however has its flaws, and I typically route most of my commands to cygwin executables so that I have a similar environment between Windows and Linux. This guide will show you how to create a profile which gives you a command prompt-like environment.

## Execution policy
Unfortunately PowerShell by default disables running non-signed PowerShell scripts. This obstructs nearly all users, so the first thing you must do is change or bypass the execution policy. There are a number of ways of doing this.

In Windows 10, you can change it under Settings -> Update & security -> For developers.

Other ways:
https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/

## Profile
### Standard method:
PowerShell by default loads your profile from:
`~\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`

This is stored in the `$profile` variable in PowerShell:
```powershell
PS $profile
```

### Alternative method:
The default profile will be loaded when you run PowerShell from anywhere. However you can also launch a PowerShell environment from inside a batch script. This is what I do so that my PowerShell environment inherits my command prompt profile. This is also a convenient way of obtaining the Visual Studio environment.

**profile.bat:**
```batch
set PATH=%HOMEDRIVE%%HOMEPATH%\.bin
call C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat
powershell -NoProfile -NoExit -Command ". profile.ps1"
```

## Prompt
The default behaviour of PowerShell's prompt is similar to Command Prompt. It is more user friendly as it allows selection, cut and paste using the cursor keys, CTRL+C and CTRL+V respectively. Unlike command prompt, tab completion does not remove all the characters after your caret. One gotcha is that `.\` is now required to run any executable within the current directory (like in bash), thankfully this is automatically inserted when pressing TAB.

I like to enable the following features:

```powershell
# Prevents the same line appearing twice when navigating through the prompt history
Set-PSReadLineOption -HistoryNoDuplicates:$true
```
I am not sure why this is not enabled by default as I do not see how navigating through duplicate history items is useful.

```powershell
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward
```
This is one of my favourite abilities which I first came across in [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh). This allows you to type the first part of a command and then have it auto complete with a previously typed command using the up and down arrow keys. It makes searching and running historical commands so much faster to do.

You can set your prompt text by defining a `prompt` function. Here is the one I use which displays the name of current directory you are in:

```powershell
function prompt
{
    $pwd = (Get-Location).Path
    $folderName = [System.IO.Path]::GetFileName($pwd)
    if ($folderName -eq "")
    {
        $folderName = $pwd
    }
    Write-Host "PS " -NoNewline
    Write-Host $folderName -ForegroundColor DarkCyan -NoNewline
    return " ";
}
```


## Aliases
PowerShell works by defining and running cmdlets which are functions that you can define or import as opposed to running separate applications. This allows .NET objects to be passed around rather than plain text which offers many advantages. PowerShell by default aliases many well known shell commands to their equivalent cmdlets.

You can view all your aliases by with:
```powershell
PS Get-Alias
```

In some cases you may not want some of these. Here is how to remove them:
```powershell
Remove-Item alias:cp
Remove-Item alias:curl
Remove-Item alias:diff -Force
Remove-Item alias:dir
Remove-Item alias:ls
Remove-Item alias:mv
Remove-Item alias:rm
Remove-Item alias:rmdir
Remove-Item function:mkdir
```
**Note:** mkdir is a function, not an alias. You also sometimes have to remove an item with `-Force` for whatever reason.

You can set an alias as follows:
```
Set-Alias code code-insiders
```

## Functions
I won't go in to the details of how to write functions. Here are some examples though of how to write basic functions for re-introducing cmd commands.

```powershell
function mklink { cmd /c "mklink $args" }
function rmdir { cmd /c "rmdir $args" }
```
PowerShell isn't as good as command prompt at handling symbolic links. I therefore use cmd to create and remove them. This is only necessary for command prompt commands which are not executables. For example this is not required for `findstr`.

```powershell
function dir { cmd /c "dir $args" }
```
If you prefer command prompt's style of directory listings. You will not be able to pipe to other cmdlets as your output will be text rather than objects.

```powershell
function strargs($args2)
{
    $result = @()
    foreach ($arg in $args2) {
        $rarg = $arg;
        if ($arg.Contains(' ')) {
            $rarg = """$rarg"""
        }
        $result += $rarg
    }
    return $result
}
function ls
{
    Invoke-Expression ("Get-ChildItem " + (strargs $args)) | Select-Object Mode, LastWriteTime, Length, Name, Target | Format-Table
}
```
A demonstration of defining your own custom directory listing style and how to pipe the output to another cmdlet.

## Calling batch scripts
One of the lacking features of PowerShell is that there is no built in method for calling a batch script and have its environment persisting. This means you have to create a function which executes the batch script with a `set` command at the end and then re-parse the values of the environment variables back into PowerShell.

I have provided a [copy](Call-Batch.ps1) from [nightroman/PowerShelf](https://github.com/nightroman/PowerShelf) adjacent to this guide. You can import it from your profile with:
```powershell
Import-Module Call-Batch.ps1
```
