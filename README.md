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
