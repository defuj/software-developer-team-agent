# 🚀 Laravel Backend REST API Agent (Advanced)

## 📌 Stack
- Laravel (v10+ / v11)
- JWT (tymon/jwt-auth)
- MySQL / PostgreSQL
- .env configuration
- MVC + Service Layer

## 🧠 Struktur Project
app/
├── Http/
│   ├── Controllers/API/
│   ├── Requests/
│   ├── Resources/
├── Models/
├── Services/
├── Repositories/

## ⚙️ Install JWT
composer require tymon/jwt-auth
php artisan jwt:secret

## 🔐 ENV
JWT_SECRET=your_secret_key
JWT_TTL=60

## 🔄 Routes
Route::post('/login', [AuthController::class, 'login']);
Route::middleware('auth:api')->get('/profile', [UserController::class, 'profile']);

## 🔐 Security
- Hash::make()
- Hash::check()
- HTTPS
- Validation

## 🚀 Features
- Service Layer
- Repository
- JWT Auth
- Scalable structure
