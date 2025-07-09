# Install SonarScanner for .NET

> ℹ️ **SonarScanner for .NET** adalah tool yang akan membaca kode C#, lalu menganalisa kode itu dan mengirim hasil analisisnya ke **SonarQube Dashboard**.

🔹 Buka Command Line dan jalankan :
```sh
dotnet tool install --global dotnet-sonarscanner
```

🔹 Cek apakah berhasil :
```sh
dotnet tool list --global
```

Jika berhasil maka akan muncul hasil seperti di bawah :
```sh
Package Id               Version      Commands
---------------------------------------------------------
dotnet-sonarscanner      10.1.2       dotnet-sonarscanner
```

## Tambahkan Konfigurasi di Proyek UI & API
Di root folder proyek, tambahkan file `SonarQube.Analysis.xml`:

```xml
<!-- UI -->
<?xml version="1.0" encoding="utf-8" ?>

<SonarQubeAnalysisProperties  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.sonarsource.com/msbuild/integration/2015/1">
  
  <Property Name="sonar.host.url">http://192.168.2.220:6500</Property>
  <Property Name="sonar.token">[token]</Property>

  <Property Name="sonar.inclusions">Program.cs,Components/**/*.cs,Pages/**/*.cs</Property>
  <Property Name="sonar.scanner.scanAll">false</Property>
 
</SonarQubeAnalysisProperties>
<!-- UI -->

<!-- API -->
<?xml version="1.0" encoding="utf-8" ?>

<SonarQubeAnalysisProperties  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.sonarsource.com/msbuild/integration/2015/1">
  
  <Property Name="sonar.host.url">http://192.168.2.220:6500</Property>
  <Property Name="sonar.token">[token]</Property>

  <Property Name="sonar.inclusions">/API/**/*.cs,/DAL/**/*.cs,/Domain/**/*.cs,/Service/**/*.cs</Property>
  <Property Name="sonar.exclusions">/API/Program.cs</Property>
  <Property Name="sonar.scanner.scanAll">false</Property>
 
</SonarQubeAnalysisProperties>
<!-- API -->
```

Tambahkan kode di bawah ini di file `.env`:
```sh
# sonarqube
SonarQube.Analysis.xml
.sonarqube/
```

Done! Sekarang, lanjut ke **Running the SonarScanner**