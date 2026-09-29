# Yöntem

İngilizce asıl metin: [METHOD.md](../METHOD.md). İki metin arasında fark olursa
İngilizce metin geçerlidir.

Gösterim: konumlar metre cinsinden `(x, y, z)` biçimindedir. Eksen vektörleri
birimsizdir. Bir koordinat çerçevesi `CS = (o, xa, ya, za)` diye yazılır: bir
orijin ve üç eksen vektörü. Hepsi ebeveyn çerçeveye göre verilir. Formüller sonlu,
birbirine dik birim eksenler varsayar.

Ebeveyn (İng. parent): bir kaydın bağlı olduğu kayıt. Çocuk (İng. child): bir
ebeveyne bağlı kayıt.

## 1. Dosya kapsayıcısı

- Bir `.evo` dosyası standart bir ZIP arşividir (`PK\x03\x04`). Girdilerin bir
  kısmı *sıkıştırmasız* (stored), bir kısmı *deflate* ile sıkıştırılmıştır. Bu
  yüzden girdileri ham dosyanın içinde konum arayarak değil, bir ZIP kütüphanesi
  üzerinden okuyun
- Proje kaydı `Project/ProjectData/ProjectData.dat` girdisidir. ISO 10303-21
  ("STEP fiziksel dosyası") sözdiziminde düz metindir. Başlığı bir DIALux şemasının
  adını verir. Bu şemanın kamuya açık bir EXPRESS tanımı
  bilinmiyor
- Her veri satırı bir kayıttır: `#N = Tür(alan, alan, ...);`
- Yanındaki `Project/ProjectData/ProjectData.xml` girdisi kayıt türlerini ve alan
  adlarını tanımlar. Bir alanın sırası belirsiz olduğunda işe yarar

## 2. Kayıt dilbilgisi

**Kayıtlar.** `#N = Tür( ... );` kalıbını eşleyin. Gövdede noktalı virgül ya da
parantez içeren tırnaklı metinler bulunabilir. Kayıt kimlikleri tektir. Aynı
kimliğin yinelenmesi hatadır.

**Alan bölme.** Gövdeyi, parantez derinliği 0 olan ve tırnak dışında kalan
virgüllerden bölün. Tırnaklı metnin içinde `''` kaçışlı bir kesme işaretidir ve
metni bitirmez. Dengesiz parantez veya kapanmamış tırnak kaydı geçersiz kılar.
Mevcut başvuru ayrıştırıcısı böyle kayıtları uyarıyla atlar ve dönüşümü
her durumda durdurmaz.

**Başvurular.** Başvuru, tırnaklı metinlerin dışındaki `#N` biçimidir.
Başvuruları toplamadan önce tırnaklı metinleri çıkarın, çünkü metnin içindeki
`#123` bir bağ değildir.

