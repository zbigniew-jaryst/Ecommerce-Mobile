# Ecommerce-Mobile — Przewodnik dla dewelopera

> Prosty opis projektu dla osoby, która widzi go po raz pierwszy.

---

## Czym jest ta aplikacja?

To **mobilna aplikacja e-commerce** zbudowana w React Native + Expo. Nie jest to starter ani szablon — zawiera kompletny, działający sklep internetowy z pełnym przepływem zakupowym: przeglądanie produktów, koszyk, checkout, lista życzeń i profil użytkownika.

Aplikacja **nie łączy się z żadnym backendem**. Wszystkie dane są statyczne (zakodowane na sztywno w plikach TypeScript i JSON). Nie ma logowania, nie ma bazy danych, nie ma żadnych requestów HTTP.

---

## Stos technologiczny

| Technologia                   | Wersja   |
| ----------------------------- | -------- |
| React Native                  | 0.79.1   |
| Expo                          | ^53.0.0  |
| Expo Router (routing)         | ~5.0.2   |
| TypeScript (strict)           | ~5.8.3   |
| React Navigation (dolne taby) | ^7.2.0   |
| React Native Reanimated       | ~3.17.4  |
| Lucide React Native (ikony)   | ^0.475.0 |

---

## Jak uruchomić projekt?

```bash
npm install
npx expo start
```

Następnie:

- **Telefon**: zeskanuj QR kod aplikacją Expo Go
- **Przeglądarka**: wciśnij `w`
- **Android emulator**: wciśnij `a`
- **iOS simulator** (tylko macOS): wciśnij `i`

---

## Struktura katalogów — co gdzie jest?

```
app/                  ← wszystkie ekrany i konfiguracja routingu
  (tabs)/             ← 5 głównych zakładek (dolna nawigacja)
    index.tsx         ← ekran główny (Home)
    products.tsx      ← katalog produktów z wyszukiwarką i filtrami
    cart.tsx          ← koszyk
    wishlist.tsx      ← lista życzeń
    profile.tsx       ← profil użytkownika
  product/[id].tsx    ← szczegóły produktu (dynamiczny URL)
  checkout.tsx        ← proces zakupu
  orders.tsx          ← historia zamówień
  addresses.tsx       ← zarządzanie adresami
  payment-methods.tsx ← metody płatności
  edit-profile.tsx    ← edycja profilu
  settings.tsx        ← ustawienia aplikacji
  help-support.tsx    ← pomoc i kontakt

components/           ← wielokrotnego użytku komponenty UI
  cart/CartItem.tsx
  common/SectionHeader.tsx
  home/BannerCarousel.tsx
  home/CategorySection.tsx
  home/FlashSellSection.tsx
  products/ProductCard.tsx
  products/ProductGrid.tsx

context/
  AppContext.tsx       ← JEDYNE źródło prawdy dla całego stanu aplikacji

data/                 ← statyczne dane (produkty, koszyk)
  products.ts         ← 24 produkty w 3 tablicach
  cart.ts             ← 15 początkowych pozycji w koszyku
  *.json              ← pliki JSON (większość NIE jest importowana w runtime)

types/                ← interfejsy TypeScript
  product.ts
  cart.ts
  user.ts
  banner.ts
  category.ts

hooks/
  useFrameworkReady.ts ← shim dla platformy web
```

---

## Jak działa stan aplikacji?

Cały współdzielony stan aplikacji żyje w **jednym pliku**: `context/AppContext.tsx`.

`AppContext` zarządza czterema domenami:

| Domena            | Co przechowuje                                                             |
| ----------------- | -------------------------------------------------------------------------- |
| **Koszyk**        | lista produktów, łączna kwota, funkcje add/remove/update/clear             |
| **Lista życzeń**  | zapisane produkty, funkcje add/remove/check                                |
| **Produkty**      | tablica produktów, funkcja `getProductById`                                |
| **Wyszukiwanie**  | zapytanie, wyniki (obliczane automatycznie)                                |
| **Responsywność** | wymiary ekranu, flagi `isSmallDevice` / `isMediumDevice` / `isLargeDevice` |

