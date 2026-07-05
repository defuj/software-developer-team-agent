---
name: flutter-dio-multi-service
description: >-
  Multi-microservice Dio networking patterns for Flutter: cookie-based session auth with cookie_jar,
  shared PersistCookieJar across multiple Dio instances, Bearer token fallback, AuthInterceptor with
  401 detection on both onResponse and onError, static cross-cutting logout callback, ApiService thin
  typed wrapper with response envelope validation, dual-format envelope parser (status/success),
  cursor-based pagination, and dual-layer error hierarchy (Exception + Failure). Designed for production
  Flutter apps connecting to multiple backend microservices.
license: MIT
metadata:
  author: opencode-agent-kit
  version: '1.0.0'
  target_agent: flutter-developer
  stack:
    - Dart 3
    - Flutter SDK
    - Dio
    - cookie_jar
    - dio_cookie_manager
    - flutter_secure_storage
    - dartz
    - path_provider
---

# Flutter: Multi-Service Dio & Cookie Auth

**Target Agent:** @flutter-developer  
**Stack:** Dart 3 · Dio · cookie_jar · dio_cookie_manager · flutter_secure_storage · dartz · path_provider

Production patterns for connecting Flutter apps to multiple backend microservices with cookie-based authentication, typed error handling, response envelope parsing, and cursor pagination.

---

## Table of Contents

