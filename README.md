# mattermost-project
Docker-Compose-using-Mattermost (OnlyOffice-MinIO-PostreSQL)

Bu proje, Docker Compose kullanarak Mattermost, OnlyOffice, PostgreSQL ve MinIO servislerini entegre etmektedir.

## Gereksinimler

- Docker
- Docker Compose

## Kurulum

1. **Repository'i Klonlayın:**
    ```bash
    git clone https://github.com/kullaniciadi/mattermost-projesi.git
    cd mattermost-projesi
    ```

2. **Servisleri Başlatın:**
    ```bash
    docker-compose up -d
    ```

3. **Mattermost'a Erişim:**
    - Tarayıcınızı açın ve `http://localhost:8065` adresine gidin.

## Yapılandırma

### Docker Compose Dosyası

#### PostgreSQL Servisi

```yaml
postgres:
  image: postgres:latest
  environment:
    POSTGRES_DB: mattermost
    POSTGRES_USER: mmuser
    POSTGRES_PASSWORD: mmuserpassword
  volumes:
    - postgres-data:/var/lib/postgresql/data
  networks:
    - mattermost-network
Açıklama:

image: PostgreSQL'in en son sürümünü kullanır.
environment:
POSTGRES_DB: Oluşturulacak veritabanı adı.
POSTGRES_USER: Veritabanı kullanıcısı.
POSTGRES_PASSWORD: Veritabanı şifresi.
volumes: Kalıcı veri depolama için kullanılır.
networks: Belirtilen Docker ağına bağlanır.
Mattermost Servisi
yaml
Kodu kopyala
mattermost:
  image: mattermost/mattermost-team-edition:latest
  environment:
    MM_SQLSETTINGS_DATASOURCE: postgres://mmuser:mmuserpassword@postgres:5432/mattermost?sslmode=disable&connect_timeout=10
    MM_FILESETTINGS_DRIVERNAME: amazons3
    MM_FILESETTINGS_AMAZONS3ACCESSKEYID: minioaccesskey
    MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY: miniosecretkey
    MM_FILESETTINGS_AMAZONS3BUCKET: mattermost
    MM_FILESETTINGS_AMAZONS3ENDPOINT: minio:9000
    MM_FILESETTINGS_AMAZONS3SSL: "false"
    MM_FILESETTINGS_AMAZONS3SIGNV2: "true"
  volumes:
    - ./plugins:/mattermost/plugins
    - mattermost-data:/mattermost/data
  ports:
    - "8065:8065"
  depends_on:
    - postgres
    - minio
    - onlyoffice
  networks:
    - mattermost-network
Açıklama:

image: Mattermost'un en son sürümünü kullanır.
environment:
MM_SQLSETTINGS_DATASOURCE: PostgreSQL bağlantı URI'sı.
MM_FILESETTINGS_DRIVERNAME: Dosya depolama sürücüsü olarak amazons3 kullanır.
MM_FILESETTINGS_AMAZONS3ACCESSKEYID: MinIO erişim anahtarı.
MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY: MinIO gizli erişim anahtarı.
MM_FILESETTINGS_AMAZONS3BUCKET: Depolama kovası adı.
MM_FILESETTINGS_AMAZONS3ENDPOINT: MinIO endpoint'i.
MM_FILESETTINGS_AMAZONS3SSL: SSL kullanımı.
MM_FILESETTINGS_AMAZONS3SIGNV2: V2 imzalama.
volumes: Kalıcı veri ve eklenti depolama için kullanılır.
ports: Web arayüzü için port ayarları.
depends_on: Diğer servislerin başlatılmasını bekler.
networks: Belirtilen Docker ağına bağlanır.
MinIO Servisi
yaml
Kodu kopyala
minio:
  image: minio/minio:latest
  environment:
    MINIO_ROOT_USER: minioaccesskey
    MINIO_ROOT_PASSWORD: miniosecretkey
    MINIO_DEFAULT_BUCKETS: mattermost
  ports:
    - "9000:9000"
    - "9001:9001"
  volumes:
    - minio-data:/data
  command: server /data --console-address ":9001"
  networks:
    - mattermost-network
Açıklama:

image: MinIO'nun en son sürümünü kullanır.
environment:
MINIO_ROOT_USER: MinIO yönetici kullanıcı adı.
MINIO_ROOT_PASSWORD: MinIO yönetici şifresi.
MINIO_DEFAULT_BUCKETS: Varsayılan kova adı.
ports: HTTP ve konsol portları.
volumes: Kalıcı veri depolama.
command: MinIO sunucu başlatma komutu.
networks: Belirtilen Docker ağına bağlanır.
OnlyOffice Servisi
yaml
Kodu kopyala
onlyoffice:
  image: onlyoffice/documentserver:7.1
  environment:
    JWT_ENABLED: "true"
    JWT_SECRET: myjwtsecret
    USE_UNAUTHORIZED_STORAGE: true
    JWT_IN_BODY: true
    JWT_HEADER: AuthorizationJwt
  volumes:
    - onlyoffice-data:/var/www/onlyoffice/Data
  ports:
    - "80:80"
  networks:
    - mattermost-network
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost/healthcheck"]
    interval: 30s
    timeout: 10s
    retries: 3
Açıklama:

image: OnlyOffice Document Server'ın belirtilen sürümünü kullanır.
environment:
JWT_ENABLED: JWT kimlik doğrulaması etkin.
JWT_SECRET: JWT gizli anahtarı.
USE_UNAUTHORIZED_STORAGE: Yetkisiz depolama kullanımı.
JWT_IN_BODY: JWT'nin HTTP istek gövdesinde iletilmesi.
JWT_HEADER: JWT'nin HTTP başlığında iletilmesi.
volumes: Kalıcı veri depolama.
ports: HTTP trafiği için port.
networks: Belirtilen Docker ağına bağlanır.
healthcheck: Sağlık kontrolü ayarları.
Docker Volume'leri
yaml
Kodu kopyala
volumes:
  postgres-data:
    PostgreSQL verilerini saklamak için.
  minio-data:
    MinIO verilerini saklamak için.
  mattermost-data:
    Mattermost verilerini saklamak için.
  onlyoffice-data:
    OnlyOffice verilerini saklamak için.
Docker Ağı
yaml
Kodu kopyala
networks:
  mattermost-network:
    driver: bridge
Kullanım
1. Mattermost ve MinIO Hesap Oluşturma
Mattermost: http://localhost:8065 adresine giderek yeni bir hesap oluşturun.
MinIO: http://localhost:9001 adresine giderek yeni bir hesap oluşturun.
2. MinIO'da Bucket Oluşturma
MinIO'da mattermost adında bir bucket oluşturun.

3. PostgreSQL'de OnlyOffice İçin Yeni Database Oluşturma
PostgreSQL yönetim aracını kullanarak onlyoffice adında yeni bir veritabanı oluşturun.

sql
Kodu kopyala
CREATE DATABASE onlyoffice;
GRANT ALL PRIVILEGES ON DATABASE onlyoffice TO ooouser;
4. Mattermost Plugins Dosyası Kurulumu
Docker Compose dosyasında tanımladığınız ./plugins konumuna Mattermost eklentilerini yükleyin.

5. Mattermost Konfigürasyonu
Tarayıcınızdan Mattermost'a erişin (localhost:8065).
Yönetici olarak giriş yapın ve gerekli ayarları yapılandırın.


Bu kılavuzu takip ederek Mattermost, MinIO, PostgreSQL ve OnlyOffice servislerini başarıyla kurabilir ve yapılandırabilirsiniz. Herhangi bir sorunuz olursa bana sormaktan çekinmeyin.






postgres:
  image: postgres:latest
  environment:
    POSTGRES_DB: mattermost
    POSTGRES_USER: mmuser
    POSTGRES_PASSWORD: mmuserpassword
  volumes:
    - postgres-data:/var/lib/postgresql/data
  networks:
    - mattermost-network 
    ´´´
PostgreSQL Servisi Açıklaması:
- **image**: PostgreSQL'in Docker Hub'dan (`postgres` deposundan) en son sürümünü (`latest` etiketiyle) kullanır. PostgreSQL'in en son resmi sürümünü sağlayan resmi Docker imajıdır.

- **environment**:
  - `POSTGRES_DB`: PostgreSQL'de oluşturulacak olan veritabanının adı. Burada `mattermost` olarak belirlenmiştir.
  - `POSTGRES_USER`: PostgreSQL'e bağlanacak kullanıcının adı. Burada `mmuser` olarak belirlenmiştir.
  - `POSTGRES_PASSWORD`: PostgreSQL kullanıcısının şifresi. Burada `mmuserpassword` olarak belirlenmiştir. Bu ortam değişkenleri, PostgreSQL veritabanının ilk kurulumunda kullanılır ve bağlantı ayarlarını belirler.

- **volumes**:
  - Host (ana bilgisayar) dosya sistemi ile PostgreSQL konteyneri arasında kalıcı veri depolama sağlar. `postgres-data:/var/lib/postgresql/data` adında bir Docker volume'ü tanımlar ve PostgreSQL veritabanı verilerini bu volume üzerine kaydeder. Bu, PostgreSQL konteynerinin durduğunda veya yeniden başladığında veri kaybını önlemek için önemlidir.

- **networks**:
  - Docker ağı tanımlar ve PostgreSQL servisinin hangi ağa bağlı olacağını belirtir. `mattermost-network` adında bir Docker ağına bağlanır. Bu ağ, PostgreSQL ile diğer servisler (örneğin Mattermost, MinIO) arasında iletişimi sağlar.

  ```yaml
