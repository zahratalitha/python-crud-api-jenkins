# Python CRUD API

Project praktikum CI/CD menggunakan Jenkins, GitHub, Docker, dan ngrok.

## Deskripsi
Aplikasi REST API sederhana untuk menyimpan, melihat, dan menghapus pesan.

## Endpoint API

| Method | Endpoint | Fungsi |
|---|---|---|
| GET | `/health` | Memeriksa kondisi API |
| GET | `/messages` | Melihat semua pesan |
| POST | `/messages` | Menambahkan pesan |
| DELETE | `/messages?index=0` | Menghapus pesan berdasarkan index |

## Teknologi
- Python 3.11
- Docker
- Jenkins
- GitHub
- ngrok