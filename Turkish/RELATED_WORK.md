# İlgili çalışmalar

İngilizce asıl metin: [RELATED_WORK.md](../RELATED_WORK.md). İki metin arasında
fark olursa İngilizce metin geçerlidir.

Adresler ve yayımlanmış kod 2026-09-27 tarihinde, beyan edilen lisanslar
2026-09-28 tarihinde denetlenmiştir. Aşağıdaki depo oluşturma tarihleri bağlam
sağlar, bir kaydı ilk kimin yorumladığını kanıtlamaz. Başka çalışmalar gözden
kaçmış olabilir.

## Genel biçim tanımları

- **PKWARE APPNOTE**, `.evo` dosyalarının kullandığı ZIP kapsayıcısını tanımlar
  <https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT>
- **ISO 10303-21:2016**, proje kaydının kullandığı düz metin alışveriş sözdizimini
  tanımlar. DIALux'a özgü alanların anlamını tanımlamaz
  <https://www.iso.org/standard/63141.html>

## DIALux biçimleri üzerine kamuya açık çalışmalar

- **EvoParse** (GitHub kullanıcısı luciodias), oluşturulma tarihi 2026-06-26.
  README dosyasında MIT yazıyor ancak depoda lisans dosyası yok.
  <https://github.com/luciodias/EvoParse>
  `.evo` / `.dat` dosyalarının `ProjectData` kaydını ISO 10303-21 (STEP) metni
  olarak okur ve `ProjectData.xml` üzerinden üretilen, türleri tanımlı Python
  modellerine dönüştürür

- **evo-reader** (GitHub kullanıcısı abubakr3800), oluşturulma tarihi 2026-09-15,
  lisans beyan edilmemiş.
  <https://github.com/abubakr3800/evo-reader>
  `.evo` dosyalarını DIALux olmadan okur. STEP proje kaydından armatür örneklerini
  çıkarır. Konumu her örneğin kendi `CoordSys3D` kaydından, ürün kimliğini
  başvuruları izleyerek bulur. Odaları da mümkün olduğu ölçüde çıkarır. `.rsl`
  sonuç dosyalarındaki ikili veriyi istatistiksel, sezgisel yöntemlerle tarayarak
  aday aydınlık değerleri çıkarır. Armatür başına haritalar noktasal kaynak
  modeliyle tahmin edilir

- **evo-builder** (GitHub kullanıcısı abubakr3800), oluşturulma tarihi 2026-09-21,
  lisans beyan edilmemiş.
  <https://github.com/abubakr3800/evo-builder>
  Python ile `.evo` dosyaları yazar ve biçimi belgeler: ZIP düzeni, STEP proje
  kaydı, `ProjectData.xml`, Boost.Serialization arşivi olarak `ScenegraphScene`
  ikili dosyası ve `.rsl` sonuç dosyaları. Dizi modülü, bir alan dizisindeki her
  armatür elemanının dizi orijinine göre ötelemesini taşıyan kendi `CoordSys3D`
  kaydına sahip olduğunu belgeler. Fotometri modülü `LightDistributionData`
  düzenini belgeler: C düzlemleri, gama açıları ve C düzlemine göre sıralanmış,
  cd/klm cinsinden ışık şiddeti değerleri

- **BHoM DIALux_Toolkit** (BHoM), 2019 ile 2024 arası.
  <https://github.com/BHoM/DIALux_Toolkit>
  DIALux ile veri alışverişini STF değişim biçimi üzerinden yapar, `.evo`
  dosyalarını kullanmaz

## Bağlam ve kaynaklar

STEP proje kaydını kaydedilmiş bir dosyayla karşılaştırmak için EvoParse
kullanılmıştır. Kamuya açık kaynak incelemesinde evo-reader ve evo-builder da
bulunmuştur. evo-builder, METHOD.md içindeki 3. ve 4. bölümlerde ele alınan dizi
çerçevelerini ve `LightDistributionData` düzenini belgeler. evo-reader, proje ve
sonuç kayıtlarını inceler. Açıklamaların örtüştüğü yerlerde bu çalışmalara atıf
verilir. İnceleme kapsamlı değildir ve öncelik kanıtı oluşturmaz.

## Kamuya açık çalışmalarla birlikte kapsam

2026-09-27 tarihinde yayımlanmış kod ve belgelerde görülebildiği kadarıyla:

1. **Kullanım yönü.** Bu notlar, projeyi Radiance sahnesi olarak yeniden kurmak ve
   aydınlatmayı yeniden hesaplamak için okur. evo-builder, DIALux için proje
   yazar. evo-reader, DIALux'un önceden hesapladığı sonuçları gösterir
2. **Okuma sırasında dünya konumu.** Birleştirme kuralı
   `world = apply(arrangementCS, elementCS.o)` ve ışık yönü
   `-rot(arrangementCS, elementCS.za)` olarak verilir. İkisi de kullandıkları
   çerçeveyle birlikte formül olarak yazılır
3. **Okuyucu tarafındaki bir tuzak.** Dizi çerçevesini çocuk ötelemesi olarak
   almak konumları ikiye katlar veya üst üste yığar. Bir projede 142 konumun
   113'ü yanlıştı. Notlar belirtiyi ve düzeltmeyi sağlayan kuralı verir
4. **Fotometrinin Radiance'a aktarılması.** Armatürden kandela tablosuna ve lamba
   akısına uzanan kayıt zinciri, LM-63 çıktısı ve `ies2rad` kaynaklarının `xform`
   ile yerleştirilmesi anlatılır. evo-builder ters yönde dönüşüm yapar, LDT/IES
   dosyalarını proje kaydına aktarır
5. **Oda kabuklarının Radiance'a aktarılması.** İki oda kaydı türü zemin, duvar ve
   tavan çokgenlerine dönüştürülür
6. **Sonuç dosyaları.** Bu notlar yalnızca yapı gözlemlerini içerir ve **aydınlık
   değerleri çıkarmaz**. evo-reader aday değerleri istatistiksel olarak çıkarır.
   evo-builder `.rsl` dosyalarını belgeler ve çözümleme betikleri içerir. Üç
   açıklama ayrıntılı olarak karşılaştırılmamıştır

## Kod

Atıf verilen projelerden kaynak kod bu depoya dahil edilmemiştir. Python
çeviricisi ayrı olarak ctlux-core içinde tutulur.
