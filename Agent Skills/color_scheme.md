# 🎨 Color System & Design Specifications

Based on `docs/UI_design.md`

## 🎨 Brand Identity: "Modern SaaS"

**Primary Color (Indigo)**: Represents creativity, stability, and trust.
**Neutral Palette (Slate)**: Cool grays feel more modern and tech-focused than warm grays.

### ☀️ Light Mode Tokens

- **Primary Brand**: `#4F46E5` (Indigo-600)
  - _Hover_: `#4338CA` (Indigo-700)
- **Background (App)**: `#F8FAFC` (Slate-50) – _Slightly off-white to reduce eye strain._
- **Surface (Cards/Modals)**: `#FFFFFF` (White)
- **Text (Primary)**: `#0F172A` (Slate-900) – _High contrast._
- **Text (Secondary)**: `#64748B` (Slate-500) – _Readable gray._
- **Border**: `#E2E8F0` (Slate-200)

### 🌙 Dark Mode Tokens (Deep Blue-Black)

- **Primary Brand**: `#6366F1` (Indigo-500) – _Lighter for better visibility on dark bg._
  - _Hover_: `#818CF8` (Indigo-400)
- **Background (App)**: `#020617` (Slate-950) – _Not pure black; very deep blue for premium feel._
- **Surface (Cards/Modals)**: `#0F172A` (Slate-900) – _Slightly lighter than bg._
- **Text (Primary)**: `#F8FAFC` (Slate-50)
- **Text (Secondary)**: `#94A3B8` (Slate-400)
- **Border**: `#1E293B` (Slate-800)

### 🚦 Semantic Colors (Both Modes)

- **Success (Approved/Sent)**: `#10B981` (Emerald-500)
- **Warning (Pending)**: `#F59E0B` (Amber-500)
- **Error (Rejected/Failed)**: `#EF4444` (Red-500)
- **Info (Draft)**: `#3B82F6` (Blue-500)

## 8️⃣ Design Specifications (Strict Mode)

### 📐 Spacing & Sizing (8pt Grid)

- **Base Unit**: 4px (`0.25rem`)
- **Padding (Card)**: 24px (`p-6`)
- **Padding (Page)**: 32px (`p-8`)
- **Gap (Form Fields)**: 16px (`gap-4`)
- **Gap (Sections)**: 40px (`gap-10`)

### 🔘 BorderRadius

- **Buttons/Inputs**: 8px (`rounded-md`) – _Modern, friendly but professional._
- **Cards/Modals**: 16px (`rounded-xl`) – _Slightly softer for containers._
- **Badges/Tags**: 9999px (`rounded-full`)

### 🌑 Shadows (Elevation)

- **Card (Subtle)**: `0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)` (`shadow`)
- **Dropdown/Popover**: `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)` (`shadow-lg`)
- **Modal**: `0 25px 50px -12px rgb(0 0 0 / 0.25)` (`shadow-2xl`)

### 🖼️ Iconography

- **Style**: Outline / Stroke (2px width).
- **Library**: Lucide React.
- **Size**:
  - Small (Button): 16px
  - Base (Nav): 20px
  - Large (Feature): 24px
