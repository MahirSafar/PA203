# Smart Home System (Ağıllı Ev İdarəetmə Sistemi)

Bu tapşırıqda məqsəd **Polymorphism**, **Inheritance**, **Encapsulation**, **Method Overriding & Overloading**, **Encapsulation (Access Modifiers, `init`, `readonly`)** kimi anlayışları tətbiq etməkdir.

---

## 1. Base Class: `SmartDevice`

Bir `SmartDevice` base class-ı yaradın. Class daxilində aşağıdakı property-lər və sahələr olmalıdır:

* **Property-lər:**
  * `DeviceName` (string) — `init` accessor ilə (obyekt yaradıldıqdan sonra dəyişdirilə bilməz).
  * `Brand` (string) — `readonly` field vasitəsilə və ya `init` ilə dəstəklənsin.
  * `PowerRatingWatts` (double) — Cihazın saatlıq enerji sərfiyyatı gücü.
  * `IsOn` (bool) — Cihazın açıq/qapalı olma vəziyyəti.
  * `TotalEnergyConsumedKWh` (double) — Cihazın ümumi işlətdiyi enerji (kWh ilə).

* **Constructor:**
  * Parameter-lər: `string deviceName`, `string brand`, `double powerRatingWatts`
  * **Qaydalar:**
    * `DeviceName` və `Brand` boş və ya `null` ola bilməz.
    * `PowerRatingWatts` 0-dan böyük olmalıdır.
    * `IsOn` default olaraq `false` olmalıdır.
    * `TotalEnergyConsumedKWh` default olaraq `0.0` olmalıdır.

* **Method-lar:**
  * `TurnOn()`
    * Cihazı işə salır (`IsOn = true`).
  * `TurnOff()`
    * Cihazı söndürür (`IsOn = false`).
  * **`Operate(int minutes)` (virtual)**
    * Cihazı müəyyən dəqiqə ərzində işlədir və enerjini hesablayıb `TotalEnergyConsumedKWh` üzərinə əlavə edir.
    * **Hesablama düsturu:**
      $$\text{consumedKWh} = \frac{\text{PowerRatingWatts} \times \text{minutes}}{1000 \times 60}$$
    * **Şərtlər:**
      * `minutes > 0` olmalıdır.
      * `IsOn == true` olmalıdır.
  * **`GetDeviceInfo()` (virtual)**
    * Cihaz haqqında məlumat qaytarır və ya konsola yazdırır.
    * **Format:**
      ```text
      [DEVICE INFO]
      Type: Generic Smart Device
      Name: Living Room Hub
      Brand: Tuya
      Power Rating: 15W
      Status: ON
      Total Energy Consumed: 0.150 kWh
      ```

---

## 2. Derived Class 1: `SmartLight`

`SmartDevice`-dan miras alan `SmartLight` class-ı yaradın.

* **Əlavə Property-lər:**
  * `BrightnessPercent` (int) — İşığın parlaqlığı (0 – 100 arası).
  * `Color` (string) — İşığın rəngi (Məsələn: "Warm White", "RGB Red").

* **Constructor:**
  * Parameter-lər: `deviceName`, `brand`, `powerRatingWatts`, `initialBrightness`, `color`
  * **Qaydalar:**
    * Base class constructor-u `base(...)` ilə çağırılmalıdır.
    * `initialBrightness` 0 ilə 100 arasında olmalıdır (əks halda default `50` götürülsün).

* **Metod Overloading (Yeni Method):**
  * `SetBrightness(int percent)` — Parlaqlığı 0–100 aralığında dəyişir.
  * `SetBrightness(int percent, string newColor)` — Parlaqlıqla bərabər rəngi də dəyişir.

* **Method Overriding (`Operate(int minutes)`):**
  * İşığın parlaqlığı enerji sərfiyyatına təsir edir!
  * **Yenilənmiş hesablama:**
    $$\text{effectiveWatts} = \text{PowerRatingWatts} \times \left(\frac{\text{BrightnessPercent}}{100.0}\right)$$
    $$\text{consumedKWh} = \frac{\text{effectiveWatts} \times \text{minutes}}{1000 \times 60}$$
  * Şərt: `IsOn == true` və `minutes > 0`.

