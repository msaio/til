# PowerShell (pwsh)
Today i convert this shell script function into powershell script

```
for browser in "$@"
	do
		# Select all process with name
		pgrep -f -a "$browser" | \
		# Get all renders
		grep 'type=renderer' | \
		# Exclude extensions
		grep -v "extension" | \
		# Select only PID
		egrep -o '^[0-9]{0,}' | \
		# Finally, kill'em all!
		while read pid; do kill $pid; done 
		# One line
		# > pgrep -f -a "$browser" | grep 'type=renderer' | grep -v "extension" | egrep -o '^[0-9]{0,}' | while read pid; do kill $pid; done
	done
```
into
```
function Flush-Browsers {
    param (
        [Parameter(Mandatory=$true, ValueFromRemainingArguments=$true)]
        [string[]]$browsers
    )

    foreach ($browser in $browsers) {
        Get-Process -Name $browser |
        Where-Object { $_.MainWindowTitle -ne "" } |
        ForEach-Object {
            $browserProcesses = $_.Id
            $childProcesses = Get-WmiObject Win32_Process -Filter "ParentProcessId='$($browserProcesses)'" |
                              Where-Object CommandLine -notlike "*extension*" |
                              Where-Object CommandLine -like "*type=renderer*"

            foreach ($childProcess in $childProcesses) {
                Stop-Process -Id $childProcess.ProcessId -Force
            }
        }
    }
}
```
### NOTEk
To publish the function so we can use anywhere in pwsh, we need to a file

> C:\Users\msaio\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1

You can check if file existed with:
```
Test-Path $PROFILE
```
`$PROFILE` is alway mapped to above file

To edit it, run `code $PROFILE`
then run `. $PROFILE` to make it avail 
just like `source ~/.bashrc` in linux

####  Not so soon :)
Error:
>. : File C:\Users\msaio\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 cannot be loaded because ruknning scripts is disabled on this system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
>At line:1 char:3
>+ . $PROFILE
>+   ~~~~~~~~
>    + CategoryInfo          : SecurityError: (\:) [], PSSecurityException
>    + FullyQualifiedErrorId : UnauthorizedAccess

To deal with it, check this [thread](https://stackoverflow.com/questions/41117421/ps1-cannot-be-loaded-because-running-scripts-is-disabled-on-this-system)
or just run **Powershell with administrator**
and run
```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted
```
then hit `. $PROFILE` and things will be fine

#### BTW
Above fix just a workaround solution, it will give attacker rights to excute script then

