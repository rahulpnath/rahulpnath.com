---
title: "Generating PDF: .Net Core and Azure Web Application"
comments: true
date: 2020-03-02
description: Using NReco library to generate PDF files on Azure Web App running .Net Core.
cover: /images/net_pdf.jpg
---

Generating a PDF is one of those features that come along in a while and gets me thinking.

_How do I do this now?_

Previously I had written about [dynamically generating a large PDF from website contents](/blog/generating-a-large-pdf-from-website-contents/). The PDF library I used that then, did have the limitation of not being able to run on Azure Web App. It was because of [Azure sandbox restrictions](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#pdf-generation-from-html).

In this post we will look at how we can generate PDF in an Azure Web App and running .Net Core, what the limitations are and some tips and tricks to help with the development. I am using the [NReco HTML-to-PDF Generator for .Net](https://www.nrecosite.com/pdf_generator_net.aspx), which is a C# wrapper over [WkHtmlToPdf](https://wkhtmltopdf.org/).

> To use NReco HTML-To-PDF Generator with .Net Core, you need a [license](https://www.nrecosite.com/pdf_generator_net.aspx).

### Generating the PDF

##### Generate HTML

To generate the PDF, we first need to generate HTML. [RazorLight](https://github.com/toddams/RazorLight) is a template engine based on Razor for .Net Core. It is available as a [NuGet package](https://www.nuget.org/packages/RazorLight/). I am using the latest available pre-release version - [2.0.0-beta4](https://www.nuget.org/packages/RazorLight/2.0.0-beta4). Razor light supports templates from Files / EmbeddedResources / Strings / Database or Custom Source. The source is configured when setting the RazorLightEngine to be used in the application. For .Net Core, we can inject an instance of _IRazorLightEngine_ for use in the application. The [ContentRootPath](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostenvironment.contentrootpath?view=dotnet-plat-ext-3.1) is from the IWebHostEnvironment that can be injected to the [Startup class](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/startup?view=aspnetcore-3.1#the-startup-class)

```csharp
var engine = new RazorLightEngineBuilder()
    .UseFileSystemProject($"{ContentRootPath}/PdfTemplates")
    .UseMemoryCachingProvider()
    .Build();

services.AddSingleton<IRazorLightEngine>(engine);
```

An instance of the engine is used to generate HTML from a razor view. By using _UseFileSystemProject_ function above, RazorLight picks up the templates from the provided file path. I have all the template files under a folder 'PdfTemplates'. Make sure to set the Template files (_\*.cshtml_), and any associated resource files (CSS and images) to '_Copy to Output Directory_'. RazorLight adds the templates in the path specified and makes them available against a template key. The template key format is different [based on the source](https://github.com/toddams/RazorLight#template-sources). e.g., When using filesystem template key is the relative path to the template file from the RootPath

The HtmlGenerationService below takes in a data object and generates the HTML string using the RazorLightEngine. By convention, it expects a template file (\*.cshtml) within a folder. E.g., For data type 'Quote', it expects a template with key 'Quote/Quote.cshtml'.

```csharp
public class HtmlGenerationService : IHtmlGenerationService
{
    private readonly IRazorLightEngine _razorLightEngine;

    public HtmlGenerationService(IRazorLightEngine razorLightEngine)
    {
        _razorLightEngine = razorLightEngine;
    }
    public async Task<string> Generate<T>(T data)
    {
        var template = typeof(T).Name;
        return await _razorLightEngine.CompileRenderAsync($"{template}/{template}.cshtml", data);
    }
}
```

I got the following error - _InvalidOperationException: Cannot find reference assembly 'Microsoft.AspNetCore.Antiforgery.dll' file for package Microsoft.AspNetCore.Antiforgery_ and had to set _PreserveCompilationReferences and PreserveCompilationContext_ in the csproj as mentioned [here](https://github.com/toddams/RazorLight#im-getting-cannot-find-reference-assembly-microsoftaspnetcoreantiforgerydll-exception-on-net-core-app-30-or-higher). Make sure to check the FAQ's if you are facing any error using the library.

##### Generate PDF

With the HTML generated, we can use the HtmlToPdfConverter, the [NReco wrapper](https://www.nrecosite.com/pdf_generator_net.aspx) class, to convert it to PDF format. The library is free for .Net but needs a paid license for .Net Core. It is available as a [NuGet package](https://www.nuget.org/packages/NReco.PdfGenerator.LT/) and does work fine with .Net Core 3.1 as well.

> The [wkhtmltopdf binaries](https://wkhtmltopdf.org/downloads.html) must be deployed for your target platform(s) (Windows, Linux, or OS X) with your .NET Core app.

With .Net Core, the [wkhtmltopdf](https://wkhtmltopdf.org/) executable does not get bundled as part of the NuGet package. It is because the executable differs based on the hosting OS environment. Make sure to include the [executable](https://wkhtmltopdf.org/downloads.html) and set to be copied to the bin folder. By default, the converter looks for the executable (_wkhtmltopdf.exe_) under the folder _wkhtmltopdf_. The path is configurable.

```csharp
public class PdfGeneratorService : IPdfGeneratorService
{
    ...
    public PdfGeneratorService(
        IHtmlGenerationService htmlGenerationService, NRecoConfig config) {...}

    public async Task<byte[]> Generate<T>(T data)
    {
        var htmlContent = await HtmlGenerationService.Generate(data);
        return ToPdf(htmlContent);
    }

    private byte[] ToPdf(string htmlContent)
    {
        var htmlToPdf = new HtmlToPdfConverter();
        htmlToPdf.License.SetLicenseKey(
            _config.UserName,
            _config.License
        );
        if (RuntimeInformation.IsOSPlatform(OSPlatform.OSX))
        {
            htmlToPdf.WkHtmlToPdfExeName = "wkhtmltopdf";
        }
        return htmlToPdf.GeneratePdf(htmlContent);
    }
}
```

Calling the _GeneratePdf_ function with the HTML string returns the Pdf byte array. The Pdf byte array can be returned as a File or saved for later reference.

```csharp
public async Task<IActionResult> Get(string id)
{
    ...
    var result = await PdfGenerationService.Generate(model);
    return File(result, "application/pdf", $"Quote - {model.Number}.pdf");
}
```

### Limitations

Before using any PDF generation library, make sure you read the associated [docs and FAQ's](https://www.nrecosite.com/pdf_generator_net.aspx) as most of them have one limitation or the other. It's about finding the library that fits the purpose and budget.

**Must run on a dedicated VM backed plan** : NReco does work fine in Azure Web App as long as it in on a dedicated VM-based plan (Basic, Standard, Premium). If you are running on a Free or Shared plan, NReco will not work.

**Custom fonts are not supported** : On Azure Web App, there is a [limitation on the font's](https://feedback.azure.com/forums/169385-web-apps/suggestions/32622797-support-custom-web-fonts-in-azure-app-services). Custom fonts are ignored, and system-installed fonts are used.

**Not all Browser features available** : wkhtmltopdf uses Qt WebKit rendering engine to render the HTML into PDF. You will need to play around and see what works and what doesn't. I have seen this mostly affecting with CSS (as Flexbox and CSS Grid support was unavailalbe in the version I was using).

### Development Tips & Tricks

Here are a few things that helped speed up the development of the Razor file.

##### Render Razor View While Development

Once I had the PDF generation pipeline set up, the challenge was to get the formatting with real-time feedback. I didn't want to download the PDF and verify every time I made a change.

To see the output of the razor template as and when you make changes, return the HTML content as _ContentResult_ back on the API endpoint. When calling this from a browser, it will automatically render it.

```csharp
[HttpGet]
[Route("{id}")]
public async Task<IActionResult> Get(string id, [FromQuery]bool? html)
{   ...
    if (html.GetValueOrDefault())
    {
        var htmlResult = await HtmlGenerationService.Generate(model);
        return new ContentResult() {
            Content = htmlResult,
            ContentType = "text/html",
            StatusCode = 200 };
    }

    var result = await PdfGenerationService.Generate(model);
    return File(result, "application/pdf", $"Quote - {model.Number}.pdf");
}
```

With caching turned off ( comment out _UseMemoryCachingProvider_) and using files as the source (_UseFileSystemProject_ ), RazorLightEngineProvider will load the new file every time it renders. Any time you make a change to the razor view, refresh the API endpoint for the updated HTML.

Please make sure the final PDF looks as expected since the local browser might render the HTML different from what wkhtmltopdf uses.

##### Styles in Sass

I did not want to miss out on writing Sass for CSS but did not want to set up any automated scripts/pipeline for just the templates. [Web Compiler](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.WebCompiler), a Visual Studio extension, makes it easy to compile Sass to CSS. Once you have the extension installed, right-click on the SCSS to compile to CSS. It adds a config file to the solution, and from then on automatically compiles when the SCSS file changes.

The next time you come across a feature to generate PDF's I hope this helps you get started. The source code for this is available [here](https://github.com/rahulpnath/Blog/tree/master/PdfNetCore/PdfNetCore). Set the NRecoConfig in the appsettings.json to start creating PDF's. I hope this helps!
