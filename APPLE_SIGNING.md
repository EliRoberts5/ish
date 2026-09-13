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
