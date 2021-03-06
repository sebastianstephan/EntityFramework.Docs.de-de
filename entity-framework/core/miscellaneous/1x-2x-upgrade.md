---
title: "Aktualisieren von früheren Versionen auf EF Core 2 - Core EF"
author: divega
ms.author: divega
ms.date: 8/13/2017
ms.assetid: 8BD43C8C-63D9-4F3A-B954-7BC518A1B7DB
ms.technology: entity-framework-core
uid: core/miscellaneous/1x-2x-upgrade
ms.openlocfilehash: 380f27c9f00943a2909ec7b876e151572a67dc37
ms.sourcegitcommit: ced2637bf8cc5964c6daa6c7fcfce501bf9ef6e8
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/22/2017
---
# <a name="upgrading-applications-from-previous-versions-to-ef-core-20"></a>Aktualisieren von Anwendungen aus früheren Versionen von EF Core 2.0

## <a name="procedures-common-to-all-applications"></a>Prozeduren, die allen Anwendungen gemeinsam sind

Aktualisieren einer vorhandenen Anwendung zu EF Core 2.0 erfordern:

1. Aktualisieren die Zielplattform .NET von der Anwendung in eine Anwendung ein, die .NET Standard 2.0 unterstützt. Finden Sie unter [unterstützte Plattformen](../platforms/index.md) Weitere Details.