mattermost:
  image: mattermost/mattermost-team-edition:latest
  environment:
    MM_SQLSETTINGS_DATASOURCE: postgres://mmuser:mmuserpassword@postgres:5432/mattermost?sslmode=disable&connect_timeout=10
    MM_FILESETTINGS_DRIVERNAME: amazons3
    MM_FILESETTINGS_AMAZONS3ACCESSKEYID: minioaccesskey
    MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY: miniosecretkey
    MM_FILESETTINGS_AMAZONS3BUCKET: mattermost
    MM_FILESETTINGS_AMAZONS3ENDPOINT: minio:9000
    MM_FILESETTINGS_AMAZONS3SSL: "false"
    MM_FILESETTINGS_AMAZONS3SIGNV2: "true"
  volumes:
    - ./plugins:/mattermost/plugins
    - mattermost-data:/mattermost/data
  ports:
    - "8065:8065"
  depends_on:
    - postgres
    - minio
    - onlyoffice
  networks:
    - mattermost-network
    ´´´
Mattermost Servisi Açıklaması:
- **image**: Mattermost'un Docker Hub'dan (`mattermost` deposundan) en son sürümünü (`latest` etiketiyle) kullanır. Mattermost, ekibinizin işbirliği yapabilmesi için oluşturulmuş açık kaynaklı bir mesajlaşma ve işbirliği platformudur.

