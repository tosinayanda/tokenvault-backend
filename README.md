# TokenVault Authenticator Backend

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A secure, cross-platform Time-based One-Time Password (TOTP) authenticator application built with React Native. Designed as a portfolio project demonstrating best practices in mobile development, security, and backend integration.

*(Insert Screenshot/GIF Demo Here)*

## Overview

TokenVault Authenticator provides a secure and user-friendly way to manage your two-factor authentication (2FA) accounts. It generates TOTP codes locally on your device, ensuring your secrets remain secure. This project emphasizes secure storage practices and aims for a clean, intuitive user experience.

## Key Features

* **Secure Local Storage:** Utilizes iOS Keychain and Android Keystore for storing sensitive account secrets.
* **TOTP Generation:** Implements RFC 6238 for standard TOTP code generation.
* **QR Code Scanning:** Easily add accounts by scanning QR codes.
* **Manual Account Entry:** Option to add accounts by manually entering details.
* **Biometric Protection:** (Optional) Secure app access with Face ID / Touch ID / Android Biometrics.
* **Cross-Platform:** Built with React Native for both iOS and Android.
* **Minimal Data Collection:** Designed with privacy in mind (see Security section).
* **(Optional) Secure Cloud Backup/Restore:** End-to-end encrypted backup and restore functionality.
* **(Optional) Push-Based Authentication:** Support for push notification approvals (requires backend).

## Technology Stack

* **Mobile:** React Native
* **State Management:** [Your Choice: Redux Toolkit / Zustand / Context API]
* **Navigation:** React Navigation
* **Secure Storage:** `react-native-keychain`
* **QR Scanning:** [`react-native-camera` / `react-native-vision-camera`]
* **Biometrics:** [`react-native-keychain` / `react-native-biometrics`]
* **Cryptography (for Backup):** [`react-native-quick-crypto`]
* **Push Notifications (Optional):** [`react-native-firebase`]
* **Backend (Optional):** [Your Choice: Node.js/Express, Python/Flask/Django, Go, etc.]
* **Database (Optional):** [Your Choice: PostgreSQL, MongoDB, etc.]
* **Cloud Services (Optional):** [AWS/GCP/Azure for hosting, FCM/APNS for push]

## Security Considerations

Security is paramount for an authenticator app. This project addresses security through:

1.  **Secure Credential Storage:** Secrets are stored exclusively within the platform's secure enclave (iOS Keychain / Android Keystore) via `react-native-keychain`. They are **not** stored in AsyncStorage or application memory longer than necessary.
2.  **Biometric Access:** Access to the app can be protected using device biometrics, leveraging platform-native security.
3.  **Minimal Data Collection:** The core application operates offline and collects no user data or analytics by default. Crash reporting, if implemented, is anonymized. No unnecessary permissions are requested.
4.  **(Optional) End-to-End Encrypted Backups:** If the backup feature is enabled, all secrets are encrypted **on the client device** using a key derived from the user's passphrase (e.g., via PBKDF2/Argon2) before being uploaded. The backend service **only stores the encrypted blob** and has no access to the plaintext secrets or the decryption key.
5.  **HTTPS:** All communication with the optional backend services occurs over HTTPS.
6.  **Backend Security (Optional):** Standard security practices are applied to the backend, including input validation, authentication/authorization, and rate limiting.

## Setup and Installation

1.  **Clone the repository:**
    ```bash
    git clone [your-repo-url]
    cd [repo-name]
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    # or
    yarn install
    ```
3.  **Install iOS Pods:**
    ```bash
    cd ios && pod install && cd ..
    ```
4.  **Environment Variables:** Create a `.env` file if required for API keys or backend URLs. See `.env.example`.
5.  **(Backend Setup - Optional):** Follow instructions in the `/backend` directory (if applicable).

## Running the App

* **iOS:**
    ```bash
    npm run ios
    # or
    yarn ios
    ```
* **Android:**
    ```bash
    npm run android
    # or
    yarn android
    ```

## Building for Production

* Refer to the official React Native documentation for building release versions for iOS and Android. Ensure appropriate signing configurations are set up.

## Project Structure

###  DDD Strategic & Tactical Design
a) Domains and Bounded Contexts (Strategic Design)

Applying DDD helps structure the application and potential backend services logically.

Core Domain: The primary value proposition.

Secret Management & Code Generation: This involves securely storing Time-based One-Time Password (TOTP) secrets (keys) associated with user accounts (e.g., Google, GitHub) and generating the correct 6-8 digit codes based on the current time and the secret. This is the absolute heart of the application.
Supporting Subdomains: Areas necessary for the core domain to function but aren't the primary value driver themselves.

