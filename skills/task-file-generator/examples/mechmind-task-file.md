# MechMind — Autonomous Task File 01: MVP Web App
**Session Type:** Overnight autonomous (Claude Code)
**Reference Repo:** none
**Target Repo:** https://github.com/scryptoginger/mechmind.git (create if not exists)
**Commit frequently:** After every major milestone. Push to GitHub after every commit.

---

## CONTEXT

MechMind is a vehicle maintenance tracker for DIY enthusiasts — people who wrench on their
own daily drivers, project cars, and overland rigs. It replaces the spreadsheet that every
DIY mechanic eventually builds: tracking completed work, upcoming maintenance, parts used,
torque specs, and service intervals.

What makes MechMind more than a spreadsheet is a lightweight AI layer that, given a specific
vehicle (year/make/model/trim), can pull manufacturer-recommended service schedules, torque
specs, part numbers, socket sizes, fluid capacities, and link to relevant DIY guides. The
user shouldn't have to Google "2017 Tacoma TRD OR rear diff fluid capacity" — the app
should already know.

**This session builds the MVP as a web app deployable to Vercel.** Not a native mobile app
yet. The MVP must be functional enough that the developer (Keith) would actually use it to
track maintenance on his 2017 Toyota Tacoma TRD Off-Road — that's the acceptance test.

MechMind is NOT:
- A full shop management system (no invoicing, no customer management)
- A parts marketplace or affiliate link farm
- A social platform for car enthusiasts (no forums, no sharing — yet)
- A vehicle diagnostic tool (no OBD-II, no sensor data)

Design principles:
- Mobile-first responsive design. This gets used in the garage on a phone with greasy hands.
- Data belongs to the user. No account required for MVP — local storage + optional cloud sync later.
- AI is a research assistant, not a gatekeeper. Every AI-surfaced fact should cite its source
  or be clearly marked as "verify before wrenching."
- Keep it simple. A DIY mechanic who hates apps should find this faster than their spreadsheet
  within 5 minutes of first use.

---

## PHASE 0 — Environment Setup

### 0.1 System dependencies
```bash
node --version   # Requires Node 18+
npm --version
npx --version
```

### 0.2 Initialize Next.js project
```bash
npx create-next-app@latest mechmind \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"
cd mechmind
```

### 0.3 Git setup
```bash
git init
git remote add origin https://github.com/scryptoginger/mechmind.git
```

### 0.4 Install core dependencies
```bash
npm install zustand           # Lightweight state management
npm install lucide-react      # Icons
npm install date-fns          # Date formatting
npm install zod               # Schema validation
npm install ai @ai-sdk/anthropic  # Vercel AI SDK for Claude integration
```

### 0.5 Verify Vercel deployment target
```bash
npx vercel --version          # Confirm Vercel CLI available
```

**Commit message:** `chore: initialize Next.js project with core dependencies`

---

## PHASE 1 — Data Model & Types

### 1.1 Vehicle type
Create `src/types/vehicle.ts`:
```typescript
interface Vehicle {
  id: string;
  year: number;
  make: string;
  model: string;
  trim: string;
  nickname?: string;          // "Taco", "The Crawler", etc.
  vin?: string;
  mileage: number;            // Current odometer reading
  mileageUpdatedAt: string;   // ISO date of last mileage update
  imageUrl?: string;
  createdAt: string;
}
```

### 1.2 Maintenance record type
Create `src/types/maintenance.ts`:
```typescript
interface MaintenanceRecord {
  id: string;
  vehicleId: string;
  title: string;              // "Rear diff fluid change"
  category: MaintenanceCategory;
  date: string;               // ISO date performed
  mileage: number;            // Odometer at time of service
  notes?: string;             // Free-form notes
  parts: Part[];
  cost?: number;              // Total cost (parts + fluids, no labor — you ARE the labor)
  nextDueMileage?: number;    // When this service is due again
  nextDueDate?: string;       // Or by date, whichever comes first
  status: 'completed' | 'planned' | 'in-progress';
  torqueSpecs?: TorqueSpec[]; // Relevant torque specs for this job
}

type MaintenanceCategory =
  | 'fluids'          // Oil, diff, t-case, trans, coolant, brake
  | 'brakes'          // Pads, rotors, calipers, lines
  | 'suspension'      // Shocks, springs, lifts, bushings, control arms
  | 'drivetrain'      // Axles, driveshaft, u-joints, CV joints
  | 'engine'          // Belts, plugs, filters, timing
  | 'electrical'      // Battery, alternator, lights, wiring
  | 'body'            // Bumpers, skid plates, armor, rack
  | 'tires-wheels'    // Tires, wheels, alignment, rotation, balance
  | 'other';

interface Part {
  name: string;
  partNumber?: string;
  quantity: number;
  cost?: number;
  source?: string;            // "AutoZone", "Toyota dealer", "Amazon"
}

interface TorqueSpec {
  fastener: string;           // "Drain plug", "Caliper bracket bolt"
  spec: string;               // "18 ft-lbs", "90 N-m"
  notes?: string;             // "Apply thread locker"
}
```