1. [Multi-Service Dio Setup](#1-multi-service-dio-setup)
2. [Cookie-Based Auth Interceptor](#2-cookie-based-auth-interceptor)
3. [Thin Typed ApiService](#3-thin-typed-apiservice)
4. [Dual-Format Response Envelope](#4-dual-format-response-envelope)
5. [Cursor Pagination Model](#5-cursor-pagination-model)
6. [Dual-Layer Error Hierarchy](#6-dual-layer-error-hierarchy)
7. [Service-Specific Environment Config](#7-service-specific-environment-config)
8. [Auth Bridge: Riverpod + GoRouter](#8-auth-bridge-riverpod--gorouter)
9. [Testing Multi-Service Dio](#9-testing-multi-service-dio)
10. [Complete Data Flow: Auth Example](#10-complete-data-flow-auth-example)

---

## 1. Multi-Service Dio Setup

### Problem

Modern Flutter apps often talk to **multiple backend microservices** — auth, HR, reporting, config, product — each with its own base URL but sharing the same auth context (cookies).

### Solution

Create a `DioClient` singleton that initializes one `Dio` instance per service, all sharing a single `PersistCookieJar`. This keeps cookies in sync across all services.

```dart
import 'package:cookie_jar/cookie_jar.dart';
import 'package:dio/dio.dart';
import 'package:dio_cookie_manager/dio_cookie_manager.dart';
import 'package:path_provider/path_provider.dart';

class DioClient {
  DioClient._();

  static PersistCookieJar? _cookieJar;
  static Dio? _dashboardDio;
  static Dio? _hrDio;
  static Dio? _reportDio;
  static Dio? _configDio;
  static Dio? _productDio;

  /// Must be called once before accessing any Dio instance.
  /// In main.dart: await DioClient.init();
  static Future<void> init() async {
    final dir = await getApplicationDocumentsDirectory();
    final cookiePath = '${dir.path}/app_cookies';
    _cookieJar = PersistCookieJar(
      ignoreExpires: true, // let the server decide expiry via 401
      storage: FileStorage(cookiePath),
    );
    _dashboardDio = _createDio(
      baseUrl: ApiConstants.coreBaseUrl,
      cookieJar: _cookieJar!,
    );
    _hrDio = _createDio(
      baseUrl: ApiConstants.hrBaseUrl,
      cookieJar: _cookieJar!,
    );
    _reportDio = _createDio(
      baseUrl: ApiConstants.reportBaseUrl,
      cookieJar: _cookieJar!,
    );
    _configDio = _createDio(
      baseUrl: ApiConstants.configBaseUrl,
      cookieJar: _cookieJar!,
    );
    _productDio = _createDio(
      baseUrl: ApiConstants.productBaseUrl,
      cookieJar: _cookieJar!,
    );
  }

  static PersistCookieJar get cookieJar {
    assert(_cookieJar != null, 'DioClient.init() must be called first');
    return _cookieJar!;
  }

  static Dio get dashboardDio {
    assert(_dashboardDio != null, 'DioClient.init() must be called first');
    return _dashboardDio!;
  }

  static Dio get hrDio {
    assert(_hrDio != null, 'DioClient.init() must be called first');
    return _hrDio!;
  }

  static Dio get reportDio {
    assert(_reportDio != null, 'DioClient.init() must be called first');
    return _reportDio!;
  }

  static Dio get configDio {
    assert(_configDio != null, 'DioClient.init() must be called first');
    return _configDio!;
  }

  static Dio get productDio {
    assert(_productDio != null, 'DioClient.init() must be called first');
    return _productDio!;
  }

  /// Reset all instances (e.g. on logout to clear cookie state).
  static void reset() {
    _dashboardDio = null;
    _hrDio = null;
    _reportDio = null;
    _configDio = null;
    _productDio = null;
    _cookieJar?.deleteAll();
    _cookieJar = null;
  }

  static Dio _createDio({
    required String baseUrl,
    required PersistCookieJar cookieJar,
  }) {
    final dio = Dio(
      BaseOptions(
        baseUrl: baseUrl,
        connectTimeout: const Duration(seconds: 15),
        receiveTimeout: const Duration(seconds: 15),
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
        },
        // Don't throw on error status codes — handle them in repository layer
        validateStatus: (_) => true,
      ),
    );

    dio.interceptors.addAll([
      CookieManager(cookieJar),
      AuthInterceptor(cookieJar),
      LogInterceptor(requestBody: true, responseBody: true),
    ]);

    return dio;
  }
}
```

### Key Design Decisions

| Decision                                 | Rationale                                                                                                             |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `validateStatus: (_) => true`            | Prevents Dio from throwing on HTTP error codes. All error handling is centralized in the repository/datasource layer. |
| `CookieManager` before `AuthInterceptor` | Ensures cookies are loaded/stored before the auth interceptor checks them.                                            |
| `ignoreExpires: true`                    | Server controls session expiry via 401 responses, not cookie expiry dates.                                            |
| `reset()` method                         | Called on logout to clear all cookies from disk and force all services to re-authenticate.                            |

### When to Add More Services

If your app grows beyond 5 services, extract a factory pattern:

```dart
// ponytail: if services grow past 5, refactor to a registry
class DioRegistry {
  final Map<String, Dio> _instances = {};

  Dio get(String name) {
    assert(_instances.containsKey(name), 'Dio "$name" not registered');
    return _instances[name]!;
  }

  void register(String name, String baseUrl, PersistCookieJar cookieJar) {
    _instances[name] = _createDio(baseUrl: baseUrl, cookieJar: cookieJar);
  }
}
```

---

## 2. Cookie-Based Auth Interceptor

### Problem

Bearer tokens are the default auth pattern but many enterprise backends use **cookie-based sessions**. Cookies are set by the login response and sent automatically on subsequent requests.

### Cookie Interceptor

```dart
import 'package:cookie_jar/cookie_jar.dart';
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class AuthInterceptor extends Interceptor {
  AuthInterceptor(this._cookieJar);
  final PersistCookieJar _cookieJar;

  /// Optional callback — wire to auth provider navigation from outside.
  /// Uses a static callback to bridge the interceptor (global/stateless) to
  /// Riverpod (scoped state) without making the interceptor depend on Riverpod.
  static void Function()? onUnauthorized;

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    _ensureBearerFallback(options);
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // Catch 401 even when validateStatus is true (which prevents onError
    // from firing for HTTP status codes).
    if (response.statusCode == 401) {
      _onUnauthorized();
    }
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (err.response?.statusCode == 401) {
      _onUnauthorized();
    }
    handler.next(err);
  }

  void _onUnauthorized() {
    _cookieJar.deleteAll();
    onUnauthorized?.call();
  }
}
```

### Bearer Fallback for Cross-Origin Services

Some services (e.g., HR) may not share cookies due to cross-origin constraints. Add a Bearer token fallback:

```dart
  Future<void> _ensureBearerFallback(RequestOptions options) async {
    if (!options.baseUrl.contains('hr')) return;

    // Check if we already have cookies for this request
    final cookies = await _cookieJar.loadForRequest(options.uri);
    if (cookies.isNotEmpty) return;

    // Fallback: try Bearer token from secure storage
    try {
      const storage = FlutterSecureStorage();
      final token = await storage.read(key: 'access_token');
      if (token != null && token.isNotEmpty) {
        options.headers['Authorization'] = 'Bearer $token';
      }
    } catch (_) {
      // Secure storage not available (web) — skip Bearer fallback
    }
  }
```

### Static OnUnauthorized Bridge

The static `onUnauthorized` callback is wired once at app startup to trigger logout:

```dart
// In app_router.dart:
AuthInterceptor.onUnauthorized = () {
  try {
    ref.read(authProvider.notifier).setUnauthenticated();
  } catch (_) {
    // Ignore — called after provider disposal
  }
};
```

### Auth Interceptor Flow

```
Request → [CookieManager loads cookies] → [AuthInterceptor checks Bearer fallback]
  ↘ Response ← [Dio executes request]
     ↘ If 401 → [AuthInterceptor._onUnauthorized]
       → [cookieJar.deleteAll()]
       → [onUnauthorized callback]
       → [authProvider.setUnauthenticated()]
       → [GoRouter redirects to /login]
```

### Warning: Avoid Infinite Loops

When the interceptor catches a 401, **do not call `logout()`** (which makes a `POST /auth/logout` call that would also 401 since cookies are already cleared). Instead call `setUnauthenticated()` — a local state-only method that does not make API calls:

```dart
void setUnauthenticated() {
  ref.read(storeSelectionProvider.notifier).clear();
  clearBusinessAnalysisCache();
  state = const AuthInitial();
}
```

---

## 3. Thin Typed ApiService

### Problem

Raw Dio calls in every data source mean repeated boilerplate for response validation, error mapping, and type casting.

### Solution

Create a thin wrapper that centralizes error mapping and response envelope validation:

```dart
import 'package:dio/dio.dart';

class ApiService {
  ApiService(this._dio);
  final Dio _dio;

  Future<Map<String, dynamic>> get(
    String path, {
    Map<String, dynamic>? queryParameters,
  }) async {
    try {
      final response = await _dio.get<Map<String, dynamic>>(
        path,
        queryParameters: queryParameters,
      );
      return _validate(response.data);
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<List<dynamic>> getList(
    String path, {
    Map<String, dynamic>? queryParameters,
  }) async {
    try {
      final response = await _dio.get<List<dynamic>>(
        path,
        queryParameters: queryParameters,
      );
      return response.data ?? [];
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<Map<String, dynamic>> post(String path, {Object? data}) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(path, data: data);
      return _validate(response.data);
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<Map<String, dynamic>> put(String path, {Object? data}) async {
    try {
      final response = await _dio.put<Map<String, dynamic>>(path, data: data);
      return _validate(response.data);
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<Map<String, dynamic>> delete(String path) async {
    try {
      final response = await _dio.delete<Map<String, dynamic>>(path);
      return _validate(response.data);
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Map<String, dynamic> _validate(Map<String, dynamic>? data) {
    if (data == null) {
      throw ServerException('Empty response', statusCode: 500);
    }
    return ApiResponseHelper.requireSuccess(data);
  }

  AppException _mapError(DioException e) {
    if (e.type == DioExceptionType.connectionTimeout ||
        e.type == DioExceptionType.receiveTimeout) {
      return NetworkException('Connection timeout');
    }
    if (e.type == DioExceptionType.connectionError) {
      return NetworkException('No internet connection');
    }
    final statusCode = e.response?.statusCode;
    final body = e.response?.data as Map<String, dynamic>?;
    final message = body?['message'] as String? ?? e.message ?? 'Unknown error';
    if (statusCode == 401) {
      return UnauthorizedException(message);
    }
    if (statusCode == 404) {
      return ServerException(message, statusCode: 404);
    }
    if (statusCode == 422) {
      return ValidationException(
        message,
        fieldErrors: body?['errors'] as Map<String, List<String>>?,
      );
    }
    return ServerException(message, statusCode: statusCode ?? 500);
  }
}
```

### Usage in Data Sources

```dart
class ProductDataSource {
  ProductDataSource({Dio? dio}) : _api = ApiService(dio ?? DioClient.productDio);
  final ApiService _api;

  Future<Map<String, dynamic>> getProducts({String? cursor}) async {
    return _api.get('/products', queryParameters: {
      if (cursor != null) 'cursor': cursor,
    });
  }

  Future<Map<String, dynamic>> createProduct(Map<String, dynamic> data) async {
    return _api.post('/products', data: data);
  }
}
```

---

## 4. Dual-Format Response Envelope

### Problem

Different backend services use different response envelope formats. The auth service returns `{ status: true, message: "...", data: {...} }` while another service uses `{ success: true, message: "...", data: {...} }`.

### Solution

A helper that checks both formats:

```dart
/// Helper to handle two response envelope formats:
///   Auth / HR:  { status, message, data }
///   Others: { success, message, data }
abstract class ApiResponseHelper {
  ApiResponseHelper._();

  static bool isSuccess(Map<String, dynamic> json) =>
      json['status'] == true || json['success'] == true;

  static String getMessage(Map<String, dynamic> json) =>
      json['message'] as String? ?? '';

  static Map<String, dynamic>? getData(Map<String, dynamic> json) =>
      json['data'] as Map<String, dynamic>?;

  static List<dynamic>? getDataList(Map<String, dynamic> json) =>
      json['data'] as List<dynamic>?;

  /// Throws if the response is not a success.
  static Map<String, dynamic> requireSuccess(Map<String, dynamic> json) {
    if (!isSuccess(json)) {
      throw ApiResponseException(getMessage(json));
    }
    return json;
  }
}

class ApiResponseException implements Exception {
  ApiResponseException(this.message);
  final String message;

  @override
  String toString() => 'ApiResponseException: $message';
}
```

### Usage in Repository Implementation

```dart
class AuthRepositoryImpl implements AuthRepository {
  // ...
  Future<Either<Failure, AuthSession>> login(
    String username,
    String password,
  ) async {
    try {
      final json = await _dataSource.login(username, password);
      final data =
          ApiResponseHelper.requireSuccess(json)['data']
              as Map<String, dynamic>;
      return Right(_parseSession(data));
    } on AppException catch (e) {
      return Left(_mapFailure(e));
    } on ApiResponseException catch (e) {
      return Left(ServerFailure(message: e.message));
    } on DioException catch (e) {
      return Left(_mapDioFailure(e));
    }
  }
}
```

---

## 5. Cursor Pagination Model

### Problem

Many APIs return paginated results using cursor-based pagination. There is no standard model in the Flutter ecosystem.

### Solution

A generic `CursorPage<T>` model:

```dart
class CursorPage<T> {
  const CursorPage({
    required this.items,
    required this.hasMore,
    this.nextCursor,
  });

  factory CursorPage.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromItem,
  ) {
    final itemsJson = json['items'] as List<dynamic>? ?? [];
    return CursorPage(
      items: itemsJson
          .map((e) => fromItem(e as Map<String, dynamic>))
          .toList(growable: false),
      nextCursor: json['next_cursor'] as String?,
      hasMore: json['has_more'] as bool? ?? false,
    );
  }

  final List<T> items;
  final String? nextCursor;
  final bool hasMore;
}
```

### Usage with Riverpod Notifier

```dart
class ProductListState {
  const ProductListState({
    this.items = const [],
    this.nextCursor,
    this.hasMore = true,
    this.isLoading = false,
    this.isLoadingMore = false,
    this.error,
  });

  final List<ProductItem> items;
  final String? nextCursor;
  final bool hasMore;
  final bool isLoading;
  final bool isLoadingMore;
  final String? error;

  ProductListState copyWith({
    List<ProductItem>? items,
    String? nextCursor,
    bool? hasMore,
    bool? isLoading,
    bool? isLoadingMore,
    String? error,
  }) {
    return ProductListState(
      items: items ?? this.items,
      nextCursor: nextCursor ?? this.nextCursor,
      hasMore: hasMore ?? this.hasMore,
      isLoading: isLoading ?? this.isLoading,
      isLoadingMore: isLoadingMore ?? this.isLoadingMore,
      error: error ?? this.error,
    );
  }
}

class ProductListNotifier extends Notifier<ProductListState> {
  @override
  ProductListState build() => const ProductListState();

  Future<void> loadProducts() async {
    state = state.copyWith(isLoading: true, error: null);
    // ... fetch and update state
  }

  Future<void> loadMore() async {
    if (!state.hasMore || state.isLoadingMore) return;
    state = state.copyWith(isLoadingMore: true);
    // ... fetch next page with state.nextCursor
  }
}
```

---

## 6. Dual-Layer Error Hierarchy

### Problem

Clean Architecture uses `Either<Failure, T>` in the domain layer, but low-level code (Dio, file I/O) typically uses `throw`. A dual-layer model bridges the two worlds cleanly.

### Layer 1: Exceptions (for imperative/thrown errors)

```dart
class AppException implements Exception {
  AppException(this.message, {this.statusCode});
  final String message;
  final int? statusCode;

  @override
  String toString() => 'AppException: $message (status: $statusCode)';
}

class ServerException extends AppException {
  ServerException(super.message, {super.statusCode});
}

class NetworkException extends AppException {
  NetworkException(super.message);
}

class UnauthorizedException extends AppException {
  UnauthorizedException([super.message = 'Unauthorized'])
    : super(statusCode: 401);
}

class CacheException extends AppException {
  CacheException(super.message);
}

class ValidationException extends AppException {
  ValidationException(super.message, {this.fieldErrors})
    : super(statusCode: 422);
  final Map<String, List<String>>? fieldErrors;
}
```

### Layer 2: Failures (for Either/functional errors)

```dart
sealed class Failure {
  const Failure({required this.message, this.statusCode});
  final String message;
  final int? statusCode;
  String get displayMessage => message;
}

class ServerFailure extends Failure {
  const ServerFailure({required super.message, super.statusCode});
}

class NetworkFailure extends Failure {
  const NetworkFailure({required super.message});
}

class UnauthorizedFailure extends Failure {
  const UnauthorizedFailure({required super.message});
}

class NotFoundFailure extends Failure {
  const NotFoundFailure({required super.message});
}

class ValidationFailure extends Failure {
  const ValidationFailure({required super.message, this.fieldErrors});
  final Map<String, List<String>>? fieldErrors;
}

class CacheFailure extends Failure {
  const CacheFailure({required super.message});
}

class UnknownFailure extends Failure {
  const UnknownFailure({super.message = 'Something went wrong'});
}
```

### Mapping Exceptions to Failures in Repository

```dart
Failure _mapFailure(AppException e) {
  if (e is UnauthorizedException) {
    return UnauthorizedFailure(message: e.message);
  }
  return ServerFailure(message: e.message, statusCode: e.statusCode);
}

Failure _mapDioFailure(DioException e) {
  if (e.type == DioExceptionType.badResponse &&
      e.response?.statusCode == 401) {
    return const UnauthorizedFailure(message: 'Access token missing');
  }
  if (e.type == DioExceptionType.connectionError ||
      e.type == DioExceptionType.connectionTimeout) {
    return const NetworkFailure(message: 'No internet connection');
  }
  return ServerFailure(message: e.message ?? 'Something went wrong');
}
```

---

## 7. Service-Specific Environment Config

### Problem

Each microservice has a different base URL, and these differ between development, staging, and production.

### Solution

Use `--dart-define-from-file` for compile-time configuration:

```dart
class ApiConstants {
  ApiConstants._();

  static const String coreBaseUrl = String.fromEnvironment(
    'CORE_BASE_URL',
    defaultValue: 'https://api.example.com/core/v1',
  );

  static const String hrBaseUrl = String.fromEnvironment(
    'HR_BASE_URL',
    defaultValue: 'https://api.example.com/hr/v1',
  );

  static const String reportBaseUrl = String.fromEnvironment(
    'REPORT_BASE_URL',
    defaultValue: 'https://api.example.com/report/v1',
  );

  static const String configBaseUrl = String.fromEnvironment(
    'CONFIG_BASE_URL',
    defaultValue: 'https://api.example.com/config/v1',
  );

  static const String productBaseUrl = String.fromEnvironment(
    'PRODUCT_BASE_URL',
    defaultValue: 'https://api.example.com/product/v1',
  );

  static const Duration requestTimeout = Duration(seconds: 15);
  static const String cookieStorageKey = 'app_cookies';
}
```

**env.properties (gitignored):**

```properties
CORE_BASE_URL=https://api.example.com/core/v1
HR_BASE_URL=https://api.example.com/hr/v1
REPORT_BASE_URL=https://api.example.com/report/v1
CONFIG_BASE_URL=https://api.example.com/config/v1
PRODUCT_BASE_URL=https://api.example.com/product/v1
```

**Run commands:**

```bash
flutter run --dart-define-from-file=env.properties
flutter build apk --dart-define-from-file=env.properties
flutter build ios --dart-define-from-file=env.properties
```

**env.properties.example (committed):**

```properties
CORE_BASE_URL=
HR_BASE_URL=
REPORT_BASE_URL=
CONFIG_BASE_URL=
PRODUCT_BASE_URL=
```

---

## 8. Auth Bridge: Riverpod + GoRouter

### Problem

GoRouter's `refreshListenable` expects a `Listenable`, but Riverpod's reactive state is provider-based. A bridge is needed so auth state changes trigger GoRouter redirect re-evaluation.

### Solution

A `ChangeNotifier` that listens to Riverpod auth state and notifies GoRouter:

```dart
class AuthRouterBridge extends ChangeNotifier {
  AuthRouterBridge(Ref ref) {
    ref.listen<AuthState>(authProvider, (_, _) {
      // Defer notifyListeners to the next microtask to avoid racing with
      // other ref.listen callbacks (e.g. login screen) that may navigate
      // during the same Riverpod notification cycle.
      Future.microtask(() {
        notifyListeners();
      });
    });
  }
}

final routerProvider = Provider<GoRouter>((ref) {
  final bridge = AuthRouterBridge(ref);
  ref.onDispose(bridge.dispose);

  // Wire the static interceptor so a 401 from any Dio call triggers logout.
  AuthInterceptor.onUnauthorized = () {
    try {
      ref.read(authProvider.notifier).setUnauthenticated();
    } catch (_) {
      // Ignore — called after provider disposal
    }
  };

  return GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/splash',
    refreshListenable: bridge,
    redirect: _buildRedirect(ref),
    routes: [
      GoRoute(
        path: '/splash',
        pageBuilder: (_, _) =>
            PageTransition.fadeScalePage(child: const SplashScreen()),
      ),
      GoRoute(
        path: '/login',
        pageBuilder: (_, _) =>
            PageTransition.fadeScalePage(child: const LoginScreen()),
      ),
      // ... authenticated routes with parentNavigatorKey for modal routes
    ],
  );
});

GoRouterRedirect _buildRedirect(Ref ref) =>
    (BuildContext context, GoRouterState state) {
      final authState = ref.read(authProvider);
      final isLoggedIn = authState is AuthAuthenticated;
      final location = state.matchedLocation;
      final isPublicRoute = location == '/splash' || location == '/login';

      // Stay on splash while auth check is running
      if (authState is AuthChecking && location == '/splash') return null;
      // No session — redirect from splash to login
      if (authState is AuthInitial && location == '/splash') return '/login';
      // Not logged in and not on a public page — redirect to login
      if (!isLoggedIn && !isPublicRoute) return '/login';
      // Already logged in and on a public page — go to dashboard
      if (isLoggedIn && isPublicRoute) return '/dashboard/sales';

      return null;
    };
```

### Auth State Machine

```
               ┌──────────┐
               │ Checking │  ← App starts here (splash screen)
               └────┬─────┘
                    │
          ┌─────────┴──────────┐
          ▼                    ▼
    ┌─────────┐          ┌──────────┐
    │ Initial │          │Authentic.│  ← GoRouter redirects to /dashboard
    └────┬────┘          └──────────┘
         │                      │
         ▼                      ▼ (401 from any API / logout)
    ┌─────────┐                 │
    │  Login  │                 │
    │ Screen  │                 │
    └────┬────┘                 │
         │ (successful login)   │
         └──────────────────────┘
```

---

## 9. Testing Multi-Service Dio

### Dependencies

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
```

### Test Pattern: Data Source

```dart
class MockDio extends Mock implements Dio {}

void main() {
  late MockDio mockDio;
  late AuthDataSource dataSource;

  setUp(() {
    mockDio = MockDio();
    dataSource = AuthDataSource(dio: mockDio);
  });

  group('login', () {
    test('returns raw json on success', () async {
      when(() => mockDio.post(
        any(),
        data: any(named: 'data'),
      )).thenAnswer((_) async => Response(
        requestOptions: RequestOptions(path: ''),
        statusCode: 200,
        data: {'status': true, 'data': {'token': 'abc'}},
      ));

      final result = await dataSource.login('admin', 'pass123');

      expect(result, {'status': true, 'data': {'token': 'abc'}});
      verify(() => mockDio.post(
        any(),
        data: {'username': 'admin', 'password': 'pass123', 'is_mobile': true},
      )).called(1);
    });
  });
}
```

### Test Pattern: Repository

```dart
class MockAuthDataSource extends Mock implements AuthDataSource {}

void main() {
  late MockAuthDataSource mockDataSource;
  late AuthRepositoryImpl repository;

  setUp(() {
    mockDataSource = MockAuthDataSource();
    repository = AuthRepositoryImpl(mockDataSource);
  });

  group('login', () {
    test('returns AuthSession on success', () async {
      when(() => mockDataSource.login(any(), any())).thenAnswer(
        (_) async => {'status': true, 'data': {/* ... */}},
      );

      final result = await repository.login('admin', 'pass123');

      expect(result.isRight(), true);
    });

    test('returns UnauthorizedFailure on 401', () async {
      when(() => mockDataSource.login(any(), any())).thenThrow(
        UnauthorizedException('Invalid credentials'),
      );

      final result = await repository.login('admin', 'pass123');

      expect(result.isLeft(), true);
      result.fold(
        (failure) => expect(failure, isA<UnauthorizedFailure>()),
        (_) => fail('Expected failure'),
      );
    });
  });
}
```

---

## 10. Complete Data Flow: Auth Example

```
┌─────────────────────────────────────────────────────────────┐
│  main.dart                                                  │
│  await DioClient.init();                                     │
│  → Creates 5 Dio instances, each with CookieManager        │
│    + AuthInterceptor, all sharing PersistCookieJar          │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  SplashScreen                                               │
│  → ref.read(authProvider.notifier).checkSession()            │
│  → GET /auth/me via _dataSource.getMe()                     │
│  → cookie_jar auto-sends existing cookies                   │
│  → If 401 → AuthInterceptor._onUnauthorized → AuthInitial   │
│  → If 200 → AuthRouterBridge.notifyListeners()              │
│  → GoRouter redirect() → /dashboard/sales                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  LoginScreen                                                │
│  → ref.read(authProvider.notifier).login(user, pass)         │
│  → POST /auth/login via _dataSource.login()                 │
│  → Server sets Set-Cookie header                            │
│  → CookieManager saves to PersistCookieJar                  │
│  → ApiResponseHelper checks { status: true } envelope       │
│  → Repository maps json → AuthSession entity                │
│  → AuthNotifier state = AuthAuthenticated                    │
│  → AuthRouterBridge.notifyListeners()                        │
│  → GoRouter redirect() → /dashboard/sales                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Complete pubspec Dependencies

```yaml
dependencies:
  dio: ^5.7.0
  cookie_jar: ^4.0.8
  dio_cookie_manager: ^3.1.1
  flutter_secure_storage: ^9.2.4
  path_provider: ^2.1.5
  dartz: ^0.10.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
```

---

## Migration Guide: Bearer → Cookie Auth

If your app currently uses Bearer tokens and you need to switch to cookies:

1. **Add dependencies**: `cookie_jar`, `dio_cookie_manager`
2. **Create DioClient** with `PersistCookieJar` and `CookieManager` interceptor
3. **Replace AuthInterceptor** — instead of reading/setting Bearer tokens, let `CookieManager` handle cookies automatically
4. **Keep Bearer fallback** — some cross-origin services may still need it
5. **Update login flow** — the server now sets `Set-Cookie` instead of returning a token JSON body
6. **Update logout flow** — call `DioClient.reset()` to clear all cookies from disk
7. **Add `validateStatus: (_) => true`** to all Dio instances so 401 is handled in the interceptor, not thrown as an exception
