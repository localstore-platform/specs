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
