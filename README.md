# Saster

Disaster-reporting and response coordination system for the barangay, municipal, and provincial pipeline of Ibajay and Kalibo, Aklan.

- **Web portal:** PHP 8.0, Apache, MySQL
- **Mobile app:** Flutter (Android)

This guide explains how to run the system locally and how to build the APK.

---

## Requirements

- [XAMPP](https://www.apachefriends.org/) (PHP 8.0 or newer, Apache, MySQL)
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (only needed to build the mobile app)
- An Android phone or emulator (for the app)

---

## A. Run the Web System

1. Install XAMPP and open the **XAMPP Control Panel**.
2. Click **Start** for both **Apache** and **MySQL**.
3. Copy the project folder to:
   ```
   C:\xampp\htdocs\vsp\sastergpt
   ```
4. Open phpMyAdmin in your browser:
   ```
   http://localhost/phpmyadmin
   ```
5. Click **New** and create a database named `capstone`.
6. Select the `capstone` database, click **Import**, choose the `capstone.sql` file, then click **Go**.
7. Check the database settings in `[CONFIG FILE NAME]`:

   | Setting  | Value       |
   |----------|-------------|
   | Host     | `localhost` |
   | User     | `root`      |
   | Password | (empty)     |
   | Database | `capstone`  |

8. Open the web login page:
   ```
   http://localhost/vsp/sastergpt/public
   ```

---

## B. Build the Mobile App (APK)

1. Open a terminal (Command Prompt or PowerShell) and go to the app folder:
   ```
   cd C:\xampp\htdocs\vsp\sastergpt\flutter\flutter_saster
   ```
2. Download the dependencies:
   ```
   flutter pub get
   ```
3. Find your computer's IP address. Open Command Prompt, type `ipconfig`, and look for the **IPv4 Address** of your Wi-Fi adapter (for example `192.168.1.10`).
4. Open `app_config.dart` and replace the IP address with **your own IP address** (keep the rest of the path):
   ```dart
   class ApiConfig {
     static const String baseUrl = 'http://YOUR_IP_ADDRESS/vsp/sastergpt/api';
     static const String fileBaseUrl = 'http://YOUR_IP_ADDRESS/vsp/sastergpt';
   }
   ```
   - The phone and the computer must be on the same Wi-Fi network.
   - A phone cannot use `localhost` to reach your computer.
   - Windows Firewall may block the connection. If the app cannot connect, allow Apache (port 80) through the firewall.
5. Build the APK:
   ```
   flutter build apk --release
   ```
6. The finished APK is saved at:
   ```
   build\app\outputs\flutter-apk\app-release.apk
   ```
7. Copy the APK to the phone and install it.

---

## Test Login

| Role | Username | Password |
|------|----------|----------|
| `[ROLE]` | `[USERNAME]` | `[PASSWORD]` |

---

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| Page not found | Apache is running and the folder path is correct |
| Database connection error | MySQL is running and the `capstone` database was imported |
| App cannot connect | The IP address is correct and both devices are on the same network |
| Contact the main Developer for more question | bulay og yarn | 
