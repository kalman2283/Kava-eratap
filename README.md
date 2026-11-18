# Kava Eratap: Android app

Shows every nakamal in Eratap village with live kava status (*Kava i stap*, *Kava i klosap finis*, *Kava i finis*, *Nakamal i klos*). Customers order kava and pay cash in vatu on delivery. Owners update status and handle orders.

The app is a native Android shell (`MainActivity.java`, a WebView) around `app/src/main/assets/index.html`. All phones share data through **Firebase Firestore**, which keeps working offline and syncs when the phone has internet again.

---

## 1. Create the Firebase database (one time, about 10 minutes, free)

1. Go to <https://console.firebase.google.com> → **Add project** → name it e.g. `kava-eratap`. You can turn Analytics off.
2. **Authentication → Get started → Sign-in method:** enable **Anonymous** (customers, no password needed)
   and **Email/Password** (nakamal owners sign in with it).
3. **Build → Firestore Database → Create database.** Pick a location near Vanuatu (e.g. `australia-southeast1`) and start in **production mode**.
4. Firestore → **Rules** tab → paste in everything from `firestore.rules` → **Publish**.
5. **Project settings (⚙️) → Your apps → Web (`</>`)** → register an app → copy the `firebaseConfig` object.

The free Spark plan easily covers a village: 50,000 reads and 20,000 writes per day.

## 2. Add your config

Put the config into `app/src/main/assets/firebase-config.js`:

```js
window.FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "kava-eratap.firebaseapp.com",
  projectId: "kava-eratap",
  storageBucket: "kava-eratap.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

The Firebase web config isn't a secret: the security rules protect the data. If you'd rather keep it out of the repo, save that same text as a GitHub repository secret named `FIREBASE_CONFIG_JS`, and the build will use it.

While the config is `null`, the app runs in **demo mode** with three sample nakamal, and changes stay on one phone.

## 3. Build the APK

### Option A: GitHub builds it for you (no Android tools needed)
1. Create a GitHub repo and push this folder to the `main` branch.
2. The **Build Android APK** workflow (in the **Actions** tab) runs automatically and takes about 5 minutes.
3. Download `KavaEratap.apk` from the repo's **Releases** page. If the repo is public, share that Releases link with villagers so they can download it straight to their phones.

Every push to `main` creates a new release with a higher version number, which installs as an update.

### Option B: Android Studio
Open this folder in Android Studio (Ladybug or newer, JDK 17) → **Build → Build App Bundle(s) / APK(s) → Build APK(s)**.
Command line: `./gradlew assembleRelease` → `app/build/outputs/apk/release/app-release.apk`.

## 4. Install on Android phones

1. Download `KavaEratap.apk` on the phone, or send it by WhatsApp, Bluetooth or USB.
2. Open it. Android will ask to allow installing from this source (Chrome, Files, WhatsApp…) → **Settings → Allow from this source** → back → **Install**.
3. Open **Kava Eratap**. It needs Android 7.0 or newer.

## How owners get started
- Customers never see the owner tools. Owners tap the **🔑** button in the header (or "Nakamal owner? Sign in" under the list) and **Create account** with email + password once, then **Sign in** on any phone.
- Then **Ona blong nakamal** → **Add my nakamal** → enter name, map pin, prices in VT and delivery fee.
- Tap a status button whenever kava changes. Customers see it within seconds.
- New orders appear under **Orders**: Accept → On the way → Delivered (cash received).

## Things to know
- **Owner accounts.** Anyone can create an owner account, but they can only change the nakamal they added themselves. To remove a fake nakamal, delete it in Firebase console → Firestore → `bars`; to block an account, disable it under Authentication → Users.
- **Signing key.** `app/kava-shared.keystore` (password `android`) signs every build so updates install over each other. It's in the repo, so it's fine for village sideloading, but **make a private key before publishing on Google Play** and keep it out of git.
- **Google Play.** To publish there, build an `.aab` (`./gradlew bundleRelease`) with your private key and register a Play developer account (one-time US$25).
- **Notifications.** Owners see new orders when the app is open (a red badge on Orders). Background push alerts would need Firebase Cloud Messaging, a possible next step.

## Project layout
```
app/src/main/assets/index.html          the app (UI + Firestore logic)
app/src/main/assets/firebase-config.js  your Firebase config
app/src/main/java/.../MainActivity.java WebView shell, back button, tel: links
firestore.rules                         database security rules
.github/workflows/build-apk.yml         cloud build → APK on Releases
```
