<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Poker Life Tracker Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .card { background-color: white; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); padding: 1.5rem; margin-bottom: 1.5rem; }
        .btn { display: inline-flex; align-items: center; justify-content: center; padding: 0.5rem 1rem; border-radius: 0.5rem; font-weight: 600; transition: all 0.2s; cursor: pointer; }
        .btn:disabled { opacity: 0.5; cursor: not-allowed; }
        .btn-sm { padding: 0.25rem 0.75rem; font-size: 0.875rem; }
        .btn-xs { padding: 0.125rem 0.5rem; font-size: 0.75rem; }
        .btn-primary { background-color: #4f46e5; color: white; }
        .btn-primary:hover:not(:disabled) { background-color: #4338ca; }
        .btn-secondary { background-color: #e5e7eb; color: #374151; }
        .btn-secondary:hover:not(:disabled) { background-color: #d1d5db; }
        .btn-danger { background-color: #ef4444; color: white; }
        .btn-danger:hover:not(:disabled) { background-color: #dc2626; }
        .input-field, .textarea-field, .select-field { width: 100%; padding: 0.5rem 0.75rem; border-radius: 0.5rem; border: 1px solid #d1d5db; }
        .nav-btn { color: #4b5563; border-bottom: 2px solid transparent; padding: 0.75rem; white-space: nowrap; }
        .nav-btn.active { color: #4f46e5; border-bottom-color: #4f46e5; font-weight: 700; }
        .page { display: none; }
        .page.active { display: block; }
        .profit { color: #16a34a; }
        .loss { color: #dc2626; }
        .modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background-color: rgba(0, 0, 0, 0.6); display: flex; justify-content: center; align-items: center; z-index: 50; }
        .modal-content { background-color: white; padding: 2rem; border-radius: 0.75rem; max-width: 90%; width: 600px; max-height: 80vh; overflow-y: auto; }
        .prose { max-width: none; }
        .prose strong { color: #374151; }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #4f46e5;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        .btn .loader { width: 20px; height: 20px; border-width: 2px; margin-left: 0.5rem; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .toast {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background-color: #2d3748;
            color: white;
            padding: 1rem 1.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
            z-index: 100;
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 0.3s ease, transform 0.3s ease;
        }
        .toast.show {
            opacity: 1;
            transform: translateY(0);
        }
        .collapsible-content {
            max-height: 1000px; /* Large enough to not clip content */
            overflow: hidden;
            transition: max-height 0.5s ease-in-out;
        }
        .collapsible-content.collapsed {
            max-height: 0;
        }
         .month-header {
            cursor: pointer;
            background-color: #f9fafb;
        }
        .month-header:hover {
            background-color: #f3f4f6;
        }
        .session-row.hidden {
            display: none;
        }
        .chevron-icon {
            transition: transform 0.3s ease;
        }
        .chevron-icon.rotate-180 {
            transform: rotate(180deg);
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div id="loading-overlay" class="modal-overlay">
        <div class="text-center">
            <div class="loader"></div>
            <p class="text-white font-semibold">Cargando y autenticando...</p>
        </div>
    </div>

    <div class="container mx-auto p-4 md:p-6">
        <header class="text-center mb-6">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-900">Poker Life Tracker Pro</h1>
            <p class="text-gray-600 mt-2">Tu centro de mando para el rendimiento dentro y fuera de las mesas.</p>
        </header>

        <nav class="bg-white rounded-lg shadow-sm mb-6 p-2 flex justify-center flex-wrap">
            <button id="nav-dashboard" class="nav-btn" onclick="navigateTo('dashboard')">Dashboard</button>
            <button id="nav-today" class="nav-btn" onclick="navigateTo('today')">Sesión Actual</button>
            <button id="nav-bankroll" class="nav-btn" onclick="navigateTo('bankroll')">Finanzas</button>
            <button id="nav-history" class="nav-btn" onclick="navigateTo('history')">Historial</button>
            <button id="nav-study" class="nav-btn" onclick="navigateTo('study')">Estudio de Manos</button>
            <button id="nav-journal" class="nav-btn" onclick="navigateTo('journal')">Bitácora</button>
            <button id="nav-habits" class="nav-btn" onclick="navigateTo('habits')">Rutina / Hábitos</button>
            <button id="nav-activity-log" class="nav-btn" onclick="navigateTo('activity-log')">Log de Actividad</button>
        </nav>

        <main>
            <div id="page-dashboard" class="page"></div>
            <div id="page-today" class="page"></div>
            <div id="page-bankroll" class="page"></div>
            <div id="page-history" class="page"></div>
            <div id="page-study" class="page"></div>
            <div id="page-journal" class="page"></div>
            <div id="page-habits" class="page"></div>
            <div id="page-activity-log" class="page"></div>
        </main>
    </div>

    <div id="generic-modal" class="modal-overlay hidden">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-4">
                <h2 id="modal-title" class="text-2xl font-bold text-indigo-700"></h2>
                <button onclick="closeModal()" class="text-gray-500 hover:text-gray-800 text-2xl font-bold">&times;</button>
            </div>
            <div id="modal-body" class="prose"></div>
        </div>
    </div>
    <div id="toast-notification" class="toast"></div>
    <input type="file" id="csv-import-input" class="hidden" accept=".csv" onchange="handleFileUpload(event)">

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, deleteDoc, onSnapshot, collection, addDoc, query, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- GLOBAL STATE, CONFIG & FIREBASE ---
        let db, auth, userId;
        let currentPage = 'dashboard';
        let currentSession = { tournaments: [] };
        let sessionHistory = [];
        let journalPosts = [];
        let studyHands = [];
        let habitData = {};
        let activityLog = [];
        let defaultBuyIns = [5.50, 10.80, 16.50, 22.00, 25.00, 33.00, 55.00, 109.00];
        let chartInstance = null;
        let bankrollDistributionChart = null;
        let dashboardChart = null;
        let fileHandler = { imageBase64: null, mimeType: null, target: null };
        let platformNames = {
            ggpoker: "GGPoker", pokerstars: "PokerStars", winamax: "Winamax",
            bodog: "Bodog", acr: "ACR", muchbetter: "MuchBetter", binance: "Binance"
        };
        let bankrollBalances = {};
        let withdrawals = [];
        let customHabits = [];
        let unsubscribes = []; // To store listener unsubscribes
        let mockDataLoaded = false;
        let currentHistoryFilter = 'all';

        // --- INITIALIZATION ---
        window.addEventListener('DOMContentLoaded', () => {
            populatePageHTML();
            setupEventListeners();
            initializeAppAndAuth();
        });
        
        async function initializeAppAndAuth() {
            try {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
                if (!firebaseConfig) {
                    showError("La configuración de Firebase no está disponible. La aplicación no puede funcionar sin conexión a la base de datos.");
                    return;
                }
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("Authenticated User ID:", userId);
                        await loadAllDataFromFirestore();
                        document.getElementById('loading-overlay').classList.add('hidden');
                    } else {
                        try {
                           const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                           if (initialAuthToken) {
                             await signInWithCustomToken(auth, initialAuthToken);
                           } else {
                             await signInAnonymously(auth);
                           }
                        } catch (error) {
                            console.error("Anonymous sign-in failed:", error);
                            showError("No se pudo iniciar sesión anónimamente. Se requiere autenticación para guardar datos.");
                        }
                    }
                });
            } catch (error) {
                console.error("Firebase initialization failed:", error);
                showError("Error al inicializar la base de datos. Por favor, recarga la página.");
            }
        }

        async function loadAllDataFromFirestore() {
            unsubscribes.forEach(unsub => unsub());
            unsubscribes = [];

            const collectionsToSync = {
                'config': (data) => {
                    mockDataLoaded = data.mockDataLoaded || false;
                    if (data.defaultBuyIns) defaultBuyIns = data.defaultBuyIns;
                    if (data.bankrollBalances) bankrollBalances = data.bankrollBalances;
                    if (data.platformNames) platformNames = data.platformNames;
                    if(data.customHabits) customHabits = data.customHabits;
                    else {
                        customHabits = [
                            { id: 'water', name: 'Vasos de Agua en Sesión', type: 'counter' },
                            { id: 'hands', name: 'Manos Estudiadas', type: 'counter' },
                            { id: 'meditation', name: 'Meditación', type: 'checkbox' },
                            { id: 'exercise', name: 'Actividad Física', type: 'checkbox' },
                            { id: 'warmup', name: 'Calentamiento Pre-Sesión', type: 'checkbox' }
                        ];
                    }
                },
                'currentSession': (data) => {
                    currentSession = data || {};
                    if (!Array.isArray(currentSession.tournaments)) {
                        currentSession.tournaments = [];
                    }
                },
                'withdrawals': (data) => {
                    withdrawals = data ? data.history || [] : [];
                },
                'sessionHistory': (doc) => {
                    sessionHistory = doc.history || [];
                     if (!mockDataLoaded) {
                        loadMockDataIfEmpty();
                        mockDataLoaded = true; 
                        saveData.config();
                    }
                },
                'studyHands': (data) => {
                    studyHands = data ? data.hands || [] : [];
                },
                'journalPosts': (data) => {
                    journalPosts = data ? data.posts || [] : [];
                },
                'habitData': (data) => {
                    habitData = data || {};
                },
                'activityLog': (data) => {
                    activityLog = data ? data.log || [] : [];
                }
            };
            
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
            
            const promises = Object.keys(collectionsToSync).map(collectionName => {
                return new Promise((resolve, reject) => {
                    const docRef = doc(db, 'artifacts', appId, 'users', userId, collectionName, 'data');
                    const unsub = onSnapshot(docRef, (docSnap) => {
                        console.log(`Received update for ${collectionName}`);
                        collectionsToSync[collectionName](docSnap.data() || {});
                        resolve();
                    }, (error) => {
                        console.error(`Error listening to ${collectionName}:`, error);
                        reject(error);
                    });
                    unsubscribes.push(unsub);
                });
            });

            await Promise.all(promises);
            navigateTo('dashboard');
        }

        // --- GENERIC HELPER FUNCTIONS ---
        function formatCurrency(num) { return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(num); }
        function openModal(title, content, actions = '') {
            document.getElementById('modal-title').innerHTML = title;
            document.getElementById('modal-body').innerHTML = content + `<div class="mt-6 flex justify-end gap-3">${actions}</div>`;
            document.getElementById('generic-modal').classList.remove('hidden');
        }
        function closeModal() { document.getElementById('generic-modal').classList.add('hidden'); }
        function showError(message) {
             openModal("Error", `<p class="text-red-600">${message}</p>`, `<button class="btn btn-secondary" onclick="closeModal()">Cerrar</button>`);
        }
        function showToast(message) {
            const toast = document.getElementById('toast-notification');
            toast.textContent = message;
            toast.classList.add('show');
            setTimeout(() => {
                toast.classList.remove('show');
            }, 3000);
        }
        
        // --- DATA SAVING (CENTRALIZED) ---
        const saveData = {
            config: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'config', 'data');
                await setDoc(docRef, { defaultBuyIns, bankrollBalances, platformNames, mockDataLoaded, customHabits }, { merge: true });
            },
            currentSession: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'currentSession', 'data');
                await setDoc(docRef, currentSession);
            },
            withdrawals: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'withdrawals', 'data');
                await setDoc(docRef, { history: withdrawals });
            },
            sessionHistory: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'sessionHistory', 'data');
                await setDoc(docRef, { history: sessionHistory });
            },
            studyHands: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'studyHands', 'data');
                await setDoc(docRef, { hands: studyHands });
            },
            journalPosts: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'journalPosts', 'data');
                await setDoc(docRef, { posts: journalPosts });
            },
            habitData: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'habitData', 'data');
                await setDoc(docRef, habitData, { merge: true });
            },
            activityLog: async () => {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-poker-tracker';
                const docRef = doc(db, 'artifacts', appId, 'users', userId, 'activityLog', 'data');
                await setDoc(docRef, { log: activityLog });
            }
        };

        // --- UI & NAVIGATION ---
        function populatePageHTML() {
            const main = document.querySelector('main');
            main.innerHTML = `
            <div id="page-dashboard" class="page"></div>
            <div id="page-today" class="page"></div>
            <div id="page-bankroll" class="page"></div>
            <div id="page-history" class="page"></div>
            <div id="page-study" class="page"></div>
            <div id="page-journal" class="page"></div>
            <div id="page-habits" class="page"></div>
            <div id="page-activity-log" class="page"></div>`;
        }
        function setupEventListeners() {
            document.body.addEventListener('click', (event) => {
                const target = event.target;
                const navButton = target.closest('.nav-btn');
                if (navButton) {
                    navigateTo(navButton.id.split('-')[1]);
                    return;
                }

                if (target.closest('[onclick="closeModal()"]')) {
                    closeModal();
                    return;
                }
                
                const button = target.closest('button');
                if (button) {
                     const actionMap = {
                        'add-custom-buyin-btn': addCustomBuyIn,
                        'reconcile-session-btn': openReconciliationModal,
                        'save-balances-btn': saveAndRecalculateBankroll,
                        'add-platform-btn': openAddPlatformModal,
                        'manual-entry-btn': openManualEntryModal,
                        'import-csv-btn': () => document.getElementById('csv-import-input').click(),
                        'export-csv-btn': exportHistoryToCSV,
                        'add-habit-btn': openAddHabitModal,
                        'save-platform-btn': saveNewPlatform,
                        'save-habit-btn': saveNewHabit,
                        'save-manual-entry-btn': saveManualEntry,
                        'execute-reconciliation-btn': executeReconciliation,
                        'add-withdrawal-btn': addWithdrawal,
                        'confirm-withdrawal-btn': confirmWithdrawal,
                    };
                    if (actionMap[button.id]) {
                        actionMap[button.id](event);
                        return;
                    }
                     if (button.closest('[onclick^="executeWithdrawal"]')) {
                        const onclickAttr = button.getAttribute('onclick');
                        eval(onclickAttr);
                    }
                }
                
                if(target.closest('#history-filter-buttons')) handleHistoryFilter(event);
                if(target.closest('[data-collapsible-trigger]')) handleHistoryPageClick(event);
                if(target.closest('.month-header')) handleHistoryPageClick(event);
                if(target.closest('[id^=tab-]')) {
                     const tabName = target.id.split('-')[1];
                     switchFinanceTab(tabName);
                }
                 if (target.closest('[onclick^="updateHabit"]')) {
                    const onclickAttr = target.closest('[onclick^="updateHabit"]').getAttribute('onclick');
                    eval(onclickAttr);
                }
            });

             document.body.addEventListener('submit', (event) => {
                 event.preventDefault();
                 switch(event.target.id){
                     case 'add-study-hand-form': handleStudyFormSubmit(event); break;
                     case 'add-journal-form': handleJournalFormSubmit(event); break;
                 }
             });
        }
        function navigateTo(pageId, fullRender = true) {
            currentPage = pageId;
            document.querySelectorAll('.page').forEach(p => p.classList.toggle('active', p.id === `page-${pageId}`));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.toggle('active', b.id.split('-')[1] === pageId));
            
            const renderFunctions = {
                'dashboard': renderDashboardPage,
                'today': renderTodayPage,
                'bankroll': renderBankrollPage,
                'history': renderHistoryPage,
                'study': renderStudyPage,
                'journal': renderJournalPage,
                'habits': renderHabitsPage,
                'activity-log': renderActivityLogPage,
            };
            const pageElement = document.getElementById(`page-${pageId}`);
            if(renderFunctions[pageId] && (pageElement.innerHTML.trim() === '' || fullRender)) {
                renderFunctions[pageId]();
            } else if (renderFunctions[pageId]) {
                // For pages with data that might update, re-render the dynamic parts
                if (pageId === 'dashboard') renderDashboardPage();
                else if (pageId === 'bankroll') renderBankrollPage();
                else if (pageId === 'history') renderHistoryPage();
            }
        }
        
        // --- Activity Log ---
        async function logActivity(type, description, date = null) {
            activityLog.unshift({
                date: date || new Date().toLocaleString('es-ES'),
                type,
                description
            });
            await saveData.activityLog();
        }

        function renderActivityLogPage(){
            const page = document.getElementById('page-activity-log');
            if(!page) return;
            page.innerHTML = `<div class="card"><h2 class="text-2xl font-semibold mb-4">Log de Actividad Financiera</h2><div class="overflow-x-auto"><table class="min-w-full bg-white text-sm"><thead class="bg-gray-200"><tr><th class="py-2 px-3 text-left">Fecha</th><th class="py-2 px-3 text-left">Tipo</th><th class="py-2 px-3 text-left">Descripción</th></tr></thead><tbody id="activity-log-body"></tbody></table></div></div>`;
            const tbody = document.getElementById('activity-log-body');
            tbody.innerHTML = activityLog.length === 0 ? `<tr><td colspan="3" class="text-center p-4 text-gray-500">No hay actividad registrada.</td></tr>` : activityLog.map(log => {
                return `<tr class="border-b">
                    <td class="py-2 px-3 whitespace-nowrap">${log.date}</td>
                    <td class="py-2 px-3"><span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-blue-100 text-blue-800">${log.type}</span></td>
                    <td class="py-2 px-3">${log.description}</td>
                </tr>`;
            }).join('');
        }

        // --- BANKROLL & PLATFORMS ---
        function renderBankrollPage() {
            const page = document.getElementById('page-bankroll');
             if(!page) return;
            page.innerHTML = `<div class="grid grid-cols-1 lg:grid-cols-2 gap-6"><div class="card"><h2 class="text-xl font-semibold mb-4">Saldos por Plataforma</h2><div id="bankroll-balances-form" class="space-y-3"></div><div class="mt-4"><button id="add-platform-btn" class="btn btn-secondary w-full">+ Agregar Plataforma</button></div><button id="save-balances-btn" class="btn btn-primary w-full mt-4">Guardar Saldos</button></div><div class="card"><h2 class="text-xl font-semibold mb-4 text-center">Gestión de Bankroll</h2><div class="bg-indigo-50 p-6 rounded-lg text-center mb-6"><p class="text-lg font-medium text-indigo-800">Bankroll Total</p><p id="bankroll-total-display" class="text-5xl font-bold text-indigo-600">$0.00</p></div><div id="bankroll-management-tiers" class="space-y-4"></div></div></div><div class="card"><h2 class="text-xl font-semibold mb-4">Movimientos Financieros</h2><div class="border-b border-gray-200"><nav class="-mb-px flex space-x-8" aria-label="Tabs"><button id="tab-withdrawals" class="border-indigo-500 text-indigo-600 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">Retiros</button><button id="tab-adjustments" class="border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">Ajustes</button></nav></div><div id="finance-tab-content" class="mt-4"></div></div><div class="grid grid-cols-1 md:grid-cols-2 gap-6"><div class="card"><h2 class="text-xl font-semibold mb-4">Distribución del Bankroll</h2><canvas id="bankroll-distribution-chart"></canvas></div><div class="card"><h2 class="text-xl font-semibold mb-4">Resumen de Retiros Mensuales</h2><div id="withdrawals-summary-container"></div></div></div>`;
            const form = document.getElementById('bankroll-balances-form');
            if(form) {
                form.innerHTML = Object.keys(platformNames).map(key => {
                    const value = bankrollBalances[key] || 0;
                    return `<div><label for="balance-${key}" class="block text-sm font-medium text-gray-700">${platformNames[key]}</label><input type="number" id="balance-${key}" data-platform="${key}" class="input-field mt-1" value="${value}" placeholder="0.00"></div>`;
                }).join('');
            }
            updateBankrollDisplay();
            switchFinanceTab('withdrawals'); 
            renderBankrollDistributionChart();
            renderWithdrawalsSummary();
        }

        function openAddPlatformModal() {
            const modalContent = `
                <form id="add-platform-form" class="space-y-4">
                    <div>
                        <label for="platform-name" class="block text-sm font-medium text-gray-700">Nombre de la Nueva Plataforma</label>
                        <input type="text" id="platform-name" class="input-field" required>
                    </div>
                </form>
            `;
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button type="button" id="save-platform-btn" class="btn btn-primary">Guardar Plataforma</button>`;
            openModal('Agregar Nueva Plataforma', modalContent, actions);
        }

        async function saveNewPlatform() {
            const nameInput = document.getElementById('platform-name');
            const name = nameInput.value.trim();
            if (!name) {
                showError("El nombre de la plataforma no puede estar vacío.");
                return;
            }

            const key = name.toLowerCase().replace(/\s+/g, '');
            if (platformNames[key] || Object.values(platformNames).some(val => val.toLowerCase() === name.toLowerCase())) {
                showError("La plataforma ya existe.");
                return;
            }

            platformNames[key] = name;
            await saveData.config();
            await logActivity('Configuración', `Nueva plataforma agregada: ${name}`);
            showToast(`Plataforma "${name}" agregada.`);
            closeModal();
        }

        
        async function saveAndRecalculateBankroll(event) {
            const button = event.target.closest('button');
            const originalText = button.innerHTML;
            button.innerHTML = `<div class="loader"></div>`;
            button.disabled = true;

            const oldBalances = { ...bankrollBalances };
            const newBalances = {};
            document.querySelectorAll('#bankroll-balances-form input').forEach(input => {
                newBalances[input.dataset.platform] = parseFloat(input.value) || 0;
            });
            
            for (const key in newBalances) {
                const oldVal = oldBalances[key] || 0;
                const newVal = newBalances[key];
                if (oldVal !== newVal) {
                    await logActivity('Ajuste de Bankroll', `Saldo en ${platformNames[key]} cambiado de ${formatCurrency(oldVal)} a ${formatCurrency(newVal)}.`);
                }
            }

            bankrollBalances = newBalances;
            await saveData.config();
            updateBankrollDisplay();
            showToast("Saldos guardados.");
            button.innerHTML = originalText;
            button.disabled = false;
        }
        function updateBankrollDisplay() {
            const totalBankroll = Object.values(bankrollBalances).reduce((sum, val) => sum + (val || 0), 0);
            if(document.getElementById('bankroll-total-display')) {
                document.getElementById('bankroll-total-display').textContent = formatCurrency(totalBankroll);
            }
            const tiersContainer = document.getElementById('bankroll-management-tiers');
            if(tiersContainer) {
                const tiers = { "Juego Conservador (1000 BIs)": 1000, "Juego Estándar (700 BIs)": 700, "Juego Agresivo (500 BIs)": 500 };
                tiersContainer.innerHTML = Object.keys(tiers).map(label => {
                    const abi = totalBankroll > 0 ? totalBankroll / tiers[label] : 0;
                    return `<div class="flex justify-between items-center py-2 border-t"><span class="font-medium">${label}:</span><span class="font-bold text-lg text-gray-900">ABI de ${formatCurrency(abi)}</span></div>`;
                }).join('');
            }
        }
        function addWithdrawal() {
            const modalContent = `
                <div class="space-y-4">
                    <div>
                        <label for="withdrawal-amount" class="block text-sm font-medium text-gray-700">Monto</label>
                        <input type="number" id="withdrawal-amount" class="input-field" placeholder="200.00" step="0.01">
                    </div>
                    <div>
                        <label for="withdrawal-desc" class="block text-sm font-medium text-gray-700">Descripción</label>
                        <input type="text" id="withdrawal-desc" class="input-field" placeholder="Gastos personales">
                    </div>
                </div>`;
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button id="confirm-withdrawal-btn" class="btn btn-primary">Siguiente</button>`;
            openModal('Registrar Retiro', modalContent, actions);
        }

        function confirmWithdrawal() {
            const amount = parseFloat(document.getElementById('withdrawal-amount').value);
            const description = document.getElementById('withdrawal-desc').value.trim();
            if (isNaN(amount) || amount <= 0 || !description) { showError("Ingresa un monto y descripción válidos."); return; }
            
            let modalContent = `<p class="mb-4">Selecciona la plataforma de origen para el retiro de ${formatCurrency(amount)}:</p><div class="grid grid-cols-2 gap-2">`;
            modalContent += Object.keys(platformNames).map(key => {
                const balance = bankrollBalances[key] || 0;
                const disabled = balance < amount ? 'disabled' : '';
                return `<button class="btn btn-secondary" onclick="executeWithdrawal('${key}', ${amount}, '${description.replace(/'/g, "\\'")}')" ${disabled}>${platformNames[key]} <span class="text-xs ml-2">(${formatCurrency(balance)})</span></button>`;
            }).join('');
            modalContent += '</div>';
            
            openModal('Seleccionar Origen del Retiro', modalContent, `<button class="btn btn-secondary" onclick="addWithdrawal()">Atrás</button>`);
        }

        async function executeWithdrawal(platformKey, amount, description) {
            bankrollBalances[platformKey] = (bankrollBalances[platformKey] || 0) - amount;
            withdrawals.unshift({ date: new Date().toLocaleDateString('es-ES'), amount, description, source: platformNames[platformKey] });
            
            await logActivity('Retiro', `Retiro de ${formatCurrency(amount)} desde ${platformNames[platformKey]} (${description}).`);
            await Promise.all([saveData.config(), saveData.withdrawals()]);
            closeModal();
            showToast(`Retiro registrado.`);
        }
        function renderWithdrawalsHistory() {
            const container = document.getElementById('finance-tab-content');
            if(!container) return;
            let content = `<div class="max-h-96 overflow-y-auto border rounded-lg"><table class="min-w-full text-sm"><thead class="bg-gray-100"><tr><th class="py-2 px-3 text-left">Fecha</th><th class="py-2 px-3 text-left">Monto</th><th class="py-2 px-3 text-left">Descripción</th><th class="py-2 px-3 text-left">Origen</th></tr></thead><tbody>`;
            content += withdrawals.length === 0 ? `<tr><td colspan="4" class="text-center p-4 text-gray-500">No hay retiros.</td></tr>` : withdrawals.map(w => `<tr class="border-b"><td class="py-2 px-3">${w.date}</td><td class="py-2 px-3">${formatCurrency(w.amount)}</td><td class="py-2 px-3">${w.description}</td><td class="py-2 px-3 font-semibold">${w.source || ''}</td></tr>`).join('');
            content += '</tbody></table></div>';
            container.innerHTML = content;
        }
        
        function renderBankrollAdjustmentsLog() {
            const container = document.getElementById('finance-tab-content');
            if(!container) return;
            const adjustments = activityLog.filter(log => log.type === 'Ajuste de Bankroll');
            let content = `<div class="max-h-96 overflow-y-auto border rounded-lg"><table class="min-w-full text-sm"><thead class="bg-gray-100"><tr><th class="py-2 px-3 text-left">Fecha</th><th class="py-2 px-3 text-left">Descripción</th></tr></thead><tbody>`;
            content += adjustments.length === 0 ? `<tr><td colspan="2" class="text-center p-4 text-gray-500">No hay ajustes manuales.</td></tr>` : adjustments.map(log => `<tr class="border-b"><td class="py-2 px-3">${log.date}</td><td class="py-2 px-3">${log.description}</td></tr>`).join('');
            content += '</tbody></table></div>';
            container.innerHTML = content;
        }

        function switchFinanceTab(tabName) {
            const tabs = ['withdrawals', 'adjustments'];
            tabs.forEach(tab => {
                const tabEl = document.getElementById(`tab-${tab}`);
                tabEl.classList.remove('border-indigo-500', 'text-indigo-600');
                tabEl.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
            });
            const activeTab = document.getElementById(`tab-${tabName}`);
            activeTab.classList.add('border-indigo-500', 'text-indigo-600');
            activeTab.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');

            if (tabName === 'withdrawals') {
                renderWithdrawalsHistory();
            } else {
                renderBankrollAdjustmentsLog();
            }
        }
        
        // --- Bankroll Charts ---
        function renderBankrollDistributionChart() {
            const canvas = document.getElementById('bankroll-distribution-chart');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            if(bankrollDistributionChart) {
                bankrollDistributionChart.destroy();
            }
            
            const labels = Object.keys(platformNames).map(key => platformNames[key]);
            const data = Object.keys(platformNames).map(key => bankrollBalances[key] || 0);

            const filteredLabels = [];
            const filteredData = [];
            data.forEach((value, index) => {
                if (value > 0) {
                    filteredLabels.push(labels[index]);
                    filteredData.push(value);
                }
            });
            
            if (filteredData.length === 0) {
                ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
                ctx.font = "16px Inter";
                ctx.fillStyle = "#a0aec0";
                ctx.textAlign = "center";
                ctx.fillText("No hay saldos para mostrar.", ctx.canvas.width / 2, 50);
                return;
            }

            bankrollDistributionChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: filteredLabels,
                    datasets: [{
                        label: 'Distribución de Bankroll',
                        data: filteredData,
                        backgroundColor: [ '#4f46e5', '#818cf8', '#a78bfa', '#c4b5fd', '#f59e0b', '#fbbf24', '#fcd34d', '#fef3c7', '#10b981', '#34d399', '#6ee7b7', '#a7f3d0' ],
                        hoverOffset: 4
                    }]
                },
                options: { responsive: true, plugins: { legend: { position: 'top' } } }
            });
        }
        
        function renderWithdrawalsSummary() {
            const container = document.getElementById('withdrawals-summary-container');
            if (!container) return;

            const summary = withdrawals.reduce((acc, w) => {
                const [day, month, year] = w.date.split('/');
                const key = `${year}-${month}`;
                if (!acc[key]) {
                    acc[key] = 0;
                }
                acc[key] += w.amount;
                return acc;
            }, {});

            const sortedMonths = Object.keys(summary).sort().reverse();
            
            if (sortedMonths.length === 0) {
                 container.innerHTML = `<p class="text-center text-gray-500">No hay retiros registrados.</p>`;
                 return;
            }

            let tableHtml = `<div class="max-h-60 overflow-y-auto border rounded-lg"><table class="min-w-full text-sm">
                <thead class="bg-gray-100"><tr><th class="py-2 px-3 text-left">Mes</th><th class="py-2 px-3 text-left">Total Retirado</th></tr></thead><tbody>`;

            const monthNames = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];
            
            sortedMonths.forEach(key => {
                const [year, month] = key.split('-');
                const monthName = `${monthNames[parseInt(month, 10) - 1]} ${year}`;
                tableHtml += `<tr class="border-b"><td class="py-2 px-3">${monthName}</td><td class="py-2 px-3 font-semibold">${formatCurrency(summary[key])}</td></tr>`;
            });

            tableHtml += `</tbody></table></div>`;
            container.innerHTML = tableHtml;
        }

        // --- SESIÓN DE HOY ---
        async function addCustomBuyIn() { 
            const input = document.getElementById('custom-buyin-input'); 
            const newBuyIn = parseFloat(input.value); 
            if (newBuyIn > 0 && !defaultBuyIns.includes(newBuyIn)) { 
                defaultBuyIns.push(newBuyIn); 
                defaultBuyIns.sort((a, b) => a - b); 
                input.value = ''; 
                renderDefaultTournaments(); 
                await saveData.config();
            } else { 
                showError("Buy-in inválido o ya existente."); 
            } 
        }
        function renderTodayPage() { 
            const page = document.getElementById('page-today');
            page.innerHTML = `<div class="grid grid-cols-1 md:grid-cols-3 gap-6"><div class="md:col-span-1"><div class="card"><h2 class="text-xl font-semibold mb-4">Registro Rápido</h2><div class="mb-4 p-3 bg-gray-50 rounded-lg border"><h3 class="font-semibold text-gray-700 mb-2">Agregar Buy-in Personalizado</h3><div class="flex gap-2"><input type="number" id="custom-buyin-input" class="input-field" placeholder="Ej: 109" step="0.01" min="0"><button id="add-custom-buyin-btn" class="btn btn-secondary">Añadir</button></div></div><div id="default-tournaments-list" class="grid grid-cols-2 sm:grid-cols-3 gap-2"></div></div><div class="card"><h2 class="text-xl font-semibold mb-4">Finalizar Sesión</h2><p class="text-sm text-gray-600 mb-4">Calcula resultados y actualiza tu bankroll y tu historial de forma automática.</p><button id="reconcile-session-btn" class="btn btn-primary w-full">💾 Finalizar y Conciliar Sesión</button></div></div><div class="md:col-span-2"><div class="card"><h2 class="text-xl font-semibold mb-4">Resumen de la Sesión</h2><div id="summary" class="space-y-2 mb-4"><div class="flex justify-between items-center py-2 border-b"><span class="font-medium">Total de Entradas (E):</span><span id="total-entries-count" class="font-bold text-lg">0</span></div><div class="flex justify-between items-center py-2 border-b"><span class="font-medium">Total de Re-entries (RE):</span><span id="total-reentries-count" class="font-bold text-lg">0</span></div><div class="flex justify-between items-center py-2"><span class="font-semibold text-xl">GASTO TOTAL:</span><span id="grand-total" class="font-bold text-2xl text-blue-600">$0.00</span></div><div class="flex justify-between items-center py-2 border-t mt-2 pt-2"><span class="font-medium">Inicio de Sesión:</span><span id="session-start-time" class="font-bold">--</span></div></div></div><div class="card"><h2 class="text-xl font-semibold mb-4">Torneos de la Sesión</h2><div id="tournaments-list" class="space-y-3"></div></div></div></div>`;
            renderDefaultTournaments(); 
            renderTournaments(); 
            updateSummary(); 
        }
        function updateSummary() { 
            const totalEntries = currentSession.tournaments.reduce((s, t) => s + t.entries, 0); 
            const totalReentries = currentSession.tournaments.reduce((s, t) => s + t.reentries, 0); 
            document.getElementById('total-entries-count').textContent = totalEntries; 
            document.getElementById('total-reentries-count').textContent = totalReentries; 
            document.getElementById('grand-total').textContent = formatCurrency(currentSession.tournaments.reduce((s, t) => s + (t.buyin * (t.entries + t.reentries)), 0)); 
             const startTimeEl = document.getElementById('session-start-time');
            if (currentSession.startTime) {
                startTimeEl.textContent = new Date(currentSession.startTime).toLocaleString('es-ES');
            } else {
                startTimeEl.textContent = '--';
            }
        }
        function renderTournaments() { 
            const listEl = document.getElementById('tournaments-list'); 
            listEl.innerHTML = currentSession.tournaments.length === 0 ? `<p class="text-gray-500 text-center py-4">Agrega torneos desde el panel de Registro Rápido.</p>` : [...currentSession.tournaments].sort((a,b) => b.buyin - a.buyin).map(t => `<div class="p-4 border rounded-lg bg-gray-50"><div class="flex flex-wrap items-center justify-between gap-x-4 gap-y-2"><p class="font-bold text-lg text-indigo-700 grow">Torneo: ${formatCurrency(t.buyin)}</p><div class="flex items-center gap-2"><span class="text-sm font-medium">Entradas:</span><button class="btn btn-secondary btn-xs" data-action="adjust-count" data-buyin="${t.buyin}" data-type="entries" data-amount="-1">-</button><span class="font-semibold w-6 text-center">${t.entries}</span><button class="btn btn-secondary btn-xs" data-action="adjust-count" data-buyin="${t.buyin}" data-type="entries" data-amount="1">+</button></div><div class="flex items-center gap-2"><span class="text-sm font-medium">Re-entries:</span><button class="btn btn-secondary btn-xs" data-action="adjust-count" data-buyin="${t.buyin}" data-type="reentries" data-amount="-1">-</button><span class="font-semibold w-6 text-center">${t.reentries}</span><button class="btn btn-secondary btn-xs" data-action="adjust-count" data-buyin="${t.buyin}" data-type="reentries" data-amount="1">+</button></div></div></div>`).join(''); 
        }
        function renderDefaultTournaments() { 
            const listEl = document.getElementById('default-tournaments-list'); 
            listEl.innerHTML = defaultBuyIns.map(b => `<div class="flex flex-col items-center justify-center bg-gray-100 p-2 rounded-md border"><span class="font-bold text-gray-800">${formatCurrency(b)}</span><div class="flex gap-1 mt-1"><button class="btn btn-primary btn-xs" data-action="add-bullet" data-buyin="${b}" data-is-reentry="false">+E</button><button class="btn btn-secondary btn-xs" data-action="add-bullet" data-buyin="${b}" data-is-reentry="true">+RE</button></div></div>`).join(''); 
        }
        async function addBullet(buyin, isReentry) { 
            if (!currentSession.startTime) {
                currentSession.startTime = new Date().toISOString();
            }
            let t = currentSession.tournaments.find(t => t.buyin === buyin); 
            if (t) { 
                isReentry ? t.reentries++ : t.entries++; 
            } else { 
                currentSession.tournaments.push({ buyin, entries: isReentry ? 0 : 1, reentries: isReentry ? 1 : 0 }); 
            } 
            renderTournaments(); 
            updateSummary();
            await saveData.currentSession();
        }
        async function adjustCount(buyin, type, amount) { 
            let t = currentSession.tournaments.find(t => t.buyin === buyin); 
            if (t) { t[type] += amount; 
                if (t.entries <= 0 && t.reentries <= 0) currentSession.tournaments = currentSession.tournaments.filter(tour => tour.buyin !== buyin); 
                renderTournaments(); 
                updateSummary();
                await saveData.currentSession();
            } 
        }

        // --- CONCILIACIÓN DE SESIÓN & HISTORIAL ---
        function openReconciliationModal() {
            const investment = currentSession.tournaments.reduce((s, t) => s + (t.buyin * (t.entries + t.reentries)), 0);
            if(investment === 0) { showError("No hay torneos en la sesión para conciliar."); return; }
            
            let modalContent = `<div class="space-y-4">
                <div>
                    <label for="session-date" class="block text-sm font-medium text-gray-700">Fecha de la Sesión</label>
                    <input type="date" id="session-date" class="input-field mt-1">
                </div>
                <div class="p-3 bg-gray-100 rounded-lg text-center">Inversión de la Sesión: <strong class="text-lg">${formatCurrency(investment)}</strong></div>
                <div><p class="mb-2 text-sm font-medium text-gray-700">Ingresa los saldos finales de las plataformas donde jugaste.</p></div>
                <div id="reconciliation-form" class="space-y-3 max-h-60 overflow-y-auto pr-2">`;
            
            modalContent += Object.keys(platformNames).map(key => {
                const balance = bankrollBalances[key] || 0;
                return `<div><label for="recon-balance-${key}" class="block text-sm font-medium text-gray-700">${platformNames[key]}</label><input type="number" id="recon-balance-${key}" data-platform="${key}" class="input-field mt-1" value="${balance.toFixed(2)}" placeholder="${balance.toFixed(2)}"></div>`;
            }).join('');
            
            modalContent += `</div></div>`;
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button id="execute-reconciliation-btn" class="btn btn-primary">Calcular y Guardar</button>`;
            
            openModal('Finalizar y Conciliar Sesión', modalContent, actions);
            document.getElementById('session-date').value = new Date().toISOString().split('T')[0];
        }
        
        async function executeReconciliation() {
            const investment = currentSession.tournaments.reduce((s, t) => s + (t.buyin * (t.entries + t.reentries)), 0);
            const initialTotalBankroll = Object.values(bankrollBalances).reduce((sum, val) => sum + (val || 0), 0);
            const newBalances = { ...bankrollBalances };
            let finalTotalBankroll = 0;
            
            document.querySelectorAll('#reconciliation-form input').forEach(input => {
                const platformKey = input.dataset.platform;
                const value = input.value;
                newBalances[platformKey] = parseFloat(value) || 0;
                finalTotalBankroll += newBalances[platformKey];
            });

            const profitLoss = finalTotalBankroll - (initialTotalBankroll - investment);
            bankrollBalances = newBalances;
            
            const totalEntries = currentSession.tournaments.reduce((acc, t) => acc + t.entries, 0);
            const totalReentries = currentSession.tournaments.reduce((acc, t) => acc + t.reentries, 0);
            
            const dateValue = document.getElementById('session-date').value;
            const sessionDate = new Date(dateValue + 'T00:00:00'); // Avoid timezone issues

            const sessionEntry = {
                date: sessionDate.toLocaleDateString('es-ES', { day: '2-digit', month: '2-digit', year: 'numeric'}),
                startTime: currentSession.startTime,
                investment,
                tournaments: totalEntries + totalReentries,
                totalEntries: totalEntries,
                totalReentries: totalReentries,
                profitLoss,
                finalBankroll: finalTotalBankroll
            };
            sessionHistory.push(sessionEntry);
            await logActivity('Sesión Finalizada', `P/L de ${formatCurrency(profitLoss)}`);
            
            currentSession = { tournaments: [], startTime: null };
            await sortAndSaveHistory();
            await Promise.all([saveData.config(), saveData.currentSession()]);
            showToast("¡Sesión conciliada y guardada con éxito!");
            closeModal();
            navigateTo('history');
        }

        function openManualEntryModal() {
            const modalContent = `
                <form id="manual-entry-form" class="space-y-4">
                    <div>
                        <label for="manual-date" class="block text-sm font-medium text-gray-700">Fecha</label>
                        <input type="date" id="manual-date" class="input-field" required>
                    </div>
                    <div>
                        <label for="manual-investment" class="block text-sm font-medium text-gray-700">Inversión ($)</label>
                        <input type="number" step="0.01" id="manual-investment" class="input-field" required>
                    </div>
                    <div>
                        <label for="manual-entries" class="block text-sm font-medium text-gray-700">Entradas</label>
                        <input type="number" id="manual-entries" class="input-field" required>
                    </div>
                    <div>
                        <label for="manual-reentries" class="block text-sm font-medium text-gray-700">Reentradas</label>
                        <input type="number" id="manual-reentries" class="input-field" required>
                    </div>
                     <div>
                        <label for="manual-pl" class="block text-sm font-medium text-gray-700">Profit/Loss ($)</label>
                        <input type="number" step="0.01" id="manual-pl" class="input-field" required>
                    </div>
                    <div>
                        <label for="manual-bankroll" class="block text-sm font-medium text-gray-700">Bankroll Final ($)</label>
                        <input type="number" step="0.01" id="manual-bankroll" class="input-field" required>
                    </div>
                </form>
            `;
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button id="save-manual-entry-btn" class="btn btn-primary">Guardar Sesión</button>`;
            openModal('Agregar Sesión Manualmente', modalContent, actions);
        }

        async function saveManualEntry() {
            const dateValue = document.getElementById('manual-date').value;
            const investment = parseFloat(document.getElementById('manual-investment').value);
            const totalEntries = parseInt(document.getElementById('manual-entries').value);
            const totalReentries = parseInt(document.getElementById('manual-reentries').value);
            const profitLoss = parseFloat(document.getElementById('manual-pl').value);
            const finalBankroll = parseFloat(document.getElementById('manual-bankroll').value);

            if (!dateValue || isNaN(investment) || isNaN(totalEntries) || isNaN(totalReentries) || isNaN(profitLoss) || isNaN(finalBankroll)) {
                showError("Por favor, completa todos los campos con valores válidos.");
                return;
            }
            
            const sessionDate = new Date(dateValue + 'T00:00:00');
            const formattedDate = sessionDate.toLocaleDateString('es-ES', { day: '2-digit', month: '2-digit', year: 'numeric'})

            sessionHistory.push({
                date: formattedDate,
                investment,
                tournaments: totalEntries + totalReentries,
                totalEntries: totalEntries,
                totalReentries: totalReentries,
                profitLoss,
                finalBankroll
            });

            await logActivity('Entrada Manual', `Sesión agregada para el ${formattedDate} con P/L de ${formatCurrency(profitLoss)}`);
            await sortAndSaveHistory();
            showToast("Sesión manual agregada.");
            closeModal();
        }

        async function sortAndSaveHistory() {
            sessionHistory.sort((a, b) => {
                const dateA = new Date(a.date.split('/').reverse().join('-'));
                const dateB = new Date(b.date.split('/').reverse().join('-'));
                return dateA - dateB;
            });
            await saveData.sessionHistory();
        }

        function renderHistoryPage() { 
            const page = document.getElementById('page-history');
             if(!page) return;
            page.innerHTML = `<div class="card"><div class="flex justify-between items-center mb-4 flex-wrap gap-4"><div><h2 class="text-2xl font-semibold">Historial</h2></div><div class="flex gap-2 flex-wrap" id="history-filter-buttons"><button data-filter="all" class="btn btn-sm btn-primary">Todos</button><button data-filter="30" class="btn btn-sm btn-secondary">Últimos 30 Días</button><button data-filter="90" class="btn btn-sm btn-secondary">Últimos 90 Días</button><button data-filter="year" class="btn btn-sm btn-secondary">Este Año</button></div></div><div class="flex justify-end mb-4 gap-2 flex-wrap"><button id="import-csv-btn" class="btn btn-secondary btn-sm">Importar desde CSV</button><button id="export-csv-btn" class="btn btn-secondary btn-sm">Exportar a CSV</button><button id="manual-entry-btn" class="btn btn-primary btn-sm">Agregar Entrada Manual</button></div><div class="card" id="history-table-card"><div class="flex justify-between items-center mb-4 cursor-pointer" data-collapsible-trigger="table"><h3 class="text-xl font-semibold">Sesiones Detalladas</h3><svg id="toggle-table-icon" class="chevron-icon w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg></div><div id="collapsible-table-content" class="collapsible-content"><div class="overflow-x-auto" id="history-table-container"></div></div></div><div class="card mt-6" id="history-chart-card"><div class="flex justify-between items-center mb-4 cursor-pointer" data-collapsible-trigger="chart"><h2 class="text-xl font-semibold">Resultados del Período Seleccionado</h2><svg id="toggle-chart-icon" class="chevron-icon w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg></div><div id="collapsible-chart-content" class="collapsible-content"><canvas id="history-chart"></canvas></div></div>`;
            applyHistoryFilter(true);
        }
        
        function handleHistoryPageClick(event) {
            const collapsibleTrigger = event.target.closest('[data-collapsible-trigger]');
            if (collapsibleTrigger) {
                const triggerType = collapsibleTrigger.dataset.collapsibleTrigger;
                toggleCollapsible(`collapsible-${triggerType}-content`, `toggle-${triggerType}-icon`);
                return;
            }
            
            const monthHeader = event.target.closest('.month-header');
            if (monthHeader) {
                const monthKey = monthHeader.dataset.monthKey;
                toggleMonth(monthKey);
            }
        }


        function handleHistoryFilter(event) {
            if(!event.target.closest('button')) return;
            const button = event.target.closest('button');
            currentHistoryFilter = button.dataset.filter;
            
            document.querySelectorAll('#history-filter-buttons button').forEach(btn => {
                btn.classList.remove('btn-primary');
                btn.classList.add('btn-secondary');
            });
            button.classList.add('btn-primary');
            button.classList.remove('btn-secondary');

            applyHistoryFilter(true); // Force collapse on filter change
        }

        function applyHistoryFilter(forceCollapse = false) {
            let filteredHistory = sessionHistory;
            const now = new Date();

            if (currentHistoryFilter !== 'all') {
                let startDate = new Date();
                if (currentHistoryFilter === '30') {
                    startDate.setDate(now.getDate() - 30);
                } else if (currentHistoryFilter === '90') {
                     startDate.setDate(now.getDate() - 90);
                } else if (currentHistoryFilter === 'year') {
                    startDate = new Date(now.getFullYear(), 0, 1);
                }
                
                filteredHistory = sessionHistory.filter(s => {
                    const sessionDate = new Date(s.date.split('/').reverse().join('-'));
                    return sessionDate >= startDate;
                });
            }
            
            renderHistoryTable(filteredHistory, forceCollapse);
            renderHistoryChart(filteredHistory);
        }
        
        function renderHistoryTable(data, forceCollapse) {
            const historyData = data || sessionHistory;
            const container = document.getElementById('history-table-container');

            if (!container) return;

            if (historyData.length === 0) {
                 container.innerHTML = `<p class="text-center text-gray-500 py-4">No hay sesiones para este período.</p>`;
                 return;
            }

            const groupedByMonth = historyData.reduce((acc, session) => {
                const [day, month, year] = session.date.split('/');
                const monthKey = `${year}-${month}`;
                if (!acc[monthKey]) {
                    acc[monthKey] = [];
                }
                acc[monthKey].push(session);
                return acc;
            }, {});

            const monthNames = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];
            let tableHtml = '<table class="min-w-full bg-white text-sm">';
            
            Object.keys(groupedByMonth).sort().reverse().forEach((monthKey, index) => {
                const sessionsInMonth = groupedByMonth[monthKey];
                const [year, month] = monthKey.split('-');
                const monthName = `${monthNames[parseInt(month,10)-1]} ${year}`;
                const monthPL = sessionsInMonth.reduce((acc, s) => acc + s.profitLoss, 0);

                const isCollapsed = forceCollapse || index > 0;

                tableHtml += `<thead class="bg-gray-50"><tr class="month-header border-b-2 border-gray-200" data-month-key="${monthKey}"><th colspan="8" class="py-2 px-3 text-left font-semibold text-gray-700">${monthName} - P/L: <span class="${monthPL >= 0 ? 'profit' : 'loss'}">${formatCurrency(monthPL)}</span></th><th class="py-2 px-3 text-right"><svg id="chevron-${monthKey}" class="chevron-icon w-5 h-5 ${isCollapsed ? '' : 'rotate-180'}" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg></th></tr></thead>`;
                tableHtml += `<tbody id="month-body-${monthKey}" class="${isCollapsed ? 'hidden' : ''}">`;
                
                [...sessionsInMonth].reverse().forEach(s => {
                    const roiPercent = s.investment > 0 ? (s.profitLoss / s.investment) * 100 : 0;
                    const originalIndex = sessionHistory.findIndex(hist => hist.date === s.date && hist.investment === s.investment && hist.finalBankroll === s.finalBankroll);
                    
                    tableHtml += `<tr class="session-row border-b"><td class="py-2 px-3">${s.date}</td><td class="py-2 px-3">${formatCurrency(s.investment)}</td><td class="py-2 px-3">${s.tournaments}</td><td class="py-2 px-3">${s.totalEntries !== undefined ? s.totalEntries : s.tournaments}</td><td class="py-2 px-3">${s.totalReentries !== undefined ? s.totalReentries : '-'}</td><td class="py-2 px-3 font-bold ${s.profitLoss>=0?'profit':'loss'}">${formatCurrency(s.profitLoss)}</td><td class="py-2 px-3 font-semibold ${roiPercent>=0?'profit':'loss'}">${roiPercent.toFixed(2)}%</td><td class="py-2 px-3 font-bold">${formatCurrency(s.finalBankroll)}</td><td class="py-2 px-3"><button class="btn btn-danger btn-xs" onclick="confirmDeleteHistoryEntry(${originalIndex})">X</button></td></tr>`;
                });
                tableHtml += `</tbody>`;
            });

            tableHtml += '</table>';
            container.innerHTML = tableHtml;
        }

        function confirmDeleteHistoryEntry(index) {
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button class="btn btn-danger" onclick="deleteHistoryEntry(${index})">Eliminar</button>`;
            openModal("Confirmar Eliminación", "<p>¿Estás seguro de que quieres eliminar esta entrada del historial? Esta acción no se puede deshacer.</p>", actions);
        }
        async function deleteHistoryEntry(index) { 
            const deletedSession = sessionHistory[index];
            await logActivity('Historial', `Sesión del ${deletedSession.date} eliminada.`);
            sessionHistory.splice(index, 1); 
            await saveData.sessionHistory();
            closeModal();
        } 
        function renderHistoryChart(data) { 
            const historyData = data || sessionHistory;
            const ctx = document.getElementById('history-chart').getContext('2d'); 
            if(chartInstance){chartInstance.destroy();} 
            chartInstance=new Chart(ctx,{type:'line',data:{labels:historyData.map(s=>s.date),datasets:[{label:'Bankroll',data:historyData.map(s=>s.finalBankroll),borderColor:'#4f46e5',yAxisID:'yBank'},{label:'P/L Sesión',data:historyData.map(s=>s.profitLoss),backgroundColor:historyData.map(s=>s.profitLoss>=0?'rgba(22, 163, 74, 0.5)':'rgba(220, 38, 38, 0.5)'),type:'bar',yAxisID:'yPL'}]},options:{scales:{yBank:{position:'left',title:{display:true,text:'Bankroll Total ($)'}},yPL:{position:'right',title:{display:true,text:'P/L Sesión ($)'},grid:{drawOnChartArea:false}}}}});
        }
        
        // --- CSV Import/Export ---
        function exportHistoryToCSV() {
            let csvContent = "data:text/csv;charset=utf-8,Fecha,Inversion,Torneos (Balas),Entradas,Reentradas,P/L Sesion,Bankroll Final\n";
            sessionHistory.forEach(session => {
                const row = [
                    session.date,
                    session.investment,
                    session.tournaments,
                    session.totalEntries,
                    session.totalReentries,
                    session.profitLoss,
                    session.finalBankroll
                ].join(",");
                csvContent += row + "\n";
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "historial_poker.csv");
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }
        
        function handleFileUpload(event) {
            const file = event.target.files[0];
            if (!file) { return; }
            const reader = new FileReader();
            reader.onload = async function(e) {
                try {
                    const contents = e.target.result;
                    const lines = contents.split(/[\r\n]+/).filter(line => line.trim() !== '');
                    if (lines.length < 2) {
                        showError("El archivo CSV está vacío o no tiene datos.");
                        return;
                    }
                    const headers = lines.shift().toLowerCase().split(',').map(h => h.trim().replace(/"/g, ''));
                    
                    const requiredHeaders = ['fecha', 'inversion', 'p/l sesion', 'bankroll final'];
                    const hasRequired = requiredHeaders.every(rh => headers.includes(rh));
                    const hasBullets = headers.includes('torneos (balas)');
                    const hasDetail = headers.includes('entradas') && headers.includes('reentradas');

                    if (!hasRequired || (!hasBullets && !hasDetail)) {
                        showError("El archivo CSV no tiene las columnas requeridas. Asegúrate de que incluya: Fecha, Inversion, P/L Sesion, Bankroll Final, y ('Torneos (Balas)' o 'Entradas' y 'Reentradas').");
                        return;
                    }

                    const newSessions = lines.map(line => {
                        const values = line.split(',');
                        const sessionData = {};
                        headers.forEach((header, index) => {
                            sessionData[header] = values[index] ? values[index].trim().replace(/"/g, '') : '';
                        });

                        const investment = parseFloat(sessionData.inversion);
                        const profitLoss = parseFloat(sessionData['p/l sesion']);
                        const finalBankroll = parseFloat(sessionData['bankroll final']);

                        let totalEntries, totalReentries, tournaments;

                        if (hasDetail) {
                            totalEntries = parseInt(sessionData.entradas);
                            totalReentries = parseInt(sessionData.reentradas);
                            tournaments = totalEntries + totalReentries;
                        } else {
                            tournaments = parseInt(sessionData['torneos (balas)']);
                            totalEntries = tournaments;
                            totalReentries = 0;
                        }

                        return {
                            date: sessionData.fecha,
                            investment: isNaN(investment) ? 0 : investment,
                            tournaments: isNaN(tournaments) ? 0 : tournaments,
                            totalEntries: isNaN(totalEntries) ? 0 : totalEntries,
                            totalReentries: isNaN(totalReentries) ? 0 : totalReentries,
                            profitLoss: isNaN(profitLoss) ? 0 : profitLoss,
                            finalBankroll: isNaN(finalBankroll) ? 0 : finalBankroll
                        };
                    }).filter(s => s.date && !isNaN(s.investment)); 

                    if(newSessions.length > 0) {
                        sessionHistory.push(...newSessions);
                        await logActivity('Importación', `Se importaron ${newSessions.length} sesiones desde CSV.`);
                        await sortAndSaveHistory();
                        showToast(`${newSessions.length} sesiones importadas.`);
                    } else {
                        showError("No se encontraron sesiones válidas para importar en el archivo.");
                    }
                } catch (error) {
                    showError("Ocurrió un error al procesar el archivo CSV. Asegúrate de que el formato es correcto.");
                    console.error("CSV Import Error:", error);
                }
            };
            reader.readAsText(file);
            event.target.value = '';
        }

        // --- ESTUDIO, BITÁCORA Y HÁBITOS ---
        function handleFileChange(event, type) {
            const file = event.target.files[0];
            const preview = document.getElementById(`${type}-image-preview`);
            if(!file) { fileHandler = { imageBase64: null, mimeType: null, target: null }; preview.classList.add('hidden'); return; }
            fileHandler.mimeType = file.type;
            fileHandler.target = type;
            const reader = new FileReader();
            reader.onload = (e) => { fileHandler.imageBase64 = e.target.result; preview.src = fileHandler.imageBase64; preview.classList.remove('hidden'); };
            reader.readAsDataURL(file);
        }
        async function handleStudyFormSubmit(e) { 
            e.preventDefault(); 
            const title = document.getElementById('study-hand-title').value; 
            const notes = document.getElementById('study-hand-notes').value; 
            if(!title || !notes || !fileHandler.imageBase64 || fileHandler.target !== 'study') { showError("Todos los campos y la imagen son obligatorios."); return; } 
            studyHands.unshift({ id: Date.now(), date: new Date().toLocaleString('es-ES'), title, notes, image: fileHandler.imageBase64, mimeType: fileHandler.mimeType }); 
            await saveData.studyHands();
            e.target.reset(); 
            document.getElementById('study-image-preview').classList.add('hidden'); 
            fileHandler = { imageBase64: null, mimeType: null, target: null }; 
        }
        function renderStudyPage() { 
            const page = document.getElementById('page-study');
            page.innerHTML = `
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-1">
                    <div class="card">
                        <h2 class="text-xl font-semibold mb-4">Agregar Mano para Estudiar</h2>
                        <form id="add-study-hand-form" class="space-y-4">
                            <div>
                                <label for="study-hand-title" class="block text-sm font-medium text-gray-700">Título / Situación</label>
                                <input type="text" id="study-hand-title" class="input-field" placeholder="Ej: BvB 3bet pot con AKo" required>
                            </div>
                            <div>
                                <label for="study-hand-notes" class="block text-sm font-medium text-gray-700">Notas / Dudas</label>
                                <textarea id="study-hand-notes" class="textarea-field" rows="4" placeholder="¿Debí apostar en el river? ¿Qué rangos paga el villano?" required></textarea>
                            </div>
                            <div>
                                <label for="study-hand-image" class="block text-sm font-medium text-gray-700">Captura de la Mano</label>
                                <input type="file" id="study-hand-image" onchange="handleFileChange(event, 'study')" accept="image/*" class="input-field" required>
                                <img id="study-image-preview" class="mt-4 rounded-lg hidden w-full" alt="Previsualización de la mano">
                            </div>
                            <button type="submit" class="btn btn-primary w-full">Guardar y Analizar</button>
                        </form>
                    </div>
                </div>
                <div class="lg:col-span-2">
                    <div class="card">
                        <h2 class="text-xl font-semibold mb-4">Manos Guardadas</h2>
                        <div id="study-hands-feed" class="space-y-4 max-h-[70vh] overflow-y-auto pr-2">
                            </div>
                    </div>
                </div>
            </div>`;

            const feed = document.getElementById('study-hands-feed'); 
            feed.innerHTML = studyHands.length === 0 ? `<p class="text-center p-4 text-gray-500">Aún no has guardado manos.</p>` : studyHands.map(h => `<div class="bg-gray-50 p-4 rounded-lg border"><div class="flex justify-between items-start"><div><h3 class="font-bold text-lg text-indigo-700">${h.title}</h3><p class="text-sm text-gray-500 mb-2">${h.date}</p></div><button class="btn btn-danger btn-sm" onclick="confirmDeleteStudyHand(${h.id})">X</button></div><div class="md:flex md:gap-4"><img src="${h.image}" class="rounded-lg w-full md:w-1/2 mb-4 md:mb-0"><div class="w-full md:w-1/2"><h4 class="font-semibold mb-1">Notas:</h4><p class="text-sm bg-white p-2 rounded whitespace-pre-wrap border">${h.notes}</p><button class="btn btn-primary btn-sm w-full mt-3" onclick="analyzeStudyHand(${h.id})">Analizar con IA</button></div></div></div>`).join(''); 
        }
        function confirmDeleteStudyHand(id) {
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button class="btn btn-danger" onclick="deleteStudyHand(${id})">Eliminar</button>`;
            openModal("Confirmar Eliminación", "<p>¿Seguro que quieres eliminar esta mano de estudio?</p>", actions);
        }
        async function deleteStudyHand(id) { 
            studyHands = studyHands.filter(h => h.id !== id); 
            await saveData.studyHands();
            closeModal();
        } 
        async function handleJournalFormSubmit(e) { 
            e.preventDefault(); 
            const text = document.getElementById('journal-text-input').value; 
            if(!text) { showError("La descripción es obligatoria."); return; } 
            journalPosts.unshift({ id: Date.now(), date: new Date().toLocaleString('es-ES'), text, image: (fileHandler.target === 'journal' ? fileHandler.imageBase64 : null) }); 
            await saveData.journalPosts();
            e.target.reset(); 
            document.getElementById('journal-image-preview').classList.add('hidden'); 
            fileHandler = { imageBase64: null, mimeType: null, target: null }; 
        }
        function renderJournalPage() { 
            const page = document.getElementById('page-journal');
            page.innerHTML = `
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-1">
                    <div class="card">
                        <h2 class="text-xl font-semibold mb-4">Nueva Entrada en Bitácora</h2>
                        <form id="add-journal-form" class="space-y-4">
                            <div>
                                <label for="journal-text-input" class="block text-sm font-medium text-gray-700">Describe tu sesión, pensamientos, estado de ánimo...</label>
                                <textarea id="journal-text-input" class="textarea-field" rows="6" placeholder="Hoy me sentí muy concentrado. Noté que los rivales estaban 3-beteando mucho..." required></textarea>
                            </div>
                            <div>
                                <label for="journal-image-input" class="block text-sm font-medium text-gray-700">Adjuntar Imagen (Opcional)</label>
                                <input type="file" id="journal-image-input" onchange="handleFileChange(event, 'journal')" accept="image/*" class="input-field">
                                <img id="journal-image-preview" class="mt-4 rounded-lg hidden w-full" alt="Previsualización de imagen">
                            </div>
                            <button type="submit" class="btn btn-primary w-full">Guardar Entrada</button>
                        </form>
                    </div>
                </div>
                <div class="lg:col-span-2">
                    <div class="card">
                        <h2 class="text-xl font-semibold mb-4">Historial de Bitácora</h2>
                        <div id="journal-feed" class="space-y-4 max-h-[70vh] overflow-y-auto pr-2">
                            </div>
                    </div>
                </div>
            </div>`;
            const feed = document.getElementById('journal-feed'); 
            feed.innerHTML = journalPosts.length === 0 ? `<p class="text-center p-4 text-gray-500">No hay entradas.</p>` : journalPosts.map(p => `<div class="bg-gray-50 p-4 rounded-lg border"><p class="text-sm text-gray-500 mb-2">${p.date}</p><p class="mb-4 whitespace-pre-wrap">${p.text}</p>${p.image ? `<img src="${p.image}" class="rounded-lg max-h-96 w-auto mx-auto mb-4">` : ''}<div class="text-right"><button class="btn btn-danger btn-sm" onclick="confirmDeleteJournalPost(${p.id})">X</button></div></div>`).join(''); 
        }
        function confirmDeleteJournalPost(id) {
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button class="btn btn-danger" onclick="deleteJournalPost(${id})">Eliminar</button>`;
            openModal("Confirmar Eliminación", "<p>¿Seguro que quieres eliminar esta entrada de la bitácora?</p>", actions);
        }
        async function deleteJournalPost(id) { 
            journalPosts = journalPosts.filter(p => p.id !== id); 
            await saveData.journalPosts();
            closeModal();
        } 
        function getTodayDateString() { return new Date().toISOString().split('T')[0]; }
        function renderHabitsPage() { 
            const page = document.getElementById('page-habits');
            page.innerHTML = `
            <div class="card max-w-2xl mx-auto">
                <div class="text-center mb-6">
                    <h2 class="text-2xl font-semibold">Rutina y Hábitos Diarios</h2>
                    <p id="habits-date" class="text-gray-500"></p>
                </div>
                <div id="habits-container" class="space-y-6">
                    </div>
                <div class="mt-6 border-t pt-4">
                     <button id="add-habit-btn" class="btn btn-secondary w-full">Personalizar y Agregar Hábitos</button>
                </div>
            </div>`;
            const today = getTodayDateString(); 
            document.getElementById('habits-date').textContent = new Date().toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }); 
            const habitsContainer = document.getElementById('habits-container');
            const todayHabits = habitData[today] || {};
            
            habitsContainer.innerHTML = customHabits.map(habit => {
                let controlHtml = '';
                if (habit.type === 'counter') {
                    const count = todayHabits[habit.id] || 0;
                    controlHtml = `<div class="flex items-center gap-4"><button class="btn btn-secondary" onclick="updateHabit('${habit.id}', -1)">-</button><span id="habit-${habit.id}-count" class="font-bold text-xl w-8 text-center">${count}</span><button class="btn btn-secondary" onclick="updateHabit('${habit.id}', 1)">+</button></div>`;
                } else {
                    const isChecked = todayHabits[habit.id] || false;
                    controlHtml = `<input type="checkbox" id="habit-${habit.id}-check" class="h-6 w-6 rounded text-indigo-600 focus:ring-indigo-500" onchange="updateHabit('${habit.id}', this.checked)" ${isChecked ? 'checked' : ''}>`;
                }
                return `<div class="flex justify-between items-center bg-gray-50 p-4 rounded-lg"><label class="font-semibold text-lg">${habit.name}</label>${controlHtml}</div>`;
            }).join('');
        }
        async function updateHabit(habitId, value) { 
            const today = getTodayDateString(); 
            if(!habitData[today]) habitData[today] = {}; 

            const habitDef = customHabits.find(h => h.id === habitId);
            if (habitDef.type === 'checkbox') {
                 habitData[today][habitId] = value;
            } else { // counter
                 habitData[today][habitId] = (habitData[today][habitId] || 0) + value;
                 if(habitData[today][habitId] < 0) habitData[today][habitId] = 0; 
            }
            
            renderHabitsPage(); 
            await saveData.habitData();
        }
         function openAddHabitModal() {
            const modalContent = `
                <form id="add-habit-form" class="space-y-4">
                    <div>
                        <label for="habit-name" class="block text-sm font-medium text-gray-700">Nombre del Hábito</label>
                        <input type="text" id="habit-name" class="input-field" required>
                    </div>
                    <div>
                        <label for="habit-type" class="block text-sm font-medium text-gray-700">Tipo de Seguimiento</label>
                        <select id="habit-type" class="select-field">
                            <option value="counter">Contador (Ej: Vasos de agua)</option>
                            <option value="checkbox">Checklist (Ej: Meditación)</option>
                        </select>
                    </div>
                </form>
            `;
            const actions = `<button class="btn btn-secondary" onclick="closeModal()">Cancelar</button><button id="save-habit-btn" class="btn btn-primary">Guardar Hábito</button>`;
            openModal('Agregar Nuevo Hábito de Rutina', modalContent, actions);
        }
        async function saveNewHabit() {
            const name = document.getElementById('habit-name').value.trim();
            const type = document.getElementById('habit-type').value;

            if (!name) {
                showError("El nombre del hábito no puede estar vacío.");
                return;
            }
            const id = name.toLowerCase().replace(/\s+/g, '_');
            if (customHabits.some(h => h.id === id)) {
                showError("Ya existe un hábito con un nombre similar.");
                return;
            }
            
            customHabits.push({ id, name, type });
            await saveData.config();
            renderHabitsPage();
            showToast("Hábito agregado a tu rutina.");
            closeModal();
        }

        // --- GEMINI AI ANALYSIS ---
        async function analyzeStudyHand(id) {
            const hand = studyHands.find(h => h.id === id);
            if (!hand) {
                showError("No se pudo encontrar la mano para analizar.");
                return;
            }

            const modalTitle = "Análisis con IA de Gemini";
            let modalContent = `<div class="flex justify-center items-center flex-col gap-4"><div class="loader"></div><p class="text-center font-semibold">Analizando la mano, por favor espera...</p></div>`;
            openModal(modalTitle, modalContent, '');

            try {
                const prompt = `
                    Eres un coach de poker experto y de clase mundial. Analiza la siguiente mano de poker.
                    El jugador te ha dado un título y notas sobre la mano, junto con una captura de pantalla.
                    
                    Título: "${hand.title}"
                    Notas del jugador: "${hand.notes}"

                    Basándote en la información y la imagen, proporciona un análisis completo. Cubre los siguientes puntos:
                    1.  **Análisis Pre-flop:** ¿La decisión fue correcta? ¿Cuáles son los rangos de manos probables para cada jugador involucrado?
                    2.  **Análisis Post-flop (calle por calle, si aplica):** Evalúa las decisiones en el flop, turn y river. ¿El tamaño de las apuestas fue el correcto? ¿Qué nos dice la acción sobre las manos de los oponentes?
                    3.  **Líneas Alternativas:** ¿Qué otras formas se podría haber jugado la mano? ¿Cuáles son los pros y contras de cada una?
                    4.  **Conclusión y Lecciones Clave:** Resume el análisis y ofrece 2-3 puntos clave que el jugador debería recordar para futuras manos similares.

                    Sé claro, conciso y utiliza un lenguaje que un jugador de poker intermedio pueda entender. Formatea tu respuesta usando Markdown.
                `;
                
                const base64ImageData = hand.image.split(',')[1];
                
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [
                    { text: prompt },
                    { inlineData: { mimeType: hand.mimeType, data: base64ImageData } }
                ]});

                const payload = { contents: chatHistory };
                const apiKey = ""; // <-- REEMPLAZA ESTO CON TU API KEY DE GOOGLE AI STUDIO
                
                if (!apiKey) {
                    showError("La API Key de Gemini no ha sido configurada. Por favor, añádela en el código para activar el análisis con IA.");
                    return;
                }

                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-pro-vision:generateContent?key=${apiKey}`;

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: chatHistory })
                });
                
                if (!response.ok) {
                    const errorBody = await response.json();
                    console.error("API Error Body:", errorBody);
                    throw new Error(`Error en la API: ${errorBody.error?.message || response.statusText}`);
                }
                
                const result = await response.json();

                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content.parts.length > 0) {
                    const analysisText = result.candidates[0].content.parts[0].text;
                    const formattedHtml = analysisText
                        .replace(/### (.*)/g, '<h3 class="text-lg font-bold mt-4 mb-2">$1</h3>')
                        .replace(/## (.*)/g, '<h2 class="text-xl font-bold mt-6 mb-3">$1</h2>')
                        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
                        .replace(/\* (.*)/g, '<li class="ml-5 list-disc">$1</li>')
                        .replace(/(\r\n|\n|\r)/g, '<br>');
                        
                    openModal(modalTitle, `<div class="text-left prose">${formattedHtml}</div>`, `<button class="btn btn-secondary" onclick="closeModal()">Cerrar</button>`);
                } else {
                    console.error("Invalid AI Response:", result);
                    throw new Error("La respuesta de la IA no contiene un análisis válido.");
                }

            } catch (error) {
                console.error("Error analyzing hand:", error);
                showError(`Ocurrió un error al analizar la mano: ${error.message}`);
            }
        }
        
        async function loadMockDataIfEmpty() {
            if (sessionHistory.length > 0 || mockDataLoaded) return;

            console.log("Generando datos de demostración...");

            let currentBankroll = 5000; 
            const startDate = new Date();
            startDate.setMonth(startDate.getMonth() - 6); 
            
            const tempSessionHistory = [];
            const tempActivityLog = [];
            const tempWithdrawals = [];
            
            tempActivityLog.push({ date: new Date(startDate).toLocaleString('es-ES'), type: 'Inicialización', description: `Bankroll de demostración inicializado en ${formatCurrency(currentBankroll)}.` });

            for (let i = 0; i < 180; i++) {
                const currentDate = new Date(startDate);
                currentDate.setDate(startDate.getDate() + i);

                // Add a session on about 60% of the days
                if (Math.random() > 0.4) {
                    const totalEntries = Math.floor(Math.random() * 20) + 5;
                    const totalReentries = Math.floor(Math.random() * totalEntries * 0.5);
                    const investment = (Math.random() * 15 + 10) * (totalEntries + totalReentries); 
                    const profitLoss = (Math.random() - 0.48) * investment * 1.8;

                    currentBankroll += profitLoss;
                    
                    tempSessionHistory.push({
                        date: currentDate.toLocaleDateString('es-ES', { day: '2-digit', month: '2-digit', year: 'numeric'}),
                        investment,
                        tournaments: totalEntries + totalReentries,
                        totalEntries,
                        totalReentries,
                        profitLoss,
                        finalBankroll: currentBankroll
                    });
                     tempActivityLog.unshift({date: currentDate.toLocaleString('es-ES'), type: 'Sesión Finalizada', description: `P/L de ${formatCurrency(profitLoss)}`});
                }
                
                // Add a withdrawal about once a month
                if (i > 0 && i % 45 === 0 && Math.random() > 0.5) {
                    const withdrawalAmount = Math.floor(Math.random() * 301) + 200;
                    currentBankroll -= withdrawalAmount;
                    
                    const withdrawal = {
                        date: currentDate.toLocaleDateString('es-ES'),
                        amount: withdrawalAmount,
                        description: 'Gastos mensuales',
                        source: 'GGPoker'
                    };
                    tempWithdrawals.push(withdrawal);
                    tempActivityLog.unshift({date: currentDate.toLocaleString('es-ES'), type: 'Retiro', description: `Retiro de ${formatCurrency(withdrawalAmount)} desde GGPoker`});
                }
            }
            
            // Set final balances after all operations
            bankrollBalances = {
                ggpoker: currentBankroll * 0.6,
                pokerstars: currentBankroll * 0.4,
            };
            sessionHistory = tempSessionHistory;
            withdrawals = tempWithdrawals;
            activityLog = tempActivityLog.concat(activityLog);

            await Promise.all([
                saveData.sessionHistory(),
                saveData.activityLog(),
                saveData.withdrawals(),
                saveData.config()
            ]);
            
            showToast("Datos de demostración cargados.");
        }

        function renderDashboardPage() {
            const page = document.getElementById('page-dashboard');
             page.innerHTML = `<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6"><div class="card text-center"><h3 class="text-gray-500 font-semibold">Bankroll Total</h3><p id="db-total-bankroll" class="text-3xl font-bold text-indigo-600">--</p></div><div class="card text-center"><h3 class="text-gray-500 font-semibold">P/L Total</h3><p id="db-total-pl" class="text-3xl font-bold">--</p></div><div class="card text-center"><h3 class="text-gray-500 font-semibold">ROI Total</h3><p id="db-total-roi" class="text-3xl font-bold">--</p></div><div class="card text-center"><h3 class="text-gray-500 font-semibold">Torneos Totales</h3><p id="db-total-tournaments" class="text-3xl font-bold">--</p></div></div><div class="card"><h2 class="text-2xl font-semibold mb-4">Evolución del Bankroll</h2><canvas id="dashboard-chart"></canvas></div><div class="card mt-6"><h2 class="text-2xl font-semibold mb-4">Resumen Semanal</h2><div class="overflow-x-auto"><table class="min-w-full bg-white text-sm"><thead class="bg-gray-200"><tr><th class="py-2 px-3 text-left">Día</th><th class="py-2 px-3 text-left">Torneos</th><th class="py-2 px-3 text-left">Entradas</th><th class="py-2 px-3 text-left">Reentradas</th><th class="py-2 px-3 text-left">P/L Total</th><th class="py-2 px-3 text-left">ROI</th></tr></thead><tbody id="weekly-summary-body"></tbody></table></div></div>`;
            
            const totalPl = sessionHistory.reduce((acc, s) => acc + s.profitLoss, 0);
            const totalInvestment = sessionHistory.reduce((acc, s) => acc + s.investment, 0);
            const totalTournaments = sessionHistory.reduce((acc, s) => acc + s.tournaments, 0);
            const totalRoi = totalInvestment > 0 ? (totalPl / totalInvestment) * 100 : 0;
            const totalBankroll = Object.values(bankrollBalances).reduce((acc, v) => acc + v, 0);

            document.getElementById('db-total-bankroll').textContent = formatCurrency(totalBankroll);
            const plEl = document.getElementById('db-total-pl');
            plEl.textContent = formatCurrency(totalPl);
            plEl.className = `text-3xl font-bold ${totalPl >= 0 ? 'profit' : 'loss'}`;

            const roiEl = document.getElementById('db-total-roi');
            roiEl.textContent = `${totalRoi.toFixed(2)}%`;
            roiEl.className = `text-3xl font-bold ${totalRoi >= 0 ? 'profit' : 'loss'}`;
            
            document.getElementById('db-total-tournaments').textContent = totalTournaments;

             const canvas = document.getElementById('dashboard-chart');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            if(dashboardChart){dashboardChart.destroy();} 
            
            const sortedHistory=[...sessionHistory];
            dashboardChart = new Chart(ctx,{type:'line',data:{labels:sortedHistory.map(s=>s.date),datasets:[{label:'Bankroll',data:sortedHistory.map(s=>s.finalBankroll),borderColor:'#4f46e5', tension: 0.1}]},options:{scales:{y:{title:{display:true,text:'Bankroll Total ($)'}}}}});
        
            // Render Weekly Summary
            const weeklySummary = { 0: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 1: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 2: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 3: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 4: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 5: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, 6: { tournaments: 0, totalEntries: 0, totalReentries: 0, investment: 0, profitLoss: 0 }, };
            sessionHistory.forEach(session => {
                const [day, month, year] = session.date.split('/');
                const sessionDate = new Date(`${year}-${month}-${day}`);
                let dayOfWeek = sessionDate.getDay(); 
                dayOfWeek = (dayOfWeek === 0) ? 6 : dayOfWeek - 1; 

                weeklySummary[dayOfWeek].tournaments += session.tournaments || 0;
                weeklySummary[dayOfWeek].totalEntries += session.totalEntries || 0;
                weeklySummary[dayOfWeek].totalReentries += session.totalReentries || 0;
                weeklySummary[dayOfWeek].investment += session.investment || 0;
                weeklySummary[dayOfWeek].profitLoss += session.profitLoss || 0;
            });
            const dayNames = ["Lunes", "Martes", "Miércoles", "Jueves", "Viernes", "Sábado", "Domingo"];
            const weeklySummaryBody = document.getElementById('weekly-summary-body');
            let tableHtml = '';
            for (let i = 0; i < 7; i++) {
                const dayData = weeklySummary[i];
                const roi = dayData.investment > 0 ? (dayData.profitLoss / dayData.investment) * 100 : 0;
                tableHtml += `<tr class="border-b"><td class="py-2 px-3 font-semibold">${dayNames[i]}</td><td class="py-2 px-3">${dayData.tournaments}</td><td class="py-2 px-3">${dayData.totalEntries}</td><td class="py-2 px-3">${dayData.totalReentries}</td><td class="py-2 px-3 font-bold ${dayData.profitLoss >= 0 ? 'profit' : 'loss'}">${formatCurrency(dayData.profitLoss)}</td><td class="py-2 px-3 font-semibold ${roi >= 0 ? 'profit' : 'loss'}">${roi.toFixed(2)}%</td></tr>`;
            }
            weeklySummaryBody.innerHTML = tableHtml;
        }


        // --- Make functions available globally for onclick handlers ---
        window.navigateTo = navigateTo;
        window.closeModal = closeModal;
        window.confirmDeleteHistoryEntry = confirmDeleteHistoryEntry;
        window.deleteHistoryEntry = deleteHistoryEntry;
        window.confirmDeleteStudyHand = confirmDeleteStudyHand;
        window.deleteStudyHand = deleteStudyHand;
        window.confirmDeleteJournalPost = confirmDeleteJournalPost;
        window.deleteJournalPost = deleteJournalPost;
        window.updateHabit = updateHabit;
        window.analyzeStudyHand = analyzeStudyHand;
        window.openManualEntryModal = openManualEntryModal;
        window.saveManualEntry = saveManualEntry;
        window.handleFileUpload = handleFileUpload;
        window.exportHistoryToCSV = exportHistoryToCSV;
        window.openAddPlatformModal = openAddPlatformModal;
        window.saveNewPlatform = saveNewPlatform;
        window.handleHistoryFilter = handleHistoryFilter;
        window.switchFinanceTab = switchFinanceTab;
        window.openAddHabitModal = openAddHabitModal;
        window.saveNewHabit = saveNewHabit;
        window.handleHistoryPageClick = handleHistoryPageClick;
        window.handleFileChange = handleFileChange;
        window.openReconciliationModal = openReconciliationModal;
        window.executeReconciliation = executeReconciliation;
        window.addWithdrawal = addWithdrawal;
        window.confirmWithdrawal = confirmWithdrawal;
        window.executeWithdrawal = executeWithdrawal;
        window.addCustomBuyIn = addCustomBuyIn;
        window.adjustCount = adjustCount;
        window.addBullet = addBullet;

    </script>
</body>
</html>