Każdy ekran i komponent, który potrzebuje tych danych, wywołuje hook `useAppContext()`.

Dane płyną **jednostronnie**: `AppContext` → ekrany/komponenty → mutatory → ponowny render.

---

## Routing — jak działa nawigacja?

Projekt używa **Expo Router v5** — routing oparty na strukturze plików (podobnie jak Next.js, ale dla mobile).

Drzewo nawigacji wygląda tak:

```
/ (Stack główny)
├── (tabs)/           ← dolna belka z 5 zakładkami
│   ├── index         → Home
│   ├── products      → Katalog produktów
│   ├── cart          → Koszyk
│   ├── wishlist      → Lista życzeń
│   └── profile       → Profil
├── product/[id]      → Szczegóły produktu
├── checkout          → Checkout
├── orders            → Historia zamówień
├── addresses         → Adresy
├── payment-methods   → Metody płatności
├── edit-profile      → Edycja profilu
├── settings          → Ustawienia
└── help-support      → Pomoc
```

Nawigacja między ekranami odbywa się przez `router.push()`, `router.back()`, `router.replace()` z pakietu `expo-router`. Parametry (np. kategoria, fraza wyszukiwania) są przekazywane przez URL params i odczytywane przez `useLocalSearchParams()`.

---

## Główne przepływy użytkownika

**Zakupy:**
Home → Produkty → Szczegóły produktu → Koszyk → Checkout → powrót do Home

**Profil:**
Profil → (Zamówienia | Adresy | Metody płatności | Edycja profilu | Ustawienia | Pomoc)

---

## Dane w aplikacji

Wszystkie dane są statyczne. Oto co faktycznie jest używane w runtime:

| Plik                   | Zawartość                                    | Używany?                              |
| ---------------------- | -------------------------------------------- | ------------------------------------- |
| `data/products.ts`     | 24 produkty (featured, flash sale, wishlist) | ✅ tak                                |
| `data/cart.ts`         | 15 pozycji w koszyku (stan początkowy)       | ✅ tak                                |
| `data/products.json`   | duplikat danych produktów                    | ❌ nie                                |
| `data/cart.json`       | rozszerzone dane koszyka                     | ❌ nie                                |
| `data/categories.json` | 16 kategorii                                 | ❌ nie (dane są inline w komponencie) |
| `data/banners.json`    | 10 banerów                                   | ❌ nie (dane są inline w komponencie) |
| `data/user.json`       | profil użytkownika                           | ❌ nie (dane są hardcoded w ekranach) |

---

## Kluczowe rzeczy, które warto wiedzieć zanim zaczniesz

1. **`AppContext.tsx` to centrum wszystkiego.** Zmiana interfejsu `AppContextType` wymaga aktualizacji w ~15 plikach. Zanim coś tam zmienisz, sprawdź wszystkich konsumentów.

2. **Dwa komponenty importują dane z pominięciem kontekstu.** `FlashSellSection.tsx` i `products.tsx` importują produkty bezpośrednio z `data/products.ts`, zamiast przez `AppContext`. To celowy skrót, ale oznacza, że zmiany w stanie kontekstu nie wpłyną na te komponenty.

3. **Ekrany drugorzędne mają lokalny stan.** `orders.tsx`, `addresses.tsx`, `payment-methods.tsx`, `edit-profile.tsx` — każdy z nich definiuje własne dane i interfejsy lokalnie, niezależnie od `types/user.ts`. Typy z `types/user.ts` są zdefiniowane, ale nie są używane w runtime.

4. **Brak testów.** W projekcie nie ma żadnych testów jednostkowych ani e2e.

5. **Brak backendu, brak autentykacji.** Wszystkie ekrany są dostępne bez logowania. Nie ma żadnych requestów sieciowych.

6. **TypeScript strict mode jest włączony.** Alias `@/*` mapuje na root projektu.

---

## Powiązany dokument

Szczegółowa analiza architektury dostępna jest w pliku [`ARCHITECTURAL_ANALYSIS_REPORT.md`](./ARCHITECTURAL_ANALYSIS_REPORT.md).
