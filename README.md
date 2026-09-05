# Kino-Klub və Bilet Satışı Sistemi — SQL Tapşırıqlar Toplusu (50 Sual)

Bu sənəd **MSSQL (Microsoft SQL Server)** mühiti üçün nəzərdə tutulmuş 50 praktiki sorğu tapşırığını əhatə edir. Layihə bir **Kino-Klub və Bilet Satışı Sistemi** ssenarisi üzərində qurulmuşdur.

---

## 🏗️ Verilənlər Bazası Strukturu (Database Schema Overview)

Aşağıdakı cədvəllər və onlar arasındakı əlaqələr artıq bazada mövcuddur və məlumatlarla doldurulmuşdur.

# Kino-Klub və Bilet Satışı Sistemi — Cədvəllər və Sütunlar (Database Schema)

---

### 1. Filmlər (Movies)
* **`MovieID`** (INT, Primary Key, IDENTITY)
* **`Title`** (NVARCHAR(150), NOT NULL)
* **`ReleaseYear`** (INT)
* **`Rating`** (DECIMAL(3,1))
* **`Duration`** (INT) — *dəqiqə ilə*
* **`Status`** (NVARCHAR(50)) — *Aktiv, Arxiv və s.*

---

### 2. FilmTəfərrüatları (MovieDetails) — *(1:1 Əlaqə)*
* **`MovieID`** (INT, Primary Key, Foreign Key -> `Filmlər.MovieID`)
* **`Budget`** (DECIMAL(15,2))
* **`Country`** (NVARCHAR(100))
* **`Language`** (NVARCHAR(50))
* **`Description`** (NVARCHAR(MAX))

---

### 3. Janrlar (Genres)
* **`GenreID`** (INT, Primary Key, IDENTITY)
* **`GenreName`** (NVARCHAR(50), NOT NULL, UNIQUE)

---

### 4. FilmJanrları (MovieGenres) — *(N:M Keçid Cədvəli)*
* **`MovieID`** (INT, Foreign Key -> `Filmlər.MovieID`)
* **`GenreID`** (INT, Foreign Key -> `Janrlar.GenreID`)
* *(Primary Key: `MovieID` + `GenreID`)*

---

### 5. Aktyorlar (Actors)
* **`ActorID`** (INT, Primary Key, IDENTITY)
* **`FirstName`** (NVARCHAR(50), NOT NULL)
* **`LastName`** (NVARCHAR(50), NOT NULL)
* **`BirthDate`** (DATE)

---

### 6. FilmAktyorları (MovieActors) — *(N:M Keçid Cədvəli)*
* **`MovieID`** (INT, Foreign Key -> `Filmlər.MovieID`)
* **`ActorID`** (INT, Foreign Key -> `Aktyorlar.ActorID`)
* **`RoleName`** (NVARCHAR(100)) — *Aktyorun rolu (istəyə bağlı)*
* *(Primary Key: `MovieID` + `ActorID`)*

---

### 7. Zallar (Halls)
* **`HallID`** (INT, Primary Key, IDENTITY)
* **`HallName`** (NVARCHAR(50), NOT NULL)
* **`Capacity`** (INT, NOT NULL)

---

### 8. Seanslar (Sessions) — *(1:N Əlaqələr)*
* **`SessionID`** (INT, Primary Key, IDENTITY)
* **`MovieID`** (INT, Foreign Key -> `Filmlər.MovieID`)
* **`HallID`** (INT, Foreign Key -> `Zallar.HallID`)
* **`StartTime`** (DATETIME, NOT NULL)
* **`Status`** (NVARCHAR(50)) — *Active, Cancelled və s.*

---

### 9. Müştərilər (Customers)
* **`CustomerID`** (INT, Primary Key, IDENTITY)
* **`FirstName`** (NVARCHAR(50), NOT NULL)
* **`LastName`** (NVARCHAR(50), NOT NULL)
* **`Email`** (NVARCHAR(100))
* **`Phone`** (NVARCHAR(20))
* **`City`** (NVARCHAR(50))

---

### 10. MüştəriKartları (CustomerCards) — *(1:1 Əlaqə)*
* **`CustomerID`** (INT, Primary Key, Foreign Key -> `Müştərilər.CustomerID`)
* **`CardNumber`** (NVARCHAR(20), NOT NULL, UNIQUE)
* **`BonusPoints`** (INT, DEFAULT 0)
* **`Status`** (NVARCHAR(20)) — *Aktiv, Deaktiv*

