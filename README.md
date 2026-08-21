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



























Gemini
New chat
Search chats
Images
Library
Перевод задания AeonGrid Ultra
C# və Proqramlaşdırma Sual-Cavabları
Generated Regex Method Implementation Error
Bilgisayar Dillerinin Evrimi: Bootstrapping
Proqramlaşdırma Mövzuları: Sual-Cavab
Markdown faylını paylaşma platformları
Proqramlaşdırma Mövzuları Dərinlikli Sorğusu
Veri Doğruluğu ve Tutarlılık Talebi
İkinin Qüvvətini Tapan C# Alqoritmi
CV Düzəlişləri Üçün AI Prompt
CV Göndərildikdən Sonra Referans Təsdiqi
C# Proqramlaşdırma: Başlanğıc Sual-Cavab
Dəyişənlərin dəyərini dəyişmək
JavaScript Sayı Problemleri ve Çözümleri
CV Analysis Prompt Translation
İş Axtaran Üçün Uyğun Status Seçimi
Askerlik Durumu Terimleri: Deferred ve Exempt
Google Hesabı Giriş və Qeydiyyat
Stripe İnteqrasiyası Onion Architecture
C# Stripe Test Ödəniş İnteqrasiyası
Onion MVC Abunəlik Sistemi Qurulumu
OBS-də video həcmini azaltma ayarları
Şifrəni unutdum funksiyası əlavə etmək
Coffee Equipment Product Descriptions
Frontend Agent SEO Prompt Məsuliyyəti
Product Description Generation Request
Məhsul adlarının qısaldılması
Product Categorization for Coffee Equipment
Conversation with Gemini
rus dilnde yaz eynisin:



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

Вот точный перевод всей спецификации программирования «AeonGrid Ultra» на русский язык:

Задание по программированию: «AeonGrid Ultra» — Мультизональная умная энергосеть и комплекс управления инфраструктурными материалами
В этом проекте вам предстоит с нуля разработать все структуры данных, исключения, сервисные службы и интерактивную симуляцию управления для корпоративного энергетического комплекса AeonGrid Ultra. Проект управляет подключаемыми к сети генераторами энергии, сетевыми трансформаторами (подстанциями), сертифицированными инженерами и сложными операциями передачи энергии, применяемыми в реальности.

ОСНОВНЫЕ ПРАВИЛА И ОГРАНИЧЕНИЯ:

Использование динамических коллекций (List<T>, ArrayList, Dictionary<K,V>, LinkedList и т. д.) СТРОГО ЗАПРЕЩЕНО!

Все коллекции должны храниться внутри стандартных массивов (Array). При добавлении нового элемента в массив его размер должен динамически увеличиваться с использованием механизма ref и Array.Resize.

В проекте НЕЛЬЗЯ использовать struct и record! Разрешено использовать только объекты class и interface.

Все данные должны строго соответствовать принципам инкапсуляции (Encapsulation): поля (fields) должны быть скрыты, а для доступа к ним должны применяться соответствующие правила get/set.

1. Custom Exceptions (Пользовательские исключения)
Создайте следующие классы пользовательских исключений для обработки различных ошибочных сценариев в системе:

NotFoundException: Вызывается, если искомый объект (генератор, инженер, подстанция или задача) по его ID/серийному номеру не найден в массиве.

NotAvailableException: Вызывается, если генератор находится на техническом обслуживании, инженер занят другой задачей или в выбранной зоне ведутся ремонтные работы.

GridOverloadException: Вызывается, если вырабатываемое напряжение/мощность превышает максимальную пропускную способность линии или если частота сети выходит за критические пределы.

InsufficientClearanceException: Вызывается, если уровень допуска безопасности инженера (Clearance Level) не соответствует рейтингу безопасности регулируемого генератора.

2. Enum (Перечисления)
GridZone: ZoneNorth, ZoneSouth, ZoneEast, ZoneWest, ZoneCentral (Зоны деятельности энергосети).

TransferStatus: Scheduled, Transmitting, Stabilized, Interrupted, Terminated (По умолчанию: Scheduled).

FrequencyUnit: Hertz, KiloHertz, MegaHertz.

MaintenanceState: Operational, UnderInspection, Decommissioned.

3. Два разных класса и явные / неявные преобразования операторов (Implicit / Explicit)
Создайте два различных класса для управления электрическим напряжением (разностью потенциалов) в сети:

VoltVoltage (Class)
Magnitude: double свойство (Не может быть отрицательным. Если отрицательное — вызывается GridOverloadException).

Constructor: Принимает параметр Magnitude.

KiloVoltVoltage (Class)
Magnitude: double свойство (Не может быть отрицательным. Если отрицательное — вызывается GridOverloadException).

Constructor: Принимает параметр Magnitude.

Операторы:
Для класса KiloVoltVoltage напишите оператор implicit (неявного) преобразования к типу VoltVoltage (1 кВ=1000 В).

Для класса VoltVoltage напишите оператор explicit (явного) преобразования к типу KiloVoltVoltage (1 В=0.001 кВ).