* **Method Overriding (`GetDeviceInfo()`):**
  * Base class-ın məlumatlarını genişləndirir, `Type` sətrini `Smart Light` edir və əlavə olaraq parlaqlıq və rəng sətrini çıxarır:
    ```text
    Brightness: 80%
    Color: Warm White
    ```

---

## 3. Derived Class 2: `SmartAirConditioner`

`SmartDevice`-dan miras alan `SmartAirConditioner` class-ı yaradın.

* **Əlavə Property-lər:**
  * `TargetTemperature` (double) — Təyin olunmuş dərəcə.
  * `CurrentMode` (string) — Rejim ("Cool", "Heat", "FanOnly").

* **Constructor:**
  * Parameter-lər: `deviceName`, `brand`, `powerRatingWatts`, `targetTemp`, `mode`
  * **Qaydalar:**
    * Base class constructor-u `base(...)` ilə çağırılmalıdır.
    * `targetTemp` 16°C ilə 30°C arasında olmalıdır.

* **Method Overriding (`Operate(int minutes)`):**
  * Kondisionerin rejimindən asılı olaraq enerji multiplikatoru dəyişir:
    * `"Cool"` rejimində: Güc **1.2x** vurulur.
    * `"Heat"` rejimində: Güc **1.5x** vurulur.
    * `"FanOnly"` rejimində: Güc **0.3x** vurulur.
  * **Hesablama:**
    $$\text{effectiveWatts} = \text{PowerRatingWatts} \times \text{ModeMultiplier}$$
    $$\text{consumedKWh} = \frac{\text{effectiveWatts} \times \text{minutes}}{1000 \times 60}$$

* **Method Overriding (`GetDeviceInfo()`):**
  * `Type` sətrini `Smart Air Conditioner` edir və əlavə olaraq rejim və dərəcəni göstərir:
    ```text
    Target Temp: 22°C
    Mode: Cool
    ```

---

## 4. System Manager (Polymorphism & Collection Control): `SmartHomeHub`

Ayrı bir `SmartHomeHub` class-ı yaradın. Bu class sistemdəki bütün cihazları idarə edəcək.

* **Property & Fields:**
  * `private List<SmartDevice> devices` — Bütün cihazları saxlayan kolleksiya.

* **Method-lar:**
  * `AddDevice(SmartDevice device)` — Siyahıya cihaz əlavə edir.
  * `TurnAllOn()` — Siyahıdakı bütün cihazların `TurnOn()` metodunu çağırır.
  * `TurnAllOff()` — Siyahıdakı bütün cihazların `TurnOff()` metodunu çağırır.
  * **`RunAutomation(int minutes)` (Polymorphic Call!):**
    * Dövrlə (`foreach`) siyahıdakı bütün cihazların **`Operate(minutes)`** metodunu çağırır.
    * *Diqqət:* Bütün cihazlar eyni `Operate(minutes)` kimi çağırılsa da, runtime zamanı `SmartLight` öz parlaqlığına görə, `SmartAirConditioner` isə öz rejiminə görə enerjini hesablamalıdır!
  * **`DisplayAllReport()` (Polymorphic Call!):**
    * Dövrlə (`foreach`) siyahıdakı hər bir cihazın `GetDeviceInfo()` metodunu çağırır və ən sonda bütün ev üzrə **ümumi xərclənən enerjini** (`TotalEnergyConsumedKWh`) toplayıb ekrana yazdırır.

---

## 5. `Program.cs` (Test Ssenarisi)

`Main` metodu daxilində aşağıdakı addımları test edin:

1. `SmartHomeHub` obyekti yaradın.
2. Aşağıdakı obyektləri yaradıb `base(...)` constructor çağırışlarının düzgünlüyünü təmin edin:
   * 1 ədəd `SmartLight` (məsələn: Brand="Philips", Power=10W, Brightness=80%, Color="Warm White").
   * 1 ədəd `SmartAirConditioner` (məsələn: Brand="LG", Power=1500W, TargetTemp=21°C, Mode="Cool").
