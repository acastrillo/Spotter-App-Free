# 📱 Spotter iOS App Store Deployment Plan
*Complete roadmap to transform the Spotter fitness app into a production-ready iOS application*

---

## 🎯 Current Status Overview

### Where We Are Now
We have successfully built a sophisticated fitness tracking web application using Next.js 15 with intelligent workout parsing capabilities. The app has evolved from an AI-dependent system to a streamlined, user-controlled workout creation platform.

### Recent Accomplishments
- ✅ **Removed AI dependency** - Replaced OpenAI-based processing with smart algorithmic parsing
- ✅ **Implemented smart workout parser** - Recognizes 90+ exercises, workout structures (ladders, AMRAP, EMOM), and automatically assigns appropriate units
- ✅ **Fixed parsing integration** - Exercise names now properly populate in the editable table
- ✅ **Maintained existing UI/UX** - All visual design and user experience remains intact

### Core Functionality
- **Instagram Import**: Fetches workout captions from Instagram URLs
- **Smart Parsing**: Automatically detects workout formats and exercise types
- **Editable Tables**: Users can manually customize all parsed workout data
- **Workout Library**: Save and manage personal workout collections
- **Calendar View**: Schedule and track workout sessions
- **Settings Management**: User preferences and customization

---

## 🎨 Detailed UI Architecture Analysis
*Preserving the beautiful existing design throughout iOS deployment*

### 🌑 Color Palette & Theme
The app features a sophisticated dark theme optimized for fitness applications:

```css
Colors:
- Background: #0C0F12 (Dark navy - perfect for gym environments)
- Primary: #00D0BD (Cyan accent - energizing and modern)
- Secondary: #7A7EFF (Purple accent - premium feel)
- Text Primary: High contrast white for readability
- Text Secondary: Muted grays for hierarchy
- Surface: Elevated dark cards with subtle borders
```

**Why This Works for iOS:**
- Dark theme reduces eye strain in low-light gym environments
- High contrast ensures accessibility compliance
- Cyan/purple combination is unique in fitness app space
- Colors translate perfectly to iOS dark mode standards

### 📱 Layout & Navigation System
**Current responsive structure that works excellently for mobile:**

#### **Desktop/Tablet Layout:**
- **Header Bar**: Clean horizontal navigation with logo, main nav items, and user actions
- **Sidebar**: Contextual navigation (when needed)
- **Main Content**: Centered max-width containers with proper spacing
- **Grid Systems**: Responsive grid layouts for workout cards and calendar views

#### **Mobile Layout** (Perfect for iOS):
- **Fixed Header**: Stays at top with essential actions and branding
- **Bottom Navigation**: Mobile-first tab bar with 5 primary sections:
  - 🏠 **Home**: Dashboard and recent workouts
  - ➕ **Add**: Import/create workouts (primary action)
  - 📚 **Library**: Saved workout collection
  - 📅 **Calendar**: Schedule and tracking
  - ⚙️ **Settings**: User preferences

#### **Navigation Hierarchy:**
```
Home (Dashboard)
├── Recent Workouts
├── Quick Actions
└── Progress Overview

Add Workout
├── URL Import Tab
├── Manual Entry Tab
└── Edit Workout Flow
    ├── Workout Details (left sidebar)
    └── Exercise Table (main area)

Library
├── Workout Categories
├── Search & Filter
└── Workout Detail Views
    ├── View Mode
    └── Edit Mode

Calendar
├── Monthly View
├── Daily Detail
└── Workout Sessions

Settings
├── User Profile
├── App Preferences
└── Data Management
```

### 🃏 Component System Architecture

#### **Cards & Containers:**
- **Workout Cards**: Elegant elevated cards with rounded corners
- **Info Cards**: Status indicators with icons and clear hierarchy
- **Input Cards**: Form containers with proper focus states
- **Action Cards**: CTA sections with prominent buttons

#### **Tables & Data Display:**
- **Editable Workout Table**: Sophisticated inline editing system
  - Click-to-edit cells with auto-save
  - Dropdown suggestions for exercise names
  - Smart unit detection and formatting
  - Add/remove/duplicate row actions
- **Exercise Rows**: Clean tabular data with proper mobile responsiveness
- **Summary Displays**: Key metrics and workout breakdowns

#### **Forms & Inputs:**
- **Text Inputs**: Clean design with proper focus states
- **Textareas**: Multi-line input with character counters
- **Buttons**: Multiple variants (primary, secondary, outline, destructive)
- **Tabs**: Smooth tab switching for import methods
- **Dropdowns**: Context-aware suggestions and autocomplete

