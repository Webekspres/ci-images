# ci-images

Image CI Webekspres. Semua tools build/test ada di image, jadi runner cukup menjalankan kontainer.
Dipakai lewat `container:` di job GitHub Actions; template workflow organisasi ada di
[Webekspres/.github](https://github.com/Webekspres/.github/tree/main/workflow-templates).

Repo ini **publik**: jangan pernah menaruh rahasia di Dockerfile atau workflow.

## Image dan isi

| Image | Tag bergerak | Isi |
|---|---|---|
| `ghcr.io/webekspres/ci-php` | `8.2`, `8.3`, `8.4` | PHP (Debian bookworm) + ekstensi bcmath, calendar, exif, gd, gmp, intl, mysqli, opcache, pcntl, pdo_mysql, pdo_pgsql, redis, soap, sockets, zip; composer 2; Node 22 + npm; Bun 1; klien MySQL, Postgres, Redis |
| `ghcr.io/webekspres/ci-node` | `20`, `22` | Node (Debian bookworm) + npm; Bun 1; git, openssl; klien MySQL, Postgres, Redis |
| `ghcr.io/webekspres/ci-python` | `3.12` | Python + uv/uvx; git; klien MySQL, Postgres, Redis |
| `ghcr.io/webekspres/ci-flutter` | `3.44.6` | Flutter 3.44.6, Android SDK (platform 37 + build-tools terbaru, cmdline-tools, NDK 28.2), JDK 17, Node 22 |

Klien database dipakai oleh action [`tunggu-db`](https://github.com/Webekspres/.github/tree/main/actions/tunggu-db)
(`mysqladmin`, `pg_isready`, `redis-cli`).

## Kapan dibangun ulang

- Otomatis tiap **tanggal 1** (patch keamanan base image dan versi patch terbaru PHP/Node/Python).
- Setiap perubahan di `images/**` atau workflow build yang masuk ke `main`.
- Manual: Actions → **Build images** → Run workflow.

## Memilih tag: bergerak, beku, atau digest

Setiap build menerbitkan dua tag, dan daftarnya beserta digest dicatat di
[Releases](https://github.com/Webekspres/ci-images/releases) (`build-YYYY.MM.DD`):

| Cara | Contoh | Kapan dipakai |
|---|---|---|
| Tag bergerak | `ci-php:8.4` | bawaan untuk CI: selalu mendapat patch keamanan bulanan |
| Tag beku | `ci-php:8.4-2026.10.09` | build harus sama persis untuk sementara (mis. menyelidiki tes yang tiba-tiba gagal setelah rebuild) |
| Digest | `ci-php@sha256:…` | reproduksibel penuh; salin dari rilis |

Tag beku tidak ditimpa build berikutnya (kecuali build ulang pada hari yang sama). Kembali ke tag
bergerak secepatnya agar tetap mendapat patch keamanan.

## Memilih versi

Tag **mengikuti versi proyek**, bukan sebaliknya: pakai tag yang sama dengan
`composer.json` (`require.php`), `.nvmrc`/`engines.node`, `.python-version`/`requires-python`,
versi Flutter proyek, dan Dockerfile produksi. Proyek tidak dipaksa naik versi demi CI.

## Menambah versi atau stack

- **Versi baru** dari image yang sudah ada (mis. PHP 8.5, Node 24): tambah satu baris di matrix
  `.github/workflows/build.yml`, misalnya `{ image: ci-node, tag: '24', args: 'NODE_MAJOR=24' }`.
- **Stack baru**: buat `images/<nama>/Dockerfile` (base Debian bookworm, `git config --system --add safe.directory '*'`,
  `WORKDIR /workspace`), tambahkan ke matrix, dan tambahkan uji cepat di langkah "Uji cepat".
- Buka PR ke `main`; build berjalan setelah merge dan rilis digest dibuat otomatis.
- Image baru di GHCR bersifat privat secara default: buat publik di setelan package agar bisa ditarik runner.

## Contoh

```yaml
jobs:
  test:
    runs-on: arc-ci
    container:
      image: ghcr.io/webekspres/ci-php:8.4
    services:
      mysql:
        image: mysql:8.4
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
    env:
      DB_HOST: mysql   # nama service, bukan 127.0.0.1
    steps:
      - uses: actions/checkout@v4
      - uses: Webekspres/.github/actions/tunggu-db@main
        with: { type: mysql, password: root }
      - run: composer install && php artisan test
```