3. Hər iki cihazı `SmartHomeHub`-a əlavə edin (`AddDevice`).
4. **Polimorfizm Testi:**
   * Cihazları işə salın (`TurnAllOn()`).
   * 60 dəqiqəlik avtomatlaşdırma işlədin (`RunAutomation(60)`).
   * Cihazların parlaqlıq/rejim parametrlərini dəyişin (məsələn, İşığı `SetBrightness(20)` edin, Kondisioneri `"FanOnly"` rejiminə keçirin).
   * Yenidən 120 dəqiqəlik avtomatlaşdırma işlədin (`RunAutomation(120)`).
5. **Report Testi:**
   * `DisplayAllReport()` metodunu çağıraraq həm individual cihazların fərqli output-larını, həm də ümumi xərclənən enerjini ekranda görün.

















# Smart Home System (Система управления умным домом)

Цель задания — применить на практике такие концепции, как **Polymorphism**, **Inheritance**, **Encapsulation**, **Method Overriding & Overloading**, а также **Access Modifiers**, `init` и `readonly`.

---

## 1. Базовый класс: `SmartDevice`

Создайте базовый класс `SmartDevice`. Внутри класса должны быть следующие свойства и поля.

### Свойства

- `DeviceName` (`string`) — имя устройства. Использовать `init`, чтобы после создания объекта его нельзя было изменить.
- `Brand` (`string`) — бренд устройства. Реализовать через `readonly` field или `init`.
- `PowerRatingWatts` (`double`) — мощность устройства в ваттах.
- `IsOn` (`bool`) — состояние устройства: включено или выключено.
- `TotalEnergyConsumedKWh` (`double`) — общее количество потреблённой энергии в kWh.

### Constructor

Параметры:

```csharp
string deviceName
string brand
double powerRatingWatts
```

Правила:

- `DeviceName` и `Brand` не могут быть `null` или пустыми.
- `PowerRatingWatts` должен быть больше `0`.
- `IsOn` по умолчанию должен быть `false`.
- `TotalEnergyConsumedKWh` по умолчанию должен быть `0.0`.

### Методы

#### `TurnOn()`

Включает устройство:

```text
IsOn = true
```

#### `TurnOff()`

Выключает устройство:

```text
IsOn = false
```

#### `Operate(int minutes)` — `virtual`

Запускает устройство на определённое количество минут и рассчитывает потреблённую энергию, добавляя её к `TotalEnergyConsumedKWh`.

Формула:

```text
consumedKWh = (PowerRatingWatts × minutes) / (1000 × 60)
```

Условия:

- `minutes > 0`
- `IsOn == true`

#### `GetDeviceInfo()` — `virtual`

Возвращает информацию об устройстве или выводит её в консоль.

Формат:

```text
[DEVICE INFO]
Type: Generic Smart Device
Name: Living Room Hub
Brand: Tuya
Power Rating: 15W
Status: ON
Total Energy Consumed: 0.150 kWh
```

---

## 2. Производный класс: `SmartLight`

Создайте класс `SmartLight`, который наследуется от `SmartDevice`.

### Дополнительные свойства

- `BrightnessPercent` (`int`) — яркость от `0` до `100`.
- `Color` (`string`) — цвет освещения, например `"Warm White"` или `"RGB Red"`.

### Constructor

Параметры:

```text
deviceName
brand
powerRatingWatts
initialBrightness
color
```

Правила:

- Constructor базового класса должен быть вызван через `base(...)`.
- `initialBrightness` должен находиться в диапазоне `0–100`.
- Если значение выходит за этот диапазон, установить значение `50`.

### Method Overloading

Создайте два метода с одинаковым названием `SetBrightness`.

#### `SetBrightness(int percent)`

Изменяет только яркость в диапазоне `0–100`.

#### `SetBrightness(int percent, string newColor)`

Изменяет яркость и одновременно цвет.

Таким образом, здесь необходимо продемонстрировать **Method Overloading**.

### Method Overriding — `Operate(int minutes)`

Переопределите метод `Operate`.

