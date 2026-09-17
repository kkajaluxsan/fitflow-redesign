# FitFlow Frontend

Flutter mobile client (iOS + Android) and Next.js web client.

## Structure
```
frontend/
├── lib/          # Dart source
│   ├── core/     # Theme, routing, networking, DI
│   ├── features/ # dashboard, workout, nutrition, progress, community, settings
│   └── shared/   # Reusable widgets and charts
├── web/          # Next.js 15 web client
└── assets/       # Fonts, icons, design tokens exported from Figma
```

## Commands
```bash
flutter pub get
flutter run                 # Debug on a connected device
flutter test
flutter build apk --release
flutter build ipa --release

cd web && npm install && npm run dev    # Next.js web client
```

## Design source
Screens follow the direction chosen in Lab 03: Variant 3 (Data-Focused & Detailed), on 390 × 844 frames.
