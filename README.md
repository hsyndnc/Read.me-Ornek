<h1 align="center"> 💳 Paywall Backend Case Project </h1>
<p align="center">
<em><b>Paywall</b>, farklı merchant'ların güvenli ve hızlı ödeme almasını sağlayan merkezi bir ödeme işleme altyapısıdır. Bu proje; <b>AuthApi</b> ve <b>PaymentApi</b> olmak üzere iki ana servisten oluşan, yüksek performanslı ve ölçeklenebilir bir ödeme sistemini modeller. </em>
</p>

⚙️ Kurulum & Çalıştırma
Proje .NET 8 kullanılarak geliştirilmiştir. Docker Compose ile tüm bağımlılıkları (PostgreSQL, Redis, ElasticSearch) tek seferde ayağa kaldırabilirsiniz.

git clone https://github.com/kullanici/paywall-backend-case

# Docker bağımlılıklarını başlatın
docker-compose up -d

# Veritabanını güncelleyin (Code-First Migration)
dotnet ef database update --project Paywall.Persistence