Яркость лампы влияет на потребление энергии.

Формула:

```text
effectiveWatts = PowerRatingWatts × (BrightnessPercent / 100.0)

consumedKWh = (effectiveWatts × minutes) / (1000 × 60)
```

Условия:

- `IsOn == true`
- `minutes > 0`

### Method Overriding — `GetDeviceInfo()`

Переопределите `GetDeviceInfo()`.

В строке `Type` должно быть:

```text
Type: Smart Light
```

Также добавьте информацию о яркости и цвете:

```text
Brightness: 80%
Color: Warm White
```

---

## 3. Производный класс: `SmartAirConditioner`

Создайте класс `SmartAirConditioner`, который наследуется от `SmartDevice`.

### Дополнительные свойства

- `TargetTemperature` (`double`) — установленная температура.
- `CurrentMode` (`string`) — текущий режим: `"Cool"`, `"Heat"` или `"FanOnly"`.

### Constructor

Параметры:

```text
deviceName
brand
powerRatingWatts
targetTemp
mode
```

Правила:

- Constructor базового класса должен вызываться через `base(...)`.
- `targetTemp` должен находиться в диапазоне от `16°C` до `30°C`.

### Method Overriding — `Operate(int minutes)`

Переопределите `Operate`.

Потребление энергии зависит от текущего режима кондиционера.

Множители:

- `"Cool"` → `1.2x`
- `"Heat"` → `1.5x`
- `"FanOnly"` → `0.3x`

Формула:

```text
effectiveWatts = PowerRatingWatts × ModeMultiplier

consumedKWh = (effectiveWatts × minutes) / (1000 × 60)
```

### Method Overriding — `GetDeviceInfo()`

Переопределите `GetDeviceInfo()`.

В строке `Type` должно быть:

```text
Type: Smart Air Conditioner
```

Также выведите температуру и режим:

```text
Target Temp: 22°C
Mode: Cool
```

---

## 4. System Manager: `SmartHomeHub`

Создайте отдельный класс `SmartHomeHub`.

Этот класс будет управлять всеми устройствами в системе.

### Поле

```csharp
private List<SmartDevice> devices;
```

Коллекция должна хранить все устройства.

### Методы

#### `AddDevice(SmartDevice device)`

Добавляет устройство в список.

#### `TurnAllOn()`

Вызывает `TurnOn()` у каждого устройства в списке.

#### `TurnAllOff()`

Вызывает `TurnOff()` у каждого устройства.

### `RunAutomation(int minutes)` — Polymorphism

С помощью `foreach` пройдите по всем устройствам и вызовите:

```csharp
Operate(minutes);
```

Важно: хотя все устройства вызывают один и тот же метод `Operate(minutes)`, во время выполнения программы должен вызываться метод конкретного класса.

Например:

- `SmartLight` рассчитывает энергию с учётом яркости.
- `SmartAirConditioner` рассчитывает энергию с учётом режима.

Это демонстрирует **Polymorphism**.

### `DisplayAllReport()` — Polymorphism

С помощью `foreach` вызовите `GetDeviceInfo()` у каждого устройства.

После этого необходимо подсчитать общее количество потреблённой энергии всеми устройствами и вывести результат на экран.

---

## 5. `Program.cs` — тестовый сценарий

В методе `Main` протестируйте систему по следующему сценарию.

### Шаг 1

Создайте объект:

```text
SmartHomeHub
```

### Шаг 2

Создайте следующие устройства.

#### SmartLight

```text
Brand = Philips
Power = 10W
Brightness = 80%
Color = Warm White
```

#### SmartAirConditioner

```text
Brand = LG
Power = 1500W
Target Temperature = 21°C
Mode = Cool
```

Убедитесь, что constructor базового класса вызывается правильно через `base(...)`.

### Шаг 3

Добавьте оба устройства в `SmartHomeHub` через:

```text
AddDevice(...)
```

### Шаг 4 — Проверка Polymorphism

Включите все устройства:

```text
TurnAllOn()
```

Запустите автоматизацию на **60 минут**:

```text
RunAutomation(60)
```

