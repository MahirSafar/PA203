# EduCorp Management System - Praktik C# Layihə Tapşırığı

Bu tapşırıq proqramlaşdırma əsaslarından başlayaraq OOP, Generics, Yaddaş İdarəetməsi, Fayl Sistemləri və Serialization mövzularını əhatə edən kompleks konsol tətbiqidir.

---

## 🎯 Layihənin Məqsədi
Təlim mərkəzi üçün tələbələrin və kursların idarə olunmasını təmin edən, verilənləri JSON faylında saxlayan tam funksional sistem hazırlamaq.

---

## 📋 Tapşırıq Mərhələləri və Struktur

### HİSSƏ 1: Tiplər, Abstraktsiyalar və Modeller

#### 1. Enum-lar
* **`StudentType`**: `Standard`, `Scholarship`, `VIP`
* **`CourseCategory`**: `Programming`, `Design`, `Marketing`

#### 2. Xüsusi İstisnalar (Custom Exceptions)
* **`NotFoundException`**: Axtarılan id-li obyekt tapılmadıqda atılır.
* **`CapacityExceededException`**: Kursda limit dolduqda atılır.

#### 3. İnterfeyslər (Interfaces)
* **`IEntity`**:
  * `int Id { get; }`
* **`IBaseService<T>`** (Generic Interface, `where T : class, IEntity` constraint ilə):
  * `void Add(T entity)`
  * `T GetById(int id)`
  * `List<T> GetAll()`
  * `void Remove(int id)`

#### 4. Baza və Model Sinifləri
* **`Person`** (Abstract Class — `implements IEntity`):
  * `private static int _idCounter` (Statik id generatoru)
  * `public int Id { get; }`
  * `public string Name { get; set; }`
  * `public string Surname { get; set; }`
  * `public readonly DateTime CreatedAt;` (Obyekt yarandığı anı saxlayır)
  * `public abstract string GetRole();` (Abstract metod)

* **`Student`** (Sealed Class — `inherits Person`):
  * `private double _gpa;` → Encapsulation: `GPA` yalnız 0.0 ilə 4.0 arasında ola bilər, əks halda exception fırladılmalıdır.
  * `public StudentType Type { get; set; }`
  * `public override string GetRole()` → Polymorphism istifadə edərək `"Student"` mətni qaytarır.
  * **Explicit Operator Casting:** `Student` obyekti `(string)` kimi cast edildikdə `"Name Surname - GPA: X"` formatında mətn qaytarmalıdır.

* **`Course`** (`implements IEntity`):
  * `public int Id { get; }`
  * `public string Title { get; init; }` (Init-only property)
  * `public CourseCategory Category { get; set; }`
  * `public const int MaxCapacity = 20;` (Const field)
  * `private List<Student> _students;`
  * **Indexer:** `public Student this[int index]` → Kurs daxilindəki tələbəyə indeksi ilə (`course[0]`) müraciət imkanı yaradır.

---

### HİSSƏ 2: Helper və Extension Sinifləri

#### 1. Extension Metodlar
* **`StringExtensions`** (Static Class):
  * `ToCapitalize(this string text)` extension metodu daxil edilən string-in ilk hərfini böyük, qalanlarını kiçik hərfə çevirməlidir (Məsələn: `"aLİ"` → `"Ali"`).

#### 2. Helper Metodlar
* **`DateTimeHelper`** (Static Class):
  * `CalculateAge(DateTime birthDate)` metodu doğum tarixi ilə bugünkü tarix arasındakı fərqi `TimeSpan` istifadə edərək yaş kimi hesablayır və qaytarır.

---

### HİSSƏ 3: Servis Təbəqəsi, Yaddaş İdarəetməsi və Serialization

#### 1. Generic Servis Sinifi
* **`CourseManager<T>`** (Generic Class — `implements IBaseService<T>, IDisposable`, constraint: `where T : Person, IEntity`):
  * Daxildə `List<T>` saxlayır.
  * `Add` metodu daxilində `MaxCapacity` yoxlanılır. Əgər limit aşılarsa `CapacityExceededException` fırladılır.
  * `IDisposable` realizasiyası edilməlidir (`Dispose()` çağırıldıqda `GC.SuppressFinalize(this)` istifadə olunaraq resurslar azad olunmalıdır).

#### 2. Fayl Sistemləri və Serialization
* **`DataManager`** (Static Class):
  * Qovluqda `Files` qovluğunun və daxilində `database.json` faylının olub-olmadığını `Directory` və `File` sinifləri vasitəsilə yoxlayır. Yoxdursa avtomatik yaradır.
  * `SaveToJson<T>(T data)` → Obyektləri **JSON Serialization** edərək fayla yazır (`StreamWriter` vasitəsilə).
  * `LoadFromJson<T>()` → JSON faylını oxuyur (`StreamReader` vasitəsilə) və **Deserialization** edib obyekti qaytarır.

---

### HİSSƏ 4: Program.cs və Konsol Interfeysi

Proqram başladıqda `Files/database.json` faylı avtomatik oxunmalı (Deserialization) və mövcud məlumatlar yaddaşa yüklənməlidir.

#### Menyu Strukturu:
```text
=== EDUCORP MANAGEMENT SYSTEM ===
1. Yeni Tələbə Əlavə Et
2. ID-yə Görə Tələbə Axtar
3. Tələbəni Sil
4. Bütün Tələbələri Çapa Çıxart
5. Məlumatları JSON Faylına Yadda Saxla (Save)
0. Çıxış
```

#### İcra Şərtləri və Qaydalar:
1. **Daxil etmə (Option 1):** İstifadəçidən ad və soyad alındıqdan sonra `ToCapitalize()` extension metodu ilə formata salınır. `GPA` dəyəri encapsulation qaydalarına uyğun validasiya olunur.
2. **Axtarış (Option 2):** İstifadəçi ID daxil edir. Tələbə Xətti Axtarış (Linear Search - O(n)) alqoritmi və ya Indexer istifadə edilərək tapılır. Tapılmadıqda `NotFoundException` atılır və `try-catch` bloku ilə tutulub ekrana xəta mesajı yazılır.
3. **Silmə (Option 3):** Daxil edilən ID-li obyekt tapılır və siyahıdan çıxarılır.
4. **Siyahılama (Option 4):** Bütün tələbələr `foreach` dövrü ilə fırlanır. Hər tələbənin `GetRole()` metodu çağırılır və ekrana yazdırılarkən `(string)student` istifadə edilərək **Explicit Casting** icra olunur.
5. **Yadda Saxlama (Option 5):** `using` blokundan istifadə edilərək `DataManager` vasitəsilə verilənlər `database.json` faylına yazılır.

---

## 🛠 Cavablandırılmalı Vurğular (Checklist)

- [ ] `if-else`, `switch-case`, `do-while`, `foreach` dövr strukturları
- [ ] Interface və Abstract Class realizasiyası
- [ ] Encapsulation, Inheritance və Polymorphism məntiqi
- [ ] Generic Class və Generic Interface (`where` constraint ilə)
- [ ] Linear Search alqoritmi və Big-O anlayışı (O(n))
- [ ] Indexer, Custom Exception, Enums və Static Class/Extension Metodlar
- [ ] Explicit Casting Operator (`(string)student`)
- [ ] `init`, `readonly`, `const` açar sözlərinin istifadəsi
- [ ] `IDisposable` və `GC.SuppressFinalize` ilə yaddaş idarəetməsi
- [ ] JSON Serialization/Deserialization, `StreamWriter`/`StreamReader`, `using` statement
