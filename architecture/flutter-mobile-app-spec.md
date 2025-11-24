# Flutter Mobile App - Technical Specification

**Last Updated:** October 22, 2025  
**Platform:** Flutter 3.x (iOS + Android)  
**Target:** Owner-only mobile experience for daily monitoring and AI-powered decision making

---

## Table of Contents

1. [App Architecture](#app-architecture)
2. [Project Structure](#project-structure)
3. [State Management](#state-management-riverpod)
4. [Core Features](#core-features-implementation)
5. [UI/UX Guidelines](#uiux-guidelines)
6. [Push Notifications](#push-notifications-implementation)
7. [Offline Strategy](#offline-strategy)
8. [Performance Optimization](#performance-optimization)

---

## App Architecture

### Clean Architecture Layers

```
lib/
├── core/                  # Shared utilities, constants, base classes
│   ├── api/              # REST API client
│   ├── constants/        # App constants, colors, strings
│   ├── error/            # Error handling
│   ├── storage/          # Local storage (Hive)
│   └── utils/            # Helper functions
├── features/             # Feature-based modules
│   ├── auth/
│   │   ├── data/         # API calls, models
│   │   ├── domain/       # Business logic
│   │   └── presentation/ # UI, state providers
│   ├── dashboard/
│   ├── recommendations/
│   ├── notifications/
│   └── locations/
└── main.dart
```

### Tech Stack

```yaml
# pubspec.yaml
dependencies:
  flutter: sdk: flutter
  
  # State Management
  flutter_riverpod: ^2.4.0       # Simple, performant state management
  
  # API & Network
  dio: ^5.3.0                     # HTTP client with interceptors
  retrofit: ^4.0.0                # Type-safe REST client
  json_annotation: ^4.8.0
  
  # Local Storage
  hive: ^2.2.3                    # Fast NoSQL database
  hive_flutter: ^1.1.0
  flutter_secure_storage: ^9.0.0  # Secure token storage
  
  # Push Notifications
  firebase_core: ^2.24.0
  firebase_messaging: ^14.7.0
  flutter_local_notifications: ^16.2.0
  
  # UI Components
  fl_chart: ^0.65.0               # Beautiful charts
  shimmer: ^3.0.0                 # Loading skeleton
  cached_network_image: ^3.3.0    # Image caching
  pull_to_refresh: ^2.0.0         # Pull-to-refresh
  
  # Utilities
  intl: ^0.18.0                   # Vietnamese formatting
  timeago: ^3.5.0                 # "2 giờ trước"
  url_launcher: ^6.2.0            # Open web portal
  package_info_plus: ^5.0.0       # App version
  
  # Biometric Auth
  local_auth: ^2.1.7              # Face ID / Fingerprint
```

---

## Project Structure

### Complete Directory Tree

```
owner_mobile_app/
├── lib/
│   ├── core/                           # Shared foundation layer
│   │   ├── api/                        # HTTP client & API configuration
│   │   │   ├── api_client.dart         # Dio instance with interceptors
│   │   │   ├── api_interceptor.dart    # Auth token, logging, error handling
│   │   │   └── api_endpoints.dart      # Centralized endpoint constants
│   │   ├── constants/                  # Design system & app constants
│   │   │   ├── app_colors.dart         # Color palette (Vietnamese theme)
│   │   │   ├── app_text_styles.dart    # Typography definitions
│   │   │   ├── app_spacing.dart        # Spacing system (4dp base)
│   │   │   └── app_strings.dart        # Hardcoded strings (Vietnamese)
│   │   ├── error/                      # Error handling
│   │   │   ├── app_exception.dart      # Custom exception types
│   │   │   ├── error_handler.dart      # Global error mapper
│   │   │   └── error_messages.dart     # User-friendly Vietnamese messages
│   │   ├── storage/                    # Local persistence
│   │   │   ├── hive_config.dart        # Hive initialization & adapters
│   │   │   ├── secure_storage.dart     # Token storage (flutter_secure_storage)
│   │   │   └── cache_manager.dart      # Metrics & offline data caching
│   │   ├── utils/                      # Helper utilities
│   │   │   ├── formatters.dart         # VN currency, date/time formatters
│   │   │   ├── validators.dart         # Phone, input validation
│   │   │   └── logger.dart             # Logging utility
│   │   └── widgets/                    # Reusable UI components
│   │       ├── app_button.dart         # Primary/secondary/text buttons
│   │       ├── phone_input.dart        # Vietnamese phone input
│   │       ├── empty_state.dart        # Empty list states
│   │       ├── error_widget.dart       # Error display with retry
│   │       ├── shimmer_loading.dart    # Skeleton loading states
│   │       └── app_bottom_nav.dart     # Bottom navigation bar
│   │
│   ├── features/                       # Feature modules (Clean Architecture)
│   │   ├── auth/                       # Authentication & onboarding
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   │   ├── otp_request.dart        # Request DTOs
│   │   │   │   │   ├── otp_request.g.dart      # JSON serialization
│   │   │   │   │   ├── auth_response.dart      # Response DTOs
│   │   │   │   │   └── auth_response.g.dart
│   │   │   │   ├── auth_api.dart               # Retrofit API interface
│   │   │   │   ├── auth_api.g.dart             # Generated API client
│   │   │   │   └── auth_repository.dart        # Data layer logic
│   │   │   ├── domain/
│   │   │   │   ├── models/
│   │   │   │   │   ├── user.dart               # Domain entities
│   │   │   │   │   ├── user.freezed.dart       # Immutable models
│   │   │   │   │   └── user.g.dart
│   │   │   │   └── use_cases/
│   │   │   │       ├── login_with_otp.dart     # Business logic
│   │   │   │       └── logout.dart
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   ├── auth_provider.dart      # Riverpod state
│   │   │       │   └── current_user_provider.dart
│   │   │       ├── screens/
│   │   │       │   ├── phone_input_screen.dart
│   │   │       │   └── otp_screen.dart
│   │   │       └── widgets/
│   │   │           └── otp_input.dart          # Custom OTP widget
│   │   │
│   │   ├── dashboard/                  # Home dashboard
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   │   ├── dashboard_metrics.dart  # API response models
│   │   │   │   │   └── dashboard_metrics.g.dart
│   │   │   │   ├── dashboard_api.dart
│   │   │   │   ├── dashboard_api.g.dart
│   │   │   │   └── dashboard_repository.dart
│   │   │   ├── domain/
│   │   │   │   └── models/
│   │   │   │       ├── metric_card.dart        # UI domain models
│   │   │   │       └── metric_card.freezed.dart
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   ├── today_metrics_provider.dart
│   │   │       │   └── selected_location_provider.dart
│   │   │       ├── screens/
│   │   │       │   └── dashboard_screen.dart
│   │   │       └── widgets/
│   │   │           ├── metric_card.dart
│   │   │           ├── top_selling_items.dart
│   │   │           └── revenue_trend_chart.dart
│   │   │
│   │   ├── recommendations/            # AI recommendations
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   │   ├── recommendation.dart     # Vietnamese properties
│   │   │   │   │   └── recommendation.g.dart   # title, description in VN
│   │   │   │   ├── recommendations_api.dart
│   │   │   │   └── recommendations_repository.dart
│   │   │   ├── domain/
│   │   │   │   └── models/
│   │   │   │       └── recommendation_priority.dart  # Enum: critical, high, normal
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   └── recommendations_provider.dart
│   │   │       ├── screens/
│   │   │       │   ├── recommendations_screen.dart
│   │   │       │   └── recommendation_detail_screen.dart
│   │   │       └── widgets/
│   │   │           ├── recommendation_card.dart
│   │   │           └── priority_badge.dart
│   │   │
│   │   ├── notifications/              # Push notifications & alerts
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   │   ├── notification.dart
│   │   │   │   │   └── notification.g.dart
│   │   │   │   ├── notifications_api.dart
│   │   │   │   └── notifications_repository.dart
│   │   │   ├── domain/
│   │   │   │   └── models/
│   │   │   │       └── notification_type.dart  # Enum: ai_recommendation, alert, summary
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   └── notifications_provider.dart
│   │   │       ├── screens/
│   │   │       │   └── notifications_screen.dart
│   │   │       └── widgets/
│   │   │           └── notification_card.dart
│   │   │
│   │   └── locations/                  # Multi-location management
│   │       ├── data/
│   │       │   ├── models/
│   │       │   │   ├── location.dart   # Vietnamese address fields
│   │       │   │   └── location.g.dart
│   │       │   ├── locations_api.dart
│   │       │   └── locations_repository.dart
│   │       └── presentation/
│   │           ├── providers/
│   │           │   └── locations_provider.dart
│   │           └── widgets/
│   │               └── location_dropdown.dart  # Chain shop selector
│   │
│   ├── config/                         # Environment configuration
│   │   ├── environment.dart            # Dev/staging/prod configs
│   │   └── firebase_options.dart       # Generated by FlutterFire CLI
│   │
│   ├── main.dart                       # Production entry point
│   ├── main_dev.dart                   # Development entry point
│   └── main_prod.dart                  # Production entry point (explicit)
│
├── test/                               # Mirror lib/ structure
│   ├── core/
│   │   ├── api/
│   │   │   └── api_client_test.dart
│   │   └── utils/
│   │       └── formatters_test.dart    # Test VN currency formatting
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   │   └── auth_repository_test.dart
│   │   │   └── presentation/
│   │   │       └── providers/
│   │   │           └── auth_provider_test.dart
│   │   └── dashboard/
│   │       └── presentation/
│   │           └── screens/
│   │               └── dashboard_screen_test.dart  # Widget tests
│   └── test_helpers/
│       ├── mock_data.dart              # Vietnamese test fixtures
│       └── mock_providers.dart         # Riverpod mocks
│
├── integration_test/                   # End-to-end tests
│   └── app_test.dart                   # Full flow: login → dashboard
│
├── assets/                             # Static resources
│   ├── images/
│   │   ├── logo.png
│   │   └── empty_states/               # Vietnamese empty state illustrations
│   ├── fonts/                          # Optional: custom Vietnamese fonts
│   └── translations/                   # Future: i18n JSON files
│       └── vi_VN.json
│
├── android/                            # Android native code
├── ios/                                # iOS native code
├── pubspec.yaml                        # Dependencies
├── analysis_options.yaml               # Linting rules
└── README.md
```

### Feature Module Pattern (Clean Architecture)

Each feature follows this structure:

```
feature_name/
├── data/              # External data sources (API, local DB)
│   ├── models/        # DTOs (match API JSON structure exactly)
│   ├── *_api.dart     # Retrofit interface (@GET, @POST)
│   └── *_repository.dart  # Implements domain repository interface
├── domain/            # Business logic (platform-independent)
│   ├── models/        # Domain entities (app's internal representation)
│   └── use_cases/     # Business rules (optional for simple CRUD)
└── presentation/      # UI layer
    ├── providers/     # Riverpod state management
    ├── screens/       # Full-page widgets
    └── widgets/       # Feature-specific components
```

### **Example: Dashboard Feature**

```dart
// data/models/dashboard_metrics.dart (API DTO)
@JsonSerializable()
class DashboardMetrics {
  @JsonKey(name: 'location_id')
  final String locationId;
  
  @JsonKey(name: 'revenue')
  final MetricValue revenue;
  
  @JsonKey(name: 'orders')
  final MetricValue orders;
  
  DashboardMetrics({
    required this.locationId,
    required this.revenue,
    required this.orders,
  });
  
  factory DashboardMetrics.fromJson(Map<String, dynamic> json) =>
      _$DashboardMetricsFromJson(json);
}

// domain/models/metric_card.dart (UI model)
@freezed
class MetricCard with _$MetricCard {
  const factory MetricCard({
    required String title,           // "Doanh thu hôm nay"
    required String value,            // "2,5tr"
    required double changePercent,    // 15.5
    required Color color,
    required IconData icon,
  }) = _MetricCard;
}

// presentation/providers/today_metrics_provider.dart
final todayMetricsProvider = FutureProvider.autoDispose.family<DashboardMetrics, String>(
  (ref, locationId) async {
    final repo = ref.watch(dashboardRepositoryProvider);
    return repo.getTodayMetrics(locationId);
  },
);

// presentation/screens/dashboard_screen.dart
class DashboardScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final metricsAsync = ref.watch(todayMetricsProvider(locationId));
    
    return metricsAsync.when(
      data: (metrics) => DashboardContent(metrics: metrics),
      loading: () => ShimmerLoading(),
      error: (error, _) => ErrorWidget(error),
    );
  }
}
```

### Generated Files Strategy

**Code Generation Tools:**

```yaml
# pubspec.yaml
dev_dependencies:
  build_runner: ^2.4.0
  json_serializable: ^6.7.0      # @JsonSerializable()
  freezed: ^2.4.0                # @freezed for immutable models
  freezed_annotation: ^2.4.0
  retrofit_generator: ^7.0.0     # @RestApi()
  hive_generator: ^2.0.0         # @HiveType()
```

**Generate Command:**

```bash
# One-time generation
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (auto-regenerate on file changes)
flutter pub run build_runner watch
```

**Generated File Types:**

| Pattern | Purpose | Example |
|---------|---------|---------|
| `*.g.dart` | JSON serialization | `dashboard_metrics.g.dart` |
| `*.freezed.dart` | Immutable models with copyWith | `user.freezed.dart` |
| `*.adapter.g.dart` | Hive type adapters | `cached_metrics.adapter.g.dart` |
| API client implementation | Retrofit REST client | `auth_api.g.dart` |

**Annotation Examples:**

```dart
// JSON serialization (data/models)
@JsonSerializable()
class Recommendation {
  @JsonKey(name: 'recommendation_id')
  final String id;
  
  @JsonKey(name: 'title_vn')
  final String title;  // Already in Vietnamese from API
  
  Recommendation({required this.id, required this.title});
  
  factory Recommendation.fromJson(Map<String, dynamic> json) =>
      _$RecommendationFromJson(json);
}

// Freezed immutable models (domain/models)
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required String phone,
    required Tenant tenant,
  }) = _User;
  
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Hive local storage
@HiveType(typeId: 0)
class CachedMetrics extends HiveObject {
  @HiveField(0)
  final String locationId;
  
  @HiveField(1)
  final double revenue;
  
  CachedMetrics({required this.locationId, required this.revenue});
}

// Retrofit API client
@RestApi(baseUrl: "https://api.quanly.ai/v1")
abstract class DashboardApi {
  factory DashboardApi(Dio dio) = _DashboardApi;
  
  @GET("/locations/{id}/metrics/today")
  Future<ApiResponse<DashboardMetrics>> getTodayMetrics(
    @Path("id") String locationId,
  );
}
```

### .gitignore (Generated Files)

```gitignore
# Generated files (DO NOT commit)
*.g.dart
*.freezed.dart
*.adapter.g.dart

# Exceptions: Keep firebase_options.dart (FlutterFire config)
!firebase_options.dart

# Build outputs
build/
.dart_tool/
```

### File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Screens | `*_screen.dart` | `dashboard_screen.dart` |
| Widgets | Descriptive noun | `metric_card.dart` |
| Providers | `*_provider.dart` | `today_metrics_provider.dart` |
| Repositories | `*_repository.dart` | `dashboard_repository.dart` |
| Models (data) | Singular noun | `recommendation.dart` |
| Models (domain) | Singular noun | `user.dart` |
| APIs | `*_api.dart` | `auth_api.dart` |
| Constants | `app_*.dart` | `app_colors.dart` |
| Utilities | Plural noun | `formatters.dart` |

### Environment-Specific Entry Points

```dart
// lib/main_dev.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Development configuration
  final config = AppConfig.dev;
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  runApp(
    ProviderScope(
      overrides: [
        configProvider.overrideWithValue(config),
      ],
      child: MyApp(),
    ),
  );
}

// lib/main_prod.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Production configuration
  final config = AppConfig.prod;
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  // Disable debug prints in production
  debugPrint = (String? message, {int? wrapWidth}) {};
  
  runApp(
    ProviderScope(
      overrides: [
        configProvider.overrideWithValue(config),
      ],
      child: MyApp(),
    ),
  );
}

// Run commands
// flutter run -t lib/main_dev.dart
// flutter run -t lib/main_prod.dart --release
```

---

## State Management: Riverpod

### Why Riverpod?

- ✅ Simple to learn (vs Bloc)
- ✅ Compile-safe (vs Provider)
- ✅ Auto-dispose (memory efficient)
- ✅ Built-in loading/error states
- ✅ Easy testing

### Example: Dashboard Metrics Provider

```dart
// features/dashboard/data/dashboard_repository.dart
class DashboardRepository {
  final ApiClient _api;
  
  DashboardRepository(this._api);
  
  Future<DashboardMetrics> getTodayMetrics(String locationId) async {
    final response = await _api.get('/locations/$locationId/metrics/today');
    return DashboardMetrics.fromJson(response.data['data']);
  }
}

// features/dashboard/presentation/dashboard_provider.dart
final dashboardRepositoryProvider = Provider((ref) {
  final api = ref.watch(apiClientProvider);
  return DashboardRepository(api);
});

final todayMetricsProvider = FutureProvider.autoDispose.family<DashboardMetrics, String>(
  (ref, locationId) async {
    final repo = ref.watch(dashboardRepositoryProvider);
    return repo.getTodayMetrics(locationId);
  },
);

// Usage in UI
class DashboardScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedLocation = ref.watch(selectedLocationProvider);
    final metricsAsync = ref.watch(todayMetricsProvider(selectedLocation.id));
    
    return metricsAsync.when(
      loading: () => ShimmerLoading(),
      error: (error, stack) => ErrorWidget(error),
      data: (metrics) => DashboardContent(metrics: metrics),
    );
  }
}
```

---

## Core Features Implementation

### 1. Authentication: Phone OTP

```dart
// features/auth/data/auth_api.dart
@RestApi(baseUrl: "https://api.quanly.ai/v1")
abstract class AuthApi {
  factory AuthApi(Dio dio) = _AuthApi;
  
  @POST("/auth/otp/request")
  Future<OtpSessionResponse> requestOtp(@Body() OtpRequest request);
  
  @POST("/auth/otp/verify")
  Future<AuthResponse> verifyOtp(@Body() OtpVerifyRequest request);
}

// features/auth/presentation/otp_screen.dart
class OtpScreen extends ConsumerStatefulWidget {
  final String phone;
  
  @override
  _OtpScreenState createState() => _OtpScreenState();
}

class _OtpScreenState extends ConsumerState<OtpScreen> {
  final _otpController = TextEditingController();
  String? _sessionId;
  
  Future<void> _requestOtp() async {
    try {
      final response = await ref.read(authApiProvider).requestOtp(
        OtpRequest(phone: widget.phone),
      );
      
      setState(() => _sessionId = response.sessionId);
      
      // Show toast
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Mã OTP đã được gửi đến ${widget.phone}')),
      );
    } catch (e) {
      _handleError(e);
    }
  }
  
  Future<void> _verifyOtp() async {
    if (_otpController.text.length != 6) return;
    
    try {
      final response = await ref.read(authApiProvider).verifyOtp(
        OtpVerifyRequest(
          sessionId: _sessionId!,
          code: _otpController.text,
        ),
      );
      
      // Save tokens securely
      await ref.read(authStorageProvider).saveTokens(
        accessToken: response.accessToken,
        refreshToken: response.refreshToken,
      );
      
      // Navigate to dashboard
      Navigator.of(context).pushReplacement(
        MaterialPageRoute(builder: (_) => DashboardScreen()),
      );
    } catch (e) {
      _handleError(e);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Xác thực OTP')),
      body: Padding(
        padding: EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Nhập mã OTP',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Text(
              'Mã đã được gửi đến ${widget.phone}',
              style: TextStyle(color: Colors.grey[600]),
            ),
            SizedBox(height: 32),
            
            // OTP Input (6 digits)
            PinCodeTextField(
              length: 6,
              controller: _otpController,
              onChanged: (value) {
                if (value.length == 6) {
                  _verifyOtp();
                }
              },
            ),
            
            SizedBox(height: 24),
            
            // Resend button
            TextButton(
              onPressed: _canResend ? _requestOtp : null,
              child: Text('Gửi lại mã'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 2. Dashboard Home Screen

```dart
// features/dashboard/presentation/dashboard_screen.dart
class DashboardScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(currentUserProvider);
    final selectedLocation = ref.watch(selectedLocationProvider);
    final metricsAsync = ref.watch(todayMetricsProvider(selectedLocation.id));
    
    return Scaffold(
      appBar: AppBar(
        title: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Chào buổi ${_getGreeting()}, ${user.name}'),
            if (user.tenant.isChain)
              LocationDropdown(
                locations: user.tenant.locations,
                selected: selectedLocation,
                onChanged: (location) {
                  ref.read(selectedLocationProvider.notifier).state = location;
                },
              ),
          ],
        ),
        actions: [
          // Notification bell with badge
          NotificationBadge(),
        ],
      ),
      
      body: RefreshIndicator(
        onRefresh: () async {
          ref.invalidate(todayMetricsProvider);
        },
        child: metricsAsync.when(
          loading: () => _buildShimmerLoading(),
          error: (error, _) => _buildError(error),
          data: (metrics) => _buildDashboard(context, ref, metrics),
        ),
      ),
      
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: 0,
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Trang chủ',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.auto_awesome),
            label: 'AI',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.notifications),
            label: 'Thông báo',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.more_horiz),
            label: 'Thêm',
          ),
        ],
      ),
    );
  }
  
  Widget _buildDashboard(BuildContext context, WidgetRef ref, DashboardMetrics metrics) {
    return SingleChildScrollView(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Today's Metrics Cards
          _buildMetricCard(
            title: 'Doanh Thu Hôm Nay',
            value: _formatCurrency(metrics.revenue.current),
            change: metrics.revenue.changePercent,
            icon: Icons.attach_money,
            color: Colors.green,
          ),
          
          SizedBox(height: 12),
          
          Row(
            children: [
              Expanded(
                child: _buildMetricCard(
                  title: 'Đơn Hàng',
                  value: '${metrics.orders.current}',
                  change: metrics.orders.changePercent,
                  icon: Icons.shopping_cart,
                  color: Colors.blue,
                  compact: true,
                ),
              ),
              SizedBox(width: 12),
              Expanded(
                child: _buildMetricCard(
                  title: 'Giá Trị TB',
                  value: _formatCurrency(metrics.averageOrderValue.current),
                  change: metrics.averageOrderValue.changePercent,
                  icon: Icons.trending_up,
                  color: Colors.orange,
                  compact: true,
                ),
              ),
            ],
          ),
          
          SizedBox(height: 24),
          
          // AI Recommendations Section
          _buildSectionHeader('🤖 AI Khuyến Nghị', onViewAll: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (_) => RecommendationsScreen()),
            );
          }),
          
          SizedBox(height: 12),
          
          RecommendationsPreview(locationId: metrics.locationId),
          
          SizedBox(height: 24),
          
          // Top Selling Items
          _buildSectionHeader('🔥 Bán Chạy Nhất Hôm Nay'),
          
          SizedBox(height: 12),
          
          TopSellingItems(locationId: metrics.locationId),
          
          SizedBox(height: 24),
          
          // Revenue Trend Chart
          _buildSectionHeader('📈 Xu Hướng 7 Ngày'),
          
          SizedBox(height: 12),
          
          RevenueTrendChart(locationId: metrics.locationId),
        ],
      ),
    );
  }
  
  Widget _buildMetricCard({
    required String title,
    required String value,
    required double change,
    required IconData icon,
    required Color color,
    bool compact = false,
  }) {
    return Container(
      padding: EdgeInsets.all(compact ? 12 : 16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 10,
            offset: Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              Container(
                padding: EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: color.withOpacity(0.1),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Icon(icon, color: color, size: compact ? 20 : 24),
              ),
              Spacer(),
              _buildChangeIndicator(change),
            ],
          ),
          
          SizedBox(height: compact ? 8 : 12),
          
          Text(
            value,
            style: TextStyle(
              fontSize: compact ? 20 : 28,
              fontWeight: FontWeight.bold,
            ),
          ),
          
          SizedBox(height: 4),
          
          Text(
            title,
            style: TextStyle(
              fontSize: compact ? 12 : 14,
              color: Colors.grey[600],
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildChangeIndicator(double changePercent) {
    final isPositive = changePercent >= 0;
    final color = isPositive ? Colors.green : Colors.red;
    final icon = isPositive ? Icons.arrow_upward : Icons.arrow_downward;
    
    return Container(
      padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(4),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, color: color, size: 14),
          SizedBox(width: 4),
          Text(
            '${changePercent.abs().toStringAsFixed(1)}%',
            style: TextStyle(
              color: color,
              fontWeight: FontWeight.bold,
              fontSize: 12,
            ),
          ),
        ],
      ),
    );
  }
}
```

### 3. AI Recommendations Screen

```dart
// features/recommendations/presentation/recommendations_screen.dart
class RecommendationsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedLocation = ref.watch(selectedLocationProvider);
    final recommendationsAsync = ref.watch(
      recommendationsProvider(selectedLocation.id),
    );
    
    return Scaffold(
      appBar: AppBar(
        title: Text('🤖 AI Khuyến Nghị'),
        actions: [
          FilterButton(), // Filter by priority, status
        ],
      ),
      body: recommendationsAsync.when(
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, _) => ErrorWidget(error),
        data: (recommendations) {
          if (recommendations.isEmpty) {
            return EmptyState(
              icon: Icons.auto_awesome,
              title: 'Chưa có khuyến nghị',
              subtitle: 'AI đang phân tích dữ liệu...',
            );
          }
          
          return ListView.separated(
            padding: EdgeInsets.all(16),
            itemCount: recommendations.length,
            separatorBuilder: (_, __) => SizedBox(height: 12),
            itemBuilder: (context, index) {
              final rec = recommendations[index];
              return RecommendationCard(
                recommendation: rec,
                onTap: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => RecommendationDetailScreen(
                        recommendationId: rec.id,
                      ),
                    ),
                  );
                },
                onApprove: () async {
                  await _showApproveDialog(context, ref, rec);
                },
                onDismiss: () async {
                  await _showDismissDialog(context, ref, rec);
                },
              );
            },
          );
        },
      ),
    );
  }
  
  Future<void> _showApproveDialog(
    BuildContext context,
    WidgetRef ref,
    Recommendation rec,
  ) async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Xác nhận duyệt'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(rec.title),
            SizedBox(height: 12),
            Text(
              'Dự kiến tăng doanh thu: ${_formatCurrency(rec.impact.estimatedIncrease)}',
              style: TextStyle(
                color: Colors.green,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: Text('Hủy'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pop(context, true),
            child: Text('Duyệt ngay'),
          ),
        ],
      ),
    );
    
    if (confirmed == true) {
      try {
        await ref.read(recommendationsRepositoryProvider).approve(rec.id);
        
        // Haptic feedback
        HapticFeedback.heavyImpact();
        
        // Show success
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('✅ Đã duyệt. AI đang thực hiện...'),
            backgroundColor: Colors.green,
          ),
        );
        
        // Refresh list
        ref.invalidate(recommendationsProvider);
      } catch (e) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('Lỗi: ${e.toString()}'),
            backgroundColor: Colors.red,
          ),
        );
      }
    }
  }
}