### 1.3 AI research result type
Create `src/types/research.ts`:
```typescript
interface ResearchResult {
  vehicleId: string;
  query: string;
  response: string;           // Markdown-formatted AI response
  sources?: string[];         // URLs or reference names
  createdAt: string;
  category: 'schedule' | 'specs' | 'howto' | 'parts' | 'general';
}
```

### 1.4 Zustand store
Create `src/store/index.ts`:
Implement a Zustand store with `persist` middleware (localStorage) containing:
- `vehicles: Vehicle[]`
- `records: MaintenanceRecord[]`
- `researchHistory: ResearchResult[]`
- CRUD actions for all three collections
- `getRecordsForVehicle(vehicleId)` selector
- `getUpcomingMaintenance(vehicleId)` selector — returns records where `nextDueMileage` or `nextDueDate` is approaching

**Commit message:** `feat: data model, types, and Zustand store with localStorage persistence`

---

## PHASE 2 — Vehicle Management UI

### 2.1 Add Vehicle form
Create `src/components/AddVehicleForm.tsx`:
- Fields: year (number), make (text), model (text), trim (text), nickname (optional), current mileage
- Year/make/model could be free-text for MVP — no VIN decoder yet
- Validate with Zod
- On submit, add to store

### 2.2 Vehicle list / garage view
Create `src/app/page.tsx` (home route):
- Display all vehicles as cards
- Each card shows: nickname (or year/make/model), current mileage, count of maintenance records, next upcoming service
- Tap a card → navigate to `/vehicle/[id]`
- "Add Vehicle" button → opens AddVehicleForm

### 2.3 Vehicle detail page
Create `src/app/vehicle/[id]/page.tsx`:
- Vehicle header: name, year/make/model/trim, mileage (editable — tap to update)
- Tabs or sections: Maintenance Log, Upcoming, AI Research
- Maintenance Log: list of completed/in-progress records, sorted by date descending
- Upcoming: planned records + any records where next-due is approaching
- AI Research: research history for this vehicle (Phase 5)

### 2.4 Layout and navigation
Create `src/components/Layout.tsx`:
- Mobile-first. Bottom nav bar with: Garage (home), Add Record (+), Research (magnifying glass)
- Responsive: on desktop, sidebar nav instead of bottom bar
- Dark mode support via Tailwind `dark:` classes — garages are dark

**Commit message:** `feat: vehicle management UI — garage view, add vehicle, vehicle detail`

---

## PHASE 3 — Maintenance Record Management

### 3.1 Add/Edit Maintenance Record form
Create `src/components/MaintenanceRecordForm.tsx`:
- Fields: title, category (dropdown), date, mileage, status, notes
- Parts sub-form: add/remove parts with name, part number, quantity, cost, source
- Torque specs sub-form: add/remove specs with fastener name, spec value, notes
- Next-due fields: mileage interval and/or date interval
- Pre-populate vehicleId from context

### 3.2 Maintenance record detail view
Create `src/components/MaintenanceRecordDetail.tsx`:
- Full view of a record with all fields
- Edit button → opens form in edit mode
- Delete with confirmation

### 3.3 Maintenance timeline view
Create `src/components/MaintenanceTimeline.tsx`:
- Chronological list of all records for a vehicle
- Color-coded by category
- Visual indicator for status (completed ✅, planned 📋, in-progress 🔧)
- Filter by category