Secure Storage Interface: Abstracting the interaction with the platform's secure storage (iOS Keychain, Android Keystore). The core domain uses this interface without needing to know the platform-specific details.
QR Code Parsing: Handling the scanning and interpretation of otpauth:// URIs.
Biometric Access Control: Managing access to the app or secrets using device biometrics.
Generic Subdomains: Off-the-shelf solutions or common problems.

UI Framework (React Native): The rendering engine and component library.
Navigation: Handling screen transitions.
(If Backend Exists) User Authentication: Standard user login/signup for backend services.
Bounded Contexts:

These represent logical boundaries within which a specific domain model applies.

AuthenticatorApp (Mobile Client Context):

Responsibilities: Manages the user interface, interacts with the device's secure storage via the interface, handles QR code scanning, orchestrates TOTP code generation and display, manages biometric prompts.
Model: Account (name, issuer, secret reference), TOTPCode (value, expiry time), SecureStorageKey.
Language: Terms like "Account", "Secret", "Code", "Scan QR".
Initially, this might be the ONLY context for an offline-only app.
AccountBackupService (Backend Context - Optional):

Responsibilities: Handles user authentication (generic), securely receives encrypted blobs from the mobile client, stores these blobs associated with a user, and serves them back upon request. It must not have knowledge of the plaintext secrets or the decryption key.
Model: User, EncryptedBackupBlob, BackupMetadata (timestamp, device info).
Language: "User Account", "Encrypted Backup", "Restore Point".
Interaction: AuthenticatorApp interacts via a defined API (e.g., REST API) for uploading/downloading blobs. Uses an Anti-Corruption Layer (the API client in the app) to translate between its model and the backend's.
PushNotificationService (Backend Context - Optional):

Responsibilities: Manages device push token registration (linking tokens to users), receives authentication requests from trusted external services, sends push notifications via APNS/FCM, receives approval/denial responses from the mobile app, and relays the status back to the initiating service.
Model: DeviceRegistration (userId, pushToken, deviceId), PushAuthRequest (requestId, userId, context, status), ExternalServiceCredentials.
Language: "Push Token", "Device Registration", "Auth Request", "Approve/Deny".
Interaction: AuthenticatorApp registers its token. External services trigger requests. AuthenticatorApp sends responses back. Uses APIs for communication.
b) Main Use Cases (Tactical Design - User Goals)

Based on the contexts:

Within AuthenticatorApp Context (Core Offline Functionality):

Add Account Manually: User provides account details (name, secret key) to store securely.
Add Account via QR Code: User scans a QR code; the app parses it and stores the account securely.
View Current TOTP Codes: User opens the app (potentially after biometric auth) to see the list of accounts and their currently valid codes.
Copy TOTP Code: User taps a code to copy it to the clipboard for pasting.
Delete Account: User removes an account and its associated secret from secure storage.
(Optional) Secure App Access: User enables biometric lock for app entry.
Involving AccountBackupService Context (Optional Feature):

Enable Cloud Backup: User sets up a strong passphrase/recovery key within the app.
Backup Accounts: User triggers a backup; the app encrypts all secrets client-side using the derived key and uploads the blob to the backend.
Restore Accounts: On a new/reset device, the user logs into the backend (if required), enters their passphrase; the app downloads the blob, decrypts it client-side, and restores the accounts.
Involving PushNotificationService Context (Optional Feature):

Register for Push Authentication: User links the app instance to an external service (outside the scope of this app's primary function, but the app needs to handle the registration). The app registers its push token with the PushNotificationService.
Receive & Respond to Push Authentication: User receives a push notification asking to approve/deny a login attempt for an external service. User taps Approve or Deny within the app.

### Project Structure

/
├── src/
│   ├── assets/          # Images, fonts
│   ├── components/      # Reusable UI components
│   ├── constants/       # App constants
│   ├── contexts/        # React Context providers (if used)
│   ├── hooks/           # Custom hooks
│   ├── navigation/      # Navigation stack setup
│   ├── screens/         # Screen components
│   ├── services/        # API clients, utility services (e.g., SecureStorageService)
│   ├── store/           # State management (Redux/Zustand)
│   ├── theme/           # Styling, colors, fonts
│   └── utils/           # Helper functions
├── ios/                 # iOS native project
├── android/             # Android native project
├── backend/             # Optional backend service code
├── .env.example         # Example environment variables
├── Gemfile              # iOS dependencies (CocoaPods)
├── package.json
└── README.md

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
