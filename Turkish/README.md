# EVO aydınlatma yerleşimini Radiance için çıkarma: yöntem notları

Bu notlar, DIALux evo `.evo` proje dosyasındaki aydınlatma düzeninin
(armatür konumları, ışık yönleri, fotometri ve oda kabuğu) nasıl çıkarılıp bir
Radiance sahnesi olarak yeniden kurulacağını anlatır. Çalışma, birlikte
çalışabilirlik amaçlı dosya biçimi çözümlemesidir: yalnızca kayıtlı proje
dosyaları incelenmiştir, programın kendisi incelenmemiştir.
Notlar neyin okunabildiğini, neyin okunamadığını ve bu çalışmanın kamuya açık diğer
çalışmalarla ilişkisini açıkça yazar.

Bu bağımsız çalışmanın DIAL GmbH ile bağlantısı yoktur. Şirket tarafından
onaylanmaz veya desteklenmez. DIALux, DIAL GmbH şirketinin markasıdır. Adı
burada kaynak dosyaları kaydeden uygulamayı belirtir.

İngilizce asıl metin: [README.md](../README.md). İki metin arasında fark olursa
İngilizce metin geçerlidir.

## Kapsam

- Girdi: bir DIALux evo `.evo` proje dosyası (evo 5.13 ile gözlendi)
- Çıktı: armatürlerin dünya konumu ve ışık yönü, IES LM-63 dosyaları olarak
  fotometri (sınırları METHOD.md, bölüm 8'de) ve oda kabuğu (zemin, duvarlar,
  tavan), hepsi Radiance'a eşlenir
- Kapsam dışı: hesap sonuçları (`.rsl`), ikili sahne grafiği, mobilya, dokular,
  gün ışığı. Çevirici mobilyayı da okur. Bu notlar bilerek aydınlatma düzeni
  ve oda kabuğuyla sınırlı tutulmuştur

## Özet

| Konu | Durum |
| --- | --- |
| Kap (ZIP) ve proje kaydı (ISO 10303-21 metni) | Anlatılıyor. Daha önceki kamuya açık çalışmalar da anlatıyor (bkz. RELATED_WORK.md) |
| Kayıt dilbilgisi: alan bölme, başvurular, metin kaçışları | Anlatılıyor. Çözülen kaçışlar: `''`, `\X2\`, `\X4\` |
| Armatürün dünya konumu ve ışık yönü | Anlatılıyor, dizilerle ilgili bilinen bir tuzak dahil. Dizi yapısını evo-builder da belgeliyor |
| Armatür fotometrisi (kandela tablosu + lümen -> IES -> Radiance) | Veri yolu olarak anlatılıyor. Sınırları var (METHOD.md, bölüm 8): cd/klm ölçeği bir yorumdur, açıklık boyutu yer tutucudur, ışık ekseni etrafındaki dönüş ve kapanış C düzlemi yazılmaz. Kandela kaydının düzenini evo-builder da belgeliyor. DIALux sonuçlarıyla karşılaştırılmadı |
| Oda kabuğu (taban çizgisi + yükseklik -> zemin, duvarlar, tavan) | İki kayıt türü için anlatılıyor |
| Sonuç dosyaları (`.rsl`) | Yalnızca yapı gözlemleri, **çözülmedi, aydınlık değeri çıkarılmıyor** |
| İkili sahne grafiği (`ScenegraphScene`) | Çözülmedi |
| Işık rengi / renk sıcaklığı | Doğrulanmadı. Nötr beyaz kullanılıyor |

## Belgeler

- [METHOD.md](METHOD.md): yöntem ve formüller
- [RELATED_WORK.md](RELATED_WORK.md): `.evo` biçimi üzerine kamuya açık diğer
  çalışmalar ve bu notların farkı
- [PROVENANCE.md](PROVENANCE.md): yöntem kapsamı ve kanıt sınırları
- [LICENSE](../LICENSE): MIT OR Apache-2.0
- [CITATION.md](CITATION.md): bu notlara atıf bilgisi

## Çevirici

Yöntem notlarıdır. Çalışan Python çeviricisi
[ctlux-core](https://github.com/JamesAugustus/ctlux-core) deposundadır
(MIT OR Apache-2.0). Çekirdek deponun kökünden EVO komutu:
`python3 -B -m core input.evo output`. Bu depoda örnek proje dosyası,
fotometri verisi veya DIALux uygulama dosyası bulunmaz.

## Geliştirme notu

Çözümleme ve bulgular yazara aittir. Yapay zekâ araçları cümle kurmada ve metni
düzenli tutmada yardımcı oldu. Ana çıkarım tek bir gerçek projeye dayanır. Seçili fotometri denetimleri ve
`.rsl` başlık gözlemleri ikinci bir projede yinelenmiştir. Kaynak proje
dosyaları dağıtılmaz (bkz. METHOD.md, bölüm 8).

## Lisans kapsamı

Copyright (C) 2026 James Augustus

Bu depodaki özgün yöntem notları, belgeler ve varsa örnek kod dahil tüm özgün içerik **MIT OR Apache-2.0** seçeneğiyle sunulur. [MIT](../LICENSE-MIT) veya [Apache 2.0](../LICENSE-APACHE) lisanslarından birini seçebilirsiniz. Seçilen lisansın koşulları uygulanır. İkisine birden uymak gerekmez

Her iki seçenek ticari kullanıma ve kapalı ürünlerde dağıtıma izin verir. Kaynak kodunun veya özel değişikliklerin yayımlanması zorunlu değildir

- MIT seçeneğinde telif ve izin bildirimi yazılımın tüm kopyalarında veya önemli bölümlerinde korunur
- Apache 2.0 seçeneğinde alıcılara lisans verilir, değiştirilen dosyalar belirgin bildirimlerle işaretlenir ve ilgili kaynak bildirimleri 4. bölüm uyarınca korunur
- Apache 2.0 seçeneğinde ilgili [NOTICE](../NOTICE) atfı dağıtılan NOTICE dosyası veya belgeler gibi 4(d) bölümünün izin verdiği bir yerde sunulur
- MIT seçildiğinde Apache NOTICE koşulları uygulanmaz

Her iki seçenek ilgili telif bildirimlerini korur. Reklam atfı, özel bir kullanıcı arayüzü atfı veya akademik atıf zorunluluğu getirmez

Üçüncü taraf alıntıları, kodları ve markaları kendi koşullarını korur ve bu bildirimle yeniden lisanslanmaz. Yalnız özgün içerikte sahip olunan haklar verilir. Fikirler, yöntemler ve dosya biçimi olguları üzerinde bu lisanslarla münhasır telif hakkı kurulmaz. Yalnız bu fikir veya olgularla hazırlanan bağımsız uygulamalarda bilimsel kaynak gösterme gönüllüdür ve [CITATION.md](CITATION.md) içinde rica edilir
