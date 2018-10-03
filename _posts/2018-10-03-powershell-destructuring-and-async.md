---
layout: post
title: Powershell assignment destructuring, async, and blocks
---

Powershell supports destructuring assignments
```powershell
$nugetVersion = gitversion | ConvertFrom-Json
($versionPrefix, $versionSuffix) = $nugetVersion.NuGetVersion -split '-'
```

-----------------

Running tasks async in powershell is as simple as `Start-Job`, `Get-Job`, `Wait-Job`, and `Receive-Job`
```powershell
#given a code block
$nugetPackCmd = { param($cwd, $dirName, $versionPrefix, $versionSuffix)  #insert real code here }

#and a set of projects to build
$projectsToBuild | %{  
  $ignore = Start-Job -ScriptBlock $nugetPackCmd -ArgumentList $(Get-Location), $($_.Name), $versionPrefix, $versionSuffix  
}  
Get-Job | Wait-Job | Receive-Job; Remove-Job -State Completed  
```
Note that no global state is captured as closures, including the current working directory.  
Additionally, to reuse Blocks in other contexts they can be run similarly with `Invoke-Command`
```powershell
$projectsToBuild | %{ Invoke-Command $nugetPackCmd -ArgumentList $(Get-Location), $($_.Name), $versionPrefix, $versionSuffix }
```
