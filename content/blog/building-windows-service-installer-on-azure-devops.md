---
title: "Building Windows Service Installer on Azure Devops"
comments: true
categories: 
- Azure DevOps
- Azure
date: 2019-03-25
keywords: 
description: Continuosly building a windows installer on Azure DevOps using VdProj or WIX
primaryImage: azure_devops_wix.jpg
---

Recently I was looking into packaging a Windows Service as an MSI installer. I wanted the MSI created in the build pipeline, in this case [Azure DevOps](https://azure.microsoft.com/en-au/services/devops/), and publish the MSI as a build artifact. The windows service uses .Net Framework and looking around for installer options I found mainly two approaches discussed below.

### Visual Studio Installer Projects (*.VdProj)

 [Microsoft Visual Studio Installer Projects](https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2017InstallerProjects) is available as an extension to Visual Studio and provides support for Visual Studio Installer Projects in Visual Studio. By adding this setup project to the solution, you can create a setup file that steps through a wizard and installs your application. If you are looking to how to set up the Installer project, this [stackoverflow answer](https://stackoverflow.com/questions/9021075/how-to-create-an-installer-for-a-net-windows-service-using-visual-studio/9021107#9021107) shows you exactly how. Once you have the installer project set up locally and have the MSI file generated on building solution, we can set this up in Azure DevOps pipeline and automate it.

> The Visual Studio Installer Projects require a custom build agent.

The only way I could find to get the Installer Project to run and build out an MSI file was to set up a custom build agent. [Hosted agents do not support](https://developercommunity.visualstudio.com/idea/382210/add-support-to-build-installer-project-vdproj-in-a.html) this at the moment. I set the [custom agent on a Windows machine](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops) and have not tried on any of the other variants. The only tricky thing with setting up the custom agent was step 4 under [Prepare Permissions](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#permissions). To find the scope '**Agent Pools (read, manage)**', make sure you click the 'Show all/less scopes' link towards the bottom of the page (as shown in the image below) - At times some things just miss your eyes! Rest was pretty straightforward, and you can have the custom build agent set up in minutes. 

![Azure DevOps - Custom Agent token setup](/images/azure_devops_custom_agent_token.jpg)

In your build pipeline definition make sure to select the new custom agent as your default Agent pool. The [Build VS Installer](https://marketplace.visualstudio.com/items?itemName=dutchworkz.BuildInstaller#overview) is a custom task that can be used to build your Visual Studio Installer projects (.vdproj files). Since MSBuild cannot be used to build these projects you need to make sure you have Visual Studio installed on the agent with the [Installer Projects extension](https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2017InstallerProjects) installed. Setting up the custom task is straightforward - you can either choose to build just one particular installer-project in the solution or all of them in the solution.

![Azure Devops - Build Pipeline](/images/azure_devops_vdproj.jpg)

I ran into the error message *An error occurred while validating. HRESULT = '8000000A'*, when running this build through the pipeline. Soon figured out that this was [faced by others in the past](https://stackoverflow.com/questions/8648428/an-error-occurred-while-validating-hresult-8000000a/41788791#41788791). Running the *DisableOutOfProcBuild.exe* solved the issue. To do this in the pipeline add a command line task (Set EnableOutOfProcBuild step in the image above) and use the scripts based on the appropriate VS version.

Make sure to either select the 'Create artifact for .msi' option in the custom build task or manually copy it out to the artifacts directory. The build now generates an MSI every time!

### WIX

WiX is an open source project that provides a set of tools that build Windows Installation Packages. The installer packages are XML based, and the learning curve is relatively steep. However, it offers a lot more features and capabilities over the Visual Studio installer project we saw above. Microsoft hosted agents support building WIX projects, and I was able to successfully run them on the [Hosted VS2017 agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml). 

> *WIX projects can run on the Hosted VS2017 agent. Just this one reason makes WIX a far better choice over VdProj if you are starting fresh.*

![Azure Devops WIX](/images/azure_devops_wix.jpg)

If you are running on a custom build agent, you will have to install the [Wix toolset](http://wixtoolset.org/) for everything to work. The default build task in Azure DevOps is all that is required to build the project as [WIX integrates well with MSBuild](http://wixtoolset.org/documentation/manual/v3/msbuild/). As you can see WIX it is easier setup and lesser hassles, so definitely recommend using that path if you are not already with VdProj. 

Hope this helps you set up building installer projects on Azure DevOps.