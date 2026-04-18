# KinderTrack — Firebase Setup Guide
## Complete step-by-step from zero to deployed

---

## PART 1 — FIRESTORE SECURITY RULES

Go to: **Firebase Console → Firestore → Rules tab**
Paste this and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ── Helper functions ──────────────────────────────────────────
    function isAdmin() {
      return request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }

    function isTeacher() {
      return request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'teacher';
    }

    function teacherClass() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.assignedClass;
    }

    function isSignedIn() {
      return request.auth != null;
    }

    // ── users collection ─────────────────────────────────────────
    // Only admin can read all users. Each user can read their own doc.
    match /users/{userId} {
      allow read:  if isAdmin() || request.auth.uid == userId;
      allow write: if isAdmin();
    }

    // ── students collection ──────────────────────────────────────
    // Admin: full access. Teacher: read only their class.
    match /students/{studentId} {
      allow read:   if isAdmin()
                    || (isTeacher() && resource.data.classId == teacherClass());
      allow create: if isAdmin();
      allow update: if isAdmin();
      allow delete: if isAdmin();
    }

    // ── staff collection ─────────────────────────────────────────
    // Admin only
    match /staff/{staffId} {
      allow read, write: if isAdmin();
    }

    // ── attendance collection ────────────────────────────────────
    // Doc ID format: "YYYY-MM-DD_classId" (e.g. "2025-04-16_playgroup")
    // Admin: full access. Teacher: only their class.
    match /attendance/{docId} {
      allow read:   if isAdmin()
                    || (isTeacher() && resource.data.classId == teacherClass());
      allow create: if isAdmin()
                    || (isTeacher() && request.resource.data.classId == teacherClass());
      allow update: if isAdmin()
                    || (isTeacher()
                        && resource.data.classId == teacherClass()
                        && resource.data.locked == false);
      allow delete: if isAdmin();
    }

    // ── staffAttendance collection ───────────────────────────────
    // Admin only
    match /staffAttendance/{docId} {
      allow read, write: if isAdmin();
    }

    // ── holidays collection ──────────────────────────────────────
    // Admin can write. All signed-in users can read (so calendar works).
    match /holidays/{dateId} {
      allow read:  if isSignedIn();
      allow write: if isAdmin();
    }

  }
}
```

---

## PART 2 — ENABLE AUTHENTICATION

### Step A — Enable Email/Password
1. Go to **Firebase Console → Authentication → Sign-in method**
2. Click **Email/Password** → Enable → Save

### Step B — Enable Google Sign-In
1. Still in Sign-in method tab
2. Click **Google** → Enable
3. Set a **Project support email** (your email)
4. Click Save

### Step C — Add your domain to Authorized Domains
1. Scroll down on the Sign-in method page
2. Under **Authorized domains**, add:
   - `localhost` (already there)
   - `kindertrack-school.web.app` (already there after deploy)
   - Any custom domain you'll use

---

## PART 3 — CREATE ADMIN USER

### Step A — Create in Firebase Auth
1. Go to **Authentication → Users → Add user**
2. Enter:
   - Email: `admin@kindertrack.com` (or whatever you want)
   - Password: a strong password
3. Click **Add user**
4. Copy the **User UID** shown in the table

### Step B — Create Firestore document for admin
1. Go to **Firestore → Data**
2. Click **Start collection** → Collection ID: `users`
3. Document ID: paste the **UID from Step A**
4. Add these fields:

```
Field: role          Type: string   Value: admin
Field: name          Type: string   Value: School Admin
Field: email         Type: string   Value: admin@kindertrack.com
Field: createdAt     Type: timestamp  Value: (click the timestamp button)
```

5. Click **Save**

---

## PART 4 — CREATE TEACHER ACCOUNTS

Do this for each of your 4 teachers:

### Option A — Google Sign-In teacher (recommended)
Teachers sign in with Google on first visit. Then you create their Firestore record.

**When a teacher signs in with Google for the first time:**
1. They'll get an error: "Your account is not set up"
2. Go to **Authentication → Users** — you'll see their account listed
3. Copy their **UID**
4. Go to **Firestore → users collection** → New document
5. Document ID: the teacher's **UID**
6. Add fields:

```
Field: role           Type: string   Value: teacher
Field: name           Type: string   Value: Mrs. Sunita Rao
Field: email          Type: string   Value: sunita@gmail.com
Field: assignedClass  Type: string   Value: playgroup
Field: createdAt      Type: timestamp
```

7. Now the teacher can sign in and access only Playgroup.

### Option B — Email/Password teacher
1. Go to **Authentication → Users → Add user**
2. Enter teacher email + password
3. Copy the UID
4. Create Firestore doc as above (same fields)

---

## PART 5 — assignedClass values (must match exactly)

| Teacher   | assignedClass value |
|-----------|---------------------|
| Teacher A | `playgroup`         |
| Teacher B | `nursery`           |
| Teacher C | `lkg`               |
| Teacher D | `ukg`               |

---

## PART 6 — EXAMPLE FIRESTORE DOCUMENTS

### users/{adminUid}
```json
{
  "role": "admin",
  "name": "School Admin",
  "email": "admin@kindertrack.com",
  "createdAt": "2025-04-16T09:00:00Z"
}
```

### users/{teacherUid}
```json
{
  "role": "teacher",
  "name": "Mrs. Sunita Rao",
  "email": "sunita.rao@gmail.com",
  "assignedClass": "playgroup",
  "createdAt": "2025-04-16T09:00:00Z"
}
```

### students/{autoId}
```json
{
  "name": "Aarav Sharma",
  "roll": "01",
  "classId": "playgroup",
  "createdAt": "2025-04-16T09:00:00Z",
  "createdBy": "{adminUid}"
}
```

### attendance/2025-04-16_playgroup
```json
{
  "date": "2025-04-16",
  "classId": "playgroup",
  "locked": true,
  "records": {
    "{studentId1}": "present",
    "{studentId2}": "absent",
    "{studentId3}": "present"
  },
  "updatedAt": "2025-04-16T10:30:00Z",
  "updatedBy": "{teacherUid}"
}
```

### staffAttendance/2025-04-16
```json
{
  "date": "2025-04-16",
  "records": {
    "{staffId1}": { "clockIn": "08:15", "clockOut": "15:45" },
    "{staffId2}": { "absent": true }
  },
  "updatedAt": "2025-04-16T08:30:00Z"
}
```

### holidays/2025-04-14
```json
{
  "markedBy": "{adminUid}",
  "markedAt": "2025-04-13T12:00:00Z"
}
```

### staff/{autoId}
```json
{
  "name": "Mrs. Sunita Rao",
  "role": "Playgroup Teacher",
  "createdAt": "2025-04-16T09:00:00Z"
}
```

---

## PART 7 — ADD SEED DATA (optional, to prepopulate students)

In Firestore → students collection, add documents manually for each student,
or use this helper — open browser DevTools Console while logged in as admin and paste:

```javascript
// Run this ONCE in browser console after logging in as admin
// Adds sample students to Firestore