#### **Feedback Systems:**
- **Loading States**: Subtle spinners and skeleton loaders
- **Success States**: Green checkmarks and confirmation messages
- **Error States**: Clear error messaging with recovery options
- **Empty States**: Helpful guidance when no data exists

### 🎛️ Interactive Elements

#### **Micro-interactions:**
- **Hover Effects**: Subtle state changes on interactive elements
- **Focus States**: Clear keyboard navigation indicators
- **Button Press**: Smooth click animations
- **Loading Transitions**: Elegant state changes during processing

#### **Gesture Support** (Ready for iOS):
- **Tap Targets**: Properly sized for finger interaction (44px minimum)
- **Swipe Actions**: Table row actions and navigation
- **Pull to Refresh**: Ready for native iOS implementation
- **Long Press**: Context menus and secondary actions

### 📐 Typography & Spacing
- **Font Stack**: Geist Sans and Geist Mono (modern, readable)
- **Type Scale**: Well-defined hierarchy from headlines to captions
- **Line Heights**: Optimized for readability across device sizes
- **Spacing System**: 8px grid system for consistent layouts

---

## 🏗️ Complete iOS App Store Deployment Plan

### Phase 1: Production App Foundation (3-4 weeks)

#### 🔐 Authentication System Overhaul
**Current Issue**: Mock authentication system will be rejected by Apple App Store
**Solution**: Implement production-ready authentication

**Recommended Tech Stack:**
- **Firebase Authentication** (Google-backed, iOS-optimized)
- **Alternative**: Supabase Auth (open source, PostgreSQL-based)

**Implementation Tasks:**
1. **Remove Mock Authentication** 
   ```typescript
   // Replace current store/index.ts mock system
   // with real authentication provider
   ```

2. **Implement Real Auth Flow**
   - Email/password registration
   - Email verification
   - Password reset functionality
   - Social login options (Google, Apple)

3. **Add Apple Sign-In** (Required for iOS apps with social login)
   - Apple mandates Apple Sign-In if other social options exist
   - Seamless iOS integration

4. **Session Management**
   - Secure token handling
   - Automatic session refresh
   - Proper logout functionality

#### 🗄️ Backend Database Migration
**Current Issue**: localStorage is not production-viable
**Solution**: Cloud-based database with offline sync

**Recommended Architecture:**
- **Database**: Supabase (PostgreSQL) or Firebase Firestore
- **Real-time Sync**: Automatic data synchronization across devices
- **Offline Support**: Local database with conflict resolution

**Data Schema Design:**
```sql
-- Users table
users (
  id uuid primary key,
  email text unique,
  created_at timestamp,
  profile jsonb
)

-- Workouts table  
workouts (
  id uuid primary key,
  user_id uuid references users(id),
  title text,
  description text,
  exercises jsonb,
  tags text[],
  created_at timestamp,
  updated_at timestamp,
  source_url text,
  difficulty text,
  estimated_duration integer
)

-- Workout Sessions table
workout_sessions (
  id uuid primary key,
  user_id uuid references users(id),
  workout_id uuid references workouts(id),
  scheduled_date date,
  completed_date timestamp,
  notes text,
  performance_data jsonb
)
```

**Migration Tasks:**
1. **Set up Supabase project**
2. **Create data migration scripts** from localStorage
3. **Implement data API layer**
4. **Add offline sync capabilities**
5. **Test data consistency and backup**

#### 🔒 Security Implementation
**Apple App Store Security Requirements:**

1. **Data Encryption**
   - Encrypt sensitive data at rest
   - Use HTTPS for all API communications
   - Implement proper API authentication

2. **Privacy Compliance**
   - GDPR compliance for EU users
   - CCPA compliance for California users
   - Clear data collection disclosure

3. **Content Security Policy (CSP)**
   ```typescript
   // next.config.ts additions
   const nextConfig = {
     async headers() {
       return [
         {
           source: '/(.*)',
           headers: [
             {
               key: 'Content-Security-Policy',
               value: "default-src 'self'; script-src 'self' 'unsafe-eval';"
             }
           ]
         }
       ]
     }
   }
   ```

#### 📊 Analytics & Monitoring
**Production Monitoring Stack:**

1. **Error Tracking**: Sentry for crash reporting
2. **Performance Monitoring**: Core Web Vitals tracking
3. **User Analytics**: Privacy-compliant usage analytics
4. **Health Monitoring**: API uptime and response time tracking

