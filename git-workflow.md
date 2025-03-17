# 📌 **GIT WORKFLOW CHEATSHEET**

## **1️⃣ PAGI pas baru buka VS CODE (Update Branch Sebelum Mulai Coding)**

💻 **Buka laptop → masuk VS Code → buka terminal → pastikan di `main` atau `Staging`.**

### **🔄 Tarik Update Terbaru dari Remote Repository**

```sh
[branch: main | Staging]

# Pastikan di branch utama
git switch main  # atau Staging

# Tarik update terbaru dari remote repository
git pull origin main  # atau Staging

# Tarik update terbaru dari remote repository branch masing-masing
git pull origin dev_name
```

### **🔄 Update Branch Pribadi (dev_name)**

```sh
[branch: dev_name]

# Pindah ke branch kerja masing-masing
git switch dev_name

# Tarik update terbaru dari branch dev_name
git pull origin dev_name

# Merge branch main/Staging ke branch kerja
git merge main  # atau Staging
```

👉 **Jika terjadi konflik, selesaikan di VS Code sebelum lanjut kerja!**

## **2️⃣ KETIKA NGERJAIN TASK (Commit Setiap Selesai 1 Task, Push Setelah 5 Task)**

💡 **Pastikan sekarang ada di branch kerja (`dev_name`)!**

### **🔍 Cek Status Perubahan (Setiap Selesai 1 Task)**

```sh
[branch: dev_name]

git status
```

### **📌 Commit Setiap Selesai 1 Task (Gunakan Format yang Benar!)**

```sh
[branch: dev_name]

git add .
git commit -m "Feat: Menambahkan fitur login"
git commit -m "Fix: Memperbaiki bug pada validasi input"
git commit -m "Refactor: Merapikan struktur kode pada controller"
```

❌ **TAPI BELUM PUSH!** (Push hanya setelah 5 task)

### **📤 Push Setelah 5 Task**

Jika sudah selesai 5 task (atau sesuai kesepakatan), baru push:

```sh
[branch: dev_name]

git push origin dev_name
```

## **3️⃣ SEBELUM PULANG (Push & Merge ke main/Staging)**

💻 **Buka laptop → masuk VS Code → buka terminal → pastikan di branch kerja (`dev_name`)**

### **📤 Commit & Push Perubahan Terakhir**

```sh
[branch: dev_name]

git add .
git commit -m "Feat: Final update untuk task hari ini"
git push origin dev_name
```

### **🔄 Merge ke Branch Utama (main/Staging)**

```sh
[branch: main | Staging]

# Pindah ke main/Staging
git switch main  # atau Staging

# Tarik update terbaru sebelum merge
git pull origin main  # atau Staging

# Merge branch dev_name ke main/Staging
git merge dev_name

# Cek status terakhir
git status
```

👉 **Jika ada perubahan yang belum di-<i>commit</i>:**

```sh
[branch: main | Staging]

git add .
git commit -m "Refactor: Update dari dev_name"
git push origin main  # atau Staging
```

👉 **Jika tidak ada perubahan:**

```sh
[branch: main | Staging]

git push origin main  # atau Staging
```

## **💡 Tips**

✅ **Jangan kerja langsung di `main` atau `Staging`, selalu di `dev_name`!**  
✅ **Sebelum merge, selalu pull update terbaru dari remote biar nggak konflik.**  
✅ **Kalau ada konflik, selesaikan di VS Code sebelum push.**  
✅ **Commit setiap selesai 1 task, tapi push setelah 5 task!**  
✅ **Gunakan format commit message yang benar (`Feat | Fix | Refactor: <pesan commit>`)**  
✅ **Sebelum pulang, pastikan semua sudah di-push & di-merge.**
