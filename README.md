# NearByPlaces

An Android application that uses the device's current location and Google Maps Platform to find nearby places. Users can switch between business, store, gas-station, and restaurant searches and view matching places as map markers.

## Features
- Google Maps map view and current-device-location marker.
- Runtime coarse/fine location permission flow.
- Nearby search categories: business, store, gas station, and restaurant.
- Category-specific marker artwork and place names/coordinates in marker details.
- Retrofit service interface for Google Places Nearby Search responses.

## Technology
| Area | Implementation |
|---|---|
| Language / UI | Kotlin, Android Views, ViewBinding, Material BottomNavigationView |
| Maps / location | Google Maps SDK, Fused Location Provider |
| Networking / JSON | Retrofit 2.3.0, Gson converter |
| Minimum Android | API 21 |
| Compile / target SDK | 32 / 32 |
| Gradle wrapper | 7.3.3 |

## Application flow
1. `MapsActivity` inflates `ActivityMapsBinding` and obtains the `SupportMapFragment`.
2. The app checks/request location permission and creates a high-accuracy location request.
3. `FusedLocationProviderClient` supplies location updates. The latest coordinates are stored and the user's map marker is moved.
4. Selecting a bottom-navigation item calls `nearByPlace(type)`.
5. `getUrl()` builds a Google Places Nearby Search URL using current coordinates, a 10,000-metre radius, and the selected place type.
6. `Common.googleApiService` exposes `IGoogleAPIService.getNearbyPlaces()`. Gson maps the response into the models under `Model/`.
7. Each result becomes a marker with a category icon. The map camera moves to the returned place.
8. Location updates are removed when the activity stops.

## Source structure
- `MapsActivity.kt` — map lifecycle, permissions, location updates, category selection, Places request, and marker rendering.
- `Common.kt` — shared Retrofit service access.
- `Remote/IGoogleAPIService.kt` — Retrofit dynamic-URL request.
- `Remote/RetrofitClient.kt` — Retrofit client and Gson converter.
- `Model/` — place, geometry, location, opening-hours, photo, and viewport response models.
- `res/drawable-v24/` — category marker assets.
- `res/menu/bottom_menu.xml` — category navigation.

## Google Maps configuration
The app requires Maps SDK for Android to render the map and Places API for nearby search.

The manifest reads the Maps SDK key from the `MAPS_API_KEY` manifest placeholder. Add that value to root `local.properties` (keep it untracked) and ensure Gradle supplies the placeholder.

**Important:** `MapsActivity.getUrl()` still contains the literal `YOUR_API_KEY` in the Places REST request. Configuring the manifest placeholder alone does not configure this request. Replace it with a securely injected Places key or, preferably, route Places requests through a backend so the key is not shipped in the app. Restrict keys by API and application.

## Run locally
1. Install Android Studio and the Android SDK required by the Gradle configuration.
2. Clone the repository and open its root in Android Studio.
3. Configure the Maps SDK key as `MAPS_API_KEY` in local Gradle configuration.
4. Configure the Places request key separately as described above; do not commit credentials.
5. Enable Maps SDK for Android and Places API in Google Cloud and configure billing/key restrictions.
6. Sync Gradle and run the `app` configuration on a device/emulator with Google Play services and location access.

```bash
./gradlew assembleDebug
```

Windows: `gradlew.bat assembleDebug`. The debug APK is normally under `app/build/outputs/apk/debug/`.

## Permissions and implementation notes
- The manifest declares `ACCESS_COARSE_LOCATION` and `ACCESS_FINE_LOCATION`. Nearby search requires location access.
- Search currently uses a fixed 10 km radius and adds `keyword=cruise`; review this if results seem unexpectedly narrow.
- Mobile-app API keys can be extracted. A backend proxy is the safer production design.
- High-accuracy location updates run at short intervals; review battery impact and lifecycle handling before production.
- The repository contains the Android client only; no backend is included.
- A `PlaceDetails` activity is declared in the manifest, but the primary documented flow is nearby-marker search.
- No explicit license file was present when this README was prepared. Check licensing terms before redistribution.
