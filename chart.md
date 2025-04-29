## Daftarin semua informasi tentang chart di setting **master dashboard** (module Config - IFINSYS)

### Masuk/Login ke module Config - IFINSYS
### Cari halaman submenu Dashboard menu System Setting, click add untuk tambah data
Ada beberapa bagian yang ada di halaman Master Dashboard Info
1. General Info  
Di bagian form paling atas untuk informasi tentang dashboard secara general
2. Chart Setting  
Isinya konfigurasi chart yang menentukan gimana chartnya muncul dan apa aja yang muncul dengan chartnya.
3. Data Source  
Isinya untuk atur dari mana data dari API yang akan ditampilkan di chartnya. Ada 2 tipe source, Internal/External, pilih Internal jika ngambil dari API iFinancing, sesuaikan modulenya, untuk bagian 'API Module' diisi dengan nama controller dan nama routenya (format: '<ControllerName>/<RouteName>')

## Buat API-nya
### Start from the repo
### Next to the service
### Last but not the least, controller

## Bagian UI
### Buat 1 component = 'DashboardCollectionComponent'
- Buat folder 'DashboardCollectionComponent'
- Buat file 'DashboardCollection.razor'  
// kode

- Buat file 'DashboardCollection.razor.cs'  
// kode