### 3.4 Upcoming maintenance alerts
Create `src/components/UpcomingMaintenance.tsx`:
- List of records where `nextDueMileage` is within 1,000 miles of current mileage OR `nextDueDate` is within 30 days
- Sort by urgency (closest to due first)
- Color: green (>1000 mi / >30 days), yellow (500-1000 mi / 14-30 days), red (<500 mi / <14 days)

**Commit message:** `feat: maintenance record CRUD, timeline, and upcoming alerts`

---

## PHASE 4 — Seed Data: 2017 Tacoma TRD OR

Create `src/data/seed.ts` — a function that pre-populates the store with a sample vehicle and records for testing and demo purposes.

### 4.1 Vehicle
```typescript
{
  year: 2017,
  make: "Toyota",
  model: "Tacoma",
  trim: "TRD Off-Road",
  nickname: "Taco",
  mileage: 85000,  // Approximate — adjust to actual
}
```

### 4.2 Sample planned maintenance records
Based on common Tacoma TRD OR service items, create planned records for:

1. **Rear differential fluid change** — Category: fluids, Status: planned
   - Parts: 2.5 qt 75W GL-5 gear oil, crush washer
   - Torque specs: fill plug 36 ft-lbs, drain plug 36 ft-lbs
   - Interval: every 30,000 miles

2. **Front differential fluid change** — Category: fluids, Status: planned
   - Parts: 2.3 qt 75W GL-5 gear oil
   - Torque specs: fill plug 36 ft-lbs, drain plug 36 ft-lbs
   - Interval: every 30,000 miles

3. **Transfer case fluid change** — Category: fluids, Status: planned
   - Parts: 1.5 qt 75W GL-5 gear oil (verify — some Tacomas use ATF)
   - Torque specs: fill plug 27 ft-lbs, drain plug 27 ft-lbs
   - Note: "Verify fluid type for 2017 TRD OR — some forums say Toyota ATF WS, manual says 75W"

4. **Transmission fluid change** — Category: fluids, Status: planned
   - Parts: ~3.5 qt Toyota ATF WS (drain and fill, not flush)
   - Torque specs: drain plug 15 ft-lbs (verify — soft aluminum pan)

5. **Brake pads and rotors (front)** — Category: brakes, Status: planned
   - Torque specs: caliper bracket bolts 79 ft-lbs, caliper slide pins 25 ft-lbs, lug nuts 83 ft-lbs

6. **Brake pads and rotors (rear)** — Category: brakes, Status: planned
   - Torque specs: caliper bracket bolts 65 ft-lbs, lug nuts 83 ft-lbs

7. **Front suspension** — Category: suspension, Status: planned

8. **Rear suspension** — Category: suspension, Status: planned

### 4.3 Seed trigger
Add a "Load demo data" button to the garage view (only shows when vehicle list is empty). On click, run the seed function. This doubles as the test harness.

**Important:** All torque specs and fluid capacities above are approximate. Mark each with a note: *"AI-sourced — verify against your owner's manual before wrenching."* This models the behavior the AI research feature should follow.

**Commit message:** `feat: seed data for 2017 Tacoma TRD OR with planned maintenance`

---

## PHASE 5 — AI Research Assistant

### 5.1 API route
Create `src/app/api/research/route.ts`:
- POST endpoint accepting `{ vehicleId, query, vehicle }` (vehicle object included so the AI has context)
- Uses Vercel AI SDK with Claude Sonnet as the model
- System prompt:

```
You are a knowledgeable vehicle maintenance assistant. The user owns a
{year} {make} {model} {trim}. Answer their question with specific,
actionable information for their exact vehicle. Include:
- Specific part numbers (OEM and common aftermarket) when relevant
- Torque specifications with units (ft-lbs and N-m)
- Fluid types and capacities
- Socket/wrench sizes needed
- Step-by-step procedures when asked for guides
- Manufacturer-recommended service intervals

Always note when information should be verified against the owner's manual.
Format your response in Markdown.
```

- Stream the response back using Vercel AI SDK streaming
- Save the result to the research history in the store (client-side, after completion)

### 5.2 Research UI
Create `src/components/ResearchChat.tsx`:
- Simple chat-style interface scoped to a vehicle
- Input field with suggested queries: "What are the torque specs for brake jobs?", "Recommended service schedule", "What socket sizes do I need for a diff fluid change?", "OEM vs aftermarket brake pad options"
- Streaming response display (Markdown rendered)
- Research history below the input, collapsible

