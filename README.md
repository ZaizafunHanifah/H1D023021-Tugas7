# H1D023021 - Tugas 7: Flutter App dengan Routes, Side Menu, Login & Local Storage

Aplikasi Flutter yang mengimplementasikan sistem autentikasi, navigasi routes, side menu (drawer), dan penyimpanan lokal untuk tugas mata kuliah.

## 👤 Informasi Developer
- **Nama**: Zaizafun Hanifah Zainnur Hanun
- **NIM**: H1D023021
- **Shift**: C
- **Shift KRS**: A

## Struktur Kode

### `lib/main.dart`
File utama aplikasi yang mengatur konfigurasi aplikasi dan routing.

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:shared_preferences/shared_preferences.dart';
// Import semua halaman aplikasi

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'H1D023021 Tugas 7',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      routerConfig: _router, // Konfigurasi routing menggunakan Go Router
    );
  }
}

// Konfigurasi Go Router dengan redirect logic
final GoRouter _router = GoRouter(
  initialLocation: '/',
  redirect: (context, state) async {
    // Logic untuk redirect berdasarkan status login
    final prefs = await SharedPreferences.getInstance();
    final isLoggedIn = prefs.getBool('isLoggedIn') ?? false;

    if (!isLoggedIn && state.matchedLocation != '/') {
      return '/'; // Redirect ke login jika belum login
    }
    if (isLoggedIn && state.matchedLocation == '/') {
      return '/home'; // Redirect ke home jika sudah login
    }
    return null;
  },
  routes: [
    GoRoute(path: '/', builder: (context, state) => const LoginPage()),
    GoRoute(path: '/home', builder: (context, state) => const HomePage()),
    GoRoute(path: '/profile', builder: (context, state) => const ProfilePage()),
    GoRoute(path: '/settings', builder: (context, state) => const SettingsPage()),
  ],
);
```

**Penjelasan**:
- Menggunakan `MaterialApp.router` untuk routing modern
- `GoRouter` mengatur navigasi dengan redirect otomatis
- Logic redirect memeriksa status login dari SharedPreferences
- Routes didefinisikan untuk setiap halaman aplikasi

### `lib/login_page.dart`
Halaman login dengan form autentikasi dan animasi.

```dart
class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> with TickerProviderStateMixin {
  final _formKey = GlobalKey<FormState>();
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();
  late AnimationController _animationController;
  late Animation<double> _fadeAnimation;

  @override
  void initState() {
    super.initState();
    // Setup animasi fade-in
    _animationController = AnimationController(
      duration: const Duration(milliseconds: 1000),
      vsync: this,
    );
    _fadeAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _animationController, curve: Curves.easeIn),
    );
    _animationController.forward();
  }

  Future<void> _login() async {
    if (_formKey.currentState!.validate()) {
      // Validasi kredensial sederhana
      if (_usernameController.text == 'admin' && _passwordController.text == 'password') {
        final prefs = await SharedPreferences.getInstance();
        await prefs.setBool('isLoggedIn', true);
        await prefs.setString('username', _usernameController.text);

        if (mounted) {
          context.go('/home'); // Navigasi ke home setelah login berhasil
        }
      } else {
        // Tampilkan error message
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Invalid credentials')),
        );
      }
    }
  }
}
```

**Penjelasan**:
- `TickerProviderStateMixin` untuk animasi smooth
- Form validation menggunakan `GlobalKey<FormState>`
- Animasi fade-in saat halaman dimuat
- Autentikasi sederhana (username: admin, password: password)
- Penyimpanan status login ke SharedPreferences
- Navigasi ke home menggunakan `context.go()`

### `lib/home_page.dart`
Halaman utama dengan drawer dan bottom navigation.

```dart
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  String _username = '';

  @override
  void initState() {
    super.initState();
    _loadUserData(); // Load data user dari SharedPreferences
  }

  Future<void> _loadUserData() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _username = prefs.getString('username') ?? 'User';
    });
  }

  Future<void> _logout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('isLoggedIn', false);
    await prefs.remove('username');

    if (mounted) {
      context.go('/'); // Redirect ke login setelah logout
    }
  }
}
```

**Penjelasan**:
- Load data user saat halaman diinisialisasi
- Drawer dengan header yang menampilkan nama user
- Menu navigasi lengkap (Home, Profile, Settings, Logout)
- Bottom navigation bar untuk navigasi cepat
- Fungsi logout yang clear session dan redirect ke login

### `lib/profile_page.dart`
Halaman profil yang menampilkan informasi personal.

```dart
class ProfilePage extends StatefulWidget {
  const ProfilePage({super.key});

  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  // State management untuk halaman profile
}
```

**Penjelasan**:
- Menggunakan StatefulWidget untuk drawer functionality
- Drawer identik dengan home page
- Menampilkan informasi lengkap developer
- Bottom navigation dengan current index = 1 (Profile)

### `lib/settings_page.dart`
Halaman pengaturan dengan berbagai opsi konfigurasi.

```dart
class SettingsPage extends StatefulWidget {
  const SettingsPage({super.key});

  @override
  State<SettingsPage> createState() => _SettingsPageState();
}

class _SettingsPageState extends State<SettingsPage> {
  bool _notificationsEnabled = true;
  bool _darkModeEnabled = false;
  String _language = 'English';

  // UI untuk switch notifications, dark mode, dan language picker
  // Drawer dan bottom navigation seperti halaman lain
}
```

**Penjelasan**:
- State management untuk berbagai setting (notifications, dark mode, language)
- Switch widgets untuk toggle settings
- Dialog untuk pemilihan bahasa
- Snackbar feedback saat menyimpan settings
- Drawer dan bottom navigation konsisten

## Dependencies

### `pubspec.yaml`
```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.8
  shared_preferences: ^2.2.3  # Untuk penyimpanan lokal
  go_router: ^14.2.0         # Untuk routing modern
```

**Penjelasan Dependencies**:
- `shared_preferences`: Menyimpan data sederhana secara lokal (login status, username)
- `go_router`: Routing deklaratif dengan support deep linking dan redirect
- `cupertino_icons`: Icon pack untuk iOS style icons

**Login Credentials**:
   - Username: `admin`
   - Password: `password`

## Local Storage Implementation

```dart
// Simpan status login
await prefs.setBool('isLoggedIn', true);
await prefs.setString('username', _usernameController.text);

// Load data user
final prefs = await SharedPreferences.getInstance();
_username = prefs.getString('username') ?? 'User';

// Logout - clear session
await prefs.setBool('isLoggedIn', false);
await prefs.remove('username');
```
