# Flutter Theming & Localization Guide

Complete implementation guide for app theming and multilingual support using `easy_localization`.

## Table of Contents
1. [Theming](#theming)
   - [Basic Theme Setup](#basic-theme-setup)
   - [Custom Theme Extensions](#custom-theme-extensions)
2. [Localization with easy_localization](#localization-with-easy_localization)
   - [Setup](#setup)
   - [Translation Files](#translation-files)
   - [Usage](#usage)
3. [Combined Implementation](#combined-implementation)
4. [Best Practices](#best-practices)

---

## Theming

### Basic Theme Setup

```dart
MaterialApp(
  theme: ThemeData.light().copyWith(
    primaryColor: Colors.blue[800],
    cardColor: Colors.white,
  ),
  darkTheme: ThemeData.dark().copyWith(
    primaryColor: Colors.blueGrey[800],
    cardColor: Colors.grey[900],
  ),
  themeMode: ThemeMode.system, // Auto switch based on device
)
```

### Custom Theme Extensions
1. Define extension:
```dart
class AppColors extends ThemeExtension<AppColors> {
  final Color success;
  final Color warning;
  final Color info;

  AppColors({
    required this.success,
    required this.warning,
    required this.info,
  });

  @override
  ThemeExtension<AppColors> copyWith() { /*...*/ }

  @override
  ThemeExtension<AppColors> lerp(ThemeExtension<AppColors>? other, double t) { /*...*/ }
}
```
2. Add to theme:
```dart
theme: ThemeData(
  extensions: <ThemeExtension<dynamic>>[
    AppColors(
      success: Colors.green,
      warning: Colors.amber,
      info: Colors.blue,
    ),
  ],
)
```
3. Usage:
```dart
final colors = Theme.of(context).extension<AppColors>()!;
Container(color: colors.success);
```

---

## Localization with `easy_localization`

### Setup

1. Add dependencies:
```dart
dependencies:
  easy_localization: ^3.0.1
```
2. Initialize in `main.dart`:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await EasyLocalization.ensureInitialized();

  runApp(
    EasyLocalization(
      path: 'assets/translations',
      supportedLocales: [Locale('en'), Locale('ar')],
      fallbackLocale: Locale('en'),
      child: MyApp(),
    ),
  );
}
```

## Translation Files
`assets/translations/en.json`
```json
{
  "welcome": "Welcome",
  "user": {
    "profile": "Profile",
    "settings": "Settings"
  },
  "messages": {
    "notification": "You have {count} new message | You have {count} new messages"
  }
}
```
`assets/translations/ar.json`
```json
{
  "welcome": "مرحبا",
  "user": {
    "profile": "الملف الشخصي",
    "settings": "الإعدادات"
  },
  "messages": {
    "notification": "لديك {count} رسائل جديدة | لديك {count} رسائل جديدة"
  }
}
```

### Usage
```dart
// Simple text
Text('welcome'.tr()),

// Nested keys
Text('user.profile'.tr()),

// Pluralization
Text('messages.notification'.plural(5)),

// Arguments
Text('greeting'.tr(args: ['John'])),

// Context shortcut
Text(context.tr('welcome')),
```

---

## Combined Implementation

### App Configuration
```dart
MaterialApp(
  localizationsDelegates: context.localizationDelegates,
  supportedLocales: context.supportedLocales,
  locale: context.locale,
  theme: AppThemes.lightTheme,
  darkTheme: AppThemes.darkTheme,
  themeMode: settings.themeMode,
)
```

### Settings Getx
```dart
class AppSettings extends GetxController {
  ThemeMode _themeMode = ThemeMode.system;
  Locale _locale = const Locale('en');

  // Getters
  ThemeMode get themeMode => _themeMode;
  Locale get locale => _locale;

  // Setters
  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    update();
  }

  void setLocale(Locale locale) {
    _locale = locale;
    update();
  }
}
```

### Language Switch Widget
```dart
DropdownButton<Locale>(
  value: context.locale,
  items: [
    DropdownMenuItem(
      value: const Locale('en'),
      child: Text('English'),
    ),
    DropdownMenuItem(
      value: const Locale('ar'),
      child: Text('العربية'),
    ),
  ],
  onChanged: (locale) {
    if (locale != null) {
      context.setLocale(locale);
      Getx.of<AppSettings>(context, listen: false).setLocale(locale);
    }
  },
)
```

### Theme Switch Widget
```dart
SwitchListTile(
  title: Text('dark_mode'.tr()),
  value: Getx.of<AppSettings>(context).themeMode == ThemeMode.dark,
  onChanged: (value) {
    Getx.of<AppSettings>(context, listen: false)
      .setThemeMode(value ? ThemeMode.dark : ThemeMode.light);
  },
)
```

---

## Best Practices
### Theming
1. Use `Theme.of(context)` instead of hardcoded values.
2. Create reusable text styles in theme.
3. Test contrast ratios for accessibility.
4. Use theme extensions for custom colors.

### Localization
1. Organize translations by feature
```
translations/
├── auth/
│   ├── en.json
│   └── ar.json
└── profile/
    ├── en.json
    └── ar.json
```
2. Use descriptive key names (auth.login.button).
3. Provide context for translators:
```json
{
  "welcome": "Welcome"
}
```
