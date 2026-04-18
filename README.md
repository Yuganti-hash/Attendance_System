# 📋 Attendance System

> A web-based attendance management system built with HTML, Firebase Authentication, and Cloud Firestore. Designed for schools and classrooms to mark, store, and track attendance in real time — with a clean, beginner-friendly interface.

---

## 📖 Description

The **Attendance System** is a lightweight, fully browser-based application that allows administrators and teachers to manage student and staff attendance effortlessly. It leverages Firebase's real-time cloud infrastructure to store and sync attendance records instantly — no server setup required.

Whether you are marking daily attendance for a single class or generating month-end reports for an entire school, this system provides all the tools you need in one place.

---

## ✨ Features

- ✅ **Mark Attendance** — Quickly mark students as Present or Absent using an interactive swipe-card interface
- ☁️ **Firebase Integration** — All attendance data is stored securely in Cloud Firestore
- 🔄 **Real-Time Updates** — Changes sync instantly across devices without page reloads
- 🔐 **Role-Based Login** — Separate login flows for Admins and Teachers
- 👥 **Student & Staff Management** — Add, import (CSV), and export student and staff records
- 📅 **Attendance Calendar** — View attendance history for any date with class-wise breakdowns
- 📊 **Monthly Reports** — Generate and export PDF reports (Summary, Student, Staff)
- 📱 **Mobile Responsive** — Fully usable on phones and tablets
- 🎉 **Holiday Marking** — Mark specific dates as holidays to exclude them from reports

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| **HTML5** | Application structure and layout |
| **CSS3 (Vanilla)** | Styling, animations, and responsive design |
| **JavaScript (Vanilla)** | Application logic and Firebase interactions |
| **Firebase Authentication** | User login (Email/Password + Google Sign-In) |
| **Cloud Firestore** | Real-time NoSQL database for attendance data |
| **Firebase Hosting** | Static site deployment |
| **jsPDF** | PDF report generation in the browser |

---

## 📁 Folder Structure

```
Attendance_System/
│
├── index.html                # Main application (single HTML file)
├── 404.html                  # Custom 404 error page for Firebase Hosting
├── firebase.json             # Firebase project configuration
├── firestore.rules           # Firestore security rules
├── firestore.indexes.json    # Firestore composite index definitions
├── .firebaserc               # Firebase project alias
├── FIREBASE-SETUP-GUIDE.md   # Step-by-step Firebase setup guide
└── README.md                 # Project documentation (this file)
```

> **Note:** This is a single-page application. All HTML, CSS, and JavaScript live inside `index.html`.

---

## ⚙️ Installation Steps

Follow these steps to get the project running on your local machine.

### Step 1 — Clone the Repository

```bash
git clone https://github.com/Yuganti-hash/Attendance_System.git
cd Attendance_System
```

### Step 2 — Install Firebase CLI

```bash
npm install -g firebase-tools
```

### Step 3 — Log In to Firebase

```bash
firebase login
```

### Step 4 — Link to Your Firebase Project

```bash
firebase use YOUR_PROJECT_ID
```

> Replace `YOUR_PROJECT_ID` with the ID of your Firebase project from the [Firebase Console](https://console.firebase.google.com/).

---

## 🔥 Firebase Setup Instructions

### 1. Create a Firebase Project

1. Go to [https://console.firebase.google.com/](https://console.firebase.google.com/)
2. Click **"Add Project"** and follow the setup wizard
3. Once created, click **"Web"** ( `</>` ) to register a web app

### 2. Enable Authentication

1. In the Firebase Console, go to **Build → Authentication**
2. Click **"Get Started"**
3. Enable the following sign-in providers:
   - ✅ **Email/Password**
   - ✅ **Google**

### 3. Create a Firestore Database

1. Go to **Build → Firestore Database**
2. Click **"Create Database"**
3. Choose **"Start in production mode"**
4. Select a region (e.g., `asia-south1`)

### 4. Add Your Firebase Config to the App

Open `index.html` and find the `firebaseConfig` object. Replace the placeholder values with your project's credentials:

```js
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

> You can find these values in Firebase Console → Project Settings → Your Apps.

### 5. Deploy Firestore Security Rules

```bash
firebase deploy --only firestore:rules
```

### 6. Create Your First Admin User

1. Go to Firebase Console → **Authentication** → Add a user manually (email + password)
2. Copy the user's **UID** from the Authentication panel
3. Go to **Firestore Database** → Create a collection named `users`
4. Add a document with the UID as the Document ID and these fields:

```
role  : "admin"
name  : "Your Name"
email : "your@email.com"
```

---

## ▶️ How to Run the Project

### Option A — Run Locally with Firebase Emulator (Recommended)

```bash
firebase emulators:start
```

Then open your browser and go to:

```
http://localhost:5000
```

### Option B — Deploy to Firebase Hosting (Live)

```bash
firebase deploy --only hosting
```

Your app will be live at:

```
https://YOUR_PROJECT_ID.web.app
```

### Option C — Open Directly in Browser

Since the app is plain HTML, you can also just double-click `index.html` to open it locally. However, Firebase features (login, database) require the app to be served over HTTP — so the emulator or hosting method is preferred.

---

## 🔐 User Roles

| Role | What They Can Do |
|---|---|
| **Admin** | Full access — manage students, staff, attendance, reports, and holidays |
| **Teacher** | Can only mark attendance for their assigned class |

---

## 📄 CSV Import Format

You can bulk-import students and staff using CSV files.

**Students CSV:**
```csv
name,roll,class
Aarav Sharma,01,Nursery
Priya Patel,02,LKG
```

**Staff CSV:**
```csv
name,role,class
Ms. Anjali,Class Teacher,Nursery
Mr. Rohan,Assistant,LKG
```

> Class names must be one of: `Playgroup`, `Nursery`, `LKG`, `UKG` (case-insensitive).

---

## 🚀 Future Improvements

- [ ] 📧 Email notifications for low attendance
- [ ] 📲 Progressive Web App (PWA) support for offline use
- [ ] 🌐 Multi-language / localization support
- [ ] 📷 Face recognition for automated attendance
- [ ] 👨‍👩‍👧 Parent portal to view their child's attendance
- [ ] 📆 Leave management and approval workflow
- [ ] 🔔 Push notifications for absent students

---

## 🤝 Contributing

Contributions are welcome! Here's how to get started:

1. Fork this repository
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Make your changes and commit: `git commit -m "Add your feature"`
4. Push to your branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

---

## 👩‍💻 Author

**Yuganti Hatwar**

- 📧 Email: [yugantihatwar43@gmail.com](mailto:yugantihatwar43@gmail.com)
- 🐙 GitHub: [@Yuganti-hash](https://github.com/Yuganti-hash)

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

---

<div align="center">
  <strong>Made with ❤️ for easy, modern attendance tracking</strong>
</div>
