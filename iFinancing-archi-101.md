iFinancing360 = **microservice.** \
Tiap service punya 2 project terpisah:

1. **UI** (Blazor)
2. **API** (ASP.NET Core)

Setiap tabel pasti punya **satu set layer lengkap**.


## API

Struktur API dibagi jadi **empat layer utama**.

### 1. Model (Business Object)

- Representasi dari struktur tabel di database.
- Model juga bisa berisi **column** dari tabel lain yang punya relasi, supaya bisa digunakan jadi **return type** di repository.
- Inherit dari `BaseModel`, yang otomatis punya **tujuh kolom umum**:
  `Id`, `CreDate`, `CreBy`, `CreIpAddress`, `ModDate`, `ModBy`, `ModIpAddress`.

Code  :
- **Format Nama File:** `[TableName].cs`.

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

- Tempat berisi kode untuk akses database langsung.
- Semua SQL statement ditulis di sini.

Code :
- **Format Nama File:** `[TableName]Repository.cs`, `I[TableName]Repository.cs`
- Tiap tabel punya **dua file**:
  - Class repository: `[TableName]Repository.cs`
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
  - Interface repository: `I[TableName]Repository.cs`
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
- Setiap function cuma boleh ada **satu** SQL statement (misal: SELECT sendiri, INSERT sendiri, dll).
```cs
#region getrows 
public async Task<List<MasterFaq>> GetRows(IDbTransaction transaction, string? keyword, int offset, int limit)
{
  var p = db.Symbol();

  string query = $@"
    select 
      id        AS ID
      ,question AS Question
      ,answer   AS Answer
      ,filename AS  FileName
      ,paths    AS  Paths
    from 
      {tableBase}
    where 
    (
      lower(question)       like  lower({p}Keyword)
      or lower(answer)      like  lower({p}Keyword)
      or lower(filename)    like  lower({p}Keyword)
      or case is_active
          when 1 then 'yes'
          else		'no'
      end					          like  lower({p}Keyword)
    )
    order by
    mod_date desc ";

  query = QueryLimitOffset(query);

  var parameters = new
  {
    Keyword = $"%{keyword}%",
    Offset = offset,
    Limit = limit
  };

  return await _command.GetRows<MasterFaq>(transaction, query, parameters);
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

  return await _command.Insert(transaction, query, model);
}
#endregion
```
**Mekanisme Query Parameter**

Saat membuat query di repository, ada konsep parameter yang perlu diperhatikan:

1. Placeholder Parameter ({p})

- {p} adalah placeholder untuk parameter yang nantinya diisi nilai saat eksekusi query.

- Contoh: where id = {p}ID berarti query akan mencari ID yang nilainya diisi dari parameter.

2. Tipe Parameter

- **Anonymous Type** → Biasanya dipakai untuk parameter sederhana.
```cs
var parameters = new { id };
var parameters = new { Keyword = "heeloooww" };
```

- **Class Object** → Biasanya pakai model yang dikirim dari UI, misalnya MasterFaq model dari form input.
```cs
return await _command.Update(transaction, query, model);
```

- Yang penting, key dalam parameter harus **sama persis (case-sensitive)** dengan yang ada di query `{p}`.

3. Parameter matching

- Saat query dieksekusi, sistem akan mencocokkan placeholder `{p}` dengan key yang ada dalam parameter.

- Jika key tidak cocok, query akan `return` null untuk value dari key itu.


### 3. Service (Business Logic Layer)

- Tempat untuk logic dari business process.
- Memanggil function dari repository untuk ambil atau manipulasi data.
- Jadi penghubung antara repository dan controller.

Code :
- **Format Nama File:** `[TableName]Service.cs`, `I[TableName]Service.cs`
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

### 4. Controller (API Layer)

- Tempat mendefinisikan **endpoint API** dan menghubungkannya ke service.
- Handle request dari UI atau service lain.

Code :
- **Format Nama File:** `[TableName]Controller.cs`
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

- **DataGrid Component** (Tabel List): `[TableName]DataGrid.razor`
- **Form Component** (Detail View): `[TableName]Form.razor`
- Tiap komponen punya dua file:
  - `.razor` (Markup/UI)
  - `.razor.cs` (Logic Komponen)

### 2. Halaman (Pages)

- Halaman terdiri dari beberapa komponen, biasanya:
  - **Satu DataGrid Component**
  - **Satu Form Component**
- Halaman juga tempat mendefinisikan **route** untuk akses UI.

## Konvensi Penamaan

- **PascalCase** untuk nama kolom di database.
- **Format Nama File:**
  - `[TableName].cs` untuk model.
  - `[TableName]Repository.cs`, `I[TableName]Repository.cs` untuk repository.
  - `[TableName]Service.cs`, `I[TableName]Service.cs` untuk service.
  - `[TableName]Controller.cs` untuk controller.
  - `[TableName]DataGrid.razor`, `[TableName]Form.razor` untuk UI components.