<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
    
    <title>Guardian Assistant Pro</title>
    
    <style>
        :root { --g-green: #00754a; --g-orange: #ffc107; --bg: #f8f9fa; }
        body { font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; margin: 0; background: var(--bg); padding-bottom: 80px; -webkit-tap-highlight-color: transparent; }
        
        /* App Header */
        .header { background: var(--g-green); color: white; padding: 45px 15px 15px; text-align: center; position: sticky; top: 0; z-index: 100; font-weight: bold; letter-spacing: 1px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        
        /* Search Box */
        .search-area { padding: 15px; background: white; position: sticky; top: 85px; z-index: 99; }
        #searchInput { width: 100%; padding: 15px 20px; border: 2px solid #eee; border-radius: 30px; box-sizing: border-box; font-size: 16px; outline: none; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
        #searchInput:focus { border-color: var(--g-green); }

        /* Category Scroll */
        .category-bar { display: flex; overflow-x: auto; background: white; padding: 10px; gap: 10px; scrollbar-width: none; }
        .category-bar::-webkit-scrollbar { display: none; }
        .cat-btn { padding: 10px 20px; background: #eee; border-radius: 25px; white-space: nowrap; border: none; font-size: 14px; font-weight: 600; color: #555; }
        .cat-btn.active { background: var(--g-green); color: white; }

        /* Product Cards */
        .list { padding: 15px; }
        .card { background: white; border-radius: 15px; padding: 15px; margin-bottom: 12px; display: flex; align-items: center; justify-content: space-between; box-shadow: 0 2px 8px rgba(0,0,0,0.05); border-left: 5px solid transparent; transition: 0.2s; }
        .card:active { transform: scale(0.98); border-left-color: var(--g-green); }
        .info { flex-grow: 1; }
        .brand { font-size: 11px; color: var(--g-green); font-weight: 800; text-transform: uppercase; margin-bottom: 4px; }
        .name { font-size: 15px; font-weight: 600; color: #333; margin-bottom: 4px; }
        .price { color: #e60000; font-weight: 800; font-size: 18px; margin: 0; }
        .aisle { font-size: 11px; color: #888; margin-top: 6px; display: block; }

        /* Navigation Bar */
        .nav { position: fixed; bottom: 0; width: 100%; background: white; display: flex; border-top: 1px solid #eee; height: 70px; align-items: center; padding-bottom: env(safe-area-inset-bottom); }
        .nav-btn { flex: 1; text-align: center; border: none; background: none; color: #888; font-size: 11px; font-weight: bold; }
        .nav-btn.active { color: var(--g-green); }
        .scan-circle { width: 55px; height: 55px; background: var(--g-green); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; margin-top: -30px; border: 5px solid var(--bg); font-size: 20px; }

        /* Scanner Modal */
        #scanner { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; z-index: 1000; }
        #v { width: 100%; height: 100%; object-fit: cover; }
        .scan-ui { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; flex-direction: column; justify-content: center; align-items: center; color: white; pointer-events: none; }
        .scan-box { width: 260px; height: 260px; border: 2px solid var(--g-orange); border-radius: 20px; box-shadow: 0 0 0 2000px rgba(0,0,0,0.6); position: relative; }
        .scan-line { position: absolute; width: 100%; height: 2px; background: var(--g-orange); top: 50%; animation: scanAnim 2s infinite; }
        @keyframes scanAnim { 0% { top: 0; } 50% { top: 100%; } 100% { top: 0; } }
    </style>
</head>
<body>

<div class="header">GUARDIAN SMART ASSISTANT</div>

<div id="main-ui">
    <div class="search-area">
        <input type="text" id="searchInput" placeholder="Search brands (e.g., Wardah, Bio-Essence)..." onkeyup="search()">
    </div>

    <div class="category-bar" id="catBar"></div>
    <div class="list" id="productList"></div>
</div>

<div id="scanner">
    <video id="v" autoplay playsinline></video>
    <div class="scan-ui">
        <div class="scan-box"><div class="scan-line"></div></div>
        <p style="margin-top: 20px; font-weight: bold;">POINT AT BARCODE</p>
        <button onclick="stopScan()" style="pointer-events: auto; margin-top: 30px; padding: 12px 40px; border-radius: 30px; border: none; background: white; font-weight: bold;">BACK TO HOME</button>
    </div>
</div>

<div class="nav">
    <button class="nav-btn active">🏠<br>HOME</button>
    <div class="nav-btn" onclick="startScan()"><div class="scan-circle">🔍</div>SCAN</div>
    <button class="nav-btn" onclick="alert('Categories Loaded')">📂<br>ITEMS</button>
</div>

<script>
    // MASSIVE DATABASE (120+ Items)
    const rawBrands = {
        "Skincare": ["Cetaphil", "Hada Labo", "Bio-Essence", "Wardah", "Loreal", "Eucerin", "Simple", "Garnier", "Neutrogena", "Sunsilk", "Olay", "Cosrx", "Nivea", "Clinelle", "Aiken"],
        "Health": ["Panadol", "Gaviscon", "Blackmores", "Hurix's", "Flavettes", "Brands", "Eno", "Tiger Balm", "Vicks", "Strepsils", "Eye Mo", "Betadine", "Dettol Antiseptic", "Woods", "Cap Ibu & Anak"],
        "Personal Care": ["Dettol Shower", "Colgate", "Pantene", "Dove", "Lifebuoy", "Rexona", "Sunplay", "Sensodyne", "Oral-B", "Shokubutsu", "May", "Ginvera", "Kotex", "Libresse", "Carefree"],
        "Cosmetics": ["Maybelline", "Silkygirl", "Revlon", "Wardah Colorfit", "In2It", "Kate", "Peripera", "Essence", "Catrice", "Elianto"],
        "Baby": ["Johnson's", "Huggies", "MamyPoko", "Pureen", "Pigeon", "Cetaphil Baby", "Sebamed", "Biolane"]
    };

    let fullDb = [];
    Object.keys(rawBrands).forEach(cat => {
        rawBrands[cat].forEach(brand => {
            // Adding multiple items per brand to reach 120+ items
            for(let i=1; i<=3; i++) {
                fullDb.push({
                    b: brand,
                    n: `${brand} ${cat} Series ${i}`,
                    p: (Math.random() * 85 + 5).toFixed(2),
                    c: cat,
                    a: `Aisle ${Math.floor(Math.random() * 10) + 1}`
                });
            }
        });
    });

    const categories = ["All", ...Object.keys(rawBrands)];

    function render(data) {
        const list = document.getElementById('productList');
        list.innerHTML = data.map(i => `
            <div class="card">
                <div class="info">
                    <div class="brand">${i.b}</div>
                    <div class="name">${i.n}</div>
                    <p class="price">RM ${i.p}</p>
                    <span class="aisle">📍 Section: ${i.c} | ${i.a}</span>
                </div>
            </div>
        `).join('');
    }

    function search() {
        const query = document.getElementById('searchInput').value.toLowerCase();
        const filtered = fullDb.filter(i => i.n.toLowerCase().includes(query) || i.b.toLowerCase().includes(query));
        render(filtered);
    }

    function filter(cat, btn) {
        document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        render(cat === "All" ? fullDb : fullDb.filter(i => i.c === cat));
    }

    function startScan() {
        document.getElementById('scanner').style.display = 'block';
        navigator.mediaDevices.getUserMedia({video: {facingMode: "environment"}})
        .then(s => document.getElementById('v').srcObject = s);
    }

    function stopScan() {
        document.getElementById('scanner').style.display = 'none';
        const s = document.getElementById('v').srcObject;
        if(s) s.getTracks().forEach(t => t.stop());
    }

    // Initialize UI
    document.getElementById('catBar').innerHTML = categories.map(c => 
        `<button class="cat-btn ${c==='All'?'active':''}" onclick="filter('${c}', this)">${c}</button>`
    ).join('');
    render(fullDb);
</script>
</body>
</html>
