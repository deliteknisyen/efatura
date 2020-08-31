# 🧾 eFatura


### 🚩Kurulum

**Kendi verileriniz ile test etmek için:**

https://earsivportal.efatura.gov.tr/intragiris.html

**Test hesaplarıyla test etmek için:**

https://earsivportaltest.efatura.gov.tr/login.jsp

**Paket Kurulumu:**

    composer require deliteknisyen/efatura


### 🚩Özellikler

- Fatura oluşturma.
- İki tarih arası fatura sorgulama.
- Menü listesini görüntüleme.
- Fatura detaylarını görüntüleme.
- Türkçe veya İngilizce seçenekleriyle fatura modeli oluşturma.
- Fatura imzalama/onaylama.
- Faturayı HTML olarak çıktı alma.
- Faturanın indirme adresini alma.
- Faturayı iptal etme.
- Varolan bir faturayı sorgulama.
- Kullanıcı bilgilerini çekme. (Şirketinizin temel bilgileri)
- Kullanıcı bilgilerini güncelleme.
- SMS ile Fatura doğrulama ve onaylama.
- Faturayı PDF olarak çıktı alma.

### 🚩Örnekler

**Giriş**

Bir client oluşturarak genel yapıyı projemize dahil ediyoruz.
```php
use furkankadioglu\eFatura\InvoiceManager;
$client = new InvoiceManager();
```
Giriş bilgilerinizi chain fonksiyonlarla tanımlayabiliyorsunuz, bu production için geçerlidir.
```php
// Production environment
$client->setUsername("XXX")->setPassword("YYY");
// VEYA
$client->setCredentials("XXX", "YYY");
```

Alttaki kullanım ile **test modu**nda çalıştırabilir, firmanızın bilgileri olmadan otomatik test girişi yapabilirsiniz. Bu aşamadan sonraki tüm işlemleriniz test hesabıyla gerçekleşir.
```php
// Test Environment
$client->setDebugMode(true)->setTestCredentials();
```
Ayrıca bilgilerinizi görüntülemek isterseniz:
```php
$client->getCredentials();
```

**Faturalandırma**

Faturaların listelenmesi aşağıdaki şekilde yapılıyor:
```php
// Tüm faturaları listele
$client->getInvoicesFromAPI("01/01/2020", "08/02/2020");
```
Yeni bir fatura oluşturmak isterseniz, bir kaç seçeneğiniz mevcut, kullanım alışkanlığı olarak ingilizceye alışmışlar için iki farklı yöntem var, ilk aşamada Türkçe'den gidelim.

Örnek olarak şu şekilde bir fatura oluşturabiliriz:
```php
$fatura_detaylari  = [
"belgeNumarasi"  =>  "",
"faturaTarihi"  =>  "08/02/2020",
"saat"  =>  "09:07:48",
"paraBirimi"  =>  "TRY",
"dovzTLkur"  =>  "0",
"faturaTipi"  =>  "SATIS",
"vknTckn"  =>  "11111111111",
"aliciUnvan"  =>  "FURKAN KADIOGLU",
"aliciAdi"  =>  "FURKAN",
"aliciSoyadi"  =>  "KADIOGLU",
"binaAdi"  =>  "",
"binaNo"  =>  "",
"kapiNo"  =>  "",
"kasabaKoy"  =>  "",
"vergiDairesi"  =>  "MALTEPE",
"ulke"  =>  "Türkiye",
"bulvarcaddesokak"  =>  "DENEME SK. DENEME MAH.",
"mahalleSemtIlce"  =>  "",
"sehir"  =>  " ",
"postaKodu"  =>  "",
"tel"  =>  "",
"fax"  =>  "",
"eposta"  =>  "",
"websitesi"  =>  "",
"iadeTable"  => [],
"ozelMatrahTutari"  =>  "0",
"ozelMatrahOrani"  =>  0,
"ozelMatrahVergiTutari"  =>  "0",
"vergiCesidi"  =>  " ",
"malHizmetTable"  => [],
"tip"  =>  "İskonto",
"matrah"  =>  100,
"malhizmetToplamTutari"  =>  100,
"toplamIskonto"  =>  "0",
"hesaplanankdv"  =>  18,
"vergilerToplami"  =>  18,
"vergilerDahilToplamTutar"  =>  118,
"odenecekTutar"  =>  118,
"not"  =>  "xxx",
"siparisNumarasi"  =>  "",
"siparisTarihi"  =>  "",
"irsaliyeNumarasi"  =>  "",
"irsaliyeTarihi"  =>  "",
"fisNo"  =>  "",
"fisTarihi"  =>  "",
"fisSaati"  =>  " ",
"fisTipi"  =>  " ",
"zRaporNo"  =>  "",
"okcSeriNo"  =>  ""
];
```
Faturayı oluşturmak yetmez tabi, ürün veya hizmet de girmek lazım, oda şu şekilde oluyor.
```php
$fatura_detaylari["malHizmetTable"][] = [
"malHizmet"  =>  "Yazılım Geliştirme",
"miktar"  =>  28,
"birim"  =>  "DAY",
"birimFiyat"  =>  "3",
"fiyat"  =>  "84",
"iskontoArttm"  =>  "İskonto",
"iskontoOrani"  =>  0,
"iskontoTutari"  =>  "0",
"iskontoNedeni"  =>  "",
"malHizmetTutari"  =>  "99",
"kdvOrani"  =>  18,
"kdvTutari"  =>  "15.12",
"vergininKdvTutari"  =>  "0"
];
```
Değişkenler Türkçe olduğundan dolayı **mapWithTurkishKeys** fonksiyonunu kullanıyoruz.
```php
use furkankadioglu\eFatura\Invoice;
$inv  =  new Invoice();
$inv->mapWithTurkishKeys($fatura_detaylari); // Key yapısı türkçe 🇹🇷
// VEYA
$inv->mapWithEnglishKeys($invoice_details); // Key yapısı ingilizce 🇺🇸
```

