# 📌 Dynamic Form

Dynamic Form merupakan fitur dimana user dapat menambahkan sebuah atau beberapa field secara dinamis, tanpa perlu melakukan perubahan di <i>source code</i> dan deployment secara manual secara berulang.

## 📂 Database Schema

#### 1️⃣ Buat tabel `master_form`, `form_controls` di database.

Buat table di semua koneksi (PostgreSQL, SQL Server), Jika sudah ada check lagi struktur tablenya

```sql
CREATE TABLE master_form (
  id varchar(50) NOT NULL,
  ukey bigserial NOT NULL,
  cre_date timestamp NOT NULL,
  cre_by varchar(50) NOT NULL,
  cre_ip_address varchar(50) NULL,
  mod_date timestamp NOT NULL,
  mod_by varchar(50) NOT NULL,
  mod_ip_address varchar(50) NULL,
  "name" varchar(50) NOT NULL,
  "label" varchar(50) NOT NULL,
  is_active int4 NULL,
  display_order int4 NULL,
  code varchar(10) NULL,
  CONSTRAINT pk_master_form_id PRIMARY KEY (id),
  CONSTRAINT uq_master_form_code UNIQUE (code)
);

CREATE TABLE form_controls (
  id varchar(50) NOT NULL,
  ukey bigserial NOT NULL,
  cre_date timestamp NOT NULL,
  cre_by varchar(15) NOT NULL,
  cre_ip_address varchar(15) NOT NULL,
  mod_date timestamp NOT NULL,
  mod_by varchar(15) NOT NULL,
  mod_ip_address varchar(15) NOT NULL,
  master_form_id varchar(50) NOT NULL,
  component_name varchar(100) NOT NULL,
  "label" varchar(50) NULL,
  value varchar(50) NULL,
  visible int4 NOT NULL,
  required int4 NULL,
  disabled int4 NULL,
  area_value varchar(50) NULL,
  step varchar(10) NULL,
  min int4 NULL,
  max int4 NULL,
  "style" varchar(250) NULL,
  is_active int4 NOT NULL,
  display_order int4 NULL,
  "name" varchar(50) NULL,
  CONSTRAINT pk_form_controls_id PRIMARY KEY (id),
  CONSTRAINT fk_form_controls_master_form_id FOREIGN KEY (master_form_id) REFERENCES master_form(id)
);
```

#### 2️⃣ Buat tabel extension(ext) dari tabel header

Buat berdasarkan form/header yang akan meng-implement fitur ini. Format nama table: `[header_table_name]_ext`, contoh: menu "Client Personal Info" (table: `client_personal_info_ext`)

```sql
CREATE TABLE [header_table_name]_ext (
  id varchar(50) NOT NULL,
  cre_date timestamp NOT NULL,
  cre_by varchar(15) NOT NULL,
  cre_ip_address varchar(15) NOT NULL,
  mod_date timestamp NOT NULL,
  mod_by varchar(15) NOT NULL,
  mod_ip_address varchar(15) NOT NULL,
  --
  [header_table_name]_id varchar(50) NOT NULL,
  "keyy" varchar(50) NULL,
  value varchar(50) NULL,
  --
  CONSTRAINT pk_[header_table_name]_ext_id PRIMARY KEY (id),
  CONSTRAINT fk_[header_table_name]_ext_masterbencmark_id FOREIGN KEY ([header_table_name]_id) REFERENCES [header_table_name](id)
);
```

## ⚙️ Backend/API

### 🌍 Domain Layer

#### 1️⃣ Tambahkan model `MasterForm`, `FormControls`, dan `ExtendModel`

Class model berada pada folder `Domain/Abstract/Models`. Jika sudah ada, tidak perlu ditambahkan.

-   `MasterForm`

```csharp
namespace Domain.Models
{
  public class MasterForm : BaseModel
  {
    public string? Code { get; set; }
    public string? Name { get; set; }
    public string? Label { get; set; }
    public int? IsActive { get; set; }
  }
}
```

