---
title: "JsDoc kullanımı ve API açıklamaları  oluşturma"
excerpt: "JSDoc açıklama etiketleri kullanarak markdown & html formatında API belgeleri oluşturma"
toc: true
tags: 
  - jsdoc
  - eslint
  - bitbucket
  - markdown
---

[JSDoc](https://devdocs.io/jsdoc/) JavaScript kaynak dosyalarına ek açıklama için kullanılan metin işaretleme dilidir. JSDoc yorumları kullanarak programcılar uygulamanın ne yaptığını yazabilir ve bunu dökümante edebilir. Bundan sonra yazılanlar farklı araçlar vasıtası ile HTML veya Zengin Metin Biçimi haline getirilir.

## Kurulum

```bash
npm install jsdoc --save-dev
```

[ES6 Class](http://es6-features.org/#ClassDefinition)'larının düzgün işlenmesi için

```bash
npm install jsdoc-export-default-interop --save-dev
```

## Kullanım

[JSDoc](https://devdocs.io/jsdoc/about-configuring-jsdoc) configürasyonu

jsdoc_conf.json
```json
{
  "tags": {
      "allowUnknownTags": true
  },
  "source": {
      "includePattern": ".+\\.js(doc|x)?$",
      "excludePattern": "(^|\\/|\\\\)_"
  },
  "plugins": ["node_modules/jsdoc-export-default-interop/dist/index"],
  "templates": {
      "cleverLinks": false,
      "monospaceLinks": false,
      "default": {
          "outputSourceFiles": true
      }
  }
}
```

npm script;

```json
  "scripts": {
    "doc-html": "jsdoc <işlenecek js dosyalarının bulunduğu klasör> -r -c <configürasyon dosyasının adı>.json -d <çıktı klasörü>"
  }
```

örnek;

```json
  "scripts": {
    "doc-html": "jsdoc lib -r -c jsdoc_conf.json -d jsdoc_out"
  }
```

HTML dosyalarını statik olarak sunmak için;

```bash
npx serve jsdoc_out
```

## Markdown üretimi

```bash
npm install jsdoc-to-markdown --save-dev
```

Eger markdown dosyası BitBucket üzerinde tutulacak ise;

```bash
npm install dmd-bitbucket --save-dev
```

[Döküman şablonu](https://github.com/jsdoc2md/jsdoc-to-markdown/wiki/Create-a-README-template) oluşturma

ApiDocs.hbs
```text
# Documentation

{{>main}}
```

npm script;

```json
  "scripts": {
    "doc-md": "jsdoc2md --plugin dmd-bitbucket -c jsdoc_conf.json -t <şablon dosyası>.hbs lib/**/*.js > <çıktı>.md",
  }
```

örnek;

```json
  "scripts": {
    "doc-md": "jsdoc2md --plugin dmd-bitbucket -c jsdoc_conf.json -t ApiDocs.hbs lib/**/*.js > ApiDocs.md",
  }
```

## Eslint ile JsDoc açıklamalarının kontrolü

```bash
npm install eslint eslint-plugin-jsdoc --save-dev
```

```yaml
---
plugins:
  - jsdoc
extends:
  - plugin:jsdoc/recommended
```

npm script;

```json
  "scripts": {
    "lint": "eslint .",
    "doc-md": "jsdoc2md --plugin dmd-bitbucket -c jsdoc_conf.json -t ApiDocs.hbs lib/**/*.js > ApiDocs.md",
    "doc-html": "jsdoc lib -r -c jsdoc_conf.json -d jsdoc_out"
  },
```