// Recommendation Card Widget
class RecommendationCard extends StatelessWidget {
  final Recommendation recommendation;
  final VoidCallback onTap;
  final VoidCallback onApprove;
  final VoidCallback onDismiss;
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
        side: BorderSide(
          color: _getPriorityColor().withOpacity(0.3),
          width: 2,
        ),
      ),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                children: [
                  _buildPriorityBadge(),
                  SizedBox(width: 8),
                  Text(
                    _getTimeAgo(recommendation.createdAt),
                    style: TextStyle(
                      fontSize: 12,
                      color: Colors.grey[600],
                    ),
                  ),
                  Spacer(),
                  Icon(
                    Icons.chevron_right,
                    color: Colors.grey[400],
                  ),
                ],
              ),
              
              SizedBox(height: 12),
              
              Text(
                recommendation.title,
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
              
              SizedBox(height: 8),
              
              Text(
                recommendation.description,
                style: TextStyle(
                  fontSize: 14,
                  color: Colors.grey[700],
                ),
              ),
              
              SizedBox(height: 12),
              
              Container(
                padding: EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: Colors.green.withOpacity(0.1),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Row(
                  children: [
                    Icon(Icons.trending_up, color: Colors.green, size: 20),
                    SizedBox(width: 8),
                    Text(
                      'Dự kiến tăng ${_formatCurrency(recommendation.impact.estimatedIncrease)}',
                      style: TextStyle(
                        color: Colors.green,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    Spacer(),
                    Text(
                      '${(recommendation.impact.confidence * 100).toInt()}% tin cậy',
                      style: TextStyle(
                        fontSize: 12,
                        color: Colors.grey[600],
                      ),
                    ),
                  ],
                ),
              ),
              
              SizedBox(height: 12),
              
              Row(
                children: [
                  Expanded(
                    child: OutlinedButton(
                      onPressed: onDismiss,
                      child: Text('Bỏ qua'),
                    ),
                  ),
                  SizedBox(width: 8),
                  Expanded(
                    child: ElevatedButton(
                      onPressed: onApprove,
                      child: Text('Duyệt ngay'),
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  Color _getPriorityColor() {
    switch (recommendation.priority) {
      case 'critical':
        return Colors.red;
      case 'high':
        return Colors.orange;
      case 'normal':
        return Colors.blue;
      default:
        return Colors.grey;
    }
  }
  
  Widget _buildPriorityBadge() {
    final color = _getPriorityColor();
    String label;
    
    switch (recommendation.priority) {
      case 'critical':
        label = 'ƯU TIÊN CAO';
        break;
      case 'high':
        label = 'QUAN TRỌNG';
        break;
      default:
        label = 'TRUNG BÌNH';
    }
    
    return Container(
      padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(4),
      ),
      child: Text(
        label,
        style: TextStyle(
          color: color,
          fontSize: 11,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}
```

---

## UI/UX Guidelines

### Design System

**Color Palette:**

```dart
// lib/core/constants/app_colors.dart
class AppColors {
  // Primary Colors (Vietnamese street food inspired)
  static const primary = Color(0xFFFF6B35);      // Vibrant orange (cà phê sữa đá)
  static const primaryDark = Color(0xFFE55528);
  static const primaryLight = Color(0xFFFF8C61);
  
  // Secondary Colors
  static const secondary = Color(0xFF004E89);     // Deep blue (trust)
  static const secondaryLight = Color(0xFF1A6FB7);
  
  // Semantic Colors
  static const success = Color(0xFF10B981);       // Green (positive metrics)
  static const error = Color(0xFFEF4444);         // Red (alerts, declining)
  static const warning = Color(0xFFF59E0B);       // Amber (moderate priority)
  static const info = Color(0xFF3B82F6);          // Blue (informational)
  
  // Neutral Colors
  static const background = Color(0xFFF9FAFB);    // Light grey background
  static const surface = Color(0xFFFFFFFF);       // White cards
  static const border = Color(0xFFE5E7EB);        // Light grey borders
  
  // Text Colors
  static const textPrimary = Color(0xFF111827);   // Near black
  static const textSecondary = Color(0xFF6B7280); // Grey
  static const textDisabled = Color(0xFF9CA3AF);  // Light grey
}
```

**Typography:**

```dart
// lib/core/constants/app_text_styles.dart
class AppTextStyles {
  // Font Family (supports Vietnamese diacritics)
  static const fontFamily = 'Roboto'; // Or 'SF Pro' for iOS
  
  // Headings
  static const h1 = TextStyle(
    fontSize: 28,
    fontWeight: FontWeight.bold,
    height: 1.3,
    letterSpacing: -0.5,
  );
  
  static const h2 = TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    height: 1.3,
  );
  
  static const h3 = TextStyle(
    fontSize: 20,
    fontWeight: FontWeight.w600,
    height: 1.4,
  );
  
  // Body Text
  static const bodyLarge = TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.normal,
    height: 1.5,
  );
  
  static const body = TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    height: 1.5,
  );
  
  static const bodySmall = TextStyle(
    fontSize: 12,
    fontWeight: FontWeight.normal,
    height: 1.4,
  );
  
  // Special
  static const button = TextStyle(
    fontSize: 15,
    fontWeight: FontWeight.w600,
    height: 1.2,
  );
  
  static const caption = TextStyle(
    fontSize: 11,
    fontWeight: FontWeight.normal,
    height: 1.3,
    letterSpacing: 0.3,
  );
}
```

**Spacing System:**

```dart
// lib/core/constants/app_spacing.dart
class AppSpacing {
  static const xxs = 4.0;
  static const xs = 8.0;
  static const sm = 12.0;
  static const md = 16.0;
  static const lg = 24.0;
  static const xl = 32.0;
  static const xxl = 48.0;
  
  // Common paddings
  static const screenPadding = EdgeInsets.all(md);
  static const cardPadding = EdgeInsets.all(md);
  static const listItemPadding = EdgeInsets.symmetric(horizontal: md, vertical: sm);
}
```

### Component Library

**Primary Button:**

```dart
// lib/core/widgets/app_button.dart
class AppButton extends StatelessWidget {
  final String text;
  final VoidCallback? onPressed;
  final bool isLoading;
  final AppButtonType type;
  
  const AppButton({
    required this.text,
    required this.onPressed,
    this.isLoading = false,
    this.type = AppButtonType.primary,
  });
  
  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      height: 48, // Minimum touch target
      child: ElevatedButton(
        onPressed: isLoading ? null : onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: _getBackgroundColor(),
          foregroundColor: _getTextColor(),
          elevation: type == AppButtonType.primary ? 2 : 0,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          padding: EdgeInsets.symmetric(horizontal: AppSpacing.md),
        ),
        child: isLoading
            ? SizedBox(
                height: 20,
                width: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  valueColor: AlwaysStoppedAnimation(_getTextColor()),
                ),
              )
            : Text(text, style: AppTextStyles.button),
      ),
    );
  }
  
  Color _getBackgroundColor() {
    if (type == AppButtonType.primary) return AppColors.primary;
    if (type == AppButtonType.secondary) return AppColors.background;
    return Colors.transparent;
  }
  
  Color _getTextColor() {
    if (type == AppButtonType.primary) return Colors.white;
    return AppColors.textPrimary;
  }
}

enum AppButtonType { primary, secondary, text }
```

**Input Field (Phone Number):**

```dart
// lib/core/widgets/phone_input.dart
class PhoneInput extends StatelessWidget {
  final TextEditingController controller;
  final String? errorText;
  final VoidCallback? onChanged;
  
  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: controller,
      keyboardType: TextInputType.phone,
      decoration: InputDecoration(
        labelText: 'Số điện thoại',
        hintText: '0901234567',
        prefixIcon: Icon(Icons.phone, color: AppColors.textSecondary),
        errorText: errorText,
        filled: true,
        fillColor: AppColors.surface,
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide(color: AppColors.border),
        ),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide(color: AppColors.border),
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide(color: AppColors.primary, width: 2),
        ),
        errorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide(color: AppColors.error, width: 2),
        ),
      ),
      onChanged: (_) => onChanged?.call(),
    );
  }
}
```

**Empty State:**

```dart
// lib/core/widgets/empty_state.dart
class EmptyState extends StatelessWidget {
  final IconData icon;
  final String title;
  final String subtitle;
  final String? actionText;
  final VoidCallback? onAction;
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: EdgeInsets.all(AppSpacing.xl),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(
              padding: EdgeInsets.all(AppSpacing.lg),
              decoration: BoxDecoration(
                color: AppColors.background,
                shape: BoxShape.circle,
              ),
              child: Icon(
                icon,
                size: 64,
                color: AppColors.textSecondary,
              ),
            ),
            SizedBox(height: AppSpacing.lg),
            Text(
              title,
              style: AppTextStyles.h3.copyWith(color: AppColors.textPrimary),
              textAlign: TextAlign.center,
            ),
            SizedBox(height: AppSpacing.xs),
            Text(
              subtitle,
              style: AppTextStyles.body.copyWith(color: AppColors.textSecondary),
              textAlign: TextAlign.center,
            ),
            if (actionText != null && onAction != null) ...[
              SizedBox(height: AppSpacing.lg),
              AppButton(
                text: actionText!,
                onPressed: onAction,
                type: AppButtonType.secondary,
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

### Navigation Patterns

**Bottom Navigation:**

```dart
// lib/core/widgets/app_bottom_nav.dart
class AppBottomNav extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentIndex = ref.watch(bottomNavIndexProvider);
    
    return BottomNavigationBar(
      currentIndex: currentIndex,
      onTap: (index) {
        ref.read(bottomNavIndexProvider.notifier).state = index;
      },
      type: BottomNavigationBarType.fixed,
      backgroundColor: AppColors.surface,
      selectedItemColor: AppColors.primary,
      unselectedItemColor: AppColors.textSecondary,
      selectedFontSize: 12,
      unselectedFontSize: 12,
      items: [
        BottomNavigationBarItem(
          icon: Icon(Icons.home_outlined),
          activeIcon: Icon(Icons.home),
          label: 'Trang chủ',
        ),
        BottomNavigationBarItem(
          icon: Icon(Icons.auto_awesome_outlined),
          activeIcon: Icon(Icons.auto_awesome),
          label: 'AI',
        ),
        BottomNavigationBarItem(
          icon: Icon(Icons.notifications_outlined),
          activeIcon: Icon(Icons.notifications),
          label: 'Thông báo',
        ),
        BottomNavigationBarItem(
          icon: Icon(Icons.more_horiz),
          activeIcon: Icon(Icons.menu),
          label: 'Thêm',
        ),
      ],
    );
  }
}

final bottomNavIndexProvider = StateProvider<int>((ref) => 0);
```

**Hero Animations:**

```dart
// Animate menu item images between list and detail
Hero(
  tag: 'menu_item_${item.id}',
  child: CachedNetworkImage(
    imageUrl: item.imageUrl,
    width: 60,
    height: 60,
  ),
)
```

### Loading States

**Shimmer Loading (Dashboard Metrics):**

```dart
// lib/core/widgets/shimmer_loading.dart
class MetricCardShimmer extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Shimmer.fromColors(
      baseColor: Colors.grey[300]!,
      highlightColor: Colors.grey[100]!,
      child: Container(
        padding: EdgeInsets.all(AppSpacing.md),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Container(
              width: 40,
              height: 40,
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(8),
              ),
            ),
            SizedBox(height: AppSpacing.sm),
            Container(
              width: 100,
              height: 24,
              color: Colors.white,
            ),
            SizedBox(height: AppSpacing.xs),
            Container(
              width: 60,
              height: 14,
              color: Colors.white,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Pull-to-Refresh:**

```dart
RefreshIndicator(
  onRefresh: () async {
    ref.invalidate(todayMetricsProvider);
    // Wait for new data
    await ref.read(todayMetricsProvider(locationId).future);
  },
  color: AppColors.primary,
  child: ListView(...),
)
```

### Vietnamese Localization

**Number Formatting:**

```dart
// lib/core/utils/formatters.dart
import 'package:intl/intl.dart';

class VNFormatters {
  // Currency: 75.000₫
  static String currency(num amount) {
    final formatter = NumberFormat.currency(
      locale: 'vi_VN',
      symbol: '₫',
      decimalDigits: 0,
    );
    return formatter.format(amount);
  }
  
  // Compact: 1,2tr (1.2 million), 450k (450 thousand)
  static String compactCurrency(num amount) {
    if (amount >= 1000000000) {
      return '${(amount / 1000000000).toStringAsFixed(1)}tỷ';
    } else if (amount >= 1000000) {
      return '${(amount / 1000000).toStringAsFixed(1)}tr';
    } else if (amount >= 1000) {
      return '${(amount / 1000).toStringAsFixed(0)}k';
    }
    return currency(amount);
  }
  
  // Percentage: +15,5%
  static String percentage(double value) {
    final sign = value >= 0 ? '+' : '';
    final formatter = NumberFormat('#,##0.0', 'vi_VN');
    return '$sign${formatter.format(value)}%';
  }
}
```

**Date/Time Formatting:**

```dart
// lib/core/utils/formatters.dart
class VNDateFormatters {
  // Full date: "Thứ 2, 25/11/2025"
  static String fullDate(DateTime date) {
    final weekday = ['CN', 'Hai', 'Ba', 'Tư', 'Năm', 'Sáu', 'Bảy'][date.weekday % 7];
    final formatted = DateFormat('dd/MM/yyyy', 'vi_VN').format(date);
    return 'Thứ $weekday, $formatted';
  }
  
  // Relative: "2 giờ trước", "Hôm qua"
  static String relative(DateTime date) {
    final now = DateTime.now();
    final difference = now.difference(date);
    
    if (difference.inMinutes < 1) return 'Vừa xong';
    if (difference.inHours < 1) return '${difference.inMinutes} phút trước';
    if (difference.inDays < 1) return '${difference.inHours} giờ trước';
    if (difference.inDays == 1) return 'Hôm qua';
    if (difference.inDays < 7) return '${difference.inDays} ngày trước';
    
    return DateFormat('dd/MM/yyyy').format(date);
  }
  
  // Time: "14:30"
  static String time(DateTime date) {
    return DateFormat('HH:mm', 'vi_VN').format(date);
  }
}
```

**Text Tone Guidelines:**

```dart
// Action buttons (use imperatives, friendly)
'Duyệt ngay'      // Approve now (not 'Phê duyệt' - too formal)
'Xem chi tiết'    // View details
'Thử lại'         // Try again
'Cập nhật'        // Update

// Status labels (concise)
'Đang xử lý...'   // Processing...
'Thành công'      // Success
'Đã hoàn thành'   // Completed

// Empty states (encouraging)
'Chưa có dữ liệu' // No data yet (not 'Không có dữ liệu' - too negative)
'Bắt đầu thôi!'   // Let's start!
```

### Accessibility

**Touch Targets:**

```dart
// Minimum 48x48dp for all interactive elements
AppButton(...)  // Already 48dp height

// Small icons need padding
IconButton(
  iconSize: 24,
  padding: EdgeInsets.all(12), // Total: 48x48
  icon: Icon(Icons.close),
  onPressed: onClose,
)
```

**Color Contrast:**

```dart
// Text on background (4.5:1 ratio)
Text(
  'Doanh thu hôm nay',
  style: TextStyle(
    color: AppColors.textPrimary, // #111827 on #FFFFFF = 18.6:1 ✓
  ),
)

// White text on primary (4.5:1 ratio)
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: AppColors.primary, // #FF6B35 with white = 3.6:1 ✗
    // Fix: Use darker primary for text backgrounds
    backgroundColor: AppColors.primaryDark, // #E55528 with white = 4.8:1 ✓
  ),
)
```

**Screen Reader Labels:**

```dart
// Semantic labels for icons
Semantics(
  label: 'Đóng',
  button: true,
  child: IconButton(
    icon: Icon(Icons.close),
    onPressed: onClose,
  ),
)

// Exclude decorative elements
Semantics(
  excludeSemantics: true,
  child: Container(...), // Decorative background
)
```

---

## Push Notifications Implementation

### Firebase Cloud Messaging Setup

```dart
// core/notifications/fcm_service.dart
class FCMService {
  final FirebaseMessaging _fcm = FirebaseMessaging.instance;
  final FlutterLocalNotificationsPlugin _localNotifications =
      FlutterLocalNotificationsPlugin();
  