Sonrasında bunu InvoiceManager'a kayıt etmemiz gerekiyor. Oda bu şekilde:
```php
$client->setInvoice($inv);
```
Sonrasında da taslak oluşturuyoruz:
```php
$client->createDraftBasicInvoice();
```

**Kullanıcı Bilgileri**

Bu kısım firmanızın eArşiv'de kayıtlı olan bilgileridir. Bu bilgileri alabilir ve güncelleyebilirsiniz.

👉Aynı zamanda bu bilgileri almak, fatura oluştururken ihtiyaç duyacağınız bir çok veri ihtiyacınızı da karşılar.

```php
$userInformations = $client->getUserInformationsData();
```
Bu işlem size bir adet UserInformations sınıfı döndürür. Bu sınıftaki verilerinizin tamamını set ve get metodlarıyla değiştirebilirsiniz:

```php
// Sadece vknTckn değiştirilemez.
$userInformations = $userInformations->setUnvan("FRKN Yazılım")->setApartmanNo("4");
$apartmanNo = $userInformations->getApartmanNo(); // 4
```

Ayrıca bu sınıfın verilerini toplu olarak almak isterseniz aşağıdaki kullanımı uygulayabilirsiniz, aynı fonksiyon Invoice sınıfı içinde geçerli:

```php
$userInformations->export(); // Array olarak tüm değişkenleri döndürür.
```


Aynı zamanda bu sınıfı kendiniz oluşturabilir ve array olarak veriyi sağlayabilirsiniz. Sonrasında da şu şekilde sunucuya göndeririz:

```php
$client->setUserInformations($userInformations); // Manager'a tanımla.
$client->sendUserInformationsData(); // Sunucuya gönder.
```

### 🚩Fonksiyonel Özellikler
(İndirme/Onaylama/HTML Çıktısını Alma/İptal vb.)


**Onaylamak için:**
```php
$client->signDraftInvoice();
```
**HTML çıktısını almak için:**
```php
$client->getInvoiceHTML();
```

**PDF çıktısını almak için:**
```php
$client->getInvoicePDF();
```

**İndirme linkini almak için:**
```php
$client->getDownloadURL();
```

**Faturayı iptal etmek için:**
```php
$client->cancelInvoice();
```

**SMS doğrulaması yapmak için:**
```php
$client->sendSMSVerification($telefon); // Operasyon id döndürür.
```

**SMS doğrulamasını onaylamak için:**
```php
$client->verifySMSCode($kod, $operasyonId);
```

**Varolan bir faturayı sorgulamak için:**
```php
$oldInvoice = new Invoice();
$oldInvoice->setUuid("e8277cfa-4ac9-11ea-a5b5-acde48001122");
$client->setInvoice($oldInvoice)->getInvoiceFromAPI();
// {"faturaUuid":"8a4423bc-4aca-11ea-8c30-acde48001122","faturaTarihi":"09\/02\/2020"...
```

### 🚩Alternatif Kullanımlar

**Kısaltılmış Kullanımlar:**

Uzun gelmiş olabilir. 😂 Gayet doğal, chain methodlar ile hayatımızı kolaylaştırıyoruz. Tek satırda işimizi halledelim:
```php
$client->setDebugMode(true) // Test urlsine geçtik 
->setTestCredentials() // Test bilgilerini aldık
->setInvoice($inv) // Faturamızı sınıfa tanımladık
->createDraftBasicInvoice() // Taslak faturamızı oluşturduk
->signDraftInvoice() // Faturamızı onayladık
->getDownloadURL(); // İndirme adresini aldık

// https://earsivportaltest.efatura.gov.tr/earsiv-services/download?token=b8b6c261c511a9b2757279c0111b538a2f02d98ae2df6205448d002687cab8cf74ce04d187bf0c6ce839dee40a6a8aad003aa6d5946ba02a0942ceb10bde327f&ettn=85933f42-4ab1-11ea-922e-acde48001122&belgeTip=FATURA&onayDurumu=Onaylandı&cmd=downloadResource
```

