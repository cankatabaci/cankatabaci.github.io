---
title       : "GitHub Actions ile NPM Paket Yayımlanmasını Otomatikleştirmek"
description : "NPM'de yayımlanacak bir paketin, GitHub Actions kullanarak otomatik olarak yayımlanması."
tags        : [ npm, "github-actions" ]
toc         : true
---

## Giriş

### Problemin Çıkış Noktası
Futbol temalı bir projede lig maçlarının fikstürünün oluşturulması için bir NPM package'i hazırlandı. Round-robin tournament algoritması ile her katılımcının diğer tüm katılımcılar ile sırayla karşılaştığı, lig usulü bir fikstür oluşturucusu olan bu paketin NPM'de yayımlanmasının otomatikleştirilmesi gerekiyordu.

![Example image](/rrt.png)    


### Çözüm Yolu
Versiyon kontrol olarak GitHub kullandığım için, `GitHub Actions` ile NPM'i entegre edip, her yeni versiyon çıktığımda, bu paketin otomatik olarak NPM'de yayımlanması.


## NPM  Access Token Oluşturmak
NPM üzerinde dışarıdan bir işlem yapabilmemiz için Token tabanlı bir yetkilendirme mekanizması kullanıyor. NPM'e giriş yaptıktan sonra, profil fotoğrafımıza tıklayıp, açılan menüden "Access Tokens" menüsünü açıyoruz.

![Example image](/npm-menu-token.png)    


Açılan ekranda, `Generate New Token` butonuna tıkladıktan sonra, açılan ekranda, oluşturacağımız token için bir isim girmemiz ve ardından bu token'ın türünü seçmemiz gerekiyor. Biz GitHub workflow kullanacağımız için oluşturacağımız token `Automation` türünde olmalı.

![Example image](/npm-generate-token.png)  

Generate Token butonuna tıkladıktan sonra, gelen ekranda 1 kereye mahsus token'ın tam hali gözükecek. Bu token'ın tam hali bir daha asla görüntülenemeyeceği için bir yere not almalısınız.

![Example image](/npm-copy-token.png)  

## GitHub Secret Key Oluşturmak
Oluşturduğumuz bu token'ı GitHub'da kullanabilmemiz için, hangi repository'de kullanacaksak o repository'ye secret olarak eklememiz gerekiyor. Bunun için, ilgili repository'yi açtıktan sonra, Settings menüsünden, Secrets > Actions altından yeni bir key-value olarak ekliyoruz. Yeni secret'ı oluştururken verdiğimiz `name` değeri, GitHub workflow dosyasında birebir aynı isimle kullanacağımız için önemli. 

![Example image](/github-actions-secret.png)  

Oluşturduğumuz secret'ı kullanarak, NPM'e paket yayımlamak için, GitHub Actions'ı kullanacağız. GitHub Actions, ilgili repository'nin `.github/workflows/` dizini altında bulunan .yml dosyaları ile konfigüre edilir ve .yml'da kurguladığınız şekilde çalışır. Aşağıda örnek bir .yml dosyasını görebilirsiniz.


## npm-publish.yml dosyası

```yml
name: Publish Package to npmjs
on:
  release:
    types: [published]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      # Setup .npmrc file to publish to npm
      - uses: actions/setup-node@v2
        with:
          node-version: '16.x'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```
> **Not:** Son satırda bulunan `secrets.NPM_TOKEN` ile repository özelinde tanımladığımız `NPM_TOKEN` isimli secret'ı kullanmaktadır.

> **Not:** 3. satırdaki release tanımı ile, bu repository'den yeni bir release çıkıldığında bu Action'ın çalışması sağlanmıştır. Eğer spesifik bir branch'e (aşağıdaki örnekte main branch'i) her commit'te çalışması isteniyorsa aşağıdaki şekilde düzenlemesi gerekir:

```yml
name: Publish Package to npmjs
 on:
   push:
     branches:
      - main
```     


## GitHub Actions
`.github/workflows/` dizini altına oluştuğumuz `npm-publish.yml` dosyasını GitHub'a commitledikten sonra, GitHub Action'larımız ilgili konfigürasyonlara göre çalışacaktır. 

![Example image](/github-actions.png)  


## GitHub Status Badge
Oluşturduğumuz GitHub Action'ın başarılı çalışıp-çalışmadığını bir Badge ile görüntüleyebiliriz. Badge'i oluşturduktan sonra bağlantıyı kopyalayarak ilgili projenin README.MD dosyasına eklerseniz hem projenin GitHub sayfasında hem de NPM sayfasında gözükecektir.

![Example image](/create-badge.png)  
![Example image](/readme-badge.png)  


## NPM
Son olarak, yayımladığımız package'i NPM üzerinde görüntüleyebiliriz:

![Example image](/npm-homepage.png)  


İlgili projeye [GitHub](https://github.com/cankatabaci/fixture-creator) ve [NPM](https://www.npmjs.com/package/fixture-creator) üzerinden ulaşabilirsiniz.

## Kaynaklar
Round-robin tournament fotoğrafı: [Qniemiec](https://commons.wikimedia.org/wiki/File:Round-robin_tournament_10teams_en.png), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0), via Wikimedia Commons

GitHub Actions: [Publishing NodeJs Packages](https://docs.github.com/en/actions/publishing-packages/publishing-nodejs-packages)