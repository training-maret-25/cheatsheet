# ▶️ Running the SonarScanner

> ℹ️ **Diperhatikan sebelumn menjalankan SonarQube!**  
Make sure ketika ingin menjalankan command, pindah ke branch Staging  
Sudah melakukan step-by-step Instalasi SonarQube for .Net SonarQube

### 1. Jalankan SonarScanner di CLI Local/PC  
Di level root folder proyek UI/API , di branch Staging, buka terminal, lalu jalankan command ini di root:
```sh
dotnet sonarscanner begin -v:"360.0.1" -k:"<PROJECT_KEY>" -s:"<path_to_SonarQube.Analysis.xml>"
dotnet build
dotnet sonarscanner end
```

> 📌 **Breakdown Command**  
-k:"<PROJECT_KEY>"  
-s:"<path_to_SonarQube.Analysis.xml>" → File path yang mengarahkan ke file config SonarQube.Analysis.xml (Klik kanan > Copy Path)

### 2. Cek Hasil Analisis  
Ada 2 cara untuk melakukan check hasil analisis :  
🔹 Dashboard
- Buka SonarQube Server
- Pilih Project (ex. IFINTMS UI)
- Cek hasil analisis