### Phase 2: iOS Capacitor Integration (2-3 weeks)

#### ⚙️ Capacitor Setup & Configuration
**Prerequisites:**
- macOS with Xcode 15+ installed
- Apple Developer Account ($99/year)
- iOS Simulator and physical iPhone for testing

**Step 1: Install Capacitor**
```bash
cd spotter-free
npm install @capacitor/core @capacitor/cli
npm install @capacitor/ios
npx cap init "Spotter" "com.yourcompany.spotter"
```

**Step 2: Configure Next.js for Static Export**
```typescript
// next.config.ts modifications
const nextConfig = {
  output: 'export',
  trailingSlash: true,
  images: { 
    unoptimized: true,
    domains: [] // Remove external image domains
  },
  assetPrefix: process.env.NODE_ENV === 'production' ? './' : '',
}
```

**Step 3: Create Capacitor Config**
```typescript
// capacitor.config.ts
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.yourcompany.spotter',
  appName: 'Spotter',
  webDir: 'out',
  bundledWebRuntime: false,
  ios: {
    scheme: 'Spotter',
    contentInset: 'automatic',
  },
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      backgroundColor: '#0C0F12',
      showSpinner: false,
    },
    StatusBar: {
      style: 'light',
      backgroundColor: '#0C0F12'
    }
  }
};

export default config;
```

#### 📱 Essential iOS Plugins Installation
```bash
# Core iOS functionality
npm install @capacitor/status-bar
npm install @capacitor/splash-screen
npm install @capacitor/keyboard
npm install @capacitor/haptics
npm install @capacitor/network
npm install @capacitor/filesystem
npm install @capacitor/share
npm install @capacitor/camera
npm install @capacitor/device

# iOS-specific features
npm install @capacitor/push-notifications
npm install @capacitor-community/health-kit
```

#### 🔧 iOS-Specific Code Integration
**Haptic Feedback Integration:**
```typescript
// lib/ios-features.ts
import { Haptics, ImpactStyle } from '@capacitor/haptics';

export const triggerHaptic = async (style: ImpactStyle = ImpactStyle.Medium) => {
  if (Capacitor.isNativePlatform()) {
    await Haptics.impact({ style });
  }
};

// Usage in buttons
const handleSaveWorkout = async () => {
  await triggerHaptic(ImpactStyle.Light);
  // ... save logic
};
```

**Network Status Monitoring:**
```typescript
// hooks/useNetworkStatus.ts
import { Network } from '@capacitor/network';
import { useState, useEffect } from 'react';

export const useNetworkStatus = () => {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    Network.addListener('networkStatusChange', status => {
      setIsOnline(status.connected);
    });
  }, []);

  return isOnline;
};
```

**iOS Safe Area Handling:**
```css
/* globals.css additions */
.ios-safe-area {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

#### 🏗️ Build Process Setup
```bash
# Build and sync process
npm run build          # Build Next.js static export
npx cap sync           # Sync web assets to native platforms
npx cap open ios       # Open in Xcode
```

### Phase 3: iOS Native Features (2-3 weeks)

#### 🍎 Apple Health Kit Integration
**Why This Matters**: Deep iOS ecosystem integration increases App Store favorability

```bash
npm install @capacitor-community/health-kit
```

**Implementation:**
```typescript
// lib/health-kit.ts
import { HealthKit } from '@capacitor-community/health-kit';

export const syncWorkoutToHealth = async (workout: Workout) => {
  try {
    await HealthKit.requestAuthorization({
      read: ['workouts', 'activeEnergyBurned'],
      write: ['workouts', 'activeEnergyBurned']
    });

    await HealthKit.writeData({
      dataType: 'workouts',
      data: [{
        workoutActivityType: 'functionalStrengthTraining',
        duration: workout.estimatedDuration * 60, // Convert to seconds
        totalEnergyBurned: estimateCalories(workout),
        startDate: new Date(),
        endDate: new Date(Date.now() + workout.estimatedDuration * 60 * 1000)
      }]
    });
  } catch (error) {
    console.error('HealthKit sync failed:', error);
  }
};
```

#### 📢 Push Notifications Setup
**Workout Reminders & Motivation:**

```typescript
// lib/notifications.ts
import { PushNotifications } from '@capacitor/push-notifications';