- **environment**:
  - `MM_SQLSETTINGS_DATASOURCE`: PostgreSQL veritabanına bağlantı bilgilerini içeren URI. `mmuser` kullanıcı adı ve `mmuserpassword` şifresi ile bağlanır.
  - `MM_FILESETTINGS_DRIVERNAME`: Dosya depolama sürücüsü olarak `amazons3` (Amazon S3 uyumlu) kullanılır.
  - `MM_FILESETTINGS_AMAZONS3ACCESSKEYID`: MinIO servisine erişmek için kullanılan erişim anahtarı.
  - `MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY`: MinIO servisine erişmek için kullanılan gizli erişim anahtarı.
  - `MM_FILESETTINGS_AMAZONS3BUCKET`: Mattermost için dosyaların depolanacağı Amazon S3 uyumlu kova (bucket) adı.
  - `MM_FILESETTINGS_AMAZONS3ENDPOINT`: MinIO servisinin URL'si (`minio:9000`), Mattermost'un dosyaları depolamak için erişeceği adres.
  - `MM_FILESETTINGS_AMAZONS3SSL`: MinIO ile SSL kullanımını belirtir (`false` olarak ayarlanmıştır).
  - `MM_FILESETTINGS_AMAZONS3SIGNV2`: MinIO ile V2 stilinde imzalama kullanımını belirtir (`true` olarak ayarlanmıştır).

- **volumes**:
  - Host üzerindeki `./plugins` klasörünü Mattermost konteyneri içindeki `/mattermost/plugins` klasörüne bağlar, bu şekilde Mattermost eklentilerini yükleyebilirsiniz.
  - Docker volume'ü olan `mattermost-data:/mattermost/data`, Mattermost konteyneri içindeki `/mattermost/data` klasörüne kalıcı veri depolamak için tanımlanmıştır.