const sampleStudents = [
  { name: 'Aarav Sharma',  roll: '01', classId: 'playgroup' },
  { name: 'Diya Patel',    roll: '02', classId: 'playgroup' },
  { name: 'Rohan Mehta',   roll: '03', classId: 'playgroup' },
  { name: 'Priya Singh',   roll: '04', classId: 'playgroup' },
  { name: 'Arjun Nair',    roll: '05', classId: 'playgroup' },
  { name: 'Kavya Reddy',   roll: '01', classId: 'nursery' },
  { name: 'Vivaan Joshi',  roll: '02', classId: 'nursery' },
  { name: 'Ananya Kumar',  roll: '03', classId: 'nursery' },
  { name: 'Ishaan Gupta',  roll: '04', classId: 'nursery' },
  { name: 'Aisha Khan',    roll: '01', classId: 'lkg' },
  { name: 'Dev Malhotra',  roll: '02', classId: 'lkg' },
  { name: "Mia D'Souza",   roll: '03', classId: 'lkg' },
  { name: 'Rahul Verma',   roll: '04', classId: 'lkg' },
  { name: 'Neha Chopra',   roll: '05', classId: 'lkg' },
  { name: 'Sara Iyer',     roll: '01', classId: 'ukg' },
  { name: 'Om Trivedi',    roll: '02', classId: 'ukg' },
  { name: 'Tara Bose',     roll: '03', classId: 'ukg' },
  { name: 'Rishi Pillai',  roll: '04', classId: 'ukg' },
  { name: 'Zara Ansari',   roll: '05', classId: 'ukg' },
];

