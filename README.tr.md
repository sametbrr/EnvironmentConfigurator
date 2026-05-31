[![NuGet](https://img.shields.io/nuget/v/EnvironmentConfigurator.svg)](https://www.nuget.org/packages/EnvironmentConfigurator)
[![Downloads](https://img.shields.io/nuget/dt/EnvironmentConfigurator.svg)](https://www.nuget.org/packages/EnvironmentConfigurator)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)

# EnvironmentConfigurator

Tek çağrıyla ortama duyarlı konfigürasyon yükleyen, publish profilleri ve appsettings dosyalarını otomatik oluşturan ASP.NET Core paketi.

> 🇬🇧 For English see [README.md](README.md)

---

## Hızlı Başlangıç

```bash
dotnet add package EnvironmentConfigurator
```

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.AddEnvironmentConfiguration();   // appsettings.{Environment}.json + ortam değişkenlerini yükler
var app = builder.Build();
app.Run();
```

Kurulumdan sonraki ilk derlemede eksik config dosyaları projenize otomatik olarak oluşturulur.

---

## Özellikler

- Tek satırda `appsettings.json` → `appsettings.{Environment}.json` → ortam değişkenlerini yükler.
- Kurulumdan sonraki ilk `dotnet build` işleminde eksik konfigürasyon dosyalarını **otomatik kopyalar** (var olanları asla üzerine yazmaz).
- `dotnet add package` ile mevcut projelere saniyeler içinde entegre olur.
- **.NET 8 ve üzerini** hedefler (net9 / net10 dahil).

---

## Gereksinimler

- .NET 8.0 veya üzeri
- ASP.NET Core

---

## Kurulum

```bash
dotnet add package EnvironmentConfigurator
```

[nuget.org](https://www.nuget.org/packages/EnvironmentConfigurator) üzerinde mevcuttur.

---

## Kullanım

### Tek satır (önerilen)

```csharp
using EnvironmentConfigurator;

var builder = WebApplication.CreateBuilder(args);
builder.AddEnvironmentConfiguration();
var app = builder.Build();
app.Run();
```

`AddEnvironmentConfiguration`, .NET 8 `IHostApplicationBuilder` üzerinde çalışır; doğrudan `WebApplicationBuilder` ile kullanın.

### Esnek kullanım

Builder'a erişiminiz yoksa veya `IConfigurationBuilder` seviyesinde çalışıyorsanız:

```csharp
builder.Configuration.AddEnvironmentJsonFiles(builder.Environment);
```

### Seçenekler (`EnvironmentConfiguratorOptions`)

```csharp
builder.AddEnvironmentConfiguration(options =>
{
    options.BaseSettingsOptional        = false;   // appsettings.json yoksa hata fırlat (varsayılan: false)
    options.ReloadOnChange              = true;    // dosya değişiminde yeniden yükle (varsayılan: true)
    options.IncludeEnvironmentVariables = true;    // ortam değişkenleri kaynağını ekle (varsayılan: true)
    options.BasePath                    = "config"; // JSON dosyaları başka klasördeyse
});
```

---

## Otomatik Oluşturma (Dosya Üretimi)

Paketi kurduktan ve **ilk kez** derledikten sonra, projenizde henüz bulunmayan aşağıdaki dosyalar otomatik olarak oluşturulur:

```
appsettings.Beta.json
appsettings.Production.json
web.config
Properties/PublishProfiles/Beta-FolderProfile.pubxml
Properties/PublishProfiles/Development-FolderProfile.pubxml
Properties/PublishProfiles/Production-FolderProfile.pubxml
```

- **Var olan dosyalar asla üzerine yazılmaz** — yalnızca eksik olanlar eklenir, düzenlemeleriniz korunur.
- Oluşturulan dosyalar derleme çıktısında loglanır: `EnvironmentConfigurator: scaffolded ...`
- `.pubxml` şablonları projeye özel `ProjectGuid` içermez — her dosyada `EnvironmentName` ayarlanır.

### Otomatik oluşturmayı kapatma

`.csproj` dosyanıza ekleyin:

```xml
<PropertyGroup>
  <EnvironmentConfiguratorScaffold>false</EnvironmentConfiguratorScaffold>
</PropertyGroup>
```

---

## Ortam Seçimi

Hangi `appsettings.{X}.json` dosyasının yükleneceğine `ASPNETCORE_ENVIRONMENT` değişkeni karar verir.

| Senaryo | Nerede Ayarlanır |
|---|---|
| Debug / yerel | `Properties/launchSettings.json` → profil `environmentVariables` |
| Komut satırı | `ASPNETCORE_ENVIRONMENT=Beta dotnet run` |
| IIS / Web Deploy | `web.config` → `<environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />` |

**Override mantığı:** `appsettings.json` ortak ayarları tutar; `appsettings.{Environment}.json` yalnızca o ortam için değişen anahtarları geçersiz kılar.

`launchSettings.json` örneği:

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

Visual Studio'da publish profillerini (`.pubxml`) kullanın. Her profil ortamını `<EnvironmentName>` ile ayarlar; doğru konfigürasyon publish anında seçilir.

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

## Örnek Uygulama

[EnvironmentConfiguratorApi](EnvironmentConfiguratorApi), paketin tam çalışan canlı bir örneğidir. Paketi `ProjectReference` ile kullanır ve ortam bazlı `appsettings` dosyaları, publish profilleri ve `launchSettings.json` ile gelir. `/test/environment-name` endpoint'i aktif ortam adını döner.

```bash
cd EnvironmentConfiguratorApi
ASPNETCORE_ENVIRONMENT=Beta dotnet run
# başka bir terminalde:
curl http://localhost:5000/test/environment-name   # → "Beta"
```

---

## Proje Yapısı

```
EnvironmentConfigurator.sln
├── EnvironmentConfigurator/           → NuGet paketi (kod + scaffold hedefi + şablonlar)
├── EnvironmentConfiguratorApi/        → paketi kullanan örnek API
└── EnvironmentConfigurator.Tests/     → xUnit testleri
```

---

## Paketi Yerelde Derleme

```bash
dotnet pack EnvironmentConfigurator -c Release -o ./artifacts
# → artifacts/EnvironmentConfigurator.1.0.3.nupkg
```

---

## Lisans

MIT — bkz. [LICENSE.txt](LICENSE.txt).
