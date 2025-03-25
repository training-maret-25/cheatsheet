iFinancing360 = **microservice.** \
Tiap service punya 2 project terpisah:

1. **UI** (Blazor)
2. **API** (ASP.NET Core)

Setiap tabel pasti punya **satu set layer lengkap**.

## API

Struktur API dibagi jadi **empat layer utama**.

### 1. Model (Business Object)

-   Representasi dari struktur tabel di database.
-   Model juga bisa berisi **column** dari tabel lain yang punya relasi, supaya bisa digunakan jadi **return type** di repository.
-   Inherit dari `BaseModel`, yang otomatis punya **tujuh kolom umum**:
    `Id`, `CreDate`, `CreBy`, `CreIpAddress`, `ModDate`, `ModBy`, `ModIpAddress`.

Code :

-   **Format Nama File:** `[TableName].cs`.

```cs
namespace Domain.Models
{
  public class MasterFaq : BaseModel
  {
      /*
      - Wajib inherit ke BaseModel
      - Wajib implement ke interfacenya sendiri

      - ClassName wajib sama dengan FileName
      - ClassName, FileName, ColumnName wajib pakai PascalCase
      */
      public string? FileName {get; set;}
  }
}
```

### 2. Repository (Data Access Layer)

-   Tempat berisi kode untuk akses database langsung.
-   Semua SQL statement ditulis di sini.

Code :

-   **Format Nama File:** `[TableName]Repository.cs`, `I[TableName]Repository.cs`
-   Tiap tabel punya **dua file**:

    -   Class repository: `[TableName]Repository.cs`

    ```cs
    namespace Domain.Abstract.Repository
    {
      public interface IMasterFaqRepository : IBaseRepository<MasterFaq>
      {
        /*
          - Wajib inherit ke IBaseRepository
        */
      }
    }
    ```

    -   Interface repository: `I[TableName]Repository.cs`

    ```cs
    namespace DAL
    {
      public class MasterFaqRepository : BaseRepository, IMasterFaqRepository
      {
        /*
          - Wajib inherit ke BaseRepository
          - Wajib implement ke interfacenya sendiri
        */
      }
    }
    ```

-   Component penting dalam `function/method` di `repository`

```cs
// pastikan menambahkan #region & #endregion agar bisa pakai "Fold All Regions" (`ctrl` + `shift` + `p`)
#region FunctionName
// parameter wajib: IDbTransaction, sisanya optional
public async Task<int> FunctionName(IDbTransaction transaction, string id, string code)
{
  var p = db.Symbol(); // Wajib menyertakan ini jika ada placeholder di query string

  string query = $@"
    select
      id          AS  ID
      ,question   AS  Question
    from
      {tableBase}
    where
      id = {p}ID
      and code = {p}Code ";
  /*
    - Query string: tempat untuk menaruh query SQL
    - jika menggunakan alias (AS/as) harus sama persis (Case Sensitive) dengan di Model
    - untuk parameter (value dari luar), gunakan {p} sebagai placeholder
  */

  var param = new {
    ID = id,
    Code = code
  };
  /*
    - Nanti placeholder itu diisi dengan value di sini
    - Keynya harus sama persis (Case Sensitive) dengan placeholder di query string
  */

  return await _command.DeleteByID(transaction, query, param);
  // function ini biasanya terdiri dari 3 param (transaction, query, parameter)
  /*
    - Penggunaan function sesuai dengan kebutuhan fungsi dan jumlah data yang ditarik (select .. from ..)
    - Tipe function:
     GetRows<ModelName> -> ketika tarik lebih dari 1 data
     GetRow<MasterFaq> -> ketika tarik hanya 1 data
     Insert -> masukan data
     Update -> update data
     DeleteByID -> hapus data, dan khusus delete, param ke-3 hanya menerima string
  */
}
#endregion
```

-   Setiap function cuma boleh ada **satu** SQL statement (misal: SELECT sendiri, INSERT sendiri, dll).

