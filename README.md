# Proqramlaşdırma Tapşırığı: "AeonGrid Ultra" — Multi-Zonalı Ağıllı Enerji Şəbəkəsi və İnframaterial İdarəetmə Kompleksi

Bu layihədə siz **AeonGrid Ultra** korporativ energetika kompleksinin bütün məlumat strukturlarını, xətaları, xidmət servislərini və interaktiv idarəetmə simulyasiyasını sıfırdan qurmalısınız. Layihə şəbəkəyə qoşulan enerji generatorlarını, şəbəkə transformatorlarını, sertifikatlaşdırılmış mühəndisləri və reallıqda tətbiq edilən mürəkkəb enerji transfer əməliyyatlarını idarə edir.

> **ƏSAS QAYDALAR VƏ MƏHDUDİYYƏTLƏR:**
> 1. **Dinamik kolleksiyaların (`List<T>`, `ArrayList`, `Dictionary<K,V>`, `LinkedList` və s.) istifadəsi KƏSKİN QADAĞANDIR!**
> 2. Bütün kolleksiyalar **Array** (massiv) daxilində saxlanılmalıdır. Massivə yeni element əlavə edildikdə onun ölçüsü `ref` və `Array.Resize` mexanizmi vasitəsilə dinamik olaraq artırılmalıdır.
> 3. Layihədə **`struct` və `record` istifadə etmək OLMAZ!** Yalnız `class` və `interface` obyektlərindən istifadə edilməlidir.
> 4. Bütün məlumatlar **Encapsulation** prinsiplərinə tam uyğun olmalı, sahələr (fields) gizlədilməli və müvafiq get/set qaydaları tətbiq olunmalıdır.

---

## 1. Custom Exception-lar (Xüsusi İstisnalar)
Sistemdə baş verə biləcək fərqli xəta senariləri üçün aşağıdakı xüsusi Exception klasslarını yaradın:

* **`NotFoundException`**: Axtarılan generator, mühəndis, transformator və ya tapşırıq ID-sinə/Serial nömrəsinə uyğun obyekt massivdə tapılmadıqda atılır.
* **`NotAvailableException`**: Generator təmir rejimində olduqda, mühəndis başqa tapşırıqla məşğul olduqda və ya seçilmiş zonada təmir işləri getdikdə atılır.
* **`GridOverloadException`**: İstehsal edilən gərginlik/güc xəttin maksimal daşıma tutumunu aşdıqda və ya şəbəkə tezliyi kritik həddən çıxdıqda atılır.
* **`InsufficientClearanceException`**: Mühəndisin təhlükəsizlik icazə səviyyəsi (Clearance Level) tənzimlənən generatorun təhlükəsizlik reytinqinə uyğun gəlmədikdə atılır.

---

## 2. Enum-lar
1. **`GridZone`**: `ZoneNorth`, `ZoneSouth`, `ZoneEast`, `ZoneWest`, `ZoneCentral` (Şəbəkənin fəaliyyət zonaları).
2. **`TransferStatus`**: `Scheduled`, `Transmitting`, `Stabilized`, `Interrupted`, `Terminated` (Default: `Scheduled`).
3. **`FrequencyUnit`**: `Hertz`, `KiloHertz`, `MegaHertz`.
4. **`MaintenanceState`**: `Operational`, `UnderInspection`, `Decommissioned`.

---

## 3. İki Fərqli Class və Implicit / Explicit Operator Çevrilmələri
Şəbəkədə elektrik gərginliyini (potential difference) idarə etmək üçün iki fərqli class yaradın:

### `VoltVoltage` (Class)
* **`Magnitude`**: `double` property (Mənfi ola bilməz. Mənfi olarsa `GridOverloadException` atılsın).
* **Constructor**: `Magnitude` parametrini qəbul edir.

### `KiloVoltVoltage` (Class)
* **`Magnitude`**: `double` property (Mənfi ola bilməz. Mənfi olarsa `GridOverloadException` atılsın).
* **Constructor**: `Magnitude` parametrini qəbul edir.

### Operatorlar:
* `KiloVoltVoltage` klassı üçün `VoltVoltage` tipinə **`implicit`** çevrilmə operatoru yazın ($1\text{ kV} = 1000\text{ V}$).
* `VoltVoltage` klassı üçün `KiloVoltVoltage` tipinə **`explicit`** çevrilmə operatoru yazın ($1\text{ V} = 0.001\text{ kV}$).

---

## 4. Abstrakt Class: `PowerGenerator` (Enerji Generatoru)
* **`Id`**: `int` (`private set`, yalnız `ctor`-da avtomatik 1 vahid artır).
* **`SerialNumber`**: `string` (Boş/null ola bilməz, `Trim()` və `ToUpper()` edilməlidir).
* **`CommissionDate`**: `DateTime` (Gələcək tarix ola bilməz, gələcək tarix olarsa `ArgumentException` atılsın).
* **`OutputVoltage`**: `VoltVoltage` class tipində.
* **`TargetZone`**: `GridZone` enum tipində.
* **`State`**: `MaintenanceState` enum tipində (Default: `Operational`).
* **`IsOnline`**: `bool` (Default: `false`).
* **`Readonly / Init / Const` sahələri**:
  * `public const string GRID_CODE = "AEON-ULTRA-GRID"`;
  * `public readonly DateTime RegisteredAt`; (ctor-da assign edilir)
  * `public string HardwareRevision { get; init; }`
