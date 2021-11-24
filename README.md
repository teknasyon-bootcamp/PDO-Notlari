# PDO NOTLARI - Teknasyon PHP Bootcamp  

PDO veritabanlarına bağlanmamıza yarayan bir **API**'dır. (Okunuşu : AyPiAy, İng. Application Programming Interface, Tr. Uygulama Programlama Arayüzü) Bilgisayarımızda işletim sisteminin donanımlara ulaşmasını sağlayan yazılımlara driver denir. PDO API'ını buna benzeterek  veritabanlarıyla iletişim kurmamızı sağlayan bir driver olduğunu söyleyebiliriz. Her veritabanı işleminde veritabanı bağlantımzı kurmalıyız.

Bunun için bir DSN (Okunuş: DiEsEn, Ing. Data Source Name, Tr. Veri Kaynağı Adı) yazmamız gerekiyor ki PDO verileri hangi sunucu, port ve veritabınından (PDO MySQL dışında farklı veritabanlarını da desteklemektedir.) çekeceğini anlasın.

**host :** Veritabanının hangi makinada olduğunu belirtir. Veritabanı aynı makinada ise **localhost** uzak bir makinada ise sunucunun ip adresi yazılır. 

**port :** Bir sunucu birden fazla servisi sunabilir. Bu servislerin birbirleriyle çakışmaması için port (Tr. Liman) adı verilen bir yapı kullanılır. Örneğin 80 numaralı portu Nginx kullanırken aynı sunucudaki MySQL 3306 numaralı portu kullanarak hizmet verebilir. MySQL ve MariaDB varsayılan olarak 3306 numaralı portu kullanır. Eğer port bilgisi girmezseniz PDO otomatik olarak bu portu kullanacaktır.

**dbname:** Veritabanı adı.

```php 
$pdo = new PDO('mysql:host=localhost;port=3306;dbname=exampleDb','username','password');
```

Tablodaki verilerin hepsini dizi olarak çekmek için bir veritabanı sorgusu yazalım. Sondaki kısmı ileride açıklayacağız. Şu anlık orada sihir oluyor :mage: 

```php 
$query = $pdo->query('SELECT * FROM posts', PDO::FETCH_ASSOC);

$query->fetch();    //sadece ilk kaydı al
$query->fetch();    //sadece ikinci kaydı al
$query->fetch();    //sadece üçüncü kaydı al
```


Bunu teker teker yapmak yerine bir seferde çekebiliriz.
```php 
$query->fetchAll();    //Tüm kayıtları şak diye al
```

## PDO MODları

Bu modlarla PDO'ya verilerin hangi yapıda alınacağını  belirtiyoruz. Temel PDO modları şunlardır; 


**FETCH_ASSOC :** Kayıtları sütün adlarının index olduğu bir dizi olarak dönderir.

**FETCH_BOTH :** Varsayılandır. Kayıtları hem  sıfırdan başlayan bir index yapına hem de sütün adlarıyla oluşmuş bir index yapısına sahip dizi dönderir.

**FETCH_OBJ :** Kayıtları değer verilmediği takdirde anonim sınıf nesnesi olarak dönderir.

**FETCH_CLASS :** Kayıtları işaret edilen sınıfın nesnesi olarak dönderir.

**FETCH_BOUND :** Veritabanındaki her sütünün verisini bir değişken olarak getir. (Genellikle bindColumn ile kullanılır. İleride örneği var.)


En baştaki örneğimizi yeniden incelersek verileri dizi şeklinde almak istediğimizi fark edeceğiz. 

```php
$query = $pdo->query('SELECT * FROM posts', PDO::FETCH_ASSOC);
$posts = $query->fetchAll();
```

Normalde PDO modu direk query metodunun parametresi olarak yazılmaz. Bunun yerine fetchAll metoduna yazılır. Fakat ikisi de çalışacaktır. 

```php
$query = $pdo->query('SELECT  * FROM posts');
$posts = $query->fetchAll(PDO::FETCH_ASSOC);
```

Bu kez de gelen verileri Post sınfının nesneleri olarak alalım.
```php 
$query = $pdo->query('SELECT * FROM posts',PDO::FETCH_CLASS, Post::class);
$posts = $query->fetchAll();
```

Oluşturmak istediğim nesnelerin sınııfında contruct fonksiyonu çeşitli parametreler istiyorsa bunu şu şekilde verebilir.

```php 
$query = $pdo->query('SELECT * FROM posts');
$posts = $query->fetchAll(PDO::FETCH_CLASS, Post::class, [1,'parametre1', 'parametre2']);
```
Aynı sorgu şu şekillerde de yazılabilir.

```php
$query = $pdo->query('SELECT * FROM posts');
$query->setFetchMode(PDO::FETCH_CLASS, Post::class, [1,'parametre1', 'parametre2']);
$posts = $query->fetchAll();
```
veya... (Bu tek bir kayıt  çeker..)

```php
$query = $pdo->query('SELECT * FROM posts');
$posts = $query->fetchObject(Post::class, [1,'parametre1', 'parametre2']);
```

