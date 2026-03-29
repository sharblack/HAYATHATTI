<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SEMA-KARTALLARI | Maritime Command v69</title>
    
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
            --recon: #f43f5e; --navy: #0ea5e9; /* Sahil Güvenlik Rengi */
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
        .mission-item { background: #000; border-left: 3px solid var(--success); padding: 10px; font-size: 12px; border-radius: 6px; margin-bottom: 8px; }

        .uav-grid { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; height: 100vh; gap: 5px; padding: 5px; }
        .uav-box { background: #000; border: 1px solid #1e293b; position: relative; border-radius: 8px; overflow: hidden; }
        .uav-label { position: absolute; top: 15px; left: 15px; background: var(--primary); padding: 6px 14px; font-size: 10px; font-weight: 800; border-radius: 6px; z-index: 2000; color: #fff; letter-spacing: 1px;}
        .cam-crosshair { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 50px; height: 50px; border: 2px solid rgba(16, 185, 129, 0.6); border-radius: 50%; z-index: 2000; pointer-events: none; }
        .cam-crosshair::before { content: ''; position: absolute; top: -15px; bottom: -15px; left: 50%; width: 2px; background: rgba(16, 185, 129, 0.6); }
        .cam-crosshair::after { content: ''; position: absolute; left: -15px; right: -15px; top: 50%; height: 2px; background: rgba(16, 185, 129, 0.6); }

        .hud-box { background: rgba(0,0,0,0.8); border: 1px solid #334155; border-radius: 12px; padding: 20px; margin-bottom: 20px; font-family: 'Courier New', Courier, monospace; font-size: 12px; color: var(--success); border-left: 4px solid var(--success); line-height: 1.6; transition: 0.3s; }
        .terminal { flex: 1; background: rgba(0,0,0,0.6); border-radius: 12px; padding: 15px; font-family: monospace; font-size: 11px; color: #94a3b8; overflow-y: auto; border: 1px solid #1e293b; }
        .log-line { border-left: 3px solid var(--primary); padding-left: 10px; margin-bottom: 6px; }

        .icon-assembly { background: #10b981; color: white; border-radius: 50%; width: 28px; height: 28px; display: flex; justify-content: center; align-items: center; border: 2px solid white; font-size: 13px; box-shadow: 0 0 10px rgba(16,185,129,0.8); cursor: pointer; }
        .road-block-marker { background: rgba(0,0,0,0.85); border: 2px solid var(--danger); border-radius: 6px; padding: 6px 10px; color: #fff; font-size: 11px; font-weight: bold; display: flex; align-items: center; gap: 8px; white-space: nowrap; box-shadow: 0 0 15px rgba(239,68,68,0.5); margin-left: -50%; margin-top: -50%; text-transform: uppercase; letter-spacing: 0.5px; }
        
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
            <img src="Gemini_Generated_Image_pmneyqpmneyqpmne.jpg" style="width:110px; border-radius:18px; border:2px solid var(--primary); margin-bottom:20px;">
            <h1 style="color:#fff; margin:0; letter-spacing:4px; font-weight:800;">HAYAT HATTI</h1>
            <p style="color:var(--primary); font-size:11px; text-transform:uppercase; margin-bottom:30px; letter-spacing:2px;">SEMA-KARTALLARI C4ISR v69 (MARITIME)</p>
            <input type="text" id="sicil" class="login-input" placeholder="Operatör Sicil No">
            <input type="password" id="sifre" class="login-input" placeholder="Erişim Şifresi">
            <button class="login-btn" onclick="login()">SİSTEMİ BAŞLAT</button>
        </div>
    </div>

    <div id="ai-approval-modal">
        <div class="detail-card">
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #334155; padding-bottom:20px; margin-bottom:20px;">
                <div>
                    <h2 id="ai-phase-title" style="color:var(--warning); margin:0; font-weight:800; font-size:22px;">🚨 SİSTEM TAVSİYESİ</h2>
                    <div id="ai-subtitle" style="color:#94a3b8; font-size:12px; margin-top:5px; letter-spacing:1px;">OTONOM RİSK YÖNETİMİ & SEVKİYAT PLANI</div>
                </div>
                <div id="ai-badge" style="background:rgba(245, 158, 11, 0.1); border:1px solid var(--warning); padding:10px 20px; border-radius:12px; color:var(--warning); font-weight:800; font-size:14px; text-align:center;">
                    <i class="fas fa-robot"></i> AI PREDİCTİVE
                </div>
            </div>
            
            <p id="ai-plan-desc" style="color:#e2e8f0; font-size:13px; line-height:1.6; margin-bottom:25px;">Sistem afetin gerçekleştiği zamana bağlı olarak aşağıdaki kurum ve araçların bölgeye intikal etmesini tavsiye etmektedir. <b>Parametreleri menülerden düzenleyebilir</b> ve operasyonu başlatabilirsiniz.</p>

            <div id="ai-plan-container" style="max-height: 350px; overflow-y:auto; scrollbar-width:thin; padding-right:10px;"></div>

            <div style="display:grid; grid-template-columns:1fr 1fr; gap:15px; margin-top:30px;">
                <button onclick="rejectAIPlan()" style="background:transparent; border:1px solid var(--danger); color:var(--danger); padding:18px; border-radius:12px; cursor:pointer; font-weight:800; font-size:14px; transition:0.3s;"><i class="fas fa-times"></i> İPTAL ET (MANUEL DEVAM)</button>
                <button onclick="approveAIPlan()" style="background:var(--success); color:#fff; border:none; padding:18px; border-radius:12px; cursor:pointer; font-weight:800; font-size:14px; box-shadow:0 10px 30px rgba(16,185,129,0.3);"><i class="fas fa-check-double"></i> PLANI ONAYLA VE DESTEK SEVK ET</button>
            </div>
        </div>
    </div>

    <div id="detail-modal">
        <div class="detail-card">
            <h2 id="d-title" style="color:var(--primary); margin:0; border-bottom:1px solid #334155; padding-bottom:15px; font-weight:800;">STRATEJİK VERİ MATRİSİ</h2>
            <div id="d-content" style="display:grid; grid-template-columns:1fr 1fr; gap:15px; margin-top:25px;"></div>
            <button onclick="closeModal('detail-modal')" style="margin-top:30px; background:var(--danger); color:#fff; border:none; padding:16px; border-radius:12px; cursor:pointer; font-weight:800; width:100%;">KAPAT</button>
        </div>
    </div>

    <div id="hq-modal">
        <div class="detail-card">
            <h2 id="hq-title" style="color:var(--primary); margin:0; border-bottom:1px solid #334155; padding-bottom:15px; font-weight:800; display:flex; justify-content:space-between; align-items:center;">
                <span id="hq-name">KARARGAH KOMUTA</span>
                <i class="fas fa-satellite-dish"></i>
            </h2>
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:20px; margin-top:20px;">
                <div>
                    <div class="hq-section" style="margin:0 0 20px 0;"><h3 class="hq-title"><i class="fas fa-users"></i> PERSONEL MEVCUDU (20 ROL)</h3><div id="hq-personnel" class="personnel-grid"></div></div>
                    <div class="hq-section" style="margin:0;"><h3 class="hq-title"><i class="fas fa-satellite"></i> GÖREVDEKİ ARAÇLAR</h3><div id="hq-active-missions" style="max-height: 120px; overflow-y:auto; scrollbar-width:none;"></div></div>
                </div>
                <div class="hq-section" style="margin:0;">
                    <h3 class="hq-title"><i class="fas fa-crosshairs"></i> YENİ GÖREV ATAMASI</h3>
                    <div style="font-size:10px; color:#94a3b8; margin-bottom:5px;">GÖREV TİPİ:</div><select id="hq-mission" class="clean-select"></select>
                    <div style="font-size:10px; color:#94a3b8; margin-bottom:5px;">FAYDALI YÜK / DONANIM:</div><select id="hq-payload" class="clean-select"></select>
                    <div style="font-size:10px; color:#94a3b8; margin-bottom:5px;">OPERASYON ARACI:</div><select id="hq-vehicle" class="clean-select"></select>
                    <div style="font-size:10px; color:#94a3b8; margin-bottom:5px;">ATANACAK EKİP:</div><select id="hq-team" class="clean-select"></select>
                    <div style="font-size:10px; color:#94a3b8; margin-bottom:5px;">ÖNCELİK:</div>
                    <select id="hq-priority" class="clean-select"><option>Kritik (CODE RED)</option><option>Yüksek (Sarı)</option><option>Normal Rutin</option></select>
                    <button class="btn-act" style="background:var(--success); color:#fff; margin-top:20px; margin-bottom:0; font-weight:800; letter-spacing:1px; height:50px;" onclick="prepareDeployment()"><i class="fas fa-route"></i> HEDEF SEÇ (HARİTADAN)</button>
                </div>
            </div>
            <button onclick="closeModal('hq-modal')" style="background:var(--danger); color:#fff; border:none; padding:16px; border-radius:12px; cursor:pointer; font-weight:800; width:100%; margin-top:20px;">PANELİ KAPAT</button>
        </div>
    </div>

    <div id="camera-overlay">
        <div class="uav-grid">
            <div class="uav-box"><div class="uav-label">CAM 01: YÜKSEK İRTİFA (GENEL HARİTA)</div><div class="cam-crosshair"></div><div id="cam-n" style="width:100%; height:100%;"></div></div>
            <div class="uav-box"><div class="uav-label" style="background:var(--warning); color:#000;">CAM 02: ALT İRTİFA (TAKTİK YAKIN ÇEKİM)</div><div class="cam-crosshair" style="border-color:var(--warning); border-width:1px;"></div><div id="cam-o" style="width:100%; height:100%;"></div></div>
            <div class="uav-box"><div class="uav-label" style="background:var(--danger);">CAM 03: TERMAL FLIR (ISI SENSÖRÜ)</div><div class="cam-crosshair" style="border-color:var(--danger);"></div><div id="cam-t" style="filter:invert(1) hue-rotate(180deg) brightness(1.3) contrast(1.6); width:100%; height:100%;"></div></div>
            <div class="uav-box" style="padding:40px; background:#020617; display:flex; flex-direction:column; justify-content:center;">
                <h2 style="color:var(--primary); margin-top:0; border-bottom:1px solid #333; padding-bottom:10px;">ISR TELEMETRY DATA</h2>
                <div style="color:var(--success); font-family:monospace; font-size:14px; line-height:2.2;">> SPEED: <span style="color:#fff">0.1 (Precision Mode)</span><br>> ALTITUDE: <span style="color:#fff" id="tele-alt">18,500 FT</span><br>> PAYLOAD: <span style="color:#fff" id="tele-payload">EO/IR + Termal</span><br>> UPLINK: <span style="color:#fff">%100 SECURE</span><br>> STATUS: <span style="color:#fff" id="tele-status">MONITORING TARGET</span></div>
                <button onclick="closeModal('camera-overlay')" style="margin-top:auto; background:var(--danger); color:#fff; border:none; padding:18px; border-radius:12px; cursor:pointer; font-weight:800; width:100%;">GÖRÜNTÜYÜ KES</button>
            </div>
        </div>
    </div>

    <div id="app">
        <div id="sim-clock-panel">
            <div class="clock-time" id="clock-display">00:00 <span class="clock-day">GÜN 1</span></div>
            <div style="border-left:1px solid #334155; height:30px;"></div>
            <button class="clock-btn" onclick="advanceTime(1)"><i class="fas fa-forward"></i> +1 SAAT</button>
            <button class="clock-btn" onclick="advanceTime(6)"><i class="fas fa-fast-forward"></i> +6 SAAT</button>
            <button class="clock-btn" onclick="advanceTime(24)"><i class="fas fa-step-forward"></i> +24 SAAT</button>
        </div>

        <div id="map"></div>

        <div class="ui-panel" id="sidebar">
            <div class="panel-header"><i class="fas fa-layer-group" style="color:var(--primary);"></i> STRATEJİK YÖNETİM (4 PANEL)</div>
            <div class="section-label">1. HARİTA KATMANLARI & RADAR AĞI</div>
            <div class="grid-2">
                <button class="btn-map active" id="btn-dark" onclick="switchMap('dark')">SİBER MOD</button>
                <button class="btn-map" id="btn-sat" onclick="switchMap('satellite')">UYDU MODU</button>
                <button class="btn-map" id="btn-norm" onclick="switchMap('normal')">NORMAL MOD</button>
                <button class="btn-map" id="btn-traf" onclick="switchMap('traffic')">TRAFİK MODU</button>
            </div>
            <div class="section-label">2. DOĞAL AFET & VAKA YÖNETİMİ</div>
            <select id="disaster-select" class="clean-select">
                <option>Denizde Kayıp / Batan Gemi</option>
                <option>Çığ Düşmesi</option><option>Toprak Kayması (Heyelan)</option><option>Deprem</option><option>Sel / Su Taşkını</option><option>KBRN Sızıntısı</option>
            </select>
            <button class="btn-act btn-red" onclick="setMode('vaka')"><i class="fas fa-map-pin"></i> Vaka Merkezini İşaretle</button>
            
            <div class="section-label">3. ROTALAMA MANTIĞI (OTONOM & ÇOKLU-NOKTA)</div>
            <div class="grid-2">
                <button class="btn-map active" id="btn-auto" onclick="setRouteMode('auto')">OTONOM (TEK HEDEF)</button>
                <button class="btn-map" id="btn-man" onclick="setRouteMode('manual')">MANUEL (WAYPOINT)</button>
            </div>
            <div class="section-label">4. ALTYAPI HASARI & YOL KAPATMA</div>
            <select id="closure-select" class="clean-select"><option>1. Köprü Çökmesi</option><option>2. Ana Arter Göçüğü</option><option>3. Çığ Düşmesi (Yol İptal)</option><option>4. Tünel İçi Yangın</option><option>5. Kimyasal Sızıntı</option><option>6. Sel / Su Taşkını</option></select>
            <button class="btn-act" style="background:#1e293b; color:#fff;" onclick="setMode('closure')"><i class="fas fa-ban"></i> Yol Kapat (Haritadan Seç)</button>
            <div class="section-label">5. SİVİL TAHLİYE KORİDORU</div>
            <button class="btn-act" style="border:1px solid var(--success); color:var(--success); background:transparent;" onclick="setMode('evacuation')"><i class="fas fa-route"></i> Çoklu Güvenli Koridor Çiz</button>
        </div>

        <div class="ui-panel" id="fleet-panel">
            <div class="panel-header" style="font-size:12px; margin-bottom:15px; padding-bottom:10px;"><i class="fas fa-fighter-jet" style="color:var(--primary);"></i> AKTİF FİLO AĞI (ODAKLAN)</div>
            <div id="fleet-list" style="overflow-y:auto; scrollbar-width:none; display:flex; flex-direction:column; gap:8px;">
                <div style="color:#64748b; font-size:11px; text-align:center;">Şu an sahada aktif araç yok.</div>
            </div>
        </div>

        <div class="ui-panel" id="right-panel">
            <div class="section-label" style="margin-top:0;">CANLI TELEMETRİ (HIZ: 0.1)</div>
            <div id="live-hud" class="hud-box">SİSTEM HAZIR. KARARGAH SEÇİNİZ VEYA SAATİ İLERLETİNİZ...</div>
            <div class="section-label">OPERASYON LOGLARI</div>
            <div class="terminal" id="term"></div>
        </div>
    </div>

    <script>
        let map, cN, cO, cT, osrmRouter;
        let interactionMode = 'idle', routeMode = 'auto', activeUnits = {}, currentL = 'dark';
        let tempPoints = [], evacPoints = [], activeRouteLine = null, currentDisasterLoc = null;
        let roadObstacles = [];
        let baseLayers = {};
        let tempRouteGroup; 
        let windowTempPreviewLine, windowTempEvacLine;

        let simHour = 0;
        let phasesTriggered = { p1: false, p2: false, p3: false };
        let aiDeployQueue = [];
        let isAutoDeploying = false;
        let pendingDeployment = { hqKey: '', isAir: false, vehicle: '', team: '', mission: '', payload: '', priority: '' };

        // YENİ: Sahil Güvenlik (Samsun Limanı) eklendi
        const POS = { 
            afad: [41.2818, 36.2750], 
            air: [41.2588, 36.5488], 
            logistics: [41.2850, 36.2600], 
            fuel: [41.2700, 36.3500],
            coastguard: [41.2930, 36.3450] 
        };
        
        const STATION_DATA = { assembly: [ { k: "Alan Kapasitesi", v: "8.500 Kişi" }, { k: "Mevcut Durum", v: "Açık - Aktif Tahliye Alanı" }, { k: "Çadır & Barınma", v: "1500 Çadır Kuruldu" }, { k: "Altyapı Hizmetleri", v: "Seyyar Tuvalet, Jeneratör Aydınlatma" }, { k: "İkmal Durumu", v: "Günlük Sıcak Yemek ve Temiz Su Dağıtımı" }, { k: "Medikal Destek", v: "Sahra Tipi İlk Yardım İstasyonu" } ] };
        const TR_STATIONS = { assembly: [ [41.265, 36.315], [41.250, 36.350], [41.240, 36.250], [41.220, 36.300], [41.275, 36.290] ] };

        // YENİ: Sahil Güvenlik Veritabanı
        const HQ_CONFIG = {
            afad: { 
                name: "SAMSUN AFAD ANA KARARGAHI", isAir: false, 
                missions: ["Arama-Kurtarma (SAR)", "Enkaz Altı Dinleme", "KBRN Tahliyesi", "Sualtı Kurtarma", "Dağcılık Kurtarma", "Maden Kazası Müdahalesi", "Kentsel Arama", "Çığ Müdahalesi", "Sel Suları Tahliyesi", "Tıbbi Triyaj", "Acil Barınma Kurulumu", "Psikososyal Destek", "İletişim Ağı Kurma", "Hasar Tespit", "Enkaz Kaldırma", "Tahliye Koridoru Açma", "Kayıp Şahıs Arama", "Trafik Kaza Kırımı", "Asılsız İhbar Kontrolü", "İnsani Yardım Dağıtımı"], 
                payloads: ["Ağır Kurtarma Teçhizatı", "Sismik Dinleme Kiti", "Termal Arama Sensörü", "Hidrolik Kesici Takım", "KBRN Koruyucu Giysi", "Dalgıç Ekipmanı", "Şişme Bot", "K9 Arama Köpeği", "Seyyar Aydınlatma Kulesi", "Jeneratör (50kVA)", "Uydu Telefonu (BGAN)", "İnsansız Su Altı Aracı", "Enkaz Altı Yılan Kamera", "Kurtarma Çadırı", "Geniş İlkyardım Çantası", "Kar Aracı Paleti", "Çekme Halatı (Tonajlı)", "Radyasyon Ölçer", "Gaz Dedektörü", "Portatif Telsiz Rölesi"], 
                vehicles: ["Mercedes Unimog U4000", "KBRN Temizleme Aracı", "Otokar Akrep II (Zırhlı)", "4x4 Kurtarma Aracı", "Zodyak Bot Taşıyıcı", "Amfibik Kurtarma (ATV)", "Mobil Koordinasyon Tırı", "Otoyol Çekicisi", "Mini Ekskavatör", "Bobcat Yükleyici", "Kamyonet (Lojistik)", "Kar Motoru", "Hovercraft (Bataklık)", "Köpek Taşıma Aracı", "Personel Midibüsü", "Tam Donanımlı Ambulans", "Çok Amaçlı İtfaiye", "Drone İletişim Aracı", "Mobil Mutfak Aracı", "Motosikletli Kurye"], 
                teams: ["AFAD Ağır Kurtarma", "UMKE Medikal", "JAK Jandarma", "SAK Sualtı Kurtarma", "KBRN Müdahale Timi", "K9 Arama Timi", "Dağ Kurtarma Ekibi", "Gönüllü İrtibat Ekibi", "Hasar Tespit Mühendisleri", "Psikososyal Destek Ekibi", "İletişim Teknolojileri", "İnsani Yardım Dağıtım", "Lojistik Planlama", "Mobil Komuta Ekibi", "İtfaiye Destek", "İHA Operatör Timi", "İlkyardım Eğitimcileri", "Telsiz Kriz Ağı Timi", "Basın/Halkla İlişkiler", "Ulaşım Koordinasyon"], 
                personnel: [{r: "Arama Kurtarma Uzmanı", c: 45}, {r: "Enkaz Dinleme Tek.", c: 12}, {r: "K9 İdarecisi", c: 8}, {r: "KBRN Uzmanı", c: 15}, {r: "Acil Tıp (ATT)", c: 30}, {r: "UMKE Doktoru", c: 10}, {r: "İş Makinesi Operatörü", c: 25}, {r: "Lojistik Uzmanı", c: 15}, {r: "Saha İletişim Subayı", c: 8}, {r: "Psikososyal Destek", c: 12}, {r: "Dalgıç", c: 10}, {r: "Dağcı Uzman", c: 14}, {r: "İnşaat Müh.", c: 8}, {r: "Drone Operatörü", c: 6}, {r: "Ağır Vasıta Şoförü", c: 20}, {r: "Telsiz Operatörü", c: 10}, {r: "Kriz Yönetim", c: 5}, {r: "Paramedik", c: 22}, {r: "Biyolog", c: 4}, {r: "Vinç Operatörü", c: 8}] 
            },
            air: { 
                name: "SAMSUN ÇARŞAMBA HAVA ÜSSÜ", isAir: true, 
                missions: ["Keşif / Gözetleme (ISR)", "Hava İndirme İntikali", "Havadan Medikal Tahliye", "Hedef İşaretleme", "Hasar Tespiti", "Havadan İkmal Atımı", "Telsiz Rölesi", "Yangın Söndürme", "Yakın Hava Desteği", "Elektronik Harp", "Sınır Güvenliği", "VIP Tahliye", "Deniz Yüzeyi Arama", "Gece Görüşlü Devriye", "KBRN Bulut Analizi", "Topografik Haritalama", "Kordon Altına Alma", "Taktik Nakliye", "Hızlı Halatla İnme", "Çevresel İzleme"], 
                payloads: ["Elektro-Optik (EO/IR)", "Medikal Tahliye Sepeti", "Lazer İşaretleyici", "SAR Radarı", "Taktik Veri Bağı", "Bambi Bucket (Su)", "Yardım Paraşütü", "Elektronik Destek", "Lidar Sensörü", "Flare Atıcı", "Telsiz Röle Podu", "Termal Kamera", "Kurtarma Vinci", "Geniş Alan Gözetleme", "MegaFon", "Aydınlatma Fişeği", "Meteoroloji Sensörü", "C-Band Verici", "IFF Anteni", "JP-8 Havada İkmal Yakıtı"], 
                vehicles: ["Bayraktar TB2 İHA", "AKINCI TİHA", "Sikorsky S-70", "T-129 ATAK", "ANKA-S SİHA", "Aksungur İHA", "CH-47 Chinook", "AS532 Cougar", "C-130 Hercules", "A400M Atlas", "Bell 429 Polis Heli", "T-70 Genel Maksat", "Kamikaze İHA", "Mini İHA (Gözcü)", "CN-235 Sahil Güvenlik", "AW139 Arama Kurtarma", "Gökbey", "VIP Jet", "Yangın Söndürme Uçağı", "KC-135R İkmal Uçağı"], 
                teams: ["Otonom (İHA/Ekipsiz)", "Hava İndirme Komando", "SAR Hava Kurtarma", "Özel Kuvvetler (MAK)", "Medikal Ekip", "Kargo İkmal Ekibi", "Hava Trafik", "Uçuş Bakım Timi", "Aviyonik Onarım", "Elektronik Harp Timi", "İstihbarat Analiz", "Meteoroloji", "Kurtarma Yüzücüleri", "Hava İtfaiye", "Yakıt İkmal Takımı", "Kaza Kırım Ekibi", "VIP Koruma", "Paraşütçü", "Silah Sistemleri", "Yakıt İkmal Takımı"], 
                personnel: [{r: "İHA Pilotu", c: 18}, {r: "TİHA Subayı", c: 8}, {r: "Helikopter Pilotu", c: 24}, {r: "Uçak Teknisyeni", c: 45}, {r: "Aviyonik Uzmanı", c: 15}, {r: "Hava Kontrol", c: 12}, {r: "Meteoroloji Subayı", c: 6}, {r: "Komando", c: 50}, {r: "SAR Dalgıcı", c: 15}, {r: "JP-8 İkmalcisi", c: 10}, {r: "Yükleme Uzmanı", c: 14}, {r: "İstihbarat Analisti", c: 12}, {r: "Uçuş Tabibi", c: 8}, {r: "Radar Operatörü", c: 10}, {r: "Paraşüt Teknisyeni", c: 6}, {r: "Helikopter Makinisti", c: 20}, {r: "Uçuş Görevlisi", c: 30}, {r: "İletişim Subayı", c: 8}, {r: "Silah Teknisyeni", c: 10}, {r: "Kaza Kırım", c: 4}] 
            },
            logistics: { 
                name: "TÜRKİYE AFET LOJİSTİK MERKEZİ", isAir: false, 
                missions: ["Çadır Kent Kurulumu", "Sıcak Yemek Dağıtımı", "Tıbbi Malzeme İkmali", "Temiz Su Tedariği", "Kuru Gıda Nakli", "Seyyar Tuvalet", "Jeneratör Sevkiyatı", "Giysi Dağıtımı", "Isıtıcı Dağıtımı", "Battaniye İkmali", "Bebek Maması Dağıtımı", "Sahra Hastanesi Kurulumu", "Kan Ünitesi Taşıma", "Mobil Fırın", "Lojistik Depo", "Moloz Taşıma", "Atık Yönetimi", "Cenaze Taşıma", "Gönüllü Transferi", "Hasarlı Araç Çekimi"], 
                payloads: ["10.000 Kap Yemek", "Seyyar Sahra Hastanesi", "Konteyner Barınma", "5.000 Battaniye", "10 Ton İçme Suyu", "Mobil Fırın Modülü", "Kan Üniteleri", "2.000 Aile Çadırı", "Seyyar WC Kabini", "Endüstriyel Isıtıcı", "Hijyen Setleri", "Bebek Malzemesi", "Acil İlaç", "Jeneratör (200kVA)", "Projektör", "Defin Malzemeleri", "Ağır İş Makineleri", "Yedek Parça", "Akaryakıt Varilleri", "Gıda Kolisi"], 
                vehicles: ["Mobil Aşevi Tırı", "Kızılay Kan Aracı", "Ağır Yük Tırı", "Frigofirik Kamyon", "Konteyner Tırı", "Forklift (5 Ton)", "Teleskopik Yükleyici", "Kapalı Kamyonet", "Su Tankeri", "Vidanjör", "Mobil Fırın Aracı", "İletişim Aracı", "Seyyar Hastane Tırı", "Mobil Çamaşırhane", "Cenaze Aracı", "VIP Minibüs", "Personel Otobüsü", "Çöp Kamyonu", "Depo Aracı", "Kargo Aracı"], 
                teams: ["Kızılay İkmal Ekibi", "Sahra Hastanesi Kurulum", "Çadır Kent Yöneticileri", "Aşevi Aşçıları", "Su Dağıtım", "Tıbbi Lojistik", "Soğuk Zincir", "Depo Sayım", "İndirme/Bindirme", "Hijyen Timi", "Atık Yönetim", "Gıda Güvenliği", "İletişim Ekibi", "Filo Yöneticileri", "Gümrük Timi", "Kayıt Ekibi", "Hasar Onarım", "Jeneratör Bakım", "Gönüllü Koordinatörleri", "Psikososyal Lojistik"], 
                personnel: [{r: "Tır Şoförü", c: 50}, {r: "Forklift Operatörü", c: 30}, {r: "Depo Uzmanı", c: 15}, {r: "Soğuk Zincir", c: 10}, {r: "Kızılay Aşçısı", c: 40}, {r: "Çadır Kurulum", c: 60}, {r: "Biyomedikal Tek.", c: 8}, {r: "İkmal Subayı", c: 6}, {r: "Gümrük Uzmanı", c: 5}, {r: "Jeneratör Tek.", c: 14}, {r: "Gıda Mühendisi", c: 6}, {r: "Su Tesisatçısı", c: 20}, {r: "Elektrik Teknisyeni", c: 25}, {r: "Vinç Operatörü", c: 12}, {r: "Veri Giriş", c: 15}, {r: "Lojistik Müdürü", c: 4}, {r: "Hijyen Uzmanı", c: 10}, {r: "Araç Bakım", c: 18}, {r: "Otobüs Şoförü", c: 20}, {r: "Ambulans Şoförü", c: 15}] 
            },
            fuel: { 
                name: "STRATEJİK ANA YAKIT TESİSİ", isAir: false, 
                missions: ["Jeneratör Yakıt İkmali", "Araç Filosu İkmali", "Tehlikeli Madde Nakli", "Helikopter Yakıt", "İHA JP-8 İkmali", "Rafineri Transferi", "Boru Hattı Güvenliği", "Sızıntı Müdahalesi", "KBRN Temizliği", "Yangın Söndürme", "Basınçlı Gaz", "Oksijen Tüpü", "Deniz İkmali", "Mobil Tanker", "Kalite Kontrol", "Pompa Onarımı", "Patlayıcı Transferi", "Çevre Güvenliği", "Kaçak Tespiti", "Depo İzolasyonu"], 
                payloads: ["50.000 Litre Motorin", "30.000 Litre Benzin", "Yüksek Basınçlı Gaz", "JP-8 Jet Yakıtı", "Havacılık Benzini", "Oksijen Tüpleri", "Söndürme Köpüğü", "Sızıntı Bariyeri", "Mobil Pompa", "KBRN Dekontaminasyon", "Azot Tüpleri", "Doğalgaz Regülatörü", "Yedek Tank", "Analiz Kiti", "Patlayıcı C4", "Yangın Tozu", "Madeni Yağ", "Antifriz", "Asit Nötralize", "Basınç Valfleri"], 
                vehicles: ["Akaryakıt Tankeri", "Mobil Jeneratör", "Uçak Yakıt Tankeri", "Taktik Tanker", "LPG Taşıma Tırı", "Oksijen Kamyoneti", "ADR Tırı", "Köpük Kulesi", "Sızıntı Müdahale", "Vidanjör", "Mobil Laboratuvar", "Devriye Aracı", "Zırhlı Araç", "Tehlikeli Madde Forklift", "İtfaiye Öncü", "Acil Kurtarma", "KBRN Tespit", "Transfer Pompası", "Endüstriyel Vinç", "Personel Aracı"], 
                teams: ["ADR Timi", "Mobil İkmal Ekibi", "JP-8 Takımı", "Tesis İtfaiyesi", "Sızıntı Önleme", "Gaz Ölçüm", "Rafineri Bakım", "Boru Hattı Güvenlik", "Oksijen Tedarik", "Laboratuvar Timi", "Tesis Koruma", "Valf Teknisyenleri", "Yangın Uzmanları", "Çevre Müdahale", "Patlayıcı İmha", "Sevkiyat Planlama", "Kriz Yönetim", "Araç Bakım", "İSG Uzmanları", "İrtibat Takımı"], 
                personnel: [{r: "Pompa Operatörü", c: 25}, {r: "ADR Şoförü", c: 40}, {r: "Rafineri Uzmanı", c: 8}, {r: "Köpük Uzmanı", c: 15}, {r: "Tank Teknisyeni", c: 10}, {r: "Jeneratör Ekibi", c: 12}, {r: "Tesis Güvenlik", c: 20}, {r: "KBRN Uzmanı", c: 14}, {r: "Kimyager", c: 8}, {r: "Sızıntı Ekibi", c: 22}, {r: "İSG Uzmanı", c: 6}, {r: "Uçak İkmalcisi", c: 15}, {r: "Valf Ustası", c: 18}, {r: "Operatör", c: 6}, {r: "Tamirci", c: 10}, {r: "İtfaiye Şoförü", c: 12}, {r: "Atık Uzmanı", c: 4}, {r: "Sevkiyat Yöneticisi", c: 5}, {r: "Çevre Mühendisi", c: 4}, {r: "İlkyardımcı", c: 10}] 
            },
            coastguard: {
                name: "SAMSUN SAHİL GÜVENLİK KOMUTANLIĞI", isSea: true, isAir: false,
                missions: ["Denizde Arama Kurtarma (SAR)", "Kayıp Şahıs (Deniz)", "Boğulma Vakası", "Gemi Yangını/Kaza", "Düzensiz Göçmen Müdahale", "Deniz Kirliliği / Sızıntı", "Kaçakçılık Operasyonu", "Tıbbi Tahliye (MEDEVAC)", "Fırtına Sürüklenme", "Sualtı Enkaz Arama", "Şüpheli Paket/Mayın", "Deniz Haydutluğu", "Yasadışı Su Ürünleri Avcılığı", "Kıyı Emniyeti Desteği", "VIP Deniz Koruması", "Denizaltı Kurtarma Desteği", "Buzlanma/Donma Müdahale", "Sel Suları Tahliyesi (Zodyak)", "Liman Güvenliği", "Tatbikat / Devriye"],
                payloads: ["Can Yeleği ve Simidi", "Sualtı ROV (Gözlem Aracı)", "Termal Kamera (FLIR)", "Dalgıç Ekipmanı", "Yangın Söndürme Topu", "Ağır Çekme Halatı", "Petrol Bariyeri (Sızıntı)", "İlk Yardım ve Oksijen Kiti", "Deniz Yüzeyi Aydınlatma", "Acil Durum Şamandırası", "KBRN Koruyucu Dalış Kıyafeti", "Sualtı Kesici/Kaynak", "Sonar (Yandan Taramalı)", "Şişme Can Salı", "Megafon ve Siren Sistemi", "Deniz Suyu Arıtma Kiti", "Balistik Zırh (SÖH)", "Sinyal Fişeği", "Yedek Yakıt Tankı", "Telsiz Röle Sistemi"],
                vehicles: ["TCSG Arama Kurtarma Gemisi", "Hızlı Müdahale Botu", "Zodyak Bot (Şişme)", "Sahil Güvenlik Helikopteri (AB-412)", "İnsansız Deniz Aracı (İDA)", "Arama Kurtarma Uçağı (CN-235)", "Dalgıç İntikal Aracı", "Devriye Gemisi", "Amfibik Hovercraft", "Yangın Söndürme Römorkörü", "Cankurtaran Jeti (Jetski)", "Liman Kontrol Botu", "Sahil Devriye Aracı (4x4)", "Mobil Radar Aracı", "Sualtı Kurtarma Çanı", "Deniz Kirliliği Müdahale Gemisi", "Lojistik Destek Gemisi", "Kayıt/Kabul Teknesi", "Taktik İHA (Gemi Konuşlu)", "Tahliye Feribotu"],
                teams: ["Dalgıç Emniyet (DEGAK)", "Sualtı Taarruz/Özel Harekat", "Arama Kurtarma Mürettebatı", "İlk Yardım Tıbbi Ekip", "Deniz Kirliliği Uzmanları", "Gemi Yangın Müdahale Timi", "Gemi Zapt ve Müsadere", "Helikopter Uçuş Ekibi", "İnsansız Sistem (İDA) Operatörleri", "Sonar ve Radar Analistleri", "Kıyı Koordinasyon Ekibi", "Gemi Makine Teknisyenleri", "Seyir ve İntikal Subayları", "Psikolojik Destek Ekibi", "Hukuk ve Kayıt İşlemleri", "Liman Güvenlik Muhafızları", "KBRN Deniz Timi", "Halkla İlişkiler ve Basın", "Lojistik İkmal Ekibi", "Meteoroloji Uzmanları"],
                personnel: [{r: "Gemi Komutanı", c: 12}, {r: "Dalgıç", c: 35}, {r: "Radar Operatörü", c: 20}, {r: "Helikopter Pilotu", c: 8}, {r: "Özel Harekatçı", c: 40}, {r: "Gemi Makinisti", c: 25}, {r: "Sıhhiye/İlk Yardım", c: 18}, {r: "ROV Operatörü", c: 10}, {r: "Telsiz Operatörü", c: 15}, {r: "Bot Kaptanı", c: 30}, {r: "Cankurtaran", c: 50}, {r: "Kirlilik Uzmanı", c: 8}, {r: "Sonar Uzmanı", c: 12}, {r: "İDA Pilotu", c: 6}, {r: "Gemi Aşçısı", c: 10}, {r: "Silah Uzmanı", c: 14}, {r: "Meteorolog", c: 4}, {r: "Kıyı Güvenlik", c: 45}, {r: "Dalgıç Teknisyeni", c: 12}, {r: "Lojistik Subayı", c: 8}]
            }
        };

        function login() { 
            if(document.getElementById('sicil').value === "1919") { 
                document.getElementById('login-screen').style.display='none'; document.getElementById('app').style.display='block'; initMap(); 
            } else { alert("Erişim Reddedildi!"); }
        }

        function initMap() {
            map = L.map('map', { zoomControl: false, attributionControl: false }).setView([41.2800, 36.3300], 12);
            baseLayers.dark = L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png').addTo(map);
            baseLayers.satellite = L.tileLayer('https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}');
            baseLayers.normal = L.tileLayer('https://mt1.google.com/vt/lyrs=m&x={x}&y={y}&z={z}');
            baseLayers.traffic = L.tileLayer('https://mt1.google.com/vt/lyrs=m,traffic&x={x}&y={y}&z={z}');

            osrmRouter = L.Routing.osrmv1({ language: 'tr', profile: 'driving' });
            tempRouteGroup = L.layerGroup().addTo(map);

            L.marker(POS.afad, { icon: L.divIcon({ html: '<div style="background:var(--warning); width:35px; height:35px; border-radius:8px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; box-shadow:0 0 15px rgba(245,158,11,0.8); cursor:pointer;"><i class="fas fa-shield-alt" style="color:#000; font-size:16px;"></i></div>', className:''}) }).addTo(map).on('click', () => openHQ('afad'));
            L.marker(POS.air, { icon: L.divIcon({ html: '<div style="background:var(--primary); width:35px; height:35px; border-radius:8px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; box-shadow:0 0 15px rgba(59,130,246,0.8); cursor:pointer;"><i class="fas fa-plane" style="font-size:16px;"></i></div>', className:''}) }).addTo(map).on('click', () => openHQ('air'));
            L.marker(POS.logistics, { icon: L.divIcon({ html: '<div style="background:#10b981; width:35px; height:35px; border-radius:8px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; box-shadow:0 0 15px rgba(16,185,129,0.8); cursor:pointer;"><i class="fas fa-box" style="font-size:16px;"></i></div>', className:''}) }).addTo(map).on('click', () => openHQ('logistics'));
            L.marker(POS.fuel, { icon: L.divIcon({ html: '<div style="background:#f97316; width:35px; height:35px; border-radius:8px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; box-shadow:0 0 15px rgba(249,115,22,0.8); cursor:pointer;"><i class="fas fa-gas-pump" style="font-size:16px;"></i></div>', className:''}) }).addTo(map).on('click', () => openHQ('fuel'));
            
            // YENİ: Sahil Güvenlik Markeri (Samsun Limanı)
            L.marker(POS.coastguard, { icon: L.divIcon({ html: '<div style="background:var(--navy); width:35px; height:35px; border-radius:8px; border:2px solid #fff; display:flex; align-items:center; justify-content:center; box-shadow:0 0 15px rgba(14,165,233,0.8); cursor:pointer;"><i class="fas fa-anchor" style="color:#fff; font-size:16px;"></i></div>', className:''}) }).addTo(map).on('click', () => openHQ('coastguard'));

            TR_STATIONS.assembly.forEach(pos => { L.marker(pos, { icon: L.divIcon({ html: '<div class="icon-assembly"><i class="fas fa-campground"></i></div>', className:''}) }).addTo(map).on('click', () => openDetail('Bölgesel Toplanma Alanı', 'assembly')); });
            map.on('click', handleMapClick);
            log("SİSTEM: Mavi Vatan Deniz Hakimiyet Ağı aktifleştirildi. Gemiler denizde direkt rotalanacaktır.");
        }

        function updateActiveUnitsPanel() {
            const panel = document.getElementById('fleet-panel'); const list = document.getElementById('fleet-list'); list.innerHTML = '';
            const keys = Object.keys(activeUnits);
            if (keys.length === 0) { panel.style.display = 'none'; list.innerHTML = '<div style="color:#64748b; font-size:11px; text-align:center;">Şu an sahada aktif araç yok.</div>'; return; }
            panel.style.display = 'block';
            keys.forEach(id => {
                const u = activeUnits[id];
                let sColor = u.state === 'returning' ? '#f59e0b' : (u.state === 'loitering' ? 'var(--recon)' : (u.state === 'completed' || u.state === 'securing' ? '#10b981' : (u.isAir?'#8b5cf6':'#3b82f6')));
                if(u.hqKey === 'coastguard' && u.state === 'outbound') sColor = 'var(--navy)';
                let sText = u.state === 'returning' ? 'ÜSSE DÖNÜYOR' : (u.state === 'loitering' ? 'KEŞİFTE' : (u.state === 'completed' ? 'TAMAMLANDI' : (u.state === 'securing' ? 'BLOKE ETTİ' : 'GÖREVDE')));
                list.innerHTML += `<div class="fleet-item" onclick="focusUnit('${id}')"><div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:5px;"><b style="color:${sColor}; font-size:12px;">${u.v}</b><span style="background:${sColor}; color:#000; padding:2px 6px; border-radius:4px; font-weight:900; font-size:9px;">${sText}</span></div><div style="color:#94a3b8; font-size:10px;">Yakıt: %${u.fuel.toFixed(1)} | Ekip: ${u.name}</div></div>`;
            });
        }

        function focusUnit(id) { const u = activeUnits[id]; if(u) { map.panTo(u.marker.getLatLng()); updateHUD(u); log(`SİSTEM: Odak noktası değiştirildi -> ${u.v}`); } }

        function advanceTime(h) {
            simHour += h; let day = Math.floor(simHour / 24) + 1; let hr = simHour % 24; document.getElementById('clock-display').innerHTML = `${hr.toString().padStart(2, '0')}:00 <span class="clock-day">GÜN ${day}</span>`; log(`ZAMAN: Saat ilerletildi. Gün ${day}, Saat ${hr.toString().padStart(2, '0')}:00`);
            if (currentDisasterLoc) {
                if (simHour >= 1 && !phasesTriggered.p1) { triggerAIApproval(1); phasesTriggered.p1 = true; }
                else if (simHour >= 24 && !phasesTriggered.p2) { triggerAIApproval(2); phasesTriggered.p2 = true; }
                else if (simHour >= 48 && !phasesTriggered.p3) { triggerAIApproval(3); phasesTriggered.p3 = true; }
            }
        }

        function generateSelectHTML(options, selectedValue, idPrefix) { let html = `<select id="${idPrefix}" class="ai-mini-select">`; options.forEach(opt => { let sel = opt === selectedValue ? 'selected' : ''; html += `<option value="${opt}" ${sel}>${opt}</option>`; }); return html + `</select>`; }

        function triggerAIApproval(phase) {
            document.getElementById('ai-approval-modal').style.display = 'block'; const container = document.getElementById('ai-plan-container'); container.innerHTML = '';
            let titleText = ""; let planItems = []; document.getElementById('ai-badge').innerHTML = `<i class="fas fa-clock"></i> CHRONOS AI`; document.getElementById('ai-subtitle').innerText = `ZAMAN BAZLI OTONOM SEVKİYAT PLANI`; document.getElementById('ai-plan-desc').innerHTML = `Sistem afetin gerçekleştiği zamana bağlı olarak aşağıdaki kurum ve araçların bölgeye intikal etmesini tavsiye etmektedir. <b>Parametreleri menülerden düzenleyebilir</b> ve operasyonu başlatabilirsiniz.`;

            if (phase === 1) { titleText = "0-24 SAAT (KRİTİK ARAMA KURTARMA & KEŞİF)"; planItems = [ { id:'ai-1', hq: 'afad', v: '4x4 Kurtarma Aracı', t: 'AFAD Ağır Kurtarma', m: 'Arama-Kurtarma (SAR)', p: 'Ağır Kurtarma Teçhizatı' }, { id:'ai-2', hq: 'air', v: 'Bayraktar TB2 İHA', t: 'Otonom (İHA/Ekipsiz)', m: 'Keşif / Gözetleme (ISR)', p: 'Elektro-Optik (EO/IR)' } ]; } 
            else if (phase === 2) { titleText = "24-48 SAAT (BARINMA, LOJİSTİK & İKMAL)"; planItems = [ { id:'ai-1', hq: 'logistics', v: 'Ağır Yük Tırı', t: 'Çadır Kent Yöneticileri', m: 'Çadır Kent Kurulumu', p: '2.000 Aile Çadırı' }, { id:'ai-2', hq: 'fuel', v: 'Akaryakıt Tankeri', t: 'Mobil İkmal Ekibi', m: 'Jeneratör Yakıt İkmali', p: '50.000 Litre Motorin' } ]; } 
            else if (phase === 3) { titleText = "48-72 SAAT (REHABİLİTASYON & AĞIR ENKAZ)"; planItems = [ { id:'ai-1', hq: 'logistics', v: 'Ağır Yük Tırı', t: 'Hijyen Timi', m: 'Seyyar Tuvalet', p: 'Seyyar WC Kabini' }, { id:'ai-2', hq: 'afad', v: 'Mini Ekskavatör', t: 'AFAD Ağır Kurtarma', m: 'Enkaz Kaldırma', p: 'Ağır Kurtarma Teçhizatı' } ]; }
            
            // Eğer afet Denizdeyse 1. Fazda Sahil Güvenlik öner
            const dType = document.getElementById('disaster-select').value;
            if (phase === 1 && dType.includes('Deniz')) {
                titleText = "0-24 SAAT (DENİZDE ARAMA KURTARMA - SAR)";
                planItems = [ { id:'ai-1', hq: 'coastguard', v: 'TCSG Arama Kurtarma Gemisi', t: 'Arama Kurtarma Mürettebatı', m: 'Denizde Arama Kurtarma (SAR)', p: 'Termal Kamera (FLIR)' }, { id:'ai-2', hq: 'air', v: 'Bayraktar TB2 İHA', t: 'Otonom (İHA/Ekipsiz)', m: 'Deniz Yüzeyi Arama', p: 'Elektro-Optik (EO/IR)' } ];
            }

            document.getElementById('ai-phase-title').innerText = `🕒 SİSTEM TAVSİYESİ: ${titleText}`;
            planItems.forEach(item => { const conf = HQ_CONFIG[item.hq]; container.innerHTML += `<div class="ai-plan-item" data-hq="${item.hq}" data-id="${item.id}"><div class="ai-plan-hq"><i class="fas fa-building"></i> ${conf.name} ÇIKIŞLI BİRİM</div><div style="display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px;"><div><div style="font-size:10px; color:#94a3b8;">ARAÇ:</div>${generateSelectHTML(conf.vehicles, item.v, `${item.id}-v`)}</div><div><div style="font-size:10px; color:#94a3b8;">EKİP:</div>${generateSelectHTML(conf.teams, item.t, `${item.id}-t`)}</div><div><div style="font-size:10px; color:#94a3b8;">FAYDALI YÜK:</div>${generateSelectHTML(conf.payloads, item.p, `${item.id}-p`)}</div></div></div>`; });
        }

        function generateReconReport(u) {
            const riskLevels = ["KRİTİK (KIRMIZI KOD)", "YÜKSEK (SARI KOD)"]; const r = riskLevels[Math.floor(Math.random() * riskLevels.length)];
            document.getElementById('ai-approval-modal').style.display = 'block'; document.getElementById('ai-phase-title').innerText = `📡 SAHA İSTİHBARAT RAPORU: ${r}`; document.getElementById('ai-badge').innerHTML = `<i class="fas fa-satellite"></i> ISR REPORT`; document.getElementById('ai-subtitle').innerText = `İHA KEŞİF BİLDİRİMİ VE DESTEK TALEBİ`; document.getElementById('ai-plan-desc').innerHTML = `<b>${u.v}</b> unsuru tarafından yapılan havadan keşifte hedeflenen bölgede <b>${r}</b> risk tespit edilmiştir. Sistem hedeflenen tam koordinata (Keşif Noktası) acil müdahale ekiplerinin sevkini önermektedir.`;
            const container = document.getElementById('ai-plan-container'); container.innerHTML = '';
            
            // Eğer deniz üzerindeyse Sahil Güvenlik desteği iste
            let planItems = [];
            if (u.hqKey === 'coastguard' || document.getElementById('disaster-select').value.includes('Deniz')) {
                planItems = [ { id:'recon-1', hq: 'coastguard', v: 'Hızlı Müdahale Botu', t: 'Dalgıç Emniyet (DEGAK)', m: 'Denizde Arama Kurtarma (SAR)', p: 'Sualtı ROV (Gözlem Aracı)' }, { id:'recon-2', hq: 'afad', v: 'Tam Donanımlı Ambulans', t: 'UMKE Medikal', m: 'Tıbbi Triyaj', p: 'İlk Yardım ve Oksijen Kiti' } ];
            } else {
                planItems = [ { id:'recon-1', hq: 'afad', v: '4x4 Kurtarma Aracı', t: 'AFAD Ağır Kurtarma', m: 'Arama-Kurtarma (SAR)', p: 'Termal Arama Sensörü' }, { id:'recon-2', hq: 'logistics', v: 'Ağır Yük Tırı', t: 'Sahra Hastanesi Kurulum', m: 'Sahra Hastanesi Kurulumu', p: 'Seyyar Sahra Hastanesi' } ];
            }

            planItems.forEach(item => { const conf = HQ_CONFIG[item.hq]; container.innerHTML += `<div class="ai-plan-item" data-hq="${item.hq}" data-id="${item.id}" style="border-left-color:var(--danger);"><div class="ai-plan-hq" style="color:var(--danger);"><i class="fas fa-ambulance"></i> ACİL DESTEK: ${conf.name}</div><div style="display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px;"><div><div style="font-size:10px; color:#94a3b8;">ARAÇ:</div>${generateSelectHTML(conf.vehicles, item.v, `${item.id}-v`)}</div><div><div style="font-size:10px; color:#94a3b8;">EKİP:</div>${generateSelectHTML(conf.teams, item.t, `${item.id}-t`)}</div><div><div style="font-size:10px; color:#94a3b8;">FAYDALI YÜK:</div>${generateSelectHTML(conf.payloads, item.p, `${item.id}-p`)}</div></div></div>`; });
            currentDisasterLoc = [u.targetLat, u.targetLng];
        }

        function generateFuelReport(u) {
            document.getElementById('ai-approval-modal').style.display = 'block'; document.getElementById('ai-phase-title').innerText = `⚠️ KRİTİK YAKIT UYARISI`; document.getElementById('ai-badge').innerHTML = `<i class="fas fa-gas-pump"></i> FUEL ALERT`; document.getElementById('ai-subtitle').innerText = `HAVADA YAKIT İKMAL TALEBİ`; document.getElementById('ai-plan-desc').innerHTML = `<b>${u.v}</b> unsurunun yakıt seviyesi kritik eşiğe (<b>%50</b>) düşmüştür. Sistemin kesintisiz ISR görevine devam edebilmesi için Hava Üssünden <b>KC-135R İkmal Uçağı</b> sevk edilmesi tavsiye edilmektedir.`;
            const container = document.getElementById('ai-plan-container'); container.innerHTML = ''; let planItems = [ { id:'fuel-1', hq: 'air', v: 'KC-135R İkmal Uçağı', t: 'Yakıt İkmal Takımı', m: 'Havadan İkmal Atımı', p: 'JP-8 Havada İkmal Yakıtı' } ];
            planItems.forEach(item => { const conf = HQ_CONFIG[item.hq]; container.innerHTML += `<div class="ai-plan-item" data-hq="${item.hq}" data-id="${item.id}" style="border-left-color:var(--warning);"><div class="ai-plan-hq" style="color:var(--warning);"><i class="fas fa-plane"></i> ACİL İKMAL: ${conf.name}</div><div style="display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px;"><div><div style="font-size:10px; color:#94a3b8;">ARAÇ:</div>${generateSelectHTML(conf.vehicles, item.v, `${item.id}-v`)}</div><div><div style="font-size:10px; color:#94a3b8;">EKİP:</div>${generateSelectHTML(conf.teams, item.t, `${item.id}-t`)}</div><div><div style="font-size:10px; color:#94a3b8;">FAYDALI YÜK:</div>${generateSelectHTML(conf.payloads, item.p, `${item.id}-p`)}</div></div></div>`; });
            currentDisasterLoc = [u.marker.getLatLng().lat, u.marker.getLatLng().lng];
        }

        function rejectAIPlan() { document.getElementById('ai-approval-modal').style.display = 'none'; log("KOMUTA: Sistem tavsiyesi reddedildi."); }

        function approveAIPlan() {
            document.getElementById('ai-approval-modal').style.display = 'none'; const items = document.querySelectorAll('.ai-plan-item');
            items.forEach(el => { const id = el.getAttribute('data-id'); const hq = el.getAttribute('data-hq'); const v = document.getElementById(`${id}-v`).value; const t = document.getElementById(`${id}-t`).value; const p = document.getElementById(`${id}-p`).value; aiDeployQueue.push({ hqKey: hq, isAir: HQ_CONFIG[hq].isAir, vehicle: v, team: t, mission: 'Tavsiye/Destek Görevi', payload: p, priority: 'Kritik (CODE RED)' }); });
            log(`KOMUTA: Plan onaylandı. Otonom ardışık sevk başlıyor...`); processDeployQueue();
        }

        function processDeployQueue() { if(aiDeployQueue.length === 0 || isAutoDeploying) return; isAutoDeploying = true; let req = aiDeployQueue.shift(); pendingDeployment = req; executeRoutingPhase(currentDisasterLoc, false, true); }
        function switchMap(type) { map.removeLayer(baseLayers[currentL]); baseLayers[type].addTo(map); currentL = type; ['dark', 'sat', 'norm', 'traf'].forEach(id => { document.getElementById('btn-' + id).classList.remove('active'); }); let btnId = type === 'satellite' ? 'sat' : type === 'normal' ? 'norm' : type === 'traffic' ? 'traf' : 'dark'; document.getElementById('btn-' + btnId).classList.add('active'); }
        function setRouteMode(mode) { routeMode = mode; ['auto', 'man'].forEach(id => { document.getElementById('btn-' + id).classList.remove('active'); }); document.getElementById('btn-' + (mode==='manual'?'man':'auto')).classList.add('active'); tempPoints = []; tempRouteGroup.clearLayers(); if(document.getElementById('btn-calc-route')) document.getElementById('btn-calc-route').remove(); }
        function closeModal(id) { document.getElementById(id).style.display='none'; }
        function openDetail(title, type) { document.getElementById('detail-modal').style.display='block'; document.getElementById('d-title').innerText = title; const dataArray = STATION_DATA[type] || STATION_DATA.assembly; document.getElementById('d-content').innerHTML = dataArray.map(d => `<div class="stat-item"><b>${d.k}</b>${d.v}</div>`).join(''); }

        function openHQ(hqKey) {
            const hq = HQ_CONFIG[hqKey]; document.getElementById('hq-modal').style.display='block'; document.getElementById('hq-name').innerText = hq.name;
            const fillSelect = (id, list) => { const el = document.getElementById(id); el.innerHTML = '<option value="">-- Seçiniz --</option>'; list.forEach(item => el.innerHTML += `<option value="${item}">${item}</option>`); };
            fillSelect('hq-mission', hq.missions); fillSelect('hq-payload', hq.payloads); fillSelect('hq-vehicle', hq.vehicles); fillSelect('hq-team', hq.teams);
            const pGrid = document.getElementById('hq-personnel'); pGrid.innerHTML = ''; hq.personnel.forEach(p => { pGrid.innerHTML += `<div class="role-item"><span>${p.r}</span> <span class="role-count">${p.c}</span></div>`; });
            pendingDeployment.hqKey = hqKey; pendingDeployment.isAir = hq.isAir; 
        }

        function prepareDeployment() {
            const v = document.getElementById('hq-vehicle').value; const t = document.getElementById('hq-team').value;
            if(!v) { alert("Araç seçimi zorunludur!"); return; } if(!t && !v.includes('İHA') && !v.includes('TİHA') && !v.includes('Uçağı') && !v.includes('İDA')) { alert("Ekip seçimi ZORUNLUDUR!"); return; }
            pendingDeployment.vehicle = v; pendingDeployment.team = t || 'Otonom Sistem'; pendingDeployment.mission = document.getElementById('hq-mission').value || 'Genel Görev'; pendingDeployment.payload = document.getElementById('hq-payload').value || 'Standart'; pendingDeployment.priority = document.getElementById('hq-priority').value;
            closeModal('hq-modal'); tempPoints = [];
            if(routeMode === 'auto') { if(!currentDisasterLoc) { alert("Haritada işaretli afet/vaka yok!"); return; } executeRoutingPhase(currentDisasterLoc, false, false); } else { interactionMode = 'routing_target'; map.getContainer().style.cursor = 'crosshair'; log(`KOMUTA: ${v} aracı çoklu hedef ataması için hazır. Haritaya noktaları tıklayın.`); }
        }

        function setMode(m) { interactionMode = m; if(m === 'evacuation') { evacPoints = []; tempRouteGroup.clearLayers(); } map.getContainer().style.cursor = 'crosshair'; if(m==='closure') log("KOMUTA: Kapatılacak yolu haritadan tıklayarak seçin."); }

        function autoDispatchPoliceToBlock(targetPos) {
            const startPos = POS.afad; const waypoints = [L.Routing.waypoint(L.latLng(startPos[0], startPos[1])), L.Routing.waypoint(L.latLng(targetPos[0], targetPos[1]))];
            osrmRouter.route(waypoints, function(err, routes) {
                let coords = []; if(err || routes.length === 0) { for(let i=0; i<=100; i++) coords.push([ startPos[0] + (targetPos[0] - startPos[0]) * (i/100), startPos[1] + (targetPos[1] - startPos[1]) * (i/100) ]); } else { coords = routes[0].coordinates.map(c => [c.lat, c.lng]); }
                const id = 'P-' + Math.floor(Math.random()*9000); const m = L.marker(coords[0], { icon: L.divIcon({ html: `<div style="background:#0ea5e9; width:28px; height:28px; border-radius:50%; border:2px solid #fff; display:flex; align-items:center; justify-content:center; color:#fff; box-shadow:0 0 15px #0ea5e9;"><i class="fas fa-car-side"></i></div>`, className:''}) }).addTo(map);
                activeUnits[id] = { marker: m, path: coords, seg: 0, prog: 0, fuel: 100, isAir: false, hqKey: 'afad', name: 'EGM Trafik/Asayiş', v: 'Polis Müdahale Otosu', mission: 'Çevre Güvenliği Sağlama', payload: 'Bariyer & Şerit', priority: 'Kritik (CODE RED)', state: 'outbound_police' };
                setInterval(() => moveUnit(id), 50); log(`SİSTEM: EGM Polis Otosu otonom olarak sevk edildi.`); updateActiveUnitsPanel();
            });
        }

        function handleMapClick(e) {
            const ll = [e.latlng.lat, e.latlng.lng];
            if(interactionMode === 'vaka') {
                const dType = document.getElementById('disaster-select').value; L.circle(e.latlng, { radius: 1000, color: 'var(--danger)', fillOpacity: 0.3 }).addTo(map); L.marker(e.latlng, { icon: L.divIcon({ html: `<div style="background:var(--danger); color:#fff; font-size:10px; font-weight:bold; padding:2px 5px; border-radius:4px; white-space:nowrap;">${dType}</div>`, className:''}) }).addTo(map);
                currentDisasterLoc = ll; log(`VAKA: ${dType} işaretlendi.`); interactionMode = 'idle'; map.getContainer().style.cursor = 'grab';
            } else if(interactionMode === 'routing_target') {
                if(routeMode === 'manual') {
                    tempPoints.push(ll); let vColor = pendingDeployment.isAir ? '#8b5cf6' : (pendingDeployment.hqKey==='coastguard'?'#0ea5e9':'#3b82f6'); L.circleMarker(ll, {radius: 6, color: vColor, fillOpacity: 1, weight: 2}).addTo(tempRouteGroup); let previewPoints = [POS[pendingDeployment.hqKey], ...tempPoints]; if(windowTempPreviewLine) tempRouteGroup.removeLayer(windowTempPreviewLine);
                    windowTempPreviewLine = L.polyline(previewPoints, {color: vColor, weight: 3, dashArray: '5,5', opacity: 0.6}).addTo(tempRouteGroup);
                    if(!document.getElementById('btn-calc-route')) { const btn = document.createElement('button'); btn.id = 'btn-calc-route'; btn.innerHTML = `<i class="fas fa-route"></i> ROTAYI HESAPLA VE ÇİZ`; btn.style = "position:absolute; bottom:80px; left:50%; transform:translateX(-50%); z-index:2000; padding:15px 30px; background:var(--primary); color:#fff; border:none; border-radius:10px; font-weight:800; cursor:pointer; font-size:14px; box-shadow:0 10px 30px rgba(59,130,246,0.5);"; btn.onclick = () => { btn.remove(); executeRoutingPhase(null, true, false); interactionMode = 'idle'; map.getContainer().style.cursor = 'grab'; }; document.body.appendChild(btn); }
                } else { executeRoutingPhase(ll, false, false); interactionMode = 'idle'; map.getContainer().style.cursor = 'grab'; }
            } else if(interactionMode === 'closure') {
                const rawType = document.getElementById('closure-select').value; const cType = rawType.split('. ')[1]; L.marker(e.latlng, { icon: L.divIcon({ html: `<div class="road-block-marker"><i class="fas fa-ban"></i> <span>${cType}</span></div>`, className:''}) }).addTo(map); roadObstacles.push(ll); log(`UYARI: ${cType} nedeniyle yol trafiğe kapatıldı!`); interactionMode = 'idle'; map.getContainer().style.cursor = 'grab'; autoDispatchPoliceToBlock(ll);
            } else if(interactionMode === 'evacuation') {
                evacPoints.push(ll); L.circleMarker(ll, {radius: 6, color: '#10b981', fillOpacity: 1, weight: 2, color: '#fff'}).addTo(tempRouteGroup);
                if(evacPoints.length > 1) { if(windowTempEvacLine) tempRouteGroup.removeLayer(windowTempEvacLine); windowTempEvacLine = L.polyline(evacPoints, {color: '#10b981', weight: 8, dashArray: '15, 15', opacity: 0.7}).addTo(tempRouteGroup); }
                if(!document.getElementById('btn-confirm-evac')) { const btn = document.createElement('button'); btn.id = 'btn-confirm-evac'; btn.innerHTML = `<i class="fas fa-check-double"></i> KORİDORU ONAYLA VE AÇ`; btn.style = "position:absolute; bottom:80px; left:50%; transform:translateX(-50%); z-index:2000; padding:15px 30px; background:var(--success); color:#fff; border:none; border-radius:10px; font-weight:800; cursor:pointer; font-size:14px; box-shadow:0 10px 30px rgba(16,185,129,0.5);"; btn.onclick = () => { btn.remove(); interactionMode='idle'; map.getContainer().style.cursor='grab'; if(evacPoints.length > 1) L.polyline(evacPoints, {color: '#10b981', weight: 8, dashArray: '15, 15', opacity: 0.7, lineCap: 'round'}).addTo(map); tempRouteGroup.clearLayers(); evacPoints = []; log("SİSTEM: Güvenli Tahliye Koridoru sivil geçişe açıldı."); }; document.body.appendChild(btn); }
            }
        }

        function applyObstacleAvoidance(points) {
            if(roadObstacles.length === 0) return points; let newPoints = [points[0]]; let detourAdded = false;
            for (let i = 0; i < points.length - 1; i++) {
                let p1 = points[i]; let p2 = points[i+1]; let obs = roadObstacles[0]; 
                let minLat = Math.min(p1[0], p2[0]) - 0.08; let maxLat = Math.max(p1[0], p2[0]) + 0.08; let minLng = Math.min(p1[1], p2[1]) - 0.08; let maxLng = Math.max(p1[1], p2[1]) + 0.08;
                if(obs[0] > minLat && obs[0] < maxLat && obs[1] > minLng && obs[1] < maxLng && !detourAdded) {
                    let dx = p2[0] - p1[0]; let dy = p2[1] - p1[1]; let len = Math.sqrt(dx*dx + dy*dy) || 0.001; let midLat = (p1[0] + p2[0]) / 2; let midLng = (p1[1] + p2[1]) / 2; let perpLat = -dy / len; let perpLng = dx / len; let dotProd = (obs[0] - midLat) * perpLat + (obs[1] - midLng) * perpLng; let sign = dotProd > 0 ? -1 : 1; 
                    let detourLat = midLat + perpLat * sign * 0.04; let detourLng = midLng + perpLng * sign * 0.04; newPoints.push([detourLat, detourLng]); detourAdded = true;
                    L.marker([detourLat, detourLng], { icon: L.divIcon({ html: `<div class="detour-pulse" style="background:var(--warning); width:18px; height:18px; border-radius:50%; border:2px solid #000; display:flex; justify-content:center; align-items:center;"><div style="position:absolute; left:25px; background:var(--warning); color:#000; padding:4px 8px; border-radius:6px; font-weight:900; font-size:10px; white-space:nowrap; box-shadow:0 0 10px rgba(0,0,0,0.5);">⚠️ OTONOM KAÇIŞ ROTASI</div></div>`, className:'' }) }).addTo(tempRouteGroup);
                }
                newPoints.push(p2);
            }
            return newPoints;
        }

        function executeRoutingPhase(targetPos, isManualChain = false, isAuto = false) {
            const startPos = POS[pendingDeployment.hqKey]; let pointsToRoute = isManualChain ? [startPos, ...tempPoints] : [startPos, targetPos]; tempRouteGroup.clearLayers(); 
            
            // YENİ: Sahil Güvenlik (Deniz) veya Hava Araçları Karayolu (OSRM) KULLANMAZ! Doğrudan giderler.
            if(pendingDeployment.isAir || HQ_CONFIG[pendingDeployment.hqKey].isSea) { 
                drawFallbackRoute(pointsToRoute, isAuto); 
            } else { 
                if(roadObstacles.length > 0 && !isManualChain) { log(`🚨 ANA ARTER KAPALI! Engel Aşma algoritması devrede...`); let safePoints = applyObstacleAvoidance(pointsToRoute); setTimeout(() => { drawRoadRoute(safePoints, isAuto); }, 1500); } else { drawRoadRoute(pointsToRoute, isAuto); }
            }
        }

        let calculatedCoordinates = [];
        function drawRoadRoute(points, isAuto) {
            const waypoints = points.map(p => L.Routing.waypoint(L.latLng(p[0], p[1])));
            osrmRouter.route(waypoints, function(err, routes) { if(err || routes.length === 0) { log("OSRM HATA: Otoyol bulunamadı. Failsafe devrede."); drawFallbackRoute(points, isAuto); } else { calculatedCoordinates = routes[0].coordinates.map(c => [c.lat, c.lng]); showRouteAndConfirm(isAuto); } });
        }

        function drawFallbackRoute(points, isAuto) {
            calculatedCoordinates = []; for (let j = 0; j < points.length - 1; j++) { let pStart = points[j]; let pEnd = points[j+1]; for(let i=0; i<=50; i++) calculatedCoordinates.push([ pStart[0] + (pEnd[0] - pStart[0]) * (i/50), pStart[1] + (pEnd[1] - pStart[1]) * (i/50) ]); } showRouteAndConfirm(isAuto);
        }

        function showRouteAndConfirm(isAuto) {
            // Renk ayarlaması (Sahil güvenlik araçları Mavi Vatan renginde)
            let routeColor = '#3b82f6';
            if (pendingDeployment.isAir) routeColor = '#8b5cf6';
            if (pendingDeployment.hqKey === 'coastguard') routeColor = 'var(--navy)';

            if(activeRouteLine) map.removeLayer(activeRouteLine); activeRouteLine = L.polyline(calculatedCoordinates, {color: routeColor, weight: 5, dashArray: '10,10'}).addTo(map);
            if (isAuto) { confirmAndStartDeployment(); } else {
                if(document.getElementById('btn-confirm-route')) document.getElementById('btn-confirm-route').remove();
                
                let btnIcon = `<i class="fas fa-truck-moving"></i>`;
                if(pendingDeployment.isAir) btnIcon = `<i class="fas fa-fighter-jet"></i>`;
                if(pendingDeployment.hqKey === 'coastguard') btnIcon = `<i class="fas fa-ship"></i>`;

                const btn = document.createElement('button'); btn.id = 'btn-confirm-route'; btn.innerHTML = `${btnIcon} ONAYLA VE GÖREVE ÇIK`; btn.style = "position:absolute; bottom:80px; left:50%; transform:translateX(-50%); z-index:2000; padding:15px 30px; background:var(--success); color:#fff; border:none; border-radius:10px; font-weight:800; cursor:pointer; font-size:14px; box-shadow:0 10px 30px rgba(16,185,129,0.5);"; btn.onclick = () => { btn.remove(); confirmAndStartDeployment(); tempPoints=[]; }; document.body.appendChild(btn);
            }
        }

        function confirmAndStartDeployment() {
            let varColor = '#3b82f6';
            if (pendingDeployment.isAir) varColor = '#8b5cf6';
            else if (pendingDeployment.hqKey === 'logistics') varColor = '#10b981';
            else if (pendingDeployment.hqKey === 'fuel') varColor = '#f97316';
            else if (pendingDeployment.hqKey === 'coastguard') varColor = 'var(--navy)';

            let iconType = 'fa-truck';
            if (pendingDeployment.vehicle.includes('Helikopter')) iconType = 'fa-helicopter';
            else if (pendingDeployment.vehicle.includes('Uçağı') || pendingDeployment.vehicle.includes('İHA') || pendingDeployment.vehicle.includes('TİHA')) iconType = 'fa-plane';
            else if (pendingDeployment.hqKey === 'coastguard' && (pendingDeployment.vehicle.includes('Gemisi') || pendingDeployment.vehicle.includes('Bot') || pendingDeployment.vehicle.includes('Römorkör') || pendingDeployment.vehicle.includes('Hovercraft'))) iconType = 'fa-ship';
            else if (pendingDeployment.hqKey === 'coastguard' && pendingDeployment.vehicle.includes('Jet')) iconType = 'fa-motorcycle'; // Jetski için
            else if (pendingDeployment.hqKey === 'coastguard') iconType = 'fa-life-ring';

            const id = 'U-' + Math.floor(Math.random()*9000);
            const m = L.marker(calculatedCoordinates[0], { icon: L.divIcon({ html: `<div style="background:${varColor}; width:32px; height:32px; border-radius:50%; border:2px solid #fff; display:flex; align-items:center; justify-content:center; color:#fff; cursor:pointer; box-shadow:0 0 15px ${varColor};"><i class="fas ${iconType}"></i></div>`, className:''}) }).addTo(map);
            
            // YENİ: İnsansız Deniz Araçlarına (İDA) da Taktik Kamera eklendi.
            const hasCamera = pendingDeployment.vehicle.includes('İHA') || pendingDeployment.vehicle.includes('TİHA') || pendingDeployment.vehicle.includes('SİHA') || pendingDeployment.vehicle.includes('İDA');
            
            if(hasCamera) {
                m.on('click', () => openCamera(id)); 
            } else {
                m.on('click', () => alert("SİSTEM UYARISI: Bu araç sınıfında Taktik ISR Kamera beslemesi bulunmamaktadır."));
            }

            activeUnits[id] = { marker: m, path: calculatedCoordinates, seg: 0, prog: 0, fuel: 100, fuelAlertTriggered: false, isAir: pendingDeployment.isAir, hqKey: pendingDeployment.hqKey, name: pendingDeployment.team, v: pendingDeployment.vehicle, mission: pendingDeployment.mission, payload: pendingDeployment.payload, priority: pendingDeployment.priority, state: 'outbound' };
            setInterval(() => moveUnit(id), 50); log(`KOMUTA: ${pendingDeployment.vehicle} hedeflerine doğru intikale başladı.`);
            if(activeRouteLine) { map.removeLayer(activeRouteLine); activeRouteLine = null; tempRouteGroup.clearLayers(); }
            
            updateActiveUnitsPanel(); isAutoDeploying = false; setTimeout(() => { processDeployQueue(); }, 1500);
        }

        function moveUnit(id) {
            let u = activeUnits[id]; if(!u) return;

            u.fuel -= u.isAir ? 0.015 : 0.005; 
            if (u.fuel <= 0) u.fuel = 0.1;

            if (u.isAir && u.fuel <= 50 && !u.fuelAlertTriggered && u.state !== 'returning') {
                u.fuelAlertTriggered = true; generateFuelReport(u); log(`⚠️ KRİTİK: ${u.v} yakıt seviyesi %50'nin altında! Havada ikmal uyarısı verildi.`);
            }

            if (u.state === 'loitering') {
                u.loiterTicks--;
                u.fuel -= 0.002; 
                updateHUD(u); updateActiveUnitsPanel();
                if (!u.marker.options.icon.options.className.includes('loiter-ring')) {
                    let iconType = 'fa-plane';
                    if (u.v.includes('Helikopter')) iconType = 'fa-helicopter';
                    if (u.v.includes('İDA') || u.v.includes('Bot')) iconType = 'fa-ship';

                    u.marker.setIcon(L.divIcon({ html: `<div class="loiter-ring" style="background:var(--recon); width:32px; height:32px; border-radius:50%; border:2px solid #fff; display:flex; align-items:center; justify-content:center; color:#fff; cursor:pointer;"><i class="fas ${iconType}"></i></div>`, className:'loiter-ring' }));
                }
                if (u.loiterTicks <= 0) {
                    u.state = 'returning'; u.path = [...u.path].reverse(); u.seg = 0; u.prog = 0;
                    
                    let iconType = 'fa-plane';
                    if (u.v.includes('Helikopter')) iconType = 'fa-helicopter';
                    if (u.v.includes('İDA') || u.v.includes('Bot')) iconType = 'fa-ship';
                    
                    let varColor = u.hqKey === 'coastguard' ? 'var(--navy)' : '#8b5cf6';
                    
                    u.marker.setIcon(L.divIcon({ html: `<div style="background:${varColor}; width:32px; height:32px; border-radius:50%; border:2px solid #fff; display:flex; align-items:center; justify-content:center; color:#fff; cursor:pointer; box-shadow:0 0 15px ${varColor};"><i class="fas ${iconType}"></i></div>`, className:'' }));
                    log(`UNSUR: ${u.v} keşif görevini tamamladı. RTB (Üsse Dönüş) başladı.`);
                    generateReconReport(u); 
                }
            } 
            else if(u.seg < u.path.length - 1) {
                u.prog += u.isAir ? 0.015 : 0.03; 
                if(u.prog >= 1) { u.prog = 0; u.seg++; }
                else { let p1 = u.path[u.seg], p2 = u.path[u.seg+1]; let lat = p1[0]+(p2[0]-p1[0])*u.prog, lng = p1[1]+(p2[1]-p1[1])*u.prog; u.marker.setLatLng([lat, lng]); updateHUD(u); updateActiveUnitsPanel(); if(u.isAir && document.getElementById('camera-overlay').style.display === 'block') syncCam(lat, lng, u); }
            } else {
                if (u.isAir && u.state === 'outbound') {
                    const canRecon = u.v.includes('İHA') || u.v.includes('TİHA') || u.v.includes('SİHA');
                    if(canRecon) {
                        u.state = 'loitering'; u.loiterTicks = 120; u.targetLat = u.marker.getLatLng().lat; u.targetLng = u.marker.getLatLng().lng; log(`HAVA UNSURU: ${u.v} hedef bölgede. Sensörler aktif, RİSK ANALİZİ (Loiter) yapılıyor.`); updateHUD(u); updateActiveUnitsPanel();
                    } else {
                        u.state = 'returning'; u.path = [...u.path].reverse(); u.seg = 0; u.prog = 0; log(`LOJİSTİK/İKMAL UÇAĞI: ${u.v} hedefe ulaştı. RTB başladı.`);
                    }
                } 
                else if (u.state === 'outbound_police') { u.state = 'securing'; log(`SİSTEM: EGM Polis aracı kapalı yola ulaştı. Çevre güvenliği sağlandı.`); updateHUD(u); updateActiveUnitsPanel(); } 
                else if (u.state === 'returning') {
                    map.removeLayer(u.marker); log(`SİSTEM: ${u.v} üsse güvenli dönüş yaptı. Araç filodan çekildi.`); delete activeUnits[id]; updateActiveUnitsPanel(); document.getElementById('live-hud').innerHTML = "SİSTEM HAZIR. KARARGAH SEÇİNİZ VEYA SAATİ İLERLETİNİZ...";
                }
                else if (u.state === 'outbound' && !u.isAir) { 
                    // YENİ: İnsansız Deniz Araçları (İDA) da denizde keşif / Loiter yapsın
                    if(u.v.includes('İDA')) {
                        u.state = 'loitering'; u.loiterTicks = 120; u.targetLat = u.marker.getLatLng().lat; u.targetLng = u.marker.getLatLng().lng; log(`SAHİL GÜVENLİK İDA: ${u.v} hedef bölgede sonar/kamera taraması (Loiter) yapıyor.`); updateHUD(u); updateActiveUnitsPanel();
                    } else if(u.state !== 'completed') { 
                        u.state = 'completed'; log(`SİSTEM: ${u.v} operasyonunu tamamladı.`); updateHUD(u); updateActiveUnitsPanel(); 
                    } 
                }
            }
        }

        function updateHUD(u) {
            let statusText = "HEDEFE İNTİKAL"; let statusColor = "#10b981"; 
            if(u.state === 'returning') { statusText = "ÜSSE DÖNÜYOR"; statusColor = "#f59e0b"; } else if(u.state === 'loitering') { statusText = "BÖLGE KEŞFİ (LOITER)"; statusColor = "var(--recon)"; } else if(u.state === 'completed') { statusText = "GÖREV TAMAMLANDI"; statusColor = "#3b82f6"; } else if(u.state === 'securing') { statusText = "BÖLGE GÜVENLİKTE (BLOKE)"; statusColor = "#ef4444"; }
            document.getElementById('live-hud').innerHTML = `<div style="color:#fff; margin-bottom:5px; font-weight:800;">EKİP: ${u.name}</div><div style="margin-bottom:5px; color:#94a3b8;">ARAÇ: ${u.v}</div><div style="margin-bottom:5px; color:#f59e0b; font-size:10px;">GÖREV: ${u.mission}</div><div style="border-top:1px solid #333; padding-top:5px; margin-top:5px;">YAKIT: %${u.fuel.toFixed(1)} | <span style="color:${statusColor}; font-weight:800;">${statusText}</span></div>`;
            const teleStatus = document.getElementById('tele-status'); if(teleStatus) teleStatus.innerText = statusText;
        }

        function openCamera(id) {
            document.getElementById('camera-overlay').style.display = 'block'; const u = activeUnits[id]; const pos = u.marker.getLatLng();
            document.getElementById('tele-payload').innerText = u.payload; 
            
            // İDA'lar için irtifa 0 gösterilir
            let altText = `${u.v.includes('TİHA')?'25,000':(u.v.includes('İHA')?'18,500':'12,500')} FT`;
            if(u.v.includes('İDA')) altText = "SEA LEVEL (0 FT) - SONAR ACTIVE";
            document.getElementById('tele-alt').innerText = altText;
            
            setTimeout(() => { if(cN) { cN.remove(); cO.remove(); cT.remove(); } cN = L.map('cam-n', { zoomControl:false, attributionControl:false }).setView(pos, 13); L.tileLayer('https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}').addTo(cN); cO = L.map('cam-o', { zoomControl:false, attributionControl:false }).setView(pos, 18); L.tileLayer('https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}').addTo(cO); cT = L.map('cam-t', { zoomControl:false, attributionControl:false }).setView(pos, 16); L.tileLayer('https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}').addTo(cT); }, 50); 
        }

        function syncCam(lat, lng, u) { if(cN && cO && cT) { cN.setView([lat, lng], 13, {animate:false}); cO.setView([lat, lng], 18, {animate:false}); cT.setView([lat, lng], 16, {animate:false}); } }
        function log(m) { const t = document.getElementById('term'); t.innerHTML += `<div class="log-line">[${new Date().toLocaleTimeString()}] ${m}</div>`; t.scrollTop = t.scrollHeight; }
    </script>
</body>
</html>
