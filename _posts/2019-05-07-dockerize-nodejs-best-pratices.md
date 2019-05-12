---
title: "Docker ile Node.js Uygulaması Oluşturma"
excerpt: "Docker imajı oluşturma, genel ve güvenlik odaklı uygulama önerileri"
tags: 
  - nodejs
  - docker
  - uygulama-önerileri
---

1. Sabit etiketler ve minimal imajlar kullanın

Mümkün ise mutlaka Node.js LTS sürümlerini sabit etiket ile kullanın, sabit etiket kullanmak güvenlik yamalarını otomatik olarak almayı engellese de kontrolsüz güncellemelerin uygulamanızı kırmasını engeller.

```text
FROM node:12.2.0-alphine
```

1. Küçük ve güvenli imajları için [çok aşamalı derleme](https://docs.docker.com/develop/develop-images/multistage-build/) kullanın

İmaj derlemesi sırasında ihtiyaç duyabileceğiniz şifreleri ya da hassas bilgileri ara katmanlarda kullanarak, son imaja hiçbir hassas verinin ulaşmamasını sağlayabilirsiniz.

Farklı katmanlarda farklı imajlar kullanılabileceği gibi, son katmanda mümkün ise minimal imajlar tercih edin (`*-alpine gibi`). Minimal imajlar daha az saldırı yüzeyi ve daha güvenli uygulama imajları demektir.

```text
FROM node:12.2.0 AS build-env

ARG github_token_arg=default_value
ENV github_token=$github_token_arg

...

FROM node:12.2.0-alpine AS runtime-env

...
```

Örnekteki `github_token_arg` asağıdaki gibi `--build-arg` parametresi kullanılarak sağlanabilir.

```text
docker build --build-arg github_token_arg=1234 -t myapp .
```

1. Npm kütüphanelerini ayrı bir katmanda yükleyin

Bunun temelde iki faydası vardır:

- Üst üste yapılan derlemelerde önbelleğe alınmış Docker katmanlarından faydalanmamızı sağlar
- package.json dosyasında belirtilen kütüphane versiyonları otomatik olarak sabitlenmiş olur

```text
FROM node:12.2.0 AS build-env

ARG github_token_arg=default_value
ENV github_token=$github_token_arg

# package.json and package-lock.json
COPY package*.json ./

RUN npm install --production

...

FROM node:12.2.0-alpine AS runtime-env

RUN mkdir /app

COPY --from=build-env ./ /app
...
```

NOT: `npm install` yerine `npm install --production` kullanmak `devDependencies` altındaki kütüphaneleri yüklemeyeceği icin son imaj yükünü küçültmeye fayda sağlar.

1. En az ayrıcalıklı kullanıcı

Docker varsayılan olarak root kullanıcı ile çalışır, bu uygulamanın Docker ana bilgisayarında root kullanıcı erişimine sahip olabileceği anlamına gelir. Eger uygulamada bir güvenik açığı var ise, bu saldırı yüzeyini artırır ve ayrıcaklık yükseltme saldırıları için kolay bir yol sağlar.

```text
FROM node:12.2.0 AS build-env

ARG github_token_arg=default_value
ENV github_token=$github_token_arg

# package.json and package-lock.json
COPY package*.json ./

RUN npm install --production

FROM node:12.2.0-alpine AS runtime-env

RUN mkdir /app

COPY --from=build-env ./ /app

RUN chown -R node:node /app

USER node

EXPOSE <uygulama portu>

CMD [“node”, “index.js”]
```

Riski en aza indirmek için, yukarıdaki gibi uygulamanın sadece belirli bir grup ve kullanıcı ile sınırlandırılması faydalıdır.

Referanslar:

- [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- [Docker Image Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)
- [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md)

