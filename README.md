# ci-images

Image CI Webekspres. Tools dikunci di image, jadi runner (PC) cukup punya Docker.
Dipakai lewat `container:` di job GitHub Actions, dan nanti oleh runner ARC.

| Image | Tag | Isi |
|---|---|---|
| `ghcr.io/webekspres/ci-php` | `8.3`, `8.4` | PHP + ekstensi Laravel/Bagisto, composer, Node 22, Bun, mysql-client |
| `ghcr.io/webekspres/ci-node` | `22` | Node 22, Bun, git, openssl |
| `ghcr.io/webekspres/ci-python` | `3.12` | Python 3.12, uv |

Contoh:

```yaml
jobs:
  tests:
    runs-on: [self-hosted, heavy, docker]
    container: ghcr.io/webekspres/ci-php:8.3
    services:
      mysql:
        image: mysql:8.4
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
    env:
      DB_HOST: mysql   # nama service, bukan 127.0.0.1; tanpa mapping port host
    steps:
      - uses: actions/checkout@v4
      - run: composer install && php artisan test
```

Menambah stack baru: buat `images/<nama>/Dockerfile`, tambahkan baris di matrix `.github/workflows/build.yml`.
Image dibangun ulang otomatis tiap tanggal 1 untuk patch keamanan.
Repo ini publik: jangan pernah menaruh rahasia di Dockerfile.