const sampleStaff = [
  { name: 'Mrs. Sunita Rao',   role: 'Playgroup Teacher' },
  { name: 'Mrs. Preethi Nair', role: 'Nursery Teacher' },
  { name: 'Mr. Anil Kumar',    role: 'LKG Teacher' },
  { name: 'Mrs. Rekha Joshi',  role: 'UKG Teacher' },
  { name: 'Mr. Vijay Sharma',  role: 'Peon / Helper' },
  { name: 'Mrs. Lata Patil',   role: 'Receptionist' },
];

const db2 = firebase.firestore();
const ts  = firebase.firestore.FieldValue.serverTimestamp();

Promise.all([
  ...sampleStudents.map(s => db2.collection('students').add({ ...s, createdAt: ts })),
  ...sampleStaff.map(s => db2.collection('staff').add({ ...s, createdAt: ts })),
]).then(() => console.log('✅ Seed data added!'))
  .catch(e => console.error('❌ Error:', e));
```

---

## PART 8 — WHERE TO PASTE FIREBASE CODE IN YOUR HTML

The file `KinderTrack-Firebase.html` is self-contained. Here's what was changed vs the original:

1. **Lines 8–11** — Firebase CDN scripts added in `<head>`
2. **Login HTML** — replaced username/password tabs with Admin email/password + Teacher Google button
3. **`<script>` block start** — `firebase.initializeApp()` + `db.enablePersistence()`
4. **`DATA` object** — still exists as local cache; Firestore populates it via listeners
5. **`auth.onAuthStateChanged()`** — replaces `doLogin()`, handles all auth automatically
6. **All write functions** — `addStudent()`, `markCard()`, `toggleHoliday()` etc. now call Firestore
7. **Teacher role** — new `page-teacher-attendance` page with restricted swipe view

---

## PART 9 — FIREBASE HOSTING DEPLOYMENT

### Prerequisites — install Firebase CLI once
```bash
npm install -g firebase-tools
firebase login
```

### Step 1 — Initialize in your project folder
```bash
mkdir kindertrack
cd kindertrack
cp /path/to/KinderTrack-Firebase.html index.html
firebase init
```

When prompted:
- **Which features?** → Select `Hosting` (spacebar to select, Enter to confirm)
- **Select a project** → Choose `kindertrack-school`
- **What do you want to use as your public directory?** → `.` (just a dot)
- **Configure as a single-page app?** → `N`
- **Set up automatic builds with GitHub?** → `N`
- **File . already exists. Overwrite?** → `N`

### Step 2 — Deploy
```bash
firebase deploy
```

After deploy you'll see:
```
Hosting URL: https://kindertrack-school.web.app
```

### Step 3 — Add deployed URL to Firebase Auth
1. Go to Firebase Console → Authentication → Sign-in method
2. Scroll to **Authorized domains**
3. Click **Add domain**
4. Type: `kindertrack-school.web.app`
5. Save

### Subsequent deploys (after code changes)
```bash
firebase deploy --only hosting
```

---

## PART 10 — ROLE PERMISSION SUMMARY

| Feature                    | Admin | Teacher (own class) |
|----------------------------|-------|---------------------|
| Dashboard                  | ✅    | ❌                   |
| All classes view           | ✅    | ❌ (own class only)  |
| Add/delete students        | ✅    | ❌                   |
| Mark attendance (all)      | ✅    | ❌ (own class only)  |
| Mark attendance (own class)| ✅    | ✅                   |
| Override locked session    | ✅    | ❌                   |
| Staff attendance           | ✅    | ❌                   |
| Calendar view              | ✅    | ❌                   |
| Monthly reports            | ✅    | ❌                   |
| Mark holidays              | ✅    | ❌                   |

---

## QUICK TROUBLESHOOTING

| Problem | Fix |
|---------|-----|
| "Your account is not set up" | Create a `users/{uid}` doc in Firestore with role field |
| Google sign-in popup blocked | Browser blocked popup — allow it in address bar |
| "Missing or insufficient permissions" | Security rules issue — re-publish the rules from Part 1 |
| Students not loading | Check Firestore `students` collection exists, check security rules |
| Attendance not saving | Check browser console for errors; verify user has write permission |
| Offline badge showing | No internet — app still works, syncs when back online |
| Teacher sees wrong class | Check `assignedClass` field in their Firestore `users` doc |
