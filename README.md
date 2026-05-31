[![NuGet](https://img.shields.io/nuget/v/EnvironmentConfigurator.svg)](https://www.nuget.org/packages/EnvironmentConfigurator)
[![Downloads](https://img.shields.io/nuget/dt/EnvironmentConfigurator.svg)](https://www.nuget.org/packages/EnvironmentConfigurator)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)

# EnvironmentConfigurator

ASP.NET Core package for single-call environment-aware config loading with auto-scaffolded publish profiles and appsettings.

> 🇹🇷 Türkçe için [README.tr.md](README.tr.md)

---

## Quick Start

```bash
dotnet add package EnvironmentConfigurator
```

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.AddEnvironmentConfiguration();   // loads appsettings.{Environment}.json + env variables
var app = builder.Build();
app.Run();
```

On the first build after install, missing config files are scaffolded automatically into your project.

---

## Features

- Loads `appsettings.json` → `appsettings.{Environment}.json` → environment variables in one line.
- On the first `dotnet build` after install, **auto-copies** missing configuration files (never overwrites existing ones).
- Integrates into existing projects in seconds via `dotnet add package`.
- Targets **.NET 8 and later** (incl. net9 / net10).

---

## Requirements

- .NET 8.0 or later
- ASP.NET Core

---

## Installation

```bash
dotnet add package EnvironmentConfigurator
```

Available on [nuget.org](https://www.nuget.org/packages/EnvironmentConfigurator).

---

## Usage

### One line (recommended)

```csharp
using EnvironmentConfigurator;

var builder = WebApplication.CreateBuilder(args);
builder.AddEnvironmentConfiguration();
var app = builder.Build();
app.Run();
```

`AddEnvironmentConfiguration` runs on the .NET 8 `IHostApplicationBuilder`; use it directly with `WebApplicationBuilder`.

### Flexible variant

If you don't have access to the builder, or work at the `IConfigurationBuilder` level:

```csharp
builder.Configuration.AddEnvironmentJsonFiles(builder.Environment);
```

### Options (`EnvironmentConfiguratorOptions`)

```csharp
builder.AddEnvironmentConfiguration(options =>
{
    options.BaseSettingsOptional        = false;   // throw if appsettings.json is missing (default: false)
    options.ReloadOnChange              = true;    // reload on file change (default: true)
    options.IncludeEnvironmentVariables = true;    // add the env variables source (default: true)
    options.BasePath                    = "config"; // if JSON files live in another folder
});
```

---

## Auto-scaffold (file generation)

After installing the package and building **for the first time**, the following files not already present in your project are generated automatically:

```
appsettings.Beta.json
appsettings.Production.json
web.config
Properties/PublishProfiles/Beta-FolderProfile.pubxml
Properties/PublishProfiles/Development-FolderProfile.pubxml
Properties/PublishProfiles/Production-FolderProfile.pubxml
```

- **Existing files are never overwritten** — only missing ones are added, your edits are preserved.
- Generated files are logged in the build output: `EnvironmentConfigurator: scaffolded ...`
- The `.pubxml` templates contain no project-specific `ProjectGuid` — `EnvironmentName` is set in each file.

### Disabling scaffold

Add to your `.csproj`:

```xml
<PropertyGroup>
  <EnvironmentConfiguratorScaffold>false</EnvironmentConfiguratorScaffold>
</PropertyGroup>
```

---

## Environment selection

The `ASPNETCORE_ENVIRONMENT` variable decides which `appsettings.{X}.json` is loaded.

| Scenario | Where to set it |
|---|---|
| Debug / local | `Properties/launchSettings.json` → profile `environmentVariables` |
| Command line | `ASPNETCORE_ENVIRONMENT=Beta dotnet run` |
| IIS / Web Deploy | `web.config` → `<environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />` |

**Override logic:** `appsettings.json` holds shared settings; `appsettings.{Environment}.json` overrides only the keys that change for that environment.

`launchSettings.json` example:

```json
{
  "profiles": {
    "Beta": {
      "commandName": "Project",
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:5001;http://localhost:5000",
      "environmentVariables": { "ASPNETCORE_ENVIRONMENT": "Beta" }
    }
  }
}
```

---

## Publish / Web Deploy

In Visual Studio, use the publish profiles (`.pubxml`). Each profile sets its environment via `<EnvironmentName>`; the correct configuration is selected at publish time.

```xml
<Project>
  <PropertyGroup>
    <EnvironmentName>Beta</EnvironmentName>
    <PublishProvider>FileSystem</PublishProvider>
    <PublishUrl>bin\Release\net8.0\publish\</PublishUrl>
    <WebPublishMethod>FileSystem</WebPublishMethod>
    <TargetFramework>net8.0</TargetFramework>
    <SelfContained>false</SelfContained>
  </PropertyGroup>
</Project>
```

---

## Sample application

[EnvironmentConfiguratorApi](EnvironmentConfiguratorApi) is a fully working live example. It consumes the package via `ProjectReference` and ships with per-environment `appsettings` files, publish profiles and `launchSettings.json`. The `/test/environment-name` endpoint returns the active environment name.

```bash
cd EnvironmentConfiguratorApi
ASPNETCORE_ENVIRONMENT=Beta dotnet run
# in another terminal:
curl http://localhost:5000/test/environment-name   # → "Beta"
```

---

## Project structure

```
EnvironmentConfigurator.sln
├── EnvironmentConfigurator/           → NuGet package (code + scaffold target + templates)
├── EnvironmentConfiguratorApi/        → sample API consuming the package
└── EnvironmentConfigurator.Tests/     → xUnit tests
```

---

## Building the package locally

```bash
dotnet pack EnvironmentConfigurator -c Release -o ./artifacts
# → artifacts/EnvironmentConfigurator.1.0.3.nupkg
```

---

## License

MIT — see [LICENSE.txt](LICENSE.txt).
