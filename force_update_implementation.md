# How to Implement a Force Update in Flutter

Implementing a force update in Flutter ensures that users always have the latest version of the app. This is useful when you release an update that includes critical bug fixes or new features that need to be adopted immediately.

### Step 1: Add Dependencies

To check for the latest app version and show a prompt to users if an update is required, you can use the `package_info_plus` and `in_app_update` packages.

### 1. Add the dependencies to `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  package_info_plus: ^3.0.0
  in_app_update: ^2.0.2
```

### 2. Install the dependencies:

Run the following command in your terminal:

    flutter pub get

### Step 2: Fetch the App Version Information

The package_info_plus package helps you get the current version of the app installed on the user’s device.

    import 'package:package_info_plus/package_info_plus.dart';

    Future<String> getCurrentVersion() async {
      PackageInfo packageInfo = await PackageInfo.fromPlatform();
      return packageInfo.version; // Returns the app's version
    }

### Step 3: Fetch App Version from PlayStore

To implement the force update, you need to compare the current app version with the latest version available on your backend or a remote source.

    static const String packageName = "<Your App Package Name>";
    static const String playStoreUrl =
        "https://play.google.com/store/apps/details?id=$packageName";

    static Future<String?> fetchPlayStoreVersion() async {
      try {
        final url =
            "https://play.google.com/store/apps/details?id=$packageName&hl=en";
        final response = await http.get(Uri.parse(url));

        if (response.statusCode == 200) {
          final match = RegExp(r'\[\[\["([0-9]+\.[0-9]+\.[0-9]+)"]]')
              .firstMatch(response.body);
          return match?.group(1);
        }
      } catch (e) {
        print("Error fetching Play Store version: $e");
      }
      return null;
    }

### Step 4: Check for Update

To trigger the update when the version is outdated, use the in_app_update package.

    static Future<void> checkForUpdate(BuildContext context) async {
      try {
        final PackageInfo packageInfo = await PackageInfo.fromPlatform();
        final String currentVersion = packageInfo.version;

        final String? playStoreVersion = await fetchPlayStoreVersion();

        print("Installed Version: $currentVersion, Play Store Version: $playStoreVersion");

        if (playStoreVersion != null &&
            playStoreVersion.compareTo(currentVersion) > 0) {
          if (!context.mounted)
            return; // Ensure context is still valid before showing dialog

          showUpdateDialog(context);
        }
      } catch (e) {
        print("Error checking for update: $e");
      }
    }

### Step 5: Combine the Logic

Now, you can combine all the logic to check for updates and trigger the force update:

    import 'dart:convert';
    import 'package:flutter/material.dart';
    import 'package:http/http.dart' as http;
    import 'package:package_info_plus/package_info_plus.dart';
    import 'package:url_launcher/url_launcher.dart';

    class ForceUpgradeService {
      static const String packageName = "com.rms_teacher.app";
      static const String playStoreUrl =
          "https://play.google.com/store/apps/details?id=$packageName";

      /// ✅ Fetch latest version from Play Store
      static Future<String?> fetchPlayStoreVersion() async {
        try {
          final url =
              "https://play.google.com/store/apps/details?id=$packageName&hl=en";
          final response = await http.get(Uri.parse(url));

          if (response.statusCode == 200) {
            final match = RegExp(r'\[\[\["([0-9]+\.[0-9]+\.[0-9]+)"]]')
                .firstMatch(response.body);
            return match?.group(1);
          }
        } catch (e) {
          print("Error fetching Play Store version: $e");
        }
        return null;
      }

      /// ✅ Check for update by comparing versions
      static Future<void> checkForUpdate(BuildContext context) async {
        try {
          final PackageInfo packageInfo = await PackageInfo.fromPlatform();
          final String currentVersion = packageInfo.version;

          final String? playStoreVersion = await fetchPlayStoreVersion();

          print(
              "Installed Version: $currentVersion, Play Store Version: $playStoreVersion");

          if (playStoreVersion != null &&
              playStoreVersion.compareTo(currentVersion) > 0) {
            if (!context.mounted)
              return; // Ensure context is still valid before showing dialog

            showUpdateDialog(context);
          }
        } catch (e) {
          print("Error checking for update: $e");
        }
      }

      /// ✅ Show update dialog
      static void showUpdateDialog(BuildContext context) {
        print("Showing update dialog");
        showDialog(
          context: context,
          barrierDismissible: false,
          builder: (context) => AlertDialog(
            title: Text("Update Required"),
            content: Text("A new version of this app is available. Please update."),
            actions: [
              TextButton(
                onPressed: () {
                  print("Navigating to Play Store");
                  launchUrl(Uri.parse(playStoreUrl),
                      mode: LaunchMode.externalApplication);
                },
                child: Text("Update"),
              ),
            ],
          ),
        );
      }
    }

### Step 5: Implementation

Implement force Upgrade on main or splash screen. In my case, I have checked in splashed screen. If you want you can implement or check directly on main.

    @override
    void initState() {
      super.initState();
      WidgetsBinding.instance.addPostFrameCallback((_) async {
        await ForceUpgradeService.checkForUpdate(context);
      });
    }

### Step 5: Show Alert Box

    @override
    Widget build(BuildContext context) {
      return MaterialApp(
          home: UpgradeAlert(upgrader: Upgrader(), child: const SplashScreen()));
    }

Conclusion
By following these steps, you will ensure that users are always prompted to update to the latest version of your app when necessary.