-   `FormControls`

```csharp
namespace Domain.Models
{
  public class FormControls : BaseModel
  {
    public string? MasterFormID { get; set; }
    public string? ComponentName { get; set; }
    public string? Label { get; set; }
    public object? Value { get; set; }
    public int? Visible { get; set; }
    public int? Required { get; set; }
    public int? Disabled { get; set; }
    public object? AreaValue { get; set; }
    public string? Step { get; set; }
    public int? Min { get; set; }
    public int? Max { get; set; }
    public string? Style { get; set; }
    public int? IsActive { get; set; }
    public string? Name { get; set; }
    public int? DisplayOrder { get; set; }
    public Dictionary<string, string>? Items { get; set; }
  }
}
```

-   `ExtendModel`

```csharp
using System.Dynamic;
namespace Domain.Models
{
  public class ExtendModel : BaseModel
  {
    public string? Key { get; set; }
    public string? Value { get; set; }
    public string? ParentID { get; set; }
    public dynamic? Properties { get; set; } = new ExpandoObject();

    public void AddProperty(string propertyName, object value)
    {
      ((IDictionary<string, object>)Properties)[propertyName] = value;
    }

    public object GetProperty(string propertyName)
    {
      if (((IDictionary<string, object>)Properties).ContainsKey(propertyName))
      {
        return ((IDictionary<string, object>)Properties)[propertyName];
      }
      else
      {
        throw new Exception($"Property {propertyName} does not exist");
      }
    }
  }
}
```

#### 2️⃣ Ubah inherit dari `BaseModel` menjadi `ExtendModel`.

Pada header class yang ingin mengimplementasi fitur ini, inheritnya diubah dari `BaseModel` menjadi `ExtendModel`. Contoh : di model `ClientPersonalInfo`.

```csharp
namespace Domain.Models
{
  public class ClientPersonalInfo : BaseModel // ❌ Before
  {
    ...
  }

  public class [HeaderTableName] : ExtendModel // ✅ After
  {
    ...
  }
}
```

#### 3️⃣ Interface Repository : `IBaseExtRepository`

Buat **interface repository** baru di folder `Domain/Abstract/Repository`

```csharp
using Domain.Models;
using System.Data;

namespace Domain.Abstract.Repository
{
  public interface IBaseExtRepository<T> where T : ExtendModel
  {
    IDbConnection GetDbConnection();
    Task<int> Insert(IDbTransaction transaction, T model);
    Task<int> UpdateByID(IDbTransaction transaction, T model);
    Task<List<T>> GetRowForParent(IDbTransaction transaction, string ParentID);
  }
}
```

#### 4️⃣ Interface Repository untuk Tabel Ekstensi

Buat **interface repository** untuk tabel ekstensi dari header yang mengimplementasikan fitur Dynamic Form. Interface ini inherit ke `IBaseExtRepository<ExtendModel>` dan berada di folder `Domain/Abstract/Repository`.

```csharp
using Domain.Models;
using System.Data;

namespace Domain.Abstract.Repository
{
  public interface I[HeaderTableName]ExtRepository : IBaseExtRepository<ExtendModel> { }
}
```

#### 4️⃣ Method `GetRowForParent`

Tambahkan method `GetRowForParent` pada **interface service** dari header yang berada di `Domain/Abstract/Service`. Contoh : service `ClientPersonalInfoService`

```csharp
using Domain.Models;

namespace Domain.Abstract.Service
{
  public interface IClientPersonalInfoService : IBaseService<ClientPersonalInfo>
  {
    ...

    Task<List<ExtendModel>> GetRowForParent(string ParentID);
  }
}
```

---

### 📂 Data Access Layer (DAL) / Repository

Buat **class repository** sebagai ekstensi dari tabel header dengan format nama `[HeaderTableName]ExtRepository` (contoh: `ClientPersonalInfoExtRepository`). Letakkan class ini di folder `DAL/` dan implementasikan interface yang telah kita buat sebelumnya.

