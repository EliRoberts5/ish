# Apple signing for GitHub Actions

This project is already set up to build an unsigned IPA by default, and it will automatically switch to a signed IPA when the Apple signing secrets are populated in the GitHub repository.

## Required GitHub secrets

Add these repository secrets in GitHub:

- APPLE_CERTIFICATE_BASE64
- APPLE_CERTIFICATE_PASSWORD
- APPLE_PROVISIONING_PROFILE_BASE64
- APPLE_TEAM_ID
- APPLE_BUNDLE_ID
- KEYCHAIN_PASSWORD

### App Store Connect API key (for TestFlight uploads)

To automatically upload to TestFlight from GitHub Actions you can use an App Store Connect API key. Add this repository secret:

- `APP_STORE_CONNECT_API_KEY_BASE64`

How to generate the secret:

1. In App Store Connect go to Users and Access → Keys and create a new API Key with "App Manager" or the minimum required privileges.
2. Download the key file (AuthKey_XXXXXX.p8). Note the Key ID and Issuer ID shown in App Store Connect.
3. Create a small JSON file named `appstoreconnect_key.json` with the following structure:

```json
{
	"key_id": "YOUR_KEY_ID",
	"issuer_id": "YOUR_ISSUER_ID",
	"key": "-----BEGIN PRIVATE KEY-----\n...contents of the .p8 file...\n-----END PRIVATE KEY-----\n"
}
```

4. Base64-encode the JSON and add it as the `APP_STORE_CONNECT_API_KEY_BASE64` repository secret:

```bash
base64 -i appstoreconnect_key.json | tr -d '\n'
```

The workflow will decode this secret and use Fastlane to upload the signed `.ipa` to TestFlight.

## How to generate the values

### 1. Create or select an Apple Developer account

Use a paid Apple Developer account and create a new App ID or use an existing one.

### 2. Create a distribution certificate

On a Mac:

1. Open Xcode
2. Open Xcode > Settings > Accounts
3. Sign in with your Apple Developer account
4. Create a distribution certificate or manage certificates
5. Export the certificate as a .p12 file

Then run:

```bash
base64 -i certificate.p12 | tr -d '\n'
```

Use that output as APPLE_CERTIFICATE_BASE64.

Set APPLE_CERTIFICATE_PASSWORD to the password used when exporting the .p12 file.

### 3. Create a provisioning profile

In the Apple Developer portal:

1. Create or select an App ID matching your bundle ID
2. Create a distribution provisioning profile for that App ID
3. Download the .mobileprovision file

Then run:

```bash
base64 -i profile.mobileprovision | tr -d '\n'
```

Use that output as APPLE_PROVISIONING_PROFILE_BASE64.

### 4. Get the Team ID and bundle ID

- APPLE_TEAM_ID: your Apple Developer Team ID
- APPLE_BUNDLE_ID: the Bundle Identifier used by the app, such as app.ish.iSH or a unique custom identifier

The workflow overrides the app bundle identifier with APPLE_BUNDLE_ID when signing is enabled.

### 5. Choose a keychain password

Use any secure temporary password for the GitHub runner keychain, for example:

```text
TemporaryKeychainPass123!
```

Set that value as KEYCHAIN_PASSWORD.

## Notes

- Unsigned builds are still supported and remain the fallback when the signing secrets are missing.
- The workflow will build a signed IPA only when all required Apple signing secrets are present.
- If you are just testing installation on a device, you may still need to register the device in the Apple Developer portal and use a provisioning profile that includes it.
