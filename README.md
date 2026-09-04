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

-- MSSQL TSQL Script: Cinema Club Database Sample Data

-- 1. Genres (Janrlar)
INSERT INTO Janrlar (GenreName) VALUES 
(N'Action'), 
(N'Drama'), 
(N'Comedy'), 
(N'Sci-Fi'), 
(N'Thriller'),
(N'Horror'),
(N'Adventure'),
(N'Animation'),
(N'Crime'),
(N'Biography');

-- 2. Halls (Zallar)
INSERT INTO Zallar (HallName, Capacity) VALUES 
(N'Hall 1 - IMAX', 200),
(N'Hall 2 - Standard', 100),
(N'Hall 3 - Standard', 100),
(N'Hall 4 - Dolby Atmos', 150),
(N'VIP Hall', 35);

-- 3. Actors (Aktyorlar)
INSERT INTO Aktyorlar (FirstName, LastName, BirthDate) VALUES 
(N'Leonardo', N'DiCaprio', '1974-11-11'),
(N'Brad', N'Pitt', '1963-12-18'),
(N'Cillian', N'Murphy', '1976-05-25'),
(N'Matthew', N'McConaughey', '1969-11-04'),
(N'Morgan', N'Freeman', '1937-06-01'),
(N'Christian', N'Bale', '1974-01-30'),
(N'Tom', N'Hardy', '1977-09-15'),
(N'Anne', N'Hathaway', '1982-11-12'),
(N'Scarlett', N'Johansson', '1984-11-22'),
(N'Robert', N'Downey Jr.', '1965-04-04');

-- 4. Movies (Filmlər)
INSERT INTO Filmlər (Title, ReleaseYear, Rating, Duration, Status) VALUES 
(N'Inception', 2010, 8.8, 148, N'Active'),
(N'Interstellar', 2014, 8.7, 169, N'Active'),
(N'Oppenheimer', 2023, 8.9, 180, N'Active'),
(N'The Shawshank Redemption', 1994, 9.3, 142, N'Archived'),
(N'Fight Club', 1999, 8.8, 139, N'Archived'),
(N'The Dark Knight', 2008, 9.0, 152, N'Active'),
(N'Dunkirk', 2017, 7.8, 106, N'Active'),
(N'The Wolf of Wall Street', 2013, 8.2, 180, N'Archived'),
(N'The Prestige', 2006, 8.5, 130, N'Active'),
(N'Avengers: Endgame', 2019, 8.4, 181, N'Active');

-- 5. MovieDetails (FilmTəfərrüatları) - (1:1 Relationship, MovieID = 1 to 10)
INSERT INTO FilmTəfərrüatları (MovieID, Budget, Country, Language, Description) VALUES 
(1, 160000000.00, N'USA', N'English', N'A thief who steals corporate secrets through the use of dream-sharing technology.'),
(2, 165000000.00, N'USA', N'English', N'A team of explorers travel through a wormhole in space in an attempt to ensure humanity survival.'),
(3, 100000000.00, N'USA', N'English', N'The story of American scientist J. Robert Oppenheimer and his role in the development of the atomic bomb.'),
(4, 25000000.00, N'USA', N'English', N'Over the course of several years, two convicts form a friendship, seeking solace and eventual redemption.'),
(5, 63000000.00, N'USA', N'English', N'An insomniac office worker and a devil-may-care soap maker form an underground fight club.'),
(6, 185000000.00, N'USA', N'English', N'When the menace known as the Joker wreaks havoc and chaos on the people of Gotham, Batman must accept one of the greatest psychological tests.'),
(7, 100000000.00, N'UK', N'English', N'Allied soldiers from Belgium, the British Empire, and France are surrounded by the German Army during World War II.'),
(8, 100000000.00, N'USA', N'English', N'Based on the true story of Jordan Belfort, from his rise to a wealthy stock-broker to his fall involving crime.'),
(9, 40000000.00, N'USA', N'English', N'After a tragic accident, two stage magicians in 1890s London engage in a battle to create the ultimate illusion.'),
(10, 356000000.00, N'USA', N'English', N'After the devastating events of Infinity War, the Avengers assemble once more to reverse Thanos actions.');

-- 6. MovieGenres (FilmJanrları) - (N:M Junction Table)
INSERT INTO FilmJanrları (MovieID, GenreID) VALUES 
(1, 1), (1, 4), (1, 5),
(2, 4), (2, 2), (2, 7),
(3, 2), (3, 10),
(4, 2), (4, 9),
(5, 2), (5, 5),
(6, 1), (6, 2), (6, 9),
(7, 1), (7, 2),
(8, 2), (8, 3), (8, 9),
(9, 2), (9, 5),
(10, 1), (10, 4), (10, 7);

