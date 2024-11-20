# SqlClientSniRepro1

When using `OutDir` parameter for a solution containing an ASPNET Web Application,
the SNI .dlls don't get copied to `$(OutDir)\_PublishedWebsites\WebApp\bin`.

To reproduce, in VS command prompt do:

```powershell
<#  helpers to reset between builds 
rmdir .\packages -force -recurse
rmdir .\output -force -recurse
rm .\output.log
#>

# you need to set this path
$nuget = "local-path-to\nuget-6.8.0.exe"

. $nuget restore Example.sln


$msbuild = "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\amd64\MSBuild.exe"

. $msbuild /t:Build /p:Configuration=Release /p:Platform="Any CPU" Example.sln /p:OutDir=..\output > output.log
```

The SNI DLLs get copied to `\output`, but not to `\output\_PublishedWebsites\WebApp\bin`.


## Possible Fix

It seems that it's possible to fix the issue by updating `Microsoft.Data.SqlClient.SNI.targets` (in the
Microsoft.Data.SqlClient.SNI nuget package, /build/net462/Microsoft.Data.SqlClient.SNI.targets).

If a new target `_CopySNIWebApplicationLegacy` could be added after `CopySNIFiles`. Example:
```xml
  <Target Name="CopySNIFiles"
          Condition="'$(CopySNIFiles)' != 'false' And
                     '$(OutDir)' != '' And
                     HasTrailingSlash('$(OutDir)') And
                     Exists('$(OutDir)')"
          Inputs="@(SNIFiles)"
          Outputs="@(SNIFiles -> '$(OutDir)%(RecursiveDir)%(Filename)%(Extension)')">
    <!--
        NOTE: Copy "Microsoft.Data.SqlClient.SNI.dll" and all related files, for every
              architecture that we support, to the build output directory.        
    -->
    <Copy SourceFiles="@(SNIFiles)"
          DestinationFiles="@(SNIFiles -> '$(OutDir)%(RecursiveDir)%(Filename)%(Extension)')" />
  </Target>

  <Target Name="_CopySNIWebApplicationLegacy"
        AfterTargets="CopySNIFiles" 
        Condition="!$(Disable_CopyWebApplication) And '$(OutDir)' != '$(OutputPath)'">

    <Copy SourceFiles="@(SNIFiles)"
          DestinationFiles="@(SNIFiles -> '$(WebProjectOutputDir)\bin\%(RecursiveDir)%(Filename)%(Extension)')"/>
  </Target>
```