4. Абстрактный класс: PowerGenerator (Генератор энергии)
Id: int (private set, автоматически увеличивается на 1 только в ctor).

SerialNumber: string (Не может быть пустым/null, должны применяться Trim() и ToUpper()).

CommissionDate: DateTime (Не может быть датой из будущего, если дата из будущего — вызывается ArgumentException).

OutputVoltage: Тип класса VoltVoltage.

TargetZone: Тип enum GridZone.

State: Тип enum MaintenanceState (По умолчанию: Operational).

IsOnline: bool (По умолчанию: false).

Поля Readonly / Init / Const:

public const string GRID_CODE = "AEON-ULTRA-GRID";

public readonly DateTime RegisteredAt; (присваивается значение в ctor)

public string HardwareRevision { get; init; }

Абстрактные методы:

double CalculateCarbonOffset(TimeSpan runTime) — Рассчитывает количество сэкономленного/сокращенного углерода (в кг) в зависимости от времени работы.

double CalculateEfficiencyIndex() — Возвращает общий коэффициент полезного действия (КПД) генератора.

ToString(): Должен быть переопределен (override) и возвращать серийный номер генератора, зону, техническое состояние и значение напряжения в читаемом текстовом формате.

Требование: Нельзя создать объект PowerGenerator без передачи SerialNumber, OutputVoltage и CommissionDate (Используйте перегрузку конструкторов — Constructor Overloading).

Дочерние классы (Не могут быть унаследованы — должны быть sealed):
SolarGenerator:

PanelSurfaceArea: double (Площадь поверхности панелей в кв. метрах, не может быть отрицательной или равной 0).

EfficiencyPercentage: double (От 0 до 100).

CalculateCarbonOffset(runTime): (15.5×runTime.TotalHours)+(PanelSurfaceArea×0.8).

CalculateEfficiencyIndex(): (PanelSurfaceArea×EfficiencyPercentage)/100.0.

WindTurbineGenerator:

RotorDiameter: double (Диаметр ротора в метрах, не может быть отрицательным или равным 0).

AverageWindSpeed: double (Средняя скорость ветра в м/с).

CalculateCarbonOffset(runTime): (22.0×runTime.TotalHours)+(RotorDiameter×1.5).

CalculateEfficiencyIndex(): (RotorDiameter×AverageWindSpeed)×0.12.

HydroElectricGenerator:

WaterFlowRate: double (Расход воды в м³/с).

DamHeight: double (Высота плотины в метрах).

CalculateCarbonOffset(runTime): (35.0×runTime.TotalHours)+(DamHeight×2.1).

CalculateEfficiencyIndex(): (WaterFlowRate×DamHeight)×0.08.

5. GridSubstation (Класс сетевой подстанции)
Id: Статический автоинкрементный int.

SubstationCode: string (Формат: "SUB-XXX").

Zone: GridZone.

MaxCapacityKiloVolts: double.

ConnectedGenerators: PowerGenerator[] (Приватный массив, только подключенные к этой подстанции генераторы).

Методы:

AddGeneratorToSubstation(PowerGenerator generator): Добавляет генератор в массив (с использованием ref). Если максимальная емкость подстанции превышена, вызывается GridOverloadException.

6. Классы GridEngineer и PowerTransferTask
GridEngineer
Id: Статический автоинкрементный int.

Name, Surname: string (С помощью строковых методов первая буква обязательно должна быть заглавной).

HireDate: DateTime.

BaseSalary: double.

ClearanceZone: GridZone.

ClearanceLevel: int (Уровень допуска безопасности от 1 до 5).

IsDispatched: bool (По умолчанию: false).

Static Method:

GetCertifiedSeniorCount(GridEngineer[] engineers, DateTime maxHireDate, double minSalary, GridZone requiredZone, int minClearance)

Возвращает количество инженеров, дата приема на работу которых старше maxHireDate (опытные), зона допуска соответствует requiredZone, уровень допуска больше или равен minClearance И зарплата больше minSalary.

PowerTransferTask
Id: Статический автоинкрементный int.

GeneratorId: int.

EngineerId: int.

SubstationId: int.

Status: TransferStatus (По умолчанию: Scheduled).

StartTime: DateTime (По умолчанию: DateTime.Now).

PlannedDuration: TimeSpan (Запланированное время выполнения процесса передачи).

Методы:

UpdateStatus(TransferStatus newStatus): Если текущий статус Stabilized, Interrupted или Terminated, изменить статус нельзя и должно быть вызвано исключение GridOverloadException.

7. Интерфейсы и Сервисы
IAeonGridManager (Interface)
AddGenerator(PowerGenerator generator): Добавляет новый генератор в массив (массив расширяется через ref).

AddEngineer(GridEngineer engineer): Добавляет нового инженера в массив (массив расширяется через ref).

AddSubstation(GridSubstation substation): Добавляет новую подстанцию в массив (массив расширяется через ref).

ScheduleTransfer(int generatorId, int engineerId, int substationId, TimeSpan duration):

Если генератор, инженер или подстанция не найдены, вызывается NotFoundException.

Если инженер IsDispatched == true или генератор IsOnline == true, вызывается NotAvailableException.

