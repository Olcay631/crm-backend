# CRM Authentication and Roles API

.NET 10 ve PostgreSQL kullanan kimlik doğrulama, kullanıcı yönetimi ve rol
listeleme servisidir.

## Roller

- `SALES_REP` — Satış Temsilcisi
- `ACCOUNTANT` — Muhasebe
- `SUPER_ADMIN` — Süper Yönetici

## Yerel çalıştırma

```powershell
Copy-Item .env.example .env
docker compose up -d --build
```

Gemini ile Google Maps konum eşleştirmesini kullanmak için Google AI Studio'dan
alınan anahtar `.env` dosyasındaki `GEMINI_API_KEY` değerine yazılmalıdır. Anahtar
kaynak koda veya frontend ortam değişkenlerine eklenmemelidir.

Yerel PostgreSQL host portu `.env` içindeki `POSTGRES_PORT` ile ayarlanır. Bu
bilgisayarda WSL PostgreSQL `5432` portunu kullandığı için `POSTGRES_PORT=5433`
seçilmelidir.

## Endpointler

- `POST /api/v1/auth/login`
- `GET /api/v1/users` — yalnızca Süper Yönetici
- `POST /api/v1/users` — yalnızca Süper Yönetici; Satış Temsilcisi veya Muhasebe hesabı oluşturur
- `GET /api/v1/departments`
- `GET /api/v1/roles`
- `GET /api/v1/product-categories`
- `GET /api/v1/product-categories/{categoryId}/products`
- `GET /api/v1/products/{productId}`
- `GET /api/v1/orders` — satış temsilcisinin siparişleri
- `GET /api/v1/orders/{orderId}` — sipariş detayı
- `POST /api/v1/orders` — sipariş oluşturur
- `GET /api/v1/customers?assignmentStatus=AwaitingSalesRepresentative` — adminin temsilci atama kuyruğu
- `GET /api/v1/customers/sales-representative-candidates` — il/ilçe ve iş yüküne göre temsilci adayları
- `POST /api/v1/customers/{id}/sales-representative-assignments` — müşteriye bir veya daha fazla bölgesel temsilci atar
- `GET /api/v1/customers/pending-dealer-assignments` — satış temsilcisinin bayi atama kuyruğu
- `POST /api/v1/customers/{id}/dealer-assignments` — bölgesel müşteri atamasını bayiyle tamamlar
- `POST /api/v1/dealer-location-lookups` — Gemini ve Google Maps ile bayi konumunu eşleştirir
- `GET /health`
- `GET /health/live`
- `GET /health/ready`

## Yerel yönetici hesabı

Yerel Docker ortamı ilk migration sırasında aşağıdaki hesabı oluşturur:

- E-posta: `admin@admin.com`
- Parola: `admin123`
- Rol: `SUPER_ADMIN`

Bu değerler `.env` içindeki `INITIAL_ADMIN_EMAIL` ve
`INITIAL_ADMIN_PASSWORD` ile değiştirilebilir. Üretimde güçlü ve benzersiz bir
parola kullanılmalıdır.

## Ürün kataloğu

Katalog verisi kategori, ürün ve ürün görseli olarak üç ayrı tabloda tutulur.
Başlangıç migration'ı örnek olarak Zemin Kaplamaları, Duvar Kaplamaları ve Yapı
Kimyasalları kategorilerini; her kategori için iki ürünü ve ürünlere ait sıralı
görsel kayıtlarını ekler.

## Demo sipariş ve risk verisi

Son beş yıla yayılan 420 sipariş; ödeme, vade, tahsilat ve risk geçmişiyle
birlikte aşağıdaki komutlarla yeniden oluşturulur:

```powershell
docker compose cp scripts/reseed-demo-order-risk-data.sql postgres:/tmp/reseed-demo-order-risk-data.sql
docker compose exec -T postgres psql -U app_user -d app_db -f /tmp/reseed-demo-order-risk-data.sql
```

Bu işlem siparişe bağlı demo kayıtlarını yeniler; müşteri, bayi, ürün,
kullanıcı ve cari risk notlarını korur.

## Migration

```powershell
dotnet ef migrations add <MigrationName> --project src/App.Infrastructure --startup-project src/App.Infrastructure --output-dir Persistence/Migrations
```
