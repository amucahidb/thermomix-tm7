# Thermomix TM7 — Home Assistant

Thermomix® TM7'nin (ve TM6'nın) canlı pişirme durumunu Home Assistant'a getirir: tarif ya da mod adı, aşama, adım, kalan süre, anlık ve hedef sıcaklık, cihaz mesajları ve otomasyonlar için olaylar ("bitti", "adım değişti" gibi).

Bu entegrasyon yalnızca **izleme** yapar. Thermomix'i uzaktan başlatmak, durdurmak ya da ayarlamak mümkün değildir.

## Kurulum (Marketplace)

1. Home Assistant'ta **Ayarlar → Marketplace** sayfasını aç.
2. **Bağlantıdan ekle** (Add from link) düğmesine dokun ve şu adresi gir:
   `https://github.com/amucahidb/thermomix-tm7`
   Tür olarak **Integration** seç.
3. **Thermomix TM7**'yi kur.
4. **Ayarlar → Cihazlar ve Hizmetler → Entegrasyon ekle → Thermomix** yolunu izle ve Cookidoo hesabınla giriş yap.

Thermomix entegrasyonu daha önce kuruluysa yeniden eklemen gerekmez: aynı entegrasyonun yerine geçer, mevcut cihaz ve sensörler korunur.

## Gereksinimler

- Cookidoo hesabına bağlı, internete bağlı bir Thermomix TM7 (ya da TM6).
- Thermomix'te **Ayarlar → Uzaktan izleme** açık olmalı.
- Aktif Cookidoo aboneliği (Vorwerk uzaktan izlemeyi aboneliğe bağlıyor).
- Home Assistant 2026.10 ya da daha yeni.

## Sensörler

| Varlık | Açıklama |
|---|---|
| Pişirme durumu | Boşta, Çalışıyor, Duraklatıldı, Bitti, Onaylandı, Veri eskidi |
| Tarif | Tarifin ya da modun adı (ör. "Isıtma", "Buharda Pişirme") |
| Aşama | Süreli modlarda cihazın ne yaptığı (ör. "Isıtıyor", "Pişiriyor") |
| Adım | Rehberli tariflerde adım, ör. `5/9` |
| Kalan süre / Adım bitiş saati | Çalışan adımın kalan süresi ve bitiş zamanı |
| Anlık sıcaklık / Hedef sıcaklık | °C |
| Mesaj | Cihazın gönderdiği uyarılar |
| Pişirme (ikili) | Pişirme sürüyor mu |
| Bağlantı (ikili) | Bildirim bağlantısı açık mı |
| Pişirme olayı | Başladı, Adım değişti, Duraklatıldı, Devam etti, Bitti, Onaylandı, Mesaj, Veri eskidi, Temizlendi |

Her değişiklikte Home Assistant olay yoluna `thermomix_cooking_update` olayı da gönderilir.

Bilinen sınırlar:
- Vorwerk yalnızca **3 dakika ve üzeri** adımları, ayrıca başlatma, duraklatma, bitiş ve mesajları gönderir. Kısa adımlar görünmeyebilir.
- Manuel ısıtmada kalan süre ve adım gelmez; sadece sıcaklıklar gelir.
- Veriler Vorwerk bulutundan gelir. İnternet ya da Cookidoo kesilirse güncelleme durur.
- Kullanılan Cookidoo API'si resmi değildir ve Vorwerk tarafından değiştirilebilir.

## Nasıl çalışır

Thermomix'in pişirme durumu için yerel bir API'si yoktur. Uzaktan izleme açıkken cihaz durumunu Cookidoo bulutu üzerinden Cookidoo mobil uygulamasına Firebase bildirimi olarak gönderir. Bu entegrasyon aynı sunucuya kendi bildirim kaydını yaptırır ve uygulamanın bildirimlerinde kullanılan veriyi alır. Giriş ve kayıt için resmi olmayan [`cookidoo-api`](https://github.com/miaucl/cookidoo-api) kütüphanesini, bildirim bağlantısı için `firebase-messaging` kütüphanesini kullanır.

### TM7 düzeltmeleri

Bu depo, [surfingbytes/thermomix-ha](https://github.com/surfingbytes/thermomix-ha) projesine dayanır. TM7'de görülen şu hatalar düzeltilmiştir:

- **Bağlantı kopması:** Cookidoo bildirimdeki şifreleme değerlerini base64 dolgusu (`=`) olmadan gönderiyor. `firebase-messaging` bunu "Incorrect padding" hatasıyla reddedip bildirim istemcisini kapatıyordu.
- **Pişirme bilgisinin anında silinmesi:** TM7 tarihleri epoch *saniye* olarak gönderiyor. Milisaniye sanılınca tarih 1970'e düşüyor ve her çalışan pişirme hemen kapatılıyordu.
- **Anlık sıcaklığın boş kalması:** Sıcaklık "45°" gibi metin olarak geliyor; artık sayıya çevriliyor.
- **"Veri eskidi" durumunda takılma:** Biten ve onaylanan pişirme artık bir süre sonra "Boşta" durumuna dönüyor.
- **Yeni Aşama sensörü** ve **Türkçe** arayüz.

## Lisans

MIT. Orijinal telif hakkı notu [LICENSE](LICENSE) dosyasında korunmuştur.

Thermomix® ve Cookidoo® Vorwerk'in tescilli markalarıdır. Bu proje Vorwerk ile bağlantılı değildir.
