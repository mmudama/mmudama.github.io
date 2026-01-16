# Hugo command rule

## Hard override
Always use `hugo.exe` instead of `hugo` when generating shell commands, scripts, or examples for this repository.

### Explicit rule
Never generate the command `hugo` by itself.  
Always generate `hugo.exe`.

#### Note for humans
The version of hugo available through apt is too old to work with the hugo-coder theme. I standardized on the Windows-installed Hugo so that WSL and PowerShell always use the same version. 