---

### 11. Biletlər (Tickets) — *(1:N Əlaqələr)*
* **`TicketID`** (INT, Primary Key, IDENTITY)
* **`SessionID`** (INT, Foreign Key -> `Seanslar.SessionID`)
* **`CustomerID`** (INT, Foreign Key -> `Müştərilər.CustomerID`)
* **`SeatNumber`** (NVARCHAR(10), NOT NULL)
* **`Price`** (DECIMAL(10,2))
* **`PurchaseDate`** (DATETIME, DEFAULT GETDATE())

---

---

## 📝 50 Sorğu Tapşırığı (SQL Query Tasks)

### Bölmə 1: Əsas SQL Komandaları, Filtrləmə və Sıralama (1–10)

1. `Filmlər` cədvəlindən buraxılış ili 2020-ci ildən sonra olan filmlərin adını və xalını (Rating) çıxarın.
2. `Müştərilər` cədvəlində adında "ə" hərfi olan müştərilərin siyahısını soyadına görə əlifba sırası ilə sıralayın.
3. Reytinqi (Rating) 8.0 ilə 9.5 arasında olan filmlərin adını və reytinqini göstərin.
4. Nümayiş müddəti (Duration) 120 dəqiqədən çox olan ilk 5 filmi reytinqə görə azalan sırada çıxarın (`TOP` işlədin).
5. `Biletlər` cədvəlində qiyməti `NULL` olan (pulsuz verilmə ehtimalı olan) biletlərin siyahısını çıxarın.
6. `Müştərilər` cədvəlində elektron poçtu `gmail.com` ilə bitən müştəriləri tapın.
7. `Seanslar` cədvəlindən bu günə (cari tarixə) olan seansların siyahısını seans vaxtına görə nizamlansın.
8. Filmlərin siyahısını buraxılış ilinə görə azalan, eyni ildə olanları isə reytinqə görə artan sırada göstərin.
9. Qiyməti 10, 12 və ya 15 AZN olan biletlərin unikal (`DISTINCT`) qiymət siyahısını çıxarın.
10. `Aktyorlar` cədvəlində doğum tarixi 1980-ci ildən əvvəl olan aktyorların ad və soyadını gətirin.

---

### Bölmə 2: Constraints və Verilənlərin Dəyişdirilməsi / DML (11–15)

11. Xalı (Rating) 5.0-dən aşağı olan filmlərin statusunu `UPDATE` edərək "Arxiv" edin.
12. Telefon nömrəsi `NOT NULL` şərtini ödəməyən (yəni nömrəsi olmayan) müştərilərin siyahısını göstərin.
13. Qiyməti 5 AZN-dən az olan biletlərin qiymətini 1 AZN artırın.
14. Ləğv olunmuş seanslara (`Status = 'Cancelled'`) satılmış biletləri `Biletlər` cədvəlindən silin (`DELETE`).
15. `MüştəriKartları` cədvəlində balı 0 olan kartların statusunu "Deaktiv" olaraq yeniləyin.

---

### Bölmə 3: Aggregate Functions (16–22)

16. Kinoteatrda olan bütün filmlərin ortalama reytinq xalını (`AVG`) hesablayın.
17. Bazada ümumi neçə müştərinin qeydiyyatdan keçdiyini (`COUNT`) tapın.
18. Satılmış bütün biletlərdən əldə olunan ümumi gəliri (`SUM`) hesablayın.
19. Sistemdəki en baha biletin qiymətini (`MAX`) tapın.
20. Ən qısa filmin neçə dəqiqə olduğunu (`MIN`) tapın.
21. Elektron poçtu qeyd olunmuş (yəni `NULL` olmayan) müştərilərin sayını çıxarın.
22. `Biletlər` cədvəlində orta bilet qiyməti ilə ən baha bilet qiyməti arasındakı fərqi hesablayın.

---

### Bölmə 4: One-to-One (1:1) Əlaqəli Sorğular (23–27)

