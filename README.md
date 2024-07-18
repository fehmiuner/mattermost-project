# mattermost-project
Docker-Compose-using-Mattermost (OnlyOffice-MinIO-PostreSQL)

Bu proje, Docker Compose kullanarak Mattermost, OnlyOffice, PostgreSQL ve MinIO servislerini entegre etmektedir.

## Gereksinimler

- Docker
- Docker Compose

## Kurulum

1. **Repository'i Klonlayın:**
    ```bash
    git clone https://github.com/fehmiuner/mattermost-project.git
    cd mattermost-project
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
"Kodu kopyala"
CREATE DATABASE onlyoffice;
GRANT ALL PRIVILEGES ON DATABASE onlyoffice TO ooouser;

4. Mattermost Plugins Dosyası Kurulumu

Docker Compose dosyasında tanımladığınız ./plugins konumuna Mattermost eklentilerini yükleyin.

5. Mattermost Konfigürasyonu

Tarayıcınızdan Mattermost'a erişin (localhost:8065).
Yönetici olarak giriş yapın ve gerekli ayarları yapılandırın.


Bu kılavuzu takip ederek Mattermost, MinIO, PostgreSQL ve OnlyOffice servislerini başarıyla kurabilir ve yapılandırabilirsiniz. Herhangi bir sorunuz olursa bana sormaktan çekinmeyin.






