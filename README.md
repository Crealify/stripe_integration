# Flutter Stripe Integration (2026)

<p align="center">
  <img src="https://github.com/Crealify/stripe_integration/blob/Crealify/assets/test_card_information.png" alt="Stripe Test Cards" width="250"/>
</p>

A **production-ready Flutter Stripe integration guide (2026)** with exact file paths, updated Android setup, and clean architecture using Riverpod.

---

## 📌 Overview

This project demonstrates Stripe Payment Sheet integration with:

* `flutter_stripe`
* `dio`
* `flutter_riverpod`

Includes:

* Full Android configuration (latest)
* Payment Intent flow
* Clean folder structure

---

## ⚙️ Installation

```bash
dart pub add flutter_stripe
```

---

## 📋 Requirements (Android)

* Android **API 21+**
* Kotlin **1.8.0+**
* Android Gradle Plugin **8+**
* Gradle **9+**
* Use `FlutterFragmentActivity`
* Use `Theme.AppCompat` or `MaterialComponents`

---

## 🔧 Android Configuration (Step-by-Step with Exact Paths)

---

### 1️⃣ Enable AndroidX

📍 Path:
`android/gradle.properties`

```properties
android.useAndroidX=true
android.enableJetifier=true
```

---

### 2️⃣ Update Plugin Versions

📍 Path:
`android/settings.gradle.kts`

```kotlin
pluginManagement {
    plugins {
        id("com.android.application") version "8.13.2" apply false
        id("org.jetbrains.kotlin.android") version "2.3.10" apply false
    }
}
```

---

### 3️⃣ Update Kotlin JVM Target (NEW WAY)

📍 Path:
`android/app/build.gradle.kts`

```kotlin
kotlin {
    compilerOptions {
        jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_17)
    }
}
```

---

### 4️⃣ Update Gradle Wrapper

📍 Path:
`android/gradle/wrapper/gradle-wrapper.properties`

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-9.3.1-bin.zip
```

---

### 5️⃣ Update Themes (🔥 IMPORTANT — Fix for your error)

📍 Path:
`android/app/src/main/res/values/styles.xml`

```xml
<resources>
    <style name="LaunchTheme" parent="Theme.AppCompat.Light.NoActionBar" />
    <style name="NormalTheme" parent="Theme.MaterialComponents" />
</resources>
```

📍 Path:
`android/app/src/main/res/values-night/styles.xml`

```xml
<resources>
    <style name="LaunchTheme" parent="Theme.AppCompat.DayNight.NoActionBar" />
    <style name="NormalTheme" parent="Theme.MaterialComponents" />
</resources>
```

✔ Fixes:

* `Theme.Light.NoTitleBar not found`
* `Theme.Black.NoTitleBar not found`

---

### 6️⃣ MainActivity Update

📍 Path:
`android/app/src/main/kotlin/com/example/stripe_integration/MainActivity.kt`

```kotlin
import io.flutter.embedding.android.FlutterFragmentActivity

class MainActivity: FlutterFragmentActivity() {
}
```

---

### 7️⃣ Proguard Rules (Manual File)

📍 Path (CREATE THIS FILE):
`android/app/proguard-rules.pro`

```proguard
-dontwarn com.stripe.android.pushProvisioning.PushProvisioningActivity$g
-dontwarn com.stripe.android.pushProvisioning.PushProvisioningActivityStarter$Args
-dontwarn com.stripe.android.pushProvisioning.PushProvisioningActivityStarter$Error
-dontwarn com.stripe.android.pushProvisioning.PushProvisioningActivityStarter
-dontwarn com.stripe.android.pushProvisioning.PushProvisioningEphemeralKeyProvider
-dontwarn kotlinx.parcelize.Parceler$DefaultImpls
-dontwarn kotlinx.parcelize.Parceler
-dontwarn kotlinx.parcelize.Parcelize

-keep class com.stripe.** { *; }
```

---

## 📦 Dependencies

📍 Path:
`pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.8
  flutter_stripe: ^12.6.0
  flutter_riverpod: ^3.3.1
  dio: ^5.9.2