* **Abstract Method-lar**:
  * `double CalculateCarbonOffset(TimeSpan runTime)` — İşləmə müddətinə görə qənaət edilən/atılan karbon miqdarını (Kq ilə) hesablayır.
  * `double CalculateEfficiencyIndex()` — Generatorun ümumi faydalı iş əmsalını qaytarır.
* **ToString()**: Override edilməli, generatorun serial nömrəsini, zonasını, texniki vəziyyətini və gərginlik dəyərini oxunaqlı mətn formatında qaytarmalıdır.
* *Tələb:* `SerialNumber`, `OutputVoltage` və `CommissionDate` göndərilmədən `PowerGenerator` obyekti yaratmaq mümkün olmamalıdır (Constructor Overloading istifadə edin).

### Törəmə Class-lar (Miras alına bilməz — `sealed` olmalıdır):

1. **`SolarGenerator`**:
   * **`PanelSurfaceArea`**: `double` (Kvadrat metr ilə, mənfi və ya 0 ola bilməz).
   * **`EfficiencyPercentage`**: `double` (0 ilə 100 arasında).
   * **`CalculateCarbonOffset(runTime)`**: $(15.5 \times \text{runTime.TotalHours}) + (\text{PanelSurfaceArea} \times 0.8)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{PanelSurfaceArea} \times \text{EfficiencyPercentage}) / 100.0$.

2. **`WindTurbineGenerator`**:
   * **`RotorDiameter`**: `double` (Metr ilə, mənfi və ya 0 ola bilməz).
   * **`AverageWindSpeed`**: `double` (m/s ilə).
   * **`CalculateCarbonOffset(runTime)`**: $(22.0 \times \text{runTime.TotalHours}) + (\text{RotorDiameter} \times 1.5)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{RotorDiameter} \times \text{AverageWindSpeed}) \times 0.12$.

3. **`HydroElectricGenerator`**:
   * **`WaterFlowRate`**: `double` ($m^3/s$ ilə).
   * **`DamHeight`**: `double` (Metr ilə).
   * **`CalculateCarbonOffset(runTime)`**: $(35.0 \times \text{runTime.TotalHours}) + (\text{DamHeight} \times 2.1)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{WaterFlowRate} \times \text{DamHeight}) \times 0.08$.

---

## 5. `GridSubstation` (Şəbəkə Yarımstansiyası Klassı)
* **`Id`**: Statik artan `int`.
* **`SubstationCode`**: `string` (Format: "SUB-XXX").
* **`Zone`**: `GridZone`.
* **`MaxCapacityKiloVolts`**: `double`.
* **`ConnectedGenerators`**: `PowerGenerator[]` (Private massiv, yalnız bu stansiyaya bağlı generatorlar).
* **Metodlar**:
  * `AddGeneratorToSubstation(PowerGenerator generator)`: Massivə generator əlavə edir (`ref` ilə). Əgər stansiyanın maksimim tutumu aşılarsa `GridOverloadException` atılır.

---

## 6. `GridEngineer` və `PowerTransferTask` Class-ları

### `GridEngineer`
* **`Id`**: Statik artan `int`.
* **`Name`**, **`Surname`**: `string` (String method-ları ilə yalnız baş hərfləri böyük yazılmalıdır).
* **`HireDate`**: `DateTime`.
* **`BaseSalary`**: `double`.
* **`ClearanceZone`**: `GridZone`.
* **`ClearanceLevel`**: `int` (1 ilə 5 arasında təhlükəsizlik dərəcəsi).
* **`IsDispatched`**: `bool` (Default: `false`).
* **Static Method**: 
  * `GetCertifiedSeniorCount(GridEngineer[] engineers, DateTime maxHireDate, double minSalary, GridZone requiredZone, int minClearance)`
  * İşe qəbul tarixi `maxHireDate`-dən köhnə olan (təcrübəli), icazə zonası `requiredZone`-a uyğun olan, icazə səviyyəsi `minClearance`-dən böyük/bərabər olan VƏ maaşı `minSalary`-dən böyük olan mühəndislərin sayını qaytarır.

### `PowerTransferTask`
* **`Id`**: Statik artan `int`.
* **`GeneratorId`**: `int`.
* **`EngineerId`**: `int`.
* **`SubstationId`**: `int`.
* **`Status`**: `TransferStatus` (Default: `Scheduled`).
* **`StartTime`**: `DateTime` (Default: `DateTime.Now`).
* **`PlannedDuration`**: `TimeSpan` (Ötürmə prosesinin nə qədər çəkəcəyi).
* **Metodlar**:
  * `UpdateStatus(TransferStatus newStatus)`: Əgər hazırkı status `Stabilized`, `Interrupted` və ya `Terminated`-dirsə, status dəyişdirilə bilməz və `GridOverloadException` atılmalıdır.