  Future<void> initialize() async {
    // Request permission (iOS)
    final settings = await _fcm.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );
    
    if (settings.authorizationStatus != AuthorizationStatus.authorized) {
      print('User declined notifications');
      return;
    }
    
    // Get FCM token
    final token = await _fcm.getToken();
    print('FCM Token: $token');
    
    // Register token with backend
    await _registerToken(token!);
    
    // Listen for token refresh
    _fcm.onTokenRefresh.listen(_registerToken);
    
    // Configure local notifications
    await _configureLocalNotifications();
    
    // Handle foreground messages
    FirebaseMessaging.onMessage.listen(_handleForegroundMessage);
    
    // Handle background/terminated messages
    FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
    
    // Handle notification taps
    FirebaseMessaging.onMessageOpenedApp.listen(_handleNotificationTap);
  }
  
  Future<void> _registerToken(String token) async {
    final api = getIt<ApiClient>();
    await api.post('/notifications/devices', data: {
      'token': token,
      'platform': Platform.isIOS ? 'ios' : 'android',
      'device_id': await _getDeviceId(),
      'app_version': await _getAppVersion(),
      'os_version': Platform.operatingSystemVersion,
    });
  }
  
  Future<void> _configureLocalNotifications() async {
    const androidSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings(
      requestAlertPermission: false,
      requestBadgePermission: false,
      requestSoundPermission: false,
    );
    
    await _localNotifications.initialize(
      InitializationSettings(
        android: androidSettings,
        iOS: iosSettings,
      ),
      onDidReceiveNotificationResponse: _handleLocalNotificationTap,
    );
  }
  