```csharp
using System.Data;
using iFinancing360.DAL.Helper;
using Domain.Abstract.Repository;
using Domain.Models;

namespace DAL
{
  public class ClientPersonalInfoExtRepository : BaseRepository, IClientPersonalInfoExtRepository
  {
    private string tableName = "client_personal_info_ext";

    #region GetRowForParent
    public async Task<List<ExtendModel>> GetRowForParent(IDbTransaction transaction, string parentID)
    {
      var p = db.Symbol();

      string query = $@"
        select
          id                        as ID
          ,client_personal_info_id  as ParentID
          ,keyy                     as Keyy
          ,value                    as Value
        from
        {tableName}
        where
          client_personal_info_id = {p}ID";

      var result = await _command.GetRows<ExtendModel>(
          transaction,
          query,
          new { ID = parentID }
      );

      return result;
    }
    #endregion

    #region Insert
    public async Task<int> Insert(IDbTransaction transaction, ExtendModel module)
    {
      var p = db.Symbol();

      string query = $@"
        insert into {tableName}
        (
          id
          ,cre_date
          ,cre_by
          ,cre_ip_address
          ,mod_date
          ,mod_by
          ,mod_ip_address
          ,client_personal_info_id
          ,keyy
          ,value
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
          ,{p}ParentID
          ,{p}Keyy
          ,{p}Value
        )";

      return await _command.Insert(transaction, query, module);
    }
    #endregion

    #region UpdateByID
    public async Task<int>(IDbTransaction transaction, ExtendModel model)
    {
      var p = db.Symbol();

      string query = $@"
        update {tableName}
        set
          mod_date        = {p}ModDate
          ,mod_by         = {p}ModBy
          ,mod_ip_address = {p}ModIPAddress
          --
          ,value          = {p}Value
        where
          client_personal_info_id = {p}ParentID
        and
          keyy = {p}Keyy";

      return await _command.Update(transaction, query, model);
    }
    #endregion
  }
}

```

### 🔧 Service Layer

#### 1️⃣ Buat `InsertExt`, `UpdateExt`, dan `GetRowForParent`

Tambahkan pada **service** header yang meng-_implement_ fitur Dynamic From ini. `InsertExt` dan `UpdateExt` dibuat sebagai private method dan `GetRowForParent` diimplementasikan sesuai dengan interface.

-   `InsertExt()`

```csharp
#region InsertExt
private async Task<int> InsertExt(IDbTransaction transaction, dynamic tempProperties, ClientPersonalInfo model)
{
  var properties = await DeserializeProperties(tempProperties);

  if (properties == null) return 0;

  int resultExt = 0;

  foreach (var property in properties)
  {
    var extendModel = new ExtendModel
    {
      ID = Guid.NewGuid().ToString("N").ToLower(),
      CreDate = model.CreDate,
      CreBy = model.CreBy,
      CreIPAddress = model.CreIPAddress,
      ModDate = model.ModDate,
      ModBy = model.ModBy,
      ModIPAddress = model.ModIPAddress,
      ParentID = model.ID,
      Key = property.Key,
      Value = property.Value,
      Properties = null
    };

    resultExt += await _repoExt.Insert(transaction, extendModel);
  }

  return resultExt;
}
#endregion
```

-   `UpdateExt()`