---

## 7. Interfeyslər və Servislər

### `IAeonGridManager` (Interface)
* `AddGenerator(PowerGenerator generator)`: Massivə yeni generator əlavə edir (`ref` ilə massiv böyüdülür).
* `AddEngineer(GridEngineer engineer)`: Massivə yeni mühəndis əlavə edir (`ref` ilə massiv böyüdülür).
* `AddSubstation(GridSubstation substation)`: Massivə yeni stansiya əlavə edir (`ref` ilə massiv böyüdülür).
* `ScheduleTransfer(int generatorId, int engineerId, int substationId, TimeSpan duration)`:
  * Generator, mühəndis və ya stansiya tapılmadıqda `NotFoundException` atır.
  * Mühəndis `IsDispatched == true` və ya Generator `IsOnline == true` olarsa, `NotAvailableException` atır.
  * Generatorun `State` dəyəri `Operational` deyilsə, `NotAvailableException` atır.
  * Mühəndisin `ClearanceZone` zolağı Generatorun `TargetZone` zolağına uyğun gəlməzsə, `GridOverloadException` atır.
  * Uğurlu olduqda mühəndisin `IsDispatched` dəyərini `true`, generatorun `IsOnline` dəyərini `true` edir.
* `CompleteTransfer(int taskId)`: Tapşırığı `Stabilized` edir, generatorun `IsOnline` dəyərini `false`, mühəndisin `IsDispatched` dəyərini `false` edir.
* `GetGeneratorsByZone(GridZone zone)`: Verilmiş zonadakı bütün generatorları `PowerGenerator[]` massivi olaraq qaytarır.

### `AeonGridManager` Class
* `IAeonGridManager` interfeysini tətbiq edir.
* Daxilində `PowerGenerator[]`, `GridEngineer[]`, `GridSubstation[]` və `PowerTransferTask[]` **private massivləri** saxlayır.
* **Indexer 1**: `this[int index]` — `PowerTransferTask` massivinə birbaşa indekslə müraciət edib ötürmə tapşırığını `get` və `set` etməyə imkan verir.
* **Indexer 2 (Overloaded Indexer)**: `this[string serialNumber]` — Serial nömrəsinə görə `PowerGenerator` obyektini tapıb qaytarır (`get`).

---

## 8. Extension Metodlar
`GridExtensions` adlı static class daxilində aşağıdakı extension metodları yazın:

1. **`GetExpectedCompletionTime(this PowerTransferTask task)`**:
   * Ötürmənin təxmini bitmə vaxtını qaytarır (`task.StartTime + task.PlannedDuration`).
2. **`ConvertFrequency(double baseHertz, FrequencyUnit targetUnit)`**:
   * Şəbəkənin daxili tezlik dəyərini (Hz) parametr olaraq göndərilən vahidə (`Hertz`, `KiloHertz`, `MegaHertz`) konvertasiya edir.
   *(Formullar: $1\text{ kHz} = 1000\text{ Hz}$, $1\text{ MHz} = 1000000\text{ Hz}$)*.
3. **`FormatSerialNumber(this string rawSerial)`**:
   * Daxil edilən ixtiyari serial nömrə mətnini təmizləyir, aralıq boşluqları silir və "AGU-" prefiksi əlavə edir (Məsələn: `"  sol-123 "` -> `"AGU-SOL-123"`).

---

## 9. İnteraktiv Konsol Menyusu (Program.cs)

`Program.cs` daxilində bütün şəbəkə sistemini idarə edən, yalnız `switch-case` və dövr (while/do-while) məntiqinə əsaslanan geniş və funksional menyu təşkil edin:

```text
================ AEONGRID ULTRA ŞƏBƏKƏ İDARƏETMƏ SİSTEMİ ================
0 - Şəbəkənin Ümumi Statusunu, Stansiyaları və Statistikaları Göstər
1 - Yeni Enerji Generatoru Əlavə Et (Solar / Wind / Hydro)
2 - Yeni Şəbəkə Mühəndisi Qeydiyyata Al
3 - Yeni Şəbəkə Yarımstansiyası (GridSubstation) Yarad
4 - Enerji Ötürmə Tapşırığı (PowerTransferTask) Planlaşdır
5 - Ötürmə Tapşırığını Yekunlaşdır (Complete Transfer)
6 - Sertifikatlı Mühəndisləri Filtrələ (Engineer static metodu ilə)
7 - Gərginlik Çevrilməsini Yoxla (Implicit/Explicit Volt <-> KiloVolt)
8 - Qırmızı Düymə: İndeks və ya Serial ilə Generator/Tapşırıq Redaktəsi və Tezlik Hesablanması
9 - Sistemi Dayandır