  Future<void> _handleForegroundMessage(RemoteMessage message) async {
    final priority = message.data['priority'] ?? 'normal';
    
    if (priority == 'critical') {
      // Show full-screen alert (even if app is in foreground)
      _showCriticalAlert(message);
      HapticFeedback.heavyImpact();
    } else {
      // Show local notification banner
      _showLocalNotification(message);
    }
  }
  
  Future<void> _showLocalNotification(RemoteMessage message) async {
    final notification = message.notification;
    if (notification == null) return;
    
    const androidDetails = AndroidNotificationDetails(
      'default_channel',
      'Default',
      channelDescription: 'Default notification channel',
      importance: Importance.high,
      priority: Priority.high,
      icon: '@mipmap/ic_launcher',
    );
    
    const iosDetails = DarwinNotificationDetails();
    
    await _localNotifications.show(
      message.hashCode,
      notification.title,
      notification.body,
      NotificationDetails(
        android: androidDetails,
        iOS: iosDetails,
      ),
      payload: jsonEncode(message.data),
    );
  }
  
  void _handleNotificationTap(RemoteMessage message) {
    final data = message.data;
    final type = data['type'];
    
    // Navigate based on notification type
    switch (type) {
      case 'ai_recommendation':
        _navigateToRecommendation(data['recommendation_id']);
        break;
      case 'alert':
        _navigateToLocation(data['location_id']);
        break;
      case 'daily_summary':
        _navigateToDashboard();
        break;
    }
  }
}