**Sabit Değişkenler:**

Bir çok farklı veri tipi olduğundan ve önceden bilinmediğinde sorunlar çıkabileceğini düşünerek, bazı ihtiyaç duyulan sabit seçenekler de mevcut. Tüm değişken isimleri eArşiv de görünenlerle birebir yapıldı. Örnekten bazılarını görebilirsiniz:

```php
use furkankadioglu\eFatura\Models\Country;
use furkankadioglu\eFatura\Models\CurrencyType;
use furkankadioglu\eFatura\Models\InvoiceType;
use furkankadioglu\eFatura\Models\UnitType;

$gunBirim = UnitType::GUN; // DAY
$turkLirasi = CurrencyType::TURK_LIRASI; // TRY
$satisFaturasi = InvoiceType::SATIS; // SATIŞ
$gurcistanUlkesi = Country::GURCISTAN; // Gürcistan
```

**Anahtar Yapısını Değiştirme:**

```php
use furkankadioglu\eFatura\Invoice;
$inv  =  new Invoice();

$invoice_details = [
    "uuid" => $uuid,
    "documentNumber" => $documentNumber,
    "date" => $date,
    "time" => $time,
    "currency" => $currency,
    "currencyRate" => $currencyRate,
    "invoiceType" => $invoiceType,
    "taxOrIdentityNumber" => $taxOrIdentityNumber,
    "invoiceAcceptorTitle" => $invoiceAcceptorTitle,
    "invoiceAcceptorName" => $invoiceAcceptorName,
    "invoiceAcceptorLastName" => $invoiceAcceptorLastName,
    "buildingName" => $buildingName,
    "buildingNumber" => $buildingNumber,
    "doorNumber" => $doorNumber,
    "town" => $town,
    "taxAdministration" => $taxAdministration,
    "country" => $country,
    "avenueStreet" => $avenueStreet,
    "district" => $district,
    "city" => $city,
    "postNumber" => $postNumber,
    "telephoneNumber" => $telephoneNumber,
    "faxNumber" => $faxNumber,
    "email" => $email,
    "website" => $website,
    "refundTable" => $refundTable,
    "specialBaseAmount" => $specialBaseAmount,
    "specialBasePercent" => $specialBasePercent,
    "specialBaseTaxAmount" => $specialBaseTaxAmount,
    "taxType" => $taxType,
    "itemOrServiceList" => $itemOrServiceList,
    "type" => $type,
    "base" => $base,
    "itemOrServiceTotalPrice" => $itemOrServiceTotalPrice,
    "totalDiscount" => $totalDiscount,
    "calculatedVAT" => $calculatedVAT,
    "taxTotalPrice" => $taxTotalPrice,
    "includedTaxesTotalPrice" => $includedTaxesTotalPrice,
    "paymentPrice" => $paymentPrice,
    "note" => $note,
    "orderNumber" => $orderNumber,
    "orderData" => $orderData,
    "waybillNumber" => $waybillNumber,
    "waybillDate" => $waybillDate,
    "receiptNumber" => $receiptNumber,
    "voucherDate" => $voucherDate,
    "voucherTime" => $voucherTime,
    "voucherType" => $voucherType,
    "zReportNumber" => $zReportNumber,
    "okcSerialNumber" => $okcSerialNumber
];

$inv->mapWithEnglishKeys($invoice_details); // Key yapısı ingilizce
```

Bu şekilde de map edebileceğiniz gibi ayrıyetten getter/setter methodları da mevcut, istediğiniz her veriyi düzenleyebilirsiniz:

```php
$inv->setUuid("Buraya kendi fatura idniz") 
->setCountry("Türkiye")
->getCurrencyRate(); // TRY
```

**Toplu veri alımı ve çıkartımı:**

Fatura verisinin değişken değerlerini toplu olarak ekleyebilir veya çıkartabiliriz, şöyle:
```php
    $inv = new Invoice($data); // data arrayinden keylere göre tüm verileri alır.
    $inv->export(); // tüm verileri çıkartır.
```

### 🚩Diğer Konular

**Testleri Çalıştırma:**

```
composer test
```

**Daha Fazla Örnek:**

Daha fazla örneği [buradan](https://github.com/furkankadioglu/efatura/blob/master/example/index.php "buradan")` bulabilirsiniz.

**Uyarı**

🚨 Bu paket vergiye tabi olan belge oluşturur, hiç bir sorumluluk kabul edilmez ve ne yaptığınızdan emin olana kadar debugMode açık şekilde test verileriyle işlem yapmanız önerilir.

**Ayrıca**

Bu proje Fatih Kadir Akın'ın  [fatura.js](https://github.com/f/fatura "fatura")` projesinden yola çıkılarak PHP diline uyarlanarak yapılmıştır. Arda Kılıçdağı'na da ayrıca teşekkürler.