Если значение State генератора отличается от Operational, вызывается NotAvailableException.

Если зона допуска инженера (ClearanceZone) не совпадает с целевой зоной генератора (TargetZone), вызывается GridOverloadException.

При успехе устанавливает IsDispatched инженера в true и IsOnline генератора в true.

CompleteTransfer(int taskId): Переводит задачу в статус Stabilized, меняет IsOnline генератора на false и IsDispatched инженера на false.

GetGeneratorsByZone(GridZone zone): Возвращает все генераторы в заданной зоне в виде массива PowerGenerator[].

AeonGridManager Class
Реализует интерфейс IAeonGridManager.

Содержит внутри себя приватные массивы: PowerGenerator[], GridEngineer[], GridSubstation[] и PowerTransferTask[].

Индексатор 1: this[int index] — Позволяет обращаться непосредственно к массиву PowerTransferTask по индексу для получение (get) и установки (set) задачи передачи.

Индексатор 2 (Перегруженный индексатор): this[string serialNumber] — Находит и возвращает объект PowerGenerator по серийному номеру (get).

8. Методы расширения (Extension Methods)
Внутри статического класса GridExtensions напишите следующие методы расширения:

GetExpectedCompletionTime(this PowerTransferTask task):

Возвращает ожидаемое время завершения передачи (task.StartTime + task.PlannedDuration).

ConvertFrequency(double baseHertz, FrequencyUnit targetUnit):

Конвертирует внутреннее значение частоты сети (Гц) в единицу измерения, переданную в качестве параметра (Hertz, KiloHertz, MegaHertz).
(Формулы: 1 кГц=1000 Гц, 1 МГц=1000000 Гц).

FormatSerialNumber(this string rawSerial):

Очищает произвольный текст серийного номера, удаляет лишние пробелы и добавляет префикс "AGU-" (Например: "  sol-123 " -> "AGU-SOL-123").

9. Интерактивное консольное меню (Program.cs)
Внутри Program.cs организуйте развернутое и функциональное меню, управляющее всей энергосистемой и основанное исключительно на логике switch-case и циклов (while/do-while):

Plaintext
================ СИСТЕМА УПРАВЛЕНИЯ ЭНЕРГОСЕТЬЮ AEONGRID ULTRA ================
0 - Показать общее состояние сети, подстанции и статистику
1 - Добавить новый генератор энергии (Solar / Wind / Hydro)
2 - Зарегистрировать нового инженера сети
3 - Создать новую сетевую подстанцию (GridSubstation)
4 - Запланировать задачу передачи энергии (PowerTransferTask)
5 - Завершить задачу передачи (Complete Transfer)
6 - Отфильтровать сертифицированных инженеров (через статический метод Engineer)
7 - Проверить преобразование напряжения (Implicit/Explicit Volt <-> KiloVolt)
8 - Красная кнопка: Редактирование генератора/задачи по индексу или серийному номеру и расчет частоты
9 - Остановить систему
md formatinda yaz