**Metin.** Dış tırnakları atın, `''` yerine `'` koyun ve ISO 10303-21'in standart
kaçışlarını çözün: `\X2\hhhh...\X0\` (UTF-16BE) ve `\X4\hhhhhhhh...\X0\`
(UTF-32BE). Standardın öbür kaçışları (`\X\hh`, `\S\`) bu yöntemde çözülmez.
Böyle metinler saklandığı gibi bırakılır. Daha katı bir okuyucu bunları hata
olarak bildirmelidir.

**Çerçeveler.** Bir `CoordSys3D` kaydının tam dört alanı vardır. Her biri üçlü bir
vektördür: `(o, xa, ya, za)`.

**İki işlem** her yerde kullanılır:

```
apply(CS, p) = o + p.x * xa + p.y * ya + p.z * za     (nokta: döndür ve ötele)
rot(CS, v)   =     v.x * xa + v.y * ya + v.z * za     (yön: yalnızca döndür)
```

**Çerçevelerin birleştirilmesi.** Ebeveyn bağları `RelAggregates` ve
`RelContainedInSpatialStructure` ilişki kayıtlarından gelir: baştaki başvuru
ebeveyn kayıttır, kalan başvurular onun çocuklarıdır. Bir çocuğun yerel çerçevesi
`L = (lo, lx, ly, lz)` ve ebeveyninin dünya çerçevesi `P` ise

```
world(çocuk) = ( apply(P, lo), rot(P, lx), rot(P, ly), rot(P, lz) )
```

kökten başlayarak özyinelemeli uygulanır. Ebeveyni olmayan bir nesne, yerel
çerçevesini dünya çerçevesi olarak kullanır. Ebeveyn zincirinde döngü olması
hatadır.

## 3. Armatür zinciri (konum ve yön)

**İlgili kayıtlar.** Her armatür bir `LuminaireElement` kaydıdır. Dördüncü alanı
kendi `CoordSys3D` kaydına başvurur. Dizi olarak yerleştirilen armatürler (çizgi,
alan, daire) `RelAggregates` üzerinden bir `LuminaireArrangement` kaydının
çocuklarıdır. Dizinin dünyada (ya da kendi ebeveyninde) ayrı bir `CoordSys3D` kaydı
vardır.

**Formül.**

Dizi üyesi için `diziCS` (`arrangementCS`), kendi ebeveynleri uygulandıktan sonra
dizinin dünya çerçevesidir. `elemanCS` (`elementCS`), üyenin saklandığı hâliyle
diziye göre çerçevesidir. Dizi çerçevesi tam bir kez uygulanır.

```
dizi üyesi:       dünya_konumu = apply(diziCS, elemanCS.o)
                  eksen_z      = rot(diziCS, elemanCS.za)
tekil armatür:    dünya_konumu = elemanCS.o          (ebeveyn çerçeveler uygulandıktan sonra)
                  eksen_z      = elemanCS.za
ışık yönü:        yön = -eksen_z / |eksen_z|
```

Armatürün yerel `+z` ekseni montaj normalidir (tavana doğru), bu yüzden ışık `-z`
boyunca çıkar: aşağı bakan bir armatürde `yön = (0, 0, -1)` olur. Eğik vurgu ve
duvar yıkama armatürleri kendi eğimlerini korur, çünkü her eleman kendi
çerçevesini taşır. Işık ekseni etrafındaki dönüşleri Radiance'a aktarılmaz
(bölüm 4), bu yüzden simetrik olmayan bir dağılım yanlış yöne dönmüş olabilir.
Bkz. bölüm 8.

Her üyenin kendi çerçevesi, dizinin içindeki kendine özgü ötelemeyi zaten taşır
(örneğin 6 x 10'luk bir tavan ızgarası 60 farklı orijin verir). Bu yüzden üyeleri
yerleştirmek için parametrik dizi kayıtlarına (çizgi, alan, daire konum verisi)
gerek yoktur.

Eleman çerçevesinin orijininin, ışık yayan yüzeyin merkeziyle yaklaşık 1 cm içinde
örtüştüğü gözlendi (tek projede).

**Bilinen tuzak ve düzeltmesi.** Bu yöntemin daha eski bir sürümü, dizinin
çerçevesini bir *çocuk ötelemesi* sanıp üye konumlarına ekliyordu. Belirtiler: bir
üyenin konumu ikiye katlanır (tavanın çok üstünde görünür), öbür üyeler dizinin
orijinine yığılır ve birçok armatür aynı noktayı paylaşır. Gerçek bir projede,
düzeltmeden önce 142 konumun 113'ü yanlıştı. Doğru kural yukarıdakidir: dizinin
çerçevesi ebeveyn çerçevedir ve her üyenin kendi orijinine bir kez uygulanır.
Düzeltmeden sonra o projedeki bütün konumlar birbirinden farklıydı ve oda
çizgilerinin içindeydi.

İkinci tuzak: bir armatürün alt ağacında başka `CoordSys3D` kayıtları da bulunur
(örneğin yüzeylerdeki malzeme bağlama noktaları). "Elemanın altında bulunan,
sıfırdan farklı herhangi bir çerçeve"yi almak bunlardan birini seçer ve yanlış
konum verir. Yalnızca elemanın dördüncü alanındaki çerçeveyi kullanın.

## 4. Fotometri zinciri

```
LuminaireElement
  --RelDefinesByPrototype-->            LuminairePrototype
  --PrototypeGeometricRepresentation--> LuminairePrototypeRepresentationData   ("donanım")
