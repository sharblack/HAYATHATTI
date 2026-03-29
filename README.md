<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SEMA-KARTALLARI | Maritime Command v70</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.css" />
    <script src="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">

    <style>
        :root {
            --primary: #3b82f6; --primary-glow: rgba(59, 130, 246, 0.4);
            --danger: #ef4444; --success: #10b981; --warning: #f59e0b;
            --recon: #f43f5e; --navy: #0ea5e9;
            --bg-base: #050507; --panel-bg: rgba(13, 13, 16, 0.95);
            --border: rgba(255, 255, 255, 0.08); --text: #f8fafc;
        }

        body, html { margin: 0; padding: 0; height: 100%; font-family: 'Inter', sans-serif; background: var(--bg-base); color: var(--text); overflow: hidden; }
        .leaflet-routing-container { display: none !important; }

        #login-screen { position: fixed; inset: 0; z-index: 99999; display: flex; justify-content: center; align-items: center; background: linear-gradient(rgba(0,0,0,0.85), rgba(0,0,0,0.85)), url('https://images.unsplash.com/photo-1451187580459-43490279c0fa?q=80&w=2072') center/cover; }
        .login-box { background: rgba(10, 10, 12, 0.85); padding: 50px; border-radius: 24px; width: 400px; text-align: center; border: 1px solid var(--border); backdrop-filter: blur(25px); box-shadow: 0 0 80px var(--primary-glow); }
        .login-input { width: 100%; padding: 16px; background: rgba(0,0,0,0.6); border: 1px solid #333; color: #fff; border-radius: 12px; margin-bottom: 15px; outline: none; font-size: 14px; box-sizing: border-box; }
        .login-btn { width: 100%; padding: 18px; background: var(--primary); color: #fff; border: none; font-weight: 800; border-radius: 12px; cursor: pointer; letter-spacing: 3px; transition: 0.3s; }

        #app { display: none; height: 100vh; width: 100vw; position: relative; }
        #map { position: absolute; inset: 0; z-index: 1; background: #000; }
        
        .ui-panel { position: absolute; z-index: 1000; background: var(--panel-bg); border: 1px solid var(--border); border-radius: 16px; box-shadow: 0 10px 50px rgba(0,0,0,0.9); display: flex; flex-direction: column; backdrop-filter: blur(20px); }
        #sidebar { top: 15px; left: 15px; bottom: 15px; width: 400px; padding: 25px; overflow-y: auto; scrollbar-width: none; }
        #right-panel { top: 15px; right: 15px; bottom: 15px; width: 360px; padding: 25px; }
        
        #fleet-panel { top: 15px; right: 390px; width: 300px; padding: 20px; max-height: calc(100vh - 30px); display: none; }
        .fleet-item { background: #020617; border: 1px solid #1e293b; padding: 12px; border-radius: 10px; margin-bottom: 8px; cursor: pointer; transition: 0.2s; border-left: 3px solid var(--primary); }
        .fleet-item:hover { border-color: var(--primary); background: rgba(59, 130, 246, 0.1); transform: translateX(-5px); }

        #sim-clock-panel { position: absolute; top: 15px; left: 50%; transform: translateX(-50%); z-index: 2000; background: rgba(10, 10, 12, 0.95); border: 1px solid var(--primary); padding: 10px 20px; border-radius: 16px; display: flex; align-items: center; gap: 20px; box-shadow: 0 0 30px rgba(59, 130, 246, 0.3); backdrop-filter: blur(10px); }
        .clock-time { font-size: 24px; font-weight: 800; color: #fff; font-family: monospace; letter-spacing: 2px; display: flex; flex-direction: column; line-height: 1; }
        .clock-day { font-size: 10px; color: var(--primary); letter-spacing: 3px; text-transform: uppercase; margin-top: 4px; }
        .clock-btn { background: #1e293b; color: #94a3b8; border: 1px solid #334155; padding: 8px 15px; border-radius: 8px; cursor: pointer; font-size: 11px; font-weight: 800; transition: 0.2s; }
        .clock-btn:hover { background: var(--primary); color: #fff; border-color: var(--primary); }

        .panel-header { font-size: 16px; font-weight: 800; border-bottom: 1px solid #333; padding-bottom: 15px; margin-bottom: 15px; display: flex; align-items: center; gap: 10px; color: #fff; }
        .section-label { font-size: 11px; color: #64748b; font-weight: 800; text-transform: uppercase; letter-spacing: 2px; border-left: 3px solid var(--primary); padding-left: 10px; margin: 20px 0 10px 0; }
        
        .clean-select { width: 100%; padding: 14px; background: #000; border: 1px solid #333; color: #fff; border-radius: 10px; margin-bottom: 8px; font-size: 12px; outline: none; cursor: pointer; }
        .btn-act { width: 100%; padding: 14px; border-radius: 10px; font-weight: 700; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 10px; border: 1px solid transparent; margin-bottom: 8px; font-size: 12px; transition: 0.2s; }
        .btn-red { background: rgba(239, 68, 68, 0.1); border-color: var(--danger); color: var(--danger); }
        .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 10px; }
        .btn-map { padding: 12px; background: #000; border: 1px solid #333; color: #94a3b8; font-size: 11px; font-weight: 800; border-radius: 8px; cursor: pointer; transition: 0.3s; }
        .btn-map.active { background: var(--primary); color: #fff; border-color: var(--primary); }

        #camera-overlay, #hq-modal, #detail-modal, #ai-approval-modal { position: fixed; inset: 0; background: rgba(2, 6, 23, 0.98); z-index: 100000; display: none; }
        .detail-card { background: #0f172a; border: 1px solid var(--primary); padding: 40px; border-radius: 24px; width: 950px; margin: 40px auto; max-height: 85vh; overflow-y: auto; box-shadow: 0 0 60px rgba(59,130,246,0.15); scrollbar-width: none; }
        
        .hq-section { background: #020617; padding: 20px; border-radius: 12px; border: 1px solid #1e293b; margin-bottom: 20px; }
        .hq-title { color: var(--primary); font-size: 12px; font-weight: 800; text-transform: uppercase; margin-top: 0; margin-bottom: 15px; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .personnel-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; max-height: 250px; overflow-y: auto; scrollbar-width: thin; padding-right: 5px; }
        .role-item { background: #000; border: 1px solid #334155; padding: 8px 12px; border-radius: 6px; display: flex; justify-content: space-between; align-items: center; font-size: 11px; color: #e2e8f0; }
        .role-count { background: var(--primary); color: #fff; padding: 2px 8px; border-radius: 12px; font-weight: 800; font-size: 10px; }
        
        .uav-grid { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; height: 100vh; gap: 5px; padding: 5px; }
        .uav-box { background: #000; border: 1px solid #1e293b; position: relative; border-radius: 8px; overflow: hidden; }
        .uav-label { position: absolute; top: 15px; left: 15px; background: var(--primary); padding: 6px 14px; font-size: 10px; font-weight: 800; border-radius: 6px; z-index: 2000; color: #fff; letter-spacing: 1px;}
        .cam-crosshair { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 50px; height: 50px; border: 2px solid rgba(16, 185, 129, 0.6); border-radius: 50%; z-index: 2000; pointer-events: none; }

        .hud-box { background: rgba(0,0,0,0.8); border: 1px solid #334155; border-radius: 12px; padding: 20px; margin-bottom: 20px; font-family: 'Courier New', Courier, monospace; font-size: 12px; color: var(--success); border-left: 4px solid var(--success); line-height: 1.6; transition: 0.3s; }
        .terminal { flex: 1; background: rgba(0,0,0,0.6); border-radius: 12px; padding: 15px; font-family: monospace; font-size: 11px; color: #94a3b8; overflow-y: auto; border: 1px solid #1e293b; }
        .log-line { border-left: 3px solid var(--primary); padding-left: 10px; margin-bottom: 6px; }

        .icon-assembly { background: #10b981; color: white; border-radius: 50%; width: 28px; height: 28px; display: flex; justify-content: center; align-items: center; border: 2px solid white; font-size: 13px; box-shadow: 0 0 10px rgba(16,185,129,0.8); cursor: pointer; }
        .road-block-marker { background: rgba(0,0,0,0.85); border: 2px solid var(--danger); border-radius: 6px; padding: 6px 10px; color: #fff; font-size: 11px; font-weight: bold; display: flex; align-items: center; gap: 8px; white-space: nowrap; box-shadow: 0 0 15px rgba(239,68,68,0.5); margin-left: -50%; margin-top: -50%; text-transform: uppercase; }
        
        .ai-plan-item { background: #020617; border: 1px solid #1e293b; border-left: 4px solid var(--warning); padding: 20px; border-radius: 12px; margin-bottom: 15px; position:relative; }
        .ai-plan-hq { font-size: 14px; font-weight: 800; color: var(--warning); margin-bottom: 15px; border-bottom: 1px solid #333; padding-bottom: 10px; display: flex; align-items: center; gap: 10px;}
        .ai-mini-select { background: #000; border: 1px solid #334155; color: #e2e8f0; padding: 8px; border-radius: 6px; font-size: 11px; width: 100%; margin-top: 5px; outline: none; }
        
        .loiter-ring { animation: loiterPulse 2s infinite; }
        @keyframes loiterPulse { 0% { box-shadow: 0 0 0 0 rgba(244, 63, 94, 0.7); } 70% { box-shadow: 0 0 0 20px rgba(244, 63, 94, 0); } 100% { box-shadow: 0 0 0 0 rgba(244, 63, 94, 0); } }
    </style>
</head>
<body>

    <div id="login-screen">
        <div class="login-box">
            <h1 style="color:#fff; margin:0; letter-spacing:4px; font-weight:800;">HAYAT HATTI</h1>
            <p style="color:var(--primary); font-size:11px; text-transform:uppercase; margin-bottom:30px; letter-spacing:2px;">SEMA-KARTALLARI C4ISR v70</p>
            <input type="text" id="sicil" class="login-input" placeholder="Operatör Sicil No">
            <input type="password" id="sifre" class="login-input" placeholder="Erişim Şifresi">
            <button class="login-btn" onclick="login()">SİSTEMİ BAŞLAT</button>
        </div>
    </div>

    <div id="ai-approval-modal">
        <div class="detail-card">
            <h2 id="ai-phase-title" style="color:var(--warning); margin:0; font-weight:800; font-size:22px;">🚨 SİSTEM TAVSİYESİ</h2>
            <div id="ai-plan-container" style="max-height: 350px; overflow-y:auto; padding-right:10px; margin-top:20px;"></div>
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:15px; margin-top:30px;">
                <button onclick="rejectAIPlan()" style="background:transparent; border:1px solid var(--danger); color:var(--danger); padding:18px; border-radius:12px; cursor:pointer; font-weight:800;">İPTAL</button>
                <button onclick="approveAIPlan()" style="background:var(--success); color:#fff; border:none; padding:18px; border-radius:12px; cursor:pointer; font-weight:800;">ONAYLA</button>
            </div>
        </div>
    </div>

    <div id="hq-modal">
        <div class="detail-card">
            <h2 id="hq-name" style="color:var(--primary); margin:0; border-bottom:1px solid #334155; padding-bottom:15px; font-weight:800;">KARARGAH KOMUTA</h2>
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:20px; margin-top:20px;">
                <div>
                    <div class="hq-section"><h3 class="hq-title">PERSONEL</h3><div id="hq-personnel" class="personnel-grid"></div></div>
                </div>
                <div class="hq-section">
                    <h3 class="hq-title">YENİ GÖREV ATAMASI</h3>
                    <select id="hq-mission" class="clean-select"></select>
                    <select id="hq-payload" class="clean-select"></select>
                    <select id="hq-vehicle" class="clean-select"></select>
                    <select id="hq-team" class="clean-select"></select>
                    <button class="btn-act" style="background:var(--success); color:#fff; margin-top:20px;" onclick="prepareDeployment()"><i class="fas fa-route"></i> HEDEF SEÇ</button>
                </div>
            </div>
            <button onclick="closeModal('hq-modal')" style="background:var(--danger); color:#fff; border:none; padding:16px; border-radius:12px; width:100%; margin-top:20px;">KAPAT</button>
        </div>
    </div>

    <div id="app">
        <div id="sim-clock-panel">
            <div class="clock-time" id="clock-display">00:00 <span class="clock-day">GÜN 1</span></div>
            <button class="clock-btn" onclick="advanceTime(1)">+1S</button>
            <button class="clock-btn" onclick="advanceTime(6)">+6S</button>
        </div>
        <div id="map"></div>
        <div class="ui-panel" id="sidebar">
            <div class="panel-header">STRATEJİK YÖNETİM</div>
            <div class="section-label">AFET TİPİ</div>
            <select id="disaster-select" class="clean-select">
                <option>Denizde Kayıp / Batan Gemi</option>
                <option>Deprem</option><option>Sel</option>
            </select>
            <button class="btn-act btn-red" onclick="setMode('vaka')">Vaka İşaretle</button>
            <div class="section-label">YOL KAPATMA</div>
            <select id="closure-select" class="clean-select"><option>Köprü Çökmesi</option><option>Yol Göçüğü</option></select>
            <button class="btn-act" style="background:#1e293b; color:#fff;" onclick="setMode('closure')">Yolu Kapat</button>
        </div>
        <div class="ui-panel" id="fleet-panel">
            <div class="panel-header">AKTİF FİLO</div>
            <div id="fleet-list"></div>
        </div>
        <div class="ui-panel" id="right-panel">
            <div id="live-hud" class="hud-box">SİSTEM HAZIR.</div>
            <div class="terminal" id="term"></div>
        </div>
    </div>

    <script>
        let map, osrmRouter;
        let interactionMode = 'idle', activeUnits = {}, currentL = 'dark';
        let roadObstacles = [], currentDisasterLoc = null, simHour = 0;
        let pendingDeployment = { hqKey: '', isAir: false, vehicle: '', team: '', mission: '', payload: '' };

        const POS = { 
            afad: [41.2818, 36.2750], 
            air: [41.2588, 36.5488], 
            logistics: [41.2850, 36.2600], 
            coastguard: [41.2930, 36.3450] 
        };

        const HQ_CONFIG = {
            afad: { name: "AFAD MERKEZ", isAir: false, missions: ["Kurtarma"], payloads: ["K9"], vehicles: ["4x4 Araç"], teams: ["AFAD Tim"], personnel: [{r: "Uzman", c: 20}] },
            air: { name: "HAVA ÜSSÜ", isAir: true, missions: ["Keşif"], payloads: ["Kamera"], vehicles: ["Bayraktar TB2"], teams: ["İHA Ekibi"], personnel: [{r: "Pilot", c: 10}] },
            coastguard: { name: "SAHİL GÜVENLİK", isSea: true, isAir: false, missions: ["Deniz SAR"], payloads: ["Bot"], vehicles: ["TCSG Gemi", "Zodyak Bot", "İDA (İnsansız Deniz)"], teams: ["Dalgıç Tim"], personnel: [{r: "Kaptan", c: 15}] }
        };

        function login() { 
            if(document.getElementById('sicil').value === "1919") { 
                document.getElementById('login-screen').style.display='none'; document.getElementById('app').style.display='block'; initMap(); 
            } 
        }

        function initMap() {
            map = L.map('map', { zoomControl: false }).setView([41.2800, 36.3300], 12);
            L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png').addTo(map);
            osrmRouter = L.Routing.osrmv1({ profile: 'driving' });

            Object.keys(POS).forEach(key => {
                L.marker(POS[key], { icon: L.divIcon({ html: `<div style="background:var(--primary); width:30px; height:30px; border-radius:5px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; color:#fff;"><i class="fas fa-building"></i></div>`, className:''}) }).addTo(map).on('click', () => openHQ(key));
            });
            map.on('click', handleMapClick);
            log("SİSTEM: Otonom C4ISR Ağı Aktif.");
        }

        function openHQ(key) {
            const hq = HQ_CONFIG[key];
            document.getElementById('hq-modal').style.display='block';
            document.getElementById('hq-name').innerText = hq.name;
            const fill = (id, list) => { const el = document.getElementById(id); el.innerHTML = ''; list.forEach(item => el.innerHTML += `<option>${item}</option>`); };
            fill('hq-mission', hq.missions); fill('hq-payload', hq.payloads); fill('hq-vehicle', hq.vehicles); fill('hq-team', hq.teams);
            pendingDeployment.hqKey = key; pendingDeployment.isAir = hq.isAir;
        }

        function prepareDeployment() {
            pendingDeployment.vehicle = document.getElementById('hq-vehicle').value;
            pendingDeployment.team = document.getElementById('hq-team').value;
            closeModal('hq-modal'); interactionMode = 'routing';
            log(`KOMUTA: ${pendingDeployment.vehicle} için hedef seçin.`);
        }

        function handleMapClick(e) {
            const ll = [e.latlng.lat, e.latlng.lng];
            if(interactionMode === 'vaka') {
                currentDisasterLoc = ll; L.circle(ll, {radius: 800, color: 'red'}).addTo(map);
                log("VAKA: Afet merkezi işaretlendi."); interactionMode = 'idle';
            } else if(interactionMode === 'closure') {
                roadObstacles.push(ll); L.marker(ll, {icon: L.divIcon({html: '<div class="road-block-marker">KAPALI</div>', className:''})}).addTo(map);
                log("UYARI: Yol trafiğe kapatıldı."); interactionMode = 'idle';
            } else if(interactionMode === 'routing') {
                executeRoutingPhase(ll); interactionMode = 'idle';
            }
        }

        function applyObstacleAvoidance(points) {
            let obstacles = [...roadObstacles];
            if(currentDisasterLoc) obstacles.push(currentDisasterLoc);
            if(obstacles.length === 0) return points;

            let newPoints = [points[0]];
            for (let i = 0; i < points.length - 1; i++) {
                let p1 = points[i], p2 = points[i+1];
                obstacles.forEach(obs => {
                    let dist = Math.sqrt(Math.pow(obs[0]-p2[0],2) + Math.pow(obs[1]-p2[1],2));
                    if(dist < 0.02) { // Engel çok yakınsa saptır
                        newPoints.push([obs[0] + 0.03, obs[1] + 0.03]);
                        log("NAVİGASYON: Engel tespit edildi, otonom rota değiştirildi.");
                    }
                });
                newPoints.push(p2);
            }
            return newPoints;
        }

        function executeRoutingPhase(targetPos) {
            const hq = HQ_CONFIG[pendingDeployment.hqKey];
            const startPos = POS[pendingDeployment.hqKey];
            let routePoints = [startPos, targetPos];

            if(!pendingDeployment.isAir && !hq.isSea) {
                routePoints = applyObstacleAvoidance(routePoints);
                const waypoints = routePoints.map(p => L.Routing.waypoint(L.latLng(p[0], p[1])));
                osrmRouter.route(waypoints, (err, routes) => {
                    if(!err && routes.length > 0) startMoving(routes[0].coordinates.map(c => [c.lat, c.lng]));
                    else drawDirectPath(routePoints);
                });
            } else {
                drawDirectPath(routePoints);
            }
        }

        function drawDirectPath(points) {
            let coords = [];
            for(let i=0; i<=100; i++) {
                coords.push([points[0][0]+(points[1][0]-points[0][0])*(i/100), points[0][1]+(points[1][1]-points[0][1])*(i/100)]);
            }
            startMoving(coords);
        }

        function startMoving(path) {
            const id = 'U-' + Math.random().toString(36).substr(2, 5);
            const m = L.marker(path[0], { icon: L.divIcon({ html: `<div style="background:var(--primary); width:25px; height:25px; border-radius:50%; border:2px solid #fff; box-shadow:0 0 10px #3b82f6;"></div>`, className:''}) }).addTo(map);
            activeUnits[id] = { marker: m, path: path, seg: 0, prog: 0, hqKey: pendingDeployment.hqKey, v: pendingDeployment.vehicle, state: 'outbound', isAir: pendingDeployment.isAir };
            let timer = setInterval(() => {
                let u = activeUnits[id];
                if(!u) { clearInterval(timer); return; }
                
                if(u.seg < u.path.length - 1) {
                    u.prog += 0.05; if(u.prog >= 1) { u.prog = 0; u.seg++; }
                    let p1 = u.path[u.seg], p2 = u.path[u.seg+1];
                    u.marker.setLatLng([p1[0]+(p2[0]-p1[0])*u.prog, p1[1]+(p2[1]-p1[1])*u.prog]);
                } else {
                    // Görev Sonu Mantığı
                    if(u.state === 'outbound') {
                        if(u.isAir || HQ_CONFIG[u.hqKey].isSea) {
                            log(`SİSTEM: ${u.v} görev bitti, üsse dönüyor.`);
                            u.state = 'returning'; u.path = [...u.path].reverse(); u.seg = 0; u.prog = 0;
                        } else {
                            u.state = 'completed'; log(`SİSTEM: ${u.v} hedefe ulaştı.`); clearInterval(timer);
                        }
                    } else {
                        map.removeLayer(u.marker); delete activeUnits[id]; log(`FİLO: ${u.v} üsse döndü.`); clearInterval(timer);
                    }
                }
                updateFleetPanel();
            }, 50);
        }

        function updateFleetPanel() {
            const list = document.getElementById('fleet-list'); list.innerHTML = '';
            Object.keys(activeUnits).forEach(id => {
                const u = activeUnits[id];
                list.innerHTML += `<div class="fleet-item"><b>${u.v}</b><br><small>${u.state.toUpperCase()}</small></div>`;
            });
            document.getElementById('fleet-panel').style.display = Object.keys(activeUnits).length > 0 ? 'block' : 'none';
        }

        function advanceTime(h) { simHour += h; document.getElementById('clock-display').innerHTML = `${simHour % 24}:00 <span class="clock-day">GÜN ${Math.floor(simHour/24)+1}</span>`; }
        function log(m) { const t = document.getElementById('term'); t.innerHTML += `<div class="log-line">${m}</div>`; t.scrollTop = t.scrollHeight; }
        function closeModal(id) { document.getElementById(id).style.display='none'; }
        function setMode(m) { interactionMode = m; log(`MOD: ${m.toUpperCase()} aktif. Haritaya tıklayın.`); }
    </script>
</body>
</html>