```cs
#region getrows
public async Task<List<MasterFaq>> GetRows(IDbTransaction transaction, string? keyword, int offset, int limit)
{
  var p = db.Symbol();

  string query = $@"
    select
      id        AS ID
      ,question AS Question
      ...
    from
      {tableBase}
    where
    (
      lower(question)       like  lower({p}Keyword)
      or case is_active
          when 1 then 'yes'
          else		'no'
      end					          like  lower({p}Keyword)
      or ...
    )
    order by
    mod_date desc ";

  query = QueryLimitOffset(query);

  return await _command.GetRows<MasterFaq>(
    transaction,
    query,
    new
    {
      Keyword = $"%{keyword}%",
      Offset = offset,
      Limit = limit
    }
  );
}
#endregion

#region insert
public async Task<int> Insert(IDbTransaction transaction, MasterFaq model)
{
  var p = db.Symbol();

  string query = $@"
    insert into {tableBase}
    (
      id
      ,cre_date
      ...
    )
    values
    (
      {p}ID
      ,{p}CreDate
      ...
    )";

  return await _command.Insert(transaction, query, model);
}
#endregion
```

**Mekanisme Query Parameter**

Saat membuat query di repository, ada konsep parameter yang perlu diperhatikan:

1. Placeholder Parameter ({p})

-   {p} adalah placeholder untuk parameter yang nantinya diisi nilai saat eksekusi query.

-   Contoh: where id = {p}ID berarti query akan mencari ID yang nilainya diisi dari parameter.

2. Tipe Parameter

-   **Anonymous Type** → Biasanya dipakai untuk parameter sederhana.

```cs
var parameters = new { id };
var parameters = new { Keyword = "heeloooww" };
```

-   **Class Object** → Biasanya pakai model yang dikirim dari UI, misalnya MasterFaq model dari form input.

```cs
return await _command.Update(transaction, query, model);
```

-   Yang penting, key dalam parameter harus **sama persis (case-sensitive)** dengan yang ada di query `{p}`.

3. Parameter matching

-   Saat query dieksekusi, sistem akan mencocokkan placeholder `{p}` dengan key yang ada dalam parameter.

-   Jika key tidak cocok, query akan `return` null untuk value dari key itu.

### 3. Service (Business Logic Layer)

-   Tempat untuk logic dari business process.
-   Memanggil function dari repository untuk ambil atau manipulasi data.
-   Jadi penghubung antara repository dan controller.

Code :

-   **Format Nama File:** `[TableName]Service.cs`, `I[TableName]Service.cs`

```cs
namespace Service
{
  public class MasterFaqService : BaseService, IMasterFaqService
  {
    /*
      - Wajib inherit ke BaseService
      - Wajib implement ke interface masing-masing
    */

    private readonly IMasterFaqRepository _repo;

    public MasterFaqService(IMasterFaqRepository repo)
    {
      _repo = repo;
    }

    public async Task<List<MasterFaq>> GetRows(string? keyword, int offset, int limit)
    {
      using var connection = _repo.GetDbConnection();
      using var transaction = connection.BeginTransaction();

      /*
        - Wajib buat object connection, transaction
        - Transaction dipakai untuk :
          - context yang dikirim ke repo
          - commit transaction jika tidak ada error
          - rollback transaction jika terjadi error
      */

      try
      {
        // Business logic

        transaction.Commit();
        return result;
      }
      catch (Exception)
      {
        transaction.Rollback();
        throw;
      }
    }
  }
}
```

-   Component penting dalam `function/method` di `service`

```cs
// pastikan menambahkan #region & #endregion agar bisa pakai "Fold All Regions" (`ctrl` + `shift` + `p`)
#region FunctionName
public async Task<int> FunctionName(string param)
{
  // 2 baris kode di bawah harus ada!
  using var connection = _repo.GetDbConnection(); // ini untuk buka koneksi DB
  using var transaction = connection.BeginTransaction(); // ini untuk mengambil object transaction

  // pastikan selalu menggunakan try-catch untuk menghandle error
  try
  {
    // Your logic is here...

    transaction.Commit(); // Jika tidak ada error, maka semua query akan diexec/commit
    return countResult;
  }
  catch (Exception)
  {
    transaction.Rollback(); // Jika ada error, semua transaction/query akan di rollback
    throw;
  }
}
#endregion
```

