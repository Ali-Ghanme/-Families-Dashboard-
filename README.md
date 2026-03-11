# 👨‍👩‍👧 Families Dashboard — لوحة إدارة الأسر

A full-stack, **offline-first** admin dashboard for managing family humanitarian data. Built for **administrators and supervisors** to monitor, filter, and export family records collected by field researchers through the companion mobile app.

> 📱 **This dashboard is one part of a two-component system.**
> Field researchers collect data using the **[Families Data Collection App](#-companion-mobile-app--flutter)** (Flutter), which syncs to Supabase. This dashboard then displays, filters, and exports that data.

---

## 🔗 System Overview

```
👷 Field Researcher
        │
        ▼
📱 Flutter App (Data Collection)
   • Registers families offline
   • Syncs to Supabase when online
        │
        ▼
☁️  Supabase (PostgreSQL)
   • Single source of truth
   • Row Level Security per role
        │
        ▼
🖥️  Next.js Dashboard (This Repo)
   • Admins view, filter & export data
   • Works offline from local cache
```

---

## ✨ Features

### 📊 Dashboard
- **11 live stat cards** — total families, children, pregnant, breastfeeding, disabled, chronic diseases, large families (+8), females, infants under 2, elderly 60+, members with chronic conditions
- **Smart filters** — 12 filters including city, housing type, family status, sector, marital status, pregnancy, breastfeeding, chronic diseases, disability, large family, elderly, and infants
- **Advanced filters** panel (collapsible) for less-common fields
- **Real-time search** with 400ms debounce — searches by name, national ID, wife name, and children names
- **Sortable data table** with configurable page sizes (20 / 50 / 100 / 200)
- **Family detail drawer** — full profile view for head, wife, children, and housing info
- **Dark / Light theme** toggle with persistence

### 📤 Export
- **Excel export** — 42 columns across 6 groups with per-column toggle
- **Children report** — filter by sector, age range, gender, with 16 configurable columns
- **Comprehensive export modal** — download entire DB then export families or children

### 🔌 Offline-First Architecture
- **IndexedDB cache** — stores millions of records locally (no localStorage size limits)
- **Cache-first startup** — data loads instantly from cache on every app open, even with no internet
- **Full offline fallback** for the dashboard table, stats, and filter options
- **Local filtering & pagination** — all 12 filters work on cached data without any network call
- **Offline export page** (`/offline`) — dedicated page to sync, filter, and export entirely offline
- **Offline banner** — amber warning bar shows when operating from cache with last-sync date

### 🔐 Authentication & Authorization
- Supabase Auth with email/password
- Role-based access: `super_admin`, `admin`, `field_researcher`
- Settings page (read-only for admins, editable for super admins)
- App config management: maintenance mode, force update, minimum app version

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Next.js 14](https://nextjs.org) — App Router |
| Language | TypeScript 5 |
| Styling | Tailwind CSS 3 |
| Database | [Supabase](https://supabase.com) (PostgreSQL + Row Level Security) |
| Auth | Supabase Auth |
| Local Storage | IndexedDB (via custom `db-cache.ts`) |
| Excel Export | [SheetJS / xlsx](https://sheetjs.com) |
| Icons | [Lucide React](https://lucide.dev) |
| Font | IBM Plex Sans Arabic |
| Deployment | [Vercel](https://vercel.com) |

---

## 📁 Project Structure

```
src/
├── app/
│   ├── dashboard/          # Main dashboard page (table, stats, filters)
│   ├── offline/            # Offline sync & export page
│   ├── settings/           # App config editor
│   ├── login/              # Authentication page
│   └── layout.tsx          # Root layout with AuthProvider + ThemeProvider
│
├── components/
│   ├── dashboard/
│   │   ├── DashboardHeader.tsx       # Actions bar + theme toggle
│   │   ├── StatsSection.tsx          # 11 stat cards
│   │   ├── FiltersSection.tsx        # 12 filters + search
│   │   ├── DataTable.tsx             # Sortable paginated table
│   │   ├── FamilyDetailDrawer.tsx    # Side drawer with full family info
│   │   ├── ExportModal.tsx           # Excel/CSV export with column picker
│   │   ├── ChildrenExportModal.tsx   # Children report with filters
│   │   └── OfflineExportModal.tsx    # Full DB download + export
│   ├── offline/
│   │   ├── SyncButton.tsx            # Download with progress bar
│   │   ├── ColumnPicker.tsx          # Reusable grouped column selector
│   │   ├── ExportFamiliesPanel.tsx   # Families export panel
│   │   └── ExportChildrenPanel.tsx   # Children export with age/gender filters
│   ├── layout/
│   │   └── AppShell.tsx              # RTL sidebar + mobile navigation
│   └── settings/
│       └── AppConfigEditor.tsx       # Maintenance mode & update config
│
├── lib/
│   ├── supabase.ts              # Supabase client
│   ├── families.ts              # Fetch families, stats, filter options
│   ├── auth.ts                  # User profile, roles, sign out
│   ├── export.ts                # Excel/CSV export for online mode
│   ├── db-cache.ts              # IndexedDB: save / load / clear families
│   ├── offline-sync.ts          # Fetch all from Supabase + save to IndexedDB
│   ├── offline-filters.ts       # Filter, compute stats, extract filter options
│   └── offline-export-utils.ts  # Build Excel rows for families & children
│
├── contexts/
│   ├── AuthContext.tsx          # Auth state with onAuthStateChange
│   └── ThemeContext.tsx         # Dark/light theme with localStorage
│
└── types/
    └── index.ts                 # Family, Child, Housing, FilterState, etc.
```

---

## 📱 Companion Mobile App — Flutter

> **Repository:** (https://github.com/Ali-Ghanme/Families-Data-Collection-App)

The Flutter app is used exclusively by **field researchers** to collect family data on-site. It is the data entry point for this entire system.

### Responsibilities
- Field researchers log in with their credentials (role: `field_researcher`)
- They register new families or update existing ones via structured forms
- Data is saved **locally first** and synced to Supabase when internet is available
- Each record carries a `sync_status` (`synced` / `pending` / `error`)
- On startup, the app checks `app_config` in Supabase for maintenance mode or forced updates

### What the app does NOT do
- ❌ No data analysis or reporting
- ❌ No admin controls or user management
- ❌ No bulk export

All of that is handled by **this dashboard**.

---

## 🗄 Database Schema

### `families`
| Column | Type | Description |
|---|---|---|
| `id` | uuid | Primary key |
| `national_id` | text | Family head national ID |
| `sector_name` | text | Geographic sector |
| `head` | jsonb | `{ name, phone, age, job, marital_status, disabilities[], chronic_diseases[] }` |
| `wife` | jsonb | `{ name, phone, age, national_id, is_pregnant, is_nursing, disabilities[], chronic_diseases[] }` |
| `children` | jsonb[] | `[{ name, age, gender, national_id, birth_date, education_level, disabilities[], chronic_diseases[] }]` |
| `housing` | jsonb | `{ housing_type, family_status, original_city, original_address, current_address }` |
| `created_at` | timestamptz | Record creation timestamp |
| `sync_status` | text | Mobile app sync status |

### `profiles`
| Column | Type | Description |
|---|---|---|
| `id` | uuid | FK → `auth.users` |
| `full_name` | text | Display name |
| `phone` | text | Contact number |
| `role` | text | `super_admin` / `admin` / `field_researcher` |

### `app_config`
| Column | Type | Description |
|---|---|---|
| `is_maintenance` | boolean | Maintenance mode flag |
| `maintenance_message` | text | Message shown during maintenance |
| `force_update` | boolean | Force mobile app update |
| `min_app_version` | text | Minimum allowed app version |
| `update_url` | text | App store update URL |

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- A [Supabase](https://supabase.com) project

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/families-dashboard.git
cd families-dashboard

# 2. Install dependencies
npm install

# 3. Configure environment variables
cp .env.local.example .env.local
```

Edit `.env.local`:
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

```bash
# 4. Run the development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## 🔒 Supabase Setup

1. Create the `families`, `profiles`, and `app_config` tables using the schema above
2. Enable Row Level Security (RLS) on all tables
3. Apply the RLS policies from `supabase-rls-fix.sql`
4. Enable Email/Password auth in your Supabase project settings

---

## 🌐 Deployment (Vercel)

```bash
# Build for production
npm run build
```

Or connect your GitHub repository to Vercel and add these environment variables in the Vercel dashboard:

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
```

---

## 📱 Offline Usage

1. Navigate to **التصدير الأوفلاين** (`/offline`) in the sidebar
2. Click **تحميل البيانات** — requires internet once
3. All data is saved to IndexedDB in your browser
4. From now on, every time you open the dashboard:
   - Stats, table, and filters load instantly from cache
   - Full filtering and pagination work with zero network requests
   - Export to Excel works entirely offline

> **Note:** Offline mode requires the initial data sync to have been completed at least once on the same browser/device.

---

## 🎨 Theme

The dashboard supports **Dark** (default) and **Light** modes. The light theme uses a soft blue-tinted background with white cards and stronger accent colors for readability. Toggle with the sun/moon button in the top bar.

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```
