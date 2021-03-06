---
layout: post
title: "Automated ClickOnce Deployment of a WPF Application using Appveyor"
comments: true
categories: 
- clal
- tools
- wpf
tags: 
date: 2016-03-04 12:00:03 
keywords: 
description: 
---

This post covers the current deployment setup of [CLAL](https://github.com/rahulpnath/clal)(Command Line Application Launcher), a desktop application, that I am building. Since it is a WPF application, it supports [ClickOnce Deployment](https://msdn.microsoft.com/en-us/library/t71a733d.aspx) that enables to create self-updating applications which can install with minimum interaction from the user. ClickOnce supports different [deployment strategy](https://msdn.microsoft.com/en-us/library/71baz9ah.aspx) of which distributing it through the web is quite popular as it makes software distribution easier. It works well when the software size is not large so that application installation is faster. For CLAL, there are two deployments served from Azure: [Latest stable build](http://www.rahulpnath.com/clal/Releases/commandlineapplicationlauncherui.application) and the [Current build](http://www.rahulpnath.com/clal/Latest/commandlineapplicationlauncherui.application). 

I did not want to do manually, the entire deployment process of building the solution, running all the tests, creating the ClickOnce package and pushing it up to Azure, I decided to automate this. Since [Appveyor](https://www.appveyor.com/), a hosted distributed continuous integration service used to build and test projects, is free for open-source projects and integrates very well with application developed on the Windows platform.


### Setting up Appveyor project ###

Setting up Appveyor to read from Github is very easy. Once you authorize access to Github, Appveyor lists all the projects that you have in your Github account. After selecting a project, it creates a ['build project'](https://ci.appveyor.com/project/rahulpnath/clal) for that in Appveyor, where you can control all build related activities. Appveyor automatically pulls in your latest source code from the repository, when a build triggers. Build configurations can be specified using a [configuration file](https://www.appveyor.com/docs/appveyor-yml) (appveyor.yml) living at the repository root or using the user interface. For CLAL I exclusively use the configuration from the file and the latest version is available [here](https://github.com/rahulpnath/clal/blob/master/appveyor.yml). 
Primarily there are two branches (*master* and *development*) on the git repository which builds as the latest stable and current build. Since these two deployments have few attributes different (like the version numbers, deployment URL, update URL), I use [conditional build configuration](https://www.appveyor.com/docs/branches#conditional-build-configuration) to have separate configuration properties for the branches.

The primary things that vary for the different deployments are a few ClickOnce publishing properties, the version number, the build configurations - release/debug and the deployment locations. We will see in detail below how we handle this.

``` yaml
-
  branches:
    only:
      - master
  version: 0.2.2.0
  test:
    assemblies: '**\*.*Test.dll'
  configuration: Release
  # Rest of the configuration
  -
  version: 0.2.2.{build}
  test:
    assemblies: '**\*.*Test.dll'
  configuration: Debug
  # Rest of the configuration
```

### ClickOnce Publish Profile ###
To create the publish profile, I used the Visual Studio Publish option on the project, which generates all the [Publishing Properties](https://msdn.microsoft.com/en-us/library/ms165431.aspx#Anchor_2). Most of these values remain the same across all deployment version (release and development). For the ones that are unique to the deployment version like the PublishUrl, UpdateUrl, and ApplicationVersion I removed them from *csproj* file. The deployment version specific properties is set in the Appveyor configuration file and used by the build script to set the right values.


<img class="center" alt="ClickOnce publish settings" src="/images/clickonce_publishsetting.png" />

In the Appveyor configuration, the [before_build](https://www.appveyor.com/docs/build-configuration#script-blocks-in-build-configuration) step these values are set as environment variables, which gets [automatically passed into the MSBuild as Properties](http://help.appveyor.com/discussions/questions/980-custom-msbuild-property). The certificate required for signing ClickOnce manifest gets installed during this step. 

``` yaml
 before_build:
    - nuget restore src\CommandLineApplicationLauncher.sln
    - ps: "$env:ApplicationVersion=$env:APPVEYOR_BUILD_VERSION;$env:UpdateUrl='http://www.rahulpnath.com/clal/Releases/';
    $env:PublishUrl='http://www.rahulpnath.com/clal/Releases/';$mypwd = ConvertTo-SecureString -String \"/(Z&rbrFG){p/6W@8xZvg\" -Force
    –AsPlainText\nImport-PfxCertificate –FilePath
    C:\\projects\\clal\\src\\CommandLineApplicationLauncherUI\\CommandLineApplicationLauncherUI_TemporaryKey.pfx cert:\\currentuser\\my -Password $mypwd"
```

### Versioning ###
I am using [semantic versioning](http://semver.org/) and wanted to control the version numbers for the releases explicitly. Since ClickOnce supports only four digit version numbers, the last one always defaults to zero in the release version. For Current build (development) deployments, the fourth place is used to maintain the build number, so that I can support different build version in development. I use a [sequential number generated by appveyor](https://www.appveyor.com/docs/build-configuration#build-versioning) and set in the configuration file.
``` yaml
version: 0.2.2.{build}
```
For a  release I run the below script on the master branch, which updates the version number across the source code files and then push the changes to Github, which triggers a build to the updated version. Then I merge back the master into development so that the next build on development branch would be a build number off the latest released version. The script uses [ZeroToNine](https://github.com/ploeh/ZeroToNine) for updating AssemblyInfo files and updates the version numbers in the Appveyor configuration files.

``` powershell
param([Parameter(Mandatory=$true)][string]$version) 

# Update All AssemblyInfo file versions
$z29 = "./ExternalTools/ZeroToNine/Zero29.exe"
&$z29 -a $version

# Update Appveyor.yml
((Get-Content ./Appveyor.yml | Out-String) 
-replace "version: .*\.0", ("version: " + $version + ".0") 
-replace "version: .*\.{build}", ("version: " + $version + ".{build}")).Trim("`r`n") 
| Set-Content -NoNewline Appveyor.yml
```
     
### Artifacts and Deployment ###

The csproj file of the WPF application has *Publish* also as a default target, which results in a publish everytime the project is build. By default, the publish directory is in the bin folder under a subdirectory *app.publish*. Appveyor allows specifying folders as [artifacts](https://www.appveyor.com/docs/packaging-artifacts), which marks all the files under them as artifacts. The below script is for the latest stable build and marks it with a name 'releaseBuild'.

``` yaml
 after_build:
    - ps: $root = Resolve-Path .\src\CommandLineApplicationLauncherUI\bin\Release\app.publish;
     [IO.Directory]::GetFiles($root.Path, '*.*', 'AllDirectories') | % { Push-AppveyorArtifact $_ -FileName $_.Substring($root.Path.Length + 1) -DeploymentName releaseBuild }
```
Appveyor allows to [deploy using multiple providers](https://www.appveyor.com/docs/deployment) and [FTP](https://www.appveyor.com/docs/deployment/ftp) is one of them. I use this to deploy the artifcats generated to an Azure FTP from which I serve the installer. This is currently hosted on my blog domain. The password for the FTP location is [encrypted using the Appveyor tool](https://ci.appveyor.com/tools/encrypt). The below configuration pushes all the artifacts with the name 'releaseBuild' to the FTP folder. 

``` yaml
deploy:
    provider: FTP
    protocol: ftps
    host: waws-prod-sg1-003.ftp.azurewebsites.windows.net
    username: rahulpnath\rahulpnath
    password:
      secure: YOmcTqGUyjYpJOKAnOAfO30hb59cCBTy+Otlj+qrcAo=
    folder: /site/wwwroot/clal/Releases
    artifact: releaseBuild
```

With each push into the Github repository now we have Appveyor listening to it, pulling the latest source code, installing the code signing certificate for ClickOnce, building and running all tests in the project, publish the ClickOnce application, packaging and deploying this to the Azure FTP. There is a completely automated deployment pipeline and makes it easy to publish updates to [CLAL](https://github.com/rahulpnath/clal)!