-- 7. MovieActors (FilmAktyorları) - (N:M Junction Table)
INSERT INTO FilmAktyorları (MovieID, ActorID, RoleName) VALUES 
(1, 1, N'Cobb'),
(1, 7, N'Eames'),
(2, 4, N'Cooper'),
(2, 8, N'Brand'),
(3, 3, N'J. Robert Oppenheimer'),
(3, 10, N'Lewis Strauss'),
(4, 5, N'Ellis Boyd ''Red'' Redding'),
(5, 2, N'Tyler Durden'),
(6, 6, N'Bruce Wayne / Batman'),
(6, 3, N'Dr. Jonathan Crane'),
(7, 7, N'Farrier'),
(7, 3, N'Shiver'),
(8, 1, N'Jordan Belfort'),
(9, 6, N'Alfred Borden'),
(9, 8, N'Olivia Wenscombe'),
(10, 10, N'Tony Stark / Iron Man'),
(10, 9, N'Natasha Romanoff');

-- 8. Sessions (Seanslar)
INSERT INTO Seanslar (MovieID, HallID, StartTime, Status) VALUES 
(1, 1, '2026-09-05 15:00:00', N'Active'),
(1, 2, '2026-09-05 18:00:00', N'Active'),
(2, 1, '2026-09-05 20:00:00', N'Active'),
(3, 5, '2026-09-06 14:00:00', N'Active'),
(4, 2, '2026-09-06 17:00:00', N'Cancelled'),
(6, 1, '2026-09-06 21:00:00', N'Active'),
(6, 4, '2026-09-07 16:30:00', N'Active'),
(8, 3, '2026-09-07 19:00:00', N'Active'),
(10, 4, '2026-09-08 18:00:00', N'Active'),
(9, 2, '2026-09-08 21:30:00', N'Active');

-- 9. Customers (Müştərilər)
INSERT INTO Müştərilər (FirstName, LastName, Email, Phone, City) VALUES 
(N'John', N'Doe', N'john.doe@gmail.com', N'+15550123', N'New York'),
(N'Jane', N'Smith', N'jane.smith@gmail.com', N'+15550987', N'New York'),
(N'Robert', N'Johnson', N'robert.j@mail.com', N'+15550333', N'Los Angeles'),
(N'Alex', N'Brown', NULL, N'+15550444', N'Chicago'),
(N'Emily', N'Davis', N'emily.davis@yahoo.com', N'+15550777', N'New York'),
(N'Michael', N'Wilson', N'm.wilson@gmail.com', N'+15550999', N'Houston'),
(N'David', N'Taylor', NULL, NULL, N'Los Angeles'),
(N'Sarah', N'Anderson', N'sarah.a@gmail.com', N'+15550111', N'Chicago');

-- 10. CustomerCards (MüştəriKartları) - (1:1 Relationship)
INSERT INTO MüştəriKartları (CustomerID, CardNumber, BonusPoints, Status) VALUES 
(1, N'CARD1001', 120, N'Active'),
(2, N'CARD1002', 45, N'Active'),
(3, N'CARD1003', 0, N'Inactive'),
(5, N'CARD1004', 210, N'Active'),
(6, N'CARD1005', 85, N'Active');

-- 11. Tickets (Biletlər)
INSERT INTO Biletlər (SessionID, CustomerID, SeatNumber, Price, PurchaseDate) VALUES 
(1, 1, N'A1', 12.00, '2026-09-01 10:30:00'),
(1, 1, N'A2', 12.00, '2026-09-01 10:30:00'),
(2, 2, N'B5', 15.00, '2026-09-02 14:15:00'),
(3, 3, N'VIP1', 25.00, '2026-09-03 16:45:00'),
(1, 4, N'C10', NULL, '2026-09-04 09:00:00'),
(6, 5, N'A10', 18.00, '2026-09-04 11:20:00'),
(6, 6, N'A11', 18.00, '2026-09-04 12:00:00'),
(7, 1, N'D4', 14.00, '2026-09-04 15:30:00'),
(9, 2, N'F12', 16.00, '2026-09-04 18:00:00'),
(10, 8, N'B2', 10.00, '2026-09-04 19:10:00');

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
