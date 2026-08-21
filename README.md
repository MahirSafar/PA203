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
























````md
# Программное задание: «AeonGrid» — интеллектуальная энергосеть и система распределения генераторов

В рамках данного проекта необходимо разработать симуляцию системы управления интеллектуальным энергетическим комплексом **AeonGrid**. Система должна управлять источниками энергии, подключёнными к сети, инженерами и операциями по передаче энергии.

> **ОСНОВНЫЕ ПРАВИЛА И ОГРАНИЧЕНИЯ:**
>
> 1. Использование динамических коллекций (`List`, `ArrayList`, `Dictionary` и т. п.) **КАТЕГОРИЧЕСКИ ЗАПРЕЩЕНО**.
> 2. Все данные должны храниться в **массивах (`Array`)**. При добавлении нового элемента размер массива необходимо динамически увеличивать с помощью механизма `ref` и `Array.Resize`.
> 3. Использование `struct` и `record` в проекте **ЗАПРЕЩЕНО**. Необходимо использовать только `class` и `interface`.

---

## 1. Пользовательские исключения

Для обработки ошибок в проекте необходимо создать следующие пользовательские исключения:

- **`NotFoundException`** — выбрасывается, если в массиве не найден генератор, станция или инженер с указанным ID.
- **`NotAvailableException`** — выбрасывается, если генератор находится на ремонте/техническом обслуживании, инженер занят другим заданием или линия электросети неактивна.
- **`GridOverloadException`** — выбрасывается, если напряжение/мощность, производимая генератором, превышает допустимый предел линии электросети или сила тока достигает опасного значения.

---

## 2. Enum

Необходимо создать следующие перечисления.

### `GridZone`

Зоны электросети:

- `ZoneNorth`
- `ZoneSouth`
- `ZoneEast`
- `ZoneWest`

### `TransferStatus`

Статусы передачи энергии:

- `Scheduled`
- `Transmitting`
- `Stabilized`
- `Interrupted`

Значение по умолчанию: `Scheduled`.

### `FrequencyUnit`

Единицы измерения частоты:

- `Hertz` — основная единица;
- `KiloHertz`;
- `MegaHertz`.

---

## 3. Два класса и преобразования `implicit` / `explicit`

Для управления электрическим потенциалом (напряжением) в сети необходимо создать два отдельных класса.

### `VoltVoltage`

- **`Magnitude`** — свойство типа `double`.
- Значение не может быть отрицательным.
- Если передано отрицательное значение, необходимо выбросить `GridOverloadException`.
- Конструктор должен принимать параметр `Magnitude`.

### `KiloVoltVoltage`

- **`Magnitude`** — свойство типа `double`.
- Значение не может быть отрицательным.
- Если передано отрицательное значение, необходимо выбросить `GridOverloadException`.
- Конструктор должен принимать параметр `Magnitude`.

### Операторы преобразования

Для `KiloVoltVoltage` необходимо реализовать неявное (`implicit`) преобразование в `VoltVoltage`:

**1 kV = 1000 V**

Для `VoltVoltage` необходимо реализовать явное (`explicit`) преобразование в `KiloVoltVoltage`:

**1 V = 0.001 kV**

---

## 4. Абстрактный класс `PowerGenerator`

Создайте абстрактный класс **`PowerGenerator`**, представляющий энергетический генератор.

### Свойства

- **`Id`** — `int`, `private set`. ID должен автоматически увеличиваться на 1 при создании каждого нового генератора.
- **`SerialNumber`** — `string`.
  - Не может быть `null` или пустым.
  - Перед сохранением необходимо применить `Trim()` и `ToUpper()`.
- **`CommissionDate`** — `DateTime`.
  - Не может быть датой в будущем.
- **`OutputVoltage`** — объект класса `VoltVoltage`.
- **`TargetZone`** — значение типа `GridZone`.
- **`IsOnline`** — `bool`.
  - Значение по умолчанию: `false`.

### Readonly / Init / Const

Необходимо добавить:

