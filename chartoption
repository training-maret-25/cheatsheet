# 📊 `ChartOption` Documentation

### Overview

`ChartOption` adalah struktur data yang digunakan untuk merepresentasikan konfigurasi chart (Bar, Column, Line, Spline, dll) di dashboard web app. Struktur ini fleksibel dan mendukung berbagai jenis chart dengan data tunggal maupun multi-series, termasuk fitur drilldown.

---

## 🧩 Class Definition

```csharp
public class ChartOption
{
  public string? ID { get; set; }                     // Unique identifier (optional)
  public string? Name { get; set; }                   // Title or label of the chart or series
  public IEnumerable<string>? Categories { get; set; } // X-axis labels (e.g., months, categories)
  public IEnumerable<decimal>? Data { get; set; }     // Y-axis values (can be used with or without Series)
  public IEnumerable<ChartOption>? Series { get; set; } // Series of charts (if multi-series)
  public string? DrilldownID { get; set; }            // Optional drilldown reference ID
}
```

---

## 🧠 Rules & Patterns by Chart Type

### 📦 Bar / Column Chart

#### 1. **Simple Bar/Column (Single Series)**

-   Digunakan: `Categories` + `Data`
-   Keterangan: `Series = null`, Satu nilai untuk tiap kategori.

```csharp
ChartOption option = new()
{
  Name = "Monthly Sales",
  Categories = ["Jan", "Feb", ..., "Dec"],
  Data = [1, 2, ..., 12],
  Series = null
};
```

#### 2. **Bar/Column with Series (Satu Data per Series)**

-   Digunakan: `Categories` + beberapa `Series` (masing-masing dengan `Data.Count == 1`)
-   Keterangan: Cocok untuk perbandingan antar objek. Bisa dikasih nama atau drilldown per batang.

```csharp
ChartOption option = new()
{
  Name = "Bar Comparison",
  Categories = ["Jan", ..., "Dec"],
  Series = [
    new ChartOption { Name = "Bar 1", Data = [1] },
    new ChartOption { Name = "Bar 2", Data = [2] },
    ...
  ]
};
```

#### 3. **Bar/Column with Series (Multiple Data Points per Series)**

-   Digunakan: `Categories` + beberapa `Series` (masing-masing punya beberapa`Data`)
-   Keterangan: Tiap series merepresentasikan kumpulan data lengkap (misalnya antar departemen)

```csharp
ChartOption option = new()
{
  Name = "Department Sales",
  Categories = ["Jan", ..., "Dec"],
  Series = [
    new ChartOption { Name = "Dept A", Data = [1,2,3,...] },
    new ChartOption { Name = "Dept B", Data = [4,5,6,...] },
    ...
  ]
};
```

---

### 📈 Line / Spline Chart

#### 1. **Simple Line (Single Series)**

-   Digunakan: `Categories` + `Data`
-   Keterangan: `Series = null`. Cocok untuk trend tunggal.

```csharp
ChartOption option = new()
{
  Name = "Trend 2025",
  Categories = ["Jan", ..., "Dec"],
  Data = [1, 2, ..., 12]
};
```

#### 2. **Single Line Series with Drilldown**

-   Digunakan: `Categories` + one `Series`
-   Keterangan: Buat satu line dengan extra config.

```csharp
ChartOption option = new()
{
  Categories = ["Jan", ..., "Dec"],
  Series = [
    new ChartOption { Name = "Revenue", Data = [1,2,...], DrilldownID = "revenue_detail" }
  ]
};
```

#### 3. **Multiple Line Series**

-   Digunakan: `Categories` + multiple `Series`
-   Keterangan: Setiap series adalah garis sendiri. Cocok untuk membandingkan beberapa tren sekaligus.

```csharp
ChartOption option = new()
{
  Categories = ["Jan", ..., "Dec"],
  Series = [
    new ChartOption { Name = "Product A", Data = [...] },
    new ChartOption { Name = "Product B", Data = [...] },
    ...
  ]
};
```

---

## 🔍 Notes

-   `DrilldownID` digunakan untuk mengarahkan user ke chart detail saat klik pada elemen chart (misalnya: klik bar tertentu).
-   `Series` bisa digunakan untuk nested data, memungkinkan komposisi chart yang kompleks (multi-bar/multi-line).
-   `Data` hanya digunakan jika `Series` kosong (single series chart).
-   `Categories` wajib ada untuk semua chart yang berbasis X-axis (Bar, Line, dll).

---

## 🚀 Best Practices

| Case                             | Categories | Data | Series        | Drilldown |
| -------------------------------- | ---------- | ---- | ------------- | --------- |
| Bar chart sederhana              | ✅         | ✅   | ❌            | ❌        |
| Bar chart per batang             | ✅         | ❌   | ✅ (data = 1) | Optional  |
| Bar stacked chart                | ✅         | ❌   | ✅ (data > 1) | Optional  |
| Line chart tunggal               | ✅         | ✅   | ❌            | ❌        |
| Line chart satu dengan drilldown | ✅         | ❌   | ✅ (1 series) | ✅        |
| Multi-line chart                 | ✅         | ❌   | ✅ (multi)    | Optional  |

---