23. `Filmlər` və `FilmTəfərrüatları` cədvəlini birləşdirərək filmin adı ilə onun çəkiliş büdcəsini göstərin.
24. `Müştərilər` və `MüştəriKartları` cədvəllərini `INNER JOIN` edərək müştərinin adı, soyadı və kartındakı bonus balını çıxarın.
25. Hələ heç bir bonus kartı olmayan müştəriləri tapmaq üçün `Müştərilər` cədvəlini `MüştəriKartları` ilə `LEFT JOIN` edin və kart hissəsi `NULL` olanları süzgəcləyin.
26. Büdcəsi 50 milyon dollardan çox olan filmlərin adını və istehsalçı ölkəsini (`FilmTəfərrüatları` cədvəlindən) göstərin.
27. Bonus kartında 100-dən çox balı olan müştərilərin adını, soyadını və kart nömrəsini siyahılayın.

---

### Bölmə 5: One-to-Many (1:N) Əlaqəli Sorğular (28–36)

28. `Filmlər` və `Seanslar` cədvəllərini birləşdirərək hər seansın hansı filmə aid olduğunu və seans vaxtını göstərin.
29. `Zallar` və `Seanslar` cədvəllərini `INNER JOIN` edərək "Zal 1"-də keçiriləcək seansların siyahısını çıxarın.
30. `Biletlər` və `Müştərilər` cədvəllərini birləşdirərək "Əli Əliyev" adlı müştərinin aldığı biletlərin siyahısını göstərin.
31. Bütün filmləri və varsa onların seanslarını göstərin. Seansı olmayan filmlər də siyahıda çıxsın (`LEFT JOIN`).
32. `Seanslar` və `Biletlər` cədvəlini birləşdirərək saat 18:00-da başlayan seanslara satılan biletləri tapın.
33. Hələ heç bir bilet almamış müştərilərin siyahısını tapın (`LEFT JOIN` və `WHERE BiletID IS NULL`).
34. Hələ heç bir seansı təyin olunmamış filmlərin siyahısını çıxarın.
35. Müştərinin adı, aldığı biletin otacaq yeri (SeatNumber) və seansın başlama vaxtını eyni sorğuda göstərin.
36. Tutumu (Capacity) 100-dən çox olan zallarda təşkil olunan seansların siyahısını çıxarın.

---

### Bölmə 6: Many-to-Many (N:M) Əlaqəli Sorğular (37–44)

37. `Filmlər`, `FilmJanrları` və `Janrlar` cədvəllərini birləşdirərək filmlərin adını və qarşısında janrının adını göstərin.
38. `Filmlər`, `FilmAktyorları` və `Aktyorlar` cədvəllərini birləşdirərək "Inception" filmində çəkilən bütün aktyorların siyahısını çıxarın.
39. "Komediya" janrında olan bütün filmlərin adlarını və reytinqlərini siyahılayın.
40. "Bred Pitt" adlı aktyorun çəkildiyi bütün filmlərin adını və buraxılış ilini çıxarın.
41. Həm "Dram", həm də "Aksiyon" janrında olan filmləri tapmaq üçün uyğun `JOIN` sorğusu yazın.
42. Hələ heç bir janr mənsubiyyəti təyin olunmamış filmləri çıxarın.
43. Hələ heç bir filmə çəkilməmiş aktyorların siyahısını `LEFT JOIN` vasitəsilə tapın.
44. "Aksiyon" janrında olan və reytinqi 8.0-dən yüksək olan filmlərin siyahısını çıxarın.

---

### Bölmə 7: Mürəkkəb Birləşmələr (Multi-JOIN) və Qarışıq Sorğular (45–50)

45. **Çoxlu JOIN:** Müştərinin adı, aldığı biletin qiyməti, filmin adı və biletin aid olduğu zalın adını eyni sorğuda göstərin (`Müştərilər` + `Biletlər` + `Seanslar` + `Filmlər` + `Zallar`).
46. "Aksiyon" janrındakı filmlərə satılmış biletlərin ümumi məbləğini (`SUM`) hesablayın.
47. `FULL OUTER JOIN` istifadə edərək bütün zalları və bütün seansları eşləşdirin (uyğunlaşmayanlar da daxil olmaqla).
48. `CROSS JOIN` istifadə edərək bütün zallar ilə bütün seanslar arasında mümkün olan bütün kombinasiyaları generasiya edin.
49. Öz-özünə birləşmə (`Self Join`): `Müştərilər` cədvəlində eyni şəhərdə yaşayan başqa bir müştərisi olan müştəri cütlüklərini çıxarın.
50. "Dram" janrında olan, müddəti 100 dəqiqədən çox olan və "Zal 2"-də nümayiş olunan seansların siyahısını filmin adı və seans vaxtı ilə birgə göstərin.
