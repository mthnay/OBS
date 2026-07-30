# 📤 Site İçi Yükleme Kurulumu (Google Drive)

Site artık butona tıklayınca **kendi içinde** bir yükleme ekranı açıyor (sürükle-bırak,
önizleme, her dosya için ilerleme çubuğu + genel ilerleme). Dosyaların gerçekten Drive
klasörünüze gitmesi için **tek seferlik**, ücretsiz bir "köprü" kurmanız gerekiyor:
**Google Apps Script Web App**. Sunucu kiralamaya gerek yok, misafirler Google hesabı
olmadan yükleyebilir.

> Kurmadan da site çalışır ama **DEMO modunda** olur: arayüz ve çubuklar görünür,
> dosya gerçekten gönderilmez.

---

## Adım adım kurulum (~5 dakika)

1. **https://script.google.com** adresine gidin, Drive klasörünüzün sahibi olan
   Google hesabıyla giriş yapın.
2. Sağ üstten **Yeni proje** (New project) oluşturun.
3. Açılan `Code.gs` dosyasının içindeki her şeyi silin ve aşağıdaki kodu yapıştırın.
   `FOLDER_ID` zaten sizin klasörünüzle dolu geliyor.
4. Üstten **Dağıt → Yeni dağıtım** (Deploy → New deployment) seçin.
   - Tür (Select type): **Web uygulaması** (Web app)
   - **Çalıştıran** (Execute as): **Ben** (Me)
   - **Erişimi olan** (Who has access): **Herkes** (Anyone)
   - **Dağıt**'a basın, çıkan izinleri onaylayın (kendi hesabınız için güvenli).
5. Size bir **Web app URL** verecek (`.../exec` ile biter). Kopyalayın.
6. `index.html` dosyasını açın, en üstteki şu satıra yapıştırın:

   ```js
   const UPLOAD_ENDPOINT = "BURAYA_WEB_APP_URL";
   ```

Bitti! Artık misafirler siteden yükleyince dosyalar doğrudan Drive klasörünüze düşer.

---

## Apps Script kodu (`Code.gs` içine yapıştırın)

```javascript
// Yüklenecek Drive klasörünün kimliği (sizin klasörünüz hazır girildi):
const FOLDER_ID = "1kCjMmPLFbyn3JyfOmSW48gNjDKRRC_qX";

function doPost(e) {
  try {
    var data   = JSON.parse(e.postData.contents);
    var bytes  = Utilities.base64Decode(data.data);
    var blob   = Utilities.newBlob(bytes, data.mimeType, data.name);
    var folder = DriveApp.getFolderById(FOLDER_ID);
    var file   = folder.createFile(blob);
    return json({ ok: true, id: file.getId(), name: file.getName() });
  } catch (err) {
    return json({ ok: false, error: String(err) });
  }
}

// Tarayıcıdan test için (isteğe bağlı)
function doGet() {
  return json({ ok: true, message: "Yukleme ucu calisiyor." });
}

function json(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## Notlar & sınırlar

- **Dosya boyutu:** Apps Script'in istek sınırı nedeniyle çok büyük videolar
  (yaklaşık 50 MB üzeri) başarısız olabilir. Site, `MAX_FILE_MB` (varsayılan 100)
  üzerindeki dosyalar için misafiri önceden uyarır. Değeri `index.html` içinden
  değiştirebilirsiniz. Uzun düğün videoları için misafirlerden telefondan
  "yüksek yerine orta kalite" paylaşmalarını istemek işe yarar.
- **Kod değiştirirseniz:** Apps Script'te yeniden **Dağıt → Dağıtımı yönet →
  düzenle → Yeni sürüm** yapmanız gerekir, yoksa eski sürüm çalışır.
- **Güvenlik:** "Herkes" erişimi yalnızca *yükleme* içindir; klasörün içeriğini
  kimse bu uç noktadan göremez/indiremez. Silme/okuma yapmaz, sadece dosya ekler.
- **Alternatif (kod istemeyenler için):** Drive'da klasörü açıp sağ tık →
  Paylaş → "Bağlantısı olan herkes: Düzenleyen" yaparsanız, eski buton davranışıyla
  (yeni sekmede Drive) yükleme de mümkündür; ancak o yöntemde misafirin Google
  hesabıyla giriş yapması gerekir ve site içi ilerleme çubuğu olmaz.
