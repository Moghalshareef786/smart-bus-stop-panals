# smart-bus-stop-panals
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Public Transport Information System - Digital India</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Orbitron:wght@500;700&display=swap');
        
        :root {
            --gov-blue: #003366;
            --gov-saffron: #FF9933;
            --gov-green: #138808;
        }

        body {
            background: #0f172a;
            font-family: 'Inter', sans-serif;
            color: #f8fafc;
        }

        /* LED Board Aesthetics */
        .led-panel {
            background: #000;
            border: 8px solid #1e293b;
            border-radius: 12px;
            box-shadow: 0 0 40px rgba(0,0,0,0.6), inset 0 0 20px rgba(0,0,0,1);
        }

        .led-text {
            font-family: 'Orbitron', sans-serif;
            color: #fbbf24;
            text-transform: uppercase;
        }

        .blink {
            animation: blinker 1.5s linear infinite;
        }

        @keyframes blinker {
            50% { opacity: 0.2; }
        }

        .glass-card {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255,255,255,0.1);
        }

        /* Official Ticker */
        .ticker-wrap {
            width: 100%;
            overflow: hidden;
            background: #1e293b;
            padding: 10px 0;
            border-top: 1px solid rgba(255,255,255,0.1);
        }

        .ticker {
            display: inline-block;
            white-space: nowrap;
            animation: ticker 30s linear infinite;
        }

        @keyframes ticker {
            0% { transform: translateX(100%); }
            100% { transform: translateX(-100%); }
        }

        /* Map styling */
        #map-container {
            width: 100%;
            height: 350px;
            background: #1e293b;
            border-radius: 12px;
            overflow: hidden;
        }

        /* Government Branding */
        .gov-branding {
            display: flex;
            align-items: center;
            gap: 15px;
            border-right: 1px solid rgba(255,255,255,0.2);
            padding-right: 20px;
        }

        .gov-seal-icon {
            width: 45px;
            height: 45px;
            background: white;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 15px rgba(255,255,255,0.1);
        }
    </style>