export const setupPushNotifications = async () => {
  // Request permission
  const result = await PushNotifications.requestPermissions();
  
  if (result.receive === 'granted') {
    await PushNotifications.register();
  }

  // Schedule workout reminders
  PushNotifications.addListener('registration', token => {
    // Send token to backend for targeted notifications
  });
};

export const scheduleWorkoutReminder = async (workout: Workout, date: Date) => {
  // Implementation for local notifications
};
```

#### 📸 Camera Integration for Progress Photos
```typescript
// lib/camera.ts
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';

export const takeProgressPhoto = async () => {
  const image = await Camera.getPhoto({
    quality: 90,
    allowEditing: true,
    resultType: CameraResultType.Uri,
    source: CameraSource.Camera,
  });

  return image.webPath;
};
```

#### 🔗 Native Sharing
```typescript
// lib/sharing.ts
import { Share } from '@capacitor/share';

export const shareWorkout = async (workout: Workout) => {
  await Share.share({
    title: workout.title,
    text: `Check out this ${workout.type} workout I'm doing with Spotter!`,
    url: `https://spotter.app/workout/${workout.id}`,
  });
};
```

### Phase 4: App Store Compliance (2 weeks)

#### 🎨 App Assets Creation
**Required iOS Assets:**

1. **App Icon Set** (all required sizes):
   ```
   App Icon Sizes Needed:
   - 1024×1024 (App Store)
   - 180×180 (iPhone)
   - 167×167 (iPad Pro)
   - 152×152 (iPad)
   - 120×120 (iPhone)
   - 87×87 (iPhone)
   - 80×80 (iPad)
   - 58×58 (iPhone)
   - 40×40 (iPad)
   - 29×29 (iPhone/iPad)
   ```

2. **Launch Screen Assets**:
   - Storyboard-based launch screen (recommended)
   - Various device resolution screenshots

3. **App Store Screenshots** (Required):
   ```
   iPhone Screenshots:
   - 6.7" (iPhone 14 Pro Max): 1290×2796
   - 6.1" (iPhone 14): 1170×2532
   - 5.5" (iPhone 8 Plus): 1242×2208

   iPad Screenshots:
   - 12.9" (iPad Pro): 2048×2732
   - 11" (iPad Pro): 1668×2388
   ```

#### ⚖️ Legal Documentation
**Required for App Store Approval:**

1. **Privacy Policy** (REQUIRED):
   - Data collection practices
   - Third-party integrations
   - User rights and controls
   - Contact information

2. **Terms of Service**:
   - User responsibilities
   - Service availability
   - Limitation of liability
   - Dispute resolution

3. **Age Rating Justification**:
   - Content assessment
   - Feature documentation
   - Parental controls (if applicable)

**Template Structure:**
```markdown
# Privacy Policy for Spotter

## Data We Collect
- Account information (email, profile)
- Workout data (exercises, performance)
- Usage analytics (anonymized)

## How We Use Data
- Provide core app functionality
- Sync data across your devices
- Improve app performance

## Data Sharing
- We do not sell personal data
- Third-party integrations: [list]
- Data retention policies