"""

with open("AeonGrid_Ultra_Task.md", "w", encoding="utf-8") as f:
f.write(md_content)

print("MD file generated successfully.")


```text?code_stdout&code_event_index=1
MD file generated successfully.

Ваш файл Markdown (.md) с техническим заданием готов:

MD icon
AeonGrid_Ultra_Task
 MD 
Open
Ниже приведено полное содержимое спецификации в формате Markdown:

Markdown
# Программирование: «AeonGrid Ultra» — Мультизональная умная энергосеть и комплекс управления инфраструктурными материалами

В этом проекте вам предстоит с нуля разработать все структуры данных, исключения, сервисные службы и интерактивную симуляцию управления для корпоративного энергетического комплекса **AeonGrid Ultra**. Проект управляет подключаемыми к сети генераторами энергии, сетевыми трансформаторами (подстанциями), сертифицированными инженерами и сложными операциями передачи энергии, применяемыми в реальности.

> **ОСНОВНЫЕ ПРАВИЛА И ОГРАНИЧЕНИЯ:**
> 1. **Использование динамических коллекций (`List<T>`, `ArrayList`, `Dictionary<K,V>`, `LinkedList` и т. д.) СТРОГО ЗАПРЕЩЕНО!**
> 2. Все коллекции должны храниться внутри стандартных массивов (**Array**). При добавлении нового элемента в массив его размер должен динамически увеличиваться с использованием механизма `ref` и `Array.Resize`.
> 3. В проекте **НЕЛЬЗЯ использовать `struct` и `record`!** Разрешено использовать только объекты `class` и `interface`.
> 4. Все данные должны строго соответствовать принципам инкапсуляции (**Encapsulation**): поля (fields) должны быть скрыты, а для доступа к ним должны применяться соответствующие правила get/set.

---

## 1. Custom Exceptions (Пользовательские исключения)
Создайте следующие классы пользовательских исключений для обработки различных ошибочных сценариев в системе:

* **`NotFoundException`**: Вызывается, если искомый объект (генератор, инженер, подстанция или задача) по его ID/серийному номеру не найден в массиве.
* **`NotAvailableException`**: Вызывается, если генератор находится на техническом обслуживании, инженер занят другой задачей или в выбранной зоне ведутся ремонтные работы.
* **`GridOverloadException`**: Вызывается, если вырабатываемое напряжение/мощность превышает максимальную пропускную способность линии или если частота сети выходит за критические пределы.
* **`InsufficientClearanceException`**: Вызывается, если уровень допуска безопасности инженера (Clearance Level) не соответствует рейтингу безопасности регулируемого генератора.

---

## 2. Enum (Перечисления)
1. **`GridZone`**: `ZoneNorth`, `ZoneSouth`, `ZoneEast`, `ZoneWest`, `ZoneCentral` (Зоны деятельности энергосети).
2. **`TransferStatus`**: `Scheduled`, `Transmitting`, `Stabilized`, `Interrupted`, `Terminated` (По умолчанию: `Scheduled`).
3. **`FrequencyUnit`**: `Hertz`, `KiloHertz`, `MegaHertz`.
4. **`MaintenanceState`**: `Operational`, `UnderInspection`, `Decommissioned`.

---

## 3. Два разных класса и явные / неявные преобразования операторов (Implicit / Explicit)
Создайте два различных класса для управления электрическим напряжением (разностью потенциалов) в сети:

### `VoltVoltage` (Class)
* **`Magnitude`**: `double` свойство (Не может быть отрицательным. Если отрицательное — вызывается `GridOverloadException`).
* **Constructor**: Принимает параметр `Magnitude`.

### `KiloVoltVoltage` (Class)
* **`Magnitude`**: `double` свойство (Не может быть отрицательным. Если отрицательное — вызывается `GridOverloadException`).
* **Constructor**: Принимает параметр `Magnitude`.

### Операторы:
* Для класса `KiloVoltVoltage` напишите оператор **`implicit`** (неявного) преобразования к типу `VoltVoltage` ($1\text{ кВ} = 1000\text{ В}$).
* Для класса `VoltVoltage` напишите оператор **`explicit`** (явного) преобразования к типу `KiloVoltVoltage` ($1\text{ В} = 0.001\text{ кВ}$).

---

## 4. Абстрактный класс: `PowerGenerator` (Генератор энергии)
* **`Id`**: `int` (`private set`, автоматически увеличивается на 1 только в `ctor`).
* **`SerialNumber`**: `string` (Не может быть пустым/null, должны применяться `Trim()` и `ToUpper()`).
* **`CommissionDate`**: `DateTime` (Не может быть датой из будущего, если дата из будущего — вызывается `ArgumentException`).
* **`OutputVoltage`**: Тип класса `VoltVoltage`.
* **`TargetZone`**: Тип enum `GridZone`.
* **`State`**: Тип enum `MaintenanceState` (По умолчанию: `Operational`).
* **`IsOnline`**: `bool` (По умолчанию: `false`).
* **Поля `Readonly / Init / Const`**:
  * `public const string GRID_CODE = "AEON-ULTRA-GRID"`;
  * `public readonly DateTime RegisteredAt`; (присваивается значение в ctor)
  * `public string HardwareRevision { get; init; }`
* **Абстрактные методы**:
  * `double CalculateCarbonOffset(TimeSpan runTime)` — Рассчитывает количество сэкономленного/сокращенного углерода (в кг) в зависимости от времени работы.
  * `double CalculateEfficiencyIndex()` — Возвращает общий коэффициент полезного действия (КПД) генератора.
* **ToString()**: Должен быть переопределен (`override`) и возвращать серийный номер генератора, зону, техническое состояние и значение напряжения в читаемом текстовом формате.
* *Требование:* Нельзя создать объект `PowerGenerator` без передачи `SerialNumber`, `OutputVoltage` и `CommissionDate` (Используйте перегрузку конструкторов — Constructor Overloading).

### Дочерние классы (Не могут быть унаследованы — должны быть `sealed`):

1. **`SolarGenerator`**:
   * **`PanelSurfaceArea`**: `double` (Площадь поверхности панелей в кв. метрах, не может быть отрицательной или равной 0).
   * **`EfficiencyPercentage`**: `double` (От 0 до 100).
   * **`CalculateCarbonOffset(runTime)`**: $(15.5 \times \text{runTime.TotalHours}) + (\text{PanelSurfaceArea} \times 0.8)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{PanelSurfaceArea} \times \text{EfficiencyPercentage}) / 100.0$.

2. **`WindTurbineGenerator`**:
   * **`RotorDiameter`**: `double` (Диаметр ротора в метрах, не может быть отрицательным или равным 0).
   * **`AverageWindSpeed`**: `double` (Средняя скорость ветра в м/с).
   * **`CalculateCarbonOffset(runTime)`**: $(22.0 \times \text{runTime.TotalHours}) + (\text{RotorDiameter} \times 1.5)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{RotorDiameter} \times \text{AverageWindSpeed}) \times 0.12$.

3. **`HydroElectricGenerator`**:
   * **`WaterFlowRate`**: `double` (Расход воды в м³/с).
   * **`DamHeight`**: `double` (Высота плотины в метрах).
   * **`CalculateCarbonOffset(runTime)`**: $(35.0 \times \text{runTime.TotalHours}) + (\text{DamHeight} \times 2.1)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{WaterFlowRate} \times \text{DamHeight}) \times 0.08$.

---

## 5. `GridSubstation` (Класс сетевой подстанции)
* **`Id`**: Статический автоинкрементный `int`.
* **`SubstationCode`**: `string` (Формат: "SUB-XXX").
* **`Zone`**: `GridZone`.
* **`MaxCapacityKiloVolts`**: `double`.
* **`ConnectedGenerators`**: `PowerGenerator[]` (Приватный массив, только подключенные к этой подстанции генераторы).
* **Методы**:
  * `AddGeneratorToSubstation(PowerGenerator generator)`: Добавляет генератор в массив (с использованием `ref`). Если максимальная емкость подстанции превышена, вызывается `GridOverloadException`.

---

## 6. Классы `GridEngineer` и `PowerTransferTask`

### `GridEngineer`
* **`Id`**: Статический автоинкрементный `int`.
* **`Name`**, **`Surname`**: `string` (С помощью строковых методов первая буква обязательно должна быть заглавной).
* **`HireDate`**: `DateTime`.
* **`BaseSalary`**: `double`.
* **`ClearanceZone`**: `GridZone`.
* **`ClearanceLevel`**: `int` (Уровень допуска безопасности от 1 до 5).
* **`IsDispatched`**: `bool` (По умолчанию: `false`).
* **Static Method**: 
  * `GetCertifiedSeniorCount(GridEngineer[] engineers, DateTime maxHireDate, double minSalary, GridZone requiredZone, int minClearance)`
  * Возвращает количество инженеров, дата приема на работу которых старше `maxHireDate` (опытные), зона допуска соответствует `requiredZone`, уровень допуска больше или равен `minClearance` И зарплата больше `minSalary`.

### `PowerTransferTask`
* **`Id`**: Статический автоинкрементный `int`.
* **`GeneratorId`**: `int`.
* **`EngineerId`**: `int`.
* **`SubstationId`**: `int`.
* **`Status`**: `TransferStatus` (По умолчанию: `Scheduled`).
* **`StartTime`**: `DateTime` (По умолчанию: `DateTime.Now`).
* **`PlannedDuration`**: `TimeSpan` (Запланированное время выполнения процесса передачи).
* **Методы**:
  * `UpdateStatus(TransferStatus newStatus)`: Если текущий статус `Stabilized`, `Interrupted` или `Terminated`, изменить статус нельзя и должно быть вызвано исключение `GridOverloadException`.

---

## 7. Интерфейсы и Сервисы

### `IAeonGridManager` (Interface)
* `AddGenerator(PowerGenerator generator)`: Добавляет новый генератор в массив (массив расширяется через `ref`).
* `AddEngineer(GridEngineer engineer)`: Добавляет нового инженера в массив (массив расширяется через `ref`).
* `AddSubstation(GridSubstation substation)`: Добавляет новую подстанцию в массив (массив расширяется через `ref`).
* `ScheduleTransfer(int generatorId, int engineerId, int substationId, TimeSpan duration)`:
  * Если генератор, инженер или подстанция не найдены, вызывается `NotFoundException`.
  * Если инженер `IsDispatched == true` или генератор `IsOnline == true`, вызывается `NotAvailableException`.
  * Если значение `State` генератора отличается от `Operational`, вызывается `NotAvailableException`.
  * Если зона допуска инженера (`ClearanceZone`) не совпадает с целевой зоной генератора (`TargetZone`), вызывается `GridOverloadException`.
  * При успехе устанавливает `IsDispatched` инженера в `true` и `IsOnline` генератора в `true`.
* `CompleteTransfer(int taskId)`: Переводит задачу в статус `Stabilized`, меняет `IsOnline` генератора на `false` и `IsDispatched` инженера на `false`.
* `GetGeneratorsByZone(GridZone zone)`: Возвращает все генераторы в заданной зоне в виде массива `PowerGenerator[]`.

### `AeonGridManager` Class
* Реализует интерфейс `IAeonGridManager`.
* Содержит внутри себя **приватные массивы**: `PowerGenerator[]`, `GridEngineer[]`, `GridSubstation[]` и `PowerTransferTask[]`.
* **Индексатор 1**: `this[int index]` — Позволяет обращаться непосредственно к массиву `PowerTransferTask` по индексу для получение (`get`) и установки (`set`) задачи передачи.
* **Индексатор 2 (Перегруженный индексатор)**: `this[string serialNumber]` — Находит и возвращает объект `PowerGenerator` по серийному номеру (`get`).

---

## 8. Методы расширения (Extension Methods)
Внутри статического класса `GridExtensions` напишите следующие методы расширения:

1. **`GetExpectedCompletionTime(this PowerTransferTask task)`**:
   * Возвращает ожидаемое время завершения передачи (`task.StartTime + task.PlannedDuration`).
2. **`ConvertFrequency(double baseHertz, FrequencyUnit targetUnit)`**:
   * Конвертирует внутреннее значение частоты сети (Гц) в единицу измерения, переданную в качестве параметра (`Hertz`, `KiloHertz`, `MegaHertz`).
   *(Формулы: $1\text{ кГц} = 1000\text{ Гц}$, $1\text{ МГц} = 1000000\text{ Гц}$)*.
3. **`FormatSerialNumber(this string rawSerial)`**:
   * Очищает произвольный текст серийного номера, удаляет лишние пробелы и добавляет префикс "AGU-" (Например: `"  sol-123 "` -> `"AGU-SOL-123"`).

---

## 9. Интерактивное консольное меню (Program.cs)

Внутри `Program.cs` организуйте развернутое и функциональное меню, управляющее всей энергосистемой и основанное исключительно на логике `switch-case` и циклов (while/do-while):

```text
================ СИСТЕМА УПРАВЛЕНИЯ ЭНЕРГОСЕТЬЮ AEONGRID ULTRA ================
0 - Показать общее состояние сети, подстанции и статистику
1 - Добавить новый генератор энергии (Solar / Wind / Hydro)
2 - Зарегистрировать нового инженера сети
3 - Создать новую сетевую подстанцию (GridSubstation)
4 - Запланировать задачу передачи энергии (PowerTransferTask)
5 - Завершить задачу передачи (Complete Transfer)
6 - Отфильтровать сертифицированных инженеров (через статический метод Engineer)
7 - Проверить преобразование напряжения (Implicit/Explicit Volt <-> KiloVolt)
8 - Красная кнопка: Редактирование генератора/задачи по индексу или серийному номеру и расчет частоты
9 - Остановить систему



Your Code Academy chats aren’t used to improve our models. Gemini is AI and can make mistakes. Your privacy & GeminiOpens in a new window

Analyzing
# Программирование: «AeonGrid Ultra» — Мультизональная умная энергосеть и комплекс управления инфраструктурными материалами

В этом проекте вам предстоит с нуля разработать все структуры данных, исключения, сервисные службы и интерактивную симуляцию управления для корпоративного энергетического комплекса **AeonGrid Ultra**. Проект управляет подключаемыми к сети генераторами энергии, сетевыми трансформаторами (подстанциями), сертифицированными инженерами и сложными операциями передачи энергии, применяемыми в реальности.

> **ОСНОВНЫЕ ПРАВИЛА И ОГРАНИЧЕНИЯ:**
> 1. **Использование динамических коллекций (`List<T>`, `ArrayList`, `Dictionary<K,V>`, `LinkedList` и т. д.) СТРОГО ЗАПРЕЩЕНО!**
> 2. Все коллекции должны храниться внутри стандартных массивов (**Array**). При добавлении нового элемента в массив его размер должен динамически увеличиваться с использованием механизма `ref` и `Array.Resize`.
> 3. В проекте **НЕЛЬЗЯ использовать `struct` и `record`!** Разрешено использовать только объекты `class` и `interface`.
> 4. Все данные должны строго соответствовать принципам инкапсуляции (**Encapsulation**): поля (fields) должны быть скрыты, а для доступа к ним должны применяться соответствующие правила get/set.

---

## 1. Custom Exceptions (Пользовательские исключения)
Создайте следующие классы пользовательских исключений для обработки различных ошибочных сценариев в системе:

* **`NotFoundException`**: Вызывается, если искомый объект (генератор, инженер, подстанция или задача) по его ID/серийному номеру не найден в массиве.
* **`NotAvailableException`**: Вызывается, если генератор находится на техническом обслуживании, инженер занят другой задачей или в выбранной зоне ведутся ремонтные работы.
* **`GridOverloadException`**: Вызывается, если вырабатываемое напряжение/мощность превышает максимальную пропускную способность линии или если частота сети выходит за критические пределы.
* **`InsufficientClearanceException`**: Вызывается, если уровень допуска безопасности инженера (Clearance Level) не соответствует рейтингу безопасности регулируемого генератора.

---

## 2. Enum (Перечисления)
1. **`GridZone`**: `ZoneNorth`, `ZoneSouth`, `ZoneEast`, `ZoneWest`, `ZoneCentral` (Зоны деятельности энергосети).
2. **`TransferStatus`**: `Scheduled`, `Transmitting`, `Stabilized`, `Interrupted`, `Terminated` (По умолчанию: `Scheduled`).
3. **`FrequencyUnit`**: `Hertz`, `KiloHertz`, `MegaHertz`.
4. **`MaintenanceState`**: `Operational`, `UnderInspection`, `Decommissioned`.

---

## 3. Два разных класса и явные / неявные преобразования операторов (Implicit / Explicit)
Создайте два различных класса для управления электрическим напряжением (разностью потенциалов) в сети:

### `VoltVoltage` (Class)
* **`Magnitude`**: `double` свойство (Не может быть отрицательным. Если отрицательное — вызывается `GridOverloadException`).
* **Constructor**: Принимает параметр `Magnitude`.

### `KiloVoltVoltage` (Class)
* **`Magnitude`**: `double` свойство (Не может быть отрицательным. Если отрицательное — вызывается `GridOverloadException`).
* **Constructor**: Принимает параметр `Magnitude`.

### Операторы:
* Для класса `KiloVoltVoltage` напишите оператор **`implicit`** (неявного) преобразования к типу `VoltVoltage` ($1\text{ кВ} = 1000\text{ В}$).
* Для класса `VoltVoltage` напишите оператор **`explicit`** (явного) преобразования к типу `KiloVoltVoltage` ($1\text{ В} = 0.001\text{ кВ}$).

---

## 4. Абстрактный класс: `PowerGenerator` (Генератор энергии)
* **`Id`**: `int` (`private set`, автоматически увеличивается на 1 только в `ctor`).
* **`SerialNumber`**: `string` (Не может быть пустым/null, должны применяться `Trim()` и `ToUpper()`).
* **`CommissionDate`**: `DateTime` (Не может быть датой из будущего, если дата из будущего — вызывается `ArgumentException`).
* **`OutputVoltage`**: Тип класса `VoltVoltage`.
* **`TargetZone`**: Тип enum `GridZone`.
* **`State`**: Тип enum `MaintenanceState` (По умолчанию: `Operational`).
* **`IsOnline`**: `bool` (По умолчанию: `false`).
* **Поля `Readonly / Init / Const`**:
  * `public const string GRID_CODE = "AEON-ULTRA-GRID"`;
  * `public readonly DateTime RegisteredAt`; (присваивается значение в ctor)
  * `public string HardwareRevision { get; init; }`
* **Абстрактные методы**:
  * `double CalculateCarbonOffset(TimeSpan runTime)` — Рассчитывает количество сэкономленного/сокращенного углерода (в кг) в зависимости от времени работы.
  * `double CalculateEfficiencyIndex()` — Возвращает общий коэффициент полезного действия (КПД) генератора.
* **ToString()**: Должен быть переопределен (`override`) и возвращать серийный номер генератора, зону, техническое состояние и значение напряжения в читаемом текстовом формате.
* *Требование:* Нельзя создать объект `PowerGenerator` без передачи `SerialNumber`, `OutputVoltage` и `CommissionDate` (Используйте перегрузку конструкторов — Constructor Overloading).

### Дочерние классы (Не могут быть унаследованы — должны быть `sealed`):

1. **`SolarGenerator`**:
   * **`PanelSurfaceArea`**: `double` (Площадь поверхности панелей в кв. метрах, не может быть отрицательной или равной 0).
   * **`EfficiencyPercentage`**: `double` (От 0 до 100).
   * **`CalculateCarbonOffset(runTime)`**: $(15.5 \times \text{runTime.TotalHours}) + (\text{PanelSurfaceArea} \times 0.8)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{PanelSurfaceArea} \times \text{EfficiencyPercentage}) / 100.0$.

2. **`WindTurbineGenerator`**:
   * **`RotorDiameter`**: `double` (Диаметр ротора в метрах, не может быть отрицательным или равным 0).
   * **`AverageWindSpeed`**: `double` (Средняя скорость ветра в м/с).
   * **`CalculateCarbonOffset(runTime)`**: $(22.0 \times \text{runTime.TotalHours}) + (\text{RotorDiameter} \times 1.5)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{RotorDiameter} \times \text{AverageWindSpeed}) \times 0.12$.

3. **`HydroElectricGenerator`**:
   * **`WaterFlowRate`**: `double` (Расход воды в м³/с).
   * **`DamHeight`**: `double` (Высота плотины в метрах).
   * **`CalculateCarbonOffset(runTime)`**: $(35.0 \times \text{runTime.TotalHours}) + (\text{DamHeight} \times 2.1)$.
   * **`CalculateEfficiencyIndex()`**: $(\text{WaterFlowRate} \times \text{DamHeight}) \times 0.08$.

---

## 5. `GridSubstation` (Класс сетевой подстанции)
* **`Id`**: Статический автоинкрементный `int`.
* **`SubstationCode`**: `string` (Формат: "SUB-XXX").
* **`Zone`**: `GridZone`.
* **`MaxCapacityKiloVolts`**: `double`.
* **`ConnectedGenerators`**: `PowerGenerator[]` (Приватный массив, только подключенные к этой подстанции генераторы).
* **Методы**:
  * `AddGeneratorToSubstation(PowerGenerator generator)`: Добавляет генератор в массив (с использованием `ref`). Если максимальная емкость подстанции превышена, вызывается `GridOverloadException`.

---

## 6. Классы `GridEngineer` и `PowerTransferTask`

### `GridEngineer`
* **`Id`**: Статический автоинкрементный `int`.
* **`Name`**, **`Surname`**: `string` (С помощью строковых методов первая буква обязательно должна быть заглавной).
* **`HireDate`**: `DateTime`.
* **`BaseSalary`**: `double`.
* **`ClearanceZone`**: `GridZone`.
* **`ClearanceLevel`**: `int` (Уровень допуска безопасности от 1 до 5).
* **`IsDispatched`**: `bool` (По умолчанию: `false`).
* **Static Method**: 
  * `GetCertifiedSeniorCount(GridEngineer[] engineers, DateTime maxHireDate, double minSalary, GridZone requiredZone, int minClearance)`
  * Возвращает количество инженеров, дата приема на работу которых старше `maxHireDate` (опытные), зона допуска соответствует `requiredZone`, уровень допуска больше или равен `minClearance` И зарплата больше `minSalary`.

### `PowerTransferTask`
* **`Id`**: Статический автоинкрементный `int`.
* **`GeneratorId`**: `int`.
* **`EngineerId`**: `int`.
* **`SubstationId`**: `int`.
* **`Status`**: `TransferStatus` (По умолчанию: `Scheduled`).
* **`StartTime`**: `DateTime` (По умолчанию: `DateTime.Now`).
* **`PlannedDuration`**: `TimeSpan` (Запланированное время выполнения процесса передачи).
* **Методы**:
  * `UpdateStatus(TransferStatus newStatus)`: Если текущий статус `Stabilized`, `Interrupted` или `Terminated`, изменить статус нельзя и должно быть вызвано исключение `GridOverloadException`.

---

## 7. Интерфейсы и Сервисы

### `IAeonGridManager` (Interface)
* `AddGenerator(PowerGenerator generator)`: Добавляет новый генератор в массив (массив расширяется через `ref`).
* `AddEngineer(GridEngineer engineer)`: Добавляет нового инженера в массив (массив расширяется через `ref`).
* `AddSubstation(GridSubstation substation)`: Добавляет новую подстанцию в массив (массив расширяется через `ref`).
* `ScheduleTransfer(int generatorId, int engineerId, int substationId, TimeSpan duration)`:
  * Если генератор, инженер или подстанция не найдены, вызывается `NotFoundException`.
  * Если инженер `IsDispatched == true` или генератор `IsOnline == true`, вызывается `NotAvailableException`.
  * Если значение `State` генератора отличается от `Operational`, вызывается `NotAvailableException`.
  * Если зона допуска инженера (`ClearanceZone`) не совпадает с целевой зоной генератора (`TargetZone`), вызывается `GridOverloadException`.
  * При успехе устанавливает `IsDispatched` инженера в `true` и `IsOnline` генератора в `true`.
* `CompleteTransfer(int taskId)`: Переводит задачу в статус `Stabilized`, меняет `IsOnline` генератора на `false` и `IsDispatched` инженера на `false`.
* `GetGeneratorsByZone(GridZone zone)`: Возвращает все генераторы в заданной зоне в виде массива `PowerGenerator[]`.

### `AeonGridManager` Class
* Реализует интерфейс `IAeonGridManager`.
* Содержит внутри себя **приватные массивы**: `PowerGenerator[]`, `GridEngineer[]`, `GridSubstation[]` и `PowerTransferTask[]`.
* **Индексатор 1**: `this[int index]` — Позволяет обращаться непосредственно к массиву `PowerTransferTask` по индексу для получение (`get`) и установки (`set`) задачи передачи.
* **Индексатор 2 (Перегруженный индексатор)**: `this[string serialNumber]` — Находит и возвращает объект `PowerGenerator` по серийному номеру (`get`).

---

## 8. Методы расширения (Extension Methods)
Внутри статического класса `GridExtensions` напишите следующие методы расширения:

1. **`GetExpectedCompletionTime(this PowerTransferTask task)`**:
   * Возвращает ожидаемое время завершения передачи (`task.StartTime + task.PlannedDuration`).
2. **`ConvertFrequency(double baseHertz, FrequencyUnit targetUnit)`**:
   * Конвертирует внутреннее значение частоты сети (Гц) в единицу измерения, переданную в качестве параметра (`Hertz`, `KiloHertz`, `MegaHertz`).
   *(Формулы: $1\text{ кГц} = 1000\text{ Гц}$, $1\text{ МГц} = 1000000\text{ Гц}$)*.
3. **`FormatSerialNumber(this string rawSerial)`**:
   * Очищает произвольный текст серийного номера, удаляет лишние пробелы и добавляет префикс "AGU-" (Например: `"  sol-123 "` -> `"AGU-SOL-123"`).

---

## 9. Интерактивное консольное меню (Program.cs)

Внутри `Program.cs` организуйте развернутое и функциональное меню, управляющее всей энергосистемой и основанное исключительно на логике `switch-case` и циклов (while/do-while):

```text
================ СИСТЕМА УПРАВЛЕНИЯ ЭНЕРГОСЕТЬЮ AEONGRID ULTRA ================
0 - Показать общее состояние сети, подстанции и статистику
1 - Добавить новый генератор энергии (Solar / Wind / Hydro)
2 - Зарегистрировать нового инженера сети
3 - Создать новую сетевую подстанцию (GridSubstation)
4 - Запланировать задачу передачи энергии (PowerTransferTask)
5 - Завершить задачу передачи (Complete Transfer)
6 - Отфильтровать сертифицированных инженеров (через статический метод Engineer)
7 - Проверить преобразование напряжения (Implicit/Explicit Volt <-> KiloVolt)
8 - Красная кнопка: Редактирование генератора/задачи по индексу или серийному номеру и расчет частоты
9 - Остановить систему
```
