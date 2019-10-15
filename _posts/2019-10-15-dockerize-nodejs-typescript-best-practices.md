---
title: "Docker ile Node.js TypeScript Uygulaması Oluşturma"
excerpt: "Docker imajı oluşturma, NodeJs TypeScript derleme"
tags: 
  - nodejs
  - docker
  - typescript
  - uygulama-önerileri
---

## Derleme için gerekli kütüphanelerin yüklenmesi

```
FROM node:10.16.3 AS build-deps
COPY package*.json yarn.lock* /build/
WORKDIR /build
RUN npm install
# RUN yarn
```

## TypeScript ile yazılan kodun derlenmesi

```
FROM node:10.16.3 AS compile-env
RUN mkdir /compile
COPY --from=build-deps /build /compile
WORKDIR /compile
COPY . .
RUN ./node_modules/.bin/tsc
```

## Calışma zamanı için kütüphanelerin yüklenmesi

```
FROM node:10.16.3 AS runtime-deps
COPY package*.json yarn.lock* /build/
WORKDIR /build
RUN npm install --production
# RUN yarn --prod
```

## Son imajın oluşturulması

Son imaj için derleme ve gereksinimleri yükleme adımlarında kullanılan `node:10.16.3` imajı yerine `node:10.16.3-alpine` kullanmak çok daha küçük bir imaj üretmeyi sağlar.

Calışma anında sadece daha önceki `compile-env` aşamasının derleme çıktısını kopyalamak yeterlidir.

```
COPY --from=compile-env --chown=node:node /compile/dist /app
```

Ayrıca sadece çalışma anında gerekli olacak kütüphanleri kopyalamak imajın küçük kalmasını sağlar.

```
COPY --from=runtime-deps --chown=node:node /build /app
```

Kopyalma adımlarında `--chown=node:node` klasorlerin yetkisi sınırlandırmak ve uygulamanın çalıştığı kullanıcıyı `USER node` ile belirlemek imajın güveliğini artımaya yöneliktir.

Eğer derleme çıktısı dışında başka klasörlerinde imaja dahil olması gerekiyor ise, yine aynı şekilde imaja kopyalabilir. Normal derleme çıktısına dahil olmayan konfigürasyon klasörleri buna örnek olabilir.

````
# COPY --chown=node:node config /app/config
```

```
FROM node:10.16.3-alpine AS runtime-env
WORKDIR /app
COPY --from=compile-env --chown=node:node /compile/dist /app
COPY --from=runtime-deps --chown=node:node /build /app
# eğer var ise, diğer gerekli klasörler bu noktada kopyalanabilir
# COPY --chown=node:node config /app/config
USER node

# EXPOSE 8000
# CMD [ "node", "server.js" ]
```