2. Identifizieren Sie einen Anbieter für die Zieldatenbank mit EF Core 2.0 kompatibel sind. Finden Sie unter [EF Core 2.0 erfordert einen 2.0 Datenbankanbieter](#ef-core-20-requires-a-20-database-provider) unten.

3. Aktualisieren alle EF-Core-Pakete (Common Language Runtime und die Tools sind) auf 2.0. Finden Sie unter [installieren EF Core](../get-started/install/index.md) Weitere Details.

4. Nehmen Sie Änderungen erforderlichen Code, für die aktuelle Änderungen zu kompensieren. Finden Sie unter der [Breaking Changes](#breaking-changes) weiter unten für weitere Details.

## <a name="aspnet-core-applications"></a>ASP.NET Core-Anwendungen

1. Insbesondere finden Sie in der [neues Muster für die Initialisierung der Anwendung Dienstanbieter](#new-way-of-getting-application-services) unten beschriebenen.

> [!TIP]  
> Die Übernahme von diesem neuen Muster beim Aktualisieren von Anwendungen auf 2.0 wird dringend empfohlen, und ist erforderlich, damit für Produktfeatures wie Entity Framework Core Migrationen funktioniert. Die allgemeine andere Alternative besteht darin, [implementieren *IDesignTimeDbContextFactory\<TContext >*](xref:core/miscellaneous/cli/dbcontext-creation#from-a-design-time-factory).

2. Anwendungen, deren Zielversionen für ASP.NET Core 2.0 festgelegt sind, können neben Datenbankanbietern von Drittanbietern EF Core 2.0 ohne zusätzliche Abhängigkeiten nutzen. Allerdings müssen Anwendungen, die für frühere Versionen von ASP.NET Core zu ASP.NET Core 2.0 aktualisieren, um EF Core 2.0 zu verwenden. Weitere Informationen zum Aktualisieren von ASP.NET Core Anwendungen 2.0 finden Sie unter [der ASP.NET Core-Dokumentation auf der das Subjekt](https://docs.microsoft.com/aspnet/core/migration/1x-to-2x/).

## <a name="breaking-changes"></a>Die Lauffähigkeit der Anwendung beeinträchtigende Änderungen

Wir haben die Möglichkeit, unsere vorhandener APIs und Verhaltensweisen in 2.0 erheblich optimieren ausgeführt. Es sind einige Verbesserungen, die erforderlich sein können, Ändern vorhandener Anwendungscode, obwohl wir, die für die meisten Anwendungen in der Meinung sind die Auswirkungen niedrig ist, in den meisten Fällen erfordern nur Neukompilierung und nur minimale kommentierte Änderungen an veraltete APIs zu ersetzen.

### <a name="new-way-of-getting-application-services"></a>Neue Weise gelöschte Anwendungsdienste

Das empfohlene Muster für ASP.NET Core Webanwendungen wurde für 2.0 auf eine Weise aktualisiert, die die Logik zur Entwurfszeit unterbrochen wurde, die EF Core 1.x verwendet. Zuvor zur Entwurfszeit, EF Core versuchen Sie, den aufzurufenden `Startup.ConfigureServices` direkt, um die Anwendung Dienstanbieter zugreifen. In ASP.NET Core 2.0, wird die Konfiguration außerhalb von initialisiert die `Startup` Klasse. Anwendungen, die in der Regel mithilfe von EF Core zugreifen ihre Verbindungszeichenfolge aus der Konfiguration, `Startup` allein reicht nicht mehr. Wenn Sie eine ASP.NET Core 1.x-Anwendung aktualisieren, können Sie die folgende Fehlermeldung beim EF-Core-Tools verwenden.

> Auf 'ApplicationContext' wurde kein parameterloser Konstruktor gefunden. Fügen Sie einen parameterlosen Konstruktor 'ApplicationContext' oder Hinzufügen einer Implementierung von "IDesignTimeDbContextFactory&lt;ApplicationContext&gt;" in der gleichen Assembly als "ApplicationContext"

Ein neuer zur Entwurfszeit Hook wurde in ASP.NET Core 2.0-Standardvorlage hinzugefügt. Die statische `Program.BuildWebHost` Methode ermöglicht EF-Kerne und Zugriff auf den Dienstanbieter für die Anwendung zur Entwurfszeit. Wenn Sie eine ASP.NET Core 1.x-Anwendung aktualisieren, müssen Sie Sie aktualisieren `Program` Klasse, um wie folgt aussehen.

``` csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;

namespace AspNetCoreDotNetCore2._0App
{
    public class Program
    {
        public static void Main(string[] args)
        {
            BuildWebHost(args).Run();
        }

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
}
```

### <a name="idbcontextfactory-renamed"></a>IDbContextFactory umbenannt

Um die verschiedenen Anwendungsmuster unterstützen und Benutzern besser steuern, wie ihre `DbContext` dient zur Entwurfszeit, wir haben, in der Vergangenheit bereitgestellten die `IDbContextFactory<TContext>` Schnittstelle. Zur Entwurfszeit verwendet wird, die EF Core Tools Implementierungen dieser Schnittstelle in Ihrem Projekt zu ermitteln, und verwenden sie zum Erstellen `DbContext` Objekte.

Diese Schnittstelle hat einen sehr allgemeinen Name, die einige Benutzer versuchen, verwenden es erneut für andere irreführen `DbContext`-Szenarien zu erstellen. Sie deaktivieren Guard abgefangen wird, wenn die Tools EF versucht hat, verwenden Sie ihre Implementierung zur Entwurfszeit und Befehle wie verursacht wurden `Update-Database` oder `dotnet ef database update` fehlschlägt.

Um die starke zur Entwurfszeit Semantik dieser Schnittstelle kommunizieren zu können, haben wir es umbenannt `IDesignTimeDbContextFactory<TContext>`.

Version 2.0 der `IDbContextFactory<TContext>` noch vorhanden ist, jedoch wird als veraltet markiert.

### <a name="dbcontextfactoryoptions-removed"></a>DbContextFactoryOptions entfernt

Aufgrund der oben beschriebenen Änderungen ASP.NET Core 2.0 wurde ermittelt, die `DbContextFactoryOptions` wurde nicht mehr benötigt, auf dem neuen `IDesignTimeDbContextFactory<TContext>` Schnittstelle. Hier sind die alternativen, die Sie stattdessen verwendet werden sollte.

DbContextFactoryOptions | Alternative
--- | ---
ApplicationBasePath | AppContext.BaseDirectory
ContentRootPath | Directory.GetCurrentDirectory()
EnvironmentName | Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")

### <a name="design-time-working-directory-changed"></a>Während der Entwurfszeit-Arbeitsverzeichnis geändert

Die ASP.NET Core 2.0-Änderungen muss auch von verwendeten Arbeitsverzeichnisses `dotnet ef` veröffentlichungshäufigkeit das Arbeitsverzeichnis, von Visual Studio verwendet werden, wenn die Anwendung ausgeführt. Eine Observable Nebeneffekt hiervon ist dieser SQLite Dateinamen wird jetzt relativ zum Projektverzeichnis und nicht in das Ausgabeverzeichnis sind, wie sie verwendet werden.

### <a name="ef-core-20-requires-a-20-database-provider"></a>EF Core 2.0 erfordert einen 2.0-Datenbank-Anbieter

Für EF Core 2.0 haben wir viele vereinfachungen und Verbesserungen in die Datenbankanbieter Weise funktionieren. Dies bedeutet, dass der Anbieter 1.0.x und 1.1.x mit EF Core 2.0 nicht funktionieren.

Die SQL Server- und SQLite-Anbieter werden vom Team EF ausgeliefert und 2.0-Versionen verfügbar als Teil der 2.0 freizugeben. Die Open-Source-Drittanbieter-Anbieter für [SQL Compact](https://github.com/ErikEJ/EntityFramework.SqlServerCompact), [PostgreSQL](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL), und [MySQL](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql) für 2.0 aktualisiert werden. Wenden Sie sich an den Anbieterwriter, um alle anderen Anbieter zu erhalten.

### <a name="logging-and-diagnostics-events-have-changed"></a>Protokollierung und-Diagnose Ereignisse wurden geändert.

Hinweis: diese Änderungen sollte nicht Großteil des Anwendungscodes beeinträchtigt werden.

Die Ereignis-IDs für Nachrichten an eine [ILogger](https://github.com/aspnet/Logging/blob/dev/src/Microsoft.Extensions.Logging.Abstractions/ILogger.cs) 2.0 geändert wurden. Die Ereignis-IDs sind nun für alle EF Core-Codes eindeutig. Diese Nachrichten entsprechen nun auch dem Standardmuster für die strukturierte Protokollierung, das z.B. von MVC eingesetzt wird.

Die Protokollierungskategorien haben sich ebenfalls geändert. Nun gibt es eine bekannte Gruppe von Kategorien, auf die über [DbLoggerCategory](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/DbLoggerCategory.cs) zugegriffen werden kann.

[DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) Ereignisse verwenden nun die gleichnamigen der Ereignis-ID als die entsprechenden `ILogger` Nachrichten. Die ereignisnutzlasten sind alle Nominale Typen abgeleitet [EventData](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Diagnostics/EventData.cs).

Ereignis-IDs, Nutzlasttypen und Kategorien sind dokumentiert die [CoreEventId](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Diagnostics/CoreEventId.cs) und [RelationalEventId](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore.Relational/Diagnostics/RelationalEventId.cs) Klassen.

IDs verfügen auch vom Microsoft.EntityFrameworkCore.Infraestructure auf dem neuen Microsoft.EntityFrameworkCore.Diagnostics-Namespace verschoben.

### <a name="ef-core-relational-metadata-api-changes"></a>EF Core relationalen Metadaten-API-Änderungen

EF Core 2.0 erstellt nun eine andere [IModel](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IModel.cs)-Klasse für die verschiedenen verwendeten Anbieter. Dies geschieht in der Regel auf eine für die Anwendung transparente Weise. Dies hat eine Vereinfachung der untergeordneten Metadaten-APIs ermöglicht, sodass Zugriffe auf _gemeinsame relationale Metadatenkonzepte_ immer durch Aufrufen von `.Relational` statt `.SqlServer`, `.Sqlite` usw. erfolgen. Z. B. 1.1.x Code wie folgt:

``` csharp
var tableName = context.Model.FindEntityType(typeof(User)).SqlServer().TableName;
```

Sollte jetzt wie folgt geschrieben werden:

``` csharp
var tableName = context.Model.FindEntityType(typeof(User)).Relational().TableName;
```

Statt Methoden wie z. B. `ForSqlServerToTable`, Erweiterungsmethoden stehen jetzt zur bedingten basierend auf den aktuellen Anbieter verwendet Code zu schreiben. Zum Beispiel:

```C#
modelBuilder.Entity<User>().ToTable(
    Database.IsSqlServer() ? "SqlServerName" : "OtherName");
```

Beachten Sie, die diese Änderung gilt nur für APIs-Metadaten, die für definiert ist _alle_ relationalen Anbieter. Die API und die Metadaten bleibt unverändert, wenn es nur einen einzelnen Anbieter spezifisch ist. Gruppierte Indizes sind z. B. spezifisch für SQL Server, damit `ForSqlServerIsClustered` und `.SqlServer().IsClustered()` müssen immer noch verwendet werden.

### <a name="dont-take-control-of-the-ef-service-provider"></a>Keine Kontrolle des Dienstanbieters EF

EF Kern verwendet eine interne `IServiceProvider` (d. h. einen abhängigkeitseinschleusungscontainer) für die interne Implementierung. Anwendungen sollten EF Kern zu erstellen und Verwalten von diesem Anbieter außer in besonderen Fällen zulassen. Sollten Sie dringend, entfernen alle Aufrufe von `UseInternalServiceProvider`. Wenn eine Anwendung aufrufen muss `UseInternalServiceProvider`, dann erwägen Sie [Ablegen eines Problems](https://github.com/aspnet/EntityFramework/Issues) damit wir Möglichkeiten für Ihr Szenario zu behandeln, untersuchen können.

Aufrufen von `AddEntityFramework`, `AddEntityFrameworkSqlServer`, usw. ist nicht vom Anwendungscode erforderlich, es sei denn, `UseInternalServiceProvider` wird auch als bezeichnet. Entfernen Sie alle vorhandenen Aufrufe `AddEntityFramework` oder `AddEntityFrameworkSqlServer`usw. `AddDbContext` sollte weiterhin verwendet werden auf die gleiche Weise wie zuvor.

### <a name="in-memory-databases-must-be-named"></a>In-Memory-Datenbanken müssen benannt sein

Die globalen unbenannte in-Memory-Datenbank wurde entfernt und stattdessen alle in-Memory-Datenbanken müssen benannt sein kann. Zum Beispiel:

``` csharp
optionsBuilder.UseInMemoryDatabase("MyDatabase");
```

Dies erstellt/wird eine Datenbank mit dem Namen "MyDatabase" verwendet. Wenn `UseInMemoryDatabase` erneut aufgerufen, mit dem gleichen Namen wird die gleiche in-Memory-Datenbank ermöglicht, die von mehreren Kontext Instanzen gemeinsam genutzt werden verwendet,.

### <a name="read-only-api-changes"></a>Nur-Lese API-Änderungen

`IsReadOnlyBeforeSave`, `IsReadOnlyAferSave`, und `IsStoreGeneratedAlways` akkumuliert und mit ersetzt wurden [BeforeSaveBehavior](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IProperty.cs#L39) und [AfterSaveBehavior](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IProperty.cs#L55). Dieses Verhalten gelten für jede Eigenschaft (nicht nur vom Speicher generierte Eigenschaften) und zu bestimmen, wie der Wert der Eigenschaft verwendet werden soll, beim Einfügen in eine Datenbankzeile (`BeforeSaveBehavior`) oder beim Aktualisieren einer vorhandenen Zeile (`AfterSaveBehavior`).

Eigenschaften, die als gekennzeichnet [ValueGenerated.OnAddOrUpdate](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/ValueGenerated.cs) (z. B. für berechnete Spalten) standardmäßig ignoriert ausnahmslos derzeit für die Eigenschaft festgelegt. Dies bedeutet, dass ein vom Speicher generierter Wert immer abgerufen werden kann, unabhängig davon, ob ausnahmslos festgelegt oder für die nachverfolgte Entität geändert wurde. Dies kann geändert werden, durch Festlegen einer anderen `Before\AfterSaveBehavior`.

### <a name="new-clientsetnull-delete-behavior"></a>Neue ClientSetNull löschverhaltens

In früheren Versionen [DeleteBehavior.Restrict](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/DeleteBehavior.cs) hatte ein Verhalten für Entitäten vom Kontext verfolgt, dass mehrere übereinstimmende geschlossen `SetNull` Semantik. In der EF Core 2.0 ein neues `ClientSetNull` Verhalten wurde als Standard für optionale Beziehungen eingeführt. Dieses Verhalten hat `SetNull` Semantik für überwachten Entitäten und `Restrict` Verhalten für Datenbanken mit EF Core erstellt. In unserer Erfahrung sind dies die erwartet/nützlichsten Verhaltensweisen für nachverfolgte Objekte und die Datenbank. `DeleteBehavior.Restrict`wird jetzt für überwachten Entitäten, die bei der Einstellung für optionale Beziehungen berücksichtigt werden.

### <a name="provider-design-time-packages-removed"></a>Anbieter während der Entwurfszeit Pakete entfernt

Die `Microsoft.EntityFrameworkCore.Relational.Design` Paket entfernt wurde. Der Inhalt konsolidiert wurden `Microsoft.EntityFrameworkCore.Relational` und `Microsoft.EntityFrameworkCore.Design`.

Dies wird in der Anbieter während der Entwurfszeit Pakete weitergegeben. Diese Pakete (`Microsoft.EntityFrameworkCore.Sqlite.Design`, `Microsoft.EntityFrameworkCore.SqlServer.Design`usw.) entfernt wurden und deren Inhalt in die Haupt-Anbieters Pakete zusammengefasst.

So aktivieren Sie `Scaffold-DbContext` oder `dotnet ef dbcontext scaffold` in EF Core 2.0 müssen Sie nur das Paket für die einzelnen Anbieter zu verweisen:

``` xml
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer"
    Version="2.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools"
    Version="2.0.0"
    PrivateAssets="All" />
<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet"
    Version="2.0.0" />
```