Yukarıdaki şekilde önce nesneye veritabanından gelen değerler aktarılır ardından varsa contruct metodu çalıştırılır. Bu da construct fonksiyonunda verdiğimiz değerleri ezebileceği anlamına gelir. Bunu engellemek istersek **FETCH_PROPS_LATE** sabitini fetch mode olarak bildirmemiz gerekir. Bunu yukarıdaki fetch modeları yazdığım yerlerin hepsine **|** karakterini kullanarak ekleyebiliriz. Örnek verecek olursak;

```php
$query = $pdo->query('SELECT * FROM posts');
$posts = $pdo->fetchAll(PDO::FETCH_CLASS|PDO::FETCH_PROPS_LATE , Post::class, [1,'parametre1', 'parametre2']);
```

## Kullanıcıdan Veri Alınan İşlemlerde Sorgular

```php 
$query = $pdo->query("INSERT INTO posts(title,content) VALUES('$title','$content')");
```
Yukarıda örnek olarak bir ekleme işlemi var. Bu şekilde de insert yapılabilir ancak güvenlik nedeniyle önerilmez. Peki önerilmiyorsa neden var? Çünkü bir sonraki adımda öğreneceğimiz **prepared** (Tr. Hazırlanmış) sorgu desteği her veritabanı sisteminde yok. Bu arada şu ana kadar öğrendiğimiz her yerde **query** fonksiyonu yerine **exec** de yazabilirdik. exec işlem başarılı olunca true değeri dönderirken query fonksiyonu **PdoStatement** nesnesi dönderir.

Kayıtları getirme gibi kullanıcıdan veri almadığımız veya tekrar etmeyen sorgularda illa prepare kullanmamıza gerek yok. Zaten prepare işlemi kaynak tükettiğinden **query** fonksiyonuyla yapmanız daha mantıklı. exec fonksiyonu genellikle veritabanındaki charset v.b. ayarları yapmakta kullanılır. exec ile veri **çekilemez**. 

### Nerede Ne Kullanacağız?
Aşağıdaki algoritma ile doğru yolu bulacaksınız. (Yani PDO'da :joy:)

![](https://i.imgur.com/9eqLgro.png)

Bir örnek olarak ekleme işlemi yaptık ama siz sorguyu değiştirerek güncelleme, silme işlemleri de yapabilirsiniz. Aşağıdaki sorgu 3 tane kaydı veritabanına kaydeder. İşlem başarılıysa execute fonksiyonu true değeri dönderir. **Execute** fonksiyonu aynı zamanda parametrelere verdiğimiz tüm değerlerimizi string olarak işler.

```php
$query = "INSERT INTO posts(title,content) VALUES(?,?)";
$insert = $pdo->prepare($query);
$insert->execute(['başlık1','içerik1']);
$insert->execute(['başlık2','içerik2']);
$insert->execute(['başlık3','içerik3']);
```

## execute ile Parametre Verme

Bunun yerine sorgudaki parametrelere isim verebiliriz. Böylelikle çok fazla parametreli işlemlerde karmaşıklıktan kurtuluruz. 

```php
$query = "INSERT INTO posts(title,content) VALUES(:title,:content)";
$insertNamed = $pdo->prepare($query);
$insertNamed->execute([
    ':title' => "Haydar'ın yazısı",
    ':content' => "Haydar'ın açıklaması"
]);
```
## bindValue 

Bazen parametrelerimizin nasıl işleneceğine karar vermek isteyebiliriz.  Bunun için **bindValue** fonsiyonunu kullanabiliriz.  Sondaki PDO param modları zorunlu değildir.

```php
$query = "INSERT INTO posts(title,content) VALUES(:title,:content)";
$insertNamed = $pdo->prepare($query);
$insertNamed->bindValue(':title', "Haydar'ın başlık",PDO::PARAM_STR);
$insertNamed->bindValue(':content', "Haydar'ın içerik",PDO::PARAM_STR);
$insertNamed->execute();
```

## bindParam

Aynı şeyi **bindParam** ile de yapabiliriz ancak bind param sabit değer değil değişken gibi referans alabileceği değerleri kabul eder. Ayrıca bindParam da parametrenin işleneceği tipin yanında parametrenin  uzunluğunu da belirleyebiliriz. Tip ve uzunluğu belirlemek zorunda değiliz.

```php
$title = "baslik";
$content = "içerik";
$query = "INSERT INTO posts(title,content) VALUES(:title,:content)";
$insertNamed = $pdo->prepare($query);

$insertNamed->bindParam(':title', $title,PDO::PARAM_STR,255);
$insertNamed->bindParam(':content', $content,PDO::PARAM_STR,255);
$insertNamed->execute();
```
Bind fonksiyonlarını illa dbye veri yollanırken kullanılacak diye bir şey yok. Adı üzerinde Bind yani bağlamak, birleştirmek... **bindColumn** fonksiyonu ile veritabanından gelen değerleri değişkenlere verebiliriz.

```php
$query = "SELECT * FROM posts";
$preparedQuery = $pdo->prepare($query);
$preparedQuery->execute();
$preparedQuery->bindColumn('id',$id);
$preparedQuery->bindColumn('title',$title);
$preparedQuery->bindColumn('content',$content);
while ($preparedQuery->fetch(PDO::FETCH_BOUND)) {
    echo "id : ". $id. '</br>'; 
    echo "title : ". $title. '</br>'; 
    echo "content : ". $content. '</br>'; 
    echo "--------------". '</br>';
}
```