```csharp
public const string GRID_CODE = "AEON-GRID-AZ";
````

Также:

```csharp
public readonly DateTime RegisteredAt;
```

`RegisteredAt` должен получать значение в конструкторе.

Необходимо добавить свойство:

```csharp
public string HardwareRevision { get; init; }
```

### Абстрактный метод

Необходимо реализовать абстрактный метод:

```csharp
double CalculateCarbonOffset(TimeSpan runTime)
```

Метод должен рассчитывать количество сэкономленного/выделенного углерода в килограммах в зависимости от продолжительности работы генератора.

### `ToString()`

Метод `ToString()` необходимо переопределить.

Он должен возвращать текстовое представление генератора, содержащее:

* серийный номер;
* зону;
* значение напряжения.

### Конструкторы

Нельзя допускать создание объекта `PowerGenerator` без передачи:

* `SerialNumber`;
* `OutputVoltage`;
* `CommissionDate`.

Для этого необходимо использовать перегруженные конструкторы.

---

### Производные классы

Производные классы должны быть `sealed`, то есть дальнейшее наследование от них запрещено.

### `SolarGenerator`

Свойство:

* **`PanelSurfaceArea`** — `double`, площадь солнечных панелей в квадратных метрах.
* Значение не может быть отрицательным.

Реализация `CalculateCarbonOffset(runTime)`:

```text
(15.5 × runTime.TotalHours) + (PanelSurfaceArea × 0.8)
```

### `WindTurbineGenerator`

Свойство:

* **`RotorDiameter`** — `double`, диаметр ротора в метрах.
* Значение не может быть отрицательным.

Реализация `CalculateCarbonOffset(runTime)`:

```text
(22.0 × runTime.TotalHours) + (RotorDiameter × 1.5)
```

---

## 5. Классы `GridEngineer` и `PowerTransferTask`

## `GridEngineer`

### Свойства

* **`Id`** — статически увеличиваемый `int`.
* **`Name`** — `string`.
* **`Surname`** — `string`.
* **`HireDate`** — `DateTime`.
* **`BaseSalary`** — `double`.
* **`ClearanceZone`** — `GridZone`.
* **`IsDispatched`** — `bool`.

  * Значение по умолчанию: `false`.

Для `Name` и `Surname` необходимо использовать методы `string`, чтобы первая буква была заглавной.

Например:

```text
mahir → Mahir
aliyev → Aliyev
```

### Статический метод

Необходимо реализовать:

```csharp
GetCertifiedSeniorCount(
    GridEngineer[] engineers,
    DateTime maxHireDate,
    double minSalary,
    GridZone requiredZone)
```

Метод должен вернуть количество инженеров, которые одновременно соответствуют всем условиям:

1. Дата приёма на работу раньше `maxHireDate`.
2. `ClearanceZone` соответствует `requiredZone`.
3. Зарплата больше `minSalary`.

---

## `PowerTransferTask`

### Свойства

* **`Id`** — статически увеличиваемый `int`.
* **`GeneratorId`** — `int`.
* **`EngineerId`** — `int`.
* **`Status`** — `TransferStatus`.

  * Значение по умолчанию: `Scheduled`.
* **`StartTime`** — `DateTime`.

  * Значение по умолчанию: `DateTime.Now`.
* **`PlannedDuration`** — `TimeSpan`.

  * Продолжительность процесса передачи энергии.

### `UpdateStatus(TransferStatus newStatus)`

Необходимо реализовать метод изменения статуса.

Если текущий статус уже равен:

* `Stabilized`, или
* `Interrupted`,

то изменить его нельзя.

В этом случае необходимо выбросить:

```text
GridOverloadException
```

---

## 6. Интерфейсы и сервисы

### `IAeonGridManager`

Создайте интерфейс `IAeonGridManager`.

Он должен содержать следующие методы.

### `AddGenerator(PowerGenerator generator)`

Добавляет новый генератор в массив.

При необходимости массив должен увеличиваться с помощью `ref` и `Array.Resize`.

### `AddEngineer(GridEngineer engineer)`

Добавляет нового инженера в массив.

При необходимости массив должен увеличиваться с помощью `ref` и `Array.Resize`.

### `ScheduleTransfer(int generatorId, int engineerId, TimeSpan duration)`

Метод должен создать новое задание на передачу энергии.

Необходимо выполнить следующие проверки:

1. Если генератор не найден — выбросить `NotFoundException`.
2. Если инженер не найден — выбросить `NotFoundException`.
3. Если `engineer.IsDispatched == true` — выбросить `NotAvailableException`.
4. Если `generator.IsOnline == true` — выбросить `NotAvailableException`.
5. Если `engineer.ClearanceZone` не соответствует `generator.TargetZone` — выбросить `GridOverloadException`.

При успешном создании задания:

```text
engineer.IsDispatched = true
generator.IsOnline = true
```

### `CompleteTransfer(int taskId)`

Метод должен завершить указанное задание.

Необходимо:

1. Найти задание по `taskId`.
2. Изменить его статус на `Stabilized`.
3. Установить `generator.IsOnline = false`.
4. Установить `engineer.IsDispatched = false`.

---

## `AeonGridManager`

Создайте класс **`AeonGridManager`**, реализующий интерфейс `IAeonGridManager`.

Внутри класса необходимо хранить следующие **private-массивы**:

```csharp
private PowerGenerator[] generators;
private GridEngineer[] engineers;
private PowerTransferTask[] transferTasks;
```

Использование `List`, `Dictionary`, `ArrayList` и других динамических коллекций запрещено.

### Indexer

Необходимо реализовать индексатор:

```csharp
this[int index]
```

Он должен позволять напрямую получать и изменять элементы массива `PowerTransferTask`:

```csharp
PowerTransferTask task = manager[0];