### 5.3 Quick-research from maintenance records
Add a "Research this job" button to each maintenance record that pre-fills the research query with the record title + vehicle context. Example: user taps "Research this job" on "Rear differential fluid change" → query auto-fills: "How to change rear differential fluid on a 2017 Toyota Tacoma TRD Off-Road — fluid type, capacity, torque specs, tools needed, step-by-step"

**Commit message:** `feat: AI research assistant with streaming, vehicle-scoped context`

---

## PHASE 6 — Polish & Mobile UX

### 6.1 Mobile-first responsive pass
Review all pages and components for mobile usability:
- Touch targets minimum 44px
- No horizontal scroll on mobile
- Forms should be usable with one greasy hand (large inputs, big buttons)
- Bottom sheet pattern for add/edit forms on mobile (modal on desktop)

### 6.2 Empty states
Every list view needs an empty state:
- Garage: "No vehicles yet. Add your first ride."
- Maintenance log: "No records yet. Log your first job."
- Upcoming: "Nothing due soon. Nice work keeping up."
- Research: "Ask me anything about your [vehicle]."

### 6.3 Loading and error states
- Skeleton loaders for AI research streaming
- Error boundaries for API failures
- Offline indicator (localStorage works offline, AI research doesn't — make this clear)

### 6.4 PWA basics (optional but valuable)
Add `next-pwa` or manual service worker for:
- Installable on phone home screen
- Offline access to vehicle data and maintenance records (not AI research)
- This alone makes it "an app on my phone" without the app store

**Commit message:** `feat: mobile UX polish, empty states, loading states`

---

## PHASE 7 — Vercel Deployment

### 7.1 Environment variables
Create `.env.local` (gitignored) and `.env.example` (committed):
```
ANTHROPIC_API_KEY=your_key_here
```

### 7.2 Deploy
```bash
npx vercel --prod
```

### 7.3 Verify
- Open deployed URL on phone
- Add vehicle (2017 Tacoma TRD OR)
- Add a maintenance record
- Run an AI research query
- Verify localStorage persists on refresh

**Commit message:** `chore: Vercel deployment configuration`

---

## PHASE 8 — Final Validation

Before ending the session, run this checklist and fix anything that fails:

```bash
# 1. Build passes with no errors
npm run build

# 2. Lint passes
npm run lint

# 3. Dev server starts cleanly
npm run dev
# Visit localhost:3000 — garage page loads

# 4. Type check
npx tsc --noEmit

# 5. Test the full user flow manually:
#    a. Add a vehicle (2017 Tacoma TRD OR)
#    b. Add a maintenance record (rear diff fluid change)
#    c. View the timeline
#    d. Check upcoming maintenance
#    e. Run an AI research query ("torque specs for rear diff drain plug")
#    f. Refresh the page — data persists (localStorage)

# 6. Git status — nothing uncommitted
git status
git log --oneline
```

**Final commit message:** `chore: phase 8 validation pass — MechMind MVP session 01 complete`

---

## PUSH REMINDERS

Push to GitHub after EVERY commit. Do not batch pushes.
```bash
git push origin main
```

---

## WHAT SUCCESS LOOKS LIKE

When this session ends, Keith should be able to:
1. Open the deployed Vercel URL on his phone
2. Add his 2017 Tacoma TRD OR with current mileage
3. Load the demo seed data and see pre-populated planned maintenance
4. Add a new maintenance record for rear diff fluid change with parts, costs, and torque specs
5. See upcoming maintenance sorted by urgency
6. Ask the AI "What fluid does my transfer case take?" and get a vehicle-specific answer with caveats
7. Refresh the page — all data persists
8. Feel like this is genuinely faster than his spreadsheet

---

## NEXT SESSION PREVIEW (do not implement now, just note)

- User authentication (Clerk or NextAuth) for cloud sync across devices
- Photo attachments on maintenance records (before/after shots)
- VIN decoder for auto-populating vehicle details
- Export to PDF (maintenance history printout for selling the vehicle)
- Cost tracking dashboard (total spend per vehicle, per category, over time)
- Shared vehicle access (spouse/co-owner can view and add records)
- Receipt scanner (snap a photo of an AutoZone receipt, AI extracts parts + costs)
- OBD-II integration via Bluetooth adapter (long-term, native app territory)
- Community-contributed torque spec database (reduce AI dependency for known values)
