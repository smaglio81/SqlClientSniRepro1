# SqlClientSniRepro1

When using `OutDir` parameter for a solution containing an ASPNET Web Application,
the SNI .dlls don't get copied to `$(OutDir)\_PublishedWebsites\WebApp\bin`.

To reproduce, in VS command prompt do:
```bat
msbuild /t:Build /p:Configuration=Release /p:Platform="Any CPU" Example.sln /p:OutDir=C:\temp\output
```

The SNI DLLs get copied to `C:\temp\output`, but not to `C:\Temp\output\_PublishedWebsites\WebApp\bin`