manager[0] = task;
```

---

## 7. Extension-методы

Создайте статический класс:

```text
GridExtensions
```

В нём необходимо реализовать следующие extension-методы.

### `GetExpectedCompletionTime`

```csharp
GetExpectedCompletionTime(this PowerTransferTask task)
```

Метод должен вернуть предполагаемое время окончания передачи:

```text
task.StartTime + task.PlannedDuration
```

### `ConvertFrequency`

```csharp
ConvertFrequency(double baseHertz, FrequencyUnit targetUnit)
```

Метод должен преобразовывать внутреннюю частоту генератора из Hz в указанную единицу измерения.

Формулы:

```text
1 kHz = 1000 Hz
1 MHz = 1 000 000 Hz
```

Примеры:

```text
1000 Hz → 1000 Hz
1000 Hz → 1 kHz
1 000 000 Hz → 1 MHz
```

---

# 8. Интерактивное консольное меню

В `Program.cs` необходимо реализовать интерактивное консольное меню для управления всей сетью.

Логика меню должна быть построена **только с использованием `switch-case` и циклов**.

Меню должно выглядеть следующим образом:

```text
================ AEONGRID СИСТЕМА УПРАВЛЕНИЯ ЭНЕРГОСЕТЬЮ ================

0 - Показать общий статус и статистику сети
1 - Добавить новый энергетический генератор (SolarGenerator / WindTurbineGenerator)
2 - Зарегистрировать нового сетевого инженера
3 - Создать новое задание на передачу энергии (PowerTransferTask)
4 - Завершить задание на передачу энергии (Complete Transfer)
5 - Отфильтровать опытных инженеров (с использованием статического метода)
6 - Проверить преобразование напряжения (Implicit/Explicit Volt <-> KiloVolt)
7 - Красная кнопка: редактирование задания по индексу и расчёт частоты
8 - Остановить систему
```

## Требования к меню

### `0 — Общий статус`

Необходимо вывести:

* количество генераторов;
* количество инженеров;
* количество активных генераторов;
* количество занятых инженеров;
* количество заданий;
* количество завершённых заданий;
* общую статистику сети.

### `1 — Добавление генератора`

Пользователь должен иметь возможность выбрать тип генератора:

```text
1 - SolarGenerator
2 - WindTurbineGenerator
```

После этого необходимо запросить соответствующие параметры и добавить генератор в массив.

### `2 — Регистрация инженера`

Необходимо запросить данные инженера:

* имя;
* фамилию;
* дату приёма;
* зарплату;
* зону допуска.

После этого инженер должен быть добавлен в массив.

### `3 — Создание передачи`

Необходимо запросить:

* ID генератора;
* ID инженера;
* продолжительность передачи.

После выполнения всех проверок необходимо создать `PowerTransferTask`.

### `4 — Завершение передачи`

Пользователь вводит ID задания.

После успешного завершения:

```text
Status = Stabilized
Generator.IsOnline = false
Engineer.IsDispatched = false
```

### `5 — Поиск опытных инженеров`

Необходимо запросить:

* максимальную дату приёма;
* минимальную зарплату;
* требуемую зону.

После этого использовать:

```csharp
GridEngineer.GetCertifiedSeniorCount(...)
```

и вывести количество подходящих инженеров.

### `6 — Проверка преобразований напряжения`

Необходимо продемонстрировать:

* `implicit` преобразование `KiloVoltVoltage → VoltVoltage`;
* `explicit` преобразование `VoltVoltage → KiloVoltVoltage`.

### `7 — Красная кнопка`

Данный пункт должен демонстрировать сразу несколько возможностей системы:

1. Пользователь вводит индекс задания.
2. Через индексатор `AeonGridManager` получает соответствующий `PowerTransferTask`.
3. При необходимости изменяет его статус.
4. Рассчитывает предполагаемое время окончания через:

```csharp
GetExpectedCompletionTime()
```

5. Выполняет расчёт частоты через:

```csharp
ConvertFrequency(...)
```

### `8 — Остановка системы`

Завершает работу консольного приложения.

---

# Итоговые требования

В результате проект должен демонстрировать использование следующих возможностей C#:

* `class`;
* `abstract class`;
* `sealed class`;
* `interface`;
* `enum`;
* `static`;
* `const`;
* `readonly`;
* `init`;
* перегруженные конструкторы;
* наследование;
* полиморфизм;
* переопределение `ToString()`;
* абстрактные методы;
* пользовательские исключения;
* `implicit` и `explicit` операторы;
* extension-методы;
* indexer;
* массивы;
* `ref`;
* `Array.Resize`;
* `switch-case`;
* циклы;
* `TimeSpan`;
* `DateTime`.

**Использование `List`, `Dictionary`, `ArrayList`, `struct` и `record` запрещено.**

Все операции с коллекциями должны выполняться исключительно через массивы с их динамическим расширением.

```
```
sayını qaytarır.



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