После этого измените настройки.

Для лампы:

```text
SetBrightness(20)
```

Для кондиционера измените режим на:

```text
FanOnly
```

Затем снова запустите автоматизацию, но уже на **120 минут**:

```text
RunAutomation(120)
```

### Шаг 5 — Проверка Report

Вызовите:

```text
DisplayAllReport()
```

На экране должны отображаться:

- информация о каждом устройстве;
- состояние устройства;
- мощность;
- индивидуальное потребление энергии;
- для лампы — яркость и цвет;
- для кондиционера — температура и режим;
- в конце — **общее потребление энергии всего дома**.

---

## Основная цель задания

Главная цель задания — продемонстрировать, что через:

```csharp
List<SmartDevice>
```

можно хранить разные дочерние классы и вызывать их переопределённые методы:

```csharp
Operate()
GetDeviceInfo()
```

через **Polymorphism**.

В процессе выполнения задания должны быть продемонстрированы:

- **Inheritance**
- **Polymorphism**
- **Encapsulation**
- **Method Overriding**
- **Method Overloading**
- **Access Modifiers**
- `init`
- `readonly`
- `virtual`
- `override`
- `List<T>`
- `foreach`










# Multi-Format Document Export & Printing System (Sənəd Çapı və İxrac Sistemi)

## 1. Base Class: `Document`

Bütün sənəd tipləri üçün əsas (Base) class yaradın.

* **Property-lər & Fields:**
  * `Title` (string) — `init` accessor ilə (obyekt yaradıldıqdan sonra dəyişdirilə bilməz).
  * `Author` (string) — `readonly` field və ya `init` ilə dəstəklənsin.
  * `RawContent` (string) — Sənədin mətni/məzmunu.
  * `PageCount` (int) — Sənədin səhifə sayı (`private set`).

* **Constructor:**
  * Parameter-lər: `string title`, `string author`, `string rawContent`
  * **Qaydalar:**
    * `title`, `author` və `rawContent` boş və ya `null` ola bilməz.
    * `PageCount` avtomatik olaraq mətndəki simvol sayına görə hesablanır:  
      $$\text{PageCount} = \left\lceil \frac{\text{rawContent.Length}}{500} \right\rceil \quad (\text{minimum 1 səhifə})$$

* **Overloaded Method-lar (`UpdateContent` - Compile-time Polymorphism):**
  * `UpdateContent(string newContent)` — Məzmunu yeniləyir və `PageCount`-u yenidən hesablayır.
  * `UpdateContent(string newContent, string appendNote)` — Məzmunun sonuna qeyd (`appendNote`) əlavə edərək yeniləyir və səhifə sayını hesablaıyr.

* **Virtual Method-lar (Runtime Polymorphism):**
  * **`CalculateExportSizeKB()` (virtual double)** — Sənədin fayl ölçüsünü hesablaıyr (Base klassda hər simvol = 1 bayt kimi götürülür):
    $$\text{SizeKB} = \frac{\text{rawContent.Length}}{1024.0}$$
  * **`Export()` (virtual string)** — Sənədi standart formatda ixrac edir.
    * **Format:**
      ```text
      [DOCUMENT EXPORT]
      Title: Annual Report
      Author: John Doe
      Pages: 3
      Size: 1.45 KB
      Content: "Company financial results..."
      ```
  * **`Print()` (virtual void)** — Sənədi konsola çap edir:
    ```text
    >>> Printing Document: 'Annual Report' by John Doe (3 pages)
    ```

---

## 2. Derived Class 1: `PdfDocument`

`Document` class-ından miras alan `PdfDocument` class-ı yaradın.

* **Əlavə Property-lər:**
  * `IsEncrypted` (bool) — Şifrələnib-şifrələnmədiyini bildirir.
  * `DpiResolution` (int) — Çap keyfiyyəti (məsələn: `300` DPI).

* **Constructor:**
  * Parameter-lər: `title`, `author`, `rawContent`, `isEncrypted`, `dpiResolution`
  * Base constructor-u `base(...)` ilə çağırılmalıdır.
  * `dpiResolution` 72-dən kiçik ola bilməz (əks halda default `300` təyin edilsin).

