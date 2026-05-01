<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="theme-color" content="#050508">
    <title>MND Live Logistics | Enterprise Cloud Tracking</title>

    <!-- Premium Fonts & Icons -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;800;900&family=Outfit:wght@300;500;700;900&family=Orbitron:wght@500;700;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

    <!-- Firebase SDKs (Database Only) -->
    <script src="https://www.gstatic.com/firebasejs/10.10.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.10.0/firebase-database-compat.js"></script>

    <style>
        /* =========================================================================
           GLOBAL RESET & VARIABLES
           ========================================================================= */
        :root {
            --gold: #D4AF37;
            --gold-glow: rgba(212, 175, 55, 0.4);
            --neon-green: #00FA9A;
            --neon-cyan: #00E5FF;
            --bg-dark: #050508;
            --panel-bg: rgba(15, 15, 20, 0.85);
            --danger: #ff3333;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Outfit', sans-serif; -webkit-tap-highlight-color: transparent; outline: none; }
        body, html { width: 100vw; height: 100vh; background: var(--bg-dark); color: #fff; overflow-x: hidden; display: flex; justify-content: center; align-items: flex-start; }

        /* Cinematic Background */
        .bg-map {
            position: fixed; inset: 0; z-index: -1; opacity: 0.15;
            background-image: 
                radial-gradient(circle at 20% 30%, var(--gold-glow) 0%, transparent 50%),
                radial-gradient(circle at 80% 80%, rgba(0, 250, 154, 0.15) 0%, transparent 50%);
            animation: pulseBg 6s infinite alternate ease-in-out;
        }
        .grid-overlay {
            position: fixed; inset: 0; z-index: -1; opacity: 0.05;
            background-image: linear-gradient(var(--gold) 1px, transparent 1px), linear-gradient(90deg, var(--gold) 1px, transparent 1px);
            background-size: 40px 40px;
        }
        @keyframes pulseBg { 0% { opacity: 0.1; transform: scale(1); } 100% { opacity: 0.25; transform: scale(1.05); } }

        /* =========================================================================
           CUSTOM TOAST NOTIFICATION SYSTEM
           ========================================================================= */
        #toast-container { position: fixed; top: 20px; left: 50%; transform: translateX(-50%); z-index: 9999999; display: flex; flex-direction: column; gap: 10px; pointer-events: none; width: 90%; max-width: 400px; }
        .toast { background: rgba(10,10,15,0.98); border-left: 4px solid var(--gold); color: #fff; padding: 15px 20px; border-radius: 8px; font-weight: 600; font-size: 14px; box-shadow: 0 10px 30px rgba(0,0,0,0.8); display: flex; align-items: center; gap: 10px; animation: toastSlideIn 0.4s cubic-bezier(0.2, 0.8, 0.2, 1) forwards, toastFadeOut 0.4s 3.5s forwards; }
        .toast.success { border-color: var(--neon-green); }
        .toast.error { border-color: var(--danger); }
        .toast i { font-size: 18px; }
        @keyframes toastSlideIn { from { opacity: 0; transform: translateY(-30px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes toastFadeOut { from { opacity: 1; transform: translateY(0); } to { opacity: 0; transform: translateY(-30px); } }

        /* =========================================================================
           VIEW MANAGER (Strict SPA)
           ========================================================================= */
        .view-container { display: none; flex-direction: column; align-items: center; width: 100%; height: 100vh; padding: 40px 20px; animation: fadeInView 0.5s ease forwards; overflow-y: auto; }
        .active-view { display: flex !important; }
        @keyframes fadeInView { from { opacity: 0; transform: scale(0.98); } to { opacity: 1; transform: scale(1); } }
        ::-webkit-scrollbar { width: 6px; } ::-webkit-scrollbar-track { background: rgba(0,0,0,0.5); } ::-webkit-scrollbar-thumb { background: var(--gold); border-radius: 10px; }

        /* =========================================================================
           UI COMPONENTS (Inputs, Buttons, Cards)
           ========================================================================= */
        .app-title { font-family: 'Cinzel', serif; color: var(--gold); font-size: 28px; font-weight: 900; letter-spacing: 2px; margin-bottom: 5px; text-shadow: 0 0 15px var(--gold-glow); text-align: center; }
        .app-subtitle { color: #aaa; font-size: 13px; margin-bottom: 30px; line-height: 1.5; font-weight: 300; text-align: center; }

        .input-group { width: 100%; position: relative; margin-bottom: 18px; }
        .input-group i { position: absolute; left: 18px; top: 18px; color: var(--gold); font-size: 18px; opacity: 0.8; }
        .mn-input {
            width: 100%; padding: 16px 16px 16px 50px; background: rgba(0,0,0,0.4);
            border: 1px solid rgba(212,175,55,0.3); color: #fff; border-radius: 12px;
            font-size: 16px; outline: none; transition: 0.3s ease; box-sizing: border-box; box-shadow: inset 0 2px 10px rgba(0,0,0,0.5);
        }
        .mn-input:focus { border-color: var(--gold); background: rgba(212,175,55,0.05); box-shadow: 0 0 15px var(--gold-glow), inset 0 2px 10px rgba(0,0,0,0.5); }
        .pin-style { letter-spacing: 12px; font-size: 24px; font-weight: 900; text-align: center; padding-left: 20px; font-family: 'Orbitron', sans-serif; color: var(--neon-cyan); }
        .pin-style + i { display: none; }
        .pin-style::placeholder { color: #555; letter-spacing: 5px; font-size: 16px; font-family: 'Outfit', sans-serif; font-weight: 400; }

        .mn-btn {
            width: 100%; padding: 16px; border-radius: 12px; font-weight: 800; font-size: 15px; cursor: pointer; transition: all 0.3s ease; display: flex; justify-content: center; align-items: center; gap: 10px; border: none; letter-spacing: 1px; text-transform: uppercase;
        }
        .mn-btn:active { transform: scale(0.96); }
        .btn-gold { background: linear-gradient(135deg, var(--gold) 0%, #FFD700 100%); color: #000; box-shadow: 0 5px 25px var(--gold-glow); }
        .btn-green { background: linear-gradient(135deg, #25D366 0%, #128C7E 100%); color: #fff; box-shadow: 0 5px 20px rgba(37, 211, 102, 0.3); }
        .btn-dark { background: rgba(0,0,0,0.5); border: 1px solid var(--gold); color: var(--gold); backdrop-filter: blur(5px); }
        .btn-danger { background: rgba(255,51,51,0.1); border: 1px solid var(--danger); color: var(--danger); margin-top: 25px; }

        .dash-header { width: 100%; max-width: 600px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; border-bottom: 1px dashed rgba(212,175,55,0.3); padding-bottom: 15px; }
        .status-badge { background: rgba(0, 250, 154, 0.1); border: 1px solid var(--neon-green); color: var(--neon-green); padding: 6px 14px; border-radius: 30px; font-size: 11px; font-weight: 800; display: flex; align-items: center; gap: 6px; letter-spacing: 1px; box-shadow: 0 0 15px rgba(0, 250, 154, 0.2); transition: 0.5s; }
        .status-badge.offline { background: rgba(255, 51, 51, 0.1); border-color: var(--danger); color: var(--danger); box-shadow: 0 0 15px rgba(255, 51, 51, 0.2); }
        
        .card { width: 100%; max-width: 600px; background: rgba(10,10,15,0.85); border: 1px solid var(--glass-border); padding: 25px; border-radius: 16px; margin-bottom: 25px; backdrop-filter: blur(15px); box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        .card h3 { color: var(--gold); font-family: 'Cinzel'; margin-bottom: 18px; font-size: 18px; border-bottom: 1px dashed rgba(212,175,55,0.3); padding-bottom: 8px; display: flex; align-items: center; gap: 10px; }

        /* =========================================================================
           PREMIUM TIMELINE, HISTORY & MAP
           ========================================================================= */
        .client-list-item { background: rgba(0,0,0,0.6); padding: 15px; border-radius: 10px; border-left: 4px solid var(--neon-cyan); margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; transition: 0.3s; }
        .client-list-item:hover { background: rgba(0, 229, 255, 0.05); border-left-color: var(--neon-green); }
        .client-list-item h4 { margin:0 0 4px 0; font-size: 15px; color: #fff; font-family: 'Cinzel', serif; }
        .client-list-item p { margin:0; font-size: 12px; color: #888; }
        .client-list-actions { display: flex; align-items: center; gap: 12px; }
        .client-list-pin { font-family: 'Orbitron', monospace; font-size: 16px; color: var(--gold); font-weight: bold; background: rgba(212,175,55,0.1); padding: 6px 10px; border-radius: 6px; letter-spacing: 2px; }
        .revoke-btn { background: transparent; border: none; color: #666; font-size: 18px; cursor: pointer; transition: 0.3s; }
        .revoke-btn:hover { color: var(--danger); transform: scale(1.1); }

        /* Advanced Map Wrapper */
        .map-wrapper { width: 100%; height: 280px; border-radius: 12px; overflow: hidden; border: 2px solid #222; position: relative; background: #050505; margin-bottom: 0px; }
        .map-iframe { width: 100%; height: 100%; border: none; filter: invert(100%) hue-rotate(180deg) brightness(85%) contrast(110%) sepia(30%); pointer-events: none; transition: 1s ease; opacity: 0; }
        .map-iframe.loaded { opacity: 1; }
        .map-loader { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: var(--gold); font-size: 13px; text-align: center; font-weight: bold; letter-spacing: 1px; z-index: 1; }
        .map-radar-ring { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 150px; height: 150px; border-radius: 50%; border: 2px solid rgba(0, 250, 154, 0.4); pointer-events: none; animation: mapPing 2.5s infinite; z-index: 2; display: none; }
        @keyframes mapPing { 0% { width: 0; height: 0; opacity: 1; } 100% { width: 300px; height: 300px; opacity: 0; } }

        /* Advanced Location Display */
        .location-display { background: rgba(212,175,55,0.05); border: 1px solid rgba(212,175,55,0.2); border-radius: 8px; padding: 12px; margin-top: 15px; margin-bottom: 15px; display: flex; align-items: center; gap: 12px; }
        .location-icon { width: 35px; height: 35px; border-radius: 50%; background: rgba(212,175,55,0.15); display: flex; justify-content: center; align-items: center; color: var(--gold); font-size: 16px; }
        .location-text h4 { margin: 0 0 3px 0; font-size: 10px; color: #888; text-transform: uppercase; letter-spacing: 1px; }
        .location-text p { margin: 0; font-size: 15px; color: #fff; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 250px; }

        .telemetrics-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 15px; }
        .metric-box { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); border-radius: 8px; padding: 10px; text-align: center; }
        .metric-label { font-size: 10px; color: #888; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 5px; }
        .metric-value { font-family: 'Orbitron', sans-serif; font-size: 15px; font-weight: 700; color: var(--neon-cyan); }

        /* Public Client Timeline Feed */
        .timeline-feed { padding-left: 20px; position: relative; max-height: 400px; overflow-y: auto; padding-right: 10px; }
        .timeline-item { position: relative; margin-bottom: 20px; padding-bottom: 20px; border-bottom: 1px dashed rgba(255,255,255,0.05); animation: slideInLeft 0.4s ease forwards; }
        .timeline-item:last-child { border-bottom: none; margin-bottom: 0; padding-bottom: 0; }
        .timeline-item::before { content: ''; position: absolute; left: -26px; top: 3px; width: 12px; height: 12px; border-radius: 50%; background: var(--neon-green); box-shadow: 0 0 10px var(--neon-green); border: 2px solid #111; z-index: 2; }
        .timeline-item::after { content: ''; position: absolute; left: -21px; top: 15px; width: 2px; height: 100%; background: rgba(0, 250, 154, 0.3); z-index: 1; }
        .timeline-item:last-child::after { display: none; }
        
        .update-time { font-size: 10px; color: var(--gold); margin-bottom: 5px; font-family: 'Orbitron', sans-serif; font-weight: bold; text-transform: uppercase; letter-spacing: 1px; }
        .update-text { font-size: 14px; color: #eee; line-height: 1.5; font-weight: 500; }

        .quick-actions { display: flex; gap: 8px; margin-bottom: 15px; flex-wrap: wrap; }
        .quick-btn { background: rgba(255,255,255,0.05); border: 1px solid rgba(212,175,55,0.4); color: #fff; padding: 8px 12px; border-radius: 8px; font-size: 11px; cursor: pointer; transition: 0.3s; font-family: 'Orbitron', sans-serif; }
        .quick-btn:hover { background: rgba(212,175,55,0.2); border-color: var(--gold); color: var(--gold); }
    </style>
</head>
<body>

    <div class="bg-map"></div>
    <div class="grid-overlay"></div>
    <div id="toast-container"></div>

    <!-- ========================================================================= -->
    <!-- VIEW 1: GATEKEEPER -->
    <!-- ========================================================================= -->
    <div id="view-gatekeeper" class="view-container active-view" style="justify-content: center;">
        <div class="gatekeeper-box" style="background: rgba(15,15,20,0.85); position:relative; overflow:hidden;">
            <h1 class="app-title"><i class="fas fa-satellite-dish"></i> MND HUB</h1>
            <p class="app-subtitle">Secured Cloud Logistics Portal.<br>Enter your Mobile Number and PIN below.</p>
            
            <div class="input-group">
                <i class="fas fa-phone-alt"></i>
                <input type="tel" id="login-phone" class="mn-input" placeholder="Mobile Number *" autocomplete="off">
            </div>
            
            <div class="input-group">
                <input type="password" id="login-pin" class="mn-input pin-style" placeholder="PIN" maxlength="6" autocomplete="off">
            </div>

            <button class="mn-btn btn-gold" id="login-btn" onclick="processLogin()"><i class="fas fa-fingerprint"></i> INITIATE HANDSHAKE</button>
            <p id="login-error" style="display:none; color:var(--danger); font-size:13px; font-weight:bold; margin-top:15px;"></p>
        </div>
    </div>

    <!-- ========================================================================= -->
    <!-- VIEW 2: ADMIN DISPATCH CENTER -->
    <!-- ========================================================================= -->
    <div id="view-admin" class="view-container">
        <div class="dash-header">
            <div>
                <h2 style="color:var(--gold); font-family:'Cinzel'; margin:0; font-size:22px;">Command Center</h2>
                <span style="font-size:12px; color:#888;">Authorized Personnel Only</span>
            </div>
            <div class="status-badge"><i class="fas fa-link"></i> DB SYNCED</div>
        </div>

        <!-- Section 1: Provision / Update PIN -->
        <div class="card">
            <h3><i class="fas fa-key"></i> Provision / Update Client Access</h3>
            <p style="font-size:11px; color:#aaa; margin-bottom:15px;">Fill details below. Enter a Custom PIN or leave PIN blank to Auto-Generate.</p>
            
            <div style="display:flex; gap:10px;">
                <input type="text" id="admin-c-name" class="mn-input" style="padding:12px; margin-bottom:12px;" placeholder="Client Name *">
                <input type="tel" id="admin-c-phone" class="mn-input" style="padding:12px; margin-bottom:12px;" placeholder="Mobile No. *">
            </div>
            
            <div style="display:flex; gap:10px;">
                <input type="text" id="admin-c-event" class="mn-input" style="flex:2; padding:12px; margin-bottom:15px;" placeholder="Event Reference *">
                <input type="text" id="admin-c-custom-pin" class="mn-input" style="flex:1; padding:12px; margin-bottom:15px; font-family:'Orbitron'; text-align:center; letter-spacing:4px; color:var(--neon-cyan);" placeholder="Custom PIN (Opt)" maxlength="6">
            </div>
            
            <button class="mn-btn btn-gold" id="btn-generate" onclick="adminGeneratePin()"><i class="fas fa-microchip"></i> AUTHORIZE & SAVE TO CLOUD</button>

            <div id="admin-pin-result" style="display:none; margin-top:20px; text-align:center; background: rgba(0,0,0,0.6); padding: 20px; border-radius: 12px; border: 1px dashed var(--neon-green);">
                <p style="color:#aaa; font-size:12px; text-transform:uppercase; letter-spacing:1px;">Client Access Provisioned:</p>
                <div style="font-family:'Orbitron', monospace; font-size:40px; color:var(--neon-green); font-weight:900; letter-spacing:10px; margin:10px 0; text-shadow: 0 0 20px rgba(0,250,154,0.4);" id="display-new-pin">000000</div>
                <button class="mn-btn btn-green" onclick="adminShareWhatsApp()"><i class="fab fa-whatsapp"></i> DISPATCH VIA WHATSAPP</button>
            </div>
        </div>

        <!-- Section 2: Sequential Network History -->
        <div class="card">
            <div style="display:flex; justify-content:space-between; align-items:center;">
                <h3 style="margin:0; border:none; padding:0;"><i class="fas fa-history"></i> Network History</h3>
                <span style="font-size:11px; background:rgba(212,175,55,0.1); padding:4px 8px; border-radius:4px; color:var(--gold);">Live Sync</span>
            </div>
            <p style="font-size:11px; color:#888; margin-top:5px; margin-bottom:15px;">Sequential list of all provisioned clients.</p>
            
            <div id="admin-active-list" style="max-height: 300px; overflow-y: auto; padding-right: 5px;">
                <div style="text-align:center; color:#555; font-size:13px; padding:20px;"><i class="fas fa-circle-notch fa-spin"></i> Fetching history logs...</div>
            </div>
        </div>

        <!-- Section 3: GPS Telemetry -->
        <div class="card">
            <h3><i class="fas fa-satellite"></i> GPS Telemetry Uplink</h3>
            <p style="font-size:12px; color:#aaa; margin-bottom:15px;">Broadcast your device hardware GPS & AI Location to the Firebase Cloud.</p>
            
            <div class="telemetrics-grid" id="admin-telemetrics" style="display:none;">
                <div class="metric-box"><div class="metric-label">Speed</div><div class="metric-value" id="a-speed">0 km/h</div></div>
                <div class="metric-box"><div class="metric-label">Accuracy</div><div class="metric-value" id="a-acc">0 m</div></div>
                <div class="metric-box"><div class="metric-label">Updates</div><div class="metric-value" id="a-ping">0</div></div>
            </div>
            
            <div id="admin-ai-location" class="location-display" style="display:none;">
                <div class="location-icon"><i class="fas fa-map-marked-alt"></i></div>
                <div class="location-text">
                    <h4>Current Area Detected</h4>
                    <p id="a-loc-name">Resolving coordinates...</p>
                </div>
            </div>

            <button class="mn-btn btn-dark" id="btn-broadcast" onclick="toggleLocationBroadcast()"><i class="fas fa-location-arrow"></i> INITIATE BROADCAST</button>
        </div>

        <!-- Section 4: Premium Comms Update -->
        <div class="card">
            <h3><i class="fas fa-comment-alt"></i> Push System Update</h3>
            
            <!-- Quick Actions -->
            <div class="quick-actions">
                <button class="quick-btn" onclick="setQuickUpdate('🚚 Logistics Fleet dispatched and en route to venue.')"><i class="fas fa-truck-moving"></i> Dispatched</button>
                <button class="quick-btn" onclick="setQuickUpdate('📍 Logistics Fleet has arrived at the location. Commencing Unloading.')"><i class="fas fa-map-marker-alt"></i> Arrived</button>
                <button class="quick-btn" onclick="setQuickUpdate('⚙️ Equipment Unloaded. Stage and Audio setup in progress.')"><i class="fas fa-tools"></i> Setup Begun</button>
                <button class="quick-btn" style="border-color:rgba(255,51,51,0.5); color:#ff6b6b;" onclick="setQuickUpdate('🛑 Minor logistics delay due to route traffic. We are actively moving.')"><i class="fas fa-traffic-light"></i> Traffic Delay</button>
            </div>

            <textarea id="admin-update-text" class="mn-input" rows="2" style="resize:none; padding:15px;" placeholder="Or type a custom update here..."></textarea>
            <button class="mn-btn btn-gold" style="margin-top:12px;" id="btn-push-update" onclick="adminPushUpdate()"><i class="fas fa-paper-plane"></i> TRANSMIT TO PUBLIC</button>
        </div>

        <button class="mn-btn btn-danger" onclick="systemLogout()"><i class="fas fa-sign-out-alt"></i> TERMINATE SESSION</button>
    </div>

    <!-- ========================================================================= -->
    <!-- VIEW 3: CLIENT LIVE DASHBOARD -->
    <!-- ========================================================================= -->
    <div id="view-client" class="view-container">
        <div class="dash-header">
            <div style="width:70%;">
                <h2 id="client-dash-event" style="color:var(--gold); font-family:'Cinzel'; margin:0; font-size:20px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">Event Overview</h2>
                <span style="font-size:12px; color:#888;">Authorized: <span id="client-dash-name" style="color:#fff;">Loading...</span></span>
            </div>
            <div class="status-badge" id="client-conn-status"><i class="fas fa-circle fa-beat"></i> ONLINE</div>
        </div>

        <!-- Map & Telemetrics Area -->
        <div class="card" style="padding: 15px;">
            <div style="display:flex; justify-content:space-between; align-items:center; padding-bottom: 12px; border-bottom: 1px dashed rgba(255,255,255,0.1); margin-bottom: 15px;">
                <h3 style="margin:0; border:none; padding:0; font-size:16px;"><i class="fas fa-crosshairs"></i> Live Target Tracking</h3>
                <span id="client-gps-time" style="font-size:11px; color:#888; font-family:'Orbitron';">Awaiting Signal...</span>
            </div>
            
            <div class="telemetrics-grid">
                <div class="metric-box"><div class="metric-label">Target Speed</div><div class="metric-value" id="c-speed">--</div></div>
                <div class="metric-box"><div class="metric-label">Heading</div><div class="metric-value" id="c-heading">--</div></div>
                <div class="metric-box"><div class="metric-label">Signal Str</div><div class="metric-value" id="c-accuracy" style="color:var(--gold);">--</div></div>
            </div>

            <div class="map-wrapper">
                <div class="map-loader" id="map-loader-text"><i class="fas fa-satellite-dish fa-spin fa-2x" style="margin-bottom:10px;"></i><br>SEARCHING FOR SATELLITE...</div>
                <div class="map-radar-ring" id="map-radar-ring"></div>
                <iframe id="client-map-iframe" class="map-iframe" src="" onload="this.classList.add('loaded')"></iframe>
            </div>
            
            <!-- Advanced Nearest Village Location Display -->
            <div class="location-display" id="client-location-display" style="display:none;">
                <div class="location-icon"><i class="fas fa-map-marker-alt"></i></div>
                <div class="location-text">
                    <h4>Nearest Detected Area</h4>
                    <p id="c-location-name">Syncing AI Location...</p>
                </div>
            </div>

            <a id="ext-map-link" href="#" target="_blank" class="mn-btn btn-dark" style="font-size:13px; padding:12px; text-decoration:none; display:none; margin-top:10px;"><i class="fas fa-external-link-alt"></i> OPEN IN GOOGLE MAPS APP</a>
        </div>

        <!-- Premium Timeline Updates Area -->
        <div class="card">
            <h3 style="margin-bottom: 25px;"><i class="fas fa-bolt"></i> Logistics Comms Feed</h3>
            <div id="client-updates-area" class="timeline-feed">
                <div style="text-align:center; color:#555; font-size:13px; padding:20px; border:none; margin-left:-20px;"><i class="fas fa-circle-notch fa-spin"></i> Listening for HQ transmissions...</div>
            </div>
        </div>

        <button class="mn-btn btn-danger" onclick="systemLogout()"><i class="fas fa-power-off"></i> DISCONNECT TERMINAL</button>
    </div>


    <!-- ========================================================================= -->
    <!-- JAVASCRIPT & FIREBASE ENGINE -->
    <!-- ========================================================================= -->
    <script>
        // --- 1. PREMIUM AUDIO & NOTIFICATIONS ---
        function requestNotificationPermission() {
            if ("Notification" in window && Notification.permission !== "granted" && Notification.permission !== "denied") {
                Notification.requestPermission();
            }
        }

        function playPremiumChime() {
            try {
                // Synthetic Web Audio API Chime (Works on mobile without needing an MP3 file)
                const ctx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(880, ctx.currentTime); // High pitch A5
                osc.frequency.exponentialRampToValueAtTime(440, ctx.currentTime + 1); // Drop to A4
                gain.gain.setValueAtTime(0.3, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 1.5);
                osc.connect(gain); gain.connect(ctx.destination);
                osc.start(); osc.stop(ctx.currentTime + 1.5);
            } catch(e) { console.log("Audio not supported"); }
        }

        function triggerPushNotification(title, body) {
            playPremiumChime();
            showToast(title, 'success');
            if ("Notification" in window && Notification.permission === "granted") {
                new Notification(title, { body: body, icon: "https://cdn-icons-png.flaticon.com/512/814/814513.png" });
            }
        }

        function showToast(message, type = 'success') {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            toast.className = `toast ${type}`;
            const icon = type === 'success' ? 'fa-check-circle' : 'fa-exclamation-triangle';
            toast.innerHTML = `<i class="fas ${icon}"></i> <span>${message}</span>`;
            container.appendChild(toast);
            setTimeout(() => { toast.remove(); }, 4000);
        }

        // --- 2. FIREBASE CONFIGURATION (Direct DB connection) ---
        const firebaseConfig = {
            apiKey: "AIzaSyCZP-zuJNDW9S4sD_d4R_-nrTMjf0HD4MM",
            authDomain: "mnd-tracking.firebaseapp.com",
            databaseURL: "https://mnd-tracking-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "mnd-tracking",
            storageBucket: "mnd-tracking.firebasestorage.app",
            messagingSenderId: "794366177184",
            appId: "1:794366177184:web:3f394f0207215ccca0fec5"
        };
        
        if (!firebase.apps.length) { firebase.initializeApp(firebaseConfig); }
        const db = firebase.database();

        // --- 3. SYSTEM STATE & CONFIG ---
        const ADMIN_NUMBERS = ["9771617808", "9153635378", "7294969938", "+918544341240", "8544341240"];
        const MASTER_PIN = "121120";
        
        let geoWatchId = null; 
        let activeClientData = null; 
        let pingCount = 0;
        let lastKnownTextUpdate = 0; // For tracking new updates

        // --- 4. VIEW ROUTER ---
        function switchView(viewId) {
            document.querySelectorAll('.view-container').forEach(el => el.classList.remove('active-view'));
            document.getElementById(viewId).classList.add('active-view');
            document.querySelectorAll('.error-msg').forEach(el => el.style.display = 'none');
            window.scrollTo(0,0);
        }

        // --- 5. THE GATEKEEPER LOGIC ---
        function processLogin() {
            requestNotificationPermission(); // Ask for push notification right on interaction

            const phone = document.getElementById('login-phone').value.trim();
            const pin = document.getElementById('login-pin').value.trim();
            const err = document.getElementById('login-error');
            const btn = document.getElementById('login-btn');

            if(!phone || !pin) { err.innerHTML = "<i class='fas fa-exclamation'></i> Credentials required."; err.style.display = 'block'; return; }

            // Admin Logic
            if(ADMIN_NUMBERS.includes(phone) && pin === MASTER_PIN) {
                document.getElementById('login-phone').value = ''; document.getElementById('login-pin').value = '';
                switchView('view-admin');
                startAdminHistoryListener(); 
                showToast("Admin Protocol Authorized");
                return;
            }

            // Client Logic
            btn.innerHTML = '<i class="fas fa-circle-notch fa-spin"></i> DECRYPTING...';
            btn.disabled = true; err.style.display = 'none';

            db.ref('trackings/' + phone).once('value').then((snapshot) => {
                btn.innerHTML = '<i class="fas fa-fingerprint"></i> INITIATE HANDSHAKE';
                btn.disabled = false;

                if (snapshot.exists()) {
                    const clientData = snapshot.val();
                    // Verify PIN
                    if(clientData.pin === pin) {
                        document.getElementById('login-phone').value = ''; document.getElementById('login-pin').value = '';
                        document.getElementById('client-dash-name').innerText = clientData.name;
                        document.getElementById('client-dash-event').innerText = clientData.event;
                        
                        startClientWatchListeners(); 
                        switchView('view-client');
                        showToast(`Welcome, ${clientData.name}`);
                    } else {
                        err.innerHTML = "<i class='fas fa-shield-alt'></i> Incorrect Security PIN."; err.style.display = 'block';
                    }
                } else {
                    err.innerHTML = "<i class='fas fa-search'></i> Profile not found in Active Database."; err.style.display = 'block';
                }
            }).catch((error) => {
                btn.innerHTML = '<i class="fas fa-fingerprint"></i> INITIATE HANDSHAKE'; btn.disabled = false;
                err.innerHTML = "<i class='fas fa-wifi'></i> Server connection failed. Check Database Rules."; err.style.display = 'block';
                console.warn(error);
            });
        }

        // --- 6. ADMIN: PROVISION & UPDATE PIN ---
        function adminGeneratePin() {
            const name = document.getElementById('admin-c-name').value.trim();
            const phone = document.getElementById('admin-c-phone').value.trim();
            const event = document.getElementById('admin-c-event').value.trim();
            const customPinInput = document.getElementById('admin-c-custom-pin').value.trim();
            const btn = document.getElementById('btn-generate');

            if(!name || !phone || !event) { showToast("Name, Phone, and Event required.", "error"); return; }
            if(customPinInput && customPinInput.length !== 6) { showToast("Custom PIN must be exactly 6 digits.", "error"); return; }

            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> SAVING TO CLOUD...'; btn.disabled = true;

            // Use custom PIN if provided, otherwise Generate Random
            const finalPin = customPinInput ? customPinInput : Math.floor(100000 + Math.random() * 900000).toString();
            const timestamp = Date.now();
            const humanDate = new Date().toLocaleString('en-US', { day:'numeric', month:'short', hour:'2-digit', minute:'2-digit'});

            activeClientData = { name, phone, event, pin: finalPin, timestamp: timestamp, dateStr: humanDate };

            // Updates perfectly across all systems
            db.ref('trackings/' + phone).set(activeClientData).then(() => {
                document.getElementById('display-new-pin').innerText = finalPin;
                document.getElementById('admin-pin-result').style.display = 'block';
                
                document.getElementById('admin-c-name').value = ''; document.getElementById('admin-c-phone').value = '';
                document.getElementById('admin-c-event').value = ''; document.getElementById('admin-c-custom-pin').value = '';

                btn.innerHTML = '<i class="fas fa-microchip"></i> AUTHORIZE & SAVE TO CLOUD'; btn.disabled = false;
                showToast("Client Provisioned/Updated Successfully");
            }).catch((error) => {
                showToast("Database Permission Denied. Check Rules.", "error");
                btn.innerHTML = '<i class="fas fa-microchip"></i> AUTHORIZE & SAVE TO CLOUD'; btn.disabled = false;
            });
        }

        function adminShareWhatsApp() {
            if(!activeClientData) return;
            const msg = `👑 *MAA NIRMALA DJ - CLOUD TRACKING* 👑\n\nHello ${activeClientData.name},\nYour logistics for *${activeClientData.event}* are now online.\n\nAccess the Live Tracker here:\nhttps://maa-nirmala-dj.github.io/Live-tracking/\n\n📱 *Login Number:* ${activeClientData.phone}\n🔐 *Tracking PIN:* ${activeClientData.pin}`;
            const cleanPhone = activeClientData.phone.replace(/\D/g,'');
            window.open(`https://wa.me/91${cleanPhone}?text=${encodeURIComponent(msg)}`, '_blank');
        }

        // --- 7. ADMIN: SEQUENTIAL HISTORY LIST ---
        function startAdminHistoryListener() {
            const listArea = document.getElementById('admin-active-list');
            db.ref('trackings').orderByChild('timestamp').on('value', (snapshot) => {
                if(snapshot.exists()) {
                    let html = '';
                    const records = [];
                    snapshot.forEach(child => records.push(child.val()));
                    
                    records.reverse().forEach(data => {
                        html += `
                            <div class="client-list-item">
                                <div style="width: 55%;">
                                    <h4>${data.name}</h4>
                                    <p style="color:#D4AF37; margin-bottom:2px;">${data.event}</p>
                                    <p>${data.phone} • <span style="font-size:9px;">${data.dateStr || 'Legacy Record'}</span></p>
                                </div>
                                <div class="client-list-actions">
                                    <div class="client-list-pin" title="Active PIN">${data.pin}</div>
                                    <button class="revoke-btn" onclick="revokeAccess('${data.phone}')" title="Revoke"><i class="fas fa-trash-alt"></i></button>
                                </div>
                            </div>
                        `;
                    });
                    listArea.innerHTML = html;
                } else {
                    listArea.innerHTML = '<div style="text-align:center; color:#555; font-size:13px; padding:20px;">History is empty.</div>';
                }
            });
        }

        function revokeAccess(phone) {
            if(confirm("Are you sure you want to permanently revoke tracking access for this number?")) {
                db.ref('trackings/' + phone).remove().then(() => showToast("Access Revoked")).catch(err => showToast("Error", "error"));
            }
        }

        // --- 8. AI REVERSE GEOCODING (Nearest Village) ---
        async function fetchLocationName(lat, lng) {
            try {
                // Free OpenStreetMap API to get village/city name
                const res = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}&zoom=14`);
                const data = await res.json();
                if(data && data.address) {
                    return data.address.village || data.address.town || data.address.city || data.address.county || data.display_name.split(',')[0];
                }
                return "Unknown Zone";
            } catch(e) { return "Coordinates Locked"; }
        }

        // --- 9. ADMIN: HARDWARE GPS TELEMETRY & AI LOCATION ---
        function toggleLocationBroadcast() {
            const btn = document.getElementById('btn-broadcast');
            const metrics = document.getElementById('admin-telemetrics');
            const locDisp = document.getElementById('admin-ai-location');

            if(geoWatchId !== null) {
                // STOP
                navigator.geolocation.clearWatch(geoWatchId);
                geoWatchId = null; pingCount = 0;
                btn.innerHTML = '<i class="fas fa-location-arrow"></i> INITIATE BROADCAST';
                btn.style.background = "transparent"; btn.style.color = "var(--gold)"; btn.style.borderColor = "var(--gold)";
                metrics.style.display = 'none'; locDisp.style.display = 'none';
                db.ref('admin_location/status').set('offline').catch(err => console.warn(err)); 
                showToast("Transmission Terminated", "error");
            } else {
                // START
                if(!navigator.geolocation) { showToast("Hardware GPS not supported.", "error"); return; }
                btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> ACQUIRING SATELLITES...';
                
                geoWatchId = navigator.geolocation.watchPosition(
                    async (pos) => {
                        btn.innerHTML = '<i class="fas fa-times-circle"></i> STOP BROADCASTING';
                        btn.style.background = "rgba(255,51,51,0.1)"; btn.style.color = "var(--danger)"; btn.style.borderColor = "var(--danger)";
                        metrics.style.display = 'grid'; locDisp.style.display = 'flex';
                        
                        pingCount++;
                        const speedKmh = pos.coords.speed ? (pos.coords.speed * 3.6).toFixed(1) : 0;
                        
                        document.getElementById('a-speed').innerText = speedKmh + " km/h";
                        document.getElementById('a-acc').innerText = "±" + Math.round(pos.coords.accuracy) + "m";
                        document.getElementById('a-ping').innerText = pingCount;

                        // AI Location Resolution
                        let locName = "Resolving...";
                        if(pingCount % 10 === 1) { // Fetch name every 10th ping to save API limits
                            locName = await fetchLocationName(pos.coords.latitude, pos.coords.longitude);
                            document.getElementById('a-loc-name').innerText = locName;
                        } else {
                            locName = document.getElementById('a-loc-name').innerText;
                        }

                        // Push to Cloud
                        db.ref('admin_location').set({
                            status: 'online', lat: pos.coords.latitude, lng: pos.coords.longitude,
                            speed: speedKmh, heading: pos.coords.heading ? Math.round(pos.coords.heading) + "°" : "N/A",
                            accuracy: Math.round(pos.coords.accuracy), time: new Date().toLocaleTimeString(),
                            locationName: locName
                        });
                        
                        if(pingCount === 1) showToast("Satellite Uplink Established");
                    },
                    (err) => {
                        showToast("GPS Error. Ensure Location Services are ON.", "error");
                        if(geoWatchId !== null) navigator.geolocation.clearWatch(geoWatchId);
                        geoWatchId = null; btn.innerHTML = '<i class="fas fa-location-arrow"></i> INITIATE BROADCAST';
                    }, 
                    { enableHighAccuracy: true, maximumAge: 0, timeout: 10000 }
                );
            }
        }

        // --- 10. ADMIN: PUSH CLOUD UPDATE ---
        function setQuickUpdate(msg) { document.getElementById('admin-update-text').value = msg; }

        function adminPushUpdate() {
            const text = document.getElementById('admin-update-text').value.trim();
            const btn = document.getElementById('btn-push-update');
            if(!text) { showToast("Enter a message to transmit.", "error"); return; }

            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> TRANSMITTING...'; btn.disabled = true;

            db.ref('live_updates').push({
                text: text, time: new Date().toLocaleTimeString('en-US', {hour: '2-digit', minute:'2-digit'}), timestamp: Date.now()
            }).then(() => {
                document.getElementById('admin-update-text').value = '';
                btn.innerHTML = '<i class="fas fa-paper-plane"></i> TRANSMIT TO PUBLIC'; btn.disabled = false;
                showToast("Update Transmitted");
            }).catch(err => {
                showToast("Permission Denied", "error"); btn.disabled = false;
            });
        }

        // --- 11. CLIENT: REAL-TIME PUBLIC TIMELINE & NOTIFICATIONS ---
        let lastKnownLat = 0; let lastKnownLng = 0;

        function startClientWatchListeners() {
            lastKnownTextUpdate = Date.now(); // Ignore old updates for notification bell

            // Watch GPS & AI Location
            db.ref('admin_location').on('value', (snapshot) => {
                const badge = document.getElementById('client-conn-status');
                if(snapshot.exists() && snapshot.val().status === 'online') {
                    const data = snapshot.val();
                    badge.innerHTML = '<i class="fas fa-circle fa-beat"></i> ONLINE'; badge.className = 'status-badge';
                    document.getElementById('client-gps-time').innerHTML = `<span style="color:var(--neon-green);">Signal Locked (${data.time})</span>`;
                    document.getElementById('c-speed').innerText = data.speed + " km/h";
                    document.getElementById('c-heading').innerText = data.heading;
                    document.getElementById('c-accuracy').innerText = "±" + data.accuracy + "m";
                    
                    document.getElementById('map-loader-text').style.display = 'none';
                    document.getElementById('map-radar-ring').style.display = 'block';
                    document.getElementById('ext-map-link').style.display = 'block';
                    document.getElementById('ext-map-link').href = `https://www.google.com/maps/search/?api=1&query=${data.lat},${data.lng}`;

                    // Update AI Village Name
                    const locDisp = document.getElementById('client-location-display');
                    if(data.locationName && data.locationName !== "Resolving...") {
                        locDisp.style.display = 'flex';
                        document.getElementById('c-location-name').innerText = data.locationName;
                    }

                    // Update Map only if coordinates changed significantly to prevent flickering
                    if(Math.abs(data.lat - lastKnownLat) > 0.0001 || Math.abs(data.lng - lastKnownLng) > 0.0001) {
                        const mapUrl = `https://maps.google.com/maps?q=${data.lat},${data.lng}&z=15&output=embed`;
                        document.getElementById('client-map-iframe').src = mapUrl;
                        lastKnownLat = data.lat; lastKnownLng = data.lng;
                    }

                } else {
                    badge.innerHTML = '<i class="fas fa-exclamation-circle"></i> SIGNAL LOST'; badge.className = 'status-badge offline';
                    document.getElementById('client-gps-time').innerText = "Target Offline";
                    document.getElementById('map-radar-ring').style.display = 'none';
                }
            });

            // Watch Timeline Feed with Push Notifications
            db.ref('live_updates').orderByChild('timestamp').limitToLast(15).on('value', (snapshot) => {
                const feedArea = document.getElementById('client-updates-area');
                if(snapshot.exists()) {
                    let html = '';
                    const updates = [];
                    snapshot.forEach(child => updates.push(child.val()));
                    
                    // Render reverse so newest is on top
                    updates.reverse().forEach((update, index) => {
                        html += `
                            <div class="timeline-item">
                                <div class="update-time">${update.time}</div>
                                <div class="update-text">${update.text}</div>
                            </div>
                        `;
                        // Trigger Push Notification for strictly new updates
                        if (index === 0 && update.timestamp > lastKnownTextUpdate) {
                            triggerPushNotification("MND Logistics Update", update.text);
                            lastKnownTextUpdate = update.timestamp; // Update tracker
                        }
                    });
                    feedArea.innerHTML = html;
                } else {
                    feedArea.innerHTML = '<div style="text-align:center; color:#555; font-size:13px; padding:20px; border:none; margin-left:-20px;">Awaiting communications...</div>';
                }
            });
        }

        // --- 12. GLOBAL LOGOUT ---
        function systemLogout() {
            if(geoWatchId !== null) toggleLocationBroadcast();
            db.ref('trackings').off(); db.ref('admin_location').off(); db.ref('live_updates').off();
            
            document.getElementById('client-map-iframe').src = "";
            document.getElementById('map-loader-text').style.display = 'block';
            document.getElementById('map-radar-ring').style.display = 'none';
            document.getElementById('client-location-display').style.display = 'none';
            document.getElementById('ext-map-link').style.display = 'none';
            document.getElementById('client-gps-time').innerText = "Awaiting Signal...";
            
            showToast("Session Terminated", "error");
            switchView('view-gatekeeper');
        }
    </script>
</body>
</html>