---

### 🛠️ Default Method pada Repository & Service

Pada layer repository dan service terdapat 5 method default yang harus diimplementasikan di setiap class repository dan service:

-   `GetRows()` → Mengambil lebih dari satu data
-   `GetRowByID()` → Mengambil satu data berdasarkan ID
-   `Insert()` → Memasukkan data
-   `UpdateByID()` → Mengupdate data berdasarkan ID
-   `DeleteByID()` → Menghapus satu data berdasarkan ID

Jika ada query tambahan di repository, function definition-nya harus didaftarkan di interface repository yang sesuai. Begitu juga dengan service, jika ada logic tambahan bisa ditambahkan walaupun dalam function yang sama.

---

**Default Method di Repository (`Code Snippet`)**

-   `GetRows()`

```cs
#region getrows
public async Task<List<MasterFaq>> GetRows(IDbTransaction transaction, string? keyword, int offset, int limit)
{
  var p = db.Symbol();

  string query = $@"
    select
      id        AS ID
      ,question AS Question
      , ...
    from
      {tableBase}
    where
    (
      lower(question) like  lower({p}Keyword)
      or ...
    )
    order by
      mod_date desc ";

  /*
    Notes:
    - pastikan alias harus ada / terdaftar di model dan juga harus sama persis huruf kecil/besarnya
    - untuk kondisi, minimal ada 1 kondisi yaitu <lower(...) like lower(...)> untuk search keyword di datagrid dan harus dibungkus oleh function lower()
  */

  query = QueryLimitOffset(query);

  var parameters = new
  {
    Keyword = $"%{keyword}%", // ini dikirim dari UI
    Offset = offset, // ini dikirim dari UI
    Limit = limit // ini dikirim dari UI
  };

  /*
    Notes:
    - QueryLimitOffset() berfungsi untuk menambahkan query "offset" & "limit"
  */

  return await _command.GetRows<MasterFaq>(transaction, query, parameters);
}
#endregion
```

-   `GetRowByID()`

```cs
#region getrow
public async Task<MasterFaq> GetRowByID(IDbTransaction transaction, string id)
{
  var p = db.Symbol();

  string query = $@"
    select
      id          AS  ID
      ,question   AS  Question
      , ...
    from
      {tableBase}
    where
      id = {p}ID ";

  return await _command.GetRow<MasterFaq>(transaction, query, new { id });
}
#endregion
```

-   `Insert()`

```cs
#region insert
public async Task<int> Insert(IDbTransaction transaction, MasterFaq model)
{
  var p = db.Symbol();

  string query = $@"
    insert into {tableBase}
    (
      id
      ,cre_date
      ,cre_by
      ,cre_ip_address
      ,mod_date
      ,mod_by
      ,mod_ip_address
      ,question
      ,answer
      ,filename
      ,is_active
    )
    values
    (
      {p}ID
      ,{p}CreDate
      ,{p}CreBy
      ,{p}CreIPAddress
      ,{p}ModDate
      ,{p}ModBy
      ,{p}ModIPAddress
      ,{p}Question
      ,{p}Answer
      ,{p}FileName
      ,{p}IsActive
    )";

    /*
      Notes:
      - harus melakukan insert ke semua kolom (not nullable)
    */

  return await _command.Insert(transaction, query, model);
}
#endregion
```

-   `UpdateByID()`

