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

[cite_start]Sistem **.NET 8** sürümünü gerektirir[cite: 80]. [cite_start]Altyapıyı hızlıca ayağa kaldırmak için Docker kullanılması önerilir[cite: 81, 82, 84].

```bash
# Bağımlılıkları başlatın (PostgreSQL, Redis, ElasticSearch)
docker-compose up -d

# Projeleri çalıştırın
dotnet run --project ./Paywall.AuthApi
dotnet run --project ./Paywall.PaymentApi