* **Method Overriding:**
  * **`CalculateExportSizeKB()` (override):**
    * PDF formatında metadata və resurslar olduğu üçün fayl ölçüsü böyüyür:
      $$\text{BaseSizeKB} = \text{base.CalculateExportSizeKB()}$$
      $$\text{PdfSizeKB} = (\text{BaseSizeKB} \times 1.5) + \left(\frac{\text{DpiResolution}}{100.0}\right)$$
    * Əgər `IsEncrypted == true` olarsa, şifrələmə baytları üçün ölçüyə əlavə **10.0 KB** gəlinir.
  * **`Export()` (override):**
    * Base ixracat məlumatını genişləndirir, tipi `PDF DOCUMENT` edir və əlavə olaraq şifrələmə statusu ilə DPI göstərir:
      ```text
      [PDF DOCUMENT EXPORT]
      Title: Project Architecture
      Author: Alice
      Pages: 2
      Size: 14.20 KB
      Security: Encrypted
      Quality: 300 DPI
      Content: "System diagram..."
      ```
  * **`Print()` (override):**
    ```text
    >>> Printing PDF [DPI: 300] [Encrypted: True]: 'Project Architecture' (Size: 14.20 KB)
    ```

---

## 3. Derived Class 2: `HtmlDocument`

`Document` class-ından miras alan `HtmlDocument` class-ı yaradın.

* **Əlavə Property-lər:**
  * `CssTheme` (string) — Stil teması (məsələn: "Dark", "Light", "Bootstrap").
  * `IncludeHeaderFooter` (bool) — Başlıq və sonluğun olub-olmaması.

* **Constructor:**
  * Parameter-lər: `title`, `author`, `rawContent`, `cssTheme`, `includeHeaderFooter`
  * Base constructor-u `base(...)` ilə çağırılmalıdır.

* **Method Overriding:**
  * **`CalculateExportSizeKB()` (override):**
    * HTML teqləri və CSS strukturu ölçünü artırır:
      $$\text{HtmlSizeKB} = \text{base.CalculateExportSizeKB()} + 2.5$$
      *(Əgər `IncludeHeaderFooter == true` olarsa, əlavə **1.2 KB** gəlinir).*
  * **`Export()` (override):**
    * Məzmunu HTML teqləri daxilinə alaraq ixrac edir:
      ```text
      [HTML DOCUMENT EXPORT]
      Title: Landing Page
      Author: Web Team
      Theme: Dark
      Size: 4.20 KB
      Rendered Code:
      <html>
        <head><title>Landing Page</title></head>
        <body class="theme-dark">
          "Welcome to our product..."
        </body>
      </html>
      ```
  * **`Print()` (override):**
    ```text
    >>> Rendering & Printing HTML Web Page [Theme: Dark]: 'Landing Page'
    ```

---

## 4. Derived Class 3: `CompressedTextDocument`

`Document` class-ından miras alan `CompressedTextDocument` class-ı yaradın.

* **Əlavə Property-lər:**
  * `CompressionRatioPercent` (double) — Sıxılma faizi (məsələn: `40.0` yəni %40 sıxılıb).

* **Constructor:**
  * Parameter-lər: `title`, `author`, `rawContent`, `compressionRatioPercent`
  * `compressionRatioPercent` 1.0 ilə 90.0 arasında olmalıdır.

* **Method Overriding:**
  * **`CalculateExportSizeKB()` (override):**
    * Fayl ölçüsü sıxılma dərəcəsinə görə kiçilir:
      $$\text{CompressedSizeKB} = \text{base.CalculateExportSizeKB()} \times \left(1.0 - \frac{\text{CompressionRatioPercent}}{100.0}\right)$$
  * **`Export()` (override):**
    ```text
    [ZIP TEXT DOCUMENT EXPORT]
    Title: Archive Logs
    Author: System
    Compression: 40% Reduced
    Size: 0.60 KB
    Content: "Compressed binary data stream..."
    ```
  * **`Print()` (override):**
    ```text
    >>> Decompressing and Printing Archive: 'Archive Logs' (Saved 40% disk space)
    ```