## Your Rights
- Access your data
- Delete your account
- Export your workouts
```

#### 📝 App Store Listing Optimization
**App Store Connect Configuration:**

1. **App Name**: "Spotter - Smart Workout Builder"
2. **Subtitle**: "Instagram Workouts Made Easy"
3. **Keywords**: "fitness,workout,crossfit,gym,instagram,training,exercise"
4. **Description**:
   ```markdown
   Transform Instagram workout posts into personalized training plans with Spotter's intelligent workout parser.

   KEY FEATURES:
   • Smart Instagram Import - Convert any workout post into editable plans
   • 90+ Exercise Recognition - Automatically detects movements and assigns proper units
   • Workout Structure Detection - Supports AMRAP, EMOM, ladders, and more
   • Apple Health Integration - Sync workouts to your health data
   • Beautiful Dark Interface - Designed for gym environments
   • Offline Support - Access your workouts anywhere

   PERFECT FOR:
   • CrossFit athletes following Instagram programming
   • Personal trainers creating client workouts
   • Fitness enthusiasts tracking diverse training styles
   ```

### Phase 5: Testing & Quality Assurance (2-3 weeks)

#### 🧪 Testing Strategy
**Comprehensive Testing Plan:**

1. **Device Testing Matrix**:
   ```
   Primary Devices:
   - iPhone 15 Pro (latest)
   - iPhone 14 (popular)
   - iPhone SE (small screen)
   - iPad Pro (tablet experience)

   iOS Versions:
   - iOS 17 (latest)
   - iOS 16 (previous major)
   - iOS 15 (minimum supported)
   ```

2. **Functional Testing Checklist**:
   - [ ] Instagram URL parsing works reliably
   - [ ] Exercise table editing functions correctly
   - [ ] Data syncs between devices
   - [ ] Offline mode works as expected
   - [ ] Health Kit integration functions
   - [ ] Push notifications deliver properly
   - [ ] App performance meets standards

3. **Performance Testing**:
   - Memory usage under 50MB baseline
   - Startup time under 3 seconds
   - Network requests optimized
   - Battery usage minimal
   - No memory leaks

#### 🚀 TestFlight Beta Testing
**Beta Testing Program:**

1. **Internal Testing** (1 week):
   - Developer team testing
   - Core functionality validation
   - Performance benchmarking

2. **External Beta** (2 weeks):
   - 25-50 fitness enthusiasts
   - CrossFit gym members
   - Personal trainers
   - Collect feedback and iterate

**TestFlight Setup:**
```bash
# Build for TestFlight
npx cap build ios --prod
# Upload via Xcode or Application Loader
```

#### ✅ App Store Review Preparation
**Common Rejection Reasons to Avoid:**

1. **Functionality Issues**:
   - App crashes on launch
   - Core features don't work
   - Poor network handling

2. **Design Guidelines Violations**:
   - Non-standard UI elements
   - Poor accessibility support
   - Inconsistent navigation

3. **Metadata Issues**:
   - Misleading screenshots
   - Incorrect app description
   - Missing privacy policy

**Pre-Submission Checklist:**
- [ ] App functions without crashes
- [ ] All features work as described
- [ ] Privacy policy is accessible
- [ ] Screenshots accurately represent app
- [ ] Age rating is appropriate
- [ ] Test with poor network conditions
- [ ] Verify Apple Sign-In implementation
- [ ] Check accessibility features

### Phase 6: App Store Submission & Launch (1-2 weeks)

#### 📤 Submission Process
**Step-by-Step Submission:**

1. **Final Build Preparation**:
   ```bash
   # Final production build
   npm run build
   npx cap sync ios
   
   # Open Xcode for final review
   npx cap open ios
   ```

2. **Archive and Upload**:
   - Create archive in Xcode
   - Upload to App Store Connect
   - Wait for processing (can take hours)

3. **App Store Connect Configuration**:
   - Select build for review
   - Complete app information
   - Set pricing (free)
   - Configure availability by country

4. **Submit for Review**:
   - Answer App Store Review questions
   - Provide demo account (if needed)
   - Submit for review

#### ⏰ Review Timeline & Process
**Expected Timeline:**
- **Review Time**: 1-7 days (typically 24-48 hours in 2025)
- **Potential Rejection**: Address issues and resubmit
- **Approval**: App goes live automatically or on your scheduled date

**If Rejected - Common Next Steps:**
1. Read rejection reason carefully
2. Fix specific issues mentioned
3. Test fixes thoroughly
4. Resubmit with resolution notes

#### 🎉 Launch Day Preparation
**Go-Live Checklist:**
- [ ] App Store listing is finalized
- [ ] Backend infrastructure is scaled for users
- [ ] Monitoring and analytics are active
- [ ] Customer support process is ready
- [ ] Marketing materials are prepared
- [ ] Social media announcement ready

---

## 📊 Timeline Summary

| Phase | Duration | Key Deliverables |
|-------|----------|-----------------|
| **Phase 1: Production Foundation** | 3-4 weeks | Real auth, backend database, security |
| **Phase 2: Capacitor Integration** | 2-3 weeks | iOS app shell, basic native features |
| **Phase 3: iOS Native Features** | 2-3 weeks | HealthKit, notifications, camera |
| **Phase 4: App Store Compliance** | 2 weeks | Assets, legal docs, store listing |
| **Phase 5: Testing & QA** | 2-3 weeks | Device testing, TestFlight beta |
| **Phase 6: Submission & Launch** | 1-2 weeks | App Store review, go-live |

**Total Estimated Timeline: 12-17 weeks (3-4 months)**

---

## 💰 Cost Breakdown

### Required Costs:
- **Apple Developer Account**: $99/year
- **Database Hosting**: $0-25/month (Supabase free tier → paid)
- **Backend Hosting**: $0-20/month (Vercel/Netlify)
- **Domain Name**: $10-15/year
- **SSL Certificate**: Free (Let's Encrypt)

### Optional Costs:
- **Error Monitoring**: $0-26/month (Sentry)
- **Analytics**: Free (privacy-compliant solutions)
- **Professional App Icons**: $50-200 (if outsourced)
- **Legal Review**: $500-2000 (privacy policy/terms)

**Total Estimated Monthly Cost: $30-75/month**

---

## 🎯 Success Metrics

### App Store Performance Indicators:
- **Download Rate**: Target 100+ downloads in first month
- **Rating**: Maintain 4.0+ star rating
- **Review Sentiment**: Positive feedback on ease of use
- **Retention**: 60%+ weekly active users

### Technical Performance:
- **Crash Rate**: <0.1% crash-free users
- **Load Time**: <3 seconds app startup
- **API Response**: <500ms average response time
- **Offline Functionality**: Works without internet

### User Engagement:
- **Workout Creation**: Users create 2+ workouts per week
- **Instagram Import**: 70%+ of workouts imported from Instagram
- **Return Usage**: Users return within 7 days of first use

---

## 🔄 Post-Launch Roadmap

### Version 1.1 Features (Month 2-3):
- **Workout Sharing**: Social features within app
- **Video Integration**: Embed exercise demonstration videos
- **Advanced Analytics**: Personal training insights
- **Trainer Mode**: Features for fitness professionals

### Version 1.2 Features (Month 4-6):
- **Apple Watch Integration**: Workout tracking on wrist
- **Siri Shortcuts**: Voice-activated workout starts
- **Widgets**: Home screen quick actions
- **Advanced Parsing**: YouTube, TikTok support

### Long-term Vision (6+ months):
- **Community Features**: User-generated workout sharing
- **AI Coaching**: Personalized workout recommendations
- **Premium Subscription**: Advanced features and analytics
- **Android Version**: Cross-platform expansion

---

## 🚨 Risk Mitigation

### Technical Risks:
- **Instagram API Changes**: Build robust parsing that handles format changes
- **Apple Review Rejection**: Follow guidelines strictly, prepare for iterations
- **Performance Issues**: Conduct thorough testing across devices
- **Data Migration**: Plan careful transition from localStorage to cloud

### Business Risks:
- **Competition**: Focus on unique Instagram parsing differentiator
- **User Adoption**: Ensure onboarding is intuitive and valuable
- **Monetization**: Plan freemium model for sustainability
- **Legal Compliance**: Invest in proper legal documentation

### Contingency Plans:
- **Backup Authentication**: Multiple auth providers ready
- **Alternative Hosting**: Multiple cloud providers evaluated
- **Feature Rollback**: Ability to disable problematic features
- **Emergency Support**: 24/7 monitoring and response capability

---

*This comprehensive plan maintains the beautiful existing UI while transforming Spotter into a production-ready iOS application worthy of the App Store. Every phase is designed to preserve the user experience you love while adding the technical foundation required for success.*

---

## Status Update – 2025-09-11

What’s Done
- Removed mock authentication. Replaced the demo Zustand store with NextAuth Credentials; added `/auth/login` and a global session provider.
- Test user configured via environment variables; local testing instructions provided.
- Calendar: mark complete, schedule future workouts, markers (dot for completed, outlined ring for scheduled), count badges for multiple items, and an Upcoming panel.
- Home: auto-refresh of stats and recent completions after marking/scheduling.
- Streak popup: celebratory toast when a workout is completed.
- Repo hygiene: sanitized `.env.example` and pushed changes to GitHub.

Open Items (Phase 1 targets)
- Production-ready auth: registration, email verification, and password reset; move off env-based credentials.
- Apple Sign-In: required when adding any social providers; recommended for iOS UX.
- Database migration: persist workouts/completions/scheduled to a managed DB instead of localStorage.

Recommended Next Steps (toward App Store)
1) Finalize auth (choose one path):
   - NextAuth + Prisma + managed Postgres (Neon/Supabase). Keep current integration; add DB-backed users, Email (magic link) and/or Credentials, then Apple provider.
   - Supabase Auth. Replace the wrapper with Supabase client auth + Apple provider; manage sessions via Supabase.
2) Migrate data off localStorage: create `users`, `workouts`, `workout_completions`, `scheduled_workouts`; add API routes; one-time import from localStorage on first run.
3) App Store readiness: add Privacy Policy and Terms pages, crash/error monitoring, analytics, and Sign in with Apple (with proper redirect/deep links). If shipping as a web-wrapped app, validate auth callbacks on device (Capacitor).
