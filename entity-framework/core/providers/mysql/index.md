---
title: "MySQL-Datenbankanbieter – EF Core"
author: rowanmiller
ms.author: divega
ms.date: 10/27/2016
ms.assetid: 4900b882-79c5-40d2-a44a-ccb0292f6ed9
ms.technology: entity-framework-core
uid: core/providers/mysql/index
ms.openlocfilehash: 1500d017cb463c3f394131a79b9063ff90cce5e2
ms.sourcegitcommit: ced2637bf8cc5964c6daa6c7fcfce501bf9ef6e8
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/22/2017
---
# <a name="mysql-ef-core-database-provider"></a>MySQL-EF Core-Datenbankanbieter

Dieser Datenbankanbieter ermöglicht die Verwendung von Entity Framework Core mit MySQL. Der Anbieter wird im Rahmen des [MySQL-Projekts](http://dev.mysql.com) verwaltet.

> [!WARNING]  
> Bei diesem Anbieter handelt es sich um ein Vorabrelease.

> [!NOTE]  
> Dieser Anbieter wird nicht im Rahmen des Entity Framework Core-Projekts verwaltet. Wenn Sie einen Drittanbieter in Betracht ziehen, sollten Sie Aspekte wie Qualität, Lizenzierung und Support auswerten, um sicherzustellen, dass dieser Ihren Anforderungen entspricht.

## <a name="install"></a>Installieren

Installieren Sie das [NuGet-Paket „MySql.Data.EntityFrameworkCore“](https://www.nuget.org/packages/MySql.Data.EntityFrameworkCore).

``` powershell
Install-Package MySql.Data.EntityFrameworkCore -Pre
```

## <a name="get-started"></a>Erste Schritte

Weitere Informationen finden Sie unter [Starting with MySQL EF Core provider and Connector/Net 7.0.4](http://insidemysql.com/howto-starting-with-mysql-ef-core-provider-and-connectornet-7-0-4/) (Erste Schritte mit dem MySQL EF Core-Anbieter und Connector/Net 7.0.4).

## <a name="supported-database-engines"></a>Unterstützte Datenbank-Engines

* MySQL

## <a name="supported-platforms"></a>Unterstützte Plattformen

* .NET Framework (4.5.1 oder höher)

* .NET Core

Informieren Sie sich in der MySQL-Dokumentation [hier](https://dev.mysql.com/doc/connector-net/en/connector-net-versions.html) und [hier](https://dev.mysql.com/doc/connector-net/en/connector-net-entityframework-core.html) über Informationen zur Versionskompatibilität.
