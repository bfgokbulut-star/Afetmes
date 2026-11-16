# AfetMesh

İnternet ve baz istasyonu olmadan, yakındaki telefonlar arasında mesajlaşma ve SOS yayını yapan Android uygulaması (v0.2 prototip).

## Nasıl çalışır

- Her telefona rastgele 6 karakterlik bir **kod** verilir (ekranın üstünde görünür).
- Uygulama, Google Nearby Connections (Bluetooth + Wi-Fi) ile yakındaki AfetMesh cihazlarını bulur ve otomatik bağlanır.
- Mesajlar **flooding** ile yayılır: bir cihaz, daha önce görmediği mesajı bağlı herkese iletir. Böylece mesaj cihazdan cihaza **atlayarak** ilerler.
- Her mesajın **TTL**'i (12 atlama) vardır, mesaj id'si ile döngü engellenir.
- **Store-and-forward:** mesajlar cihazda saklanır, sonradan yanına gelen cihaza da verilir (yarın bir kurtarıcı yaklaşırsa mesajı alır).
- **SOS:** tek tuşla "Enkaz altındayım" mesajı, GPS konumu ve pil yüzdesi ile 30 sn'de bir yayınlanır. Alan cihazlarda "Alınan SOS sinyalleri" listesinde görünür, "Haritada aç" ile konuma gidilir.
- **Sesli not:** en fazla 10 sn, AMR-NB 4.75 kbps ile sıkıştırılır (~6 KB). Metin gibi atlayarak ilerler; alıcıda "Dinle" ile oynatılır. Gerçek zamanlı sesli arama değildir (kaydet-gönder).
- **Deprem algılama:** ağ çalışırken ivmeölçer izlenir. Telefon yerinde dururken 3 sn sürekli sarsıntı algılanırsa "Deprem algılandı" bildirimi çıkar. 60 sn içinde "İyiyim"e basılmazsa **SOS kendiliğinden başlar** (kullanıcı yanıt veremeyecek durumda olabilir). "Test" düğmesi ile akış denenebilir; ayarlardan kapatılabilir.
- **Pil tasarrufu:** pil %20 altında SOS yayını 60 sn'ye, %10 altında 120 sn'ye seyrekleşir.
- Servis ön planda (foreground service) çalıştığı için ekran kapalıyken de ağda kalır.

## Derleme ve çalıştırma

1. Android Studio'da `AfetMesh` klasörünü açın (Gradle senkronizasyonu için internet gerekir).
2. **En az 2 gerçek Android telefon** gerekir (emülatör Bluetooth/Wi-Fi mesh'i test edemez). Telefonlarda Google Play Hizmetleri olmalı.
3. Uygulamayı iki telefona da yükleyin, izinleri verin, "Ağı başlat"a basın.
4. Bir telefonun kodunu diğerine yazıp mesaj gönderin; ya da "SOS GÖNDER" deneyin.
5. Atlama testi: 3 telefonla A ve C'yi birbirinden uzağa (menzil dışına) koyup B'yi ortaya alın. A'dan C'ye mesaj B üzerinden geçmeli.

## Bilinen sınırlar (dürüst liste)

- **Sadece uygulama yüklü cihazlar röle olur.** Yüklü olmayan telefonlar ağa katılamaz.
- **Menzil kısa:** Bluetooth/Wi-Fi ile açık alanda onlarca metre, enkazda çok daha az. Ağ ancak yeterince yoğun kurulum olursa işe yarar.
- **Sadece Android** ve Google Play Hizmetleri gerekli. iOS ile konuşmaz.
- **Şifreleme yok:** mesajlar şifrelenmez, kimlik doğrulama yoktur (herkes bağlanabilir ve sahte mesaj atabilir).
- **Sesli arama yok**, sadece kaydedilmiş kısa sesli not var. Gerçek zamanlı ses çoklu atlamada güvenilir değil. Sesli notlar büyük olduğu için cihazda en fazla 10 tanesi saklanıp yeni bağlanana iletilir.
- **Deprem algılama sezgiseldir ve ayarlanmamıştır:** yanlış alarm (düşen telefon, titreşen masa) veya kaçırılan hafif sarsıntı olabilir. Gerçek deprem verisiyle sahada eşikleri ayarlamak gerekir. Resmi erken uyarı sistemi yerine geçmez.
- **Deprem algılama için uygulamanın çalışıyor olması gerekir:** telefon yeniden başlatılırsa uygulamayı açıp "Ağı başlat"a basmak gerekir (otomatik başlatma eklenmedi, Android ön plan servisi kısıtları yüzünden).
- **Deprem algılama pili tüketir:** ekran kapalıyken ivmeölçerin çalışması için CPU uyanık tutulur. Gece şarjdayken sorun olmaz; gündüz pil için kapatılabilir.
- **Mesafe/yön bulma yok:** Nearby sinyal gücü (RSSI) vermez. Kurtarıcıya yön göstermek için SOS içindeki GPS konumuna güvenilir.
- **Pil:** sürekli tarama pili tüketir. Düşük pilde seyrek yayın ve "sadece SOS" modu eklenmeli.
- **Arka plan kısıtları:** bazı üreticiler (Xiaomi, Huawei, Samsung) arka plan servislerini agresif kapatır; pil optimizasyonundan muaf tutmak gerekebilir.
- Kod derlenmiş ve gerçek cihazda test edilmiş değildir; ilk derlemede küçük düzeltmeler gerekebilir.

## Sonraki adımlar

1. Gerçek depremle eşik ayarı ve yanlış alarm testleri
2. Kurtarıcı modu (SOS'ları yakınlığa/aciliyete göre sıralama)
3. Mesaj imzalama/şifreleme
4. Bluetooth LE ile Nearby'a bağlı olmayan, iOS ile de uyumlu özel protokol

## Dosyalar

- `MeshService.kt`: mesh çekirdeği (keşif, bağlantı, flooding, SOS, konum)
- `Message.kt`: mesaj modeli ve serileştirme
- `MeshState.kt`: servis ile arayüz arasındaki ortak durum
- `MainActivity.kt`: arayüz (Jetpack Compose) ve izinler
- `QuakeDetector.kt`: ivmeölçer tabanlı deprem sezgisi
- `Voice.kt`: sesli not kaydı ve oynatma
