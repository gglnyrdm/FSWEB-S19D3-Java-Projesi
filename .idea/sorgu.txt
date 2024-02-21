1) ÖRNEK SORU: Yazar tablosunu KEMAL UYUMAZ isimli yazarı ekleyin.


 INSERT INTO yazar (yazarad, yazarsoyad) VALUES ('KEMAL', 'UYUMAZ');

2) Biyografi türünü tür tablosuna ekleyiniz.


INSERT INTO  tur(turadi) VALUES ('Biyografi')

3) 10A sınıfı olan ÇAĞLAR ÜZÜMCÜ isimli erkek, sınıfı 9B olan LEYLA ALAGÖZ isimli kız ve sınıfı 11C olan Ayşe Bektaş isimli kız öğrencileri tek sorguda ekleyin.


 INSERT INTO ogrenci (ograd, ogrsoyad, sinif, cinsiyet)
VALUES
    ('ÇAĞLAR', 'ÜZÜMCÜ', '10A', 'E'),
    ('LEYLA', 'ALAGÖZ', '9B', 'K'),
    ('Ayşe', 'Bektaş', '11C', 'K');


4) Öğrenci tablosundaki rastgele bir öğrenciyi yazarlar tablosuna yazar olarak ekleyiniz.

 INSERT INTO yazar (yazarad, yazarsoyad)
SELECT ograd, ogrsoyad FROM ogrenci
ORDER BY RANDOM()
LIMIT 1;

5) Öğrenci numarası 10 ile 30 arasındaki öğrencileri yazar olarak ekleyiniz.

 INSERT INTO yazar (yazarad,yazarsoyad)
SELECT ograd,ogrsoyad FROM ogrenci
WHERE ogrno > 30 AND ogrno < 50


6) Nurettin Belek isimli yazarı ekleyip yazar numarasını yazdırınız.
(Not: Otomatik arttırmada son arttırılan değer @@IDENTITY değişkeni içinde tutulur.)

INSERT INTO yazar(yazarad,yazarsoyad)
VALUES('NURETTIN','BELEK')
SELECT @IDENTITY as yazarno;

7) 10B sınıfındaki öğrenci numarası 3 olan öğrenciyi 10C sınıfına geçirin.

UPDATE ogrenci
SET sinif = '10C'
WHERE ogrno = 3

8) 9A sınıfındaki tüm öğrencileri 10A sınıfına aktarın

 UPDATE ogrenci
SET sinif = '10A'
WHERE sinif = '9A'

9) Tüm öğrencilerin puanını 5 puan arttırın.

  UPDATE ogrenci
SET puan= puan + 5

10) 25 numaralı yazarı silin.

DELETE FROM yazar WHERE yazarno = 25


11) Doğum tarihi null olan öğrencileri listeleyin. (insert sorgusu ile girilen 3 öğrenci listelenecektir)

SELECT * FROM ogrenci
WHERE dtarih IS NULL AND ograd LIKE 'u%';

12) Doğum tarihi null olan öğrencileri silin.

DELETE FROM ogrenci
WHERE dtarih IS NULL


13) Kitap tablosunda adı a ile başlayan kitapların puanlarını 2 artırın.

UPDATE kitap
SET puan = puan + 2
WHERE kitapadi LIKE 'a%'



14) Kişisel Gelişim isimli bir tür oluşturun.


INSERT INTO tur (turadi)
VALUES('Kişisel Gelişim')


15) Kitap tablosundaki Başarı Rehberi kitabının türünü bu tür ile değiştirin.

UPDATE kitap
SET turno = (
    SELECT turno
    FROM tur
    WHERE turadi = 'Kişisel Gelişim'
)
WHERE kitapadi = 'Başarı Rehberi'

16) Öğrenci tablosunu kontrol etmek amaçlı tüm öğrencileri görüntüleyen "ogrencilistesi" adında bir prosedür oluşturun.

CREATE FUNCTION teach.ogrenciListesi()
RETURNS SETOF teach.ogrenci
LANGUAGE 'sql'
AS $BODY$
    SELECT * FROM teach.ogrenci
$BODY$

SELECT * FROM teach.ogrenciListesi()

17) Öğrenci tablosuna yeni öğrenci eklemek için "ekle" adında bir prosedür oluşturun.

CREATE PROCEDURE teach.ekle(name character varying,surname character varying ,
 gender character varying , birtday character varying , class character varying , point integer)

 LANGUAGE 'sql'

 AS $BODY$
    INSERT INTO teach.ogrenci(ograd,ogrsoyad,cinsiyet,dtarih,sinif,puan)
    VALUES(name,surname,gender,birtday,class,point)
 $BODY$

CALL teach.ekle('Burak','Cevizli','E','null','10A','100')

18) Öğrenci noya göre öğrenci silebilmeyi sağlayan "sil" adında bir prosedür oluşturun.

CREATE PROCEDURE teach.sil(student_no integer)
LANGUAGE 'sql'
AS $BODY$
DELETE from teach.ogrenci
WHERE ogrno = student_no
$BODY$

CALL teach.sil(103)

19) Öğrenci numarasını kullanarak kolay bir biçimde öğrencinin sınıfını değiştirebileceğimiz bir prosedür oluşturun.
CREATE PROCEDURE guncelle(ogr_number smallint, ogr_sinif character varying)
LANGUAGE 'sql'
AS $BODY$
	UPDATE ogrenci
	SET sinif = ogr_sinif
	WHERE ogrno = ogr_number
$BODY$

CALL teach.updateClass(104,'10B')

20) Öğrenci adı ve soyadını "Ad Soyad" olarak birleştirip, ad soyada göre kolayca arama yapmayı sağlayan bir prosedür yazın.

CREATE FUNCTION teach.concat(full_name charactervarying)
RETURNS SETOF teach.ogrenci
LANGUAGE 'sql'
AS $BODY$
	SELECT * FROM teach.ogrenci
	WHERE CONCAT(ograd,ogrsoyad)
	ILIKE CONCAT('%',full_name,'%')
$BODY$

SELECT * FROM teach.concat

21) Daha önceden oluşturduğunu tüm prosedürleri silin.
DROP PROCEDURE IF EXIST 'ekle'
DROP PROCEDURE IF EXIST 'sil'
DROP PROCEDURE IF EXIST 'guncelle'

#Esnek görevler (Esnek görevlerin hepsini Select in Select ile gerçekleştirmeniz beklenmektedir.)
22) Select in select yöntemiyle dram türündeki kitapları listeleyiniz.

SELECT * FROM kitap
WHERE turno = (SELECT turno FROM tur WHERE turadi = 'DRAM')


23) Adı e harfi ile başlayan yazarların kitaplarını listeleyin.

SELECT kitapadi FROM kitap WHERE yazarno IN (SELECT yazarno FROM yazar WHERE yazarad LIKE 'E%')


24) Kitap okumayan öğrencileri listeleyiniz.

SELECT * FROM ogrenci WHERE ogrno NOT IN (SELECT ogrno FROM islem)


25) Okunmayan kitapları listeleyiniz

SELECT * FROM kitap WHERE kitapno NOT IN (SELECT kitapno FROM islem)


26) Mayıs ayında okunmayan kitapları listeleyiniz.

SELECT * FROM kitap
WHERE kitapno IN (SELECT kitapno FROM islem WHERE
EXTRACT (MONTH FROM atarih::timestamp) != 5
AND
EXTRACT (MONTH FROM vtarih::timestamp) != 5)