LightDistributionConnection(donanım, ..., LightDistribution)
  LightDistribution --> LightDistributionData   (C açıları, gama açıları, kandela değerleri)
LampTypeChannel(donanım, ..., ..., lümen)       (ışık akısı, dördüncü alan)
```

- Doğrulanmış fotometri, elemandan prototipe ve prototipten donanım kaydına
  tekil bağlantılar gerektirir. Bilinen bağlantıların çoklu veya çelişkili olması
  hatadır. Eksik bağlantılar armatürün yerleştirilmesine yine de izin verebilir,
  ancak fotometri doğrulanmış olmaz. Çevirici eşleşmeyen fotometriyi ve yaklaşık
  gücü bildirir
- `LightDistributionData`, 2., 3. ve 4. alanlar: C düzlemi açıları (artan, 0'dan
  başlar), gama açıları (artan, 0-180) ve C düzlemine göre sıralı `nC * nG` kandela
  değeri. İncelenen projelerde son C düzlemi 360'ın altındaydı (örneğin 345 ya da
  357.5). Dönel simetrik ürünlerde tek düzlem vardı
- Yöntem, tabloyu 1000 lümen başına kandela olarak kabul eder ve lamba akısıyla
  ölçekler: `cd_çıkış = cd * lümen / 1000`. evo-builder'ın belgeleri aynı birimi
  (cd/klm) yazıyor
- **Akı denetimi.** Buradaki I, saklanan cd/klm değeri değil, kandela
  cinsinden ölçeklenmiş `cd_çıkış` değeridir. Bir tablonun akısı, açılar
  radyan olmak üzere, I(C, gamma)
  sin(gamma) ifadesinin gamma için 0 ile pi, C için 0 ile 2 pi arasındaki
  integralidir. Son C düzleminden 0'a dönüş aralığı da katılır. Tek düzlem 2 pi ile
  çarpılır. Lamba akısına bölündüğünde, ilk projede 9 ürünün 7'sinde oran 1.00
  çıktı (öbür ikisi açıklanmamış). İkinci bir
  projede, 2026-09-27 tarihli ölçümde, 11 ürünün 6'sı 1.00'ın yüzde 1 yakınındaydı,
  11 ürünün 5'i 0.80 ile 0.81 arasındaydı. 1'in altındaki oran, yüzde 100'ün
  altında bir ışık çıkış oranının vereceği sonuçtur, ama bu doğrulanmadı. Bu
  ölçümlerin tabloları ve sayısal adımları pakette yoktur. Oranlar okuyucunun
  yeniden hesaplayıp doğrulayabileceği veriler değil, bildirilen gözlemlerdir. cd/klm yorumuyla
  uyumludurlar. Bu, bir DIALux hesabıyla karşılaştırma **değildir**
- Lümenin bulunmaması hatadır. Varsayılan bir akı kabul edilmez

**IES LM-63-2002 çıktısı** (donanım kaydı başına bir dosya):

```
IESNA:LM-63-2002
[TEST] not available
[TESTLAB] not available
[ISSUEDATE] not available
[MANUFAC] not read from the project record
[_NOTICE] Photometric data belongs to its owner, normally the luminaire
[MORE] manufacturer. Written from a lighting project file to rebuild
[MORE] that project.
TILT=NONE
1 <lümen> 1 <nG> <nC> 1 2 <w> <l> <h>     lamba sayısı, lamba başına lm, çarpan, sayılar, tür C, metre
1.0 1.0 0.0
<gama açıları>
<C açıları>
<her C düzlemi için nG ölçeklenmiş kandela değerinden oluşan bir satır>
```

Işık çıkış açıklığının boyutu `<w> <l> <h>` projeden okunmaz. Yöntem sabit bir yer
tutucu kullanır (0.3 x 0.3 x 0.1 m).

`LightDistributionData` kaydının ilk alanı (simetri işareti) kullanılmaz ve tablolar
genişletilmez: hangi düzlemler saklanmışsa onlar yazılır. Saklanan tek düzlem, LM-63
okuyucularınca dönel simetrik dağılım olarak okunur. 90 ya da 180'de biten tablolar
incelenen projelerde görülmedi.

C düzlemleri saklandığı gibi yazılır. LM-63 tür C dosyaları normalde 0, 90, 180 ya
da 360'ta bittiği hâlde, 360'taki kapanış düzlemi eklenmez. `ies2rad` bu biçimde
yazılan dosyaları kabul etti. Daha katı okuyucular etmeyebilir.

**Radiance'a aktarım.** `ies2rad` her IES dosyasını bir Radiance ışık kaynağına
çevirir. Her kopya `xform` ile yerleştirilir. `ies2rad` kaynağı nadiri -Z, C0
düzlemi +X olacak biçimde yazar. Dönüş nadiri ışık yönüne, C0 düzlemini dünya +X
ekseninin ışık yönüne dik izdüşümüne çevirir. Işık X boyunca yataysa dünya +Y
kullanılır. Dönüş üç açı olarak yazılır:

```
!xform -rx a -ry b -rz c -t x y z  <ies2rad çıktısı>
```

Aşağı bakan armatürün C0 düzlemi bu yüzden dünya +X ekseninde kalır. Önceki bir
çevirici sürümü `-ry beta -rz (phi - 180)` kullanıyordu ve aşağı bakan her
armatürün C0 düzlemini dünya -X eksenine çeviriyordu. Bu, Radiance'ta simetrik
olmayan bir deneme dağılımıyla ölçüldü. EVO çerçevesindeki ışık ekseni etrafındaki
dönüş aktarılmaz.

## 5. Oda zinciri

**Tür A: çokgen tabanlı mekân.**

```
Space (dördüncü alan: CoordSys3D)
  --> ... --> PolygonBasedSpaceRepresentationDataPart
                (sahip, n, çerçeve, yükseklik?, malzeme, malzeme, .Bottom., (PolyPoint2D başvuruları), .Storey.)
