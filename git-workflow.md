# 📌 **GIT WORKFLOW CHEATSHEET**

## **1️⃣ PAGI pas baru buka VS CODE (Update Branch Sebelum Mulai Coding)**

💻 **Buka laptop → masuk VS Code → buka terminal → pastikan di `Staging`.**

### **🔄 Tarik Update Terbaru dari Remote Repository**

```sh
[branch: Staging]

# Pastikan di branch utama
git switch Staging

# Tarik update terbaru dari remote repository
git pull origin Staging

# Tarik update terbaru dari remote repository branch masing-masing
git pull origin magang_name
```

### **🔄 Update Branch Pribadi (magang_name)**

```sh
[branch: magang_name]

# Pindah ke branch kerja masing-masing
git switch magang_name

# Tarik update terbaru dari branch magang_name
git pull origin magang_name

# Merge branch Staging ke branch kerja
git merge Staging
```

👉 **Jika terjadi konflik, selesaikan di VS Code sebelum lanjut kerja!**

## **2️⃣ KETIKA NGERJAIN TASK (Commit Setiap Selesai 1 Task, Push Setelah 5 Task)**

💡 **Pastikan sekarang ada di branch kerja (`magang_name`)!**

### **🔍 Cek Status Perubahan (Setiap Selesai 1 Task)**

```sh
[branch: magang_name]

git status
```

### **📌 Commit Setiap Selesai 1 Task (Gunakan Format yang Benar!)**

```sh
[branch: magang_name]

git add .
git commit -m "Feat: Menambahkan fitur login"
git commit -m "Fix: Memperbaiki bug pada validasi input"
git commit -m "Refactor: Merapikan struktur kode pada controller"
```

❌ **TAPI BELUM PUSH!** (Push hanya setelah 5 task)

### **📤 Push Setelah 5 Task**

Jika sudah selesai 5 task (atau sesuai kesepakatan), baru push:

```sh
[branch: magang_name]

git push origin magang_name
```

## **3️⃣ SEBELUM PULANG (Push & Merge ke Staging)**

💻 **Buka laptop → masuk VS Code → buka terminal → pastikan di branch kerja (`magang_name`)**

### **📤 Commit & Push Perubahan Terakhir**

```sh
[branch: magang_name]

git add .
git commit -m "Feat: Final update untuk task hari ini"
git push origin magang_name
```

### **🔄 Merge ke Branch Utama (Staging)**

```sh
[branch: Staging]

# Pindah ke Staging
git switch Staging

# Tarik update terbaru sebelum merge
git pull origin Staging

# Merge branch magang_name ke Staging
git merge magang_name

# Cek status terakhir
git status
```

👉 **Jika ada perubahan yang belum di-<i>commit</i>:**

```sh
[branch: Staging]

git add .
git commit -m "Refactor: Update dari magang_name"
git push origin Staging
```

👉 **Jika tidak ada perubahan:**

```sh
[branch: Staging]

git push origin Staging
```

## **💡 Tips**

✅ **Jangan kerja langsung di `Staging`, selalu di `magang_name`!**  
✅ **Sebelum merge, selalu pull update terbaru dari remote biar nggak konflik.**  
✅ **Kalau ada konflik, selesaikan di VS Code sebelum push.**  
✅ **Commit setiap selesai 1 task, tapi push setelah 5 task!**  
✅ **Gunakan format commit message yang benar (`Feat | Fix | Refactor: <pesan commit>`)**  
✅ **Sebelum pulang, pastikan semua sudah di-push & di-merge.**