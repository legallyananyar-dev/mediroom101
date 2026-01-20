<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MediRoom | Indian Precision Healthcare</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; scroll-behavior: smooth; }
        .glass { background: rgba(255, 255, 255, 0.85); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); }
        .animate-fade-in { animation: fadeIn 0.4s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .pulse-red { animation: pulseRed 2s infinite; }
        @keyframes pulseRed { 0%, 100% { background-color: #ef4444; } 50% { background-color: #fca5a5; } }
        .shake { animation: shake 0.5s ease-in-out; }
        @keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-5px); } 75% { transform: translateX(5px); } }
    </style>
</head>
<body class="bg-[#F9FAFB] text-slate-900 min-h-screen pb-24 selection:bg-indigo-100">

    <!-- TOP NAVIGATION -->
    <nav class="glass sticky top-0 z-50 border-b border-slate-200/60 px-6 py-4">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-2 cursor-pointer transition-transform hover:scale-[1.02]" onclick="app.navigate('landing')">
                <div class="w-10 h-10 bg-indigo-600 rounded-xl flex items-center justify-center shadow-lg shadow-indigo-100">
                    <i data-lucide="heart" class="text-white w-6 h-6 fill-current"></i>
                </div>
                <span class="font-extrabold text-xl tracking-tighter text-slate-900">MediRoom</span>
            </div>
            <div id="nav-actions" class="flex items-center gap-3">
                <!-- Action buttons injected via JavaScript -->
            </div>
        </div>
    </nav>

    <!-- MAIN APP CONTAINER -->
    <main id="app-root" class="max-w-7xl mx-auto p-6">
        <div class="flex flex-col items-center justify-center py-20 animate-pulse">
            <div class="w-12 h-12 border-4 border-indigo-600 border-t-transparent rounded-full animate-spin mb-4"></div>
            <p class="text-slate-400 font-bold uppercase tracking-widest text-xs">Initializing Secure Portal...</p>
        </div>
    </main>

    <!-- FIXED STATUS FOOTER -->
    <footer class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-sm border-t border-slate-200 py-3 flex justify-between px-6 items-center z-40">
        <p class="text-[10px] text-slate-400 font-bold uppercase tracking-[0.2em]">
            MediRoom Clinical Hub India • Room Code: 4444
        </p>
        <div id="sync-status" class="flex items-center gap-2 opacity-0 transition-opacity">
            <div id="sync-dot" class="w-1.5 h-1.5 rounded-full bg-emerald-500"></div>
            <span id="sync-text" class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Cloud Synced</span>
        </div>
    </footer>

    <script type="module">
        // Firebase Imports from CDN
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- CENTRAL DATA STORE ---
        const INITIAL_PATIENTS = [
            { 
                id: 'p1', name: 'Sunita Gupta', age: 42, condition: 'Post-CABG recovery', 
                drug: 'Atorvastatin', dosage: '20mg', refill: 'Ready for Pickup', risk: 'low',
                compliance: [1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1],
                pharmacology: {
                    logP: 6.3,
                    logP_desc: "Highly lipophilic. High Log P allows rapid liver uptake but increases muscle toxicity risk.",
                    docking: "Binding at HMG-CoA Reductase. Interaction with Ser684 ensures potency.",
                    pv_prediction: "Monitor for Creatine Kinase elevation. High signal for muscle fatigue.",
                    pv_activity: "Active symptom monitoring for myopathy reports."
                },
                doctorNotes: 'Consistent 20-min daily walks. Reports mild stiffness in legs.',
                pharmacistNotes: 'Confirmed no grapefruit juice interaction with patient.',
                recentReport: null
            },
            { 
                id: 'p2', name: 'Rajesh Kumar', age: 68, condition: 'Type 2 Diabetes', 
                drug: 'Metformin', dosage: '500mg (SR)', refill: 'Processing', risk: 'high',
                compliance: [1, 0, 1, 0, 0, 1, 0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0],
                pharmacology: {
                    logP: -1.43,
                    logP_desc: "Highly hydrophilic. Requires OCT1 transporters. Zero passive diffusion.",
                    docking: "Interacts with Mitochondrial Complex I. Indirectly activates AMPK.",
                    pv_prediction: "Risk of B12 deficiency. High vigilance for Lactic Acidosis signals.",
                    pv_activity: "Weekly eGFR monitoring and B12 level baseline checks."
                },
                doctorNotes: 'Reported severe dizziness yesterday. Renal function labs ordered.',
                pharmacistNotes: 'Switched to Sustained Release to help with GI issues.',
                recentReport: { hasProblem: true, severity: 4, timestamp: '2h ago' }
            },
            { 
                id: 'p3', name: 'Ananya Iyer', age: 31, condition: 'Asthma Management', 
                drug: 'Salbutamol', dosage: '100mcg', refill: 'Delivered', risk: 'moderate',
                compliance: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                pharmacology: {
                    logP: 0.64,
                    logP_desc: "Amphiphilic. Log P 0.64 allows rapid local absorption in lungs.",
                    docking: "Selective β2-adrenoceptor agonist. High affinity for the TM pocket.",
                    pv_prediction: "Tachycardia signal possible upon overuse. Monitor pulse.",
                    pv_activity: "Monitoring inhaler actuation frequency for overuse patterns."
                },
                doctorNotes: 'Peak flow remains stable. Use rescue inhaler max 3 times/week.',
                pharmacistNotes: 'Technique checked. Patient using spacer effectively.',
                recentReport: null
            }
        ];

        // --- APP CONTROLLER CLASS ---
        class MediRoomApp {
            constructor() {
                this.patients = INITIAL_PATIENTS;
                this.view = 'landing';
                this.activePatient = null;
                this.isUnlocked = false;
                this.db = null;
                this.auth = null;
                this.appId = typeof __app_id !== 'undefined' ? __app_id : 'default-mediroom-id';
                this.initFirebase();
            }

            async initFirebase() {
                try {
                    // Global config provided by environment
                    const firebaseConfig = JSON.parse(__firebase_config);
                    const fbApp = initializeApp(firebaseConfig);
                    this.auth = getAuth(fbApp);
                    this.db = getFirestore(fbApp);

                    // Authentication Pattern (Rule 3)
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(this.auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(this.auth);
                    }

                    onAuthStateChanged(this.auth, (user) => {
                        if (user) {
                            this.setupRealtimeSync();
                            this.render();
                        }
                    });
                } catch (error) {
                    console.error("Firebase Error. Running in local mode.", error);
                    this.render();
                }
            }

            setupRealtimeSync() {
                if (!this.auth.currentUser) return;
                
                // Firestore Path (Rule 1)
                const docRef = doc(this.db, 'artifacts', this.appId, 'public', 'data', 'app_state');
                
                // Real-time listener
                onSnapshot(docRef, (snap) => {
                    if (snap.exists()) {
                        this.patients = snap.data().patients || INITIAL_PATIENTS;
                        this.updateSyncUI(true);
                        this.render();
                    } else {
                        this.saveToCloud(); // Push initial data if cloud is empty
                    }
                }, (err) => console.error("Snapshot error:", err));
            }

            async saveToCloud() {
                if (!this.auth.currentUser || !this.db) return;
                
                this.updateSyncUI(false);
                const docRef = doc(this.db, 'artifacts', this.appId, 'public', 'data', 'app_state');
                try {
                    await setDoc(docRef, { patients: this.patients, lastUpdated: Date.now() });
                    this.updateSyncUI(true);
                } catch (err) {
                    console.error("Cloud Save Error:", err);
                }
            }

            updateSyncUI(isSynced) {
                const container = document.getElementById('sync-status');
                const dot = document.getElementById('sync-dot');
                const text = document.getElementById('sync-text');
                if (!container) return;
                container.style.opacity = '1';
                dot.className = isSynced ? 'w-1.5 h-1.5 rounded-full bg-emerald-500' : 'w-1.5 h-1.5 rounded-full bg-orange-400 animate-pulse';
                text.innerText = isSynced ? 'Cloud Synced' : 'Syncing...';
            }

            navigate(view, patientId = null) {
                this.view = view;
                if (patientId) {
                    this.activePatient = this.patients.find(p => p.id === patientId);
                }
                this.render();
                window.scrollTo(0, 0);
            }

            render() {
                const root = document.getElementById('app-root');
                const navActions = document.getElementById('nav-actions');
                if (!root) return;
                
                root.innerHTML = '';
                navActions.innerHTML = '';

                // Header Buttons
                if (this.activePatient && this.view !== 'landing') {
                    const hrBtn = document.createElement('button');
                    hrBtn.className = "bg-slate-900 text-white px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest shadow-lg flex items-center gap-2 hover:bg-slate-800 transition-colors";
                    hrBtn.innerHTML = `<i data-lucide="lock" class="w-3 h-3"></i> Health Room`;
                    hrBtn.onclick = () => this.navigate('healthroom');
                    navActions.appendChild(hrBtn);
                }

                // Routing
                switch(this.view) {
                    case 'landing': this.renderLanding(root); break;
                    case 'patient': this.renderPatientDashboard(root); break;
                    case 'doctor': this.renderDoctorDashboard(root); break;
                    case 'pharmacist': this.renderPharmacistDashboard(root); break;
                    case 'healthroom': this.renderHealthRoom(root); break;
                    case 'checkin': this.renderCheckinFlow(root); break;
                }
                lucide.createIcons();
            }

            renderLanding(root) {
                root.innerHTML = `
                    <div class="max-w-md mx-auto py-12 space-y-12 animate-fade-in text-center">
                        <div class="space-y-4">
                            <div class="w-20 h-20 bg-indigo-600 rounded-[2.5rem] flex items-center justify-center mx-auto shadow-2xl rotate-3">
                                <i data-lucide="heart" class="text-white w-10 h-10 fill-current"></i>
                            </div>
                            <h1 class="text-4xl font-black text-slate-900 tracking-tighter">MediRoom</h1>
                            <p class="text-slate-500 font-medium text-sm">Indian Collaborative Precision Care</p>
                        </div>
                        <div class="grid grid-cols-1 gap-3 text-left">
                            <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest ml-1">Patient Identity</p>
                            ${this.patients.map(p => `
                                <button onclick="app.navigate('patient', '${p.id}')" class="bg-white p-5 rounded-3xl border border-slate-200 hover:border-indigo-600 transition-all flex items-center justify-between group shadow-sm hover:shadow-md">
                                    <div class="flex items-center gap-4">
                                        <div class="w-10 h-10 bg-slate-50 rounded-2xl flex items-center justify-center group-hover:bg-indigo-600 group-hover:text-white transition-colors">
                                            <i data-lucide="user" class="w-5 h-5"></i>
                                        </div>
                                        <div>
                                            <p class="font-bold text-slate-800 text-sm">${p.name}</p>
                                            <p class="text-[9px] text-slate-400 font-black uppercase tracking-widest">${p.condition}</p>
                                        </div>
                                    </div>
                                    <i data-lucide="chevron-right" class="text-slate-200 group-hover:text-indigo-600 w-5 h-5"></i>
                                </button>
                            `).join('')}
                            <div class="pt-8 space-y-3 border-t border-slate-100 mt-4">
                                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest ml-1">Professional Gateways</p>
                                <button onclick="app.navigate('doctor')" class="w-full bg-slate-900 text-white p-5 rounded-3xl flex items-center gap-4 shadow-xl">
                                    <i data-lucide="stethoscope" class="w-6 h-6 text-indigo-400"></i>
                                    <div class="text-left"><p class="font-bold text-sm">Dr. Sunita (Clinician)</p><p class="text-[10px] text-slate-400 font-bold uppercase tracking-widest">Diagnostics & PV Portal</p></div>
                                </button>
                                <button onclick="app.navigate('pharmacist')" class="w-full bg-emerald-600 text-white p-5 rounded-3xl flex items-center gap-4 shadow-xl shadow-emerald-100">
                                    <i data-lucide="package" class="w-6 h-6 text-emerald-200"></i>
                                    <div class="text-left"><p class="font-bold text-sm">Arjun Varma (Pharmacist)</p><p class="text-[10px] text-emerald-100 font-bold uppercase tracking-widest">Hospital Referral Hub</p></div>
                                </button>
                            </div>
                        </div>
                    </div>
                `;
            }

            renderPatientDashboard(root) {
                const u = this.activePatient;
                root.innerHTML = `
                    <div class="max-w-md mx-auto space-y-6 animate-fade-in">
                        <header class="flex justify-between items-center">
                            <div><h1 class="text-2xl font-black text-slate-900">Namaste, ${u.name.split(' ')[0]}</h1><p class="text-slate-500 text-sm font-medium">Monitoring your wellness</p></div>
                            <div class="w-12 h-12 bg-white rounded-2xl border flex items-center justify-center text-slate-400 shadow-sm"><i data-lucide="bell"></i></div>
                        </header>

                        ${u.recentReport ? `
                            <div class="p-4 rounded-2xl border flex items-center gap-3 ${u.recentReport.hasProblem ? 'bg-orange-50 border-orange-100 text-orange-800' : 'bg-emerald-50 border-emerald-100 text-emerald-800'}">
                                <i data-lucide="${u.recentReport.hasProblem ? 'alert-triangle' : 'check-circle'}" class="w-5 h-5"></i>
                                <p class="text-xs font-black uppercase tracking-tight">Today: ${u.recentReport.hasProblem ? 'Symptom Logged' : 'Feeling Normal'}</p>
                            </div>
                        ` : ''}

                        <div class="bg-white p-6 rounded-[2.5rem] border border-slate-200 shadow-sm space-y-6">
                            <div>
                                <p class="text-[10px] font-black text-indigo-600 uppercase tracking-[0.2em] mb-2">Prescription</p>
                                <p class="text-2xl font-black text-slate-800">${u.drug}</p>
                                <p class="text-slate-500 font-medium text-sm mt-1">${u.dosage} • Follow Dr. Sunita's timings</p>
                            </div>
                            <div class="p-4 bg-slate-50 rounded-2xl flex items-center justify-between">
                                <div class="flex items-center gap-3"><i data-lucide="shopping-bag" class="text-slate-400 w-5 h-5"></i><span class="text-xs font-bold text-slate-600">Pharmacy Status</span></div>
                                <span class="text-[10px] font-black uppercase text-emerald-600">${u.refill}</span>
                            </div>
                            <button onclick="app.navigate('checkin')" class="w-full bg-slate-900 text-white py-5 rounded-[1.5rem] font-bold shadow-xl flex items-center justify-center gap-2 hover:bg-slate-800">Start Daily Check-in <i data-lucide="chevron-right" class="w-4 h-4"></i></button>
                        </div>

                        <div class="bg-indigo-900 p-6 rounded-[2.5rem] text-white shadow-xl relative overflow-hidden">
                            <i data-lucide="stethoscope" class="absolute top-0 right-0 p-8 text-indigo-800 w-28 h-28 opacity-40"></i>
                            <h3 class="text-xs font-black text-indigo-300 uppercase tracking-widest mb-3 flex items-center gap-2"><i data-lucide="info" class="w-4 h-4"></i> Doctor's Note</h3>
                            <p class="text-sm font-medium leading-relaxed italic relative z-10">"${u.doctorNotes}"</p>
                        </div>
                    </div>
                `;
            }

            renderCheckinFlow(root) {
                root.innerHTML = `
                    <div id="wiz-root" class="max-w-md mx-auto bg-white p-10 rounded-[2.5rem] border border-slate-200 shadow-sm animate-fade-in">
                        <button onclick="app.navigate('patient')" class="mb-10 text-slate-400 font-black text-[10px] uppercase flex items-center gap-1 tracking-widest"><i data-lucide="arrow-left" class="w-3 h-3"></i> Back</button>
                        <div id="wiz-content" class="space-y-8">
                            <div class="space-y-2 text-left">
                                <h2 class="text-3xl font-black text-slate-900 tracking-tighter leading-none">Health Scan</h2>
                                <p class="text-slate-500 font-medium">Are you feeling any physical discomfort or new symptoms today?</p>
                            </div>
                            <div class="grid grid-cols-1 gap-3">
                                <button onclick="app.renderSeveritySelector()" class="w-full py-5 rounded-2xl border-2 border-slate-100 hover:border-indigo-600 hover:bg-indigo-50 font-bold transition-all text-left px-6 flex justify-between">Yes, I feel something <i data-lucide="alert-circle"></i></button>
                                <button onclick="app.finishCheckin(false)" class="w-full py-5 rounded-2xl border-2 border-slate-100 hover:border-emerald-500 hover:bg-emerald-50 font-bold transition-all text-left px-6 flex justify-between">No, I feel fine <i data-lucide="check-circle" class="text-emerald-500"></i></button>
                            </div>
                        </div>
                    </div>
                `;
                lucide.createIcons();
            }

            renderSeveritySelector() {
                const content = document.getElementById('wiz-content');
                content.innerHTML = `
                    <div class="space-y-8 animate-fade-in">
                        <div class="space-y-2">
                            <h2 class="text-2xl font-black text-slate-900 tracking-tight">How intense?</h2>
                            <p class="text-slate-500 font-medium text-sm">Rate discomfort from 1 (mild) to 5 (severe).</p>
                        </div>
                        <div class="flex justify-between gap-2">
                            ${[1, 2, 3, 4, 5].map(n => `<button onclick="app.finishCheckin(true, ${n})" class="w-12 h-12 rounded-xl border-2 border-slate-100 hover:border-indigo-600 font-black text-slate-700 transition-all text-lg">${n}</button>`).join('')}
                        </div>
                    </div>
                `;
            }

            async finishCheckin(hasProblem, severity = 0) {
                this.activePatient.recentReport = { hasProblem, severity, timestamp: 'Just now' };
                await this.saveToCloud();
                
                const root = document.getElementById('app-root');
                root.innerHTML = `
                    <div class="max-w-md mx-auto text-center p-12 bg-white rounded-[2.5rem] shadow-sm border border-slate-200 animate-fade-in">
                        <div class="w-20 h-20 bg-emerald-50 rounded-full flex items-center justify-center mx-auto mb-6"><i data-lucide="check" class="text-emerald-500 w-10 h-10"></i></div>
                        <h2 class="text-3xl font-black text-slate-900 mb-2">Synced</h2>
                        <p class="text-slate-500 font-medium mb-8">Clinical update logged in the shared Health Room.</p>
                        ${severity >= 3 ? `<div class="bg-red-50 border border-red-100 p-5 rounded-2xl text-left mb-8"><p class="text-[10px] font-black text-red-600 uppercase mb-1 flex items-center gap-1"><i data-lucide="zap" class="w-3 h-3"></i> Severity Warning</p><p class="text-xs font-bold text-red-900 leading-tight italic">Level ${severity} discomfort. Dr. Sunita will be alerted. Please rest immediately.</p></div>` : ''}
                        <button onclick="app.navigate('patient')" class="w-full bg-slate-900 text-white py-5 rounded-2xl font-black uppercase text-xs tracking-widest">Back to Dashboard</button>
                    </div>
                `;
                lucide.createIcons();
            }

            renderDoctorDashboard(root) {
                root.innerHTML = `
                    <div class="space-y-8 animate-fade-in text-left">
                        <header class="flex justify-between items-end">
                            <div><h1 class="text-3xl font-black text-slate-900 tracking-tighter uppercase leading-none">Clinician Hub</h1><p class="text-slate-500 font-medium">Monitoring Precision PV Signals</p></div>
                            <div class="w-10 h-10 bg-indigo-100 rounded-xl flex items-center justify-center text-indigo-600 font-black shadow-sm">SA</div>
                        </header>
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                            ${this.patients.map(p => `
                                <div class="bg-white p-6 rounded-[2.2rem] border shadow-sm space-y-4 ${p.recentReport?.severity >= 3 ? 'border-red-400 ring-4 ring-red-50' : 'border-slate-100'}">
                                    <div class="flex justify-between items-start">
                                        <div><p class="font-bold text-lg text-slate-800 leading-none">${p.name}</p><p class="text-[10px] font-black text-indigo-600 uppercase tracking-widest mt-2">${p.condition}</p></div>
                                        <div class="w-3 h-3 rounded-full ${p.recentReport?.severity >= 3 ? 'pulse-red' : (p.recentReport ? 'bg-emerald-500' : 'bg-slate-200')}"></div>
                                    </div>
                                    <button onclick="app.navigate('healthroom', '${p.id}')" class="w-full bg-slate-900 text-white py-4 rounded-2xl font-black text-[10px] uppercase tracking-widest">View Room Analytics</button>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            }

            renderPharmacistDashboard(root) {
                root.innerHTML = `
                    <div class="space-y-8 animate-fade-in text-left">
                        <header class="flex justify-between items-end">
                            <div><h1 class="text-3xl font-black text-slate-900 tracking-tighter uppercase leading-none">Pharmacy Portal</h1><p class="text-slate-500 font-medium">Fulfillment & Precision Verify</p></div>
                            <div class="w-10 h-10 bg-emerald-100 rounded-xl flex items-center justify-center text-emerald-600 font-black shadow-sm">AV</div>
                        </header>
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                            ${this.patients.map(p => `
                                <div class="bg-white p-6 rounded-[2.2rem] border border-slate-100 shadow-sm space-y-4">
                                    <div class="flex justify-between items-start">
                                        <div><p class="font-bold text-lg text-slate-800 leading-none">${p.name}</p><p class="text-[10px] font-black text-emerald-600 uppercase tracking-widest mt-2">${p.drug}</p></div>
                                        <div class="text-[8px] font-black bg-emerald-50 text-emerald-600 px-2 py-1 rounded uppercase">${p.refill}</div>
                                    </div>
                                    <button onclick="app.navigate('healthroom', '${p.id}')" class="w-full bg-emerald-600 text-white py-4 rounded-2xl font-black text-[10px] uppercase tracking-widest">Molecular Verify</button>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            }

            renderHealthRoom(root) {
                if (!this.isUnlocked) {
                    root.innerHTML = `
                        <div class="max-w-md mx-auto text-center py-12 animate-fade-in">
                            <div class="w-20 h-20 bg-slate-100 rounded-[2.5rem] flex items-center justify-center mx-auto mb-8 shadow-inner"><i data-lucide="lock" class="text-slate-400 w-8 h-8"></i></div>
                            <h2 class="text-2xl font-black text-slate-900 mb-2">Health Room Passcode</h2>
                            <p class="text-slate-500 mb-10 text-sm">Enter the shared code (4444) to view clinical data.</p>
                            <input id="pin-field" type="password" maxlength="4" class="text-center text-5xl tracking-[1.5rem] w-full max-w-[220px] bg-white border-2 border-slate-200 rounded-3xl py-6 focus:border-indigo-600 outline-none transition-all font-black mb-10" placeholder="****">
                            <button onclick="app.checkPasscode()" class="w-full bg-slate-900 text-white py-5 rounded-2xl font-black text-lg">Unlock Room</button>
                        </div>
                    `;
                    document.getElementById('pin-field').focus();
                } else {
                    const p = this.activePatient;
                    root.innerHTML = `
                        <div class="max-w-5xl mx-auto space-y-8 animate-fade-in text-left">
                            <header class="flex justify-between items-center">
                                <div class="flex items-center gap-4">
                                    <div class="w-14 h-14 bg-indigo-600 rounded-2xl flex items-center justify-center text-white shadow-xl shadow-indigo-200"><i data-lucide="users"></i></div>
                                    <div><h2 class="text-2xl font-black text-slate-900 leading-none mb-1">${p.name}</h2><p class="text-xs text-slate-400 font-bold uppercase tracking-widest">Shared Portal • Dr. Sunita Aris + Arjun Varma</p></div>
                                </div>
                                <button onclick="app.isUnlocked=false; app.render();" class="text-xs bg-white border border-slate-200 px-4 py-2 rounded-xl font-bold hover:bg-slate-50 transition-colors">EXIT SECURE VIEW</button>
                            </header>

                            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                                <div class="lg:col-span-2 space-y-6">
                                    <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200 shadow-sm space-y-8">
                                        <div class="flex items-center gap-2 text-indigo-600"><i data-lucide="microscope" class="w-5 h-5"></i><h3 class="font-black text-sm uppercase tracking-widest">Molecular Precision</h3></div>
                                        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                                            <div class="space-y-4">
                                                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Log P Analysis</p>
                                                <div class="bg-indigo-50 p-6 rounded-2xl border border-indigo-100 flex items-center gap-4">
                                                    <span class="text-3xl font-black text-indigo-600">${p.pharmacology.logP}</span>
                                                    <p class="text-[11px] font-bold text-indigo-800 leading-tight">${p.pharmacology.logP_desc}</p>
                                                </div>
                                            </div>
                                            <div class="space-y-4">
                                                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Docking (HMG-CoA)</p>
                                                <div class="h-28 bg-slate-900 rounded-2xl flex items-center justify-center relative overflow-hidden">
                                                    <svg viewBox="0 0 100 100" class="w-16 h-16 text-indigo-400 animate-pulse"><path d="M20 50 Q50 10 80 50 T110 50" fill="none" stroke="currentColor" strokeWidth="3" /></svg>
                                                    <span class="absolute bottom-2 right-3 text-[9px] font-mono text-indigo-300">-8.2 kcal/mol</span>
                                                </div>
                                                <p class="text-[10px] text-slate-400 italic font-medium">${p.pharmacology.docking}</p>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="bg-indigo-900 rounded-[2.5rem] p-8 text-white shadow-xl relative overflow-hidden">
                                        <h3 class="text-xs font-black text-orange-400 uppercase tracking-widest mb-6 flex items-center gap-2"><i data-lucide="zap" class="w-4 h-4"></i> PV Signal Predictor</h3>
                                        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                            <div class="bg-white/5 p-5 rounded-2xl border border-white/10"><p class="text-[9px] font-bold text-indigo-300 uppercase mb-1">Drug Outcome</p><p class="text-xs font-medium">${p.pharmacology.pv_prediction}</p></div>
                                            <div class="bg-white/5 p-5 rounded-2xl border border-white/10"><p class="text-[9px] font-bold text-indigo-300 uppercase mb-1">Activity Loop</p><p class="text-xs font-medium">${p.pharmacology.pv_activity}</p></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="space-y-6">
                                    <div class="bg-white p-6 rounded-[2.5rem] border border-slate-200 shadow-sm space-y-4">
                                        <h3 class="text-xs font-black text-slate-400 uppercase tracking-widest">Shared Portal Notes</h3>
                                        <div class="p-4 bg-indigo-50 rounded-2xl text-[11px] font-medium text-indigo-800 leading-relaxed italic">" ${p.doctorNotes} "</div>
                                        <div class="p-4 bg-emerald-50 rounded-2xl text-[11px] font-medium text-emerald-800 leading-relaxed italic">" ${p.pharmacistNotes} "</div>
                                        <textarea id="collab-note" class="w-full bg-slate-50 border border-slate-200 rounded-2xl p-4 text-xs outline-none focus:ring-1 focus:ring-indigo-500" rows="3" placeholder="Update Dr. Sunita..."></textarea>
                                        <button onclick="app.postNote()" class="w-full bg-slate-900 text-white py-4 rounded-xl font-black text-[10px] uppercase">Post Observe</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    `;
                }
                lucide.createIcons();
            }

            checkPasscode() {
                const val = document.getElementById('pin-field').value;
                if (val === '4444') {
                    this.isUnlocked = true;
                    this.render();
                } else {
                    document.getElementById('pin-field').classList.add('shake', 'border-red-400');
                    setTimeout(() => document.getElementById('pin-field').classList.remove('shake', 'border-red-400'), 500);
                }
            }

            async postNote() {
                const note = document.getElementById('collab-note').value;
                if (!note) return;
                this.activePatient.doctorNotes = note;
                await this.saveToCloud();
                document.getElementById('collab-note').value = '';
            }
        }

        // Global instance
        window.app = new MediRoomApp();
    </script>
</body>
</html>