- **ports**:
  - Mattermost'un web arayüzü için `8065` numaralı port üzerinden erişilebilir hale getirilir.

- **depends_on**:
  - PostgreSQL (`postgres`) servisinin başlamasını bekler.
  - MinIO (`minio`) servisinin başlamasını bekler.
  - OnlyOffice (`onlyoffice`) servisinin başlamasını bekler.

- **networks**:
  - `mattermost-network` adında bir Docker ağına bağlanır. Bu ağ, Mattermost ile diğer servisler (PostgreSQL, MinIO) arasında iletişimi sağlar.


```yaml
minio:
  image: minio/minio:latest
  environment:
    MINIO_ROOT_USER: minioaccesskey
    MINIO_ROOT_PASSWORD: miniosecretkey
    MINIO_DEFAULT_BUCKETS: mattermost
  ports:
    - "9000:9000"
    - "9001:9001"
  volumes:
    - minio-data:/data
  command: server /data --console-address ":9001"
  networks:
    - mattermost-network

´´´
MinIO Servisi Açıklaması:
- **image**: MinIO'un Docker Hub'dan (`minio` deposundan) en son sürümünü (`latest` etiketiyle) kullanır. MinIO, ölçeklenebilir nesne depolama sunucusudur ve bu yapılandırmada Amazon S3 uyumlu olarak kullanılmaktadır.

- **environment**:
  - `MINIO_ROOT_USER`: MinIO yönetici kullanıcısının adı, `minioaccesskey` olarak belirlenmiştir.
  - `MINIO_ROOT_PASSWORD`: MinIO yönetici kullanıcısının şifresi, `miniosecretkey` olarak belirlenmiştir.
  - `MINIO_DEFAULT_BUCKETS`: MinIO üzerinde varsayılan olarak oluşturulacak kova (bucket) adı, `mattermost` olarak belirlenmiştir.

- **ports**:
  - `"9000:9000"`: MinIO'nun standart HTTP API portu, yani 9000 numaralı port üzerinden erişilebilir hale getirilmiştir.
  - `"9001:9001"`: MinIO konsol arayüzü için ek bir port tanımlanmıştır.

- **volumes**:
  - `minio-data:/data`: MinIO konteyneri içindeki `/data` dizinine kalıcı veri depolamak için Docker volume'ü tanımlar. Bu, MinIO'nun verileri saklamak için kullanacağı kalıcı depolama alanını sağlar.

- **command**:
  - `server /data --console-address ":9001"`: MinIO'nun `/data` dizininde sunucu olarak başlatılmasını ve konsol arayüzünün `:9001` portundan erişilebilir olmasını sağlar.

- **networks**:
  - `mattermost-network`: `mattermost-network` adında bir Docker ağına bağlanır. Bu ağ, MinIO'nun Mattermost ve diğer servislerle iletişim kurmasını sağlar.

```yaml
onlyoffice:
  image: onlyoffice/documentserver:7.1
  environment:
    JWT_ENABLED: "true"
    JWT_SECRET: myjwtsecret
    USE_UNAUTHORIZED_STORAGE: true
    JWT_IN_BODY: true
    JWT_HEADER: AuthorizationJwt
  volumes:
    - onlyoffice-data:/var/www/onlyoffice/Data
  ports:
    - "80:80"
  networks:
    - mattermost-network
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost/healthcheck"]
    interval: 30s
    timeout: 10s
    retries: 3

OnlyOffice Servisi Açıklaması:
- **image**: OnlyOffice Document Server'ın Docker Hub'dan (`onlyoffice` deposundan) 7.1 sürümünü kullanır. OnlyOffice, belge işbirliği ve işlem çözümleri sunan açık kaynaklı bir ofis yazılımı platformudur.

