## Örnek Proje Yapısı

Aşağıda **orta seviye** açıklamalarla desteklenmiş, Node.js + MongoDB içeren bir Docker Compose projesinin örnek yapısını bulabilirsiniz.

```
multi-container-app/
├── docker-compose.yml     # Tüm servis tanımlamalarını içerir
└── backend/
    ├── Dockerfile        # Node.js konteyneri için talimatlar
    ├── package.json      # Proje bağımlılıkları
    └── index.js          # Basit Express API
```

---

### 1. docker-compose.yml

```yaml
version: '3.8'

services:
  # API servisimiz
  backend:
    build: ./backend             # Dockerfile'ın bulunduğu klasör
    ports:
      - "3000:3000"            # Host:Container port eşlemesi
    depends_on:
      - mongo                    # MongoDB başlatıldıktan sonra ayağa kalk
    environment:
      - MONGO_URL=mongodb://mongo:27017/test  # DB bağlantı adresi
    networks:
      - app-network              # Ortak ağ tanımı

  # MongoDB servisi
  mongo:
    image: mongo:5               # Hazır MongoDB imajı
    volumes:
      - mongo-data:/data/db      # Veritabanı kalıcılığı
    networks:
      - app-network

# Kullanılan volume ve ağları tanımlıyoruz
volumes:
  mongo-data:

networks:
  app-network:
```

* `depends_on`: Başlatma sırasını kontrol eder, sağlık durumunu **garanti etmez**.
* `volumes`: Verileri konteyner yeniden başlatsa dahi korur.
* `networks`: Servisler arası güvenli ve izole iletişim sağlar.

---

### 2. backend/Dockerfile

```dockerfile
# 1. Hafif bir Node.js imajı seçiyoruz
FROM node:18-alpine

# 2. Çalışma dizinini ayarlıyoruz
WORKDIR /app

# 3. Package dosyalarını kopyalayıp bağımlılıkları yüklüyoruz
COPY package*.json ./
RUN npm install --production  # Sadece üretim bağımlılıkları

# 4. Uygulama dosyalarını kopyala
COPY . .

# 5. Uygulamayı dinleyeceği portu aç
EXPOSE 3000

# 6. Konteyner başladığında çalışacak komut
CMD ["node", "index.js"]
```

* `node:18-alpine`: Minimal boyutlu, güvenlik güncellemeleri olan Alpine tabanlı imaj.
* `WORKDIR`: Her çalıştırmada aynı dizini kullanır.
* `--production`: `devDependencies`’leri yüklemez, imajı hafifletir.

---

### 3. backend/index.js

```javascript
// 1. Gerekli kütüphaneleri içe aktar
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = process.env.PORT || 3000;
// Ortam değişkeninden MongoDB URL'sini al, yoksa varsayılan kullan
const MONGO_URL = process.env.MONGO_URL || 'mongodb://localhost:27017/test';

// 2. MongoDB bağlantısını başlat
mongoose.connect(MONGO_URL, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
  .then(() => console.log('✅ MongoDB bağlantısı başarılı!'))
  .catch(err => console.error('❌ MongoDB bağlantı hatası:', err));

// 3. Basit bir GET endpoint
app.get('/', (req, res) => {
  res.send('🚀 API çalışıyor ve MongoDB bağlı!');
});

// 4. Sunucuyu başlat
app.listen(PORT, () => {
  console.log(`🌍 Sunucu http://localhost:${PORT} adresinde dinliyor`);
});
```

* Ortam değişkenleri (`process.env`) kullanılarak **esneklik** sağlanır.
* Bağlantı başarı ve hata durumları `.then/.catch` ile yönetilir.
* Kod okunurluğu için **anlamlı log mesajları** eklenmiştir.

---

Bu örnekle, **orta seviye** yorum satırlarının nasıl eklendiğini ve yapılandırmanın nasıl yapıldığını görmüş olduk. Artık bu temel üzerinden `healthcheck`, ek servisler (Redis, pgAdmin, vb.) veya gelişmiş retry mekanizmaları ekleyebilirsiniz!