```csharp
#region UpdateExt
private async Task<int> UpdateExt<T>(IDbTransaction transaction, dynamic tempProperties, List<ExtendModel> extProperties, T model) where T : ExtendModel
{
  int result = 0;

  var properties = await DeserializeProperties(tempProperties);

  foreach (var property in properties)
  {
    var keyy = property.Key;
    var value = property.Value;

    if (value is not string)
    {
      value = value.ToString();
    }

    if (extProperties.Any(e => e.Key == keyy))
    {
      var extendModel = new ExtendModel
      {
        ParentID = model.ID,
        ModDate = model.ModDate,
        ModBy = model.ModBy,
        ModIPAddress = model.ModIPAddress,
        Keyy = keyy,
        Value = value,
        Properties = null
      };

      result += await _repoExt.UpdateByID(transaction, extendModel);
    }
    else
    {
      var extendModel = new ExtendModel
      {
        ID = Guid.NewGuid().ToString("N").ToLower(),
        CreDate = model.CreDate,
        CreBy = model.CreBy,
        CreIPAddress = model.CreIPAddress,
        ModDate = model.ModDate,
        ModBy = model.ModBy,
        ModIPAddress = model.ModIPAddress,
        ParentID = model.ID,
        Keyy = keyy,
        Value = value,
        Properties = null
      };

      result += await _repoExt.Insert(transaction, extendModel);
    }
  }

  return result;
}
#endregion
```

-   `GetRowForParent()`

```csharp
#region GetRowForParent
public async Task<List<ExtendModel>> GetRowForParent(string clientID)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    // khusus client personal/corporate info
    // karena yg dikirim adalah client id, maka perlu kita ambil id terlebih dahulu
    ClientPersonalInfo model = await _repo.GetRowByClientID(transaction, clientID);

    var result = await _repoExt.GetRowForParent(transaction, model.ID);

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

#### 2. Panggil method `InsertExt` dan `UpdateExt`

Setelah 2 methodnya dibuat, panggil method `InsertExt()` di dalam `Insert()` dan `UpdateExt()` di dalam `Update()`.

-   `Insert`

```csharp
public async Task<int> Insert(...)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    ...

    result += await _repo.Insert(transaction, model);

    // Insert Dynamic Form data
    var tempProperties = model.Properties;

    model.Properties = null;

    await InsertExt(transaction, tempProperties, model); // Panggil InsertExt()

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
```

-   `UpdateByID`

```csharp
public async Task<int> UpdateByID(...)
{
  using var connection = _repo.GetDbConnection();
  using var transaction = connection.BeginTransaction();

  try
  {
    ...

    result += await _repo.UpdateByID(transaction, model);

    // Update Dynamic Form data
    var extProperties = await _repoExt.GetRowForParent(transaction, model.ID!);

    var tempProperties = model.Properties;
    model.Properties = null;

    await UpdateExt(transaction, tempProperties, extProperties, model); // Panggil UpdateExt()

    transaction.Commit();
    return result;
  }
  catch (Exception)
  {
    transaction.Rollback();
    throw;
  }
}
```

### 🎮 API Layer / Controller

#### Tambahkan Function untuk Mengambil Data Dynamic Form

Tambahkan function berikut pada controller yang mengimplementasikan fitur ini. Function ini akan digunakan untuk mengambil nilai dari dynamic form jika diisi:

```csharp
[HttpGet("GetRowByExt")]
public async Task<ActionResult> GetRowByExt(string ID)
{
  try
  {
    var data = await _service.GetRowForParent(ID);
    return ResponseSuccess(data);
  }
  catch (Exception ex)
  {
    return ResponseError(ex);
  }
}
```

## 🎨 UI

#### 1️⃣ Tambahkan Properti pada UI Logic Class dari Form Component

Tambahkan properti berikut pada UI logic class dari form component (`[HeaderTableName]Form.cs`):

```csharp
#region Variables
...

JsonObject masterForm = new();
List<FormControlsModel> controls = new();
List<ExtendModel>? extend = new();
#endregion
```

#### 2️⃣ Tambahkan Method untuk Mengambil Data Dynamic Form

Tambahkan method berikut untuk mengambil data dari dynamic form beserta valuenya:

```csharp
#region Load data for Dynamic Form
private async Task<JsonObject> LoadMasterForm(string code)
{
    var res = await IFINCMSClient.GetRow<JsonObject>("MasterForm", "GetRowByCode", new { Code = code });
    return res?.Data ?? [];
}

