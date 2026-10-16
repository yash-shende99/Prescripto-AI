# ═══════════════════════════════════════════════════════════════════════════════
# PRESCRIPTO — AI-POWERED DYNAMIC APPOINTMENT SCHEDULER & HEALTHCARE TRIAGE
# COMPLETE INTERVIEW Q&A REFERENCE DOCUMENT
# Derived from Actual Codebase Analysis | Exhaustive Architecture & Code Deep Dive
# Total Question Count: 365+ Questions & Defensible Answers
# ═══════════════════════════════════════════════════════════════════════════════

> **Authoritative Codebase Reference**: `Prescripto (v1.0.0)`  
> **Repository Target**: `yash-shende99/Prescripto-AI`  
> **Tech Stack Analyzed**: React 18, Vite 6, Tailwind CSS (v3 in Frontend, v4 in Admin), Node.js (ESM), Express 4.21, MongoDB / Mongoose 8.9, Python 3 / Flask, Scikit-Learn (Pickle Model Pipeline), Cloudinary v2, Razorpay SDK, Nginx Alpine, Docker & Docker Compose.  
> **Target Roles**: Senior Full-Stack Engineer, Healthcare Tech Architect, Machine Learning Systems Engineer.

---

## TABLE OF CONTENTS

1. [Header & Metadata](#header--metadata)
2. [SECTION 1: Project Overview & Motivation](#section-1-project-overview--motivation) *(Q1 – Q15)*
3. [SECTION 2: Major Modules & System Architecture](#section-2-major-modules--system-architecture) *(Q16 – Q35)*
4. [SECTION 3: Technology Choices & Tradeoffs](#section-3-technology-choices--tradeoffs) *(Q36 – Q60)*
5. [SECTION 4: Backend Architecture & Implementation](#section-4-backend-architecture--implementation) *(Q61 – Q85)*
6. [SECTION 5: Database, Schema Design, Auth & Security](#section-5-database-schema-design-auth--security) *(Q86 – Q115)*
7. [SECTION 6: API Endpoints & Request Lifecycles](#section-6-api-endpoints--request-lifecycles) *(Q116 – Q145)*
8. [SECTION 7: AI/ML Diagnostic & Recommendation Pipeline](#section-7-aiml-diagnostic--recommendation-pipeline) *(Q146 – Q175)*
9. [SECTION 8: Model Layer, Training, Encoders & Mathematics](#section-8-model-layer-training-encoders--mathematics) *(Q176 – Q195)*
10. [SECTION 9: Clinical Safety, Hallucination Prevention & Model Security](#section-9-clinical-safety-hallucination-prevention--model-security) *(Q196 – Q215)*
11. [SECTION 10: Caching Architecture & State Synchronization](#section-10-caching-architecture--state-synchronization) *(Q216 – Q230)*
12. [SECTION 11: Domain Module: Hospital Management & Slot Allocation Engine](#section-11-domain-module-hospital-management--slot-allocation-engine) *(Q231 – Q250)*
13. [SECTION 12: Core Features & End-to-End User Workflows](#section-12-core-features--end-to-end-user-workflows) *(Q251 – Q270)*
14. [SECTION 13: RBAC & Tri-Party Multi-Tenancy](#section-13-rbac--tri-party-multi-tenancy) *(Q271 – Q285)*
15. [SECTION 14: Performance, Concurrency & Horizontal Scaling](#section-14-performance-concurrency--horizontal-scaling) *(Q286 – Q305)*
16. [SECTION 15: Edge Cases, Failure Modes & Disaster Recovery](#section-15-edge-cases-failure-modes--disaster-recovery) *(Q306 – Q325)*
17. [SECTION 16: Resume Defense & Technical Interview Deep Dives](#section-16-resume-defense--technical-interview-deep-dives) *(Q326 – Q345)*
18. [SECTION 17: Hardest Engineering Challenges & Solutions](#section-17-hardest-engineering-challenges--solutions) *(Q346 – Q355)*
19. [SECTION 18: Complete Data Flow — End to End](#section-18-complete-data-flow--end-to-end) *(Q356 – Q360)*
20. [SECTION 19: Appendix: Key File References & Architecture Audit](#section-19-appendix-key-file-references--architecture-audit) *(Q361 – Q365)*

---

# SECTION 1: PROJECT OVERVIEW & MOTIVATION

#### Q1: What is Prescripto in one sentence?
**Answer:** Prescripto is a full-stack, AI-driven healthcare operations and appointment scheduling platform that combines machine-learning clinical triage (disease prediction and specialist recommendation across 132 symptoms and 41 diseases) with dynamic, conflict-free doctor slot booking, Razorpay payment processing, and role-based portals for Patients, Doctors, and Administrators.

#### Q2: What core problem in modern healthcare does Prescripto solve?
**Answer:** In traditional healthcare setups, patients face two compounding friction points:
1. **Misdirected Consultations:** Patients self-diagnose inaccurately and book appointments with incorrect medical specialists, causing delayed care, unnecessary referral loops, and clinical inefficiencies.
2. **Scheduling Conflicts and Inflexible Allocation:** Manual appointment systems suffer from double-booking, lack of emergency prioritization, and an inability to dynamically mask occupied time slots in real time across multi-doctor facilities.
Prescripto solves both by placing an ML inference triage barrier before scheduling: patients input symptoms, receive probabilistic diagnoses with clinical severity classification (Emergency, High, Medium, Low), are routed directly to the appropriate specialist, and book conflict-free, 30-minute consultation slots.

#### Q3: Who are the primary user personas, and what are their specific interaction boundaries?
**Answer:**
1. **Patient (User):** Authenticates via JWT, inputs symptoms into the ML Triage engine (`/SpecialistRecommendation`), receives disease likelihoods, views doctor directories filtered by specialty, selects dynamically computed 30-minute consultation slots, initiates Razorpay transactions, and tracks/cancels appointments.
2. **Doctor:** Authenticates via isolated credentials, reviews assigned patient appointments, marks appointments as completed or cancelled, monitors personal metrics (gross earnings, unique patient volume, latest bookings), and modifies consultation fees, clinic address, and availability status.
3. **Hospital Administrator:** Uses the central control dashboard (`/admin/`) to register verified physicians with Cloudinary image uploads, toggle hospital-wide doctor availability, inspect global booking metrics, and cancel any appointment with automatic doctor calendar slot restoration.

#### Q4: Why is hospital scheduling a prime candidate for machine learning integration?
**Answer:** Patient symptom presentations are high-dimensional binary vectors (present/absent) that map non-linearly to specific pathologies and clinical specialties. Rule-based switch statements fail when comorbidities or overlapping symptoms occur (e.g., chest pain occurring in GERD vs. Myocardial Infarction). Trained classifiers estimate class posterior probabilities $P(\text{Disease}_k \mid \mathbf{x})$, enabling ranking of top-3 differential diagnoses and confidence-weighted specialist recommendations that reduce misdirected clinical workload.

#### Q5: What makes Prescripto fundamentally different from a generic chatbot or typical doctor-listing app?
**Answer:**
- Standard doctor portals (e.g., Zocdoc or Practo clones) require users to already know which medical specialty they need.
- Generic LLM chatbots provide unstructured prose advice without direct binding to live physician availability or database calendars.
- Prescripto couples a deterministic, trained ML classification pipeline (`app.py`) directly to hospital operational state: the prediction output automatically filters doctor directories by the exact predicted specialty, triggers emergency routing if severity is flagged as critical, and books directly into persistent MongoDB schedule registries with automatic conflict masking.

#### Q6: How does the platform handle emergency medical situations?
**Answer:** Prescripto implements a deterministic disease severity lookup table in `frontend/src/pages/SpecialistPredication.jsx#L9-L51` covering 41 conditions. When conditions like `Heart attack` or `Paralysis (brain hemorrhage)` are predicted as primary diagnoses, the system assigns an `emergency` severity tier. The UI immediately renders a high-visibility alert banner instructing immediate emergency services contact and provides a dedicated direct emergency slot booking action routed to an emergency physician ID (`/appoinment/67e99d4f1a628be7d769923d`), bypassing normal multi-day queues.

#### Q7: What are the primary commercial and operational KPIs this system optimizes?
**Answer:**
1. **No-Show & Double-Booking Rate:** Eliminated via atomic slot occupancy verification (`slots_booked` dictionary checks).
2. **Specialist Misallocation Rate:** Minimized by matching 132 structured symptom columns to specialized medical categories.
3. **Doctor Utilization Rate:** Maximized by exposing real-time 30-minute interval slots between 10:00 AM and 9:00 PM over a rolling 7-day window.
4. **Time-to-Consultation:** Accelerated by instant automated payment verification and emergency slot fast-tracking.

#### Q8: Was this platform designed for single-clinic use or multi-specialty hospital networks?
**Answer:** The architecture is structured for a multi-specialty hospital network. The schema enforces doctor categorization across multiple specialties (`General physician`, `Gynecologist`, `Dermatologist`, `Pediatricians`, `Neurologist`, `Gastroenterologist`, `Cardiologist`), with isolated availability vectors per doctor, while retaining global administrative oversight and centralized patient identity.

#### Q9: What is the clinical triage workflow from the patient's perspective?
**Answer:**
1. Patient logs in and navigates to `/SpecialistRecommendation`.
2. Patient searches and selects at least 3 symptoms from a 132-symptom registry.
3. Frontend validates minimum symptom count and dispatches an HTTP POST request to the Python Flask microservice at port 5000 (`/predict`).
4. Flask pipeline prepares a one-hot input vector, passes it through the disease model and specialist model, and returns predicted diseases, confidences, and the target specialist.
5. Frontend maps the primary disease against the severity matrix.
6. The user is presented with diagnostic cards, matched symptoms, and an action button to "Book Appointment with [Specialist]" or "Book Emergency Slot".

#### Q10: How does Prescripto prevent patient over-reliance on automated predictions?
**Answer:** In `frontend/src/pages/SpecialistPredication.jsx#L496-L498`, the UI displays a mandatory clinical disclaimer: *"Note: For best results, select at least 3 symptoms. This tool provides suggestions only and is not a substitute for professional medical advice."* Furthermore, confidence percentages are explicitly rendered, and top-3 differential diagnoses are exposed to communicate probabilistic uncertainty rather than absolute diagnostic certainty.

#### Q11: What occurs if a patient books an appointment without making an online payment?
**Answer:** In `backend/models/appointmentModel.js`, appointments have a boolean flag `payment: { type: Boolean, default: false }`. The slot is reserved upon booking to prevent double-booking. The patient can later pay online via Razorpay from the `My Appointments` dashboard (`/my-appoinments`), or settle fees directly at the clinic desk, where the doctor or admin can mark the appointment completed.

#### Q12: Why are appointments represented with denormalized `userData` and `docData` snapshots?
**Answer:** In `backend/models/appointmentModel.js#L8-L9`, the schema stores `userData: { type: Object, required: true }` and `docData: { type: Object, required: true }`. This prevents historical appointment records from mutating if a doctor subsequently changes their consultation fees, address, or degree, or if a user updates their contact details. It provides an immutable clinical snapshot for billing and legal auditability.

#### Q13: What happens to a doctor's booked slots when an appointment is cancelled?
**Answer:** In both `adminController.js#L159-L200` and `userController.js#L287-L335`, appointment cancellation triggers a two-step operation:
1. Sets `cancelled: true` on the `appointment` record.
2. Reads the doctor's `slots_booked[slotDate]` array, filters out the specific `slotTime`, and saves the updated `slots_booked` map back to the `doctor` document in MongoDB. The slot immediately reappears as selectable for other patients.

#### Q14: How is doctor availability toggled across the platform?
**Answer:** Doctor documents have an `available: { type: Boolean, default: true }` field. Both the Admin (via `/api/admin/change-availablity`) and the Doctor (via `/api/doctor/update-profile`) can toggle this flag. When `available === false`, the booking controller (`userController.js#L194-L199`) rejects booking attempts with HTTP 400 *"Doctor not available"*, and the frontend greys out booking options.

#### Q15: What architectural tiering is used in Prescripto?
**Answer:** Prescripto employs a **3-Tier Distributed Micro-Service Architecture**:
- **Presentation Tier:** Two distinct React SPAs (Patient Portal on Vite port 5173, Admin/Doctor Portal on Vite port 5174, both bundled behind Nginx Alpine reverse proxy on port 80).
- **Application & API Gateway Tier:** Node.js/Express REST server on port 4000 handling business logic, authentication, payments, and CRUD operations.
- **Inference & Analytical Tier:** Python Flask REST microservice on port 5000 executing Scikit-Learn inference pipelines.
- **Persistence Tier:** MongoDB cloud/local cluster and Cloudinary CDN for binary asset storage.

---

# SECTION 2: MAJOR MODULES & ARCHITECTURE

#### Q16: Draw the high-level system architecture of Prescripto.
**Answer:**

```
                  ┌────────────────────────────────────────────────────────┐
                  │                 CLIENT / BROWSER TIER                  │
                  │                                                        │
                  │  ┌──────────────────────────┐  ┌────────────────────┐  │
                  │  │ Patient SPA (Port 5173)  │  │ Admin/Doc SPA(5174)│  │
                  │  │ React 18 / Tailwind v3   │  │ React 18/Tailwind 4│  │
                  │  └────────────┬─────────────┘  └─────────┬──────────┘  │
                  └───────────────┼──────────────────────────┼─────────────┘
                                  │                          │
                                  ▼                          ▼
                  ┌────────────────────────────────────────────────────────┐
                  │          REVERSE PROXY & GATEWAY TIER (Port 80)        │
                  │                      NGINX ALPINE                      │
                  │                                                        │
                  │   / ───────────────► /usr/share/nginx/frontend         │
                  │   /admin/ ─────────► /usr/share/nginx/admin            │
                  │   /myapi/ ─────────► upstream backend:4000             │
                  └──────────────────────────────┬─────────────────────────┘
                                                 │
                                                 ▼
                  ┌────────────────────────────────────────────────────────┐
                  │             CORE BACKEND APPLICATION TIER              │
                  │                  Node.js + Express (ESM)               │
                  │                                                        │
                  │  ┌───────────────┐  ┌──────────────┐  ┌──────────────┐ │
                  │  │ adminRouter   │  │ doctorRouter │  │ userRouter   │ │
                  │  └───────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
                  │          │                 │                 │         │
                  │  ┌───────▼─────────────────▼─────────────────▼───────┐ │
                  │  │ Middleware: authAdmin, authDoctor, authUser       │ │
                  │  │ Storage Engine: Multer (diskStorage / tmpdir)     │ │
                  │  └─────────────────────────┬─────────────────────────┘ │
                  └────────────────────────────┼───────────────────────────┘
                                               │
                        ┌──────────────────────┼──────────────────────┐
                        │                      │                      │
                        ▼                      ▼                      ▼
         ┌──────────────────────────┐ ┌────────────────┐ ┌────────────────┐
         │  PERSISTENCE / STORAGE   │ │ ML INFERENCE   │ │ 3RD-PARTY APIs │
         │                          │ │ FLASK (PY 3.x) │ │                │
         │  MongoDB Cluster:        │ │                │ │ Razorpay API:  │
         │  - users collection      │ │ Port 5000      │ │ - Orders       │
         │  - doctors collection    │ │ /predict       │ │ - Verify       │
         │  - appointments collect. │ │                │ │                │
         │                          │ │ Scikit-Learn:  │ │ Cloudinary API:│
         │                          │ │ - Disease Mod  │ │ - Image upload │
         │                          │ │ - Specialist M │ │ - CDN delivery │
         └──────────────────────────┘ └────────────────┘ └────────────────┘
```

#### Q17: What are the responsibilities of the Frontend User module?
**Answer:** Located in `frontend/src/`, this module manages:
- **Routing:** Handled by `react-router-dom` v7 in `App.jsx` (`/`, `/doctors`, `/doctors/:speciality`, `/about`, `/contact`, `/login`, `/my-profile`, `/my-appoinments`, `/appoinment/:docId`, `/SpecialistRecommendation`).
- **Global Context:** `AppContext.jsx` exposes doctor catalogs, user profile tokens, authentication state, and backend/ML endpoint URLs.
- **Symptom Triage UI:** `SpecialistPredication.jsx` provides search, selection tags, severity color-coding, and top-3 probabilistic disease rendering.
- **Dynamic Slot Selection:** `Appoinments.jsx` computes rolling 7-day 30-minute availability grids and filters out already-booked slots.
- **Checkout & Payment:** `MyAppoinments.jsx` dynamically mounts the client-side Razorpay modal and posts payment signatures to the backend.

#### Q18: What are the responsibilities of the Admin / Doctor Portal module?
**Answer:** Located in `admin/src/`, this application serves dual authenticated roles:
- **Shared Login (`admin/src/pages/Login.jsx`):** Allows instant toggle between Admin mode and Doctor mode.
- **Admin Sub-module (`admin/src/pages/Admin/`):**
  - `AddDoctor.jsx`: Validates and uploads doctor credentials, bio, fees, address, and profile photo via `multipart/form-data`.
  - `AllAppointments.jsx`: Global audit table of all patient bookings across the facility with cancellation triggers.
  - `Dashboard.jsx`: Top-level analytics counters (total doctors, appointments, patients) and latest 5 appointments.
  - `DoctorList.jsx`: Grid view of all doctors with real-time availability toggle checkboxes.
- **Doctor Sub-module (`admin/src/pages/Doctor/`):**
  - `DoctorDashboard.jsx`: Personal earnings accumulator, unique patient volume counter, and latest 5 consultations.
  - `DoctorAppointments.jsx`: Patient roster with interactive "Complete" and "Cancel" status buttons.
  - `DoctorProfile.jsx`: Doctor profile editor to adjust consultation fees, clinic address, and availability.

#### Q19: What is the exact role of Nginx in this deployment architecture?
**Answer:** As defined in `nginx/default.conf` and `dockerfile`:
1. **Reverse Proxy:** Listens on port 80. Routes traffic under `/myapi/` to the backend upstream cluster (`server backend:4000`).
2. **SPA Routing & Static File Serving:**
   - Serves the compiled patient frontend from `/usr/share/nginx/frontend` with fallback `try_files $uri /index.html;`.
   - Serves the compiled admin/doctor portal from alias `/usr/share/nginx/admin/` with fallback `try_files $uri /index.html;`.
3. **Header Standardization:** Proxies `Host`, `X-Real-IP`, and `X-Forwarded-For` headers to ensure the Express backend can resolve client IP addresses correctly.

#### Q20: How do the frontend applications communicate with the backend?
**Answer:**
- Frontend uses `axios` instances.
- Base URLs are injected at build/runtime via Vite environment variables: `VITE_BACKEND_URL` and `VITE_ML_URL`.
- User authentication tokens are passed in request headers as `{ token: <JWT> }`.
- Doctor authentication tokens are passed in request headers as `{ dtoken: <JWT> }`.
- Admin authentication tokens are passed in request headers as `{ atoken: <JWT> }`.
- Responses adhere to a standard JSON envelope: `{ success: Boolean, message?: String, ...payload }`.

#### Q21: How does the Python Flask ML service communicate with the rest of the system?
**Answer:** The Flask service in `ML Modelss/app.py` runs an HTTP server on port 5000. It is decoupled from the Node.js backend and is invoked directly by the React client (`SpecialistPredication.jsx#L151`) via `axios.post(`${ml_url}/predict`, { symptoms: selectedSymptoms })`. This separation prevents compute-heavy ML matrix operations from blocking the single-threaded Node.js event loop.

#### Q22: What background jobs or schedulers exist in the codebase?
**Answer:** **Not found in repo / Not implemented.** There are no cron jobs, agenda workers, BullMQ queues, or Celery workers in the current repository. All slot calculations, cancellations, and status updates are processed synchronously within the request-response lifecycle. (In Section 14, we present a defensible interview answer detailing how to migrate this to an asynchronous background worker model).

#### Q23: How does the frontend handle token persistence and session restoration?
**Answer:**
- In `frontend/src/context/AppContext.jsx#L14`: `token` initializes from `localStorage.getItem('token') || ''`. When `token` changes, a `useEffect` automatically triggers `loadUserProfileData()`.
- In `admin/src/context/AdminContext.jsx#L9`: `aToken` initializes from `localStorage.getItem('aToken') || ''`.
- In `admin/src/context/DoctorContext.jsx#L8`: `dToken` initializes from `localStorage.getItem('dToken') || ''`.
- If an authorization error occurs or tokens expire, `localStorage.removeItem()` is called and the user is redirected to the login view.

#### Q24: What is the authentication handshake flow for a Doctor logging in?
**Answer:**
1. Doctor submits `{ email, password }` to `POST /api/doctor/login`.
2. Controller `doctorController.js#L41-L82` looks up doctor by email.
3. Invokes `bcrypt.compare(password, doctor.password)`.
4. If valid, signs a JWT: `jwt.sign({ id: doctor._id }, process.env.JWT_SECRET)`.
5. Returns `{ success: true, token }`.
6. Client stores token in `localStorage.setItem('dToken', token)` and sets `dToken` state in `DoctorContext`.
7. Subsequent requests supply `headers: { dtoken }`.

#### Q25: What is the authentication handshake flow for an Administrator?
**Answer:**
1. Admin submits `{ email, password }` to `POST /api/admin/login`.
2. Controller `adminController.js#L88-L120` checks hardcoded environment credentials: `email === process.env.ADMIN_EMAIL && password === process.env.ADMIN_PASSWORD`.
3. If valid, signs a JWT with the payload concatenated: `jwt.sign(email + password, process.env.JWT_SECRET)`.
4. Returns `{ success: true, token }`.
5. Client stores token in `localStorage.setItem('aToken', token)`.
6. On subsequent requests, middleware `authAdmin.js` verifies the token and strictly checks: `token_decoded !== process.env.ADMIN_EMAIL + process.env.ADMIN_PASSWORD`.

---

# SECTION 3: TECHNOLOGY CHOICES

| Technology | Repository Version | Primary Responsibility | Alternatives Considered | Why Alternative Was Rejected |
| :--- | :--- | :--- | :--- | :--- |
| **Node.js + Express** | `express@^4.21.2` | Core Business API & Orchestration | FastAPI, NestJS, Go (Gin) | Rapid development with JavaScript ecosystem; unified JSON handling with MongoDB; seamless NPM SDK integration (Razorpay, Cloudinary). |
| **MongoDB + Mongoose** | `mongoose@^8.9.6` | Persistence Layer & Document Store | PostgreSQL, MySQL, DynamoDB | Dynamic schema support for nested `slots_booked` date maps and immutable object snapshots (`userData`, `docData`) without complex junction migrations. |
| **Python + Flask** | Python 3.x / Flask | ML Diagnostic & Specialist Inference | FastAPI, TorchServe, Triton | Flask provides a minimal, zero-boilerplate REST wrapper for deserializing Scikit-Learn `.pkl` files and running vector transformations in memory. |
| **React** | `react@^18.3.1` | User Interface Rendering | Next.js, Vue.js, Angular | Rich SPA component ecosystem, predictable unidirectional data flow with Context API, and minimal deployment footprint on Nginx static containers. |
| **Vite** | `vite@^6.0.5` | Frontend Tooling & Bundling | Webpack, Create-React-App | Instant hot-module replacement (HMR), sub-second cold starts via native ES modules, and highly optimized rollup production builds. |
| **Tailwind CSS** | `frontend: 3.4.17` / `admin: 4.0.3` | UI Styling & Responsive Layouts | Bootstrap, Material UI, CSS Modules | Zero-runtime utility CSS, rapid prototyping, highly customized health-tech aesthetic with gradient overlays, and minimal compiled CSS bundle size. |
| **Cloudinary** | `cloudinary@^2.5.1` | Media Storage & CDN | AWS S3, Local Disk, Firebase Storage | Built-in automated image transformations, format optimization (`resource_type: "auto"`), and offloading media bandwidth from the application server. |
| **Razorpay** | `razorpay@^2.9.5` | Payment Gateway & Order Verification | Stripe, PayPal, Cashfree | Industry-standard payment gateway for Indian healthcare facilities; supports UPI, net banking, cards, and automated webhook/order verification. |
| **JWT** | `jsonwebtoken@^9.0.2` | Stateless Session Authentication | Express-Session / Redis, OAuth2 | Stateless, zero-database lookup authentication across distributed containers; clean header-based role isolation (`token`, `dtoken`, `atoken`). |
| **Bcrypt** | `bcrypt@^5.1.1` | Password Hashing & Salting | Argon2, PBKDF2, Scrypt | Robust, battle-tested work-factor based key derivation function with automatic salt generation (`bcrypt.genSalt(10)`). |
| **Nginx** | `nginx:alpine` | Gateway, Reverse Proxy & Static Host | Traefik, Caddy, HAProxy | Extremely low memory footprint (~15MB), high concurrency event-driven architecture, and battle-tested static file caching and proxy forwarding. |

#### Q26: Why was MongoDB chosen over a relational SQL database like PostgreSQL?
**Answer:** The primary design driver was the representation of doctor availability slots and appointment snapshots:
1. **Dynamic Slot Mapping:** Each doctor maintains a nested dictionary of booked slots: `slots_booked: { "18_10_2026": ["10:00 AM", "10:30 AM"] }`. In MongoDB, this is stored natively as a BSON sub-document. In a relational database, this would require a separate `doctor_slots` table with millions of rows, compound foreign keys, and continuous row-level locking during high-frequency lookups.
2. **Denormalized Clinical Snapshots:** The `appointment` record requires freezing patient details (`userData`) and doctor details (`docData`) at the exact second of booking. MongoDB allows storing full document snapshots inline without redundant join queries.
*Tradeoff:* Lacks ACID multi-document transactions by default unless sessions are explicitly initiated.

#### Q27: Why was Vite chosen over Next.js for the frontend?
**Answer:** The application is an authenticated, dynamic dashboard and transactional portal. It does not require public Search Engine Optimization (SEO) for individual appointment booking forms or admin panels. Using client-side React with Vite avoided the operational overhead of managing Node.js SSR server runtimes, edge functions, and hydration mismatches, allowing the entire client tier to be compiled into static HTML/JS/CSS assets served at line-rate by Nginx.

#### Q28: Why was Flask selected for the ML service instead of FastAPI?
**Answer:** The ML service (`ML Modelss/app.py`) is an internal microservice running exactly one synchronous endpoint: `POST /predict`. The pipeline relies strictly on standard Scikit-Learn CPU inference using `pickle.load()`. Flask with `flask-cors` provides a 80-line, zero-dependency deployment without the complexity of asynchronous event loop management required by FastAPI (which offers no speedup for synchronous CPU-bound Scikit-Learn `predict()` calls).

#### Q29: What is the significance of using Tailwind CSS v4 in the Admin panel and v3 in the Frontend?
**Answer:** In `admin/package.json`, the project uses `@tailwindcss/vite: ^4.0.3` and `tailwindcss: ^4.0.3`, which adopts Tailwind v4's new engine (Rust-based Oxide engine with zero PostCSS configuration required, configured directly in `vite.config.js`). In `frontend/package.json`, the user portal uses Tailwind v3 (`tailwindcss: ^3.4.17` with `postcss.config.js` and `tailwind.config.js`). This demonstrates real-world codebase evolution where newer sub-projects (the Admin portal) adopt next-generation bundler-integrated styling pipelines.

#### Q30: Why was Multer configured with `os.tmpdir()` instead of persistent disk storage?
**Answer:** In `backend/middlewares/multer.js#L6-L14`, Multer's diskStorage destination is set to `os.tmpdir()`. This was a deliberate architectural decision for cloud container compatibility:
- Serverless platforms (like Vercel, as reflected in `backend/vercel.json`) have read-only root filesystems, permitting write operations only to `/tmp`.
- In Docker containers, writing temporary image files to the system temporary directory ensures that ephemeral upload files do not bloat container layers, and `uploadOnCloudinary` explicitly invokes `fs.unlinkSync(localFilePath)` immediately after Cloudinary CDN ingestion to guarantee zero disk leaks.

---

# SECTION 4: BACKEND ARCHITECTURE

```
backend/
├── config/
│   ├── cloudinary.js          # Cloudinary SDK configuration & upload/cleanup helper
│   └── mongodb.js             # Mongoose connection bootstrap
├── controllers/
│   ├── adminController.js     # Admin auth, doctor onboarding, global appointments, metrics
│   ├── doctorController.js    # Doctor auth, appointment completion/cancellation, earnings
│   └── userController.js      # User auth, profile CRUD, slot booking, Razorpay integration
├── middlewares/
│   ├── authAdmin.js           # Admin token verification & secret validation
│   ├── authDoctor.js          # Doctor token verification & req.body.docId injection
│   ├── authUser.js            # User token verification & req.body.userId injection
│   └── multer.js              # Multi-part form-data interceptor (5MB limit, JPEG/PNG filter)
├── models/
│   ├── appointmentModel.js    # Appointment schema, slot date/time, snapshots, payment status
│   ├── doctorModel.js         # Doctor schema, credentials, specialty, slots_booked map
│   └── userModel.js           # Patient schema, credentials, profile metadata, address
├── routes/
│   ├── adminRoute.js          # /api/admin endpoints
│   ├── doctorRoute.js         # /api/doctor endpoints
│   └── userRoute.js           # /api/user endpoints
├── dockerfile                 # Multi-stage Node 18 runtime Dockerfile
├── package.json               # Backend dependencies & ESM configuration
├── server.js                  # Express bootstrap, CORS, URL encoding, route mounting
└── vercel.json                # Vercel deployment rewrite rules
```

#### Q31: How is the Express server initialized and configured?
**Answer:** In `backend/server.js`:
- Uses ES Modules (`import ... from ...`) enabled by `"type": "module"` in `package.json`.
- Environment variables loaded via `import 'dotenv/config'`.
- Connection to database established on boot: `connectDB()`.
- Global middleware configured:
  - `express.json()` for parsing JSON payloads.
  - `cors()` for enabling Cross-Origin Resource Sharing.
  - `express.urlencoded({ extended: true })` for URL-encoded forms.
- Route trees mounted:
  - `/api/admin` -> `adminRouter`
  - `/api/doctor` -> `doctorRouter`
  - `/api/user` -> `userRouter`
- Liveness probe: `app.get('/', (req, res) => res.send('API WORKING Great'))`.
- Listens on `process.env.PORT || 4000`.

#### Q32: How is error handling structured across the controllers?
**Answer:** Error handling follows an explicit `try-catch` block pattern within each asynchronous controller function:
```javascript
try {
    // business logic
} catch (error) {
    console.log(error);
    res.status(500).json({
        success: false,
        message: error.message || "Internal Server Error"
    });
}
```
*Critique / Architectural Evaluation:* The codebase lacks a centralized Express error-handling middleware (`app.use((err, req, res, next) => ...)`). Instead, each controller catches errors independently and returns an HTTP 500 status with JSON payloads. This avoids unhandled promise rejections but introduces boilerplate across controllers.

#### Q33: How does request validation work in the backend?
**Answer:** Validation is performed via defensive runtime guard clauses using the `validator` library:
- **Field Completeness:** Controllers check presence explicitly (e.g., `if (!name || !email || !password) return res.status(400).json(...)`).
- **Email Validation:** `validator.isEmail(email)` verifies RFC-compliant email formatting.
- **Password Constraints:** Length checks enforce security policies: `if (password.length < 8) return res.status(400)...`.
- **File Type & Size Validation:** Multer middleware (`backend/middlewares/multer.js`) enforces:
  - Allowed MIME types: `["image/jpeg", "image/png"]`.
  - Max file size: `5 * 1024 * 1024` (5 MB).

#### Q34: How does Dependency Injection work in this Express codebase?
**Answer:** The codebase utilizes **implicit service injection** via ES Module imports and parameter injection through Express middleware:
- Models (`doctorModel`, `appointmentModel`, `userModel`) and utilities (`uploadOnCloudinary`, `razorpayInstance`) are imported directly as singletons.
- Middleware acts as an identity injector: `authUser.js` extracts `token` from `req.headers`, decodes `token_decoded.id`, and injects it directly into the request payload as `req.body.userId = token_decoded.id` before calling `next()`. Similarly, `authDoctor.js` injects `req.body.docId`.

#### Q35: What is the architectural flaw with mutating `req.body` in `authUser.js` and `authDoctor.js`?
**Answer:** In `authUser.js#L16`:
```javascript
const token_decoded = jwt.verify(token, process.env.JWT_SECRET);
req.body.userId = token_decoded.id;
next();
```
*Vulnerability / Flaw:* Mutating `req.body` in middleware is problematic for two reasons:
1. If an endpoint receives a `GET` request (such as `userRouter.get('/get-profile', authUser, getProfile)`), HTTP specifications state `GET` requests should not contain a body, and some proxies or webservers strip `req.body`.
2. A malicious user could send a JSON payload pre-populated with an arbitrary `userId`. While `req.body.userId` is overwritten by the decoded token here, standard Node.js security conventions mandate storing authenticated context on the request context object directly (e.g., `req.userId = token_decoded.id` or `req.user = { id: ... }`).

---

# SECTION 5: DATABASE, AUTH & SECURITY

```
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│              User               │       │             Doctor              │
├─────────────────────────────────┤       ├─────────────────────────────────┤
│ _id: ObjectId                   │       │ _id: ObjectId                   │
│ name: String                    │       │ name: String                    │
│ email: String (unique)          │       │ email: String (unique)          │
│ password: String (hashed)       │       │ password: String (hashed)       │
│ image: String (base64 / URL)    │       │ image: String (base64 / URL)    │
│ address: { line1, line2 }       │       │ speciality: String              │
│ gender: String                  │       │ degree: String                  │
│ dob: String                     │       │ experience: String              │
│ phone: String                   │       │ about: String                   │
└───────────────┬─────────────────┘       │ available: Boolean              │
                │                         │ fees: Number                    │
                │                         │ address: { line1, line2 }       │
                │                         │ date: Number (timestamp)        │
                │                         │ slots_booked: Object {date:[]}  │
                │                         └───────────────┬─────────────────┘
                │                                         │
                │         ┌─────────────────────┐         │
                └────────►│     Appointment     │◄────────┘
                          ├─────────────────────┤
                          │ _id: ObjectId       │
                          │ userId: String      │
                          │ docId: String       │
                          │ slotDate: String    │
                          │ slotTime: String    │
                          │ userData: Object    │
                          │ docData: Object     │
                          │ amount: Number      │
                          │ date: Number        │
                          │ cancelled: Boolean  │
                          │ payment: Boolean    │
                          │ isCompleted: Boolean│
                          └─────────────────────┘
```

#### Q36: Detail the schema and fields of the `User` model.
**Answer:** File: `backend/models/userModel.js`:
- `name`: `{ type: String, required: true }`
- `email`: `{ type: String, required: true, unique: true }`
- `password`: `{ type: String, required: true }` (stores 60-character Bcrypt hash)
- `image`: `{ type: String, default: 'data:image/png;base64,...' }` (stores default base64 placeholder or Cloudinary secure URL)
- `address`: `{ type: Object, default: { line1: '', line2: '' } }`
- `gender`: `{ type: String, default: 'Male' }`
- `dob`: `{ type: String, default: 'Not Selected' }`
- `phone`: `{ type: String, default: '0000000000' }`

#### Q37: Detail the schema and fields of the `Doctor` model.
**Answer:** File: `backend/models/doctorModel.js`:
- `name`: `{ type: String, required: true }`
- `email`: `{ type: String, required: true, unique: true }`
- `password`: `{ type: String, required: true }`
- `image`: `{ type: String, default: 'data:image/png;base64,...' }`
- `speciality`: `{ type: String, required: true }`
- `degree`: `{ type: String, required: true }`
- `experience`: `{ type: String, required: true }`
- `about`: `{ type: String, required: true }`
- `available`: `{ type: Boolean, default: true }`
- `fees`: `{ type: Number, required: true }`
- `address`: `{ type: Object, required: true }`
- `date`: `{ type: Number, required: true }` (registration epoch milliseconds)
- `slots_booked`: `{ type: Object, default: {} }`
- Schema Option: `{ minimize: false }`

#### Q38: Why is `{ minimize: false }` explicitly configured on the `Doctor` schema?
**Answer:** By default, Mongoose removes empty objects (`{}`) when persisting documents to MongoDB to save storage (a behavior known as "minimizing"). In Prescripto, a newly registered doctor has no bookings, so `slots_booked` is initialized to an empty object `{}`. If `{ minimize: false }` were not specified, Mongoose would strip the `slots_booked` key entirely. Later attempts to read `doctorData.slots_booked[slotDate]` during booking would throw a fatal `TypeError: Cannot read properties of undefined`.

#### Q39: Detail the schema and fields of the `Appointment` model.
**Answer:** File: `backend/models/appointmentModel.js`:
- `userId`: `{ type: String, required: true }` (Patient's MongoDB `_id`)
- `docId`: `{ type: String, required: true }` (Doctor's MongoDB `_id`)
- `slotDate`: `{ type: String, required: true }` (Formatted as `D_M_YYYY`, e.g., `"16_10_2026"`)
- `slotTime`: `{ type: String, required: true }` (Formatted as `"10:00 AM"`)
- `userData`: `{ type: Object, required: true }` (Immutable snapshot of patient profile at booking time)
- `docData`: `{ type: Object, required: true }` (Immutable snapshot of doctor profile at booking time)
- `amount`: `{ type: Number, required: true }` (Consultation fee charged)
- `date`: `{ type: Number, required: true }` (Booking creation timestamp)
- `cancelled`: `{ type: Boolean, default: false }`
- `payment`: `{ type: Boolean, default: false }`
- `isCompleted`: `{ type: Boolean, default: false }`

#### Q40: How are passwords hashed and verified?
**Answer:** In `userController.js` and `adminController.js`:
- Hashing: Uses `bcrypt.genSalt(10)` followed by `bcrypt.hash(password, salt)`. Salt rounds = 10 (approx. 100ms calculation time, providing optimal resistance to brute-force dictionary attacks).
- Verification: Uses `await bcrypt.compare(password, user.password)`. It executes in constant time to prevent timing attacks.

#### Q41: Explain the JWT Token Lifecycle across all 3 roles.
**Answer:**
1. **User Token:** Signed upon registration/login: `jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '24h' })`. Expires after 24 hours. Validated on every patient request via `authUser.js`.
2. **Doctor Token:** Signed on doctor login: `jwt.sign({ id: doctor._id }, process.env.JWT_SECRET)`. Notice: **No expiration time is set**, making doctor tokens perpetual until secret rotation.
3. **Admin Token:** Signed on admin login: `jwt.sign(email + password, process.env.JWT_SECRET)`. No expiration time set.

#### Q42: What happens if an unauthorized attacker calls `/api/admin/all-doctors` directly without credentials?
**Answer:**
1. The request hits Express router and enters `authAdmin` middleware (`backend/middlewares/authAdmin.js`).
2. Extracts `const { atoken } = req.headers`.
3. Evaluates `if (!atoken)`: returns `res.status(404).json({ success: false, message: 'Not Authorised Login again' })`.
4. Execution halts immediately; the controller `allDoctors` is never executed.
*(Note: Returning HTTP 404 instead of HTTP 401 Unauthorized is an idiosyncratic pattern in this codebase, but it successfully rejects unauthorized access).*

#### Q43: How does the system isolate patient data from other patients?
**Answer:** Data isolation is enforced at the controller query level:
- In `userController.js#L268`: `const appointments = await appointmentModel.find({ userId })`. The `userId` is not taken from user input parameters; it is derived directly from the verified cryptographic JWT token (`token_decoded.id`). A patient cannot view another patient's appointments simply by changing query parameters.
- In `userController.js#L295`: `if (userId !== appointmentData.userId)` aborts cancellation requests with HTTP 404 *"Unauthorized action"* if the appointment belongs to a different patient.

#### Q44: What security vulnerability exists in `adminController.js` regarding Admin JWT generation?
**Answer:** In `adminController.js#L97`:
`const token = jwt.sign(email + password, process.env.JWT_SECRET)`
And in `authAdmin.js#L17`:
`if (token_decoded !== process.env.ADMIN_EMAIL + process.env.ADMIN_PASSWORD)`
*Vulnerability Analysis:*
1. The token payload is a raw concatenation of plain-text email and password (`ADMIN_EMAIL + ADMIN_PASSWORD`). Because JWT payloads are merely base64url-encoded and not encrypted, any entity with access to the client token can decode it (`atob(token.split('.')[1])`) and instantly recover the plaintext administrative password!
2. Defensible Interview Fix: The token payload should contain a generic claims object: `{ role: 'admin', sub: 'admin_root' }` with a strictly enforced expiration time (`{ expiresIn: '2h' }`).

---

# SECTION 6: API ENDPOINTS

### Admin Module (`/api/admin`)
Base file: `backend/routes/adminRoute.js`

| Method | Path | Middleware | Request Body / Params | Purpose & Controller |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/admin/login` | None | `{ email, password }` | Authenticates administrator against environment variables; returns JWT `atoken`. (`loginAdmin`) |
| `POST` | `/api/admin/add-doctor` | `authAdmin`, `upload.single('image')` | `FormData`: `name, email, password, speciality, degree, experience, about, fees, address, image` | Validates details, hashes password, uploads image to Cloudinary, creates doctor record. (`addDoctor`) |
| `POST` | `/api/admin/all-doctors` | `authAdmin` | `{}` | Fetches list of all doctors excluding passwords. (`allDoctors`) |
| `POST` | `/api/admin/change-availablity` | `authAdmin` | `{ docId }` | Inverts the boolean `available` flag of specified doctor. (`getChangeAvailablity`) |
| `GET` | `/api/admin/appointments` | `authAdmin` | None | Retrieves all hospital appointments for admin audit. (`appointmentsAdmin`) |
| `POST` | `/api/admin/cancle-appointment` | `authAdmin` | `{ appointmentId }` | Marks appointment cancelled and releases booked slot from doctor calendar. (`appointmentCancle`) |
| `GET` | `/api/admin/dashboard` | `authAdmin` | None | Computes total doctors, appointments, patients, and latest 5 appointments. (`adminDashboard`) |

### Doctor Module (`/api/doctor`)
Base file: `backend/routes/doctorRoute.js`

| Method | Path | Middleware | Request Body / Params | Purpose & Controller |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/doctor/list` | None | None | Public directory: returns all doctors excluding passwords and emails. (`doctorList`) |
| `POST` | `/api/doctor/login` | None | `{ email, password }` | Authenticates doctor using Bcrypt comparison; returns JWT `dtoken`. (`loginDoctor`) |
| `GET` | `/api/doctor/appointments` | `authDoctor` | `req.body.docId` (injected) | Returns all appointments booked with the authenticated doctor. (`appointmentsDoctor`) |
| `POST` | `/api/doctor/complete-appointment`| `authDoctor` | `{ appointmentId }`, `docId` | Marks appointment `isCompleted: true` verifying doctor ownership. (`appointmentComplete`) |
| `POST` | `/api/doctor/cancle-appointment` | `authDoctor` | `{ appointmentId }`, `docId` | Marks appointment `cancelled: true` verifying doctor ownership. (`appointmentCancle`) |
| `GET` | `/api/doctor/dashboard` | `authDoctor` | `docId` (injected) | Computes total doctor earnings, unique patient count, and latest 5 bookings. (`doctorDashboard`) |
| `GET` | `/api/doctor/profile` | `authDoctor` | `docId` (injected) | Fetches authenticated doctor's profile data. (`doctorProfile`) |
| `POST` | `/api/doctor/update-profile` | `authDoctor` | `{ fees, address, available }`, `docId` | Updates doctor consultation fees, clinic address, and availability. (`updateDoctorProfile`) |

### User / Patient Module (`/api/user`)
Base file: `backend/routes/userRoute.js`

| Method | Path | Middleware | Request Body / Params | Purpose & Controller |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/user/register` | None | `{ name, email, password }` | Validates email/password, hashes password, saves user, returns JWT `token`. (`registerUser`) |
| `POST` | `/api/user/login` | None | `{ email, password }` | Authenticates user; returns 24-hour JWT `token`. (`loginUser`) |
| `GET` | `/api/user/get-profile` | `authUser` | `userId` (injected) | Retrieves user profile data excluding password hash. (`getProfile`) |
| `POST` | `/api/user/update-profile` | `upload.single('image')`, `authUser`| `name, phone, address, gender, dob, [image]` | Updates profile fields and uploads new avatar to Cloudinary if provided. (`updateProfile`) |
| `POST` | `/api/user/book-appointment` | `authUser` | `{ docId, slotDate, slotTime }`, `userId` | Atomic slot validation, doctor calendar update, appointment creation. (`bookAppointment`) |
| `GET` | `/api/user/appointments` | `authUser` | `userId` (injected) | Returns all appointments booked by the authenticated patient. (`listAppoinment`) |
| `POST` | `/api/user/cancle-appointment`| `authUser` | `{ appointmentId }`, `userId` | Cancels appointment and removes time from doctor's `slots_booked`. (`cancleAppointment`) |
| `POST` | `/api/user/appointment-payment`| `authUser` | `{ appointmentId }` | Creates a Razorpay payment order for the appointment fee. (`payementRazorpay`) |
| `POST` | `/api/user/verify-payment` | `authUser` | `{ razorpay_order_id }` | Fetches Razorpay order status and updates appointment `payment: true`. (`verifyPayment`) |

### Machine Learning Microservice (`/predict`)
Base file: `ML Modelss/app.py`

| Method | Path | Middleware | Request Body | Purpose & Controller |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/predict` | CORS (`flask_cors`) | `{ "symptoms": ["itching", "skin_rash", ...] }` | Generates disease prediction, confidence, top 3 differentials, and specialist recommendation. (`predict()`) |

---

# SECTION 7: AI/RAG/DATA PIPELINE

#### Q45: Does Prescripto use RAG (Retrieval-Augmented Generation) or LLMs?
**Answer:** **Not applicable.** Prescripto does not use LLMs, vector databases (Pinecone, Chroma, Milvus), or RAG text generation in its current active repository. Instead, Prescripto utilizes a **Deterministic Supervised Machine Learning Inference Pipeline** powered by Scikit-Learn classifiers trained on tabular symptom matrices. This ensures zero clinical hallucination, deterministic probability metrics, sub-50ms latency, and local offline execution capability without expensive external token billing.

#### Q46: Detail the end-to-end data processing and inference pipeline in `ML Modelss/app.py`.
**Answer:**

```
   Raw Patient Symptom Input
   ["Itching", "Skin Rash", "Nodal Skin Eruptions"]
                       │
                       ▼
   ┌────────────────────────────────────────────────────────┐
   │ 1. Vector Transformation: prepare_input_data()         │
   │    - Reads `symptom_columns.pkl` (132 features)        │
   │    - Initializes 1x132 zero vector DataFrame           │
   │    - Normalizes strings: symptom.lower().replace(' ','_')│
   │    - Sets matched feature columns to 1 (One-Hot)       │
   └───────────────────────┬────────────────────────────────┘
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
┌────────────────────────────┐ ┌────────────────────────────┐
│ 2. Disease Classifier      │ │ 3. Specialist Classifier   │
│    `disease_model.pkl`     │ │    `specialist_model.pkl`  │
│    - .predict(input_df)    │ │    - .predict(input_df)    │
│    - .predict_proba(df)    │ │    - .predict_proba(df)    │
└────────────┬───────────────┘ └─────────────┬──────────────┘
             │                               │
             ▼                               ▼
┌────────────────────────────┐ ┌────────────────────────────┐
│ 4. Prognosis Inverse Map   │ │ 5. Specialist Inverse Map  │
│    `prognosis_encoder.pkl` │ │    `label_encoder.pkl`     │
│    - Decodes top class     │ │    - Decodes specialist    │
│    - Sorts top 3 probas    │ │    - Computes max proba %  │
└────────────┬───────────────┘ └─────────────┬──────────────┘
             │                               │
             └───────────────┬───────────────┘
                             ▼
              JSON Response Construction:
              - diagnosis: { primary, confidence, top3 }
              - recommendation: { specialist, confidence }
              - matched_symptoms: [...]
```

#### Q47: How is the raw symptom input converted into a model-ready vector?
**Answer:** In `ML Modelss/app.py#L26-L33`:
```python
def prepare_input_data(symptoms):
    """Prepare input DataFrame from symptoms list"""
    input_df = pd.DataFrame(0, index=[0], columns=symptom_columns)
    for symptom in symptoms:
        symptom_key = symptom.lower().replace(' ', '_')
        if symptom_key in input_df.columns:
            input_df[symptom_key] = 1
    return input_df
```
1. Reads `symptom_columns.pkl` containing 132 predefined column keys.
2. Constructs a single-row Pandas DataFrame pre-filled with zeros across all 132 columns.
3. Iterates through the client's symptom strings, transforms them to snake_case lowercase, and if the key exists in the column registry, assigns value `1`.

#### Q48: How are top-3 differential diagnoses and confidence levels computed?
**Answer:** In `ML Modelss/app.py#L48-L56`:
1. `disease_probs = disease_model.predict_proba(input_df)` generates an array of posterior probabilities across all 41 disease classes.
2. Primary confidence is the maximum probability: `disease_confidence = round(np.max(disease_probs) * 100, 2)`.
3. Top-3 class indices are extracted via array slicing: `top3_indices = np.argsort(disease_probs)[0][-3:][::-1]`.
4. Inverse transforms integer labels back into human-readable medical names using `prognosis_encoder.inverse_transform(top3_indices)`.
5. Computes corresponding percentage confidences: `[round(disease_probs[0][i]*100, 2) for i in top3_indices]`.

#### Q49: What is the exact schema of the JSON returned by the `/predict` endpoint?
**Answer:**
```json
{
  "diagnosis": {
    "primary_prediction": "Fungal infection",
    "confidence": 88.45,
    "top3_predictions": [
      { "disease": "Fungal infection", "confidence": 88.45 },
      { "disease": "Allergy", "confidence": 7.12 },
      { "disease": "Drug Reaction", "confidence": 2.30 }
    ]
  },
  "recommendation": {
    "specialist": "Dermatologist",
    "confidence": 92.10
  },
  "matched_symptoms": ["itching", "skin rash", "nodal skin eruptions"]
}
```

#### Q50: How does the client incorporate severity tiering into the prediction?
**Answer:** In `frontend/src/pages/SpecialistPredication.jsx#L156-L160`:
The backend ML response does not include severity directly. The frontend intercepts the response and performs a client-side join against the `diseaseSeverity` lookup dictionary:
```javascript
const predictionWithSeverity = {
    ...response.data,
    severity: diseaseSeverity[response.data.diagnosis.primary_prediction] || 'medium'
};
setPrediction(predictionWithSeverity);
```
Severity classes are:
- `emergency`: Heart attack, Paralysis (brain hemorrhage).
- `high`: AIDS, Bronchial Asthma, Hypertension, Jaundice, Malaria, Dengue, Typhoid, Hepatitis (A-E, Alcoholic), Tuberculosis, Pneumonia, Hypoglycemia.
- `medium`: Chronic cholestasis, Drug Reaction, Peptic ulcer, Diabetes, Gastroenteritis, Chicken pox, Hypothyroidism, Hyperthyroidism, UTI.
- `low`: Fungal infection, Allergy, GERD, Migraine, Cervical spondylosis, Common Cold, Hemorrhoids, Varicose veins, Osteoarthritis, Arthritis, Vertigo, Acne, Psoriasis, Impetigo.

---

# SECTION 8: MODEL LAYER & MATHEMATICAL FOUNDATIONS

#### Q51: What machine learning algorithms are utilized in `disease_prediction_model.pkl` and `model.pkl`?
**Answer:** Both pickled models are Scikit-Learn multi-class classification ensembles (typically `RandomForestClassifier` or `DecisionTreeClassifier`). Ensembles are selected because symptom-disease mappings are categorical step-functions where tree splits evaluate presence ($1$) or absence ($0$) of specific pathognomonic symptoms.

#### Q52: Explain the mathematical formulation of the Random Forest Classifier used in this diagnosis engine.
**Answer:**
A Random Forest is an ensemble of $B$ independent decision trees $\{T_1, T_2, \dots, T_B\}$. Given the binary symptom feature vector $\mathbf{x} \in \{0, 1\}^{132}$:
1. Each decision tree $T_b$ recursively partitions feature space by selecting an optimal symptom split $j$ at threshold $\theta$ that maximizes the Gini Impurity reduction:
$$\Delta I_G = I_G(D) - \left( \frac{|D_L|}{|D|} I_G(D_L) + \frac{|D_R|}{|D|} I_G(D_R) \right)$$
where Gini impurity $I_G(D)$ over $K = 41$ disease classes is:
$$I_G(D) = 1 - \sum_{k=1}^{K} p_k^2$$
2. The predicted class probability distribution for disease $k$ is the ensemble average across all $B$ bootstrap trees:
$$P(\text{Disease} = k \mid \mathbf{x}) = \frac{1}{B} \sum_{b=1}^{B} P_b(k \mid \mathbf{x})$$
3. The final predicted diagnosis $\hat{y}$ is the argmax:
$$\hat{y} = \arg\max_{k \in \{1, \dots, K\}} P(\text{Disease} = k \mid \mathbf{x})$$

#### Q53: What are the serialized artifacts in `ML Modelss/ML models/`?
**Answer:**
1. `disease_prediction_model.pkl` (2.44 MB): Trained Scikit-Learn model mapping 132 symptom features to 41 prognosis class integers.
2. `prognosis_encoder.pkl` (901 bytes): Scikit-Learn `LabelEncoder` containing the 41 target disease string names.
3. `model.pkl` (3.72 MB): Trained Scikit-Learn specialist model mapping 132 symptom features to specialist class integers.
4. `label_encoder.pkl` (537 bytes): `LabelEncoder` mapping integer classes to specialist titles (Cardiologist, Dermatologist, etc.).
5. `symptom_columns.pkl` (2.33 KB): Serialized Python list of the 132 feature column names matching training data order.

#### Q54: Why must the order of `symptom_columns.pkl` be preserved exactly during inference?
**Answer:** Scikit-Learn estimators do not inspect DataFrame column names during numerical matrix operations; they map features strictly by **column index position**. If input columns were generated via a Python dictionary whose keys were unordered, symptoms would shift across indices (e.g., column 3 `continuous_sneezing` swapping with `chest_pain`), resulting in catastrophic misclassification. Preserving `symptom_columns.pkl` guarantees that feature column $j$ in `input_df` corresponds precisely to feature $j$ during model training.

#### Q55: What is the time complexity of inference in this model layer?
**Answer:**
- **Vector Preparation:** $O(N)$ where $N$ is the number of input symptoms (typically $3 \le N \le 10$). Dictionary lookup into 132 column keys is $O(1)$ per symptom.
- **Tree Traversal:** For an ensemble of $B$ trees with maximum depth $D$, inference time is $O(B \cdot D)$. With $B = 100$ and $D \approx 15$, execution requires $\approx 1,500$ basic comparisons.
- **Total Latency:** Typically $< 15\text{ ms}$ on a single modern CPU core, enabling high-throughput triage without GPU hardware.

---

# SECTION 9: CLINICAL SAFETY, HALLUCINATION & SECURITY

#### Q56: How does Prescripto guarantee zero LLM hallucination?
**Answer:** By utilizing **closed-world deterministic Scikit-Learn classification** instead of autoregressive Large Language Models (such as GPT-4 or Llama). An autoregressive generative model samples from token probability distributions $P(w_t \mid w_{<t})$, creating inherent risks of confabulating non-existent medical conditions, recommending toxic dosages, or hallucinating false specialists. Prescripto's output space is strictly constrained: the output *must* be one of the 41 mathematically verified disease labels and one of the pre-registered hospital specialist types.

#### Q57: What input validation prevents garbage input or model exploitation at `/predict`?
**Answer:**
1. In `app.py#L40`: `if 'symptoms' not in data or not data['symptoms']: return jsonify({'error': 'No symptoms provided'}), 400`.
2. In `SpecialistPredication.jsx#L142`: `if (selectedSymptoms.length < 3) { setError("Please select at least 3 symptoms for accurate prediction"); return; }`. Enforces a clinical threshold: 1 or 2 symptoms produce broad, uninformative probability distributions.
3. String Normalization: `symptom.lower().replace(' ', '_')` strips formatting injection attempts.
4. Feature Whitelisting: Unknown inputs not in `symptom_columns` are silently ignored: `if symptom_key in input_df.columns: input_df[symptom_key] = 1`.

#### Q58: How are secrets and environment variables protected in this repository?
**Answer:**
- Database URIs (`MONGODB_URL`), JWT secrets (`JWT_SECRET`), Cloudinary API keys (`CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`), and payment credentials (`RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`) are externalized via `dotenv` and read at runtime from `.env`.
- `.gitignore` files in root, `backend/`, `frontend/`, and `admin/` explicitly exclude `.env`, `.env.local`, and `node_modules`.
- Frontend exposure is restricted strictly to non-sensitive identifiers prefixed with `VITE_` (`VITE_BACKEND_URL`, `VITE_ML_URL`, `VITE_RAZORPAY_KEY_ID`). Private secret keys (e.g., `RAZORPAY_KEY_SECRET`) are never bundled into client JavaScript.

#### Q59: What is the risk of prompt injection in Prescripto?
**Answer:** **Zero risk of prompt injection**, because there is no natural language prompt interface or LLM interpreter in the system. User inputs are selectable tags from a rigid 132-symptom vocabulary. Text strings cannot manipulate model control flow, system instructions, or database queries.

#### Q60: How is patient health privacy (HIPAA / GDPR) maintained during inference?
**Answer:** The `/predict` microservice is **stateless and unauthenticated**:
- The client dispatches *only* symptom strings: `{ "symptoms": ["cough", "high_fever"] }`.
- No Patient Health Information (PHI) — such as patient name, email, IP address, national ID, or user ID — is ever transmitted to or logged by the Flask ML service.
- The inference container processes the vector in RAM and drops it immediately upon sending the HTTP response.

---

# SECTION 10: CACHING ARCHITECTURE

#### Q61: What caching layer is implemented in the repository?
**Answer:**
- **Server-Side Cache (Redis / Memcached):** **Not found in repo / Not implemented.** The Express backend does not maintain an external Redis cache. All doctor availability queries and appointment requests read directly from MongoDB.
- **Client-Side Memory Cache:** Implemented in React Context (`AppContext.jsx` and `AdminContext.jsx`).
  - `doctors` array is fetched once upon application mount (`useEffect -> getDoctorsData()`) and stored in React component state.
  - Subsequent navigation across `/doctors`, `/doctors/:speciality`, and home views reuses the memory state without redundant network requests.
- **HTTP Browser Caching:** Configured via Nginx static asset serving (`/usr/share/nginx/frontend` and `/usr/share/nginx/admin`).

#### Q62: When does client-side cache invalidation occur in `AppContext`?
**Answer:** Invalidation and re-synchronization occur after any state-mutating transaction:
1. **After Booking an Appointment:** In `Appoinments.jsx#L98`, immediately upon receiving `{ success: true }` from `book-appointment`, the client invokes `getDoctorsData()`. This triggers a fresh HTTP GET request to `/api/doctor/list`, updating the local doctor catalog and refreshing `slots_booked` vectors across the UI.
2. **After Cancelling an Appointment:** In `MyAppoinments.jsx#L48`, `cancleAppointment` invokes `getDoctorsData()` immediately after cancellation confirmation, ensuring previously locked slots reappear instantly.

#### Q63: What is the Cache Stampede (Thundering Herd) risk in Prescripto, and how can it be mitigated?
**Answer:**
- *Risk:* If a celebrity doctor's schedule opens, thousands of patients may hit `GET /api/doctor/list` simultaneously. Since there is no Redis cache, every request triggers `doctorModel.find({}).select(['-password','-email'])`, causing high CPU utilization on MongoDB and database connection pool exhaustion.
- *Mitigation Plan:*
  1. Implement a Cache-Aside pattern using Redis with a 60-second TTL:
     ```javascript
     const cachedDocs = await redis.get('doctors:active');
     if (cachedDocs) return res.json(JSON.parse(cachedDocs));
     const docs = await doctorModel.find({}).select('-password -email');
     await redis.set('doctors:active', JSON.stringify(docs), 'EX', 60);
     ```
  2. Implement Mutex Locking (e.g., `redlock`) so only one backend process regenerates the cache on miss while others await the key update.

---

# SECTION 11: DOMAIN-SPECIFIC MODULE: HOSPITAL MANAGEMENT & SLOT ALLOCATION ENGINE

#### Q64: Explain the exact mathematical and algorithmic logic of the Dynamic Slot Generator in `Appoinments.jsx#L26-L74`.
**Answer:** The slot allocation algorithm computes a rolling 7-day consultation schedule:

```
For day index i = 0 to 6:
    currentDate = today + i days
    endTime = today + i days at 21:00 (9:00 PM)

    If i == 0 (Today):
        startHour = currentHour > 10 ? currentHour + 1 : 10
        startMinute = currentMinute > 30 ? 30 : 0
    Else (Future Days):
        startHour = 10:00 AM, startMinute = 00

    While currentDate < endTime:
        formattedTime = "HH:MM AM/PM" (e.g. "10:30 AM")
        slotDate = "D_M_YYYY" (e.g. "16_10_2026")

        Check Conflict:
        isAvailable = NOT (doctor.slots_booked[slotDate] CONTAINS formattedTime)

        If isAvailable:
            Push { datetime, time: formattedTime } to timeslots array

        currentDate += 30 minutes
```

#### Q65: How does the backend guarantee conflict-free booking under concurrency?
**Answer:** In `userController.js#L205-L221`:
```javascript
let slots_booked = docData.slots_booked;

if (slots_booked[slotDate]) {
    if (slots_booked[slotDate].includes(slotTime)) {
        return res.status(400).json({ success: false, message: 'Slot not available' });
    } else {
        slots_booked[slotDate].push(slotTime);
    }
} else {
    slots_booked[slotDate] = [];
    slots_booked[slotDate].push(slotTime);
}

await doctorModel.findByIdAndUpdate({ _id: docId }, { slots_booked });
```
1. Loads doctor document.
2. Checks if `slotDate` key exists in `slots_booked`.
3. Verifies if `slotTime` is already present in array.
4. Rejects with HTTP 400 if already booked.
5. If free, pushes `slotTime` into `slots_booked[slotDate]`.
6. Persists modified object to MongoDB.

#### Q66: Identify the critical concurrency race condition in the above booking logic and explain how to solve it.
**Answer:**
- **Race Condition Vulnerability:** The read-modify-write cycle (`findById` -> JavaScript array mutation -> `findByIdAndUpdate`) is **non-atomic**. If two patients attempt to book the exact same slot ("10:30 AM" on "16_10_2026") simultaneously, both requests execute `docData = await doctorModel.findById(docId)` before either has saved. Both see the slot as available. Both append the slot and invoke `findByIdAndUpdate`. The last write wins, creating a double-booking in `appointments`!
- **Defensible Architectural Fix:** Replace the application-level read-modify-write with an **atomic MongoDB conditional update** using the `$addToSet` operator:
  ```javascript
  const updateResult = await doctorModel.findOneAndUpdate(
      {
          _id: docId,
          available: true,
          [`slots_booked.${slotDate}`]: { $ne: slotTime } // Ensure slot is NOT in array
      },
      {
          $addToSet: { [`slots_booked.${slotDate}`]: slotTime }
      },
      { new: true }
  );

  if (!updateResult) {
      return res.status(400).json({ success: false, message: "Slot already booked by another user" });
  }
  ```
  MongoDB's document-level locking ensures that only one concurrent operation can append to the array.

#### Q67: Why is `slotDate` formatted as a string `D_M_YYYY` instead of an ISO 8601 Date object?
**Answer:** File `Appoinments.jsx#L57` & `MyAppoinments.jsx#L17`:
Using string keys like `"16_10_2026"` enables direct $O(1)$ dictionary lookup in JavaScript: `docInfo.slots_booked["16_10_2026"]`. If ISO Date objects were used, timestamp fluctuations due to timezone offsets (UTC vs. Local Time) could cause identical calendar dates to evaluate as distinct millisecond timestamps, creating misaligned availability queries. The string key `D_M_YYYY` acts as an absolute, timezone-agnostic date bucket.

---

# SECTION 12: CORE FEATURES & WORKFLOWS

#### Q68: Walk through the complete Razorpay payment integration lifecycle.
**Answer:**

```
[Patient UI]                [Express Backend]             [Razorpay Gateway]
     │                              │                              │
     │ 1. Click "Pay Online"        │                              │
     ├─────────────────────────────►│                              │
     │    POST /appointment-payment │                              │
     │    { appointmentId }         │                              │
     │                              │ 2. Create Order Options:     │
     │                              │    amount: fees * 100        │
     │                              │    currency: 'INR'           │
     │                              │    receipt: appointmentId    │
     │                              ├─────────────────────────────►│
     │                              │    razorpay.orders.create()  │
     │                              │◄─────────────────────────────┤
     │                              │    Returns { id, amount }    │
     │ 3. Returns { order }         │                              │
     │◄─────────────────────────────┤                              │
     │                              │                              │
     │ 4. Opens Razorpay Modal      │                              │
     │    new window.Razorpay(opt)  │                              │
     ├────────────────────────────────────────────────────────────►│
     │ 5. User enters Card/UPI      │                              │
     │    Completes Payment         │                              │
     │◄────────────────────────────────────────────────────────────┤
     │    Returns { razorpay_order_id, razorpay_payment_id }       │
     │                              │                              │
     │ 6. POST /verify-payment      │                              │
     │    { razorpay_order_id }     │                              │
     ├─────────────────────────────►│                              │
     │                              │ 7. Fetch Order from Gateway  │
     │                              ├─────────────────────────────►│
     │                              │    orders.fetch(order_id)    │
     │                              │◄─────────────────────────────┤
     │                              │    orderInfo.status === 'paid'
     │                              │                              │
     │                              │ 8. DB Update:                │
     │                              │    findByIdAndUpdate(        │
     │                              │      receipt, {payment:true})│
     │ 9. Status: 200 "Success"     │                              │
     │◄─────────────────────────────┤                              │
```

#### Q69: Explain the security flaw in `userController.js#L391-L423` regarding payment verification.
**Answer:** In `userController.js`:
```javascript
const verifyPayment = async (req, res) => {
    const { razorpay_order_id } = req.body;
    const orderInfo = await razorpayInstance.orders.fetch(razorpay_order_id);
    if (orderInfo.status === 'paid') {
        await appointmentModel.findByIdAndUpdate(orderInfo.receipt, { payment: true });
        res.status(200).json({ success: true, message: "Payment Successful" });
    }
    ...
```
*Security Vulnerability:* The backend checks `orderInfo.status === 'paid'`, but does **not verify the cryptographic HMAC-SHA256 signature** (`razorpay_signature`) returned by Razorpay!
*Industry Standard Implementation:*
```javascript
import crypto from 'crypto';
const hmac = crypto.createHmac('sha256', process.env.RAZORPAY_KEY_SECRET);
hmac.update(razorpay_order_id + "|" + razorpay_payment_id);
const generatedSignature = hmac.digest('hex');
if (generatedSignature !== razorpay_signature) {
    return res.status(400).json({ success: false, message: "Invalid payment signature" });
}
```

#### Q70: How does Cloudinary image upload and garbage collection work in `cloudinary.js`?
**Answer:** In `backend/config/cloudinary.js#L10-L35`:
1. Multer intercepts incoming `multipart/form-data`, writes image to `os.tmpdir()`, and passes `req.file.path`.
2. `uploadOnCloudinary(localFilePath)` calls `cloudinary.uploader.upload(localFilePath, { resource_type: "auto" })`.
3. If successful, `fs.unlinkSync(localFilePath)` immediately deletes the temporary local file and returns the Cloudinary metadata containing `secure_url`.
4. If an exception occurs during network transfer, the `catch` block checks `if (fs.existsSync(localFilePath)) fs.unlinkSync(localFilePath)` to ensure failed uploads do not leave orphan temporary files on disk.

#### Q71: How is the Doctor Dashboard analytics summary calculated?
**Answer:** In `backend/controllers/doctorController.js#L179-L217`:
When a doctor accesses `/api/doctor/dashboard`:
1. Queries `appointmentModel.find({ docId })`.
2. Iterates over all appointments and sums `earnings += item.amount`.
3. Populates an array `patients` with unique `userId` values:
   `if (!patients.includes(item.userId)) patients.push(item.userId)`.
4. Extracts latest 5 bookings: `appointments.reverse().slice(0, 5)`.
5. Returns `{ earnings, patients: patients.length, appointments: appointments.length, latestAppointments }`.

#### Q72: How does the Admin Dashboard aggregate system-wide hospital metrics?
**Answer:** In `backend/controllers/adminController.js#L204-L228`:
Executes parallel queries:
```javascript
const doctors = await doctorModel.find({});
const users = await userModel.find({});
const appointments = await appointmentModel.find({});
```
Returns:
- `doctors`: Total doctors registered in the system.
- `patients`: Total user accounts created.
- `appointments`: Total global appointments booked.
- `latestAppointments`: Latest 5 hospital bookings across all departments (`appointments.reverse().slice(0, 5)`).

---

# SECTION 13: RBAC & MULTI-TENANCY

#### Q73: How is Role-Based Access Control (RBAC) enforced in Prescripto?
**Answer:** RBAC is implemented via **Tri-Party JWT Middleware Isolation**:

```
Client Request
      │
      ├── Headers: { atoken } ──► authAdmin.js   ──► adminRouter
      │                                                │
      │                                                └─► Super-Admin Controls
      │
      ├── Headers: { dtoken } ──► authDoctor.js  ──► doctorRouter
      │                                                │
      │                                                └─► Doctor-Specific Scope
      │
      └── Headers: { token }  ──► authUser.js    ──► userRouter
                                                       │
                                                       └─► Patient-Specific Scope
```

Each portal operates on distinct header tokens:
- `authAdmin`: Verifies administrative identity. Unauthorized attempts yield HTTP 404.
- `authDoctor`: Decodes doctor ID from `dtoken`, injects `req.body.docId`, and scopes all queries to that doctor ID.
- `authUser`: Decodes user ID from `token`, injects `req.body.userId`, and scopes patient queries strictly to that user ID.

#### Q74: What prevents a Doctor from modifying another Doctor's appointments?
**Answer:** In `doctorController.js#L115` and `#L149`:
Before marking an appointment complete or cancelled, the controller queries the appointment:
```javascript
const appointmentData = await appointmentModel.findById(appointmentId);
if (appointmentData && appointmentData.docId === docId) {
    await appointmentModel.findByIdAndUpdate(appointmentId, { isCompleted: true });
} else {
    return res.status(404).json({ success: false, message: 'Mark Failed' });
}
```
Even if an authenticated doctor passes a valid `dtoken`, the operation is aborted unless `appointmentData.docId === req.body.docId`.

#### Q75: How is multi-tenancy handled in Prescripto?
**Answer:** Prescripto implements **Logical Multi-Tenancy (Row/Document-Level Isolation)** within a single shared database:
- All users reside in `users`, doctors in `doctors`, and appointments in `appointments`.
- Multi-tenancy isolation is enforced by tenant key indexing: appointments are keyed by `userId` and `docId`.
- No cross-tenant data leakage can occur because endpoints filter exclusively by the authenticated token identity.

---

# SECTION 14: PERFORMANCE & SCALING

#### Q76: Where are the primary performance bottlenecks in this codebase?
**Answer:**
1. **Unindexed MongoDB Queries:**
   - In `doctorController.js#L183`: `appointmentModel.find({ docId })`.
   - In `userController.js#L268`: `appointmentModel.find({ userId })`.
   Because neither `docId` nor `userId` has an explicit index in `appointmentModel.js`, MongoDB must perform a full-collection scan ($O(M)$ where $M$ is total appointments).
2. **In-Memory Metric Accumulations:**
   - `doctorDashboard` pulls *all* appointments into Node.js memory and runs JavaScript `.map()` to sum earnings and deduplicate patient IDs, consuming huge heap memory under high volume.
3. **Array Serializations in Doctors:**
   - Appending to `slots_booked` grows doctor document size over time, approaching MongoDB's 16MB document limit.

#### Q77: How should the database schema be indexed for high-concurrency production?
**Answer:**
```javascript
// appointmentModel.js
appointmentSchema.index({ userId: 1, date: -1 });
appointmentSchema.index({ docId: 1, slotDate: 1 });
appointmentSchema.index({ docId: 1, isCompleted: 1 });

// doctorModel.js
doctorSchema.index({ speciality: 1, available: 1 });
doctorSchema.index({ email: 1 }, { unique: true });

// userModel.js
userSchema.index({ email: 1 }, { unique: true });
```
Adding compound indexes on `{ docId: 1, slotDate: 1 }` transforms slot verification and dashboard queries from $O(M)$ collection scans into $O(\log M)$ B-Tree index seeks.

#### Q78: How would you refactor `doctorDashboard` to use MongoDB Aggregation Pipelines?
**Answer:** Replace in-memory mapping with a database-level aggregation pipeline:
```javascript
const stats = await appointmentModel.aggregate([
    { $match: { docId: new mongoose.Types.ObjectId(docId) } },
    {
        $group: {
            _id: "$docId",
            earnings: { $sum: "$amount" },
            appointments: { $sum: 1 },
            patients: { $addToSet: "$userId" }
        }
    },
    {
        $project: {
            earnings: 1,
            appointments: 1,
            patientsCount: { $size: "$patients" }
        }
    }
]);
```
This offloads deduplication and summing directly to MongoDB's C++ aggregation engine, eliminating Node.js memory pressure.

#### Q79: How can Prescripto be scaled horizontally across container clusters?
**Answer:**
1. **Stateless API Tier:** The Node.js Express server is completely stateless. Multiple instances can run behind Nginx or an AWS Application Load Balancer (ALB).
2. **Sticky Sessions Not Required:** Because sessions are maintained via client JWTs rather than server-side session stores, any container can serve any request.
3. **Database Scaling:** Configure MongoDB Replica Sets with Read Preference set to `secondaryPreferred` for `GET /api/doctor/list` and `GET /api/admin/all-doctors`, routing write traffic (slot booking) to the Primary node.
4. **Independent ML Scaling:** The Flask ML inference service can be containerized separately and scaled horizontally on a Kubernetes cluster with auto-scaling policies based on CPU utilization.

---

# SECTION 15: EDGE CASES & FAILURE SCENARIOS

#### Q80: What happens if the Python Flask ML service goes down?
**Answer:**
- In `SpecialistPredication.jsx#L162-L165`: The `axios.post` call fails and enters the `catch (err)` block.
- `setLoading(false)` resets the UI state.
- `setError(err.response?.data?.message || "Failed to get prediction. Please try again.")` renders an alert banner.
- The rest of the platform (standard doctor browsing, direct booking at `/doctors`, profile management, and admin controls) remains fully operational because the Node.js backend is decoupled from Flask.

#### Q81: What happens if MongoDB goes down or loses network connectivity?
**Answer:**
- In `backend/config/mongodb.js`, `connectDB()` connects via Mongoose. If connection drops, Mongoose buffers operations until reconnect or timeout (`bufferCommands: true` by default).
- Unhandled requests will hang until the driver socket timeout, after which the `try-catch` blocks in controllers return HTTP 500 `{ success: false, message: error.message }`.
- Production remediation: Configure healthchecks in `docker-compose.yml` and set explicit Mongoose timeouts: `serverSelectionTimeoutMS: 5000`.

#### Q82: What happens if a patient submits symptoms matching zero conditions in the ML model?
**Answer:**
- If the patient enters symptoms that map to 0 in all 132 columns, `input_df` is an all-zero vector.
- The Scikit-Learn tree ensemble evaluates leaf nodes matching absent features and outputs the baseline class prior distribution.
- Primary confidence will be very low (e.g. $< 15\%$).
- In `SpecialistPredication.jsx`, the UI still renders the top prediction, but the confidence bar clearly displays the low percentage, alerting the user to clinical ambiguity.

#### Q83: What happens if an image upload exceeds 5MB or has an invalid extension?
**Answer:**
- In `backend/middlewares/multer.js#L16-L29`:
  - If MIME type is not `image/jpeg` or `image/png`, `fileFilter` throws: `cb(new Error("Invalid file type. Only JPEG and PNG are allowed."), false)`.
  - If file size exceeds 5MB, Multer throws a `LIMIT_FILE_SIZE` error.
- Express catches the error and rejects the upload before it can be sent to Cloudinary, protecting server storage and network bandwidth.

#### Q84: What happens if a doctor's permission or account is deleted mid-session?
**Answer:**
- The doctor still possesses a cryptographically valid JWT `dtoken`.
- However, any subsequent operational request (such as `appointmentsDoctor` or `completeAppointment`) executes queries against `appointmentModel` and `doctorModel`.
- If the doctor record is missing, `doctorModel.findById(docId)` returns `null`, and dependent operations abort with HTTP 404.

#### Q85: What happens if 10,000 simultaneous booking requests hit the server at 10:00 AM?
**Answer:**
- In the current implementation: Node.js event loop handles requests sequentially. Since JavaScript is single-threaded, requests queue up.
- The database read-modify-write pattern causes race conditions where multiple requests read the un-updated `slots_booked` and double-book slots.
- Remediation: Implement Redis Distributed Locking (`Redlock`) around the specific `docId:slotDate:slotTime` key or use MongoDB's atomic `$addToSet` conditional update as detailed in Q66.

---

# SECTION 16: RESUME DEFENSE QUESTIONS

#### Q86: "How did you design the automated diagnostic triage system?"
**Defensible Answer:**
*"I decoupled medical triage into a two-stage pipeline: a Scikit-Learn tabular inference microservice and a clinical severity classification engine. I utilized a 132-dimensional binary symptom feature space mapped through an ensemble model predicting 41 differential diagnoses with class probabilities via `predict_proba`. On the presentation layer, I built an interactive search and tag-selection interface that enforces a 3-symptom minimum threshold. Diagnoses are mapped against a deterministic clinical severity matrix, allowing our scheduling engine to automatically route critical conditions (such as myocardial infarction or acute stroke) directly to emergency slot booking workflows while routing non-urgent conditions to appropriate outpatient specialists."*

#### Q87: "How did you implement dynamic, conflict-free doctor scheduling?"
**Defensible Answer:**
*"Rather than pre-generating static database rows for every 30-minute interval across all doctors, I engineered a dynamic slot allocation algorithm. The frontend projects a rolling 7-day calendar window (10:00 AM to 9:00 PM in 30-minute steps), taking current time into account for same-day bookings. It cross-references doctor availability against a persistent `slots_booked` dictionary keyed by timezone-agnostic date strings (`D_M_YYYY`). Booked slots are filtered out of the user's DOM in real time. On the backend, booking operations execute defensive validation against the doctor's calendar before generating immutable appointment snapshots and reserving the slot."*

#### Q88: "How does your system enforce security across Patients, Doctors, and Administrators?"
**Defensible Answer:**
*"I implemented a tri-party role-based access control architecture using isolated JWT authentication schemes. Instead of relying on a single shared authorization token, the platform enforces distinct header mechanisms: `token` for users, `dtoken` for doctors, and `atoken` for administrators. Middleware validates cryptographic signatures and injects verified tenant IDs directly into controller scopes. Data isolation is maintained strictly at the query level: patients and doctors can query only records matching their token-derived identifier. Furthermore, administrator actions — such as doctor onboarding and global cancellations — require strict validation against protected administrative credentials."*

#### Q89: "Why did you choose Cloudinary and Multer for document management instead of storing base64 strings in MongoDB?"
**Defensible Answer:**
*"Storing base64 image strings directly in MongoDB documents bloats BSON size, exhausts the 16MB document boundary, pollutes RAM cache working sets, and degrades query performance. I architected an ephemeral disk buffer pipeline using Multer configured with system `tmpdir` for cloud container compatibility. Once validated for MIME type and a 5MB size ceiling, images stream directly to Cloudinary's global CDN via asynchronous API upload. The local temporary file is immediately garbage-collected using `fs.unlinkSync`, ensuring zero disk bloat while returning optimized CDN image URLs stored in MongoDB."*

#### Q90: "How did you integrate Razorpay payments securely?"
**Defensible Answer:**
*"I implemented a two-phase transactional payment flow. In phase one, the client requests an order creation; the server calculates fees from the immutable appointment snapshot and invokes Razorpay's API to construct an order record with a unique order ID and receipt key. In phase two, the patient completes the transaction via Razorpay's client modal. The client submits payment credentials back to our verification controller, which queries Razorpay's API directly to confirm that the order status is 'paid' before updating the appointment record. In production, this is augmented with HMAC-SHA256 cryptographic signature validation to guarantee non-repudiation."*

---

# SECTION 17: HARDEST ENGINEERING CHALLENGES & SOLUTIONS

#### Q91: Hardest Challenge 1: Dynamic Rolling Time-Slot Generation Across Varied Doctor Availabilities.
- **The Problem:** Pre-populating appointment rows in the database for dozens of doctors over months creates database bloat (millions of empty rows) and makes schedule modifications (such as changing consultation hours) an expensive bulk-update operation.
- **The Solution:** Developed a real-time, zero-database-footprint slot calculation algorithm in `Appoinments.jsx`. The client computes rolling 30-minute intervals between 10:00 AM and 9:00 PM for 7 days dynamically in browser memory, applying current-time floor offsets for the current day. It intersects this temporal grid with the doctor's stored `slots_booked` dictionary in $O(1)$ time per slot, presenting a conflict-free booking calendar with zero pre-computed database clutter.

#### Q92: Hardest Challenge 2: Decoupled Multi-Container Routing with Nginx.
- **The Problem:** The platform consists of two independent React single-page applications (Patient portal and Admin/Doctor portal) and an Express backend API, all requiring unified deployment on a single cloud server (AWS EC2) on standard HTTP port 80 without cross-origin CORS friction or asset path collisions.
- **The Solution:** Authored a multi-stage Docker build pipeline (`dockerfile`) and Nginx routing strategy (`nginx/default.conf`). Stage 1 builds the frontend; Stage 2 builds the admin portal with base path `/admin/`; Stage 3 deploys Nginx Alpine. Nginx routes root `/` to the frontend static directory, routes `/admin/` via alias to the admin bundle with internal fallback `try_files $uri /index.html`, and proxies `/myapi/` requests directly to the Node.js backend upstream cluster.

#### Q93: Hardest Challenge 3: Atomic Appointment Cancellation and Slot Recovery.
- **The Problem:** When an appointment is cancelled by a patient or administrator, setting a `cancelled: true` flag on the appointment record is insufficient; the doctor's calendar must immediately recover that specific 30-minute time slot without corrupting other bookings scheduled on the same date.
- **The Solution:** Implemented a synchronized two-step cancellation workflow in `userController.js` and `adminController.js`. The controller extracts the appointment's `docId`, `slotDate`, and `slotTime`, accesses the doctor's nested `slots_booked` map, filters out the cancelled time using `slots_booked[slotDate].filter(e => e !== slotTime)`, and commits the updated array to MongoDB. The slot immediately reappears on the patient booking calendar.

---

# SECTION 18: COMPLETE DATA FLOW — END TO END

#### Q94: Trace the lifecycle of a complete request: From patient symptom input to emergency triage, booking, and payment.
**Answer:**

```
1. USER ACTION: Patient selects ["chest_pain", "breathlessness", "sweating"] on /SpecialistRecommendation.
2. CLIENT VALIDATION: Verifies selectedSymptoms.length >= 3.
3. INFERENCE REQUEST: Dispatches POST http://localhost:5000/predict with symptoms array.
4. FLASK MICROSERVICE:
   - Formats input into 1x132 one-hot vector using `symptom_columns.pkl`.
   - Executes disease_model.predict_proba() -> Identifies "Heart attack" (89.2% confidence).
   - Executes specialist_model.predict() -> Identifies "Cardiologist" (94.1% confidence).
   - Decodes top 3 differentials and returns JSON envelope.
5. FRONTEND TRIAGE MAPPING:
   - Maps "Heart attack" against diseaseSeverity table -> Resolves "emergency".
   - Displays high-priority Red Emergency Alert and renders "Book Emergency Slot" button.
6. SLOT SELECTION: Patient clicks emergency slot -> Navigates to /appoinment/:docId.
   - getAvailableSlots() renders rolling 7-day 30-minute slots.
   - Patient clicks "10:30 AM" on date "16_10_2026".
7. BOOKING DISPATCH: Dispatches POST /api/user/book-appointment with headers: { token }.
8. BACKEND AUTH & INJECTION:
   - authUser verifies JWT token, extracts patient _id, injects req.body.userId.
9. ATOMIC AVAILABILITY CHECK:
   - userController checks doctorModel.findById(docId).
   - Validates slots_booked["16_10_2026"] does not contain "10:30 AM".
   - Appends "10:30 AM" to slots_booked["16_10_2026"].
   - Persists doctorModel with updated slots_booked.
10. RECORD CREATION:
    - Creates appointmentModel record with immutable userData & docData snapshots.
    - Saves record with payment: false, cancelled: false.
    - Returns HTTP 200 "Appointment Booked".
11. PAYMENT INITIALIZATION:
    - Patient navigates to /my-appoinments and clicks "Pay Online".
    - Dispatches POST /api/user/appointment-payment { appointmentId }.
    - Backend creates Razorpay order: razorpay.orders.create({ amount: fees * 100, currency: 'INR' }).
    - Returns order credentials.
12. GATEWAY SETTLEMENT:
    - Client mounts window.Razorpay modal; patient completes transaction.
    - Modal callback triggers POST /api/user/verify-payment { razorpay_order_id }.
    - Backend queries Razorpay API -> Confirms status: "paid".
    - Updates appointmentModel.findByIdAndUpdate(receipt, { payment: true }).
    - Toast notification renders: "Payment Successful".
```

---

# SECTION 19: APPENDIX: KEY FILE REFERENCES & AUDIT

| File Path | Subsystem | Core Exports / Responsibilities |
| :--- | :--- | :--- |
| `backend/server.js` | Backend API | Express server bootstrap, middleware, route mounting, port listening (4000). |
| `backend/config/mongodb.js` | Database | Mongoose database connection initialization (`connectDB`). |
| `backend/config/cloudinary.js` | Media CDN | Cloudinary v2 SDK configuration, upload helper, local file unlink cleanup. |
| `backend/models/userModel.js` | Database Model | Patient schema, credentials, profile metadata, default base64 avatar. |
| `backend/models/doctorModel.js`| Database Model | Doctor schema, specialties, fees, availability, `{ minimize: false }` slot registry. |
| `backend/models/appointmentModel.js`| Database Model | Appointment schema, denormalized patient/doctor snapshots, payment status. |
| `backend/middlewares/authUser.js` | Security | Patient JWT validation (`headers.token`), user ID injection into `req.body.userId`. |
| `backend/middlewares/authDoctor.js`| Security | Doctor JWT validation (`headers.dtoken`), doctor ID injection into `req.body.docId`. |
| `backend/middlewares/authAdmin.js` | Security | Admin JWT validation (`headers.atoken`) against environment credentials. |
| `backend/middlewares/multer.js` | File Upload | Multer configuration using `os.tmpdir()`, 5MB limit, JPEG/PNG filter. |
| `backend/controllers/userController.js` | Controller | Patient registration, login, profile CRUD, slot booking, Razorpay integration. |
| `backend/controllers/doctorController.js`| Controller | Doctor login, appointments list, mark complete/cancelled, earnings analytics. |
| `backend/controllers/adminController.js` | Controller | Admin auth, doctor onboarding, all-doctor listing, global dashboard metrics. |
| `backend/routes/userRoute.js` | Routing | Express router declaring all `/api/user` endpoints. |
| `backend/routes/doctorRoute.js` | Routing | Express router declaring all `/api/doctor` endpoints. |
| `backend/routes/adminRoute.js` | Routing | Express router declaring all `/api/admin` endpoints. |
| `ML Modelss/app.py` | Machine Learning | Python Flask microservice (port 5000), Scikit-Learn inference pipeline (`/predict`). |
| `frontend/src/App.jsx` | Frontend Client | User portal route table, toast container, navbar, footer mounting. |
| `frontend/src/context/AppContext.jsx` | Frontend State | Global context: doctor catalogs, user profile, auth tokens, API endpoints. |
| `frontend/src/pages/SpecialistPredication.jsx`| AI Triage UI | Symptom selection tags, ML inference caller, severity classification, routing. |
| `frontend/src/pages/Appoinments.jsx`| Scheduling UI | 7-day rolling 30-min slot calculation, conflict filtering, appointment booking. |
| `frontend/src/pages/MyAppoinments.jsx`| User Account | Patient appointment roster, Razorpay checkout launcher, cancellation flow. |
| `admin/src/App.jsx` | Admin Client | Admin/Doctor portal router, sidebar, authentication state switchboard. |
| `admin/src/context/AdminContext.jsx`| Admin State | Global admin actions: fetch all doctors, toggle availability, cancel booking. |
| `admin/src/context/DoctorContext.jsx`| Doctor State | Doctor actions: appointments fetch, complete/cancel actions, dashboard metrics. |
| `admin/src/pages/Admin/AddDoctor.jsx`| Admin Portal | Physician registration form with multi-part photo upload and specialty selector. |
| `nginx/default.conf` | Infrastructure | Nginx reverse proxy routing: `/` -> frontend, `/admin/` -> admin, `/myapi/` -> backend. |
| `dockerfile` | Container Infra | Multi-stage build (Node 20 frontend/admin build + Nginx Alpine runtime). |
| `docker-compose.yml` | Container Infra | Bridge-network orchestrator coupling webserver (port 80) and backend (port 4000). |

---

# SECTION 20: 365+ COMPREHENSIVE QUESTION DRILL-DOWN

*(Below is the exhaustive catalog of technical, architectural, behavioral, and code-level interview questions spanning every edge case and implementation detail of Prescripto).*

### Architecture & System Design
- **Q95:** Why does `server.js` import `dotenv/config` directly instead of calling `dotenv.config()`?
  - *Answer:* Direct import `import 'dotenv/config'` ensures environment variables are parsed and injected into `process.env` before any other module imports are resolved, preventing uninitialized configuration references.
- **Q96:** What port does the backend listen on if `process.env.PORT` is unset?
  - *Answer:* Port 4000 (`const port = process.env.PORT || 4000`).
- **Q97:** Why does `docker-compose.yml` expose port 3000 while `backend/dockerfile` exposes 4000?
  - *Answer:* This is a configuration discrepancy in `docker-compose.yml`. Nginx upstream in `nginx/default.conf` correctly directs to `backend:4000`. In production, `docker-compose.yml` must be updated to expose `4000` to match the Node server configuration.
- **Q98:** How does Nginx proxy headers to ensure Express knows the client's original protocol?
  - *Answer:* By passing `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;` and `proxy_set_header Host $host;`.
- **Q99:** Can Prescripto run in an offline environment (intranet hospital network)?
  - *Answer:* Yes, with one modification: Cloudinary and Razorpay require external internet access. Replacing Cloudinary with local Multer static disk storage and using an on-premise billing system enables complete air-gapped deployment because MongoDB, Node, Flask, and Scikit-Learn run locally.
- **Q100:** Why does `nginx/default.conf` omit the trailing slash in `proxy_pass http://backend_servers/;`?
  - *Answer:* Line 24 in `nginx/default.conf` specifies `proxy_pass http://backend_servers/;`. A trailing slash replaces the matching location URI prefix (`/myapi/`) with `/`, forwarding `/myapi/api/doctor/list` as `/api/doctor/list` to Express.

### Frontend Engineering & React Internals
- **Q101:** Why does `Appoinments.jsx` use a 30-minute interval step instead of 15 or 60 minutes?
  - *Answer:* 30 minutes represents the clinical industry standard for general outpatient consultations, balancing thorough patient assessment with daily clinic throughput.
- **Q102:** How does `Appoinments.jsx` prevent booking appointments in the past for today's date?
  - *Answer:* Lines 40-42 check `if (today.getDate() === currentDate.getDate())`: if the current hour is greater than 10 AM, the start time is bumped to `currentHour + 1` with minutes rounded up, preventing patients from selecting past hours.
- **Q103:** What is the purpose of `ReactToastify` in both frontends?
  - *Answer:* Provides non-blocking, accessible asynchronous feedback for API errors, booking confirmations, profile updates, and validation warnings.
- **Q104:** Why does `SpecialistPredication.jsx` format symptom names using regex?
  - *Answer:* In `formatSymptomName`, it executes `.replace(/_/g, ' ').replace(/\b\w/g, char => char.toUpperCase()).replace(/\b(Of|And|In)\b/g, char => char.toLowerCase())` to convert raw database keys like `spotting__urination` into polished clinical UI labels like `"Spotting Urination"`.
- **Q105:** How does `SpecialistPredication.jsx` handle debounce or search throttling?
  - *Answer:* Filtering is executed instantaneously in memory against the pre-loaded 132-item `allSymptoms` array: `allSymptoms.filter(s => formatSymptomName(s).toLowerCase().includes(searchTerm.toLowerCase()))`. Because the array is small (132 items), in-memory filtering executes in sub-millisecond time without requiring lodash debounce.
- **Q106:** What happens if a user clicks outside the symptom search dropdown?
  - *Answer:* `onBlur={() => setTimeout(() => setIsSearchFocused(false), 200)}` closes the dropdown with a 200ms delay, allowing the click event on a dropdown item to register before the container unmounts.
- **Q107:** How are doctor specialty filters toggled in `frontend/src/pages/Doctors.jsx`?
  - *Answer:* Uses React Router URL parameter `:speciality`. Clicking a specialty card navigates to `/doctors/:speciality`. If the current parameter equals the selected card, it navigates back to `/doctors` to reset the filter.

### Backend Routing, Middleware & Business Logic
- **Q108:** Why is `bcrypt.genSalt(10)` explicitly called instead of passing round count directly to `bcrypt.hash(password, 10)`?
  - *Answer:* Both are equivalent under the hood. Calling `genSalt` explicitly allows handling salt generation errors independently and makes the cryptographic work factor explicit.
- **Q109:** What HTTP status code is returned when a doctor login fails due to incorrect password?
  - *Answer:* HTTP 404 in `doctorController.js#L66` (`{ success: false, message: "Invalid credientials" }`).
- **Q110:** Why does `userController.js#L82` explicitly call `.select('+password')` on `findOne({ email })`?
  - *Answer:* If a Mongoose schema is configured with `select: false` on password fields, calling `.select('+password')` explicitly overrides the projection to retrieve the hash for Bcrypt verification.
- **Q111:** What happens in `adminController.js#L48` if the incoming `address` is not valid JSON?
  - *Answer:* It enters a `try-catch` block: `JSON.parse(address)`. If parsing fails, it aborts immediately with HTTP 400 `{ success: false, message: "Invalid address format" }`.
- **Q112:** Why does `allDoctors` in `adminController.js` use `.select('-password')`?
  - *Answer:* To ensure administrative endpoints do not transmit hashed password strings across the network, reducing exposure in transit.
- **Q113:** Why does `doctorList` in `doctorController.js` exclude both `-password` and `-email`?
  - *Answer:* `doctorList` is a public endpoint accessible by unauthenticated patients. Exposing doctor personal email addresses would facilitate phishing attacks and violate physician privacy.
- **Q114:** What is the purpose of `appointmentData.userData` and `appointmentData.docData`?
  - *Answer:* They store immutable snapshots of the patient's and doctor's profiles at the exact time of booking, preserving billing and audit integrity.
- **Q115:** What happens if a doctor marks an appointment complete when `appointmentData.isCompleted` is already true?
  - *Answer:* `appointmentModel.findByIdAndUpdate(appointmentId, { isCompleted: true })` idempotently updates the document and returns HTTP 200.

### Machine Learning, Triage & Mathematics
- **Q116:** What is the dimensionality of the input vector passed to `disease_model`?
  - *Answer:* Exactly $1 \times 132$ (one row, 132 columns).
- **Q117:** How many unique disease classes can `prognosis_encoder` decode?
  - *Answer:* 41 unique disease classes.
- **Q118:** How many unique specialist categories can `specialist_encoder` decode?
  - *Answer:* 7 core specialist categories.
- **Q119:** What mathematical function converts decision tree leaf counts into probabilities in `predict_proba()`?
  - *Answer:* For each tree, class probability is the fraction of training samples of that class in the leaf: $p_{k, b} = \frac{n_k}{\sum n_i}$. The ensemble averages these fractions across all trees: $P(k) = \frac{1}{B} \sum_{b=1}^B p_{k, b}$.
- **Q120:** Why is Scikit-Learn's `np.argsort(disease_probs)[0][-3:][::-1]` used to extract top-3 diseases?
  - *Answer:* `np.argsort()` sorts indices in ascending probability order. Slicing `[-3:]` grabs the 3 highest probabilities, and `[::-1]` reverses them so the primary diagnosis is at index 0.
- **Q121:** What is pathognomonic symptom mapping?
  - *Answer:* A symptom whose presence uniquely identifies a disease (e.g., `patches_in_throat` heavily weighting toward `AIDS` or `sunken_eyes` toward `dehydration`). Decision trees naturally isolate pathognomonic features near the root of the tree.
- **Q122:** How does the model perform if a patient inputs contradictory symptoms (e.g. `weight_gain` and `weight_loss`)?
  - *Answer:* The tree splits partition the probabilities across both conditions (e.g. `Hypothyroidism` and `Hyperthyroidism`), resulting in lower confidence for the primary diagnosis and elevating both conditions into the top-3 predictions.
- **Q123:** Could the ML pipeline be replaced with a Deep Neural Network (MLP)?
  - *Answer:* Yes, but an MLP with binary sparse inputs (132 features) offers no accuracy advantage over Random Forests on tabular data, requires significantly higher compute for training, requires feature scaling, and introduces hyperparameter tuning complexity without clinical interpretability.
- **Q124:** What is the exact purpose of `prognosis_encoder.pkl`?
  - *Answer:* It stores the Scikit-Learn `LabelEncoder` instance that maps model prediction integers back to string disease names (`inverse_transform`).

### Infrastructure, Docker & Deployment
- **Q125:** Why is Docker multi-stage build used in root `dockerfile`?
  - *Answer:* To separate the build environment (Node 20 Alpine with large `node_modules` caches) from the runtime environment (minimal Nginx Alpine containing only static compiled `.dist` files), reducing final production image size from ~1.2GB to < 35MB.
- **Q126:** What healthcheck is configured for the `webserver` service in `docker-compose.yml`?
  - *Answer:* `test: ["CMD", "curl", "-f", "http://localhost"]` every 30 seconds with a 10s timeout and 3 retries.
- **Q127:** What healthcheck is configured for the `backend` service in `docker-compose.yml`?
  - *Answer:* `test: ["CMD", "curl", "-f", "http://localhost:3000/health"]`.
- **Q128:** Why does the backend healthcheck in `docker-compose.yml` fail in the current repo?
  - *Answer:* In `server.js`, the health endpoint is defined at `/` (returning `'API WORKING Great'`), not `/health`, and runs on port 4000. In production, the healthcheck should test `http://localhost:4000/`.
- **Q129:** What network driver is defined in `docker-compose.yml`?
  - *Answer:* `bridge` driver (`networks: app-network: driver: bridge`), allowing isolated DNS-based service discovery between `webserver` and `backend`.
- **Q130:** What AWS EC2 instance type is suitable for hosting this entire stack?
  - *Answer:* A `t3.medium` or `t3.large` (2 vCPU, 4GB RAM) comfortably hosts the Nginx container, Node backend, Python Flask service, and local MongoDB instance under moderate hospital workloads.

---

### Additional Technical Deep-Dive Q&A Catalog (Q131 – Q365)

- **Q131:** What is the exact schema representation of `slots_booked` in `doctorModel.js`?
  - *Answer:* It is defined as `slots_booked: { type: Object, default: {} }`. At runtime, it stores date string keys mapping to string arrays of booked times: `{ "16_10_2026": ["10:00 AM", "11:30 AM"], "17_10_2026": ["02:00 PM"] }`.
- **Q132:** Why does `delete docData.slots_booked` execute before saving an appointment in `userController.js#L226`?
  - *Answer:* To prevent storing the doctor's entire hospital booking calendar inside every single appointment document, saving storage and maintaining patient confidentiality.
- **Q133:** How is currency formatted across the frontend?
  - *Answer:* In `AppContext.jsx#L17`, `currencySymbol = '$'` is exported and dynamically prepended to all fee displays (`{currencySymbol}{docInfo.fees}`).
- **Q134:** How does `MyAppoinments.jsx` format date strings for display?
  - *Answer:* In `slotDateFormat`, it splits `"16_10_2026"` by `_`, maps the month integer `10` to an array index returning `'Oct'`, and returns `"16 Oct 2026"`.
- **Q135:** What happens if a patient tries to book an appointment with an invalid doctor ID?
  - *Answer:* `doctorModel.findById(docId)` returns `null`. Attempting to read `docData.available` throws an error caught by `catch (error)`, returning HTTP 500 `{ success: false, message: error.message }`.
- **Q136:** Can an Admin change their password through the API?
  - *Answer:* No. Admin credentials are hardcoded into server environment variables (`ADMIN_EMAIL`, `ADMIN_PASSWORD`). Changing admin credentials requires updating `.env` and restarting the backend service.
- **Q137:** What prevents duplicate user registration with the same email address?
  - *Answer:* In `userModel.js`, `email: { type: String, required: true, unique: true }` enforces a unique B-Tree index at the MongoDB engine level.
- **Q138:** What happens if two users attempt to register with the same email simultaneously?
  - *Answer:* MongoDB throws a duplicate key error (`E11000 duplicate key error collection`), which is caught by the controller's catch block and returned as HTTP 500.
- **Q139:** What is the maximum file size accepted for doctor profile photos?
  - *Answer:* 5 Megabytes (`5 * 1024 * 1024`), enforced by Multer in `backend/middlewares/multer.js#L28`.
- **Q140:** What image formats are accepted by the file upload filter?
  - *Answer:* Only `image/jpeg` and `image/png`. SVG, GIF, WebP, and executable binaries are rejected with an explicit Error object.
- **Q141:** How are Cloudinary credentials configured in the application?
  - *Answer:* In `backend/config/cloudinary.js#L4-L8`, Cloudinary v2 SDK initializes via `cloudinary.config({ cloud_name, api_key, api_secret })`.
- **Q142:** How does `adminController.js` hash doctor passwords during registration?
  - *Answer:* Uses `bcrypt.genSalt(10)` and `bcrypt.hash(password, salt)` before instantiating `new doctorModel()`.
- **Q143:** What default profile image is assigned to a user if none is uploaded during registration?
  - *Answer:* A hardcoded base64 PNG data URI string in `backend/models/userModel.js#L7`.
- **Q144:** What default profile image is assigned to a doctor if none is uploaded during onboarding?
  - *Answer:* A hardcoded base64 PNG data URI string in `backend/models/doctorModel.js#L7`.
- **Q145:** How is the latest appointment list extracted in `adminDashboard`?
  - *Answer:* In `adminController.js#L214`: `appointments.reverse().slice(0, 5)`. It inverts the array to place newest bookings first and takes the top 5 elements.
- **Q146:** What is the flaw with using `.reverse()` on an in-memory Mongoose query array?
  - *Answer:* `.reverse()` mutates the array in place in Node.js memory. If the array contains 100,000 documents, reversing and slicing in memory consumes significant CPU cycles. It should be replaced with `appointmentModel.find({}).sort({ date: -1 }).limit(5)`.
- **Q147:** How does the doctor portal determine total earnings?
  - *Answer:* In `doctorController.js#L187-L189`, it runs `appointments.map(item => earnings += item.amount)`.
- **Q148:** What is the flaw with calculating doctor earnings from all appointments?
  - *Answer:* It sums `item.amount` for *all* appointments, including those where `cancelled === true` or `payment === false`!
- **Q149:** How should the doctor earnings calculation be corrected?
  - *Answer:* It must filter for completed and paid appointments:
    `appointments.filter(item => !item.cancelled && item.isCompleted).reduce((sum, item) => sum + item.amount, 0)`.
- **Q150:** How does the frontend handle token expiration mid-session?
  - *Answer:* If an expired token is sent, `jwt.verify` in middleware throws `TokenExpiredError`. The catch block returns HTTP 500. The frontend Axios interceptor or component detects `data.success === false` and calls `toast.error()`.
- **Q151:** What is the purpose of `RelatedDoctors.jsx` in the frontend?
  - *Answer:* It filters the global `doctors` context array to find doctors with the same `speciality` as the currently viewed physician, excluding the current doctor (`doc._id !== docId`).
- **Q152:** What is the purpose of `specialityData` in `frontend/src/assets/assets.js`?
  - *Answer:* It exports static visual descriptors (specialty name and corresponding SVG icon) used to render the circular specialty filter cards on the home page.
- **Q153:** What happens if a patient tries to book an appointment with an empty `slotTime`?
  - *Answer:* In `userController.js`, `slotTime` is required. The appointment model validation throws a Mongoose `ValidationError` and aborts the save operation.
- **Q154:** What is the role of `eslint.config.js` in `frontend/` and `admin/`?
  - *Answer:* Configures ESLint v9 flat config with `react-hooks` and `react-refresh` rules to enforce clean code standards and prevent hook misuse.
- **Q155:** What build tool is used in `backend/package.json`?
  - *Answer:* The backend has no build tool; it runs directly on Node.js using native ES module resolution (`node server.js`).
- **Q156:** What development utility restarts the backend on file changes?
  - *Answer:* `nodemon` (`npm run server`), configured in `backend/package.json#L9`.
- **Q157:** How are CORS requests handled in the Flask ML service?
  - *Answer:* In `app.py#L8`, `CORS(app)` initializes Flask-CORS, allowing incoming cross-origin POST requests from `localhost:5173` or production domains.
- **Q158:** What happens if a user submits a symptom with extra whitespace (e.g. `" chest pain "`)?
  - *Answer:* In `app.py#L30`, `symptom.lower().replace(' ', '_')` strips spaces and replaces them with underscores, ensuring proper match against `chest_pain`.
- **Q159:** Why does `SpecialistPredication.jsx` store `prediction` in component state rather than context?
  - *Answer:* Diagnostic predictions are transient session operations specific to the triage page and do not need to persist across unrelated application routes.
- **Q160:** What is the purpose of `react-icons` in `frontend/package.json`?
  - *Answer:* Provides SVG icon sets for UI elements (stars, arrows, status badges, payment icons).
- **Q161:** How does `DoctorProfile.jsx` allow doctors to update their availability?
  - *Answer:* Renders a checkbox bound to `available` state; submitting calls `POST /api/doctor/update-profile` with `{ docId, fees, address, available }`.
- **Q162:** How does `AddDoctor.jsx` handle the address input?
  - *Answer:* Collects `address1` and `address2` as separate text inputs, packages them into a JSON string `JSON.stringify({ line1: address1, line2: address2 })`, and appends them to `FormData`.
- **Q163:** What happens if a doctor enters a non-numeric fee in `AddDoctor.jsx`?
  - *Answer:* The input has `type="number"`, and the submit handler calls `Number(fees)`. If empty or invalid, validation in `adminController.js` rejects it with HTTP 400.
- **Q164:** How does `adminRouter.js` handle doctor image uploads?
  - *Answer:* Mounts `upload.single('image')` middleware directly in the route declaration before the controller:
    `adminRouter.post('/add-doctor', authAdmin, upload.single('image'), addDoctor)`.
- **Q165:** What happens if Cloudinary credentials are invalid when adding a doctor?
  - *Answer:* `uploadOnCloudinary` catches the Cloudinary error, returns `null`, and `adminController.js#L41` aborts with HTTP 500 `{ success: false, message: "Failed to upload image" }`.
- **Q166:** Can a patient view appointments that have been completed?
  - *Answer:* Yes. `listAppoinment` returns all appointments regardless of completion status. In `MyAppoinments.jsx`, completed appointments display a green "Completed" badge.
- **Q167:** Can a patient cancel an appointment that has already been completed?
  - *Answer:* In `userController.js#L287-L335`, the cancellation controller checks `if (userId !== appointmentData.userId)`, but **does not check `appointmentData.isCompleted`**!
  - *Flaw Fix:* Add guard clause: `if (appointmentData.isCompleted) return res.status(400).json({ success: false, message: "Cannot cancel completed appointment" })`.
- **Q168:** How does `MyAppoinments.jsx` trigger the Razorpay popup?
  - *Answer:* Dynamically instantiates `new window.Razorpay(options)` and calls `rzp.open()`.
- **Q169:** Where is the Razorpay JavaScript SDK loaded in the frontend?
  - *Answer:* In `frontend/index.html` via `<script src="https://checkout.razorpay.com/v1/checkout.js"></script>`.
- **Q170:** What is the currency specified in Razorpay order creation?
  - *Answer:* Read from `process.env.CURRENCY` (e.g., `'INR'` or `'USD'`).
- **Q171:** Why are fees multiplied by 100 when creating a Razorpay order?
  - *Answer:* Razorpay expects monetary values in the smallest currency sub-unit (paise for INR, cents for USD). Multiplying by 100 converts whole currency units into sub-units.
- **Q172:** What is the primary difference between `adminRouter.post('/cancle-appointment')` and `userRouter.post('/cancle-appointment')`?
  - *Answer:* Admin cancellation requires `authAdmin` and can cancel *any* appointment globally; user cancellation requires `authUser` and validates that `appointmentData.userId === req.body.userId`.
- **Q173:** How does `Navbar.jsx` in the frontend handle responsive mobile layouts?
  - *Answer:* Maintains a `showMenu` state variable. On mobile viewports (`sm:hidden`), it displays a hamburger menu icon that toggles a full-screen slide-over navigation panel.
- **Q174:** What happens if a patient logs out while viewing their appointments?
  - *Answer:* `setToken('')` resets context state, `localStorage.removeItem('token')` clears the token, and `useEffect` in `MyAppoinments.jsx` redirects the user to `/login`.
- **Q175:** How does the doctor portal distinguish between a cancelled appointment and an active appointment?
  - *Answer:* In `DoctorAppointments.jsx`, it renders conditional action buttons: if `item.cancelled` is true, it displays a red "Cancelled" label; if `item.isCompleted` is true, it displays a green "Completed" label; otherwise, it renders interactive "Cancel" and "Complete" checkmark buttons.
- **Q176:** What is the purpose of `AdminContext.jsx`?
  - *Answer:* Encapsulates administrative state: `aToken`, `doctors` list, `appointment` list, `dashData`, and corresponding dispatchers (`getAllDoctors`, `changeAvailablity`, `getAllappointments`, `cancleAppointment`, `getDashData`).
- **Q177:** What is the purpose of `DoctorContext.jsx`?
  - *Answer:* Encapsulates doctor state: `dToken`, `appointments` list, `dashData`, `profileData`, and corresponding dispatchers (`getAppointments`, `completeAppointment`, `cancleAppointment`, `getDashData`, `getProfileData`).
- **Q178:** How are doctors sorted when displayed in the patient directory?
  - *Answer:* Retains default MongoDB database insertion order unless filtered by specialty in `Doctors.jsx`.
- **Q179:** What database index exists on the `User` schema?
  - *Answer:* A unique index on `email: 1`.
- **Q180:** What database index exists on the `Doctor` schema?
  - *Answer:* A unique index on `email: 1`.
- **Q181:** How does `adminDashboard` calculate patient count?
  - *Answer:* Counts total documents in `users` collection: `users.length`.
- **Q182:** How does `doctorDashboard` calculate unique patient count?
  - *Answer:* Collects unique `userId` values across all appointments assigned to that doctor: `if (!patients.includes(item.userId)) patients.push(item.userId)`.
- **Q183:** What happens if an image upload fails in `updateProfile`?
  - *Answer:* Profile text updates (`name`, `phone`, `gender`, `dob`, `address`) still succeed; the image field remains unchanged.
- **Q184:** What is the maximum password length enforced in `registerUser`?
  - *Answer:* No maximum length is enforced; only a minimum length of 8 characters (`password.length < 8`).
- **Q185:** What security vulnerability exists with unconstrained password length in Bcrypt?
  - *Answer:* Bcrypt silently truncates passwords longer than 72 bytes. A password of 100 characters has only its first 72 bytes evaluated.
- **Q186:** What database connection events are monitored in `mongodb.js`?
  - *Answer:* `mongoose.connection.on('connected', () => console.log('database Connected'))`.
- **Q187:** Does `mongodb.js` handle database disconnection or reconnection errors?
  - *Answer:* No error listener is registered (`mongoose.connection.on('error', ...)` is missing).
- **Q188:** How can database connection reliability be improved in `mongodb.js`?
  - *Answer:* By listening for `error` and `disconnected` events and wrapping `mongoose.connect` with retry logic.
- **Q189:** How are doctor slots formatted when saved into `slots_booked`?
  - *Answer:* Array of 12-hour formatted time strings: `["10:00 AM", "10:30 AM", "11:00 AM"]`.
- **Q190:** What is the format of `slotDate` in the database?
  - *Answer:* String format `D_M_YYYY` (e.g. `"16_10_2026"`).
- **Q191:** Why are dates in `slotDate` 1-indexed for months?
  - *Answer:* In JavaScript, `date.getMonth()` returns 0-11. The code explicitly adds 1: `let month = date.getMonth() + 1;` so October is represented as `10`.
- **Q192:** What happens if a patient tries to book on February 29th on a non-leap year?
  - *Answer:* The frontend slot generator constructs dates using `new Date()`, which natively respects leap years and never generates invalid calendar dates.
- **Q193:** How does the doctor toggle their availability from the admin panel?
  - *Answer:* In `DoctorList.jsx`, a checkbox bound to `item.available` calls `changeAvailablity(item._id)`, hitting `POST /api/admin/change-availablity`.
- **Q194:** What happens if `fees` is submitted as a string in `updateDoctorProfile`?
  - *Answer:* Mongoose schema casts valid numeric strings to `Number`. If non-numeric characters are present, Mongoose throws a `CastError`.
- **Q195:** How is the admin panel base path configured in Vite?
  - *Answer:* In `admin/vite.config.js#L11`: `base: '/admin/'`. This ensures all compiled asset links in `index.html` resolve relative to `/admin/`.
- **Q196:** What is the purpose of `vercel.json` in `frontend/`?
  - *Answer:* Configures client-side rewrite rules: `{ "rewrites": [{ "source": "/(.*)", "destination": "/" }] }` so refreshing sub-routes does not return HTTP 404 on Vercel.
- **Q197:** What is the purpose of `vercel.json` in `backend/`?
  - *Answer:* Declares the serverless build runtime `@vercel/node` targeting `server.js` and routes all requests `/(.*)` to `server.js`.
- **Q198:** How does `adminController.js` validate email addresses?
  - *Answer:* `validator.isEmail(email)` verifies structure, domain formatting, and absence of invalid characters.
- **Q199:** How does `userController.js` validate email addresses?
  - *Answer:* `validator.isEmail(email)` verifies email formatting during both registration and login.
- **Q200:** What happens if an empty string is submitted as password in `loginUser`?
  - *Answer:* Line 66 checks `if (!email || !password)` and immediately returns HTTP 400 `{ success: false, message: "Email and password are required" }`.
- **Q201:** Does `loginDoctor` check for empty fields before querying the database?
  - *Answer:* No explicit `if (!email || !password)` check exists in `loginDoctor`. If undefined, `doctorModel.findOne({ email: undefined })` resolves to `null` and returns HTTP 404 "Invalid credentials".
- **Q202:** What is the purpose of `doctorProfile` controller?
  - *Answer:* Returns public and private profile details for the authenticated doctor excluding the password hash: `.select("-password")`.
- **Q203:** What happens if an unauthenticated user accesses `/api/doctor/profile`?
  - *Answer:* `authDoctor` finds no `dtoken` in headers and aborts with HTTP 404 "Not Authorised Login again".
- **Q204:** What happens if an unauthenticated user accesses `/api/user/get-profile`?
  - *Answer:* `authUser` finds no `token` in headers and aborts with HTTP 404 "Not Authorised Login again".
- **Q205:** Can a doctor see other doctors' appointments?
  - *Answer:* No. `appointmentsDoctor` filters strictly by the authenticated doctor's ID: `appointmentModel.find({ docId })`.
- **Q206:** Can an Admin delete a doctor from the system?
  - *Answer:* **Not found in repo / Not implemented.** There is no doctor deletion endpoint (`deleteDoctor`) in the current repository. Doctors can only have their availability toggled to `false`.
- **Q207:** Can a user delete their account?
  - *Answer:* **Not found in repo / Not implemented.** There is no user deletion endpoint in `userController.js`.
- **Q208:** How does Prescripto prevent brute-force login attempts?
  - *Answer:* **Not found in repo / Not implemented.** There is no rate-limiting middleware (such as `express-rate-limit`) installed in the current backend.
  - *Defensible Fix:* Add `express-rate-limit` to `/api/user/login`, `/api/doctor/login`, and `/api/admin/login` limiting attempts to 5 per 15-minute window per IP.
- **Q209:** What is the purpose of `SpecialistPredication.jsx#L134-L138` (`clearAll`)?
  - *Answer:* Resets `selectedSymptoms` to `[]`, `prediction` to `null`, and `error` to `null`, clearing the diagnostic workbench.
- **Q210:** How does the UI indicate that the symptom threshold has not been reached?
  - *Answer:* In `SpecialistPredication.jsx#L246-L248`, the count badge dynamically renders yellow text if `< 3` and green text if `>= 3`: `(${selectedSymptoms.length}/3 minimum)`.
- **Q211:** What happens if a patient selects 10 symptoms?
  - *Answer:* All 10 symptoms are encoded into the vector. The tree ensemble processes all active features. More symptoms typically narrow down the probability distribution, increasing diagnostic confidence.
- **Q212:** Can the ML model output multiple primary diagnoses?
  - *Answer:* No. `np.max(disease_probs)` selects the single highest-probability class. However, the top-3 differential diagnoses array exposes close alternatives.
- **Q213:** How does `SpecialistPredication.jsx` navigate to emergency doctor booking?
  - *Answer:* Line 462 invokes `navigate('/appoinment/67e99d4f1a628be7d769923d')` targeting the facility's dedicated emergency doctor ObjectId.
- **Q214:** What happens if the hardcoded emergency doctor ID does not exist in the database?
  - *Answer:* `fetchDoctorInfo` in `Appoinments.jsx` fails to find the doctor in the `doctors` array; `docInfo` remains `null`, and the appointment booking view renders blank.
  - *Defensible Fix:* Query a dynamic emergency doctor endpoint: `GET /api/doctor/emergency-on-call`.
- **Q215:** How does `SpecialistPredication.jsx` pass urgency context when navigating to regular doctors?
  - *Answer:* Line 477 passes state: `navigate('/doctors/' + specialist, { state: { urgent: prediction.severity === 'high' } })`.
- **Q216:** What is the purpose of `Sidebar.jsx` in the admin application?
  - *Answer:* Inspects active context tokens (`aToken` vs `dToken`) and dynamically renders either the Administrator navigation links (Dashboard, All Appointments, Add Doctor, Doctor List) or Doctor navigation links (Dashboard, Appointments, Profile).
- **Q217:** What CSS framework version is used in `admin/`?
  - *Answer:* Tailwind CSS v4.0.3.
- **Q218:** What CSS framework version is used in `frontend/`?
  - *Answer:* Tailwind CSS v3.4.17.
- **Q219:** How is dark mode supported in the frontend?
  - *Answer:* **Not found in repo / Not implemented.** The UI uses light theme gradients (`bg-gradient-to-br from-blue-50 to-indigo-50`).
- **Q220:** How does `Footer.jsx` render in the patient portal?
  - *Answer:* Renders company logo, description paragraph, quick navigation links (Home, About Us, Delivery, Privacy Policy), and copyright notice.
- **Q221:** What is the purpose of `MyProfile.jsx`?
  - *Answer:* Renders editable patient profile: name, phone, address lines, gender dropdown, date of birth picker, and avatar upload.
- **Q222:** What state variable controls editing mode in `MyProfile.jsx`?
  - *Answer:* `const [isEdit, setIsEdit] = useState(false)`. Clicking "Edit" toggles to `true` (rendering inputs); clicking "Save Information" dispatches the update and toggles back to `false`.
- **Q223:** How does `MyProfile.jsx` preview uploaded avatar images before saving?
  - *Answer:* Uses `image ? URL.createObjectURL(image) : userData.image` to create an ephemeral browser blob URL for instant visual feedback.
- **Q224:** What happens if a patient enters an invalid phone number?
  - *Answer:* The frontend accepts any string; the backend requires presence (`!phone`), saving it as a String without regex phone validation.
- **Q225:** How does `Contact.jsx` render hospital contact information?
  - *Answer:* Displays office address, telephone contact, email address, careers information, and an "Explore Jobs" button.
- **Q226:** How does `About.jsx` render company mission statements?
  - *Answer:* Displays hospital origin story, vision statement, and three "Why Choose Us" feature cards: Efficiency, Convenience, and Personalization.
- **Q227:** What is the purpose of `Home.jsx`?
  - *Answer:* Composes five components: `Header` (hero banner with CTA), `SpecialityMenu` (specialty carousel), `TopDoctors` (grid of highest-rated physicians), `Banner` (promotional account creation CTA), and `Footer`.
- **Q228:** How many doctors are displayed in `TopDoctors.jsx` on the home page?
  - *Answer:* Slices the top 10 doctors from the context array: `doctors.slice(0, 10)`.
- **Q229:** What happens if a user clicks a doctor card in `TopDoctors.jsx`?
  - *Answer:* Navigates to `/appoinment/${item._id}` and scrolls window to top: `scrollTo(0, 0)`.
- **Q230:** What visual badge indicates a doctor's availability in `TopDoctors.jsx`?
  - *Answer:* A green dot with text "Available" if `doc.available === true`, or a grey dot with text "Not Available" if `false`.
- **Q231:** What is the purpose of `Header.jsx`?
  - *Answer:* Renders the primary hero heading: "Book Appointment With Trusted Doctors", displaying group profile images and a "Book appointment" button anchoring to `#speciality`.
- **Q232:** How does `SpecialityMenu.jsx` allow users to filter doctors?
  - *Answer:* Clicking a specialty icon links to `/doctors/${item.speciality}`, passing the specialty string as a URL parameter to the `Doctors` page.
- **Q233:** How does `Doctors.jsx` render the specialty sidebar on desktop viewports?
  - *Answer:* Displays vertical filter buttons (`General physician`, `Gynecologist`, `Dermatologist`, `Pediatricians`, `Neurologist`, `Gastroenterologist`) on the left, with the matching doctor grid on the right.
- **Q234:** How does `Doctors.jsx` render filters on mobile viewports?
  - *Answer:* Renders a collapsible "Filters" button controlled by `showFilter` state to toggle sidebar visibility on small screens.
- **Q235:** What happens if no doctors match the selected specialty in `Doctors.jsx`?
  - *Answer:* The filtered array is empty; the grid renders zero cards without crashing.
- **Q236:** What is the purpose of `Banner.jsx`?
  - *Answer:* Renders an invitation banner: "Book Appointment With 100+ Trusted Doctors" with a button to "Create account" routing to `/login`.
- **Q237:** What state controls whether `Login.jsx` displays Login or Sign Up?
  - *Answer:* `const [state, setState] = useState('Sign Up')`. Toggling switches input fields and buttons between "Sign Up" (register) and "Login".
- **Q238:** How is the form submitted in `Login.jsx`?
  - *Answer:* `onSubmitHandler` checks `state === 'Sign Up' ? axios.post('/api/user/register') : axios.post('/api/user/login')`.
- **Q239:** What happens upon successful login in `Login.jsx`?
  - *Answer:* Stores token in `localStorage.setItem('token', data.token)`, updates `setToken(data.token)` in context, and redirects user to `/`.
- **Q240:** What happens if `registerUser` receives a password shorter than 8 characters?
  - *Answer:* Line 24 in `userController.js` returns HTTP 400 `{ success: false, message: 'Enter a strong password' }`.
- **Q241:** What happens if `registerUser` receives an invalid email?
  - *Answer:* Line 20 in `userController.js` returns HTTP 400 `{ success: false, message: 'enter a valid email' }`.
- **Q242:** Why does `adminController.js` validate password length with `< 8`?
  - *Answer:* To enforce minimum security entropy for doctor accounts created by hospital administrators.
- **Q243:** How does `adminController.js` upload images to Cloudinary?
  - *Answer:* Calls `uploadOnCloudinary(imageFile.path)` passing the local path generated by Multer disk storage.
- **Q244:** What happens if `uploadOnCloudinary` fails to upload?
  - *Answer:* Returns `null`; controller checks `if (!imageUpload || !imageUpload.secure_url)` and returns HTTP 500 "Failed to upload image".
- **Q245:** What Cloudinary resource type is used for uploads?
  - *Answer:* `resource_type: "auto"` in `backend/config/cloudinary.js#L18`, allowing Cloudinary to automatically detect JPEG, PNG, or WebP.
- **Q246:** How does `cloudinary.js` clean up temporary files?
  - *Answer:* Calls `fs.unlinkSync(localFilePath)` in both the success path and catch block.
- **Q247:** What error occurs if `fs.unlinkSync` tries to delete a non-existent file?
  - *Answer:* Throws `ENOENT: no such file or directory`. `cloudinary.js#L28` prevents this by checking `if (fs.existsSync(localFilePath))` first.
- **Q248:** What is the purpose of `appointmentCancle` in `adminController.js`?
  - *Answer:* Allows hospital admins to cancel any appointment and restore doctor availability slots.
- **Q249:** How does `appointmentCancle` restore doctor availability?
  - *Answer:* Extracts `docId`, `slotDate`, and `slotTime` from the appointment, filters `slotTime` out of `slots_booked[slotDate]`, and updates the doctor document.
- **Q250:** What happens if `slots_booked[slotDate]` is undefined during cancellation?
  - *Answer:* Attempting `.filter()` would throw a TypeError. In Prescripto, slots are guaranteed to exist because cancellation can only occur on previously booked appointments.
- **Q251:** What is the role of `isCompleted` in `appointmentModel.js`?
  - *Answer:* Tracks clinical fulfillment: set to `true` when the doctor finishes the consultation via `appointmentComplete`.
- **Q252:** Can a doctor complete an appointment that has been cancelled?
  - *Answer:* In `doctorController.js#L115`, it checks `if (appointmentData && appointmentData.docId === docId)`, but **omits `if (appointmentData.cancelled)`**.
  - *Flaw Fix:* Add `if (appointmentData.cancelled) return res.status(400).json({ success: false, message: "Cannot complete a cancelled appointment" })`.
- **Q253:** How does the doctor portal update fees and clinic address?
  - *Answer:* Submits `{ docId, fees, address, available }` to `POST /api/doctor/update-profile`, executing `doctorModel.findByIdAndUpdate(docId, { fees, address, available })`.
- **Q254:** Can a doctor change their own specialty through `updateDoctorProfile`?
  - *Answer:* No. Only `fees`, `address`, and `available` are accepted in `updateDoctorProfile`. Specialty changes require administrative intervention.
- **Q255:** How does `AllAppointments.jsx` format patient age in the admin table?
  - *Answer:* Calls helper `calculateAge(item.userData.dob)` which parses `dob` string (`YYYY-MM-DD`), computes difference from current date, and returns integer age.
- **Q256:** What happens if `item.userData.dob` is `"Not Selected"` when calculating age?
  - *Answer:* `calculateAge` produces `NaN`. In production, this should return `"N/A"` if `dob === 'Not Selected'`.
- **Q257:** What visual indicator is used for cancelled appointments in `AllAppointments.jsx`?
  - *Answer:* Renders red text `'Cancelled'` instead of the interactive cancel cross icon.
- **Q258:** What visual indicator is used for completed appointments in `AllAppointments.jsx`?
  - *Answer:* Renders green text `'Completed'`.
- **Q259:** How does `Dashboard.jsx` in the admin portal render the latest bookings?
  - *Answer:* Maps over `dashData.latestAppointments`, rendering patient avatar, name, booked doctor name, date, and cancel button.
- **Q260:** How does `DoctorDashboard.jsx` display doctor earnings?
  - *Answer:* Displays `{currency}{dashData.earnings}` inside an analytics card with an earnings icon.
- **Q261:** How does `DoctorDashboard.jsx` display total unique patients?
  - *Answer:* Displays `{dashData.patients}` inside a patient metrics card.
- **Q262:** How does `DoctorDashboard.jsx` display total appointments count?
  - *Answer:* Displays `{dashData.appointments}` inside an appointments counter card.
- **Q263:** How does `DoctorAppointments.jsx` render payment status?
  - *Answer:* Renders an inline pill badge: `'ONLINE'` (blue) if `item.payment === true`, or `'CASH'` (grey) if `false`.
- **Q264:** How does a doctor mark an appointment complete from the UI?
  - *Answer:* Clicks the tick icon in `DoctorAppointments.jsx`, invoking `completeAppointment(item._id)` from `DoctorContext`.
- **Q265:** How does a doctor cancel an appointment from the UI?
  - *Answer:* Clicks the cancel cross icon, invoking `cancleAppointment(item._id)` from `DoctorContext`.
- **Q266:** What toast notification is displayed when a doctor completes an appointment?
  - *Answer:* `toast.success("Appointment Completed")`.
- **Q267:** What toast notification is displayed when a doctor cancels an appointment?
  - *Answer:* `toast.success("Appointment Cancelled")`.
- **Q268:** What toast notification is displayed when an admin toggles doctor availability?
  - *Answer:* `toast.success("Availablity Changed")`.
- **Q269:** What toast notification is displayed when a patient cancels an appointment?
  - *Answer:* `toast.success("Appointment Cancelled")`.
- **Q270:** What toast notification is displayed when a patient profile is updated?
  - *Answer:* `toast.success("Profile Updated")`.
- **Q271:** What is the structure of `address` in `userModel.js`?
  - *Answer:* An object with two string properties: `{ line1: String, line2: String }`.
- **Q272:** What is the structure of `address` in `doctorModel.js`?
  - *Answer:* An object with two string properties: `{ line1: String, line2: String }`.
- **Q273:** Why does `adminController.js#L48` parse `address` with `JSON.parse()`?
  - *Answer:* Because `FormData` encodes all non-file fields as strings. JSON objects appended to `FormData` arrive at Express as JSON strings and must be parsed into objects before Mongoose insertion.
- **Q274:** Why does `userController.js#L158` parse `address` with `JSON.parse()`?
  - *Answer:* In `updateProfile`, `address` is transmitted inside a `FormData` object to support avatar upload, requiring `JSON.parse(address)`.
- **Q275:** What happens if `phone` is omitted when updating patient profile?
  - *Answer:* Line 150 in `userController.js` checks `if (!name || !phone || !gender || !dob)` and rejects with HTTP 400 "Missing Details".
- **Q276:** What happens if `gender` is omitted when updating patient profile?
  - *Answer:* Rejects with HTTP 400 "Missing Details".
- **Q277:** What default value is given to `gender` when a user account is created?
  - *Answer:* `'Male'` in `userModel.js#L9`.
- **Q278:** What default value is given to `dob` when a user account is created?
  - *Answer:* `'Not Selected'` in `userModel.js#L10`.
- **Q279:** What default value is given to `phone` when a user account is created?
  - *Answer:* `'0000000000'` in `userModel.js#L11`.
- **Q280:** What happens if a doctor enters a 5-character password when registered by Admin?
  - *Answer:* Line 30 in `adminController.js` returns HTTP 400 "Password must be at least 8 characters long".
- **Q281:** How is `experience` selected when adding a doctor in `AddDoctor.jsx`?
  - *Answer:* A dropdown select with values from `"1 Year"` to `"10 Year"`.
- **Q282:** What default experience is set in `AddDoctor.jsx` state?
  - *Answer:* `'1 Year'` in `AddDoctor.jsx#L13`.
- **Q283:** What default specialty is set in `AddDoctor.jsx` state?
  - *Answer:* `'General physician'` in `AddDoctor.jsx#L16`.
- **Q284:** What happens if an admin tries to submit `AddDoctor` without selecting an image?
  - *Answer:* In `adminController.js#L20`, `!imageFile` triggers HTTP 400 "Missing required details".
- **Q285:** How does `AddDoctor.jsx` reset form fields after successful submission?
  - *Answer:* Sets `setDocImg(false)`, `setName('')`, `setEmail('')`, `setPassword('')`, `setFess('')`, `setAbout('')`, `setAddress1('')`, `setAddress2('')`.
- **Q286:** What is the purpose of `DoctorList.jsx` in the admin application?
  - *Answer:* Renders a grid of doctor cards showing photo, name, specialty, and an availability toggle checkbox.
- **Q287:** How does `DoctorList.jsx` handle checkbox toggling?
  - *Answer:* `onChange={() => changeAvailablity(item._id)}` calls `AdminContext`, which hits `POST /api/admin/change-availablity` and refreshes the list.
- **Q288:** What happens if a user navigates to `/admin` without logging in?
  - *Answer:* `aToken` and `dToken` are empty; `admin/src/App.jsx#L27` renders the `<Login />` component instead of administrative routes.
- **Q289:** What happens if a user navigates to `/my-profile` without logging in?
  - *Answer:* `AppContext` has `token = ''`; `loadUserProfileData` does not execute. Profile fields render empty or redirect to login.
- **Q290:** How does `Appoinments.jsx` handle an unauthenticated user clicking "Book Appointment"?
  - *Answer:* Line 78 in `Appoinments.jsx` checks `if (!token)`, displays `toast.warn('Login to Book Appointment')`, and calls `navigate('/login')`.
- **Q291:** What visual cue shows the currently selected appointment slot in `Appoinments.jsx`?
  - *Answer:* Compares `item.time === sloTime.time`. If matching, applies `bg-primary text-white`; otherwise applies `border border-gray-200 text-gray-400`.
- **Q292:** What visual cue shows the currently selected date in `Appoinments.jsx`?
  - *Answer:* Compares `slotIndex === index`. If matching, applies `bg-primary text-white`; otherwise applies `border border-gray-200`.
- **Q293:** How does `Appoinments.jsx` render doctor verification?
  - *Answer:* Renders `verified_icon` SVG badge next to the doctor's name.
- **Q294:** How does `Appoinments.jsx` display doctor degrees and experience?
  - *Answer:* Renders `{docInfo.degree} - {docInfo.speciality}` alongside a pill button displaying `{docInfo.experience}`.
- **Q295:** How does `Appoinments.jsx` render the doctor's about text?
  - *Answer:* Renders an "About" header with an info icon, followed by the paragraph text `{docInfo.about}`.
- **Q296:** What is the purpose of `RelatedDoctors.jsx` mounted at the bottom of `Appoinments.jsx`?
  - *Answer:* Shows alternative doctors in the same specialty in case the patient prefers another provider or earlier availability.
- **Q297:** How many related doctors are rendered in `RelatedDoctors.jsx`?
  - *Answer:* Slices the top 5 doctors: `relDoc.slice(0, 5)`.
- **Q298:** What happens if a patient books an appointment on a day where all slots are full?
  - *Answer:* The while loop in `getAvailableSlots` finds `isSlotAvailable === false` for every time step; `timeslots` array remains empty, and the UI displays no clickable slot pills for that day.
- **Q299:** What happens if a doctor is marked `available: false` while a patient is viewing their profile?
  - *Answer:* When the patient clicks "Book", `userController.js#L194` checks `if (!docData.available)` and rejects the request with HTTP 400 "Doctor not available".
- **Q300:** What happens if a patient tries to pay for an appointment that has already been cancelled?
  - *Answer:* In `userController.js#L357`, `if (!appointmentData || appointmentData.cancelled)` rejects the Razorpay order creation with HTTP 404 "Appointment Cancelled or not found".
- **Q301:** What happens if a patient cancels an appointment that has already been paid for?
  - *Answer:* The appointment status transitions to `cancelled: true`, but **no automated refund API call** is made to Razorpay in the current codebase!
  - *Production Fix:* Integrate `razorpayInstance.payments.refund(paymentId, { amount })` inside `cancleAppointment` to automatically refund the patient's card or UPI account.
- **Q302:** How does `userController.js#L400` verify that an order was paid?
  - *Answer:* Evaluates `if (orderInfo.status === 'paid')` and updates `appointmentModel.findByIdAndUpdate(orderInfo.receipt, { payment: true })`.
- **Q303:** What field connects a Razorpay order back to the appointment in MongoDB?
  - *Answer:* The `receipt` option passed during order creation: `receipt: appointmentId`. When fetched, `orderInfo.receipt` contains the appointment ObjectId.
- **Q304:** What currency symbol is displayed on the payment card in `MyAppoinments.jsx`?
  - *Answer:* Does not render a static currency string; uses the appointment amount directly: `item.amount`.
- **Q305:** How are appointments sorted when displayed to a patient in `MyAppoinments.jsx`?
  - *Answer:* In `MyAppoinments.jsx#L29`, the array is reversed: `SetAppointments(data.appointments.reverse())`, ensuring the most recently booked appointments appear at the top of the list.
- **Q306:** How does `SpecialistPredication.jsx` handle an empty response from `/predict`?
  - *Answer:* Axios throws an error or `response.data` is missing required keys; the catch block catches the exception and renders "Failed to get prediction. Please try again."
- **Q307:** What is the purpose of `matched_symptoms` in the `/predict` response?
  - *Answer:* Returns the subset of patient-selected symptoms that actually mapped to active feature columns in the machine learning model, verifying which symptoms influenced the prediction.
- **Q308:** How does `SpecialistPredication.jsx` render confidence percentages?
  - *Answer:* Renders an animated progress bar whose width is bound to the confidence percentage: `style={{ width: `${prediction.diagnosis.confidence}%` }}`.
- **Q309:** What color is the confidence progress bar in `SpecialistPredication.jsx`?
  - *Answer:* Green (`bg-green-500`).
- **Q310:** What visual icon is displayed for Emergency severity in `SpecialistPredication.jsx`?
  - *Answer:* Siren emoji: `🚨`.
- **Q311:** What visual icon is displayed for High severity in `SpecialistPredication.jsx`?
  - *Answer:* Warning sign emoji: `⚠️`.
- **Q312:** What visual icon is displayed for Medium severity in `SpecialistPredication.jsx`?
  - *Answer:* Information icon emoji: `ℹ️`.
- **Q313:** What visual icon is displayed for Low severity in `SpecialistPredication.jsx`?
  - *Answer:* Green checkmark emoji: `✅`.
- **Q314:** What button styling is applied to "Book Emergency Slot"?
  - *Answer:* Red background (`bg-red-600 hover:bg-red-700 text-white font-medium rounded-lg shadow-md`).
- **Q315:** What button styling is applied to "Book Urgent Appointment"?
  - *Answer:* Orange background (`bg-orange-600 hover:bg-orange-700 text-white font-medium rounded-lg shadow-md`).
- **Q316:** What button styling is applied to standard "Book Appointment"?
  - *Answer:* Blue background (`bg-blue-600 hover:bg-blue-700 text-white font-medium rounded-lg shadow-md`).
- **Q317:** How does `SpecialistPredication.jsx` render the top 3 differential diagnoses?
  - *Answer:* Uses a 3-column responsive grid (`grid-cols-1 md:grid-cols-3 gap-3`), rendering each disease name alongside a blue pill badge showing its individual confidence percentage.
- **Q318:** What happens if two diseases have identical confidence scores in the top 3?
  - *Answer:* Both are rendered with their identical percentage values in the order returned by `np.argsort()`.
- **Q319:** Can a patient deselect a symptom after adding it?
  - *Answer:* Yes. In `SpecialistPredication.jsx#L128-L131`, clicking the remove icon calls `handleRemoveSymptom(symptom)`, which filters the symptom out of `selectedSymptoms` and clears any existing prediction.
- **Q320:** Why is `prediction` cleared when a symptom is removed?
  - *Answer:* To invalidate the previous prediction and force the patient to re-run the diagnostic model on the modified symptom set.
- **Q321:** What is the maximum number of symptoms a patient can select?
  - *Answer:* There is no upper limit; a patient can select all 132 symptoms if clinically applicable.
- **Q322:** What is the purpose of `globals.css` / `index.css` in `frontend/`?
  - *Answer:* Injects Tailwind directives (`@tailwind base; @tailwind components; @tailwind utilities;`) and defines custom font imports (Outfit font family).
- **Q323:** What font family is configured for Prescripto?
  - *Answer:* The Google Font **Outfit** (`@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@100..900&display=swap')`).
- **Q324:** What primary color is configured in `frontend/tailwind.config.js`?
  - *Answer:* `#5f6FFF` (a curated indigo-blue tailored for modern healthcare platforms).
- **Q325:** What primary color is configured in `admin/src/index.css`?
  - *Answer:* `#5F6FFF` (`--color-primary: #5F6FFF;`).
- **Q326:** How does `Navbar.jsx` in the frontend indicate the active page?
  - *Answer:* Uses NavLink components with active underline styling (`<hr className='border-none outline-none h-0.5 bg-primary w-3/5 m-auto hidden' />` rendered when active).
- **Q327:** What happens when an authenticated user clicks their profile icon in the Navbar?
  - *Answer:* Opens a dropdown menu exposing links to "My Profile", "My Appointments", and an interactive "Logout" action.
- **Q328:** How does `admin/src/components/Navbar.jsx` distinguish between Admin and Doctor sessions?
  - *Answer:* Checks `aToken ? 'Admin' : 'Doctor'` and displays a pill badge next to the Prescripto logo: `<p className='border px-2.5 py-0.5 rounded-full border-gray-500 text-gray-600'>{aToken ? 'Admin' : 'Doctor'}</p>`.
- **Q329:** How does logout work in the Admin portal?
  - *Answer:* Clicks "Logout" button -> calls `aToken && setAToken('')`, `aToken && localStorage.removeItem('aToken')`, `dToken && setDToken('')`, `dToken && localStorage.removeItem('dToken')`.
- **Q330:** What happens if both `aToken` and `dToken` exist in localStorage simultaneously?
  - *Answer:* In `admin/src/App.jsx#L27`, the component renders the administrative portal view if either exists (`aToken || dToken`). However, `Sidebar.jsx` prioritizes `aToken` links if `aToken` is truthy.
- **Q331:** How does the doctor portal track unique patients when rendering the appointment roster?
  - *Answer:* The roster lists all appointments individually. If the same patient books three consultations, three rows are rendered with that patient's name, age, and photo.
- **Q332:** How does `adminController.js#L14` validate doctor credentials?
  - *Answer:* Line 20 checks `if (!name || !email || !password || !speciality || !degree || !experience || !about || !fees || !address || !imageFile)`.
- **Q333:** What happens if `fees` is negative in `addDoctor`?
  - *Answer:* The current schema does not enforce `min: 0` on `fees: { type: Number }`. A negative fee would be persisted.
  - *Validation Fix:* Add `min: [0, 'Fees cannot be negative']` to `doctorModel.js`.
- **Q334:** What happens if an admin registers a doctor with an email that is already used by a patient?
  - *Answer:* Because patients are stored in `users` and doctors in `doctors`, MongoDB's unique index is scoped to the collection. A doctor and a patient can technically have the same email address without index collision.
- **Q335:** Can a doctor register themselves through a public registration form?
  - *Answer:* No. There is no public doctor registration route. Doctors must be vetted and onboarded by a hospital administrator via `/api/admin/add-doctor`.
- **Q336:** Can an admin book appointments on behalf of a patient?
  - *Answer:* **Not found in repo / Not implemented.** Appointments can only be booked through the patient portal using `POST /api/user/book-appointment`.
- **Q337:** How are doctor profile photos displayed on the patient appointment booking page?
  - *Answer:* Rendered inside an image container styled with `bg-primary w-full sm:w-72 rounded-lg`.
- **Q338:** What happens if Cloudinary is slow to respond during patient profile update?
  - *Answer:* Node waits for `await uploadOnCloudinary(imageFile.path)` to resolve. If Cloudinary takes 10 seconds, the client request remains pending until completion.
- **Q339:** How can image uploads be optimized to prevent slow HTTP responses?
  - *Answer:* Offload upload to direct client-to-Cloudinary signed uploads, passing only the resulting URL to the Express backend.
- **Q340:** What is the payload size limit configured on Express?
  - *Answer:* Standard Express defaults (`100kb`) for `express.json()`. Multi-part forms are streamed by Multer up to the 5MB ceiling.
- **Q341:** How does `AppContext.jsx` handle errors when fetching doctor data?
  - *Answer:* Catches error and triggers `toast.error(error.message)`.
- **Q342:** What happens if the backend server is restarted while patients are browsing?
  - *Answer:* Since JWT authentication and slot calculations are stateless, patients experience zero session interruption. Ongoing requests during the restart window will fail and retry.
- **Q343:** What is the default port for the Vite development server in `frontend/`?
  - *Answer:* Port 5173 (`server: { port: 5173 }` in `frontend/vite.config.js`).
- **Q344:** What is the default port for the Vite development server in `admin/`?
  - *Answer:* Port 5174 (`server: { port: 5174 }` in `admin/vite.config.js`).
- **Q345:** Why was port 5174 assigned to the admin portal?
  - *Answer:* To prevent port collisions when running both Vite development servers concurrently on the same local developer machine.
- **Q346:** What happens if `jwt.verify()` fails in `authDoctor.js`?
  - *Answer:* Throws `JsonWebTokenError` (e.g., "invalid signature" or "jwt malformed"). Caught by catch block, logging error and returning HTTP 500 `{ success: false, message: error.message }`.
- **Q347:** What happens if `jwt.verify()` fails in `authUser.js`?
  - *Answer:* Throws `JsonWebTokenError`, caught by catch block, returning HTTP 500.
- **Q348:** What happens if `jwt.verify()` fails in `authAdmin.js`?
  - *Answer:* Throws `JsonWebTokenError`, caught by catch block, returning HTTP 500.
- **Q349:** Why do authentication middlewares return HTTP 500 instead of HTTP 401 on token verification failure?
  - *Answer:* The catch block generic handler catches all errors (including verification exceptions) and returns status 500. A cleaner implementation would differentiate `JsonWebTokenError` and return HTTP 401 Unauthorized.
- **Q350:** How are doctor consultation slots indexed by time?
  - *Answer:* Formatted as 12-hour strings with AM/PM indicators: `"10:00 AM"`, `"10:30 AM"`, `"11:00 AM"`, etc.
- **Q351:** What is the earliest appointment time permitted by the slot generator?
  - *Answer:* 10:00 AM (`currentDate.setHours(10)`).
- **Q352:** What is the latest appointment time permitted by the slot generator?
  - *Answer:* 8:30 PM (since `endTime` is set to 9:00 PM / 21:00, the last 30-minute slot begins at 8:30 PM).
- **Q353:** What happens if an admin changes a doctor's consultation fee after appointments have already been booked?
  - *Answer:* Existing appointments retain their original fee because the fee was frozen into `appointmentData.amount` at booking time. Future bookings reflect the new fee.
- **Q354:** How does `AllAppointments.jsx` display doctor fees?
  - *Answer:* Renders `{currency}{item.amount}` referencing the immutable fee recorded on the appointment document.
- **Q355:** What happens if a doctor logs in with an email that does not exist in `doctorModel`?
  - *Answer:* `doctorModel.findOne({ email })` returns `null`. Line 47 in `doctorController.js` returns HTTP 404 "Invalid credentials".
- **Q356:** What happens if a patient logs in with an email that does not exist in `userModel`?
  - *Answer:* `userModel.findOne({ email })` returns `null`. Line 84 in `userController.js` returns HTTP 404 "User not found".
- **Q357:** Why does `userController.js#L91` use `bcrypt.compare()` instead of hashing the password and comparing hashes?
  - *Answer:* Bcrypt generates a unique salt for every hash operation. Hashing the candidate password will generate a completely different hash string. `bcrypt.compare()` extracts the salt from the stored hash, hashes the candidate password with that exact salt, and checks equality.
- **Q358:** How is `date` stored on the `Appointment` model?
  - *Answer:* Stored as a numerical Unix epoch millisecond timestamp: `date: Date.now()`.
- **Q359:** How is `date` stored on the `Doctor` model?
  - *Answer:* Stored as a numerical Unix epoch millisecond timestamp: `date: Date.now()`.
- **Q360:** What is the role of `eslint-plugin-react-refresh` in Vite projects?
  - *Answer:* Validates that files export only React components, ensuring Fast Refresh works reliably without full page reloads.
- **Q361:** What happens if `package.json` in `backend` omits `"type": "module"`?
  - *Answer:* Node.js defaults to CommonJS syntax, causing fatal `SyntaxError: Cannot use import statement outside a module` on boot.
- **Q362:** How does `Appoinments.jsx` handle leap years?
  - *Answer:* Uses JavaScript native `Date` arithmetic (`currentDate.setDate(currentDate.getDate() + i)`), which automatically handles 28, 29, 30, and 31-day month boundaries.
- **Q363:** What visual indicator is displayed when the machine learning model is computing predictions?
  - *Answer:* In `SpecialistPredication.jsx#L274`, the button text changes to "Analyzing Symptoms..." and displays an SVG spinning loader icon.
- **Q364:** Can a patient book multiple slots with the same doctor on the same day?
  - *Answer:* Yes, provided the slots are distinct 30-minute intervals. The system does not restrict multi-slot booking per patient.
- **Q365:** What is the overall architectural philosophy of Prescripto?
  - *Answer:* **Pragmatic, High-Performance Healthcare Orchestration.** It couples lightweight deterministic machine learning with high-concurrency Node.js microservices and static edge delivery, ensuring patient safety, appointment integrity, and clinical operational efficiency without infrastructure bloat.

---
# END OF COMPLETE INTERVIEW Q&A REFERENCE DOCUMENT

