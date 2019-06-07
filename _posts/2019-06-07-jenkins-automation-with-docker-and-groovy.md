---
title: "Groovy ve Docker ile Jenkins otomasyonu"
excerpt: "Groovy ile konfigurasyon ve eklentilerin yüklenmesi, Özel Jenkins Docker imajı"
toc: true
tags: 
  - jenkins
  - docker
  - groovy
---

Jenkins ve eklentilerin konfigurasyonu ve hatta [Pipeline](https://jenkins.io/doc/book/pipeline/)'ların konfigurasyonu [Groovy](https://groovy-lang.org/) ile yapılabilir.
`init.groovy.d` dizini `JENKINS_HOME` altında opsiyonel olarak yaratabileceğiniz bir dizindir. Bu dizine, `eklentiler yüklendikten sonra` ve `jenkins hizmet vermeye başlamadan önce` çalışmasını istediğiniz Groovy betiklerini yerleştirerek, Jenkins ve eklentilerin konfigurasyonu efektif bir şekilde yapabilirsiniz.

## Groovy betiklerini yerleştirmek

Groovy betikleri asağıdaki yöntemler ile `init.groovy.d` dizini altına konabilir.

### Docker volume kullanılarak

```bash
docker run --rm -ti -p 8080:8080 \
    -v $PWD/init.groovy.d:/var/jenkins_home/init.groovy.d \
    jenkins/jenkins:lts-alpine
```

### Özel Docker yükü çıkartarak

Bu adım için, [Jenkins](https://github.com/jenkinsci/docker)'in orijinal dosyalarını kopyalayıp güncellemek en basit yöntemlerden biridir.

İhtiyaç duyacağınız dosyalar (aslında sadece `Dockerfile`'da değişiklik gerekli);

1. [Dockerfile](https://github.com/jenkinsci/docker/blob/master/Dockerfile)
2. [install-plugins.sh](https://github.com/jenkinsci/docker/blob/master/install-plugins.sh)
3. [jenkins-support](https://github.com/jenkinsci/docker/blob/master/jenkins-support)
4. [jenkins.sh](https://github.com/jenkinsci/docker/blob/master/jenkins.sh)
5. [plugins.sh](https://github.com/jenkinsci/docker/blob/master/plugins.sh)
6. [tini_pub.gpg](https://github.com/jenkinsci/docker/blob/master/tini_pub.gpg)
7. [tini-shim.sh](https://github.com/jenkinsci/docker/blob/master/tini-shim.sh)

Groovy bektiklerini kopyalama adımı dizinin yaratıldığı adımdan hemen sonrasına konabilir.

```bash
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

COPY groovy/* /usr/share/jenkins/ref/init.groovy.d/
```

#### Eklentilerin yönetimi

Bu yöntem ile ayrıca eklentilerin yüklenmesi otomatize edilebilir. Yapılaması gereken yukarıdaki dosyaların bulunduğu dizine, kullanılan eklentileri ve versiyonlarını içeren [plugins.txt](https://github.com/ridakk/jenkins/blob/master/build/plugins.txt) dosyası yaratmak ve `Dockerfile`'da en sona aşağıdaki satıları eklemektir.

```bash
RUN chmod +x /usr/local/bin/plugins.sh
COPY plugins.txt plugins.txt
RUN /usr/local/bin/plugins.sh plugins.txt
```

Bu adım ile eklentiler `docker build` sırasında ile yüklenmiş olacak, `build` süresi ve docker imaj boyutu artacak olsa bile, Jenkins ilk kez çalıştırılırken artık eklentileri indirmeyecek ve çok daha hızlı isteklere yanıt vermeye başlayacaktır. Ve de eklenti versiyonlarının yönetimi kodlanmış olacaktır.

Her eklenti ekleme ve çıkarma için aşağıdaki adımların uygulanmalıdır.

1. Jenkins UI üstünden plugin eklenir ya da çıkarılır
2. [Groovy betiği](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/get-jenkins-plugin-list.groovy) ile güncel eklenti listesi Jenkins Script Console üzerinden alınır ve `plugins.txt` güncellenir.

Bu yukarıdaki adımlar ile, eklenti ekleme ve çıkarma işlemlerinde eksik veya artık bağımlılıklar yönetilmiş olur.

[Dockerfile](https://github.com/ridakk/jenkins/blob/master/build/Dockerfile)'ın tam haline burdan erişilebilir.

#### Jenkins versiyonunu değiştirmek

ilk olarak Dockerfile'da Jenkins versiyon güncellenmeli

```bash
ENV JENKINS_VERSION ${JENKINS_VERSION:-<new-version>}
```

Sonra da ilgili versiyonun checksum'ı güncellenmelidir. Checksum'lara bu [linkten](http://mirrors.jenkins-ci.org/war-stable/) ulaşilabilir.

```bash
ARG JENKINS_SHA=<new-checksum>
```

## Groovy betiklerini hazırlama

Groovy betiklerinin bazıları ve parçaları internette bulunabilse de, en iyi kaynak Jenkins ve eklentilerin kaynak kod ve JavaDoc'ları. Eğer var ise eklentilerin test kodları da oldukça faydalı olabiliyor.

Örnek betiklerden bazıları:

- [Github eklenti konfigurasyonu](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-github-plugin.groovy)
- [NodeJs eklenti konfigurasyonu](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-nodejs-installers.groovy)
- RBAC konfigurasyonu
  - [Rol bazlı güvelik](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-rbac-security.groovy)
  - [Genel rol tanımları](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-rbac-global-roles.groovy)
  - [Proje bazlı rol tanımları](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-rbac-project-roles.groovy)
- [SonarQube eklenti konfigurasyonu](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/setup-sonarqube-plugin.groovy)
- [Pipeline da dinamik içerikli dropdown menu kullanımı](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/create-workflow-job-with-dynamic-choice-parameter.groovy)
- [Pipeline da birbirine bağımlı dropdown menulerin kullanımı](https://github.com/ridakk/jenkins/blob/master/groovy-scripts/create-workflow-job-with-cascade-choice-parameter.groovy)

Tüm betikler [için](https://github.com/ridakk/jenkins/tree/master/groovy-scripts).
