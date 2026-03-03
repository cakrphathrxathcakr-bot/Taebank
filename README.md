<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>TaeBank - Private Portfolio & Management</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@500;600;700&family=Outfit:wght@300;400;500;600;700&family=Sarabun:wght@300;400;500;600&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        :root {
            --bg-body: #F7F5F0; --text-main: #2C303A; --text-muted: #8E929B;
            --gold-light: #E2C275; --gold-dark: #B38728; --navy: #070B19;
        }

        body { font-family: 'Sarabun', sans-serif; background: var(--bg-body); color: var(--text-main); margin: 0; padding: 0; overflow: hidden; user-select: none; -webkit-tap-highlight-color: transparent; }
        .font-eng { font-family: 'Cinzel', serif; } .font-num { font-family: 'Outfit', sans-serif; }

        /* Page Transitions */
        .page-view { display: none; width: 100%; min-height: 100dvh; position: absolute; top: 0; left: 0; background: var(--bg-body); flex-direction: column; align-items: center; z-index: 10; transition: opacity 0.5s ease; } 
        .page-view.active { display: flex; z-index: 50; }

        /* Tactical Keypad */
        .key-btn { width: 75px; height: 75px; border-radius: 50%; background: #FFF; display: flex; align-items: center; justify-content: center; font-size: 30px; font-family: 'Outfit'; cursor: pointer; box-shadow: 0 4px 15px rgba(0,0,0,0.03); transition: 0.15s; border: 1px solid rgba(0,0,0,0.02); color: var(--text-main); }
        .key-btn:active { background: #E5E0D8; transform: scale(0.9); }
        .pin-dot { width: 15px; height: 15px; border-radius: 50%; background: #E5E0D8; transition: 0.3s; border: 1px solid rgba(0,0,0,0.05); }
        .pin-dot.active { background: #D4AF37; transform: scale(1.25); box-shadow: 0 0 15px rgba(212,175,55,0.5); }

        /* Dynamic Luxury Card */
        .lux-card { 
            width: 100%; aspect-ratio: 1.586/1; border-radius: 24px; padding: 28px; 
            display: flex; flex-direction: column; justify-content: space-between; 
            position: relative; overflow: hidden; box-shadow: 0 25px 50px -12px rgba(0,0,0,0.15);
            border: 1px solid rgba(255,255,255,0.3); transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .lux-card::before { content: ''; position: absolute; inset: 0; background: linear-gradient(135deg, rgba(255,255,255,0.2) 0%, transparent 100%); pointer-events: none; }
        .lux-card::after { content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%; background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%); pointer-events: none; }

        /* Rank Holograms */
        .rank-diamond { background: linear-gradient(135deg, #FFFFFF 0%, #E0F2FE 50%, #F1F5F9 100%) !important; color: #1E3A8A !important; }
        .rank-diamond::after { background: linear-gradient(45deg, transparent 40%, rgba(255,255,255,0.8) 50%, transparent 60%) !important; animation: shine 5s infinite; }
        @keyframes shine { 0% { transform: translateX(-100%) translateY(-100%) rotate(45deg); } 20%, 100% { transform: translateX(100%) translateY(100%) rotate(45deg); } }

        /* Admin UI */
        .admin-glass { background: #121C33; border: 1px solid #1E2B4D; border-radius: 20px; }
        .input-dark { background: #0B1121; border: 1px solid #1E2B4D; color: #FFF; border-radius: 12px; padding: 14px; width: 100%; outline: none; }
        .member-row { background: #FFF; border: 1px solid #E5E7EB; border-radius: 18px; padding: 18px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; transition: 0.2s; }
        
        /* Swal2 Custom */
        .swal2-popup { border-radius: 24px !important; font-family: 'Sarabun' !important; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <div id="page-login" class="page-view active justify-center">
        <div class="w-full max-w-[420px] flex-1 flex flex-col justify-between px-8">
            <div class="flex-1 flex flex-col items-center justify-center">
                <div class="w-24 h-24 mb-6 bg-white rounded-full flex items-center justify-center shadow-xl border border-gray-50 relative">
                    <i class="fas fa-shield-halved text-5xl text-[#D4AF37]"></i>
                    <div id="db-led" class="absolute bottom-1 right-1 w-4 h-4 rounded-full bg-red-500 border-2 border-white shadow-sm"></div>
                </div>
                <h1 class="text-3xl font-eng tracking-[6px] font-bold text-[var(--text-main)]">TAEBANK</h1>
                <p id="status-text" class="text-[10px] text-[var(--text-muted)] mt-2 tracking-[3px] uppercase font-semibold">Initializing Security...</p>
                
                <div class="flex gap-5 mt-10">
                    <div class="pin-dot"></div><div class="pin-dot"></div><div class="pin-dot"></div>
                    <div class="pin-dot"></div><div class="pin-dot"></div><div class="pin-dot"></div>
                </div>
            </div>
            
            <div class="w-full pb-12">
                <div class="grid grid-cols-3 gap-y-5 gap-x-8 justify-items-center max-w-[300px] mx-auto mb-8">
                    <div class="key-btn" onclick="pressKey('1')">1</div><div class="key-btn" onclick="pressKey('2')">2</div><div class="key-btn" onclick="pressKey('3')">3</div>
                    <div class="key-btn" onclick="pressKey('4')">4</div><div class="key-btn" onclick="pressKey('5')">5</div><div class="key-btn" onclick="pressKey('6')">6</div>
                    <div class="key-btn" onclick="pressKey('7')">7</div><div class="key-btn" onclick="pressKey('8')">8</div><div class="key-btn" onclick="pressKey('9')">9</div>
                    <div></div><div class="key-btn" onclick="pressKey('0')">0</div>
                    <div class="key-btn bg-transparent shadow-none border-none text-red-400" onclick="deleteKey()"><i class="fas fa-backspace text-2xl"></i></div>
                </div>
                <p class="text-center text-[9px] text-[var(--text-muted)] tracking-widest uppercase font-bold">Developed by พระมหาจักรภัทร คมฺภีรญาณเมธี</p>
            </div>
        </div>
    </div>

    <div id="page-user" class="page-view">
        <div class="w-full max-w-[420px] flex flex-col h-screen">
            <div class="pt-14 px-7 pb-4 flex justify-between items-center">
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-white shadow-sm flex items-center justify-center border border-gray-100 text-[#D4AF37] text-xl"><i class="fas fa-crown"></i></div>
                    <div><p class="text-[var(--text-muted)] text-[10px] uppercase tracking-wider font-bold">Welcome back,</p><h2 id="userNick" class="text-2xl font-bold leading-none mt-0.5 text-[var(--text-main)]">...</h2></div>
                </div>
                <button onclick="location.reload()" class="w-10 h-10 rounded-full bg-white shadow-sm text-gray-400 active:text-red-500 transition-colors"><i class="fas fa-power-off"></i></button>
            </div>
            
            <div class="flex-1 overflow-y-auto no-scrollbar pb-10 px-7 pt-4">
                <div id="memberCard" class="lux-card">
                    <div class="flex justify-between items-start z-10">
                        <div class="w-12 h-8 rounded-lg bg-gradient-to-br from-[#E2C275] to-[#B38728] shadow-inner border border-white/20"></div>
                        <span id="memberTierText" class="font-eng text-[11px] font-bold tracking-[3px] bg-black/5 px-4 py-1.5 rounded-full uppercase">STANDARD</span>
                    </div>
                    <div class="z-10 mt-6">
                        <p class="text-[10px] font-bold opacity-70 uppercase tracking-[2px] mb-1">Available Liquidity</p>
                        <div class="flex items-baseline gap-2 text-white"><span class="text-2xl font-num">฿</span><h1 class="text-[44px] font-bold font-num tracking-tight leading-none" id="userBalance">0.00</h1></div>
                    </div>
                    <div class="flex justify-between items-end z-10">
                        <div class="text-white/90"><p id="userAccNo" class="font-num text-sm tracking-[4px] font-medium">000 000</p><p id="userFullName" class="text-[11px] font-medium tracking-wide mt-1 uppercase opacity-70">...</p></div>
                        <div class="flex opacity-80"><div class="w-8 h-8 rounded-full bg-black/20 mix-blend-overlay"></div><div class="w-8 h-8 rounded-full bg-white/40 mix-blend-overlay -ml-4"></div></div>
                    </div>
                </div>

                <div class="mt-10">
                    <div class="flex justify-between items-end mb-5 px-1">
                        <h3 class="text-[17px] font-bold tracking-tight">Recent Ledger</h3>
                        <span class="text-[10px] font-bold text-[#D4AF37] uppercase tracking-widest">Last 5 Activities</span>
                    </div>
                    <div id="userTxList" class="space-y-3"></div>
                    <div id="userTxMore" class="mt-6"></div>
                </div>
            </div>
        </div>
    </div>

    <div id="page-admin" class="page-view overflow-y-auto no-scrollbar bg-[var(--navy)]">
        <div class="w-full max-w-[500px] p-6 flex flex-col gap-6">
            
            <div class="flex justify-between items-center admin-glass p-5 shadow-2xl">
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-[rgba(212,175,55,0.1)] flex items-center justify-center text-[var(--gold)] border border-[rgba(212,175,55,0.2)] shadow-inner"><i class="fas fa-user-shield text-xl"></i></div>
                    <div><h2 class="font-bold font-eng tracking-[4px] text-white">COMMAND</h2><p class="text-[9px] text-[#4B5E82] uppercase tracking-widest font-bold">Authorized Personnel Only</p></div>
                </div>
                <button onclick="location.reload()" class="bg-red-500/10 text-red-500 px-4 py-2 rounded-xl font-bold text-[10px] tracking-widest border border-red-500/20 active:bg-red-500 active:text-white transition-all">EXIT</button>
            </div>

            <div class="grid grid-cols-2 gap-4">
                <div class="admin-glass p-6 relative overflow-hidden"><p class="text-[10px] text-[var(--gold)] font-bold uppercase tracking-widest">Bank Liquidity</p><h3 id="stat-total" class="text-3xl font-bold font-num mt-2 text-white">0.00</h3><div class="absolute -right-4 -bottom-4 opacity-5 text-6xl text-[var(--gold)]"><i class="fas fa-vault"></i></div></div>
                <div class="admin-glass p-6 relative overflow-hidden"><p class="text-[10px] text-[var(--gold)] font-bold uppercase tracking-widest">Active Accounts</p><h3 id="stat-members" class="text-3xl font-bold font-num mt-2 text-white">0</h3><div class="absolute -right-4 -bottom-4 opacity-5 text-6xl text-[var(--gold)]"><i class="fas fa-users"></i></div></div>
            </div>

            <div class="admin-glass p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xs font-bold text-white uppercase tracking-widest"><i class="fas fa-chart-pie text-[var(--gold)] mr-2"></i> Wealth Distribution</h3>
                </div>
                <div class="relative h-[240px] w-full">
                    <canvas id="doughnutChart"></canvas>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4">
                <button onclick="adminAddMember()" class="bg-blue-600 text-white p-4 rounded-2xl font-bold text-xs shadow-lg flex items-center justify-center gap-3 transition-transform active:scale-95"><i class="fas fa-user-plus text-lg"></i> CREATE ACCOUNT</button>
                <label class="bg-emerald-600 text-white p-4 rounded-2xl font-bold text-xs shadow-lg flex items-center justify-center gap-3 transition-transform active:scale-95 cursor-pointer"><i class="fas fa-file-import text-lg"></i> IMPORT LEDGER <input type="file" id="csvIn" accept=".csv" class="hidden" onchange="adminImportCSV(this)"></label>
            </div>

            <div class="admin-glass p-6 min-h-[400px]">
                <div class="relative mb-6">
                    <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-[#4B5E82]"></i>
                    <input type="text" id="adminSearch" onkeyup="adminRender()" placeholder="Search accounts or names..." class="input-dark pl-11 text-sm border-none focus:ring-1 focus:ring-[var(--gold)]">
                </div>
                <div id="adminList" class="space-y-1"></div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-app.js";
        import { getDatabase, ref, onValue, set, push, update, remove } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAiCtKW-U5QWp3yUsvToJlh2pT7sjiQ4Iw",
            authDomain: "taebankwat.firebaseapp.com",
            databaseURL: "https://taebankwat-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "taebankwat",
            storageBucket: "taebankwat.firebasestorage.app",
            messagingSenderId: "134684838871",
            appId: "1:134684838871:web:4b47416599ed4ce95c5992"
        };

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        let STATE = { members: null, transactions: null };
        let currentPin = "";
        let myChart = null;
        const ADMIN_CODE = "009098";

        // Real-time Database Bridge
        onValue(ref(db, 'members'), s => { 
            STATE.members = s.val() || {}; 
            document.getElementById('db-status').classList.replace('bg-red-500', 'bg-emerald-500');
            document.getElementById('status-text').innerText = "Security System Ready";
            if(document.getElementById('page-admin').classList.contains('active')) adminRender();
        });
        onValue(ref(db, 'transactions'), s => { 
            STATE.transactions = s.val() || {}; 
            if(document.getElementById('page-admin').classList.contains('active')) adminRender();
        });

        window.pressKey = (n) => {
            if(currentPin.length < 6) {
                currentPin += n;
                const dots = document.querySelectorAll('.pin-dot');
                dots[currentPin.length - 1].classList.add('active');
                if(currentPin.length === 6) setTimeout(processEntry, 300);
            }
        };

        window.deleteKey = () => { 
            if(currentPin.length > 0) {
                document.querySelectorAll('.pin-dot')[currentPin.length - 1].classList.remove('active');
                currentPin = currentPin.slice(0, -1); 
            }
        };

        function processEntry() {
            if (STATE.members === null) { Swal.fire('Connecting...', 'Please wait for database response.', 'info'); resetPin(); return; }

            if (currentPin === ADMIN_CODE) {
                document.getElementById('page-login').style.opacity = '0';
                setTimeout(() => {
                    document.getElementById('page-login').classList.remove('active');
                    document.getElementById('page-admin').classList.add('active');
                    adminRender();
                }, 500);
            } else {
                let mKey = Object.keys(STATE.members).find(k => k === currentPin || STATE.members[k].code === currentPin);
                if (mKey) {
                    document.getElementById('page-login').style.opacity = '0';
                    setTimeout(() => {
                        document.getElementById('page-login').classList.remove('active');
                        document.getElementById('page-user').classList.add('active');
                        loadUserPortfolio(mKey);
                    }, 500);
                } else {
                    Swal.fire({ icon: 'error', title: 'ACCESS DENIED', text: 'Invalid Credentials', background:'#FFF', confirmButtonColor: '#D4AF37' });
                    resetPin();
                }
            }
        }

        function resetPin() { currentPin = ""; document.querySelectorAll('.pin-dot').forEach(d => d.classList.remove('active')); }

        // --- USER PORTFOLIO LOGIC ---
        function loadUserPortfolio(key) {
            const user = STATE.members[key];
            let txs = []; let bal = 0;
            if(STATE.transactions) {
                Object.keys(STATE.transactions).forEach(id => {
                    let t = STATE.transactions[id];
                    if (t.studentCode === key || t.studentCode === user.code) {
                        txs.push(t);
                        bal += (t.type === 'deposit' ? parseFloat(t.amount) : -parseFloat(t.amount));
                    }
                });
            }

            // High-End Rank Styling
            let bg, text, rank;
            if (bal < 500) { bg = 'linear-gradient(135deg, #F0F2F5 0%, #D1D5DB 100%)'; text = '#1F2937'; rank = 'SILVER CLASS'; }
            else if (bal < 1000) { bg = 'linear-gradient(135deg, #FDE68A 0%, #D4AF37 100%)'; text = '#422006'; rank = 'GOLD PRESTIGE'; } 
            else if (bal < 5000) { bg = 'linear-gradient(135deg, #4B5563 0%, #1F2937 100%)'; text = '#F9FAFB'; rank = 'PLATINUM ELITE'; } 
            else if (bal < 10000) { bg = 'linear-gradient(135deg, #34D399 0%, #065F46 100%)'; text = '#ECFDF5'; rank = 'EMERALD RESERVE'; } 
            else { bg = 'linear-gradient(135deg, #FFFFFF 0%, #F1F5F9 100%)'; text = '#1E3A8A'; rank = 'DIAMOND ELITE'; }
            
            const card = document.getElementById('memberCard');
            card.style.background = bg; card.style.color = text;
            if(bal >= 10000) card.classList.add('rank-diamond'); else card.classList.remove('rank-diamond');
            
            document.getElementById('memberTierText').innerText = rank;
            document.getElementById('userNick').innerText = user.nick || '-';
            document.getElementById('userFullName').innerText = user.name || '-';
            document.getElementById('userAccNo').innerText = (user.code || key).replace(/(\d{3})(\d{3})/, '$1 $2');
            
            // Balance Animation
            animateValue("userBalance", 0, bal, 1500);

            const list = document.getElementById('userTxList');
            const info = document.getElementById('userTxMore');
            list.innerHTML = '';
            txs.sort((a,b) => new Date(b.date) - new Date(a.date));
            
            if(txs.length === 0) list.innerHTML = `<div class="text-center py-12 bg-white rounded-3xl border border-gray-100"><i class="fas fa-box-open text-4xl text-gray-200 mb-3"></i><p class="text-gray-400 text-xs font-bold uppercase tracking-widest">No activities yet</p></div>`;
            else {
                txs.slice(0, 5).forEach(t => {
                    const isD = t.type === 'deposit'; const d = new Date(t.date);
                    list.innerHTML += `
                        <div class="tx-item flex justify-between items-center scale-up">
                            <div class="flex items-center gap-4">
                                <div class="w-12 h-12 rounded-full flex items-center justify-center ${isD ? 'bg-emerald-50 text-emerald-600' : 'bg-red-50 text-red-500'} border border-black/5"><i class="fas ${isD ? 'fa-arrow-down' : 'fa-arrow-up'} text-sm"></i></div>
                                <div><p class="text-[15px] font-bold text-[var(--text-main)]">${isD ? 'Received Asset' : 'Cash Withdrawal'}</p><p class="text-[11px] text-[var(--text-muted)] font-num font-medium">${d.toLocaleDateString('en-GB',{day:'2-digit',month:'short',year:'numeric'})} • ${d.toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit'})}</p></div>
                            </div>
                            <p class="font-bold font-num text-lg ${isD ? 'text-emerald-600' : 'text-red-500'}">${isD?'+':'-'}${parseFloat(t.amount).toLocaleString()}</p>
                        </div>`;
                });
                if(txs.length > 5) info.innerHTML = `<div class="bg-white/50 backdrop-blur-sm py-2 px-4 rounded-full border border-gray-100 inline-block mx-auto shadow-sm text-[10px] text-[var(--text-muted)] font-bold uppercase tracking-widest">Showing 5 of ${txs.length} activities</div>`;
            }
        }

        // --- COMMAND CENTER (ADMIN) ---
        window.adminRender = () => {
            const list = document.getElementById('adminList');
            const search = (document.getElementById('adminSearch').value || "").toLowerCase().trim();
            let tBal = 0; let tMem = 0; let chartData = []; list.innerHTML = '';

            Object.keys(STATE.members || {}).sort((a,b)=>parseInt(a)-parseInt(b)).forEach(k => {
                const m = STATE.members[k];
                if (search && !k.includes(search) && !(m.nick||'').toLowerCase().includes(search) && !(m.name||'').toLowerCase().includes(search)) return;

                tMem++; let mBal = 0;
                if(STATE.transactions) {
                    Object.keys(STATE.transactions).forEach(txId => {
                        const tx = STATE.transactions[txId];
                        if (tx.studentCode === k) mBal += (tx.type === 'deposit' ? parseFloat(tx.amount) : -parseFloat(tx.amount));
                    });
                }
                tBal += mBal;
                if (mBal > 0) chartData.push({ name: m.nick || k, bal: mBal });

                const card = document.createElement('div'); card.className = 'member-row';
                card.innerHTML = `
                    <div class="flex-1 pr-3 overflow-hidden">
                        <span class="font-num text-[10px] text-[var(--gold)] font-bold px-2.5 py-1 rounded bg-[rgba(212,175,55,0.1)] border border-[rgba(212,175,55,0.1)]">${k}</span>
                        <h4 class="font-bold text-base mt-1.5 truncate text-[var(--text-main)]">${m.nick || '-'}</h4>
                        <div class="mt-1 text-xl font-num font-bold ${mBal >= 0 ? 'text-emerald-600' : 'text-red-500'}">฿${mBal.toLocaleString()}</div>
                    </div>
                    <div class="flex flex-col gap-2 w-32 flex-shrink-0">
                        <button class="bg-[var(--gold)] text-black py-2.5 rounded-xl font-bold text-[11px] btn-atx shadow-md active:scale-95 transition-transform uppercase tracking-widest">Transact</button>
                        <div class="flex gap-1.5">
                            <button class="flex-1 bg-gray-50 h-9 rounded-xl border border-gray-100 text-gray-400 btn-ahist"><i class="fas fa-history"></i></button>
                            <button class="flex-1 bg-gray-50 h-9 rounded-xl border border-gray-100 text-blue-500 btn-aedit"><i class="fas fa-edit"></i></button>
                            <button class="flex-1 bg-gray-50 h-9 rounded-xl border border-gray-100 text-red-500 btn-adel"><i class="fas fa-trash"></i></button>
                        </div>
                    </div>
                `;

                card.querySelector('.btn-atx').addEventListener('click', () => adminTransact(k, m.nick));
                card.querySelector('.btn-ahist').addEventListener('click', () => adminHistory(k, m.nick));
                card.querySelector('.btn-aedit').addEventListener('click', () => adminEdit(k, m.nick, m.name));
                card.querySelector('.btn-adel').addEventListener('click', () => adminDelete(k));
                list.appendChild(card);
            });

            document.getElementById('stat-total').innerText = tBal.toLocaleString(undefined, {minimumFractionDigits: 2});
            document.getElementById('stat-members').innerText = tMem;
            updateAdminChart(chartData);
        };

        function updateAdminChart(dataArr) {
            const ctx = document.getElementById('doughnutChart').getContext('2d');
            dataArr.sort((a,b) => b.bal - a.bal);
            const labels = dataArr.map(d => d.name);
            const data = dataArr.map(d => d.bal);
            const colors = ['#D4AF37','#3B82F6','#60A5FA','#10B981','#8B5CF6','#F59E0B','#06B6D4','#EC4899','#6366F1','#14B8A6'];

            if (myChart) myChart.destroy();
            if (data.length === 0) { labels.push('No Assets'); data.push(1); colors.unshift('#1E2B4D'); }

            myChart = new Chart(ctx, {
                type: 'doughnut',
                data: { labels: labels, datasets: [{ data: data, backgroundColor: colors, borderWidth: 3, borderColor: '#121C33', hoverOffset: 8 }] },
                options: {
                    responsive: true, maintainAspectRatio: false, cutout: '75%', 
                    plugins: {
                        legend: { position: 'right', labels: { color: '#8B9BB4', font: {family: 'Outfit', size: 10, weight: 'bold'}, usePointStyle: true, boxWidth: 6, padding: 12 } },
                        tooltip: {
                            backgroundColor: '#070B19', titleColor: '#D4AF37', bodyColor: '#FFF',
                            callbacks: { label: (c) => ` ฿${c.raw.toLocaleString()} (${((c.raw / c.chart._metasets[0].total) * 100).toFixed(1)}%)` }
                        }
                    }
                }
            });
        }

        async function adminTransact(k, nick) {
            const res = await Swal.fire({ title: `Ledger Entry: ${nick}`, background: '#FFF', showDenyButton: true, showCancelButton: true, confirmButtonText: '<i class="fas fa-arrow-down"></i> Deposit', confirmButtonColor: '#10B981', denyButtonText: '<i class="fas fa-arrow-up"></i> Withdraw', denyButtonColor: '#EF4444' });
            if (res.isConfirmed || res.isDenied) {
                const {value:amt} = await Swal.fire({ title: 'Enter Amount', input: 'number', inputAttributes: {min:1}, confirmButtonColor: '#D4AF37' });
                if(amt) await push(ref(db, 'transactions'), { studentCode: k, type: res.isConfirmed ? 'deposit' : 'withdraw', amount: parseFloat(amt), date: new Date().toISOString() });
            }
        }

        function adminHistory(k, nick) {
            let hHTML = `<div class="max-h-60 overflow-y-auto pr-2 text-left">`;
            let userTxs = Object.keys(STATE.transactions || {}).filter(id => STATE.transactions[id].studentCode === k).sort((a,b)=>new Date(STATE.transactions[b].date)-new Date(STATE.transactions[a].date));
            if(userTxs.length === 0) hHTML += '<p class="text-center py-8 text-gray-300 uppercase tracking-widest text-[10px] font-bold">Clear Records</p>';
            userTxs.forEach(id => {
                const t = STATE.transactions[id]; const isD = t.type==='deposit';
                hHTML += `<div class="flex justify-between items-center bg-gray-50 border border-gray-100 p-3 rounded-2xl mb-2">
                    <div><p class="text-[11px] font-bold uppercase tracking-wider ${isD?'text-emerald-600':'text-red-500'}">${isD?'Credit':'Debit'}</p><p class="text-[9px] text-gray-400 font-num font-bold mt-0.5">${new Date(t.date).toLocaleDateString('en-GB')}</p></div>
                    <div class="flex items-center gap-3"><span class="font-num font-bold text-sm text-[var(--text-main)]">${isD?'+':'-'}${parseFloat(t.amount).toLocaleString()}</span><button onclick="window.adminDelTx('${id}')" class="text-red-400 w-8 h-8 rounded-full bg-red-50 hover:bg-red-500 hover:text-white transition-colors"><i class="fas fa-trash-can text-[10px]"></i></button></div>
                </div>`;
            });
            Swal.fire({ title: `<span class="font-eng tracking-widest text-base">LEDGER: ${nick}</span>`, html: hHTML + '</div>', showConfirmButton: false, showCloseButton: true });
        }

        window.adminDelTx = async (id) => { if((await Swal.fire({title:'Erase Transaction?', text: 'This action cannot be undone', icon:'warning', showCancelButton:true, confirmButtonColor: '#EF4444'})).isConfirmed) { await remove(ref(db, 'transactions/'+id)); Swal.close(); } };

        async function adminEdit(k, nick, name) {
            const {value:form} = await Swal.fire({ title: 'Edit Client Info', html: `<input id="enick" class="swal2-input" value="${nick}" placeholder="Nickname"><input id="ename" class="swal2-input" value="${name}" placeholder="Full Name">`, preConfirm: () => [document.getElementById('enick').value, document.getElementById('ename').value] });
            if(form) await update(ref(db, 'members/'+k), { nick: form[0], name: form[1] });
        }

        async function adminDelete(k) {
            if((await Swal.fire({title:'Permanent Delete?', text: 'Client data and assets will be erased!', icon:'warning', showCancelButton:true, confirmButtonColor:'#EF4444'})).isConfirmed) {
                await remove(ref(db, 'members/'+k));
                Object.keys(STATE.transactions || {}).forEach(async id => { if(STATE.transactions[id].studentCode === k) await remove(ref(db, 'transactions/'+id)); });
            }
        }

        window.adminAddMember = async () => {
            const {value:v} = await Swal.fire({ title: 'Create Client', html: `<input id="acode" class="swal2-input" placeholder="6-Digit Account ID"><input id="anick" class="swal2-input" placeholder="Nickname"><input id="aname" class="swal2-input" placeholder="Full Name">`, preConfirm: () => [document.getElementById('acode').value.replace(/[^a-zA-Z0-9]/g,''), document.getElementById('anick').value, document.getElementById('aname').value] });
            if(v && v[0]) await set(ref(db, 'members/'+v[0]), { code: v[0], nick: v[1], name: v[2] });
        };

        window.adminImportCSV = (el) => {
            Papa.parse(el.files[0], { header: true, complete: async (r) => {
                for(let row of r.data) {
                    const code = String(row.เลขที่บัญชี||'').replace(/[^a-zA-Z0-9]/g,''); if(!code) continue;
                    await set(ref(db, 'members/'+code), { code: code, nick: row['นมัสการ สามเณร..']||'', name: row.ชื่อจริง||'' });
                    const bal = parseFloat(String(row.ยอดคงเหลือ||'').replace(/,/g,''));
                    if(bal > 0) await push(ref(db, 'transactions'), { studentCode: code, type: 'deposit', amount: bal, date: new Date().toISOString() });
                }
                Swal.fire({icon:'success', title:'Import Completed', background: '#121C33', color:'#FFF'}); el.value = '';
            }});
        };

        function animateValue(id, start, end, duration) {
            const obj = document.getElementById(id);
            let startTimestamp = null;
            const step = (timestamp) => {
                if (!startTimestamp) startTimestamp = timestamp;
                const progress = Math.min((timestamp - startTimestamp) / duration, 1);
                obj.innerText = (progress * (end - start) + start).toLocaleString(undefined, {minimumFractionDigits: 2});
                if (progress < 1) window.requestAnimationFrame(step);
            };
            window.requestAnimationFrame(step);
        }
    </script>
</body>
</html>