PolyPoint2D --> (x, y)   mekân çerçevesinde taban çizgisi
```

Başvuru çeviricisi bu tür için henüz doğrulanmış bir yükseklik alanı
belirlemiyor. Üst düzey alanları soldan sağa tarar ve 1.5 değerinden büyük,
30 değerinden küçük ilk skaler sayıyı metre cinsinden yükseklik olarak kullanır.
Böyle bir sayı bulunamazsa 3 m varsayar. Bu nedenle daha önce gelen sayısal bir
alan, örneğin değeri bu aralıktaysa `n`, yanlışlıkla yükseklik olarak alınabilir. Bu türdeki her oda için, çıkarılan
veya varsayılan yüksekliği belirten bir uyarı üretilir. Sonuçları kullanmadan
önce bu varsayımı kaynak projeyle karşılaştırın.

**Tür B: kat konturu.** `RelAssociatesStoreyContourBasedSpace` bir `Space` kaydını
tek bir `StoreyContour` kaydına bağlar. Bunun `StoreyContourRepresentationData`
kaydı iki boyutlu çizgiyi taşır. Yükseklik parça kaydından, parça `.Storey.` diye
işaretliyse ebeveyn `Storey` kaydından (onun `StoreyRepresentationData` kaydı)
gelir. Eksik ya da birim çerçeveden farklı bir alt çerçeve reddedilir, çünkü anlamı
belirlenememiştir.

**Formül** (iki tür için de), `S` mekânın ya da konturun dünya çerçevesi,
`(x_i, y_i)` kapanış noktası çıkarılmış taban çizgisi:

```
zemin_i   = apply(S, (x_i, y_i, 0))
tavan_i   = apply(S, (x_i, y_i, h))
zemin     = çokgen(zemin_0 ... zemin_n-1)
tavan     = çokgen(tavan_n-1 ... tavan_0)               (ters sıra)
duvar_i   = çokgen(zemin_i+1, zemin_i, tavan_i, tavan_i+1)
```

**Yüzlerin yönü.** Bunun için ölçülen projede (dokuz oda) saklanan taban çizgisi
yukarıdan bakıldığında saat yönünün tersineydi: yukarıdaki sırayla her zemin
yukarı, her tavan aşağı, yani odanın içine bakıyordu. Burada verilen duvar sırası
da böyle bir çizgi için odanın içine bakar. Taban çizgisi saat yönünde
saklanmışsa üçü de dışarı bakar. Çizgi alanının işaretine bakın ve önce çizgiyi
ters çevirin.

Çeviricinin önceki sürümü duvarları öbür sırayla yazıyordu:
`(zemin_i, zemin_i+1, tavan_i+1, tavan_i)`. Bu yüzden duvarları, zemin ve
tavanlarıyla aynı yöne bakmıyordu. Bu notlarla birlikte yayınlanan başvuru kodu
yukarıdaki sırayı kullanır ve saat yönündeki çizgiyi önce ters çevirir. Radiance `plastic` yüzeyleri iki taraftan da
çizildiği için görüntüler etkilenmedi. Ağ dışa aktarımı, arka yüz ayıklama ve
yüzey normalini kullanan her hesap etkilenir.

Aynı dosyadaki `PolyPoint3D` kayıtları oda duvarı *değildir*: asma tavanlara ve
prizmatik hacimlere aittir.

**Varsayılan yansıtmalar.** Yüzey renkleri projeden okunmaz. Fotopik yansıtması
`rho_v = 0.265 R + 0.670 G + 0.065 B` olan Radiance `plastic` malzemeleri
kullanılır:

| Yüzey | R G B | rho_v |
| --- | --- | --- |
| zemin | 0.33 0.22 0.13 | yaklaşık 0.24 |
| duvar | 0.52 0.49 0.44 | yaklaşık 0.50 |
| tavan | 0.75 0.75 0.73 | yaklaşık 0.75 |

## 6. Sonuç dosyaları (`.rsl`): yapı gözlemleri, çözülmedi

Hesap sonuçları `Project/Results/<grup>/` altında `.rsl` dosyaları olarak saklanır.
**Bu dosyalar çözülmemiştir. Bu yöntem hiçbir aydınlık değeri çıkarmaz.**
Gözlenenler:

- **Başlık.** Her `.rsl` dosyası, Boost.Serialization yerel ikili arşiv başlığıyla
  başlar: 8 baytlık little-endian uzunluk `22`, `serialization::archive` metni,
  ardından 2 baytlık kütüphane sürümü `20` (`0x14`) ve tür boyutu bayrakları. Yerel
  arşivin düzeni, Boost'un kamuya açık kaynak kodundan alınmıştır
- **Sonuç grubu başına dosyalar.** `Datameshcolorillums0.rsl` (yalancı renkli
  yüzey ağı), `Dataillumqt0.rsl` (nicel aydınlık değerleri), `...Toc.rsl`
  dosyaları (içindekiler tabloları), `Datavislightemsurf0.rsl` (armatürlerin
  görünen ışıklı yüzeyleri), `Dataenvillum0.rsl` (çevre aydınlığı), küçük kimlik
  ve metin tabloları
- **Renkli ağ ile nicel değerler.** Renkli ağ bir görüntüleme katmanıdır (geometri
  ve renk). Nicel değerler ayrı bir dosyadadır
- **Nesne başına çerçeveleme.** Başlıktan sonra ağ dosyası, metre cinsinden
  hizalanmamış float32 koordinatlar taşır, ama köşe dizileri nesne başına eklenen
  baytlarla (sürüm, izleme ve sınıf kimliği baytları) kesilir. Düz bir dizi
  değildir
- **Aynı boyut, farklı içerik.** Farklı sonuç gruplarının ağ dosyaları tam olarak
  aynı boyutta ama farklı özet değerinde olabilir. Bu, aynı geometrinin farklı
  renkler taşımasıyla uyumludur
- **Aday çapa.** Yerel ikili arşivde bir sayı dizisi, 8 baytlık bir sayaç ve
  ardından ham değerler olarak yazılabilir. Bu, şemasız bir tarama için çapa olarak
  düşünüldü. İkinci bir projede bu imza aday değer dizilerinin başında bulunamadı,
  bu yüzden doğrulanmamış durumdadır

O ikinci projede başlık 408 sonuç dosyasının hepsinde vardı, ancak 24 baytlık köşe kaydı adımı doğrulanmadı.

## 7. Sahnenin Radiance'ta kurulması

| Parça | Nasıl üretilir | Radiance araçları |
| --- | --- | --- |
| Oda kabukları | `plastic` malzemeli `polygon` yüzeyler | (sahne metni) |
| Armatürler | `ies2rad` kaynakları, `xform` ile yerleştirilip yönlendirilir | `ies2rad`, `xform` |
| Sahne | octree | `oconv` |
| Görüntü ve aydınlık | render. Işınım (irradiance) modunda, sonra lüks = 179 x (0.265 R + 0.670 G + 0.065 B), RGB ışınım üzerinden | `rpict`, `rtrace`, `falsecolor`, `pcond` |

İç mekânlarda birkaç ortam yansıması kullanın (örneğin `-ab 5`). Tek yansıma,
yüzeyler arası yansımayı olduğundan az hesaplar.

## 8. Sınırlar

- Yalnızca DIALux evo 5.13 ile gözlendi. Ana çıkarım tek bir gerçek projeye
  dayanır. Seçili fotometri denetimleri ve `.rsl` başlık gözlemleri ikinci bir
  projede yinelendi
- Sonuçlar DIALux hesap sonuçlarıyla **karşılaştırılmamıştır**
- Çokgen tabanlı odalarda yükseklik, 1.5 değerinden büyük ve 30 değerinden küçük
  ilk skaler sayıdan çıkarılır veya varsayılan 3 m kullanılır. Alanın bu yorumu
  doğrulanmamıştır ve uyarıyla bildirilir. Hesaplamadan önce oluşan oda
  geometrisini denetleyin
- Işık rengi ve renk sıcaklığı doğrulanmamıştır. Nötr beyaz kullanılır
- Kandela ölçeklemesi (1000 lümen başına) bir yorumdur, doğrulanmış bir alan
  tanımı değildir
- IES çıktısındaki ışık çıkış açıklığı boyutu bir yer tutucudur. Armatürden
  uzaktaki aydınlık bundan pek etkilenmez. Bir bakış yönündeki parlaklık, şiddetin
  ışık yayan alanın o yöndeki izdüşümüne bölümüdür. Yer tutucu alanla yanlış olur ve
  bu kaynaklarla hesaplanan kamaşma ölçüleri (UGR, DGP) güvenilir değildir
- Işık ekseni etrafındaki dönüş Radiance'a aktarılmaz. C0 düzlemi 4. bölümdeki
  sabit kurala göre yerleştirilir. Simetrik olmayan armatürlerde (duvar yıkayıcılar, doğrusal armatürler) dağılım bu yüzden yanlış
  yöne dönmüş olabilir. Eleman çerçevesinin x ekseni C0 yönü için bir adaydır. Bu
  uygulanmadı ve doğrulanmadı
- IES dosyaları 360'taki kapanış C düzlemi olmadan yazılır
- Çeviricinin önceki sürümünün yazdığı duvar yüzleri, zemin ve tavanlarla tutarlı
  yönde değildi (bölüm 5). Yayınlanan başvuru kodu bunu düzeltir
- Kayıt taraması yalnızca tam kayıtları eşler. Çeviricinin önceki sürümünde metnin
  sonunda kesilmiş bir kayıt hata verilmeden atlanıyordu ve varsayılan 4000
  armatür sınırı bildirilmeden uygulanıyordu. Bu notlarla birlikte yayınlanan
  başvuru kodu ikisini de bildirir
- Oda yüzeyi yansıtmaları varsayılandır, proje değerleri değildir
- `.rsl` sonuç dosyaları ve ikili sahne grafiği (`Project/ScenegraphScene`)
  çözülmemiştir
- Alan sıraları gözlenen sürümde belirlenmiştir. Başka sürümler farklı olabilir.
  Beklenmeyen bir kayıt, tahmin yürütmek yerine dönüşümü durdurmalıdır

## Bağımsızlık ve markalar

Bağımsız bir çalışmadır. DIAL GmbH ile bağlantılı değildir, onun tarafından
onaylanmamıştır ve desteklenmemektedir. DIALux, DIAL GmbH şirketinin
markasıdır. Ürün adları yalnızca söz edilen biçimleri ve programları tanımlar.