- **environment**:
  - `JWT_ENABLED`: JWT (JSON Web Token) kimlik doğrulamasının etkin olup olmadığını belirtir (`true` olarak ayarlanmıştır).
  - `JWT_SECRET`: JWT için kullanılacak gizli anahtar, `myjwtsecret` olarak belirlenmiştir.
  - `USE_UNAUTHORIZED_STORAGE`: Yetkisiz depolama kullanımını belirtir (`true` olarak ayarlanmıştır).
  - `JWT_IN_BODY`: JWT'nin HTTP istek gövdelerinde iletilip iletilmeyeceğini belirtir (`true` olarak ayarlanmıştır).
  - `JWT_HEADER`: JWT'nin HTTP başlığında iletilip iletilmeyeceğini belirtir (`AuthorizationJwt` olarak ayarlanmıştır).

- **volumes**:
  - `onlyoffice-data:/var/www/onlyoffice/Data`: OnlyOffice konteyneri içindeki `/var/www/onlyoffice/Data` dizinine kalıcı veri depolamak için Docker volume'ü tanımlar. Bu, OnlyOffice'un veri saklaması için kullanılacak kalıcı depolama alanını sağlar.

- **ports**:
  - `"80:80"`: OnlyOffice'un HTTP trafiği için 80 numaralı port üzerinden erişilebilir hale getirilmiştir.

- **networks**:
  - `mattermost-network`: `mattermost-network` adında bir Docker ağına bağlanır. Bu ağ, OnlyOffice'un Mattermost ve diğer servislerle iletişim kurmasını sağlar.

- **healthcheck**:
  - `test: ["CMD", "curl", "-f", "http://localhost/healthcheck"]`: Sağlık kontrolü için kullanılacak komutu belirtir (localhost üzerinde HTTP sağlık kontrolü).
  - `interval: 30s`: Sağlık kontrolünün her 30 saniyede bir yapılmasını sağlar.
  - `timeout: 10s`: Sağlık kontrolü komutunun maksimum 10 saniye içinde tamamlanması gerektiğini belirtir.
  - `retries: 3`: Sağlık kontrolü komutunda maksimum 3 tekrar deneme yapılmasını sağlar.




## Volumes (Docker Volume'leri)

```yaml
volumes:
  postgres-data:
    PostgreSQL servisi için kullanılacak Docker volume'ü. PostgreSQL konteynerinin veritabanı verilerini saklamak için kullanılır.
  minio-data:
    MinIO servisi için kullanılacak Docker volume'ü. MinIO konteynerinin verilerini saklamak için kullanılır.
  mattermost-data:
    Mattermost servisi için kullanılacak Docker volume'ü. Mattermost konteynerinin veri dosyalarını saklamak için kullanılır.
  
## Networks (Ağ Tanımları)

```yaml
networks:
  mattermost-network:
    driver: bridge

mattermost-network: Docker Compose dosyasında tanımlanan ağlardan biridir. mattermost-network adında bir Docker bridge ağıdır.



Bu kılavuzda, Docker kullanarak Mattermost, MinIO, PostgreSQL ve OnlyOffice servislerini nasıl kuracağınızı adım adım açıklıyoruz.

## 1. Mattermost ve MinIO Hesap Oluşturma

- **http://localhost:8065/** Mattermost web sitesine gidin ve yeni bir hesap oluşturun.
- **http://localhost:9001/** MinIO web arayüzüne gidin ve yeni bir hesap oluşturun.

## 2. MinIO'da Bucket Oluşturma

MinIO'da `mattermost` adında bir bucket oluşturun.

## 3. PostgreSQL'de OnlyOffice İçin Yeni Database Oluşturma

PostgreSQL yönetim aracını kullanarak `onlyoffice` adında yeni bir veritabanı oluşturun.
CREATE DATABASE onlyoffice;
GRANT ALL PRIVILEGES ON DATABASE onlyoffice TO ooouser;


## 4. Mattermost Plugins Dosyası Kurulumu

Docker Compose dosyasında tanımladığınız `./plugins` konumuna Mattermost eklentilerini yükleyin.

## 5. Mattermost Konfigürasyonu

- Tarayıcınızdan Mattermost'a erişin (`localhost:8065` veya uygun IP/port).
- Yönetici olarak giriş yapın ve gerekli ayarları yapılandırın.

---

Bu adımları takip ederek Mattermost, MinIO, PostgreSQL ve OnlyOffice servislerini başarıyla kurabilir ve yapılandırabilirsiniz. Herhangi bir sorunuz veya ek yardıma ihtiyacınız olursa bana sorabilirsiniz.





