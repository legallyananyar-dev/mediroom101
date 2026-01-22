<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MediRoom | Clinical Precision PV</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; scroll-behavior: smooth; }
        .glass { background: rgba(255, 255, 255, 0.85); backdrop-filter: blur(12px); }
        .animate-fade-in { animation: fadeIn 0.3s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
        .recording-pulse { animation: pulse 1.5s infinite; }
        @keyframes pulse { 0% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.2); opacity: 0.5; } 100% { transform: scale(1); opacity: 1; } }
        .shake { animation: shake 0.4s ease-in-out; }
        @keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-4px); } 75% { transform: translateX(4px); } }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .danger-pulse { animation: dangerPulse 1s infinite; }
        @keyframes dangerPulse { 0% { box-shadow: 0 0 0 0 rgba(225, 29, 72, 0.7); } 70% { box-shadow: 0 0 0 15px rgba(225, 29, 72, 0); } 100% { box-shadow: 0 0 0 0 rgba(225, 29, 72, 0); } }
        .logp-indicator { transition: all 1s ease-in-out; }
    </style>
</head>
<body class="bg-[#F8FAFC] text-slate-900 min-h-screen">

    <nav id="main-nav" class="glass sticky top-0 z-50 border-b border-slate-200/60 px-6 py-3 hidden">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 bg-indigo-600 rounded-xl flex items-center justify-center">
                    <i data-lucide="activity" class="text-white w-5 h-5"></i>
                </div>
                <div>
                    <span class="font-extrabold text-lg tracking-tighter block leading-none">MediRoom</span>
                    <span class="text-[8px] font-black text-indigo-500 uppercase tracking-widest">Live PV Intelligence</span>
                </div>
            </div>
            <div id="nav-actions" class="flex items-center gap-4"></div>
        </div>
    </nav>

    <main id="app-root"></main>

    <footer id="app-footer" class="fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 py-2 flex justify-between px-6 items-center z-40 hidden">
        <div class="flex items-center gap-4">
            <span class="text-[9px] text-slate-400 font-bold uppercase tracking-widest">System: Live Cloud Sync</span>
            <span class="text-[9px] text-slate-400 font-bold uppercase tracking-widest">|</span>
            <span class="text-[9px] text-slate-400 font-bold uppercase tracking-widest">Shared Room: 4444</span>
        </div>
        <div id="sync-indicator" class="flex items-center gap-2">
            <div class="w-1.5 h-1.5 rounded-full bg-emerald-500"></div>
            <span class="text-[9px] font-black text-slate-500 uppercase tracking-tighter">Synced</span>
        </div>
    </footer>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const CREDENTIALS = { PATIENT: '12345', DOCTOR: '67891', PHARMACIST: '121212', ROOM: '4444' };

        const INITIAL_DATA = {
            patients: [{
                id: 'p1',
                name: 'Sunita Gupta',
                age: 42,
                diagnosis: 'Post-CABG (Triple Vessel Disease), Type 2 Diabetes Mellitus, Chronic Hypertension.',
                clinicalStatus: 'Stabilizing - Outpatient Monitoring',
                emergencyContact: { name: 'Raghu', relation: 'Spouse/Emergency Response', status: 'Online & Synced', notified: false },
                clinicalSummaryNote: 'Patient is on a high-intensity statin regimen post-bypass. Primary concern: monitoring for HMG-CoA reductase inhibitor-induced myopathy (muscle pain) and ensuring ACE-inhibitor/statin compatibility.',
                prescription: [
                    { 
                        id: 'm1', name: 'Atorvastatin', dose: '20mg', freq: 'Once Daily (Night)', delivered: true, restocked: true, 
                        dockingScore: '-8.4 kcal/mol', toxicity: 'Low', logP: 4.1,
                        simpleDocking: 'Strong fit: Like a tight key in a lock, preventing cholesterol buildup.',
                        simpleToxicity: 'Safe levels: Very low risk of liver or muscle stress at this dose.',
                        criticalRisk: 'Rhabdomyolysis Risk: High muscle toxicity predicted if myalgia persists.'
                    },
                    { 
                        id: 'm2', name: 'Aspirin (EC)', dose: '75mg', freq: 'Once Daily (Morning)', delivered: false, restocked: true, 
                        dockingScore: '-6.1 kcal/mol', toxicity: 'Low', logP: 1.2,
                        simpleDocking: 'Active block: Keeps blood cells from sticking together too easily.',
                        simpleToxicity: 'Stomach safe: Enteric coating protects your lining from irritation.',
                        criticalRisk: 'Internal Bleeding Risk: High risk of GI hemorrhage identified.'
                    },
                    { 
                        id: 'm3', name: 'Metformin', dose: '500mg', freq: 'Twice Daily (Post-Meal)', delivered: true, restocked: false, 
                        dockingScore: '-5.2 kcal/mol', toxicity: 'Medium', logP: -1.4,
                        simpleDocking: 'Stable fit: Helping your insulin work better to lower blood sugar.',
                        simpleToxicity: 'Digestive check: May cause minor stomach upset; take strictly after meals.',
                        criticalRisk: 'Lactic Acidosis: Severe metabolic imbalance risk due to drug-organ stress.'
                    }
                ],
                symptoms: {
                    myalgia: null,
                    darkUrine: null,
                    bruising: null,
                    dizziness: null
                },
                adherence: [
                    { day: 'Mon', status: 'taken' }, { day: 'Tue', status: 'taken' }, { day: 'Wed', status: 'missed' },
                    { day: 'Thu', status: 'taken' }, { day: 'Fri', status: 'pending' }, { day: 'Sat', status: 'pending' },
                    { day: 'Sun', status: 'pending' }
                ],
                notes: [],
                currentQuestionIndex: 0
            }]
        };

        class MediRoom {
            constructor() {
                this.state = JSON.parse(JSON.stringify(INITIAL_DATA));
                this.view = 'login';
                this.userRole = null;
                this.isRecording = false;
                this.db = null;
                this.unlockedRoom = false;
                this.appId = typeof __app_id !== 'undefined' ? __app_id : 'mediroom-p1-sync-v2';
                this.init();
            }

            async init() {
                try {
                    const fbApp = initializeApp(JSON.parse(__firebase_config));
                    const auth = getAuth(fbApp);
                    this.db = getFirestore(fbApp);
                    await signInAnonymously(auth);
                    onAuthStateChanged(auth, (user) => { if (user) this.sync(); });
                } catch (e) { this.render(); }
            }

            sync() {
                if (!this.db) return;
                onSnapshot(doc(this.db, 'artifacts', this.appId, 'public', 'data', 'state'), (snap) => {
                    if (snap.exists()) {
                        this.state = snap.data();
                        this.render();
                    }
                });
            }

            async save() {
                if (!this.db) return;
                const indicator = document.getElementById('sync-indicator');
                if (indicator) indicator.innerHTML = '<div class="w-1.5 h-1.5 rounded-full bg-orange-400 animate-pulse"></div><span class="text-[9px] font-black text-slate-500 uppercase tracking-tighter">Saving...</span>';
                await setDoc(doc(this.db, 'artifacts', this.appId, 'public', 'data', 'state'), this.state);
            }

            render() {
                const root = document.getElementById('app-root');
                const nav = document.getElementById('main-nav');
                const footer = document.getElementById('app-footer');
                const navActions = document.getElementById('nav-actions');

                if (this.view === 'login') {
                    nav.classList.add('hidden'); footer.classList.add('hidden');
                    this.renderLogin(root);
                } else {
                    nav.classList.remove('hidden'); footer.classList.remove('hidden');
                    navActions.innerHTML = `
                        <button onclick="app.view='shared-room'; app.render();" class="bg-indigo-600 text-white px-4 py-2 rounded-lg text-[10px] font-black uppercase tracking-widest">Shared Room Hub</button>
                        <button onclick="app.logout()" class="text-slate-400 hover:text-rose-500 transition-colors"><i data-lucide="log-out" class="w-5 h-5"></i></button>
                    `;
                    if (this.view === 'patient') this.renderPatient(root);
                    if (this.view === 'doctor') this.renderDoctor(root);
                    if (this.view === 'pharmacist') this.renderPharmacist(root);
                    if (this.view === 'shared-room') this.renderSharedRoom(root);
                }
                lucide.createIcons();
            }

            renderLogin(root) {
                root.innerHTML = `
                <div class="min-h-screen flex items-center justify-center p-6 bg-slate-50">
                    <div id="login-card" class="w-full max-w-md bg-white rounded-[2.5rem] p-10 shadow-xl border border-slate-100">
                        <div class="mb-8">
                            <h1 class="text-3xl font-black tracking-tighter">MediRoom</h1>
                            <p class="text-slate-500 text-sm mt-1 font-medium">Precision Pharmacovigilance System</p>
                        </div>
                        <div class="space-y-4">
                            <input id="login-id" type="tel" placeholder="Enter ID" class="w-full px-6 py-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none font-bold focus:ring-2 focus:ring-indigo-100 transition-all">
                            <button onclick="app.handleLogin()" class="w-full py-4 bg-indigo-600 text-white rounded-2xl font-black uppercase tracking-widest hover:bg-indigo-700 transition-all">Login</button>
                        </div>
                        
                        <div class="mt-10 pt-8 border-t border-slate-50">
                            <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-4">Prototype Access Codes</p>
                            <div class="grid grid-cols-2 gap-3">
                                <div class="bg-slate-50 p-3 rounded-xl">
                                    <p class="text-[9px] font-black text-slate-400 uppercase">Patient</p>
                                    <p class="text-xs font-black text-slate-700">12345</p>
                                </div>
                                <div class="bg-slate-50 p-3 rounded-xl">
                                    <p class="text-[9px] font-black text-slate-400 uppercase">Doctor</p>
                                    <p class="text-xs font-black text-slate-700">67891</p>
                                </div>
                                <div class="bg-slate-50 p-3 rounded-xl">
                                    <p class="text-[9px] font-black text-slate-400 uppercase">Pharmacist</p>
                                    <p class="text-xs font-black text-slate-700">121212</p>
                                </div>
                                <div class="bg-slate-50 p-3 rounded-xl">
                                    <p class="text-[9px] font-black text-slate-400 uppercase">Shared Hub</p>
                                    <p class="text-xs font-black text-slate-700">4444</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>`;
            }

            handleLogin() {
                const id = document.getElementById('login-id').value;
                if (id === CREDENTIALS.PATIENT) { this.userRole = 'patient'; this.view = 'patient'; }
                else if (id === CREDENTIALS.DOCTOR) { this.userRole = 'doctor'; this.view = 'doctor'; }
                else if (id === CREDENTIALS.PHARMACIST) { this.userRole = 'pharmacist'; this.view = 'pharmacist'; }
                else { document.getElementById('login-card').classList.add('shake'); setTimeout(()=>document.getElementById('login-card').classList.remove('shake'), 400); return; }
                this.render();
            }

            logout() { this.view = 'login'; this.userRole = null; this.unlockedRoom = false; this.render(); }

            // --- PATIENT VIEW ---
            renderPatient(root) {
                const p = this.state.patients[0];
                const questions = [
                    { key: 'myalgia', text: 'Are you experiencing any new or unusual muscle pain or weakness?' },
                    { key: 'darkUrine', text: 'Have you noticed your urine appearing darker than usual (cola-colored)?' },
                    { key: 'bruising', text: 'Are you seeing any unexplained bruising or small red spots on your skin?' },
                    { key: 'dizziness', text: 'Do you feel unusually dizzy or lightheaded today?' }
                ];
                
                const idx = p.currentQuestionIndex || 0;
                const isFinished = idx >= questions.length;

                root.innerHTML = `
                <div class="max-w-xl mx-auto p-6 space-y-8 pb-32">
                    <div class="bg-white rounded-[2.5rem] border border-slate-200 p-8 shadow-sm">
                         <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-4">Adherence Calendar</h3>
                         <div class="flex justify-between items-center bg-slate-50 p-4 rounded-2xl">
                             ${p.adherence.map((d, i) => `
                                <div class="flex flex-col items-center gap-1">
                                    <span class="text-[9px] font-bold text-slate-400 uppercase">${d.day}</span>
                                    <button onclick="app.toggleDay(${i})" class="w-8 h-8 rounded-full flex items-center justify-center border transition-all ${
                                        d.status === 'taken' ? 'bg-emerald-500 border-emerald-600 text-white' : 
                                        d.status === 'missed' ? 'bg-rose-500 border-rose-600 text-white' : 
                                        'bg-white border-slate-200 text-slate-300'
                                    }">
                                        <i data-lucide="${d.status === 'taken' ? 'check' : d.status === 'missed' ? 'x' : 'circle'}" class="w-3.5 h-3.5"></i>
                                    </button>
                                </div>
                             `).join('')}
                         </div>
                    </div>

                    <div class="bg-white rounded-[2.5rem] border border-slate-200 p-8 shadow-sm">
                        ${!isFinished ? `
                            <div class="mb-8 flex justify-between items-center">
                                <span class="text-[10px] font-black text-indigo-600 uppercase">Step ${idx + 1} of ${questions.length}</span>
                                <div class="h-1.5 w-32 bg-slate-100 rounded-full overflow-hidden">
                                    <div class="h-full bg-indigo-600" style="width: ${((idx + 1) / questions.length) * 100}%"></div>
                                </div>
                            </div>
                            <h2 class="text-2xl font-black mb-10 leading-tight">${questions[idx].text}</h2>
                            <div class="grid grid-cols-2 gap-4">
                                <button onclick="app.answerQ('${questions[idx].key}', true)" class="py-6 bg-rose-50 border border-rose-100 rounded-2xl text-rose-600 font-black text-lg hover:bg-rose-100 transition-colors">Yes</button>
                                <button onclick="app.answerQ('${questions[idx].key}', false)" class="py-6 bg-slate-50 border border-slate-100 rounded-2xl text-slate-600 font-black text-lg hover:bg-slate-100 transition-colors">No</button>
                            </div>
                        ` : `
                            <div class="text-center py-6">
                                <div class="w-16 h-16 bg-emerald-100 text-emerald-600 rounded-full flex items-center justify-center mx-auto mb-4"><i data-lucide="check" class="w-8 h-8"></i></div>
                                <h2 class="text-2xl font-black mb-2 tracking-tight">Daily Logs Completed</h2>
                                <p class="text-slate-500 text-sm mb-6">Updates have been shared with Sunita's care room.</p>
                                <button onclick="app.resetQ()" class="text-indigo-600 font-bold text-xs uppercase underline tracking-wider">Restart Log</button>
                            </div>
                        `}
                    </div>

                    <div class="bg-white rounded-[2.5rem] border border-slate-200 p-8 shadow-sm space-y-6">
                        <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Additional Observations</h3>
                        <textarea id="patient-note-text" class="w-full bg-slate-50 rounded-2xl p-4 text-sm font-medium outline-none border-none focus:ring-2 focus:ring-indigo-100" rows="3" placeholder="How are you feeling otherwise?"></textarea>
                        
                        <div class="flex gap-4">
                            <button onclick="app.toggleRecording()" id="voice-btn" class="flex-1 py-4 rounded-2xl flex items-center justify-center gap-3 font-black text-[10px] uppercase tracking-widest ${this.isRecording ? 'bg-rose-600 text-white shadow-lg' : 'bg-slate-100 text-slate-600'} transition-all">
                                <i data-lucide="mic" class="${this.isRecording ? 'recording-pulse' : ''} w-4 h-4"></i>
                                ${this.isRecording ? 'Capturing...' : 'Record Voice'}
                            </button>
                            <button onclick="app.addTextNote()" class="flex-1 bg-indigo-600 text-white py-4 rounded-2xl font-black text-[10px] uppercase tracking-widest hover:shadow-lg transition-all">Submit Note</button>
                        </div>
                    </div>
                </div>`;
            }

            async toggleDay(idx) {
                const states = ['pending', 'taken', 'missed'];
                const current = this.state.patients[0].adherence[idx].status;
                const next = states[(states.indexOf(current) + 1) % 3];
                this.state.patients[0].adherence[idx].status = next;
                await this.save();
                this.render();
            }

            async answerQ(key, val) {
                this.state.patients[0].symptoms[key] = val;
                this.state.patients[0].currentQuestionIndex++;
                await this.save();
                this.render();
            }

            async resetQ() {
                this.state.patients[0].currentQuestionIndex = 0;
                await this.save();
                this.render();
            }

            async toggleRecording() {
                this.isRecording = !this.isRecording;
                this.render();
                if (!this.isRecording) {
                    this.state.patients[0].notes.push({
                        type: 'voice',
                        content: 'Clinical Audio Log (' + new Date().toLocaleTimeString() + ')',
                        sender: 'Sunita',
                        timestamp: new Date().toISOString()
                    });
                    await this.save();
                }
            }

            async addTextNote() {
                const el = document.getElementById('patient-note-text');
                if (!el.value.trim()) return;
                this.state.patients[0].notes.push({
                    type: 'text',
                    content: el.value.trim(),
                    sender: this.userRole === 'patient' ? 'Sunita' : 'Doctor',
                    timestamp: new Date().toISOString()
                });
                el.value = '';
                await this.save();
                this.render();
            }

            // --- PHARMACIST VIEW ---
            renderPharmacist(root) {
                const p = this.state.patients[0];
                root.innerHTML = `
                <div class="max-w-4xl mx-auto p-6 space-y-8 pb-24">
                    <div class="bg-white rounded-[2.5rem] border border-slate-200 p-10 shadow-sm">
                        <h2 class="text-3xl font-black tracking-tighter mb-8">Inventory Console</h2>
                        <div class="space-y-4">
                            ${p.prescription.map(rx => `
                                <div class="flex items-center justify-between p-6 bg-slate-50 rounded-2xl border border-slate-100">
                                    <div class="flex-1">
                                        <h4 class="font-black text-slate-900">${rx.name}</h4>
                                        <p class="text-[10px] text-slate-500 font-bold uppercase">${rx.dose}</p>
                                    </div>
                                    <div class="flex gap-3">
                                        <button onclick="app.toggleStock('${rx.id}', 'restocked')" class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest border transition-all ${rx.restocked ? 'bg-emerald-100 border-emerald-200 text-emerald-700' : 'bg-white border-slate-200 text-slate-400'}">
                                            ${rx.restocked ? 'In Stock' : 'Out of Stock'}
                                        </button>
                                        <button onclick="app.toggleStock('${rx.id}', 'delivered')" class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest border transition-all ${rx.delivered ? 'bg-indigo-600 border-indigo-700 text-white shadow-lg' : 'bg-rose-500 border-rose-600 text-white'}">
                                            ${rx.delivered ? 'Delivered' : 'Pending'}
                                        </button>
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                </div>`;
            }

            async toggleStock(medId, field) {
                const med = this.state.patients[0].prescription.find(m => m.id === medId);
                if (med) med[field] = !med[field];
                await this.save();
                this.render();
            }

            // --- SHARED ROOM HUB ---
            renderSharedRoom(root) {
                if (!this.unlockedRoom) {
                    root.innerHTML = `
                    <div class="min-h-[80vh] flex items-center justify-center p-6">
                        <div class="bg-white p-10 rounded-[2.5rem] shadow-xl border border-slate-100 max-w-sm w-full text-center">
                            <h2 class="text-2xl font-black mb-8 tracking-tighter">Shared Room Access</h2>
                            <input id="room-pin" type="password" maxlength="4" placeholder="••••" class="w-full text-center text-3xl font-black bg-slate-50 border border-slate-200 rounded-xl p-4 mb-6 outline-none focus:ring-2 focus:ring-indigo-100 transition-all">
                            <button onclick="app.unlockRoom()" class="w-full py-4 bg-indigo-600 text-white rounded-xl font-black uppercase tracking-widest hover:bg-indigo-700">Open Hub</button>
                        </div>
                    </div>`;
                    return;
                }

                const p = this.state.patients[0];
                const allPositive = Object.values(p.symptoms).every(v => v === true);

                root.innerHTML = `
                <div class="max-w-7xl mx-auto p-6 space-y-8 pb-24">
                    <div class="grid grid-cols-1 lg:grid-cols-4 gap-8">
                        
                        <div class="lg:col-span-3 space-y-8">
                            
                            <!-- Header Info -->
                            <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200">
                                <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-6 mb-8">
                                    <div class="flex items-center gap-6">
                                        <div class="w-20 h-20 bg-slate-100 rounded-3xl flex items-center justify-center border border-slate-200">
                                            <i data-lucide="user" class="w-10 h-10 text-slate-400"></i>
                                        </div>
                                        <div>
                                            <h2 class="text-2xl font-black tracking-tight">${p.name}</h2>
                                            <p class="text-indigo-600 text-[10px] font-black uppercase tracking-widest mt-1">${p.clinicalStatus}</p>
                                        </div>
                                    </div>
                                    <div class="flex flex-col gap-2">
                                        <div class="bg-rose-50 border border-rose-100 p-4 rounded-2xl flex items-center gap-4 ${p.emergencyContact.notified ? 'danger-pulse bg-rose-600 text-white' : ''}">
                                            <div class="w-10 h-10 ${p.emergencyContact.notified ? 'bg-white text-rose-600' : 'bg-rose-500 text-white'} rounded-xl flex items-center justify-center shadow-lg">
                                                <i data-lucide="${p.emergencyContact.notified ? 'alert-octagon' : 'phone-forwarded'}" class="w-5 h-5"></i>
                                            </div>
                                            <div>
                                                <p class="text-[9px] font-black uppercase tracking-widest">Emergency: ${p.emergencyContact.name}</p>
                                                <p class="text-xs font-bold">${p.emergencyContact.notified ? 'URGENT NOTIFIED' : p.emergencyContact.status}</p>
                                            </div>
                                        </div>
                                        ${this.userRole === 'doctor' ? `
                                            <button onclick="app.notifyEmergency()" class="w-full py-2 bg-slate-900 text-white rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-rose-700 transition-colors">
                                                ${p.emergencyContact.notified ? 'Update Raghu' : 'Notify Raghu as Critical'}
                                            </button>
                                        ` : ''}
                                    </div>
                                </div>
                                
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                                    <div class="space-y-4">
                                        <h4 class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Confirmed Diagnosis</h4>
                                        <div class="p-4 bg-slate-50 rounded-2xl border border-slate-100">
                                            <p class="text-sm font-semibold text-slate-700 leading-relaxed">${p.diagnosis}</p>
                                        </div>
                                    </div>
                                    <div class="space-y-4">
                                        <h4 class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Clinical Summary Note</h4>
                                        <div class="p-4 bg-indigo-50/30 rounded-2xl border border-indigo-100/50">
                                            <p class="text-xs font-medium text-slate-600 leading-relaxed italic">"${p.clinicalSummaryNote}"</p>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <!-- LogP Assessment Feature (New) -->
                            <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200 overflow-hidden relative">
                                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-6">Physicochemical Profile: LogP Assessment</h3>
                                
                                <div class="flex flex-col md:flex-row gap-8 items-center">
                                    <!-- Pictorial Depiction -->
                                    <div class="w-full md:w-1/3 flex flex-col items-center">
                                        <div class="relative w-48 h-48 bg-slate-50 rounded-full border-4 border-slate-100 flex items-center justify-center overflow-hidden">
                                            <!-- Water (Hydrophilic) Layer -->
                                            <div class="absolute bottom-0 w-full h-1/2 bg-blue-100/50"></div>
                                            <!-- Oil (Lipophilic) Layer -->
                                            <div class="absolute top-0 w-full h-1/2 bg-amber-50/50 border-b border-slate-200"></div>
                                            
                                            <!-- Molecule Simulation -->
                                            <div id="molecule-sim" class="relative z-10 transition-all duration-[2s] flex flex-col items-center" style="transform: translateY(${allPositive ? '-40px' : '20px'})">
                                                <div class="w-12 h-12 rounded-full shadow-2xl ${allPositive ? 'bg-rose-500 animate-pulse' : 'bg-indigo-600'} flex items-center justify-center text-white text-[8px] font-black uppercase">Drug</div>
                                                ${allPositive ? '<div class="mt-2 text-[8px] font-black text-rose-600 uppercase">Tissue Accumulation</div>' : ''}
                                            </div>

                                            <!-- Labels -->
                                            <div class="absolute top-4 right-4 text-[7px] font-black text-amber-600 uppercase">Lipid Phase</div>
                                            <div class="absolute bottom-4 right-4 text-[7px] font-black text-blue-600 uppercase">Aqueous Phase</div>
                                        </div>
                                    </div>

                                    <!-- Summary Content -->
                                    <div class="flex-1 space-y-4">
                                        <div class="flex items-center gap-4">
                                            <div class="px-4 py-2 bg-slate-900 text-white rounded-xl">
                                                <p class="text-[8px] font-black uppercase text-indigo-400">Mean Pred. LogP</p>
                                                <p class="text-xl font-black tracking-tighter">4.12</p>
                                            </div>
                                            <div class="flex-1 h-2 bg-slate-100 rounded-full overflow-hidden flex">
                                                <div class="h-full bg-blue-400" style="width: 30%"></div>
                                                <div class="h-full bg-indigo-500" style="width: 70%"></div>
                                            </div>
                                        </div>

                                        <div class="p-5 rounded-2xl ${allPositive ? 'bg-rose-50 border border-rose-100' : 'bg-slate-50 border border-slate-100'}">
                                            <h4 class="text-[9px] font-black uppercase mb-2 ${allPositive ? 'text-rose-600' : 'text-slate-500'}">Critical Assessment</h4>
                                            <p class="text-xs font-semibold leading-relaxed text-slate-700">
                                                ${allPositive 
                                                    ? "<strong>High Risk Warning:</strong> The drug's high lipophilicity (LogP 4.1) is now strongly correlated with the patient's positive symptoms. Prediction suggests the molecule is crossing cellular membranes too effectively, causing <strong>myotoxicity (muscle poisoning)</strong> due to accumulation in fatty tissue." 
                                                    : "Current drug profile shows a moderate-high lipophilic tendency. This allows efficient absorption but requires active monitoring of muscle markers to ensure the drug doesn't saturate tissue beyond metabolic clearance capacity."
                                                }
                                            </p>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <!-- Molecular Simulation Grid -->
                            <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200 ${allPositive ? 'border-rose-500 bg-rose-50/20' : ''}">
                                <div class="flex justify-between items-end mb-8">
                                    <div>
                                        <h3 class="text-[10px] font-black ${allPositive ? 'text-rose-600' : 'text-slate-400'} uppercase tracking-widest">Molecular Surveillance ${allPositive ? ' - CRITICAL STATUS' : ''}</h3>
                                        <p class="text-[9px] font-bold text-slate-400 uppercase mt-1">Simulating cellular interaction vs symptoms</p>
                                    </div>
                                    ${allPositive ? '<span class="text-[9px] font-black text-white bg-rose-600 px-3 py-1 rounded-full animate-pulse uppercase">Danger Zone Simulation</span>' : ''}
                                </div>
                                
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                                    ${p.prescription.map(rx => `
                                        <div class="space-y-4 p-6 rounded-[2rem] border relative overflow-hidden transition-all duration-700 ${allPositive ? 'bg-white border-rose-300 shadow-xl scale-[1.02]' : 'bg-slate-50 border-slate-100'}">
                                            <div class="flex justify-center mb-6">
                                                <svg width="120" height="120" viewBox="0 0 100 100">
                                                    <path d="M20,50 Q20,20 50,20 Q80,20 80,50 Q80,80 50,80 Q20,80 20,50" fill="none" stroke="${allPositive ? '#FDA4AF' : '#E2E8F0'}" stroke-width="4" stroke-dasharray="2,2"/>
                                                    <g transform="${allPositive ? 'translate(15, -15) rotate(15, 50, 50)' : 'translate(0, 0)'}" style="transition: transform 1s cubic-bezier(0.4, 0, 0.2, 1)">
                                                        <circle cx="50" cy="50" r="14" fill="${allPositive ? '#E11D48' : '#4F46E5'}" class="${allPositive ? 'animate-pulse' : ''}" />
                                                    </g>
                                                </svg>
                                            </div>
                                            <div>
                                                <h4 class="font-black text-sm ${allPositive ? 'text-rose-700' : 'text-slate-800'}">${rx.name}</h4>
                                                <div class="mt-4 space-y-2">
                                                    <div class="bg-white p-3 rounded-xl border border-slate-100">
                                                        <p class="text-[9px] font-black uppercase ${allPositive ? 'text-rose-500' : 'text-indigo-400'}">${allPositive ? 'BINDING FAILURE' : 'How it fits'}</p>
                                                        <p class="text-[10px] font-medium text-slate-600 leading-tight">
                                                            ${allPositive ? 'Molecular rejection due to acute drug-organ stress.' : rx.simpleDocking}
                                                        </p>
                                                    </div>
                                                    <div class="p-3 rounded-xl border ${allPositive ? 'bg-rose-600 text-white border-rose-700' : 'bg-white border-slate-100 text-slate-600'}">
                                                        <p class="text-[9px] font-black uppercase ${allPositive ? 'text-white/70' : 'text-rose-400'}">Safety Check</p>
                                                        <p class="text-[10px] font-bold leading-tight">
                                                            ${allPositive ? rx.criticalRisk : rx.simpleToxicity}
                                                        </p>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>

                            <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200">
                                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-6">Live Symptom Dashboard</h3>
                                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                    ${Object.entries(p.symptoms).map(([k, v]) => `
                                        <div class="p-4 rounded-2xl border ${v === null ? 'bg-slate-50 border-slate-100' : v ? 'bg-rose-50 border-rose-100' : 'bg-emerald-50 border-emerald-100'}">
                                            <p class="text-[9px] font-black text-slate-400 uppercase mb-1">${k}</p>
                                            <p class="text-xs font-black ${v === null ? 'text-slate-300' : v ? 'text-rose-600' : 'text-emerald-600'}">
                                                ${v === null ? 'Waiting...' : v ? 'Positive' : 'Clear'}
                                            </p>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>
                        </div>

                        <div class="space-y-8">
                            <div class="bg-white p-8 rounded-[2.5rem] border border-slate-200">
                                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-6">Live Adherence</h3>
                                <div class="flex flex-wrap gap-2 justify-center">
                                    ${p.adherence.map(d => `
                                        <div class="w-8 h-8 rounded-full flex items-center justify-center border ${
                                            d.status === 'taken' ? 'bg-emerald-500 text-white' : 
                                            d.status === 'missed' ? 'bg-rose-500 text-white' : 
                                            'bg-slate-50 border-slate-100'
                                        }">
                                            <span class="text-[9px] font-black">${d.day[0]}</span>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>

                            ${allPositive ? `
                                <div class="bg-rose-600 text-white p-8 rounded-[2.5rem] shadow-2xl danger-pulse">
                                    <h3 class="text-[10px] font-black text-rose-200 uppercase tracking-widest mb-4">Urgent Medical Alert</h3>
                                    <p class="text-sm font-black mb-4">CRITICAL: Severe symptoms detected.</p>
                                    <p class="text-[10px] font-medium text-rose-100 italic">Docking failure simulated. Syncing with Raghu.</p>
                                </div>
                            ` : ''}

                            <div class="bg-slate-900 text-white p-8 rounded-[2.5rem]">
                                <h3 class="text-[10px] font-black text-indigo-400 uppercase tracking-widest mb-6">Pharmacy Fulfillment</h3>
                                <div class="space-y-6">
                                    ${p.prescription.map(rx => `
                                        <div class="space-y-2">
                                            <div class="flex justify-between items-center">
                                                <span class="text-xs font-black">${rx.name}</span>
                                                <span class="text-[9px] px-2 py-0.5 rounded-full ${rx.restocked ? 'bg-emerald-500/20 text-emerald-400' : 'bg-rose-500/20 text-rose-400'} font-black">
                                                    ${rx.restocked ? 'In Stock' : 'Out'}
                                                </span>
                                            </div>
                                            <div class="h-1 bg-white/10 rounded-full overflow-hidden">
                                                <div class="h-full ${rx.delivered ? 'bg-indigo-400' : 'bg-rose-400/50'}" style="width: ${rx.delivered ? '100%' : '20%'}"></div>
                                            </div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>`;
            }

            async notifyEmergency() {
                this.state.patients[0].emergencyContact.notified = true;
                this.state.patients[0].notes.push({
                    type: 'text',
                    content: 'CRITICAL ALERT: Physician triggered emergency notification to Raghu.',
                    sender: 'Doctor (SOS)',
                    timestamp: new Date().toISOString()
                });
                await this.save();
                this.render();
            }

            async addSharedNote() {
                const el = document.getElementById('shared-note-input');
                if (!el.value.trim()) return;
                this.state.patients[0].notes.push({
                    type: 'text',
                    content: el.value.trim(),
                    sender: this.userRole.charAt(0).toUpperCase() + this.userRole.slice(1),
                    timestamp: new Date().toISOString()
                });
                el.value = '';
                await this.save();
                this.render();
            }

            unlockRoom() {
                const val = document.getElementById('room-pin').value;
                if (val === '4444') { this.unlockedRoom = true; this.render(); }
                else { document.getElementById('room-pin').classList.add('shake'); setTimeout(()=>document.getElementById('room-pin').classList.remove('shake'),400); }
            }

            renderDoctor(root) {
                root.innerHTML = `
                <div class="max-w-4xl mx-auto p-6 space-y-6 text-center">
                    <div class="bg-white p-16 rounded-[2.5rem] border border-slate-200 shadow-xl">
                        <div class="w-20 h-20 bg-indigo-50 text-indigo-600 rounded-3xl flex items-center justify-center mx-auto mb-8">
                            <i data-lucide="shield-check" class="w-10 h-10"></i>
                        </div>
                        <h2 class="text-3xl font-black mb-4 tracking-tighter">Physician Portal</h2>
                        <p class="text-slate-500 mb-10 max-w-sm mx-auto">Review patient symptoms and manage risk notifications for emergency contacts.</p>
                        <button onclick="app.view='shared-room'; app.render();" class="w-full max-w-sm bg-indigo-600 text-white py-5 rounded-2xl font-black uppercase tracking-widest">Enter Shared Room Hub</button>
                    </div>
                </div>`;
            }
        }

        window.app = new MediRoom();
    </script>
</body>
</html>