---

## 5. Document Manager (Polymorphic Array Processor): `DocumentPrinter`

Ayrı bir `DocumentPrinter` class-ı yaradın. Bu class sənədləri idarə etmək üçün **`List<T>` YERİNƏ yalnız Sabit Ölçülü Massiv (`Document[]`)** istifadə etməlidir.

* **Fields:**
  * `private Document[] _documents` — Sənəd obyektlərini saxlamaq üçün massiv.
  * `private int _count` — Massivə faktiki əlavə olunmuş sənədlərin sayı.

* **Constructor:**
  * `DocumentPrinter(int capacity)` — Massivin tutumunu (`capacity`) təyin edir və `_documents = new Document[capacity]` yaradır.

* **Method-lar:**
  * **`AddDocument(Document doc)`:**
    * Massivdə yer olub-olmadığını yoxlayır (`_count < _documents.Length`).
    * Yeri varsa `_documents[_count] = doc;` edib `_count`-u 1 artırır.
    * Massiv doludursa konsola məlumat verir.
  * **`PrintAll()` (POLYMORPHIC EXECUTION):**
    * Sabit `for` və ya `foreach` dövrü vasitəsilə massivdəki bütün sənədlərin **`Print()`** metodunu çağırır.
    * *Runtime zamanı massivin elementi `Document` tipində görünsə də, realda `PdfDocument`, `HtmlDocument` və ya `CompressedTextDocument`-a uyğun xüsusi `Print()` işləməlidir!*
  * **`ExportAll()` (POLYMORPHIC EXECUTION):**
    * Dövr ilə massivdəki hər bir sənədin **`Export()`** metodunu çağırır və ekrana yazdırır.
  * **`GetTotalExportSizeKB()` (POLYMORPHIC CALCULATION):**
    * Massivdəki bütün sənədlərin polimorfik **`CalculateExportSizeKB()`** metodlarını çağıraraq ümumi tutumu (KB) toplayıb qaytarır.

---

## 6. `Program.cs` (Polimorfizm və Massiv Test Ssenarisi)

`Main` metodu daxilində aşağıdakı addımları icra edin:

1. **Polimorfik Massiv Yaradılması (List yoxdur!):**
   * 4 elementlik `Document[]` tipli massiv elan edin və daxilini fərqli törəmə klas obyektləri ilə doldurun:
     ```csharp
     Document[] docArray = new Document[4];
     docArray[0] = new Document("Standard Doc", "Admin", "Simple text content...");
     docArray[1] = new PdfDocument("Manual", "Tech Writer", "Detailed steps...", true, 300);
     docArray[2] = new HtmlDocument("Home Page", "Developer", "Welcome page...", "Dark", true);
     docArray[3] = new CompressedTextDocument("Logs", "Server", "System logs data...", 50.0);
     ```

2. **`DocumentPrinter` Manager Testi:**
   * 5 tutumlu `DocumentPrinter` obyekti yaradın.
   * Yuxarıda yaradılmış `docArray` massivindəki obyektləri dövr ilə `DocumentPrinter`-ə əlavə edin (`AddDocument`).

3. **Polimorfik Çap və İxracat Testi:**
   * `printer.PrintAll()` metodunu çağıraraq hər bir sənəd tipinin özünəxas çap cümlələrini görün.
   * `printer.ExportAll()` metodunu çağıraraq fərqli formatlı mətn çıxarışlarını görün.

4. **Polimorfik Hesablama Testi:**
   * `printer.GetTotalExportSizeKB()` metodunu çağıraraq polimorfik ölçülərin düzgün toplanıb konsola yazdırıldığını görün.

5. **Compile-time Polymorphism (Overloading) Testi:**
   * Massivdəki 1-ci elementi (`Document` tipində olanı) götürün və `UpdateContent` overloaded metodlarını çağırın:
     * `docArray[0].UpdateContent("New updated text content.");`
     * `docArray[0].UpdateContent("New updated text content.", "Note: Approved by manager.");`
