# Tong Calc RO

เครื่องคำนวณสเตตัสและดาเมจสำหรับเกม Ragnarok Online ที่รองรับอาชีพตั้งแต่ 1st Job ถึง 4th Job ครอบคลุมทั้งการคำนวณ ATK, MATK, ASPD, HP/SP, ทักษะโจมตี และการเปรียบเทียบ Preset

## 🌐 เว็บไซต์

**[https://turugrura.github.io/tong-calc-ro-host/#/](https://turugrura.github.io/tong-calc-ro-host/#/)**

## 🚀 Build & Run

**ติดตั้ง dependencies:**
```bash
npm install
```

**รัน dev server (HMR, port 4200):**
```bash
npm start
```

**Build สำหรับ production:**
```bash
npm run build
```

**รัน unit tests:**
```bash
npm test
```

**Lint (พร้อม auto-fix):**
```bash
npm run lint
```

**Deploy ไปยัง GitHub Pages:**
```bash
npm run predeploy   # build ไปที่ ../tong-calc-ro-host/docs/
npm run deploy      # copy index.html → 404.html
```

## 🗂️ โครงสร้างโปรเจกต์

```
src/app/
├── api-services/          # HTTP services (auth, preset, game data)
├── constants/             # ตารางข้อมูลเกม (element, race, enchant, weapon type)
│   └── enchant_item/      # รายการ enchant ของแต่ละ item
├── domain/                # Core types: Monster, Weapon
├── jobs/                  # คลาสอาชีพทั้งหมด (ขยายจาก CharacterBase)
├── models/                # TypeScript interfaces ของ app
├── utils/                 # Utility functions สำหรับการคำนวณ
└── layout/
    └── pages/
        └── ro-calculator/ # Calculator engine + UI หลัก
            ├── calculator.ts           # ตัว orchestrator หลัก
            ├── damage-calculator.ts    # คำนวณดาเมจ/DPS
            ├── hp-sp-calculator.ts     # คำนวณ MaxHP/MaxSP
            └── base-state-calculator.ts # คำนวณ status points
```

**Pages (lazy-loaded):**

| Path | คำอธิบาย |
|---|---|
| `/` | Calculator หลัก |
| `/shared-presets` | Preset ที่ community แชร์ไว้ |
| `/preset-summary` | เปรียบเทียบ Preset |
| `/login` | หน้า Login |

## 🔗 ลิงก์ที่เกี่ยวข้อง

- **แบบสำรวจ / Feedback:** https://forms.gle/aUYQKTfDym5Aakz9A
- **Issue Tracker:** https://docs.google.com/spreadsheets/d/1SBdqO0PzAY2rGawP_pvMAOA2erZK9_V0h2LxvcW-Epc/edit?usp=sharing
- **วิดีโอแนะนำ:** https://www.youtube.com/watch?v=ec5U-ZxvoFM&list=PLJi-aEFx61gw97he4dSOWP5-fADXegbyH&index=1&t
