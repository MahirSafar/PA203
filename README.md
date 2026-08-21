# Proqramlaşdırma Tapşırığı: "AeonGrid" — Ağıllı Enerji Şəbəkəsi və Generatorların Paylanması Sistemi

Bu layihədə siz **AeonGrid** Ağıllı Enerji İdarəetmə Kompleksinin idarəetmə simulyasiyasını yazmalısınız. Layihə şəbəkəyə qoşulan enerji mənbələrini, mühəndisləri və enerji ötürmə əməliyyatlarını idarə edir.

> **ƏSAS QAYDALAR VƏ MƏHDUDİYYƏTLƏR:**
> 1. **Dinamik kolleksiyaların (`List`, `ArrayList`, `Dictionary` və s.) istifadəsi KƏSKİN QADAĞANDIR!**
> 2. Bütün məlumatlar **Array** (massiv) daxilində saxlanılmalıdır. Massivə yeni element əlavə edildikdə onun ölçüsü `ref` və `Array.Resize` mexanizmi vasitəsilə dinamik artırılmalıdır.
> 3. Layihədə **`struct` və `record` istifadə etmək OLMAZ!** Yalnız `class` və `interface` strukturlarından istifadə edilməlidir.

---

## 1. Custom Exception-lar
Layihədə xətaların idarə edilməsi üçün aşağıdakı xüsusi istisnaları (Exception) yaradın:
* **`NotFoundException`**: Axtarılan generator, stansiya və ya mühəndis ID-sinə uyğun məlumat massivdə tapılmadıqda atılır.
* **`NotAvailableException`**: Generator təmir/texniki baxışda olduqda, mühəndis başqa tapşırıqla məşğul olduqda və ya şəbəkə xətti aktiv olmadıqda atılır.
* **`GridOverloadException`**: Generatorun istehsal etdiyi gərginlik/güc şəbəkə xəttinin daşıma limitini aşdıqda və ya cərəyan şiddəti təhlükəli həddə çatdıqda atılır.

---

## 2. Enum-lar
1. **`GridZone`**: `ZoneNorth`, `ZoneSouth`, `ZoneEast`, `ZoneWest` (Şəbəkə zonaları).
2. **`TransferStatus`**: `Scheduled`, `Transmitting`, `Stabilized`, `Interrupted` (Default: `Scheduled`).
3. **`FrequencyUnit`**: `Hertz` (Əsas), `KiloHertz`, `MegaHertz`.

---

## 3. İki Fərqli Class və Implicit / Explicit Operator Çevrilmələri
Şəbəkədə elektrik potensialını (gərginliyi) idarə etmək üçün iki fərqli class yaradın:

### `VoltVoltage` (Class)
* **`Magnitude`**: `double` property (Mənfi ola bilməz. Mənfi olarsa `GridOverloadException` atılsın).
* **Constructor**: `Magnitude` parametrini qəbul edir.

### `KiloVoltVoltage` (Class)
* **`Magnitude`**: `double` property (Mənfi ola bilməz. Mənfi olarsa `GridOverloadException` atılsın).
* **Constructor**: `Magnitude` parametrini qəbul edir.

### Operatorlar:
* `KiloVoltVoltage` üçün `VoltVoltage` tipinə **`implicit`** çevrilmə operatoru yazın ($1\text{ kV} = 1000\text{ V}$).
* `VoltVoltage` üçün `KiloVoltVoltage` tipinə **`explicit`** çevrilmə operatoru yazın ($1\text{ V} = 0.001\text{ kV}$).

---

## 4. Abstrakt Class: `PowerGenerator` (Enerji Generatoru)
* **`Id`**: `int` (`private set`, yalnız `ctor`-da avtomatik 1 vahid artır).
* **`SerialNumber`**: `string` (Boş/null ola bilməz, `Trim()` və `ToUpper()` edilməlidir).
* **`CommissionDate`**: `DateTime` (Gələcək tarix ola bilməz).
* **`OutputVoltage`**: `VoltVoltage` class tipində.
* **`TargetZone`**: `GridZone` enum tipində.
* **`IsOnline`**: `bool` (Default: `false`).
* **`Readonly / Init / Const` sahələri**:
  * `public const string GRID_CODE = "AEON-GRID-AZ"`;
  * `public readonly DateTime RegisteredAt`; (ctor-da assign edilir)
  * `public string HardwareRevision { get; init; }`
* **Abstract Method**: `double CalculateCarbonOffset(TimeSpan runTime)` — İşləmə müddətinə görə qənaət edilən/atılan karbon miqdarını (Kq ilə) hesablayır.
* **ToString()**: Override edilməli, generatorun serial nömrəsini, zonasını və gərginlik dəyərini mətn formatında qaytarmalıdır.
* *Tələb:* `SerialNumber`, `OutputVoltage` və `CommissionDate` göndərilmədən `PowerGenerator` obyekti yaratmaq mümkün olmamalıdır (Overloading constructor-lar istifadə edin).

### Törəmə Class-lar (Miras alına bilməz — `sealed` olmalıdır):
1. **`SolarGenerator`**:
   * **`PanelSurfaceArea`**: `double` (Kvadrat metr ilə, mənfi ola bilməz).
   * **`CalculateCarbonOffset(runTime)`**: $(15.5 \times \text{runTime.TotalHours}) + (\text{PanelSurfaceArea} \times 0.8)$.
