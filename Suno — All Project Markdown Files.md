# Suno — All Project Markdown Files

> This file bundles all markdown documents from the Suno project.

---

## File: `README.md`

# Expo Mobile Template

React Native mobile app with **Expo SDK 54**, **TypeScript**, and **React 19**.

**Tech Stack:** React Native 0.81 | Expo Router 6 | NativeWind 4 (Tailwind CSS) | TypeScript 5.9 | react-native-reanimated 4.x

---

## Quick Start

1. **Edit the home screen** — `app/(tabs)/index.tsx` is your app's main entry point
2. **Customize theme** — Update tokens in `theme.config.js` (used by Tailwind + runtime) and app details in `app.config.ts`
3. **Add new screens** — Use `ScreenContainer` component for proper SafeArea handling
4. **Add tab icons** — Map icons in `icon-symbol.tsx` BEFORE using in tabs

---

## Project Structure

```
app/
  _layout.tsx        ← Root layout with providers
  (tabs)/
    _layout.tsx      ← Tab bar configuration
    index.tsx        ← Home screen (EDIT THIS FIRST)
  oauth/             ← Auth callback (don't modify)
components/
  screen-container.tsx ← SafeArea wrapper (USE FOR ALL SCREENS)
  themed-view.tsx    ← View with auto theme background
  ui/
    icon-symbol.tsx  ← Tab bar icon mapping (ADD ICONS HERE FIRST)
constants/
  theme.ts           ← Runtime palette re-export (implemented in lib/_core/theme.ts)
theme.config.js      ← Single palette config (edit tokens here first)
theme.config.d.ts    ← Palette typings (update when adding new keys)
lib/_core/theme.ts   ← Runtime palette builder (shared by Tailwind + useColors)
lib/theme-provider.tsx ← Global theme context (light/dark switch)
lib/_core/nativewind-pressable.ts ← Disables Pressable className to avoid NativeWind pitfalls
hooks/
  use-auth.ts        ← Auth state hook
  use-colors.ts      ← Theme colors hook
  use-color-scheme.ts ← Dark/light mode detection
lib/
  trpc.ts            ← API client
  utils.ts           ← Utility functions (cn)
global.css           ← Tailwind directives
tailwind.config.js   ← Tailwind theme configuration
assets/images/       ← App icons and splash
```

---

## Styling with NativeWind (Tailwind CSS)

This template uses **NativeWind v4** for Tailwind CSS support in React Native.

### Basic Usage

```tsx
import { Text, View } from "react-native";

export function MyComponent() {
  return (
    <View className="flex-1 items-center justify-center p-4">
      <Text className="text-2xl font-bold text-foreground">
        Hello World
      </Text>
      <Text className="mt-2 text-muted">
        Subtitle text
      </Text>
    </View>
  );
}
```

### Available Colors (from `theme.config.js`)

Tokens are defined once in `theme.config.js` and shared by Tailwind + runtime (`useColors()`):

| Token | Usage |
|-------|-------|
| `background` | Screen/page background |
| `foreground` | Primary text |
| `muted` | Secondary text |
| `primary` | Accent/tint color |
| `surface` | Cards/elevated surfaces |
| `border` | Borders/dividers |
| `success` | Success states |
| `warning` | Warning states |
| `error` | Error states |

**Dark mode:** Use color tokens directly (e.g., `text-foreground`, `bg-background`); ThemeProvider + CSS variables switch schemes automatically, no `dark:` prefix needed.

### Layout Tips
- If content may overflow, wrap the whole page in a `ScrollView`; short lists inside can use `.map()`.
- When multiple texts/icons must be inline, set parent `flex-row` (Pressable/TouchableOpacity default to column).
- Pressable className is globally disabled; pass interaction styles via `style`.
- For text inputs that submit on keyboard, set `returnKeyType="done"` (and handle submit) to avoid “Enter does nothing” issues on mobile.

### Combining Classes

Use the `cn()` utility from `@/lib/utils`:

```tsx
import { cn } from "@/lib/utils";

<View className={cn(
  "p-4 rounded-lg",
  isActive && "bg-primary",
  disabled && "opacity-50"
)} />
```

---

## State Management Guidance

- Default: React Context + `useReducer`/`useState` (simpler, fewer pitfalls). Persist with `AsyncStorage`/`MMKV` if needed.
- If you choose Zustand:
  - Selectors must return stable references (no new objects/arrays inside selectors).
  - Subscribe to data, not functions: `useStore((s) => s.state.entries)`; derive with `useMemo`.
  - Why: unstable selectors cause stale renders or render loops.
- For server data, prefer TanStack Query (already included).
- Expo FileSystem (SDK 54+): default to `import * as FileSystem from "expo-file-system/legacy"` to avoid deprecation warnings. If you need the new API, use `import { File } from "expo-file-system/next"` and `await new File(uri).base64()`.
- Provider wiring checklist: whenever you create a new context/provider, import it in `app/_layout.tsx` and wrap the app (outermost or alongside ThemeProvider) before calling any `useXxx` hook.

## Data & API Guidance

