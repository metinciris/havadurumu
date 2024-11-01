# Excel'de Dinamik Hava Durumu ve Karşılama Mesajı Oluşturma

![Ekran Görüntüsü](https://raw.githubusercontent.com/metinciris/havadurumu/refs/heads/main/screen.png)

## Proje Özeti

Bu proje, Google E-Tablolar kullanarak hava durumu verilerini OpenWeatherMap API'sinden çekip, kullanıcıya güncel hava durumu, sıcaklık, rüzgar hızı ve saat bilgilerine göre dinamik ve doğal bir karşılama mesajı oluşturmayı amaçlar. Ayrıca, bu mesajı web üzerinde yayınlayarak geniş bir kitleye ulaştırmayı hedefler.

## Özellikler

- **Gerçek Zamanlı Hava Durumu Verileri**: OpenWeatherMap API kullanılarak güncel hava durumu bilgileri çekilir.
- **Dinamik Karşılama Mesajları**: Saat, hava durumu, sıcaklık ve rüzgar hızına göre değişen, doğal ve akıcı mesajlar oluşturulur.
- **Emojiler ve Simgeler**: Mesajlara uygun emojiler eklenerek daha çekici hale getirilir.
- **Otomatik Güncelleme**: Google Apps Script kullanılarak veriler her 30 dakikada bir otomatik olarak yenilenir.
- **Web'de Yayınlama**: E-Tablolar'daki veriler ve mesajlar web üzerinde yayınlanabilir.

## Örnek Uygulama

Projenin canlı bir örneğini [buradan](https://metinciris.com.tr/pages/yemek.php) inceleyebilirsiniz.

## Kurulum ve Kullanım

### 1. OpenWeatherMap API Anahtarı Alın

- OpenWeatherMap sitesine kaydolun ve bir API anahtarı edinin: [OpenWeatherMap API](https://openweathermap.org/api)

### 2. Google E-Tablolar'da Yeni Bir Sayfa Oluşturun

- Google Drive'da yeni bir Google E-Tablosu oluşturun.

### 3. IMPORTJSON Fonksiyonunu Ekleyin

- **Uzantılar** > **Apps Script** menüsüne gidin.
- Aşağıdaki kodu yapıştırın ve dosyayı kaydedin:

```javascript
function IMPORTJSON(url) {
  try {
    var response = UrlFetchApp.fetch(url);
    var content = response.getContentText();
    var json = JSON.parse(content);
    return parseJSONObject_(json);
  } catch (e) {
    return [["Hata oluştu: " + e.toString()]];
  }
}

function parseJSONObject_(obj) {
  var result = [];
  for (var key in obj) {
    var value = obj[key];
    if (typeof value === "object" && value !== null) {
      var subResult = parseJSONObject_(value);
      for (var i = 0; i < subResult.length; i++) {
        result.push([key + "." + subResult[i][0], subResult[i][1]]);
      }
    } else {
      result.push([key, value]);
    }
  }
  return result;
}
```

### 4. Hava Durumu Verilerini Çekin

- **"havam"** adında yeni bir sayfa ekleyin.
- A1 hücresine aşağıdaki formülü girin (API anahtarınızı eklemeyi unutmayın):

```excel
=IMPORTJSON("https://api.openweathermap.org/data/2.5/weather?q=isparta,TR&lang=en&appid=YOUR_API_KEY")
```

### 5. Verileri İşleyin

- `A1:B28` aralığında veriler görüntülenecektir.
- Gerekli verileri başka hücrelere çekmek için `VLOOKUP` fonksiyonunu kullanın. Örneğin:

  - **Hava Durumu Kodu (E1):**

    ```excel
    =VLOOKUP("weather.0.id"; $A$1:$B$28; 2; FALSE)
    ```

  - **Sıcaklık (E3 - Celsius):**

    ```excel
    =VLOOKUP("main.temp"; $A$1:$B$28; 2; FALSE) - 273,15
    ```

### 6. Gece/Gündüz Durumunu Belirleyin

- Gün doğumu ve gün batımı saatlerine göre gece veya gündüz olduğunu belirleyin.
- **E10** hücresine:

  ```excel
  =IF(
    AND(
      (E9 / 86400 + DATE(1970;1;1) + TIME(3;0;0)) >= (E7 / 86400 + DATE(1970;1;1) + TIME(3;0;0));
      (E9 / 86400 + DATE(1970;1;1) + TIME(3;0;0)) < (E8 / 86400 + DATE(1970;1;1) + TIME(3;0;0))
    );
    "gündüz";
    "gece"
  )
  ```

### 7. Mesaj Formüllerini Oluşturun

- **Selamlama Mesajı (E11):**

  ```excel
  =IFS(
    SAAT(ŞİMDİ())<6; "🌌 Gece henüz bitmedi, biraz daha dinlenebilirsin. ";
    VE(SAAT(ŞİMDİ())>=6; SAAT(ŞİMDİ())<12); "☀️ Günaydın! Yeni bir gün seni bekliyor. ";
    VE(SAAT(ŞİMDİ())>=12; SAAT(ŞİMDİ())<18); "😎 İyi günler! Umarım günün güzel geçiyordur. ";
    VE(SAAT(ŞİMDİ())>=18; SAAT(ŞİMDİ())<22); "🌇 İyi akşamlar! Gün batımının tadını çıkar. ";
    SAAT(ŞİMDİ())>=22; "🌙 Gece yarısı yaklaşıyor, dinlenme vakti. "
  )
  ```

- **Hava Durumu Mesajı (E12):**

  ```excel
  =IFS(
    VE(E1=800; E10="gündüz"); "☀️ Gökyüzü tertemiz, güneş parlıyor! ";
    VE(E1=800; E10="gece"); "🌕 Gökyüzü açık, ay ışığı parlıyor. ";
    VE(E1>=801; E1<=803); "⛅ Bulutlar gökyüzünü süslüyor. ";
    E1=804; "☁️ Gökyüzü bulutlarla kaplı. ";
    VE(E1>=500; E1<600); "🌧️ Yağmur yağıyor, şemsiyeni unutma! ";
    VE(E1>=200; E1<300); "⛈️ Fırtına yaklaşıyor, dikkatli ol. ";
    VE(E1>=600; E1<700); "❄️ Kar yağıyor, her yer beyaza bürünmüş. ";
    E1=701; "🌫️ Sisli bir hava, görüş mesafesi düşük. ";
    DOĞRU; "🌈 Hava bugün ilginç görünüyor. "
  )
  ```

- **Sıcaklık Mesajı (E13):**

  ```excel
  =IFS(
    E3<-10; "🥶 Buz gibi bir hava, evde kalmak iyi olabilir. ";
    VE(E3>=-10; E3<0); "🧥 Çok soğuk bir gün, kalın giyinmeyi unutma. ";
    VE(E3>=0; E3<10); "🧣 Serin bir hava var, bir ceket iyi olur. ";
    VE(E3>=10; E3<20); "🌼 Ilıman bir hava, dışarısı rahat. ";
    VE(E3>=20; E3<30); "🌞 Sıcak bir gün, hafif giysiler tercih et. ";
    VE(E3>=30; E3<40); "🥵 Oldukça sıcak, serin yerlerde kalmaya çalış. ";
    E3>=40; "🔥 Aşırı sıcaklar, mümkünse dışarı çıkma. "
  )
  ```

- **Rüzgar Mesajı (E14):**

  ```excel
  =IFS(
    SAAT(ŞİMDİ())>=22; "🌬️ Saat geç oldu, rüzgar nasıl olursa olsun dinlenme zamanı. ";
    SAAT(ŞİMDİ())<6; "🌬️ Gece vakti, rüzgarın sesiyle uykuya dalabilirsin. ";
    E5<1; "🍃 Neredeyse hiç rüzgar yok, hava durgun. ";
    VE(E5>=1; E5<5); "🍃 Hafif bir esinti var, yürüyüş için ideal. ";
    VE(E5>=5; E5<10); "🍃 Tatlı bir rüzgar esiyor, hava canlandırıcı. ";
    VE(E5>=10; E5<20); "💨 Rüzgar biraz kuvvetli, dikkatli ol. ";
    VE(E5>=20; E5<30); "🌪️ Sert rüzgarlar esiyor, dışarıda dikkatli ol. ";
    E5>=30; "🌪️ Fırtınalı bir hava, mümkünse evde kal. "
  )
  ```

- **Görüş Mesafesi Mesajı (E15):**

  ```excel
  =IF(
    E6<1000;
    "⚠️ Görüş mesafesi düşük, dikkatli olmalısın. ";
    ""
  )
  ```

### 8. Mesajları Birleştirin

- **E16** hücresine aşağıdaki formülü girin:

  ```excel
  =TRIM(E11 & " " & E12 & " " & "Bu arada, " & LOWER(E13) & " " & E14 & " " & E15)
  ```

### 9. Otomatik Güncelleme Ayarlayın

- **Apps Script**’te aşağıdaki kodu ekleyin:

  ```javascript
  function refreshData() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("havam");
    sheet.getRange("A1").setValue("=IMPORTJSON(\"https://api.openweathermap.org/data/2.5/weather?q=isparta,TR&lang=en&appid=YOUR_API_KEY\")");
  }

  function create30MinTrigger() {
    // Her 30 dakikada bir çalıştıran tetikleyici
    ScriptApp.newTrigger("refreshData")
        .timeBased()
        .everyMinutes(30)
        .create();
  }
  ```

- **create30MinTrigger** fonksiyonunu bir kez çalıştırın.

### 10. Verileri Web'de Yayınlayın

- **Dosya** > **Web'de Yayınla** seçeneğine gidin.
- Yayınlama ayarlarını yapın ve bağlantıyı alın.
- Bu bağlantıyı web sitenizde veya uygulamanızda kullanabilirsiniz.

## Ekran Görüntüsü

![Ekran Görüntüsü](https://raw.githubusercontent.com/metinciris/havadurumu/refs/heads/main/screen.png)

## Katkıda Bulunanlar

- [Metin Çiriş](https://metinciris.com.tr)

## Lisans

Bu proje MIT lisansı ile lisanslanmıştır. Detaylar için [LICENSE](LICENSE) dosyasına bakabilirsiniz.

## Notlar

- **API Anahtarı Güvenliği**: API anahtarınızı paylaşırken dikkatli olun ve herkese açık ortamlarda gizli tutun.
- **Hata Kontrolü**: Verilerin çekilemediği durumlar için formüllerinize hata kontrolü eklemeyi unutmayın.
- **Güncellemeler**: OpenWeatherMap API'sinde veya Google E-Tablolar fonksiyonlarında yapılan güncellemeleri takip edin ve gerekirse formüllerinizi güncelleyin.

---

Herhangi bir sorunla karşılaşırsanız veya katkıda bulunmak isterseniz lütfen bizimle iletişime geçin!
