This is demonstrating dependencies working in MSBuild at the command line, but not in Visual Studio. As far as I can tell, the issue is sub-dependencies, for example:

- A (.csproj/.proj) 
  - ProjectReference to B (.vcxproj)
    - ProjectReference to C (.vcxproj)

In Visual Studio `B` is detected as a dependency for `A`, but `C` is not detected as a dependency for anything, so we have a cascading build failure because `C` wasn't allowed to produce output in time. This results in:
```
Severity	Code	Description	Project	File	Line	Suppression State
Warning	MSB8028	The intermediate directory (x64\Debug\) contains files shared from another project (Dependency.vcxproj).  This can lead to incorrect clean and rebuild behavior.	SubDependency	C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Microsoft\VC\v170\Microsoft.CppBuild.targets	517	
Error	LNK1104	cannot open file 'C:\git\NickCraver\VcxProjRefIssue\ConsumingApp\x64\Debug\SubDependency.lib'	Dependency	C:\git\NickCraver\VcxProjRefIssue\Dependency\LINK	1	
Error	MSB6006	"link.exe" exited with code 1104.	Dependency	C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Microsoft\VC\v170\Microsoft.CppCommon.targets	1096	
Error	MSB3030	Could not copy the file "C:\git\NickCraver\VcxProjRefIssue\ConsumingApp\x64\Debug\Dependency.dll" because it was not found.	ConsumingApp	C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\amd64\Microsoft.Common.CurrentVersion.targets	5150	
Error	MSB3030	Could not copy the file "C:\git\NickCraver\VcxProjRefIssue\ConsumingApp\x64\Debug\Dependency.pdb" because it was not found.	ConsumingApp	C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\amd64\Microsoft.Common.CurrentVersion.targets	5150	
Error	MSB3030	Could not copy the file "C:\git\NickCraver\VcxProjRefIssue\ConsumingApp\x64\Debug\Dependency.dll" because it was not found.	ConsumingApp	C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\amd64\Microsoft.Common.CurrentVersion.targets	5150	
```

To reproduce:
1. Download this git repo
2. `slngen.exe .\ConsumingApp\ConsumingApp.csproj`
3. Build in Visual Studio (17.5.0 as of writing this)
4. To repro in a loop, clean between builds in VS.

Note that from the command line in MSBuild, this works fine. The issue is specific to Visual Studio solutions.

