---
layout: post
title: dotnet build version numbers and powershell
---

Spent some time getting local nuget build scripts in the custom NEventStore repo and OneEnterprise.  
It was surprisingly difficult to find answers for how to set the version number for a dotnet core build.  
I ended up learning that dotnet commands can be pass /p:<propertyname>=<value> for most all build/project properties
e.g.
```powershell
dotnet pack .\$folderName --output ..\nuget_build /p:VersionPrefix=$($version.AssemblySemVer) --version-suffix $($version.PreReleaseTag)
```
some other notes:  
* our `.csproj` files show we're using `VersionPrefix` (not `Version`) as the primary version property.

* Powershell won't string interpolate nested properties as you'd expect by default.
In order for a nested property to be interpolated the whole thing needs to be wrapped by `$()`
