<h1 align="center">
<a href="https://github.com/kullanici-adi/paywall-project">
<picture>
<source height="125" media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/gofiber/docs/master/static/img/logo-dark.svg">
<img height="125" alt="Paywall Logo" src="./indir.png">
</picture>
</a>



Paywall Backend Case Project


<a href="#">
<img src="https://img.shields.io/badge/.NET%208-512bd4?style=flat-square&logo=dotnet&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/Redis-dc382d?style=flat-square&logo=redis&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/Hangfire-522d82?style=flat-square">
</a>
<a href="#">
<img src="https://img.shields.io/badge/ElasticSearch-005571?style=flat-square&logo=elasticsearch&logoColor=white">
</a>
</h1>

<p align="center">
<em><b>Paywall</b>, farklı merchant'ların ödeme almasını sağlayan sadeleştirilmiş bir <b>ödeme işleme altyapısıdır</b>. Mühendislik yaklaşımını ve teknik karar alma süreçlerini  yansıtan bu proje; <b>AuthApi</b> ve <b>PaymentApi</b> olmak üzere iki ana servisten oluşur.</em>
</p>

## ⚙️ Installation (Kurulum)

Sistem **.NET 8** sürümünü gerektirir.Uygulamayı yerel makinenizde çalıştırmak için aşağıdaki adımları sırasıyla takip edin:

### 1. Repoyu Klonlayın
Öncelikle terminalinizi açın ve projeyi bilgisayarınıza indirin:
```bash
git clone [https://github.com/kullanici-adi/paywall-project.git](https://github.com/kullanici-adi/paywall-project.git)

İndirme tamamlandıktan sonra proje klasörüne giriş yapın:

cd paywall-project

2. Docker Compose ile Altyapıyı Başlatın
Aşağıdaki komut, docker-compose.yml dosyasındaki tüm bağımlılıkları (Veritabanı, Cache ve Log servisleri) otomatik olarak indirir ve yapılandırır:

```bash
docker-compose up-d

3. Veritabanı Şemasını Uygulayın
Konteynerlar ayağa kalktıktan sonra, PostgreSQL üzerinde tabloların oluşması için migration komutunu çalıştırın:

```bash
dotnet ef database update --project Paywall.Persistence

4. Servisleri Çalıştırın
Sistemdeki servisleri Docker üzerinden veya yerel terminalinizden başlatabilirsiniz:

Kimlik Doğrulama Servisi (AuthApi):
```bash
dotnet run --project ./Paywall.AuthApi

Ödeme İşlem Servisi (PaymentApi):
```bash
dotnet run --project ./Paywall.PaymentApi