private async Task<List<FormControlsModel>> LoadFormControls(string formID)
{
    var res = await IFINCMSClient.GetRows<FormControlsModel>("FormControls", "GetRows", new { MasterFormID = formID });
    return res?.Data ?? [];
}
#endregion
```

#### 3️⃣ Panggil Method Sebelumnya di `OnParameterSetAsync`

Tambahkan pemanggilan method `LoadMasterForm` dan `LoadFormControls` pada `OnParameterSetAsync`. Jangan lupa untuk menambahkan `AddExtendProperty` dan `SetInitialValue`, seperti contoh berikut:

```csharp
protected override async Task OnParameterSetAsync()
{
    masterForm = await LoadMasterForm("CR");
    controls = await LoadFormControls(masterForm["ID"]?.GetValue<string>());

    if (ClientID != null)
    {
        await GetRow();

        var extRes = await IFINCMSClient.GetRows<ExtendModel>("ClientPersonalInfo", "GetRowByExt", new { ID = ClientID });
        extend = extRes?.Data;

        AddExtendProperty(controls, extend, row);
    }
    else
    {
        row = new JsonObject();
        row["Properties"] = JsonValue.Create(controls.ToDictionary(x => x.Name, x => x.Value));
    }

    SetInitialValue(row, controls);
    await base.OnInitializedAsync();
}
```

#### 4️⃣ Tambahkan Dynamic Render

Tambahkan dynamic render untuk merender field secara dinamis:

```csharp
RenderFragment Form => builder =>
{
    int seq = 0;

    foreach (var control in controls)
    {
        DynamicRenderForm(builder, ref seq, control);
    }
};
```

#### 5️⃣ Tambahkan `SetExtensionProperties` di Method `OnSubmit`

Pada method `OnSubmit`, tambahkan pemanggilan method `SetExtensionProperties`, seperti contoh berikut:

```csharp
private async void OnSubmit(JsonObject data)
{
    isLoading = true;

    data = SetAuditInfo(data);
    data = row.Merge(data);

    SetExtensionProperties(data, controls, "Properties"); // Tambahkan function ini

    if (ClientID != null)
    {
        var res = await IFINCMSClient.Put("ClientPersonalInfo", "UpdateByID", data);

        if (res != null) isLoading = false;

        StateHasChanged();
    }
    else
    {
        var res = await IFINCMSClient.Post("ClientPersonalInfo", "Insert", data);

        if (res.Data != null)
            NavigationManager.NavigateTo($"/client/clientregister/{res.Data["ID"]}");

        isLoading = false;
        StateHasChanged();
    }
}
```

#### 6️⃣ Tambahkan Variabel untuk Dynamic Form di `.razor`

Tambahkan kode berikut di file `.razor` untuk membuat dynamic form secara otomatis:

```razor
@if (controls?.Count > 0)
{
    <hr style="border: none; border-top: 2px solid var(--ifinancing-color-primary); margin: 20px 0;" />

    <RadzenStack Gap="8" Style="margin-top: 1rem;">
        <RadzenRow Gap="32" Style="margin-bottom: 5rem;">
            @Form
        </RadzenRow>
    </RadzenStack>
}
```

#### 7️⃣ Tambahkan Button untuk Menuju Halaman Pengaturan Dynamic Form

Tambahkan tombol untuk mengarahkan pengguna ke halaman pengaturan Dynamic Form:

```razor
<Button ButtonStyle="ButtonStyle.Info" Icon="settings" Text="Dynamic Form Setting" Click="@GoToSetting" />
```

Tambahkan juga method berikut untuk menangani navigasi ke halaman pengaturan Dynamic Form:

```csharp
#region Go to Dynamic Form Setting
private void GoToSetting()
{
    string masterFormID = masterForm["ID"]?.GetValue<string>();
    NavigationManager.NavigateTo($"setting/dynamicform/{masterFormID}");
}
#endregion
```