- Keep data flow consistent: define shared types/schemas and ensure sender/receiver param names match (e.g., route params, API payloads).
- No mock/placeholder numbers in UI; if data is unavailable, show loading/unknown, not hardcoded values.
- Platform-specific file handling: on iOS, `MediaLibrary.getAssetsAsync()` URIs (ph://) are not readable—use `MediaLibrary.getAssetInfoAsync()` to get `localUri` before reading/uploading.
- Validate end-to-end: API → data transform → navigation params → UI render. Avoid stopping halfway. 

## Screen Layout

### The Problem

React Native screens need to handle:
- Status bar area (notch on iPhone X+)
- Home indicator area (bottom of iPhone X+)
- Tab bar overlap

### The Solution: ScreenContainer

**Always use `ScreenContainer` for screen content:**

```tsx
import { Text } from "react-native";
import { ScreenContainer } from "@/components/screen-container";

export default function MyScreen() {
  return (
    <ScreenContainer className="p-4">
      <Text className="text-2xl font-bold text-foreground">
        Welcome
      </Text>
    </ScreenContainer>
  );
}
```

`ScreenContainer` handles:
- Background color extends behind status bar
- Content stays within safe bounds
- Tab bar area handled correctly

### ScreenContainer Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `className` | string | - | Tailwind classes for content area |
| `edges` | Edge[] | `["top", "left", "right"]` | SafeArea edges to apply |
| `containerClassName` | string | - | Classes for outer background container |

```tsx
// Full-screen modal (needs all edges)
<ScreenContainer edges={["top", "bottom", "left", "right"]}>

// Screen with custom bottom handling
<ScreenContainer edges={["top", "left", "right"]}>
```

---

## Interaction Design

### Priority Order

Build in this order — don't skip to polish before functionality works:

1. **Functionality** — All buttons work, all flows complete, no dead ends
2. **Feedback** — User knows their action was received (press states, loading indicators)
3. **Polish** — Animations and transitions (only if time permits)

### Press Feedback

| Element | Feedback | Implementation |
|---------|----------|----------------|
| Primary buttons | Scale + haptic | `scale: 0.97` + `Haptics.impactAsync(Light)` |
| List items / cards | Opacity | `opacity: 0.7` on press |
| Icons / minor actions | Opacity only | `opacity: 0.6` on press |

```tsx
<Pressable
  onPress={handlePress}
  style={({ pressed }) => [
    styles.button,
    pressed && { transform: [{ scale: 0.97 }], opacity: 0.9 }
  ]}
>
```

### Haptics

Use `expo-haptics` sparingly — overuse diminishes impact:

| Context | Type |
|---------|------|
| Button tap (primary actions) | `impactAsync(ImpactFeedbackStyle.Light)` |
| Toggle / switch | `impactAsync(ImpactFeedbackStyle.Medium)` |
| Success / completion | `notificationAsync(NotificationFeedbackType.Success)` |
| Error / failure | `notificationAsync(NotificationFeedbackType.Error)` |

### Animation (Optional Polish)

Only add animations after core functionality works. Keep them subtle:

```tsx
// ✅ Good: Subtle fade in
withTiming(1, { duration: 250 })

// ✅ Good: Gentle press feedback
withTiming(0.97, { duration: 80 })

// ❌ Bad: Bouncy spring
withSpring(1, { damping: 5 })  // Too bouncy

// ❌ Bad: Dramatic scale
withTiming(0.8, { duration: 200 })  // Too much
```

**Guidelines:**
- Duration: 80-300ms for interactions, up to 400ms for transitions
- Scale changes: 0.95-0.98 range (never below 0.9)
- Prefer `withTiming` with easing over `withSpring`
- Don't animate on mount unless it adds meaning

---

## Native Features

### Audio (expo-audio)

```tsx
import { useAudioPlayer, setAudioModeAsync } from "expo-audio";

// IMPORTANT: Enable playback in iOS silent mode
await setAudioModeAsync({ playsInSilentModeIOS: true });

// Always release player on cleanup
useEffect(() => {
  return () => player.release();
}, []);

// Use real audio sources (no mock/generated placeholders). Track playback state internally for reliable UI:
// type Track = { player: AudioPlayer; volume: number; loop: boolean; isPlaying: boolean };
// track.player.play(); track.isPlaying = true; emit();
// track.player.pause(); track.isPlaying = false; emit();
// const isPlaying = track.isPlaying; // use for UI instead of player.playing (may lag on native)
```

**Getting Free Audio:** Use browser console on [pixabay.com/sound-effects](https://pixabay.com/sound-effects/):
```javascript
// 1) Open a sound page (or search results) on pixabay.com/sound-effects
// 2) Paste this in browser DevTools console to list direct mp3 links
const urls = document.documentElement.innerHTML.match(/https?:\/\/[^"'\s]+\.mp3[^"'\s]*/g) || [];
console.log(urls);
```

### Keep Screen Awake (expo-keep-awake)

```tsx
import { useKeepAwake } from "expo-keep-awake";

// Screen stays on while component is mounted
// Use for: meditation, workout, reading screens
useKeepAwake();
```

### Platform Detection

Disable native-only features on web:

```tsx
import { Platform } from "react-native";

if (Platform.OS !== "web") {
  Haptics.impactAsync(ImpactFeedbackStyle.Light);
}
```

---

## Project Conventions

### Tab Icons
Add mapping in `icon-symbol.tsx` BEFORE using in tabs — otherwise app crashes.

### Data Storage
Prefer `AsyncStorage` for local persistence. Only add backend for cross-device sync.

### Lists
Always use `FlatList` — never `ScrollView` with `.map()`.

### Styles
Use `StyleSheet.create()` outside component, or Tailwind classes. Never inline style objects.

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Broken user flows / dead ends | Verify all flows end-to-end before delivery. Every `onPress` must work. |
| Missing icon mapping | Add to `icon-symbol.tsx` BEFORE using in tabs |
| Text clipped at top/bottom | Ensure `lineHeight > fontSize` (1.2-1.5×) |
| Background gap in dark mode | Use `ScreenContainer` |
| Content under notch | Use `ScreenContainer` |
| Slow list scrolling | Use `FlatList`, never `ScrollView` with `.map()` |
| Styles recreated every render | Use `StyleSheet.create()` outside component |
| iOS crash in gesture callbacks | Gesture handlers run as worklets. Use `.runOnJS(true)` on the gesture, or wrap JS calls with `runOnJS()` |
| Web crash with AnimatedSvg | Don't use `Animated.createAnimatedComponent(Svg)`. Wrap `<Svg>` with `<Animated.View>` instead |
| Gesture not responding | Ensure `GestureHandlerRootView` wraps the app |
| State changes not persisted | Call `saveSettings()` or `AsyncStorage.setItem()` after `setState()` |
| Bottom elements hidden by Tab Bar | Use `useSafeAreaInsets().bottom` or increase `bottom` value |
| **Pressable onPress not firing** | **Never use `className` on Pressable** — always use `style` prop |

### Common Crash Patterns

- **iOS crash on gesture**: Check worklet/JS thread issues (see Common Pitfalls)
- **Web white screen**: Check browser console for errors
- **Android ANR**: Check for blocking operations on main thread

---

## Backend Capabilities

The server provides these **built-in** capabilities (no external API keys required):

| Feature | What It Provides | When To Use |
|---------|------------------|-------------|
| **LLM/AI** | Multimodal AI (text, image, audio) | Image recognition, chat, content generation |
| **User Auth** | OAuth login, session management | User accounts |
| **Database** | PostgreSQL + Drizzle ORM | Cross-device data sync |
| **File Storage** | S3-compatible storage | User-uploaded files |
| **Push Notifications** | Server-side delivery | Notify users of events |

> **Important**: For AI features, use the server's built-in LLM — do NOT ask users for API keys.

See `server/README.md` for implementation details.

---

## Delivery Checklist

Before delivering:

- [ ] All buttons and links work (no empty `onPress` handlers)
- [ ] Core user flows tested end-to-end
- [ ] `app/(tabs)/index.tsx` customized
- [ ] `tailwind.config.js` colors match brand
- [ ] `app.config.ts` app name updated
- [ ] Icon mappings added in `icon-symbol.tsx`
- [ ] No console errors on iOS, Android, and Web

---

## Core File References

Note: All TODO comments are remarks for the agent (you), not for the user.

`components/screen-container.tsx`
```tsx
import { View, type ViewProps } from "react-native";
import { SafeAreaView, type Edge } from "react-native-safe-area-context";

import { cn } from "@/lib/utils";

export interface ScreenContainerProps extends ViewProps {
  /**
   * SafeArea edges to apply. Defaults to ["top", "left", "right"].
   * Bottom is typically handled by Tab Bar.
   */
  edges?: Edge[];
  /**
   * Tailwind className for the content area.
   */
  className?: string;
  /**
   * Additional className for the outer container (background layer).
   */
  containerClassName?: string;
  /**
   * Additional className for the SafeAreaView (content layer).
   */
  safeAreaClassName?: string;
}

/**
 * A container component that properly handles SafeArea and background colors.
 *
 * The outer View extends to full screen (including status bar area) with the background color,
 * while the inner SafeAreaView ensures content is within safe bounds.
 *
 * Usage:
 * ```tsx
 * <ScreenContainer className="p-4">
 *   <Text className="text-2xl font-bold text-foreground">
 *     Welcome
 *   </Text>
 * </ScreenContainer>
 * ```
 */
export function ScreenContainer({
  children,
  edges = ["top", "left", "right"],
  className,
  containerClassName,
  safeAreaClassName,
  style,
  ...props
}: ScreenContainerProps) {
  return (
    <View
      className={cn(
        "flex-1",
        "bg-background",
        containerClassName
      )}
      {...props}
    >
      <SafeAreaView
        edges={edges}
        className={cn("flex-1", safeAreaClassName)}
        style={style}
      >
        <View className={cn("flex-1", className)}>{children}</View>
      </SafeAreaView>
    </View>
  );
}
```

`app/(tabs)/_layout.tsx`
```tsx
import { Tabs } from "expo-router";
import { useSafeAreaInsets } from "react-native-safe-area-context";

import { HapticTab } from "@/components/haptic-tab";
import { IconSymbol } from "@/components/ui/icon-symbol";
import { Platform } from "react-native";
import { useColors } from "@/hooks/use-colors";

export default function TabLayout() {
  const colors = useColors();
  const insets = useSafeAreaInsets();
  const bottomPadding = Platform.OS === "web" ? 12 : Math.max(insets.bottom, 8);
  const tabBarHeight = 56 + bottomPadding;

  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: colors.tint,
        headerShown: false,
        tabBarButton: HapticTab,
        tabBarStyle: {
          paddingTop: 8,
          paddingBottom: bottomPadding,
          height: tabBarHeight,
          backgroundColor: colors.background,
          borderTopColor: colors.border,
          borderTopWidth: 0.5,
        },
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: "Home",
          tabBarIcon: ({ color }) => <IconSymbol size={28} name="house.fill" color={color} />,
        }}
      />
    </Tabs>
  );
}
```

`app/(tabs)/index.tsx`
```tsx
import { ScrollView, Text, View, TouchableOpacity } from "react-native";

import { ScreenContainer } from "@/components/screen-container";

/**
 * Home Screen - NativeWind Example
 *
 * This template uses NativeWind (Tailwind CSS for React Native).
 * You can use familiar Tailwind classes directly in className props.
 *
 * Key patterns:
 * - Use `className` instead of `style` for most styling
 * - Theme colors: use tokens directly (bg-background, text-foreground, bg-primary, etc.); no dark: prefix needed
 * - Responsive: standard Tailwind breakpoints work on web
 * - Custom colors defined in tailwind.config.js
 */
export default function HomeScreen() {
  return (
    <ScreenContainer className="p-6">
      <ScrollView contentContainerStyle={{ flexGrow: 1 }}>
        <View className="flex-1 gap-8">
          {/* Hero Section */}
          <View className="items-center gap-2">
            <Text className="text-4xl font-bold text-foreground">Welcome</Text>
            <Text className="text-base text-muted text-center">
              Edit app/(tabs)/index.tsx to get started
            </Text>
          </View>

          {/* Example Card */}
          <View className="w-full max-w-sm self-center bg-surface rounded-2xl p-6 shadow-sm border border-border">
            <Text className="text-lg font-semibold text-foreground mb-2">NativeWind Ready</Text>
            <Text className="text-sm text-muted leading-relaxed">
              Use Tailwind CSS classes directly in your React Native components.
            </Text>
          </View>

          {/* Example Button */}
          <View className="items-center">
            <TouchableOpacity className="bg-primary px-6 py-3 rounded-full active:opacity-80">
              <Text className="text-background font-semibold">Get Started</Text>
            </TouchableOpacity>
          </View>
        </View>
      </ScrollView>
    </ScreenContainer>
  );
}
```

`lib/utils.ts`
```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

/**
 * Combines class names using clsx and tailwind-merge.
 * This ensures Tailwind classes are properly merged without conflicts.
 *
 * Usage:
 * ```tsx
 * cn("px-4 py-2", isActive && "bg-primary", className)
 * ```
 */
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

`hooks/use-colors.ts`
```tsx
import { Colors, type ColorScheme, type ThemeColorPalette } from "@/constants/theme";
import { useColorScheme } from "./use-color-scheme";

/**
 * Returns the current theme's color palette.
 * Usage: const colors = useColors(); then colors.text, colors.background, etc.
 */
export function useColors(colorSchemeOverride?: ColorScheme): ThemeColorPalette {
  const colorSchema = useColorScheme();
  const scheme = (colorSchemeOverride ?? colorSchema ?? "light") as ColorScheme;
  return Colors[scheme];
}
```

`components/ui/icon-symbol.tsx`
```tsx
// Fallback for using MaterialIcons on Android and web.

import MaterialIcons from "@expo/vector-icons/MaterialIcons";
import { SymbolWeight, SymbolViewProps } from "expo-symbols";
import { ComponentProps } from "react";
import { OpaqueColorValue, type StyleProp, type TextStyle } from "react-native";

type IconMapping = Record<SymbolViewProps["name"], ComponentProps<typeof MaterialIcons>["name"]>;
type IconSymbolName = keyof typeof MAPPING;

/**
 * Add your SF Symbols to Material Icons mappings here.
 * - see Material Icons in the [Icons Directory](https://icons.expo.fyi).
 * - see SF Symbols in the [SF Symbols](https://developer.apple.com/sf-symbols/) app.
 */
const MAPPING = {
  "house.fill": "home",
  "paperplane.fill": "send",
  "chevron.left.forwardslash.chevron.right": "code",
  "chevron.right": "chevron-right",
} as IconMapping;

/**
 * An icon component that uses native SF Symbols on iOS, and Material Icons on Android and web.
 * This ensures a consistent look across platforms, and optimal resource usage.
 * Icon `name`s are based on SF Symbols and require manual mapping to Material Icons.
 */
export function IconSymbol({
  name,
  size = 24,
  color,
  style,
}: {
  name: IconSymbolName;
  size?: number;
  color: string | OpaqueColorValue;
  style?: StyleProp<TextStyle>;
  weight?: SymbolWeight;
}) {
  return <MaterialIcons color={color} size={size} name={MAPPING[name]} style={style} />;
}
```

`tailwind.config.js`
```js
const { themeColors } = require("./theme.config");
const plugin = require("tailwindcss/plugin");

const tailwindColors = Object.fromEntries(
  Object.entries(themeColors).map(([name, swatch]) => [
    name,
    {
      DEFAULT: `var(--color-${name})`,
      light: swatch.light,
      dark: swatch.dark,
    },
  ]),
);

/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: "class",
  // Scan all component and app files for Tailwind classes
  content: ["./app/**/*.{js,ts,tsx}", "./components/**/*.{js,ts,tsx}", "./lib/**/*.{js,ts,tsx}", "./hooks/**/*.{js,ts,tsx}"],

  presets: [require("nativewind/preset")],
  theme: {
    extend: {
      colors: tailwindColors,
    },
  },
  plugins: [
    plugin(({ addVariant }) => {
      addVariant("light", ':root:not([data-theme="dark"]) &');
      addVariant("dark", ':root[data-theme="dark"] &');
    }),
  ],
};
```

`theme.config.js`
```js
/** @type {const} */
const themeColors = {
  primary: { light: '#0a7ea4', dark: '#0a7ea4' },
  background: { light: '#ffffff', dark: '#151718' },
  surface: { light: '#f5f5f5', dark: '#1e2022' },
  foreground: { light: '#11181C', dark: '#ECEDEE' },
  muted: { light: '#687076', dark: '#9BA1A6' },
  border: { light: '#E5E7EB', dark: '#334155' },
  success: { light: '#22C55E', dark: '#4ADE80' },
  warning: { light: '#F59E0B', dark: '#FBBF24' },
  error: { light: '#EF4444', dark: '#F87171' },
};

module.exports = { themeColors };
```

`app.config.ts`
```ts
// Load environment variables with proper priority (system > .env)
import "./scripts/load-env.js";
import type { ExpoConfig } from "expo/config";

// Bundle ID format: space.manus.<project_name_dots>.<timestamp>
// e.g., "my-app" created at 2024-01-15 10:30:45 -> "space.manus.my.app.t20240115103045"
// Bundle ID can only contain letters, numbers, and dots
// Android requires each dot-separated segment to start with a letter
const rawBundleId = "com.app.listeningcoach";
const bundleId =
  rawBundleId
    .replace(/[-_]/g, ".") // Replace hyphens/underscores with dots
    .replace(/[^a-zA-Z0-9.]/g, "") // Remove invalid chars
    .replace(/\.+/g, ".") // Collapse consecutive dots
    .replace(/^\.+|\.+$/g, "") // Trim leading/trailing dots
    .toLowerCase()
    .split(".")
    .map((segment) => {
      // Android requires each segment to start with a letter
      // Prefix with 'x' if segment starts with a digit
      return /^[a-zA-Z]/.test(segment) ? segment : "x" + segment;
    })
    .join(".") || "space.manus.app";
// Extract timestamp from bundle ID and prefix with "manus" for deep link scheme
// e.g., "space.manus.my.app.t20240115103045" -> "manus20240115103045"
const timestamp = bundleId.split(".").pop()?.replace(/^t/, "") ?? "";
const schemeFromBundleId = `manus${timestamp}`;

const env = {
  // App branding - update these values directly (do not use env vars)
  appName: "{{project_title}}",
  appSlug: "listening-coach",
  // S3 URL of the app logo - set this to the URL returned by generate_image when creating custom logo
  // Leave empty to use the default icon from assets/images/icon.png
  logoUrl: "",
  scheme: schemeFromBundleId,
  iosBundleId: bundleId,
  androidPackage: bundleId,
};

const config: ExpoConfig = {
  name: env.appName,
  slug: env.appSlug,
  version: "1.0.0",
  orientation: "portrait",
  icon: "./assets/images/icon.png",
  scheme: env.scheme,
  userInterfaceStyle: "automatic",
  newArchEnabled: true,
  ios: {
    supportsTablet: true,
    bundleIdentifier: env.iosBundleId,
    "infoPlist": {
        "ITSAppUsesNonExemptEncryption": false
      }
  },
  android: {
    adaptiveIcon: {
      backgroundColor: "#E6F4FE",
      foregroundImage: "./assets/images/android-icon-foreground.png",
      backgroundImage: "./assets/images/android-icon-background.png",
      monochromeImage: "./assets/images/android-icon-monochrome.png",
    },
    edgeToEdgeEnabled: true,
    predictiveBackGestureEnabled: false,
    package: env.androidPackage,
    permissions: ["POST_NOTIFICATIONS"],
    intentFilters: [
      {
        action: "VIEW",
        autoVerify: true,
        data: [
          {
            scheme: env.scheme,
            host: "*",
          },
        ],
        category: ["BROWSABLE", "DEFAULT"],
      },
    ],
  },
  web: {
    bundler: "metro",
    output: "static",
    favicon: "./assets/images/favicon.png",
  },
  plugins: [
    "expo-router",
    [
      "expo-audio",
      {
        microphonePermission: "Allow $(PRODUCT_NAME) to access your microphone.",
      },
    ],
    [
      "expo-video",
      {
        supportsBackgroundPlayback: true,
        supportsPictureInPicture: true,
      },
    ],
    [
      "expo-splash-screen",
      {
        image: "./assets/images/splash-icon.png",
        imageWidth: 200,
        resizeMode: "contain",
        backgroundColor: "#ffffff",
        dark: {
          backgroundColor: "#000000",
        },
      },
    ],
    [
      "expo-build-properties",
      {
        android: {
          buildArchs: ["armeabi-v7a", "arm64-v8a"],
          minSdkVersion: 24,
        },
      },
    ],
  ],
  experiments: {
    typedRoutes: true,
    reactCompiler: true,
  },
};

export default config;
```

`package.json`
```json
{
  "name": "app-template",
  "version": "1.0.0",
  "private": true,
  "main": "expo-router/entry",
  "scripts": {
    "dev": "concurrently -k \"pnpm dev:server\" \"pnpm dev:metro\"",
    "dev:server": "cross-env NODE_ENV=development tsx watch server/_core/index.ts",
    "dev:metro": "cross-env EXPO_USE_METRO_WORKSPACE_ROOT=1 npx expo start --web --port ${EXPO_PORT:-8081}",
    "build": "esbuild server/_core/index.ts --platform=node --packages=external --bundle --format=esm --outdir=dist",
    "start": "NODE_ENV=production node dist/index.js",
    "check": "tsc --noEmit",
    "lint": "expo lint",
    "format": "prettier --write .",
    "test": "vitest run",
    "db:push": "drizzle-kit generate && drizzle-kit migrate",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "qr": "node scripts/generate_qr.mjs"
  },
  "dependencies": {
    "@expo/vector-icons": "^15.0.3",
    "@react-native-async-storage/async-storage": "^2.2.0",
    "@react-navigation/bottom-tabs": "^7.8.12",
    "@react-navigation/elements": "^2.9.2",
    "@react-navigation/native": "^7.1.25",
    "@tanstack/react-query": "^5.90.12",
    "@trpc/client": "11.7.2",
    "@trpc/react-query": "11.7.2",
    "@trpc/server": "11.7.2",
    "axios": "^1.13.2",
    "clsx": "^2.1.1",
    "cookie": "^1.1.1",
    "dotenv": "^16.6.1",
    "drizzle-orm": "^0.44.7",
    "expo": "~54.0.29",
    "expo-audio": "~1.1.0",
    "expo-build-properties": "^1.0.10",
    "expo-constants": "~18.0.12",
    "expo-font": "~14.0.10",
    "expo-haptics": "~15.0.8",
    "expo-image": "~3.0.11",
    "expo-keep-awake": "~15.0.8",
    "expo-linking": "~8.0.10",
    "expo-notifications": "~0.32.15",
    "expo-router": "~6.0.19",
    "expo-secure-store": "~15.0.8",
    "expo-splash-screen": "~31.0.12",
    "expo-status-bar": "~3.0.9",
    "expo-symbols": "~1.0.8",
    "expo-system-ui": "~6.0.9",
    "expo-video": "~3.0.15",
    "expo-web-browser": "~15.0.10",
    "express": "^4.22.1",
    "jose": "6.1.0",
    "mysql2": "^3.16.0",
    "nativewind": "^4.2.1",
    "react": "19.1.0",
    "react-dom": "19.1.0",
    "react-native": "0.81.5",
    "react-native-gesture-handler": "~2.28.0",
    "react-native-reanimated": "~4.1.6",
    "react-native-safe-area-context": "~5.6.2",
    "react-native-screens": "~4.16.0",
    "react-native-svg": "15.12.1",
    "react-native-web": "~0.21.2",
    "react-native-worklets": "0.5.1",
    "superjson": "^1.13.3",
    "tailwind-merge": "^2.6.0",
    "zod": "^4.2.1"
  },
  "devDependencies": {
    "@expo/ngrok": "^4.1.3",
    "@types/cookie": "^0.6.0",
    "@types/express": "^4.17.25",
    "@types/node": "^22.19.3",
    "@types/qrcode": "^1.5.6",
    "@types/react": "~19.1.17",
    "concurrently": "^9.2.1",
    "cross-env": "^7.0.3",
    "drizzle-kit": "^0.31.8",
    "esbuild": "^0.25.12",
    "eslint": "^9.39.2",
    "eslint-config-expo": "~10.0.0",
    "prettier": "^3.7.4",
    "qrcode": "^1.5.4",
    "tailwindcss": "^3.4.17",
    "tsx": "^4.21.0",
    "typescript": "~5.9.3",
    "vitest": "^2.1.9"
  },
  "packageManager": "pnpm@9.12.0"
}
```

---

## File: `design.md`

# Listening Coach — Design Document

## Brand & Color Palette

| Token | Light | Dark | Purpose |
|-------|-------|------|---------|
| `primary` | `#1A6BCC` | `#4A9EFF` | CTAs, active states, record button ring |
| `background` | `#F8F9FB` | `#0F1117` | Screen backgrounds |
| `surface` | `#FFFFFF` | `#1C1E26` | Cards, panels |
| `foreground` | `#0D1117` | `#E8ECF0` | Primary text |
| `muted` | `#6B7280` | `#8B95A1` | Secondary text, timestamps |
| `border` | `#E2E8F0` | `#2D3340` | Dividers, card borders |
| `success` | `#16A34A` | `#4ADE80` | Agreement highlights |
| `warning` | `#D97706` | `#FBBF24` | Missed opportunity highlights |
| `error` | `#DC2626` | `#F87171` | Disagreement highlights, recording indicator |

## Screen List

1. **Home / Sessions Screen** — List of past recordings with date, duration, and analysis status
2. **Record Screen** — Live microphone recording with timer, waveform, and file-upload option
3. **Processing Screen** — Upload + transcription + analysis progress indicator
4. **Analysis Results Screen** — Full transcript + three coaching panels
5. **Session Detail Screen** — Re-view a past analysis result

## Primary Content & Functionality

### Home / Sessions Screen
- Header: "Listening Coach" title + settings icon
- Prominent "New Recording" floating action button (bottom-right)
- FlatList of past sessions: date, duration, short summary snippet
- Empty state illustration + CTA when no sessions exist
- Tap a session → Session Detail Screen

### Record Screen
- Large circular record button (center) with pulsing animation when recording
- Timer display (MM:SS) above the button
- Animated waveform bars below the button
- "Upload Audio File" secondary button below waveform
- "Cancel" top-left, "Done / Stop" top-right (or auto-stop on button press)
- Keep-awake enabled during recording

### Processing Screen
- Full-screen centered progress indicator
- Three sequential steps shown: "Uploading…", "Transcribing…", "Analysing…"
- Each step has a checkmark when complete
- Non-dismissable (user must wait)

### Analysis Results Screen
- Sticky header with recording date/duration + "Share" icon
- Scrollable content with four collapsible sections:
  1. **Full Transcript** (default expanded) — plain text with speaker labels if detectable
  2. **Areas of Agreement** (green accent) — bullet points of shared perspectives
  3. **Areas of Disagreement** (red accent) — bullet points of divergent views
  4. **Missed Listening Opportunities** (amber accent) — coaching tips on what was missed
- "New Recording" button at bottom

## Key User Flows

### Record & Analyse
1. User taps "New Recording" on Home
2. Record Screen opens → user taps large record button → recording starts (timer + waveform animate)
3. User taps stop button → Processing Screen appears
4. Upload → Transcribe → Analyse steps complete sequentially
5. Analysis Results Screen slides in automatically
6. User scrolls through transcript and coaching panels

### Upload & Analyse
1. User taps "Upload Audio File" on Record Screen
2. Document picker opens → user selects audio file
3. Same Processing → Results flow as above

### Review Past Session
1. User taps a session card on Home
2. Session Detail Screen shows the same Results layout
3. User can scroll, expand/collapse sections

## Navigation Structure

```
(tabs)/
  index.tsx          ← Home / Sessions list
  
app/
  record.tsx         ← Record Screen (modal push)
  processing.tsx     ← Processing Screen (modal, non-dismissable)
  results.tsx        ← Analysis Results Screen
```

---

## File: `todo.md`

# Listening Coach — TODO

- [x] Generate app icon and update branding
- [x] Update theme colors (primary blue, success green, warning amber, error red)
- [x] Update tab icons in icon-symbol.tsx
- [x] Build Home / Sessions list screen
- [x] Build Record screen with live mic recording, timer, waveform
- [x] Build file upload option on Record screen
- [x] Build Processing screen with step-by-step progress
- [x] Build server tRPC route: upload audio → transcribe → analyse
- [x] Build Analysis Results screen with four collapsible sections
- [x] Persist sessions locally with AsyncStorage
- [x] Build Session Detail screen (re-view past analysis)
- [x] End-to-end flow test
- [x] Polish: animations, haptics, empty states
- [x] Unit tests for server router (upload, transcribe, analyse, error cases)

## New Features (Round 2)

- [x] Rename app to "Suno", new indigo/gold icon
- [x] Speaker labelling: LLM identifies and labels distinct speakers in transcript
- [x] Timestamped audio excerpts: each coaching insight linked to a start/end timestamp
- [x] Audio excerpt playback: tap an insight to hear the relevant audio clip
- [x] Trend dashboard screen: receptiveness score + listening score per session over time
- [x] Score definitions: receptiveness = positive acknowledgements / (positive + dismissive responses); listening = agreements / (agreements + missed opportunities)
- [x] Dashboard: line chart for both scores across sessions
- [x] Dashboard: per-session score cards with date and duration
- [x] Update tab layout to include Dashboard tab

## New Features (Round 3)

- [x] Scrape 23 receptiveness questions from disagreeingbetter.com
- [x] Update receptiveness scoring prompt to use the 23-question framework
- [x] Session naming: user can give each session a custom title before recording
- [x] Conversation context tags: user can tag session type (Work meeting, Personal, Negotiation, etc.)
- [x] Show session title and tag on home screen session cards
- [x] Show session title and tag on results screen header
- [x] Show session title and tag on dashboard score cards

---

## File: `references/receptiveness-questions.md`

# Disagreeing Better — Receptiveness Assessment Questions

Source: https://disagreeingbetter.com/assessments

## Scoring Questions (Q1–Q18)

These 18 questions are used to compute the Receptiveness Score.
Each is answered on a 7-point scale: Strongly Disagree → Strongly Agree.

Questions marked **(R)** are reverse-scored: agreement = lower receptiveness.
Questions marked **(+)** are forward-scored: agreement = higher receptiveness.

| # | Question | Direction |
|---|----------|-----------|
| 1 | Information from people who have strong opinions that oppose mine is often designed to mislead less-informed listeners. | R |
| 2 | I find listening to opposing views informative. | + |
| 3 | People who have views that oppose mine often base their arguments on emotion rather than logic. | R |
| 4 | I value interactions with people who hold strong views opposite to mine. | + |
| 5 | I am willing to have conversations with individuals who hold strong views opposite to my own. | + |
| 6 | I often get annoyed during discussions with people with views that are very different from mine. | R |
| 7 | Some issues are just not up for debate. | R |
| 8 | I am generally curious to find out why other people have different opinions than I do. | + |
| 9 | Some ideas are simply too dangerous to be part of public discourse. | R |
| 10 | People who have views that oppose mine are often biased by what would be best for them and their group. | R |
| 11 | I feel disgusted by some of the things that people with views that oppose mine say. | R |
| 12 | People who have views that oppose mine rarely present compelling arguments. | R |
| 13 | Some points of view are too offensive to be equally represented in the media. | R |
| 14 | Listening to people with views that strongly oppose mine tends to make me angry. | R |
| 15 | I often feel frustrated when I listen to people with social and political views that oppose mine. | R |
| 16 | I consider my views on some issues to be sacred. | R |
| 17 | I like reading well thought-out information and arguments supporting viewpoints opposite to mine. | + |
| 18 | People who have opinions that are opposite to mine often have views which are too extreme to be taken seriously. | R |

## Demographic Questions (Q19–Q23)

These are not used for scoring:
- Q19: Gender
- Q20: Age
- Q21: Political Ideology
- Q22: Education
- Q23: Email (to receive results)

---

## File: `server/README.md`

# Backend Development Guide

This guide covers server-side features including authentication, database, tRPC API, and integrations. **Only read this if your app needs these capabilities.**

---

## When Do You Need Backend?

| Scenario | Backend Needed? | User Auth Required? | Solution |
|----------|-----------------|---------------------|----------|
| Data stays on device only | No | No | Use `AsyncStorage` |
| Data syncs across devices | Yes | Yes | Database + tRPC |
| User accounts / login | Yes | Yes | Manus OAuth |
| AI-powered features | Yes | **Optional** | LLM Integration |
| User uploads files | Yes | **Optional** | S3 Storage |
| Server-side validation | Yes | **Optional** | tRPC procedures |

> **Note:** Backend ≠ User Auth. You can run a backend with LLM/Storage/ImageGen capabilities without requiring user login — just use `publicProcedure` instead of `protectedProcedure`. User auth is only mandatory when you need to identify users or sync user-specific data.

---

## File Structure

```
server/
  db.ts              ← Query helpers (add database functions here)
  routers.ts         ← tRPC procedures (add API routes here)
  storage.ts         ← S3 storage helpers (can extend)
  _core/             ← Framework-level code (don't modify)
drizzle/
  schema.ts          ← Database tables & types (add your tables here)
  relations.ts       ← Table relationships
  migrations/        ← Auto-generated migrations
shared/
  types.ts           ← Shared TypeScript types
  const.ts           ← Shared constants
  _core/             ← Framework-level code (don't modify)
lib/
  trpc.ts            ← tRPC client (can customize headers)
  _core/             ← Framework-level code (don't modify)
hooks/
  use-auth.ts        ← Auth state hook (don't modify)
tests/
  *.test.ts          ← Add your tests here
```

Only touch the files with "←" markers. Anything under `_core/` directories is framework-level—avoid editing unless you are extending the infrastructure.

---

## Authentication

### Overview

The template uses **Manus OAuth** for user authentication. It works differently on native and web:

| Platform | Auth Method | Token Storage |
|----------|-------------|---------------|
| iOS/Android | Bearer token | expo-secure-store |
| Web | HTTP-only cookie | Browser cookie |

### Using the Auth Hook

```tsx
import { useAuth } from "@/hooks/use-auth";

function MyScreen() {
  const { user, isAuthenticated, loading, logout } = useAuth();

  if (loading) return <ActivityIndicator />;
  
  if (!isAuthenticated) {
    return <LoginButton />;
  }

  return (
    <View>
      <ThemedText>Welcome, {user.name}</ThemedText>
      <Button title="Logout" onPress={logout} />
    </View>
  );
}
```

### User Object

The `user` object contains:

```tsx
interface User {
  id: number;
  openId: string;        // Manus OAuth ID
  name: string | null;
  email: string | null;
  loginMethod: string;
  role: "user" | "admin";
  lastSignedIn: Date;
}
```

### Login Flow (Native)

1. User taps Login button
2. `WebBrowser.openAuthSessionAsync()` opens Manus OAuth
3. User authenticates
4. Deep link redirects to `app/oauth/callback.tsx`
5. Callback exchanges code for session token
6. Token stored in SecureStore
7. User redirected to home

### Login Flow (Web)

1. User clicks Login button
2. Browser redirects to Manus OAuth
3. User authenticates
4. Redirect back with session cookie
5. Cookie automatically sent with requests

### Protected Routes

Use `protectedProcedure` in tRPC to require authentication:

```tsx
// server/routers.ts
import { protectedProcedure } from "./_core/trpc";

export const appRouter = router({
  myFeature: router({
    getData: protectedProcedure.query(({ ctx }) => {
      // ctx.user is guaranteed to exist
      return db.getUserData(ctx.user.id);
    }),
  }),
});

```
### Frontend: Handling Auth Errors
protectedProcedure MUST HANDLE UNAUTHORIZED when user is not logged in. Always handle this in the frontend:
```tsx
try {
  await trpc.someProtectedEndpoint.mutate(data);
} catch (error) {
  if (error.data?.code === 'UNAUTHORIZED') {
    router.push('/login');
    return;
  }
  throw error;
}
```

---

## Database

### Schema Definition

Define your tables in `drizzle/schema.ts`:

```tsx
import { int, mysqlTable, text, timestamp, varchar } from "drizzle-orm/mysql-core";

// Users table (already exists)
export const users = mysqlTable("users", {
  id: int("id").autoincrement().primaryKey(),
  openId: varchar("openId", { length: 64 }).notNull().unique(),
  name: text("name"),
  email: varchar("email", { length: 320 }),
  role: mysqlEnum("role", ["user", "admin"]).default("user").notNull(),
  createdAt: timestamp("createdAt").defaultNow().notNull(),
  updatedAt: timestamp("updatedAt").defaultNow().onUpdateNow().notNull(),
});

// Add your tables
export const items = mysqlTable("items", {
  id: int("id").autoincrement().primaryKey(),
  userId: int("userId").notNull(),
  title: varchar("title", { length: 255 }).notNull(),
  description: text("description"),
  completed: boolean("completed").default(false).notNull(),
  createdAt: timestamp("createdAt").defaultNow().notNull(),
});

// Export types
export type User = typeof users.$inferSelect;
export type Item = typeof items.$inferSelect;
export type InsertItem = typeof items.$inferInsert;
```

### Running Migrations

After editing the schema, push changes to the database:

```bash
pnpm db:push
```

This runs `drizzle-kit generate` and `drizzle-kit migrate`.

### Query Helpers

Add database queries in `server/db.ts`:

```tsx
import { eq } from "drizzle-orm";
import { getDb } from "./_core/db";
import { items, InsertItem } from "../drizzle/schema";

export async function getUserItems(userId: number) {
  const db = await getDb();
  if (!db) return [];
  
  return db.select().from(items).where(eq(items.userId, userId));
}

export async function createItem(data: InsertItem) {
  const db = await getDb();
  if (!db) throw new Error("Database not available");
  
  const result = await db.insert(items).values(data);
  return result.insertId;
}

export async function updateItem(id: number, data: Partial<InsertItem>) {
  const db = await getDb();
  if (!db) throw new Error("Database not available");
  
  await db.update(items).set(data).where(eq(items.id, id));
}

export async function deleteItem(id: number) {
  const db = await getDb();
  if (!db) throw new Error("Database not available");
  
  await db.delete(items).where(eq(items.id, id));
}
```

---

## tRPC API

### Adding Routes

Define API routes in `server/routers.ts`:

```tsx
import { z } from "zod";
import { router, protectedProcedure, publicProcedure } from "./_core/trpc";
import * as db from "./db";

export const appRouter = router({
  // Public route (no auth required)
  health: publicProcedure.query(() => ({ status: "ok" })),

  // Protected routes (auth required)
  items: router({
    list: protectedProcedure.query(({ ctx }) => {
      return db.getUserItems(ctx.user.id);
    }),

    create: protectedProcedure
      .input(z.object({
        title: z.string().min(1).max(255),
        description: z.string().optional(),
      }))
      .mutation(({ ctx, input }) => {
        return db.createItem({
          userId: ctx.user.id,
          title: input.title,
          description: input.description,
        });
      }),

    update: protectedProcedure
      .input(z.object({
        id: z.number(),
        title: z.string().min(1).max(255).optional(),
        completed: z.boolean().optional(),
      }))
      .mutation(({ input }) => {
        return db.updateItem(input.id, input);
      }),

    delete: protectedProcedure
      .input(z.object({ id: z.number() }))
      .mutation(({ input }) => {
        return db.deleteItem(input.id);
      }),
  }),
});

export type AppRouter = typeof appRouter;
```

### Calling from Frontend

Use tRPC hooks in your components:

```tsx
import { trpc } from "@/lib/trpc";

function ItemList() {
  // Query
  const { data: items, isLoading, refetch } = trpc.items.list.useQuery();

  // Mutation
  const createMutation = trpc.items.create.useMutation({
    onSuccess: () => refetch(),
  });

  const handleCreate = async () => {
    await createMutation.mutateAsync({
      title: "New Item",
      description: "Description here",
    });
  };

  if (isLoading) return <ActivityIndicator />;

  return (
    <FlatList
      data={items}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => <ItemCard item={item} />}
    />
  );
}
```

### Input Validation

Use Zod schemas for type-safe validation:

```tsx
import { z } from "zod";

const createItemSchema = z.object({
  title: z.string().min(1, "Title required").max(255),
  description: z.string().max(1000).optional(),
  priority: z.enum(["low", "medium", "high"]).default("medium"),
  dueDate: z.date().optional(),
});

// In router
create: protectedProcedure
  .input(createItemSchema)
  .mutation(({ ctx, input }) => {
    // input is fully typed
  }),
```

---

## LLM Integration

Use the preconfigured LLM helpers. Credentials are injected from the platform (no manual setup required).

```ts
import { invokeLLM } from "./server/_core/llm";

/**
 * Simple chat completion
 * type Role = "system" | "user" | "assistant" | "tool" | "function";
 * type TextContent = {
 *   type: "text";
 *   text: string;
 * };
 *
 * type ImageContent = {
 *   type: "image_url";
 *   image_url: {
 *     url: string;
 *     detail?: "auto" | "low" | "high";
 *   };
 * };
 *
 * type FileContent = {
 *   type: "file_url";
 *   file_url: {
 *     url: string;
 *     mime_type?: "audio/mpeg" | "audio/wav" | "application/pdf" | "audio/mp4" | "video/mp4" ;
 *   };
 * };
 *
 * export type Message = {
 *   role: Role;
 *   content: string | Array<ImageContent | TextContent | FileContent>
 * };
 *
 * Supported parameters:
 * messages: Array<{
 *   role: 'system' | 'user' | 'assistant' | 'tool',
 *   content: string | { tool_call: { name: string, arguments: string } }
 * }>
 * tool_choice?: 'none' | 'auto' | 'required' | { type: 'function', function: { name: string } }
 * tools?: Tool[]
 */
const response = await invokeLLM({
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Hello, world!" },
  ],
});
```

Tips
- Always call llm functions from server-side code (e.g., inside tRPC procedures), to avoid exposing your API key.
- You don't need to manually set the model; the helper uses a sensible default.
- LLM responses often contain markdown. Use `<Streamdown>{content}</Streamdown>` (imported from `streamdown`) to render markdown content with proper formatting and streaming support.
- For image-based gen AI workflows, local `file://` and blob URLs don't work. Upload to S3 first, then pass the public URL to `invokeLLM()`.

### Structured Responses (JSON Schema)

Ask the model to return structured JSON via `response_format`:

```ts
import { invokeLLM } from "./server/_core/llm";

const structured = await invokeLLM({
  messages: [
    { role: "system", content: "You are a helpful assistant designed to output JSON." },
    { role: "user", content: "Extract the name and age from the following text: \"My name is Alice and I am 30 years old.\"" },
  ],
  response_format: {
    type: "json_schema",
    json_schema: {
      name: "person_info",
      strict: true,
      schema: {
        type: "object",
        properties: {
          name: { type: "string", description: "The name of the person" },
          age: { type: "integer", description: "The age of the person" },
        },
        required: ["name", "age"],
        additionalProperties: false,
      },
    },
  },
});

// The model responds with JSON content matching the schema.
// Access via `structured.choices[0].message.content` and JSON.parse if needed.
```
The helpers mirror the Python SDK semantics but produce JavaScript-first code, keeping credentials inside the server and ensuring every environment has access to the same token.

**CRITICAL Note:** `json_schema` works for flat structures. For nested arrays/objects, use `json_object` instead.
```ts
const response = await invokeLLM({
  messages: [
    {
      role: "system",
      content: `Analyze the food image. Return JSON:
{
  "foods": [{ "name": "string", "calories": number }],
  "totalCalories": number
}`
    },
    {
      role: "user",
      content: [
        { type: "text", text: "What food is this?" },
        { type: "image_url", image_url: { url: imageUrl } }
      ]
    }
  ],
  response_format: { type: "json_object" }
});
const data = JSON.parse(response.choices[0].message.content);
```

---

## Voice Transcription Integration

Use the preconfigured voice transcription helper that converts speech to text using Whisper API, no manual setup required.

Example usage:
```ts
import { transcribeAudio } from "./server/_core/voiceTranscription";

const result = await transcribeAudio({
  audioUrl: "https://storage.example.com/audio/recording.mp3",
  language: "en", // Optional: helps improve accuracy
  prompt: "Transcribe meeting notes" // Optional: context hint
});

// Returns native Whisper API response
// result.text - Full transcription
// result.language - Detected language (ISO-639-1)
// result.segments - Timestamped segments with metadata
```

Tips
- Accepts URL to pre-uploaded audio file
- 16MB file size limit enforced during transcription, size flag to be set by frontend
- Supported formats: webm, mp3, wav, ogg, m4a
- Returns native Whisper API response with rich metadata
- Frontend should handle audio capture, storage upload, and size validation

---

## Image Generation Integration

Use the preconfigured image generation helper that connects to the internal ImageService, no manual setup required.

Example usage:
```ts
import { generateImage } from "./server/_core/imageGeneration.ts";

const { url: imageUrl } = await generateImage({
  prompt: "A serene landscape with mountains"
});
// For editing:
const { url: imageUrl } = await generateImage({
  prompt: "Add a rainbow to this landscape",
  originalImages: [{
    url: "https://example.com/original.jpg",
    mimeType: "image/jpeg"
  }]
});
```

Tips
- Always call from server-side code (e.g., inside tRPC procedures) to avoid exposing API keys
- Image generation can take 5-20 seconds, implement proper loading states
- Implement proper error handling as image generation can fail

---

## ☁️ File Storage

Use the preconfigured storage helpers in `server/storage.ts`. Credentials are injected from the platform (no manual setup required). Files are stored securely and served via the built-in `/manus-storage/` path — no manual URL management needed.

```ts
import { storagePut } from "./server/storage";

// Upload bytes to storage
const fileKey = `${userId}-files/${fileName}.png`
const { key, url } = await storagePut(
  fileKey,
  fileBuffer, // Buffer | Uint8Array | string
  "image/png"
);
// url = "/manus-storage/{key}" — use directly in frontend code
// key = unique storage key — save in database
```

Tips
- Save the `key` or `url` in your database; use storage for the actual file bytes. This applies to all files including images, documents, and media.
- For file uploads, have the client POST to your server, then call `storagePut` from your backend.
- The returned `url` (e.g. `/manus-storage/...`) is automatically served via signed redirect — no manual URL signing needed.
- To delete a file, drop its `key` from your DB and any UI references — the key is the only way to reach the object, so an unreferenced file is effectively gone. Do not implement a helper to remove the underlying object; the template's storage layer does not expose a delete endpoint.

---

## ☁️ Data API

When you need external data, use the omni_search with search_type = 'api' to see there's any built-in api available in Manus API Hub access. You only have to connect other api if there's no suitable built-in api available.

---

## Owner Notifications

This template already ships with a `notifyOwner({ title, content })` helper (`server/_core/notification.ts`) and a protected tRPC mutation at `trpc.system.notifyOwner`. Use it whenever backend logic needs to push an operational update to the Manus project owner—common triggers are new form submissions, survey feedback, or workflow results.

1. On the server, call `await notifyOwner({ title, content })` or reuse the provided `system.notifyOwner` mutation from jobs/webhooks (`trpc.system.notifyOwner.useMutation()` on the client).
2. Handle the boolean return (`true` on success, `false` if the upstream service is temporarily unavailable) to decide whether you need a fallback channel.

Keep this channel for owner-facing alerts; end-user messaging should flow through your app-specific systems.

---

## Environment Variables

Available environment variables:

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | MySQL/TiDB connection string |
| `JWT_SECRET` | Session signing secret |
| `VITE_APP_ID` | Manus OAuth app ID |
| `OAUTH_SERVER_URL` | Manus OAuth backend URL |
| `VITE_OAUTH_PORTAL_URL` | Manus login portal URL |
| `OWNER_OPEN_ID` | Owner's Manus ID |
| `OWNER_NAME` | Owner's display name |
| `BUILT_IN_FORGE_API_URL` | Manus API endpoint |
| `BUILT_IN_FORGE_API_KEY` | Manus API key |

Expo runtime variables (prefixed with `EXPO_PUBLIC_`):

| Variable | Description |
|----------|-------------|
| `EXPO_PUBLIC_APP_ID` | App ID for OAuth |
| `EXPO_PUBLIC_API_BASE_URL` | API server URL |
| `EXPO_PUBLIC_OAUTH_PORTAL_URL` | Login portal URL |

---

## Testing

Write tests in `tests/` using Vitest:

```tsx
// tests/items.test.ts
import { describe, expect, it } from "vitest";
import { appRouter } from "../server/routers";

describe("items", () => {
  it("creates an item", async () => {
    const ctx = createMockContext({ userId: 1 });
    const caller = appRouter.createCaller(ctx);

    const result = await caller.items.create({
      title: "Test Item",
      description: "Test description",
    });

    expect(result).toBeDefined();
  });
});
```

Run tests:

```bash
pnpm test
```

---

## Key Files Reference

## Core File References

`drizzle/schema.ts`
```ts
import { int, mysqlEnum, mysqlTable, text, timestamp, varchar } from "drizzle-orm/mysql-core";

/**
 * Core user table backing auth flow.
 * Extend this file with additional tables as your product grows.
 * Columns use camelCase to match both database fields and generated types.
 */
export const users = mysqlTable("users", {
  /**
   * Surrogate primary key. Auto-incremented numeric value managed by the database.
   * Use this for relations between tables.
   */
  id: int("id").autoincrement().primaryKey(),
  /** Manus OAuth identifier (openId) returned from the OAuth callback. Unique per user. */
  openId: varchar("openId", { length: 64 }).notNull().unique(),
  name: text("name"),
  email: varchar("email", { length: 320 }),
  loginMethod: varchar("loginMethod", { length: 64 }),
  role: mysqlEnum("role", ["user", "admin"]).default("user").notNull(),
  createdAt: timestamp("createdAt").defaultNow().notNull(),
  updatedAt: timestamp("updatedAt").defaultNow().onUpdateNow().notNull(),
  lastSignedIn: timestamp("lastSignedIn").defaultNow().notNull(),
});

export type User = typeof users.$inferSelect;
export type InsertUser = typeof users.$inferInsert;

// TODO: Add your tables here
```

`server/db.ts`
```ts
import { eq } from "drizzle-orm";
import { drizzle } from "drizzle-orm/mysql2";
import { InsertUser, users } from "../drizzle/schema";
import { ENV } from "./_core/env";

let _db: ReturnType<typeof drizzle> | null = null;

// Lazily create the drizzle instance so local tooling can run without a DB.
export async function getDb() {
  if (!_db && process.env.DATABASE_URL) {
    try {
      _db = drizzle(process.env.DATABASE_URL);
    } catch (error) {
      console.warn("[Database] Failed to connect:", error);
      _db = null;
    }
  }
  return _db;
}

export async function upsertUser(user: InsertUser): Promise<void> {
  if (!user.openId) {
    throw new Error("User openId is required for upsert");
  }

  const db = await getDb();
  if (!db) {
    console.warn("[Database] Cannot upsert user: database not available");
    return;
  }

  try {
    const values: InsertUser = {
      openId: user.openId,
    };
    const updateSet: Record<string, unknown> = {};

    const textFields = ["name", "email", "loginMethod"] as const;
    type TextField = (typeof textFields)[number];

    const assignNullable = (field: TextField) => {
      const value = user[field];
      if (value === undefined) return;
      const normalized = value ?? null;
      values[field] = normalized;
      updateSet[field] = normalized;
    };

    textFields.forEach(assignNullable);

    if (user.lastSignedIn !== undefined) {
      values.lastSignedIn = user.lastSignedIn;
      updateSet.lastSignedIn = user.lastSignedIn;
    }
    if (user.role !== undefined) {
      values.role = user.role;
      updateSet.role = user.role;
    } else if (user.openId === ENV.ownerOpenId) {
      values.role = "admin";
      updateSet.role = "admin";
    }

    if (!values.lastSignedIn) {
      values.lastSignedIn = new Date();
    }

    if (Object.keys(updateSet).length === 0) {
      updateSet.lastSignedIn = new Date();
    }

    await db.insert(users).values(values).onDuplicateKeyUpdate({
      set: updateSet,
    });
  } catch (error) {
    console.error("[Database] Failed to upsert user:", error);
    throw error;
  }
}

export async function getUserByOpenId(openId: string) {
  const db = await getDb();
  if (!db) {
    console.warn("[Database] Cannot get user: database not available");
    return undefined;
  }

  const result = await db.select().from(users).where(eq(users.openId, openId)).limit(1);

  return result.length > 0 ? result[0] : undefined;
}

// TODO: add feature queries here as your schema grows.
```

`server/routers.ts`
```ts
import { COOKIE_NAME } from "../shared/const.js";
import { getSessionCookieOptions } from "./_core/cookies";
import { systemRouter } from "./_core/systemRouter";
import { publicProcedure, router } from "./_core/trpc";

export const appRouter = router({
  // if you need to use socket.io, read and register route in server/_core/index.ts, all api should start with '/api/' so that the gateway can route correctly
  system: systemRouter,
  auth: router({
    me: publicProcedure.query((opts) => opts.ctx.user),
    logout: publicProcedure.mutation(({ ctx }) => {
      const cookieOptions = getSessionCookieOptions(ctx.req);
      ctx.res.clearCookie(COOKIE_NAME, { ...cookieOptions, maxAge: -1 });
      return {
        success: true,
      } as const;
    }),
  }),

  // TODO: add feature routers here, e.g.
  // todo: router({
  //   list: protectedProcedure.query(({ ctx }) =>
  //     db.getUserTodos(ctx.user.id)
  //   ),
  // }),
});

export type AppRouter = typeof appRouter;
```

`server/storage.ts`
```ts
// Preconfigured storage helpers for Manus WebDev templates
// Uploads via Forge Server presigned URL to S3 (PUT direct).
// Downloads return /manus-storage/{key} paths served via 307 redirect.

import { ENV } from "./_core/env";

function getForgeConfig() {
  const forgeUrl = ENV.forgeApiUrl;
  const forgeKey = ENV.forgeApiKey;

  if (!forgeUrl || !forgeKey) {
    throw new Error(
      "Storage config missing: set BUILT_IN_FORGE_API_URL and BUILT_IN_FORGE_API_KEY",
    );
  }

  return { forgeUrl: forgeUrl.replace(/\/+$/, ""), forgeKey };
}

function normalizeKey(relKey: string): string {
  return relKey.replace(/^\/+/, "");
}

function appendHashSuffix(relKey: string): string {
  const hash = crypto.randomUUID().replace(/-/g, "").slice(0, 8);
  const lastDot = relKey.lastIndexOf(".");
  if (lastDot === -1) return `${relKey}_${hash}`;
  return `${relKey.slice(0, lastDot)}_${hash}${relKey.slice(lastDot)}`;
}

export async function storagePut(
  relKey: string,
  data: Buffer | Uint8Array | string,
  contentType = "application/octet-stream",
): Promise<{ key: string; url: string }> {
  const { forgeUrl, forgeKey } = getForgeConfig();
  const key = appendHashSuffix(normalizeKey(relKey));

  // 1. Get presigned PUT URL from Forge
  const presignUrl = new URL("v1/storage/presign/put", forgeUrl + "/");
  presignUrl.searchParams.set("path", key);

  const presignResp = await fetch(presignUrl, {
    headers: { Authorization: `Bearer ${forgeKey}` },
  });

  if (!presignResp.ok) {
    const msg = await presignResp.text().catch(() => presignResp.statusText);
    throw new Error(`Storage presign failed (${presignResp.status}): ${msg}`);
  }

  const { url: s3Url } = (await presignResp.json()) as { url: string };
  if (!s3Url) throw new Error("Forge returned empty presign URL");

  // 2. PUT file directly to S3
  const blob =
    typeof data === "string"
      ? new Blob([data], { type: contentType })
      : new Blob([data as any], { type: contentType });

  const uploadResp = await fetch(s3Url, {
    method: "PUT",
    headers: { "Content-Type": contentType },
    body: blob,
  });

  if (!uploadResp.ok) {
    throw new Error(`Storage upload to S3 failed (${uploadResp.status})`);
  }

  return { key, url: `/manus-storage/${key}` };
}

export async function storageGet(relKey: string): Promise<{ key: string; url: string }> {
  const key = normalizeKey(relKey);
  return { key, url: `/manus-storage/${key}` };
}

export async function storageGetSignedUrl(relKey: string): Promise<string> {
  const { forgeUrl, forgeKey } = getForgeConfig();
  const key = normalizeKey(relKey);

  const getUrl = new URL("v1/storage/presign/get", forgeUrl + "/");
  getUrl.searchParams.set("path", key);

  const resp = await fetch(getUrl, {
    headers: { Authorization: `Bearer ${forgeKey}` },
  });

  if (!resp.ok) {
    const msg = await resp.text().catch(() => resp.statusText);
    throw new Error(`Storage signed URL failed (${resp.status}): ${msg}`);
  }

  const { url } = (await resp.json()) as { url: string };
  return url;
}
```

`lib/trpc.ts`
```ts
import { createTRPCReact } from "@trpc/react-query";
import { httpBatchLink } from "@trpc/client";
import superjson from "superjson";
import type { AppRouter } from "@/server/routers";
import { getApiBaseUrl } from "@/constants/oauth";
import * as Auth from "@/lib/_core/auth";

/**
 * tRPC React client for type-safe API calls.
 *
 * IMPORTANT (tRPC v11): The `transformer` must be inside `httpBatchLink`,
 * NOT at the root createClient level. This ensures client and server
 * use the same serialization format (superjson).
 */
export const trpc = createTRPCReact<AppRouter>();

/**
 * Creates the tRPC client with proper configuration.
 * Call this once in your app's root layout.
 */
export function createTRPCClient() {
  return trpc.createClient({
    links: [
      httpBatchLink({
        url: `${getApiBaseUrl()}/api/trpc`,
        // tRPC v11: transformer MUST be inside httpBatchLink, not at root
        transformer: superjson,
        async headers() {
          const token = await Auth.getSessionToken();
          return token ? { Authorization: `Bearer ${token}` } : {};
        },
        // Custom fetch to include credentials for cookie-based auth
        fetch(url, options) {
          return fetch(url, {
            ...options,
            credentials: "include",
          });
        },
      }),
    ],
  });
}
```

`hooks/use-auth.ts`
```ts
import * as Api from "@/lib/_core/api";
import * as Auth from "@/lib/_core/auth";
import { useCallback, useEffect, useMemo, useState } from "react";
import { Platform } from "react-native";

type UseAuthOptions = {
  autoFetch?: boolean;
};

export function useAuth(options?: UseAuthOptions) {
  const { autoFetch = true } = options ?? {};
  const [user, setUser] = useState<Auth.User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchUser = useCallback(async () => {
    console.log("[useAuth] fetchUser called");
    try {
      setLoading(true);
      setError(null);

      // Web platform: use cookie-based auth, fetch user from API
      if (Platform.OS === "web") {
        console.log("[useAuth] Web platform: fetching user from API...");
        const apiUser = await Api.getMe();
        console.log("[useAuth] API user response:", apiUser);

        if (apiUser) {
          const userInfo: Auth.User = {
            id: apiUser.id,
            openId: apiUser.openId,
            name: apiUser.name,
            email: apiUser.email,
            loginMethod: apiUser.loginMethod,
            lastSignedIn: new Date(apiUser.lastSignedIn),
          };
          setUser(userInfo);
          // Cache user info in localStorage for faster subsequent loads
          await Auth.setUserInfo(userInfo);
          console.log("[useAuth] Web user set from API:", userInfo);
        } else {
          console.log("[useAuth] Web: No authenticated user from API");
          setUser(null);
          await Auth.clearUserInfo();
        }
        return;
      }

      // Native platform: use token-based auth
      console.log("[useAuth] Native platform: checking for session token...");
      const sessionToken = await Auth.getSessionToken();
      console.log(
        "[useAuth] Session token:",
        sessionToken ? `present (${sessionToken.substring(0, 20)}...)` : "missing",
      );
      if (!sessionToken) {
        console.log("[useAuth] No session token, setting user to null");
        setUser(null);
        return;
      }

      // Use cached user info for native (token validates the session)
      const cachedUser = await Auth.getUserInfo();
      console.log("[useAuth] Cached user:", cachedUser);
      if (cachedUser) {
        console.log("[useAuth] Using cached user info");
        setUser(cachedUser);
      } else {
        console.log("[useAuth] No cached user, setting user to null");
        setUser(null);
      }
    } catch (err) {
      const error = err instanceof Error ? err : new Error("Failed to fetch user");
      console.error("[useAuth] fetchUser error:", error);
      setError(error);
      setUser(null);
    } finally {
      setLoading(false);
      console.log("[useAuth] fetchUser completed, loading:", false);
    }
  }, []);

  const logout = useCallback(async () => {
    try {
      await Api.logout();
    } catch (err) {
      console.error("[Auth] Logout API call failed:", err);
      // Continue with logout even if API call fails
    } finally {
      await Auth.removeSessionToken();
      await Auth.clearUserInfo();
      setUser(null);
      setError(null);
    }
  }, []);

  const isAuthenticated = useMemo(() => Boolean(user), [user]);

  useEffect(() => {
    console.log("[useAuth] useEffect triggered, autoFetch:", autoFetch, "platform:", Platform.OS);
    if (autoFetch) {
      if (Platform.OS === "web") {
        // Web: fetch user from API directly (user will login manually if needed)
        console.log("[useAuth] Web: fetching user from API...");
        fetchUser();
      } else {
        // Native: check for cached user info first for faster initial load
        Auth.getUserInfo().then((cachedUser) => {
          console.log("[useAuth] Native cached user check:", cachedUser);
          if (cachedUser) {
            console.log("[useAuth] Native: setting cached user immediately");
            setUser(cachedUser);
            setLoading(false);
          } else {
            // No cached user, check session token
            fetchUser();
          }
        });
      }
    } else {
      console.log("[useAuth] autoFetch disabled, setting loading to false");
      setLoading(false);
    }
  }, [autoFetch, fetchUser]);

  useEffect(() => {
    console.log("[useAuth] State updated:", {
      hasUser: !!user,
      loading,
      isAuthenticated,
      error: error?.message,
    });
  }, [user, loading, isAuthenticated, error]);

  return {
    user,
    loading,
    error,
    isAuthenticated,
    refresh: fetchUser,
    logout,
  };
}
```

`tests/auth.logout.test.ts`
```ts
import { describe, expect, it } from "vitest";
import { appRouter } from "../server/routers";
import { COOKIE_NAME } from "../shared/const";
import type { TrpcContext } from "../server/_core/context";

type CookieCall = {
  name: string;
  options: Record<string, unknown>;
};

type AuthenticatedUser = NonNullable<TrpcContext["user"]>;

function createAuthContext(): { ctx: TrpcContext; clearedCookies: CookieCall[] } {
  const clearedCookies: CookieCall[] = [];
  
  const user: AuthenticatedUser = {
    id: 1,
    openId: "sample-user",
    email: "sample@example.com",
    name: "Sample User",
    loginMethod: "manus",
    role: "user",
    createdAt: new Date(),
    updatedAt: new Date(),
    lastSignedIn: new Date(),
  };
  
  const ctx: TrpcContext = {
    user,
    req: {
      protocol: "https",
      headers: {},
    } as TrpcContext["req"],
    res: {
      clearCookie: (name: string, options: Record<string, unknown>) => {
        clearedCookies.push({ name, options });
      },
    } as TrpcContext["res"],
  };
  
  return { ctx, clearedCookies };
}

// TODO: Remove `.skip` below once you implement user authentication
describe.skip("auth.logout", () => {
  it("clears the session cookie and reports success", async () => {
    const { ctx, clearedCookies } = createAuthContext();
    const caller = appRouter.createCaller(ctx);

    const result = await caller.auth.logout();

    expect(result).toEqual({ success: true });
    expect(clearedCookies).toHaveLength(1);
    expect(clearedCookies[0]?.name).toBe(COOKIE_NAME);
    expect(clearedCookies[0]?.options).toMatchObject({
      maxAge: -1,
      secure: true,
      sameSite: "none",
      httpOnly: true,
      path: "/",
    });
  });
});
```

---

## Common Patterns

### Optimistic Updates

Update UI immediately, revert on error:

```tsx
const toggleComplete = trpc.items.update.useMutation({
  onMutate: async (input) => {
    // Cancel outgoing queries
    await utils.items.list.cancel();
    
    // Snapshot previous value
    const previous = utils.items.list.getData();
    
    // Optimistically update
    utils.items.list.setData(undefined, (old) =>
      old?.map((item) =>
        item.id === input.id
          ? { ...item, completed: input.completed }
          : item
      )
    );
    
    return { previous };
  },
  onError: (err, input, context) => {
    // Revert on error
    utils.items.list.setData(undefined, context?.previous);
  },
  onSettled: () => {
    // Refetch after mutation
    utils.items.list.invalidate();
  },
});
```

### Pagination

```tsx
// Router
list: protectedProcedure
  .input(z.object({
    limit: z.number().min(1).max(100).default(20),
    cursor: z.number().optional(),
  }))
  .query(async ({ ctx, input }) => {
    const items = await db.getItems({
      userId: ctx.user.id,
      limit: input.limit + 1,
      cursor: input.cursor,
    });
    
    let nextCursor: number | undefined;
    if (items.length > input.limit) {
      const next = items.pop();
      nextCursor = next?.id;
    }
    
    return { items, nextCursor };
  }),

// Frontend
const { data, fetchNextPage, hasNextPage } = trpc.items.list.useInfiniteQuery(
  { limit: 20 },
  { getNextPageParam: (lastPage) => lastPage.nextCursor }
);
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Database not available" | Check `DATABASE_URL` is set |
| Auth not working | Verify OAuth callback URL matches |
| tRPC type errors | Run `pnpm check` to verify types |
| Mutations fail silently | Check browser console for errors |
| Session expired | User needs to login again |