// Background message handler (must be top-level function)
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  print('Handling background message: ${message.messageId}');
  
  // Update badge count, save notification to local DB, etc.
}
```

---

## Offline Strategy

### Hive Local Storage

```dart
// core/storage/local_database.dart
@HiveType(typeId: 0)
class CachedMetrics extends HiveObject {
  @HiveField(0)
  final String locationId;
  
  @HiveField(1)
  final DateTime date;
  
  @HiveField(2)
  final double revenue;
  
  @HiveField(3)
  final int orders;
  
  @HiveField(4)
  final double averageOrderValue;
  
  @HiveField(5)
  final DateTime cachedAt;
  
  CachedMetrics({
    required this.locationId,
    required this.date,
    required this.revenue,
    required this.orders,
    required this.averageOrderValue,
    required this.cachedAt,
  });
}

// Repository with offline support
class DashboardRepository {
  final ApiClient _api;
  final Box<CachedMetrics> _cache;
  
  Future<DashboardMetrics> getTodayMetrics(String locationId) async {
    try {
      // Try to fetch from API
      final response = await _api.get('/locations/$locationId/metrics/today');
      final metrics = DashboardMetrics.fromJson(response.data['data']);
      
      // Cache the result
      await _cache.put(
        '${locationId}_${DateTime.now().toIso8601String().split('T')[0]}',
        CachedMetrics.fromDashboardMetrics(metrics),
      );
      
      return metrics;
    } catch (e) {
      // If offline, return cached data
      final cacheKey = '${locationId}_${DateTime.now().toIso8601String().split('T')[0]}';
      final cached = _cache.get(cacheKey);
      
      if (cached != null) {
        // Show offline banner
        _showOfflineBanner();
        return DashboardMetrics.fromCached(cached);
      }
      
      rethrow;
    }
  }
}
```

---

## Performance Optimization

### 1. Image Caching

```dart
CachedNetworkImage(
  imageUrl: item.imageUrl,
  placeholder: (context, url) => Shimmer.fromColors(
    baseColor: Colors.grey[300]!,
    highlightColor: Colors.grey[100]!,
    child: Container(
      width: 60,
      height: 60,
      color: Colors.white,
    ),
  ),
  errorWidget: (context, url, error) => Icon(Icons.error),
  width: 60,
  height: 60,
  fit: BoxFit.cover,
);
```

### 2. List Optimization

```dart
// Use ListView.builder for large lists (not ListView with fixed children)
ListView.builder(
  itemCount: recommendations.length,
  itemBuilder: (context, index) {
    return RecommendationCard(recommendation: recommendations[index]);
  },
);

