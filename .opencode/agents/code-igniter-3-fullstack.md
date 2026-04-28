# 🚀 Backend REST API Agent (MVC Only - Monolith)

## 📌 Stack
- CodeIgniter 3 + chriskacerguis\\RestServer\\RestController
- JWT (firebase/php-jwt)
- MySQL / PostgreSQL
- .env configuration
- Bootstrap / Tailwind (optional frontend)

---

## 🧠 MVC Architecture

### 1. Controller
- Handle request & response
- Validasi input
- Panggil model

### 2. Model
- Query database
- CRUD operation

### 3. View
- Tampilan (HTML / Bootstrap / Tailwind)

---

## 📂 Struktur Folder

application/
├── controllers/
│   ├── api/
│   │   └── Auth.php
│   └── web/
├── models/
│   └── User_model.php
├── views/
│   ├── layouts/
│   └── pages/
├── config/
├── helpers/
├── libraries/

---

## 🔄 Flow

Client → Controller → Model → Database  
Database → Model → Controller → Response/View

---

## 🔐 JWT Flow

1. Login → Controller
2. Validasi user → Model
3. Generate JWT → Library
4. Return token

Request berikutnya:
- Header: Authorization Bearer Token
- Controller validasi JWT

---

## 🎨 Frontend (Optional)

### Bootstrap
- Admin panel
- Form, table, modal

### Tailwind
- Custom UI
- Responsive styling

---

## ⚙️ Environment (.env)

APP_ENV=development
DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=ap2t
DB_USERNAME=postgres
DB_PASSWORD=postgres
JWT_SECRET=supersecretkey
JWT_EXPIRE=3600

---

## 🔐 Security

- Gunakan password_hash & password_verify
- Jangan gunakan md5
- Gunakan HTTPS
- Validasi semua input

---

## 🚀 Kesimpulan

- MVC sederhana & cepat
- Cocok untuk monolith
- Mudah maintain
- Bisa di-upgrade ke clean architecture jika dibutuhkan