```cs
#region update
public async Task<int> UpdateByID(IDbTransaction transaction, MasterFaq model)
{
  var p = db.Symbol();

  string query = $@"
    update {tableBase}
    set
      mod_date            =  {p}ModDate
      ,mod_by             =  {p}ModBy
      ,mod_ip_address     =  {p}ModIPAddress
      --
      ,question           =  {p}Question
    where
      id = {p}ID";

  /*
     Notes:
     - tidak harus semua kolom diupdate, tapi pastikan 3 kolom "mod_" itu diupdate
  */

  return await _command.Update(transaction, query, model);
}
#endregion
```

-   `DeleteByID()`

```cs
#region delete
public async Task<int> DeleteByID(IDbTransaction transaction, string id)
{
  var p = db.Symbol();

  string query = $@"
    delete from {tableBase}
    where
      id = {p}ID";

  return await _command.DeleteByID(transaction, query, id);
}
#endregion
```

---

**Default Method di Service (`Code Snippet`)**

-   `GetRows()`

```cs
#region GetRows
public async Task<List<MasterFaq>> GetRows(string? keyword, int offset, int limit)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    // Jika ada logic tambahan, silahkan saja tambahkan kodenya, yang penting ada pemanggilan function repo yang sesuai

    var result = await _repo.GetRows(transaction, keyword, offset, limit);

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
#endregion
```

-   `GetRowByID()`

```cs
#region GetRowByID
public async Task<MasterFaq> GetRowByID(string id)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    var result = await _repo.GetRowByID(transaction, id);

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
#endregion
```

-   `Insert()`

```cs
#region Insert
public async Task<int> Insert(MasterFaq model)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    int result = await _repo.Insert(transaction, model);

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
#endregion
```

-   `UpdateByID()`

```cs
#region UpdateByID
public async Task<int> UpdateByID(MasterFaq model)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    int result = await _repo.UpdateByID(transaction, model);

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
#endregion
```

-   `DeleteByID()`

```cs
#region DeleteByID
public async Task<int> DeleteByID(string[] idList)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    int countResult = 0;

    foreach (string id in idList)
    {
      var result = await _repo.DeleteByID(transaction, id);

      if (result > 0)
      {
        countResult += result;
      }
    }

    transaction.Commit();
    return countResult;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
#endregion
```

### 4. Controller (API Layer)

-   Tempat mendefinisikan **endpoint API** dan menghubungkannya ke service.
-   Handle request dari UI atau service lain.

Code :

-   **Format Nama File:** `[TableName]Controller.cs`

```cs
[HttpGet("GetRows")] // Http method + Endpoint
public async Task<ActionResult> GetRows(string? keyword, int offset, int limit)
{
  /*
    - Wajib menggunakan try-catch
    - Jika berhasil, ResponseSuccess()
    - Jika gagal, ResponseError()
  */

  try
  {
    // Controller logic

    return ResponseSuccess(data);
  }
  catch (Exception ex)
  {
    return ResponseError(ex);
  }
}
```

## UI

UI terdiri dari **komponen** dan **halaman**, yang dibuat modular biar gampang di-maintain.

### 1. Komponen

Tiap tabel minimal punya dua komponen utama:

-   **DataGrid Component** (Tabel List): `[TableName]DataGrid.razor`
-   **Form Component** (Detail View): `[TableName]Form.razor`
-   Tiap komponen punya dua file:
    -   `.razor` (Markup/UI)
    -   `.razor.cs` (Logic Komponen)

### 2. Halaman (Pages)

-   Halaman terdiri dari beberapa komponen, biasanya:
    -   **Satu DataGrid Component**
    -   **Satu Form Component**
-   Halaman juga tempat mendefinisikan **route** untuk akses UI.

## Konvensi Penamaan

-   **PascalCase** untuk nama kolom di database.
-   **Format Nama File:**
    -   `[TableName].cs` untuk model.
    -   `[TableName]Repository.cs`, `I[TableName]Repository.cs` untuk repository.
    -   `[TableName]Service.cs`, `I[TableName]Service.cs` untuk service.
    -   `[TableName]Controller.cs` untuk controller.
    -   `[TableName]DataGrid.razor`, `[TableName]Form.razor` untuk UI components.