// Use const constructors when possible
const Text('Static text');
```

### 3. Auto-refresh with debounce

```dart
final autoRefreshProvider = StreamProvider((ref) {
  return Stream.periodic(Duration(seconds: 60), (_) async {
    // Only refresh if app is in foreground and user is on dashboard
    if (WidgetsBinding.instance.lifecycleState == AppLifecycleState.resumed) {
      ref.invalidate(todayMetricsProvider);
    }
  });
});
```

---

## Build & Deployment

### Build Commands

```bash
# Development
flutter run --flavor dev -t lib/main_dev.dart

# Production (iOS)
flutter build ios --release --flavor prod -t lib/main_prod.dart

# Production (Android)
flutter build appbundle --release --flavor prod -t lib/main_prod.dart
```

### Environment Configuration

```dart
// lib/config/environment.dart
enum Environment { dev, staging, prod }

class AppConfig {
  final Environment environment;
  final String apiBaseUrl;
  final String firebaseProjectId;
  
  AppConfig({
    required this.environment,
    required this.apiBaseUrl,
    required this.firebaseProjectId,
  });
  
  static AppConfig dev = AppConfig(
    environment: Environment.dev,
    apiBaseUrl: 'https://api-dev.quanly.ai/v1',
    firebaseProjectId: 'quanly-dev',
  );
  
  static AppConfig prod = AppConfig(
    environment: Environment.prod,
    apiBaseUrl: 'https://api.quanly.ai/v1',
    firebaseProjectId: 'quanly-prod',
  );
}
```

---

## Related Documents

- [API Specification](./api-specification.md) - Complete REST API reference
- [Database Schema](./database-schema.md) - Backend data model
- [System Diagram](./system-diagram.md) - Overall architecture