</head>
<body class="min-h-screen">

    <!-- Official Header -->
    <header class="bg-slate-900/90 sticky top-0 z-50 border-b border-white/10 backdrop-blur-md">
        <div class="container mx-auto px-6 py-4 flex flex-wrap justify-between items-center gap-4">
            <div class="flex items-center">
                <div class="gov-branding">
                    <div class="gov-seal-icon">
                        <!-- Simplified Official Seal Geometry -->
                        <svg viewBox="0 0 100 100" width="35" height="35">
                            <circle cx="50" cy="50" r="45" fill="none" stroke="#003366" stroke-width="2"/>
                            <path d="M50 5 L50 95 M5 50 L95 50" stroke="#FF9933" stroke-width="1.5"/>
                            <circle cx="50" cy="50" r="15" fill="none" stroke="#138808" stroke-width="4"/>
                            <circle cx="50" cy="50" r="4" fill="#003366"/>
                        </svg>
                    </div>
                    <div>
                        <h1 class="text-[10px] font-bold text-slate-400 uppercase tracking-widest leading-none">Government of India</h1>
                        <p class="text-xl font-bold text-white tracking-tight">Public Transport <span class="text-blue-500">Info System</span></p>
                    </div>
                </div>
            </div>

            <div class="flex items-center gap-4">
                <select id="lang-selector" onchange="changeLanguage(this.value)" class="bg-slate-800 text-white text-xs border border-white/10 rounded-lg px-3 py-2 outline-none">
                    <option value="en">English (Official)</option>
                    <option value="hi">हिन्दी (Hindi)</option>
                    <option value="te">తెలుగు (Telugu)</option>
                </select>
                <div class="flex bg-slate-800 p-1 rounded-xl border border-white/5">
                    <button onclick="switchTab('display')" id="btn-display" class="px-5 py-2 rounded-lg text-sm font-bold transition-all">
                        <span data-key="nav_display">Citizen Portal</span>
                    </button>
                    <button onclick="switchTab('operator')" id="btn-operator" class="px-5 py-2 rounded-lg text-sm font-bold transition-all">
                        <span data-key="nav_operator">Operator</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <main class="container mx-auto px-6 mt-8">
        
        <!-- CITIZEN DISPLAY -->
        <div id="view-display" class="hidden">
            <div class="flex flex-col md:flex-row md:items-end justify-between mb-8 gap-4">
                <div>
                    <div class="flex items-center gap-2 mb-2">
                        <span class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
                        <span class="text-[10px] font-bold text-slate-500 uppercase tracking-widest">Live Server Link Active</span>
                    </div>
                    <h2 class="text-3xl font-bold" data-key="display_title">Arrival Board</h2>
                    <p class="text-slate-400 text-sm" data-key="station_name">Location: Central Integrated Terminal</p>
                </div>
                <div class="glass-card px-8 py-4 rounded-2xl text-center border-blue-500/20">
                    <div id="live-clock" class="text-3xl font-bold text-blue-400 tabular-nums">00:00:00</div>
                    <div id="live-date" class="text-[10px] uppercase font-bold text-slate-500 tracking-widest mt-1">---</div>
                </div>
            </div>

            <div class="led-panel overflow-hidden">
                <div class="p-4 bg-slate-800/40 border-b border-white/10 grid grid-cols-12 gap-2 text-[10px] font-bold text-slate-500 uppercase tracking-widest">
                    <div class="col-span-2" data-key="col_line">Vehicle</div>
                    <div class="col-span-5" data-key="col_dest">Route / Destination</div>
                    <div class="col-span-3" data-key="col_status">Status</div>
                    <div class="col-span-2 text-right" data-key="col_eta">ETA</div>
                </div>
                
                <div id="display-rows" class="p-2 space-y-1 min-h-[460px]">
                    <!-- Rows Dynamic -->
                </div>

                <div class="ticker-wrap">
                    <div class="ticker text-sm text-blue-100" id="announcement-text">
                        Welcome to the official Public Transport Information System. Please maintain hygiene and follow safety protocols.
                    </div>
                </div>
            </div>
            <p class="mt-6 text-center text-slate-500 text-xs italic" data-key="click_hint">Citizen Note: Click on any row to track the vehicle's live geographical position.</p>
        </div>

        <!-- OPERATOR DASHBOARD -->
        <div id="view-operator" class="hidden">
            <div class="mb-8">
                <h2 class="text-3xl font-bold" data-key="fleet_title">Administrative Dashboard</h2>
                <p class="text-slate-400">Manage fleet dispatch and public communications.</p>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
                <!-- Summary -->
                <div class="lg:col-span-12 grid grid-cols-1 md:grid-cols-3 gap-6">
                    <div class="glass-card p-6 rounded-2xl border-t-4 border-t-blue-500">
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest" data-key="stat_active">Active Fleet</p>
                        <h4 class="text-4xl font-bold mt-2" id="stat-active">0</h4>
                    </div>
                    <div class="glass-card p-6 rounded-2xl border-t-4 border-t-green-500">
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest" data-key="stat_ontime">On Schedule</p>
                        <h4 class="text-4xl font-bold mt-2 text-green-400" id="stat-ontime">0</h4>
                    </div>
                    <div class="glass-card p-6 rounded-2xl border-t-4 border-t-orange-500">
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest" data-key="stat_delayed">Delays Reported</p>
                        <h4 class="text-4xl font-bold mt-2 text-orange-400" id="stat-delayed">0</h4>
                    </div>
                </div>

                <!-- Controls -->
                <div class="lg:col-span-4 space-y-6">
                    <div class="glass-card p-6 rounded-2xl">
                        <h3 class="font-bold text-lg mb-6 flex items-center gap-2">
                            <i class="fas fa-plus-circle text-blue-500"></i>
                            <span data-key="form_title">Dispatch New Vehicle</span>
                        </h3>
                        <div class="space-y-4">
                            <div>
                                <label class="text-[10px] font-bold text-slate-500 uppercase">Vehicle Registration</label>
                                <input type="text" id="input-bus" class="w-full bg-slate-900 border border-white/10 rounded-xl p-3 mt-1 text-white text-sm outline-none focus:border-blue-500">
                            </div>
                            <div>
                                <label class="text-[10px] font-bold text-slate-500 uppercase">Assigned Route</label>
                                <input type="text" id="input-route" class="w-full bg-slate-900 border border-white/10 rounded-xl p-3 mt-1 text-white text-sm outline-none">
                            </div>
                            <div class="grid grid-cols-2 gap-4">
                                <div>
                                    <label class="text-[10px] font-bold text-slate-500 uppercase">Initial ETA</label>
                                    <input type="text" id="input-time" placeholder="10 MIN" class="w-full bg-slate-900 border border-white/10 rounded-xl p-3 mt-1 text-white text-sm outline-none">
                                </div>
                                <div>
                                    <label class="text-[10px] font-bold text-slate-500 uppercase">State</label>
                                    <select id="input-status" class="w-full bg-slate-900 border border-white/10 rounded-xl p-3 mt-1 text-white text-sm outline-none">
                                        <option value="On Time">On Time</option>
                                        <option value="Arriving">Arriving</option>
                                        <option value="Delayed">Delayed</option>
                                    </select>
                                </div>
                            </div>
                            <button onclick="addBus()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-xl shadow-lg transition-all" data-key="btn_broadcast">PUBLISH TO BOARD</button>
                        </div>
                    </div>

                    <div class="glass-card p-6 rounded-2xl">
                        <h3 class="font-bold text-lg mb-4 text-orange-400" data-key="alert_title">Official Alert</h3>
                        <textarea id="input-announcement" class="w-full bg-slate-900 border border-white/10 rounded-xl p-4 h-24 text-sm text-white" placeholder="Type message for the scrolling ticker..."></textarea>
                        <button onclick="updateAnnouncement()" class="w-full mt-3 bg-white text-slate-900 font-bold py-3 rounded-xl hover:bg-slate-200 transition" data-key="btn_alert">UPDATE TICKER</button>
                    </div>
                </div>

                <!-- Table -->
                <div class="lg:col-span-8">
                    <div class="glass-card rounded-2xl overflow-hidden">
                        <table class="w-full text-left">
                            <thead class="bg-slate-800/50 border-b border-white/10">
                                <tr class="text-[10px] font-bold text-slate-500 uppercase tracking-widest">
                                    <th class="px-6 py-4">ID</th>
                                    <th class="px-6 py-4">Duty Details</th>
                                    <th class="px-6 py-4 text-right">Actions</th>
                                </tr>
                            </thead>
                            <tbody id="operator-list" class="divide-y divide-white/5">
                                <!-- Dynamic rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- TRACKING MODAL -->
    <div id="details-modal" class="fixed inset-0 z-[100] hidden flex items-center justify-center p-4 bg-slate-950/80 backdrop-blur-md">
        <div class="bg-slate-900 w-full max-w-5xl rounded-3xl overflow-hidden shadow-2xl border border-white/10">
            <div class="p-6 bg-slate-800 flex justify-between items-center border-b border-white/10">
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 bg-blue-500/20 rounded-2xl flex items-center justify-center text-blue-400 text-xl">
                        <i class="fas fa-satellite-dish"></i>
                    </div>
                    <div>
                        <h2 id="modal-bus-id" class="text-2xl font-bold">---</h2>
                        <p id="modal-route" class="text-xs text-slate-400 font-bold uppercase tracking-widest">---</p>
                    </div>
                </div>
                <button onclick="closeModal()" class="w-10 h-10 rounded-full hover:bg-white/10 transition flex items-center justify-center">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-12">
                <div class="lg:col-span-4 p-8 space-y-8">
                    <div>
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest mb-2" data-key="lbl_driver">Authorized Driver</p>
                        <p id="modal-driver" class="text-xl font-bold text-white">---</p>
                    </div>
                    <div>
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest mb-2" data-key="lbl_status_single">Service Health</p>
                        <span id="modal-status" class="px-3 py-1 rounded-full text-[10px] font-bold uppercase">---</span>
                    </div>
                    <div>
                        <p class="text-slate-500 text-[10px] font-bold uppercase tracking-widest mb-3" data-key="lbl_capacity">Passenger Volume</p>
                        <div class="flex items-center gap-4">
                            <div class="flex-1 h-3 bg-slate-800 rounded-full overflow-hidden">
                                <div id="modal-load-bar" class="h-full bg-blue-500 transition-all duration-1000" style="width: 0%;"></div>
                            </div>
                            <span id="modal-load-text" class="text-sm font-bold text-blue-400">0%</span>
                        </div>
                    </div>
                    <div class="pt-6 border-t border-white/5">
                        <div class="flex items-center gap-2 text-green-400 mb-2">
                            <i class="fas fa-crosshairs animate-pulse"></i>
                            <span class="text-[10px] font-bold uppercase tracking-widest">GPS Synchronized</span>
                        </div>
                        <p id="modal-coords" class="text-xs font-mono text-slate-500">LAT: 00.00 | LNG: 00.00</p>
                    </div>
                </div>

                <div class="lg:col-span-8 p-4 bg-slate-950/50">
                    <div id="map-container" class="shadow-2xl">
                        <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-8 h-8 bg-blue-500 rounded-full ring-8 ring-blue-500/20 z-10"></div>
                        <iframe 
                            id="live-google-map"
                            width="100%" 
                            height="100%" 
                            frameborder="0" 
                            style="border:0; filter: invert(90%) hue-rotate(180deg);" 
                            allowfullscreen>
                        </iframe>
                    </div>
                </div>
            </div>

            <div class="p-6 border-t border-white/10 flex justify-end">
                <button onclick="closeModal()" class="px-8 py-3 bg-slate-800 hover:bg-slate-700 text-white font-bold rounded-xl transition text-sm" data-key="btn_close">Return to Board</button>
            </div>
        </div>
    </div>

    <script>
        const translations = {
            en: {
                nav_display: "Citizen Portal", nav_operator: "Operator", station_name: "Location: Central Integrated Terminal", display_title: "Arrival Board",
                col_line: "Vehicle", col_dest: "Route / Destination", col_status: "Status", col_eta: "ETA",
                stat_active: "Active Fleet", stat_ontime: "On Schedule", stat_delayed: "Delays Reported",
                form_title: "Dispatch New Vehicle", st_ontime: "On Time", st_arriving: "Arriving", st_delayed: "Delayed",
                btn_broadcast: "PUBLISH TO BOARD", alert_title: "Official Alert", btn_alert: "UPDATE TICKER", fleet_title: "Administrative Dashboard",
                welcome: "Welcome to the official Public Transport Information System. Please maintain hygiene and follow safety protocols while traveling.",
                click_hint: "Citizen Note: Click on any row to track the vehicle's live geographical position.",
                lbl_driver: "Authorized Driver", lbl_capacity: "Passenger Volume", btn_close: "Return to Board", lbl_status_single: "Service Health"
            },
            hi: {
                nav_display: "नागरिक पोर्टल", nav_operator: "ऑपरेटर", station_name: "स्थान: केंद्रीय एकीकृत टर्मिनल", display_title: "आगमन बोर्ड",
                col_line: "वाहन", col_dest: "मार्ग / गंतव्य", col_status: "स्थिति", col_eta: "समय",
                stat_active: "सक्रिय बेड़ा", stat_ontime: "समय पर", stat_delayed: "देरी की रिपोर्ट",
                form_title: "नया वाहन भेजें", st_ontime: "समय पर", st_arriving: "आ रहा है", st_delayed: "विलंब",
                btn_broadcast: "बोर्ड पर प्रकाशित करें", alert_title: "आधिकारिक अलर्ट", btn_alert: "टिकर अपडेट करें", fleet_title: "प्रशासनिक डैशबोर्ड",
                welcome: "आधिकारिक सार्वजनिक परिवहन सूचना प्रणाली में आपका स्वागत है। कृपया स्वच्छता बनाए रखें और सुरक्षा नियमों का पालन करें।",
                click_hint: "नागरिक नोट: वाहन की लाइव स्थिति ट्रैक करने के लिए किसी भी पंक्ति पर क्लिक करें।",
                lbl_driver: "अधिकृत चालक", lbl_capacity: "यात्री संख्या", btn_close: "बोर्ड पर लौटें", lbl_status_single: "सेवा स्वास्थ्य"
            },
            te: {
                nav_display: "సిటిజన్ పోర్టల్", nav_operator: "ఆపరేటర్", station_name: "ప్రాంతం: సెంట్రల్ ఇంటిగ్రేటెడ్ టెర్మినల్", display_title: "రాక బోర్డు",
                col_line: "వాహనం", col_dest: "మార్గం / గమ్యం", col_status: "స్థితి", col_eta: "సమయం",
                stat_active: "చురుకైన వాహనాలు", stat_ontime: "సమయానికి", stat_delayed: "ఆలస్యం",
                form_title: "కొత్త వాహనం ప్రారంభం", st_ontime: "సమయానికి", st_arriving: "వస్తోంది", st_delayed: "ఆలస్యం",
                btn_broadcast: "బోర్డుకు పంపు", alert_title: "అధికారిక ప్రకటన", btn_alert: "టిక్కర్ అప్‌డేట్", fleet_title: "అడ్మినిస్ట్రేటివ్ డ్యాష్‌బోర్డ్",
                welcome: "అధికారిక ప్రజా రవాణా సమాచార వ్యవస్థకు స్వాగతం. దయచేసి భద్రతా నిబంధనలను పాటించండి.",
                click_hint: "గమనిక: వాహనం యొక్క లైవ్ లొకేషన్ చూడటానికి వరుసపై క్లిక్ చేయండి.",
                lbl_driver: "అధికారిక డ్రైవర్", lbl_capacity: "ప్రయాణీకుల సంఖ్య", btn_close: "తిరిగి వెళ్లు", lbl_status_single: "సర్వీస్ స్థితి"
            }
        };

        let currentLang = 'en';
        let busData = [
            { id: 1, bus: "IND-DL-101", route: "Inter-City Express Line", time: "2 MIN", status: "Arriving", driver: "P. Sharma", load: 78, coords: { lat: 28.6139, lng: 77.2090 } },
            { id: 2, bus: "IND-TS-205", route: "Airport Metro Shuttle", time: "10 MIN", status: "On Time", driver: "A. Reddy", load: 42, coords: { lat: 17.3850, lng: 78.4867 } },
            { id: 3, bus: "IND-KA-442", route: "District Loop Connect", time: "22 MIN", status: "Delayed", driver: "S. Murthy", load: 89, coords: { lat: 12.9716, lng: 77.5946 } }
        ];

        let announcement = translations.en.welcome;
        let gpsInterval, mapInterval;

        function changeLanguage(lang) {
            currentLang = lang;
            announcement = translations[currentLang].welcome;
            document.querySelectorAll('[data-key]').forEach(el => {
                const key = el.getAttribute('data-key');
                if (translations[currentLang][key]) el.innerText = translations[currentLang][key];
            });
            renderData();
        }

        function switchTab(tab) {
            document.getElementById('view-display').classList.add('hidden');
            document.getElementById('view-operator').classList.add('hidden');
            const btnD = document.getElementById('btn-display');
            const btnO = document.getElementById('btn-operator');
            
            if(tab === 'display') {
                document.getElementById('view-display').classList.remove('hidden');
                btnD.className = "px-5 py-2 rounded-lg text-sm font-bold transition-all bg-blue-600 text-white shadow-lg";
                btnO.className = "px-5 py-2 rounded-lg text-sm font-bold transition-all text-slate-400 hover:text-white";
            } else {
                document.getElementById('view-operator').classList.remove('hidden');
                btnO.className = "px-5 py-2 rounded-lg text-sm font-bold transition-all bg-blue-600 text-white shadow-lg";
                btnD.className = "px-5 py-2 rounded-lg text-sm font-bold transition-all text-slate-400 hover:text-white";
            }
            renderData();
        }

        function renderData() {
            const rows = document.getElementById('display-rows');
            const opList = document.getElementById('operator-list');
            rows.innerHTML = '';
            opList.innerHTML = '';

            busData.forEach(item => {
                const row = document.createElement('div');
                row.className = "grid grid-cols-12 gap-2 items-center px-4 py-5 border-b border-white/5 cursor-pointer hover:bg-white/10 transition";
                row.onclick = () => showBusDetails(item.id);
                
                let color = "text-yellow-400";
                if(item.status === 'Delayed') color = "text-red-500";
                if(item.status === 'Arriving') color = "text-green-400 blink";

                row.innerHTML = `
                    <div class="col-span-2 led-text text-xl font-bold text-white">${item.bus}</div>
                    <div class="col-span-5 led-text text-sm">${item.route}</div>
                    <div class="col-span-3 led-text text-sm ${color}">${item.status}</div>
                    <div class="col-span-2 led-text text-right text-xl">${item.time}</div>
                `;
                rows.appendChild(row);

                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="px-6 py-4 font-bold text-white">${item.bus}</td>
                    <td class="px-6 py-4 text-xs text-slate-400">
                        <span class="text-white font-medium">${item.route}</span><br>
                        Duty: ${item.driver}
                    </td>
                    <td class="px-6 py-4 text-right">
                        <button onclick="showBusDetails(${item.id})" class="text-blue-400 hover:text-white mr-4"><i class="fas fa-location-arrow"></i></button>
                        <button onclick="removeBus(${item.id})" class="text-red-500 hover:text-white"><i class="fas fa-trash-alt"></i></button>
                    </td>
                `;
                opList.appendChild(tr);
            });

            document.getElementById('stat-active').innerText = busData.length;
            document.getElementById('stat-ontime').innerText = busData.filter(b => b.status !== 'Delayed').length;
            document.getElementById('stat-delayed').innerText = busData.filter(b => b.status === 'Delayed').length;
            document.getElementById('announcement-text').innerText = announcement;
        }

        function showBusDetails(id) {
            const bus = busData.find(b => b.id === id);
            if(!bus) return;

            clearInterval(gpsInterval);
            clearInterval(mapInterval);

            document.getElementById('modal-bus-id').innerText = bus.bus;
            document.getElementById('modal-route').innerText = bus.route;
            document.getElementById('modal-driver').innerText = bus.driver;
            document.getElementById('modal-status').innerText = bus.status;
            document.getElementById('modal-load-text').innerText = bus.load + "%";
            document.getElementById('modal-load-bar').style.width = bus.load + "%";

            const statusEl = document.getElementById('modal-status');
            statusEl.className = "px-3 py-1 rounded-full text-[10px] font-bold uppercase " + 
                (bus.status === 'Delayed' ? 'bg-red-500/20 text-red-400' : 'bg-green-500/20 text-green-400');

            let lat = bus.coords.lat;
            let lng = bus.coords.lng;

            const updatePos = () => {
                lat += (Math.random() - 0.2) * 0.0001;
                lng += (Math.random() - 0.2) * 0.0001;
                document.getElementById('modal-coords').innerText = `LAT: ${lat.toFixed(5)} | LNG: ${lng.toFixed(5)}`;
            };

            const updateMap = () => {
                document.getElementById('live-google-map').src = `https://maps.google.com/maps?q=${lat},${lng}&z=15&output=embed`;
            };

            updatePos();
            updateMap();
            gpsInterval = setInterval(updatePos, 1000);
            mapInterval = setInterval(updateMap, 15000);

            document.getElementById('details-modal').classList.remove('hidden');
        }

        function closeModal() {
            document.getElementById('details-modal').classList.add('hidden');
            clearInterval(gpsInterval);
            clearInterval(mapInterval);
        }

        function addBus() {
            const b = document.getElementById('input-bus').value;
            const r = document.getElementById('input-route').value;
            const t = document.getElementById('input-time').value;
            const s = document.getElementById('input-status').value;
            if(!b || !r || !t) return;
            busData.unshift({
                id: Date.now(), bus: b.toUpperCase(), route: r, time: t, status: s,
                driver: "Official Personnel", load: Math.floor(Math.random() * 50) + 20,
                coords: { lat: 28.61, lng: 77.20 }
            });
            renderData();
            document.getElementById('input-bus').value = '';
            document.getElementById('input-route').value = '';
        }

        function removeBus(id) { 
            busData = busData.filter(b => b.id !== id); 
            renderData(); 
        }

        function updateAnnouncement() {
            const val = document.getElementById('input-announcement').value;
            if(val) { announcement = val; renderData(); }
        }

        function updateClock() {
            const now = new Date();
            document.getElementById('live-clock').innerText = now.toLocaleTimeString('en-GB');
            document.getElementById('live-date').innerText = now.toLocaleDateString(undefined, { 
                weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' 
            });
        }

        window.onload = () => {
            switchTab('display');
            setInterval(updateClock, 1000);
            updateClock();
        };
    </script>
</body>
</html>