2. **`WindTurbineGenerator`**:
   * **`RotorDiameter`**: `double` (Metr ilə, mənfi ola bilməz).
   * **`CalculateCarbonOffset(runTime)`**: $(22.0 \times \text{runTime.TotalHours}) + (\text{RotorDiameter} \times 1.5)$.

---

## 5. `GridEngineer` və `PowerTransferTask` Class-ları

### `GridEngineer`
* **`Id`**: Statik artan `int`.
* **`Name`**, **`Surname`**: `string` (String method-ları ilə ilk hərfləri böyüdülməlidir).
* **`HireDate`**: `DateTime`.
* **`BaseSalary`**: `double`.
* **`ClearanceZone`**: `GridZone`.
* **`IsDispatched`**: `bool` (Default: `false`).
* **Static Method**: `GetCertifiedSeniorCount(GridEngineer[] engineers, DateTime maxHireDate, double minSalary, GridZone requiredZone)`
  * İşe qəbul tarixi `maxHireDate`-dən köhnə olan (təcrübəli), icazə zonası `requiredZone`-a uyğun olan VƏ maaşı `minSalary`-dən böyük olan mühəndislərin sayını qaytarır.

### `PowerTransferTask`
* **`Id`**: Statik artan `int`.
* **`GeneratorId`**: `int`.
* **`EngineerId`**: `int`.
* **`Status`**: `TransferStatus` (Default: `Scheduled`).
* **`StartTime`**: `DateTime` (Default: `DateTime.Now`).
* **`PlannedDuration`**: `TimeSpan` (Ötürmə prosesinin nə qədər çəkəcəyi).
* **`UpdateStatus(TransferStatus newStatus)`**:
  * Əgər hazırkı status `Stabilized` və ya `Interrupted`-dirsə, status dəyişdirilə bilməz və `GridOverloadException` atılmalıdır.

---

## 6. Interfeyslər və Servislər

### `IAeonGridManager` (Interface)
* `AddGenerator(PowerGenerator generator)`: Massivə yeni generator əlavə edir (`ref` ilə massiv böyüdülür).
* `AddEngineer(GridEngineer engineer)`: Massivə yeni mühəndis əlavə edir (`ref` ilə massiv böyüdülür).
* `ScheduleTransfer(int generatorId, int engineerId, TimeSpan duration)`:
  * Generator və ya mühəndis tapılmadıqda `NotFoundException` atır.
  * Mühəndis `IsDispatched == true` və ya Generator `IsOnline == true` olarsa, `NotAvailableException` atır.
  * Mühəndisin `ClearanceZone` zolağı Generatorun `TargetZone` zolağına uyğun gəlməzsə, `GridOverloadException` atır.
  * Uğurlu olduqda mühəndisin `IsDispatched` dəyərini `true`, generatorun `IsOnline` dəyərini `true` edir.
* `CompleteTransfer(int taskId)`: Tapşırığı `Stabilized` edir, generatorun `IsOnline` dəyərini `false`, mühəndisin `IsDispatched` dəyərini `false` edir.

### `AeonGridManager` Class
* `IAeonGridManager` interfeysini tətbiq edir.
* Daxilində `PowerGenerator[]`, `GridEngineer[]` və `PowerTransferTask[]` **private massivləri** saxlayır.
* **Indexer**: `this[int index]` — `PowerTransferTask` massivinə birbaşa indekslə müraciət edib ötürmə tapşırığını `get` və `set` etməyə imkan verir.

---

## 7. Extension Metodlar
`GridExtensions` adlı static class daxilində aşağıdakı extension metodları yazın:
1. **`GetExpectedCompletionTime(this PowerTransferTask task)`**:
   * Ötürmənin təxmini bitmə vaxtını qaytarır (`task.StartTime + task.PlannedDuration`).
2. **`ConvertFrequency(double baseHertz, FrequencyUnit targetUnit)`**:
   * Generatorun daxili tezlik dəyərini (Hz) parametr olaraq göndərilən vahidə (`Hertz`, `KiloHertz`, `MegaHertz`) konvertasiya edir.
   *(Formullar: $1\text{ kHz} = 1000\text{ Hz}$, $1\text{ MHz} = 1000000\text{ Hz}$)*.

---

## 8. İnteraktiv Konsol Menyusu (Program.cs)

`Program.cs` daxilində bütün şəbəkəni idarə edən, yalnız `switch-case` və dövr məntiqinə əsaslanan geniş menyu təşkil edin:

```text
================ AEONGRID ŞƏBƏKƏ İDARƏETMƏ SİSTEMİ ================
0 - Şəbəkənin Ümumi Statusunu və Statistikanı Göstər
1 - Yeni Enerji Generatoru Əlavə Et (SolarGenerator / WindTurbineGenerator)
2 - Yeni Şəbəkə Mühəndisi Qeydiyyata Al
3 - Yeni Enerji Ötürmə Tapşırığı (PowerTransferTask) Başlat
4 - Ötürmə Tapşırığını Tamamla (Complete Transfer)
5 - Təcrübəli Mühəndisləri Filtrələ (Engineer static metodu ilə)
6 - Gərginlik Çevrilməsini Yoxla (Implicit/Explicit Volt <-> KiloVolt)
7 - Qırmızı Düymə: İndeksə Görə Tapşırıq Redaktəsi Və Tezlik Hesablanması
8 - Sistemi Dayandır