```

---

## 📁 Folder Structure

```
lib/
 ├── core/
 │    └── secret/
 │         └── secret.dart
 ├── stripe_service.dart
 └── main.dart
```

---

## 🔑 Stripe Keys

📍 Path:
`lib/core/secret/secret.dart`

```dart
String publishableKey = "pk_test_XXXXXXXXXXXXXXXXXXXXXXXX";
String secretKey = "                         "; // ⚠️ backend only
```

---

## 🚀 Initialization (CRITICAL FIX FOR YOUR ERROR)
## 🚀 Implementation

### `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_stripe/flutter_stripe.dart';
import 'package:stripe_integration/core/secret.dart';
import 'package:stripe_integration/stripe_service.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  Stripe.publishableKey = publishableKey;// Required
  await Stripe.instance.applySettings();

  print(Stripe.publishableKey); //key access or not

  runApp(
    ProviderScope(
      child: const MaterialApp(
        home: PayScreen(),
      ),
    ),
  );
}
```
✔ Fixes:

StripeConfigException

```

class PayScreen extends ConsumerWidget {
  const PayScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final stripeService = ref.read(stripePaymentProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Stripe Payment')),
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            await stripeService.initPaymentSheet(
              amount: '20',
              currency: 'USD',
              merchantName: 'Flutter Workshop',
            );

            await stripeService.presentPaymentSheet();
          },
          child: const Text('Pay \$20'),
        ),
      ),
    );
  }
}

```
---


### `stripe_service.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:dio/dio.dart';
import 'package:flutter_stripe/flutter_stripe.dart';
import 'package:stripe_integration/core/secret.dart';

final stripePaymentProvider = Provider((ref) => StripePaymentService());

class StripePaymentService {
  final Dio _dio = Dio();

  Future<void> initPaymentSheet({
    required String amount,
    required String currency,
    required String merchantName,
  }) async {
    final paymentIntent = await _createPaymentIntent(amount, currency);

    await Stripe.instance.initPaymentSheet(
      paymentSheetParameters: SetupPaymentSheetParameters(
        paymentIntentClientSecret: paymentIntent['client_secret'],
        merchantDisplayName: merchantName,
        style: ThemeMode.light,
      ),
    );
  }

  Future<void> presentPaymentSheet() async {
    await Stripe.instance.presentPaymentSheet();
  }

  Future<Map<String, dynamic>> _createPaymentIntent(
    String amount,
    String currency,
  ) async {
    final body = {
      'amount': (int.parse(amount) * 100).toString(),
      'currency': currency,
      'payment_method_types[]': 'card',
    };

    final response = await _dio.post(
      'https://api.stripe.com/v1/payment_intents',
      data: body,
      options: Options(
        headers: {
          'Authorization': 'Bearer $secretKey',
          'Content-Type': 'application/x-www-form-urlencoded',
        },
      ),
    );

    print("Stripe Key: ${Stripe.publishableKey}");

    return response.data;
  }
}
```

---

## 💳 Payment Flow

1. Create Payment Intent (API)
2. Initialize Payment Sheet
3. Present Payment Sheet

---

## 💳 Test Cards



<p align="center">
  <img src="https://github.com/Crealify/stripe_integration/blob/Crealify/assets/test_card_information.png" width="250"/>
</p>

* **Card:**
  Do test with above image card information OR use Stripe official test cards

* **Bank:**
  Use Stripe official test bank details

---

## ⚠️ Important Notes

* ❌ Never expose `secretKey` in production
* ✅ Always call `applySettings()`
* 💰 Amount must be in **cents**
* 🔐 Use backend for real payments

---

## 📚 Resources

* Stripe Flutter Docs
* Stripe API Docs

---

## 🤝 Contribution

Feel free to:

* Fork
* Improve
* Add features

---

## ⭐ Support

If you found this helpful, consider
 **Giving A ⭐ To The Project.**


---

## 👨‍💻 Author

**Crealify**

---

<p align="center">
  Built with Flutter & Stripe • Clean Architecture • 2026 Ready
</p>
