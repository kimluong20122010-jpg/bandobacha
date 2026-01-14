
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Bản đồ thông minh Bắc Hà, Lào Cai</title>

  <!-- Leaflet CSS -->
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    integrity="sha256-p4Qj1bYQbZkG2fGZ8G7n2AFvZlqXkG2zFEpGJQ2v5r0="
    crossorigin=""
  />

  <!-- Leaflet MarkerCluster (tùy chọn, để gom cụm điểm) -->
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css"
  />
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css"
  />

  <!-- Leaflet Locate Control (định vị người dùng) -->
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet.locatecontrol@0.78.0/dist/L.Control.Locate.min.css"
  />

  <style>
    :root {
      --bg: #0f172a;
      --panel: #111827;
      --text: #e5e7eb;
      --accent: #22c55e;
      --muted: #9ca3af;
      --border: #1f2937;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; background: var(--bg); color: var(--text); font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif; }
    #app { display: grid; grid-template-columns: 320px 1fr; grid-template-rows: auto 1fr; height: 100%; }

    header {
      grid-column: 1 / -1;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 12px 16px;
      border-bottom: 1px solid var(--border);
      background: #0b1220;
    }
    header h1 { font-size: 18px; margin: 0; letter-spacing: 0.2px; }
    header .brand { display: flex; align-items: center; gap: 10px; }
    header .brand .dot { width: 10px; height: 10px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 12px rgba(34, 197, 94, 0.7); }

    aside {
      background: var(--panel);
      border-right: 1px solid var(--border);
      padding: 14px;
      overflow-y: auto;
    }

    .section { margin-bottom: 16px; }
    .section h3 { margin: 0 0 8px; font-size: 14px; color: var(--muted); font-weight: 600; text-transform: uppercase; letter-spacing: 0.6px; }

    .search-box input {
      width: 100%;
      padding: 10px 12px;
      border-radius: 8px;
      border: 1px solid var(--border);
      background: #0b1220;
      color: var(--text);
      outline: none;
    }

    .filters { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
    .chip {
      display: flex; align-items: center; gap: 8px;
      padding: 8px 10px; border: 1px solid var(--border);
      border-radius: 8px; background: #0b1220; cursor: pointer;
      user-select: none;
    }
    .chip input { accent-color: var(--accent); }

    .legend { display: grid; grid-template-columns: 1fr; gap: 8px; }
    .legend-item { display: flex; align-items: center; gap: 8px; font-size: 14px; color: var(--muted); }
    .legend-swatch { width: 14px; height: 14px; border-radius: 3px; }

    .route-box { background: #0b1220; border: 1px solid var(--border); border-radius: 8px; padding: 10px; }
    .route-box p { margin: 0 0 8px; color: var(--muted); font-size: 14px; }
    .route-actions { display: flex; gap: 8px; }
    .btn {
      padding: 8px 10px; border-radius: 8px; border: 1px solid var(--border);
      background: #0b1220; color: var(--text); cursor: pointer;
    }
    .btn.primary { background: var(--accent); color: #062b16; border-color: #16a34a; font-weight: 600; }

    #map { width: 100%; height: 100%; }

    /* Popup tinh chỉnh */
    .leaflet-popup-content-wrapper { background: #0b1220; color: var(--text); border: 1px solid var(--border); }
    .leaflet-popup-tip { background: #0b1220; }
    .popup-title { font-weight: 700; margin-bottom: 6px; }
    .popup-meta { color: var(--muted); font-size: 13px; margin-bottom: 6px; }
    .popup-desc { font-size: 14px; line-height: 1.4; }
    .popup-actions { margin-top: 8px; display: flex; gap: 8px; }
    .popup-link {
      display: inline-block; padding: 6px 8px; border-radius: 6px;
      background: #0b1220; border: 1px solid var(--border); color: var(--text);
      text-decoration: none;
    }
    .popup-link.primary { background: var(--accent); color: #062b16; border-color: #16a34a; font-weight: 600; }
  </style>
</head>
<body>
  <div id="app">
    <header>
      <div class="brand">
        <span class="dot"></span>
        <h1>Bản đồ thông minh Bắc Hà, Lào Cai</h1>
      </div>
      <div style="color: var(--muted); font-size: 13px;">
        Dữ liệu minh họa—tọa độ gần đúng, có thể điều chỉnh
      </div>
    </header>

    <aside>
      <div class="section search-box">
        <h3>Tìm kiếm nhanh</h3>
        <input id="searchInput" type="text" placeholder="Nhập tên điểm (ví dụ: Chợ Bắc Hà)..." />
      </div>

      <div class="section">
        <h3>Bộ lọc loại điểm</h3>
        <div class="filters">
          <label class="chip"><input type="checkbox" class="filter" value="vanhoa" checked /> Văn hóa</label>
          <label class="chip"><input type="checkbox" class="filter" value="cho" checked /> Chợ</label>
          <label class="chip"><input type="checkbox" class="filter" value="banlang" checked /> Bản làng</label>
          <label class="chip"><input type="checkbox" class="filter" value="thiennhien" checked /> Thiên nhiên</label>
        </div>
      </div>

      <div class="section">
        <h3>Chú giải</h3>
        <div class="legend">
          <div class="legend-item"><span class="legend-swatch" style="background:#ef4444"></span> Văn hóa</div>
          <div class="legend-item"><span class="legend-swatch" style="background:#f59e0b"></span> Chợ</div>
          <div class="legend-item"><span class="legend-swatch" style="background:#3b82f6"></span> Bản làng</div>
          <div class="legend-item"><span class="legend-swatch" style="background:#10b981"></span> Thiên nhiên</div>
          <div class="legend-item"><span class="legend-swatch" style="background:#8b5cf6"></span> Tuyến tham quan</div>
        </div>
      </div>

      <div class="section route-box">
        <p>Tuyến gợi ý: Chợ Bắc Hà → Dinh Hoàng A Tưởng → Bản Na Hối</p>
        <div class="route-actions">
          <button id="showRouteBtn" class="btn primary">Hiển thị tuyến</button>
          <button id="clearRouteBtn" class="btn">Xóa tuyến</button>
        </div>
      </div>
    </aside>

    <main>
      <div id="map"></div>
    </main>
  </div>

  <!-- Leaflet JS -->
  <script
    src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
    integrity="sha256-o9N1j7kGStb6Qp2hYQvQ2QfZ8aA1rjzG8v+1G1bMM8M="
    crossorigin=""
  ></script>

  <!-- MarkerCluster -->
  <script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

  <!-- Locate Control -->
  <script src="https://unpkg.com/leaflet.locatecontrol@0.78.0/dist/L.Control.Locate.min.js"></script>

  <script>
    // Khởi tạo bản đồ, tâm gần thị trấn Bắc Hà
    const map = L.map('map', {
      zoomControl: true,
      minZoom: 8,
      maxZoom: 18
    }).setView([22.333, 104.286], 12);

    // Lớp nền
    const baseLayers = {
      'Bản đồ sáng': L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; OpenStreetMap'
      }),
      'Bản đồ tối': L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
        maxZoom: 19,
        attribution: '&copy; CARTO'
      }),
      'Vệ tinh': L.tileLayer('https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; OSM HOT'
      })
    };
    baseLayers['Bản đồ tối'].addTo(map);

    // Điều khiển lớp
    L.control.layers(baseLayers, null, { position: 'topleft' }).addTo(map);

    // Điều khiển định vị
    L.control.locate({
      position: 'topleft',
      strings: { title: 'Định vị vị trí của tôi' },
      flyTo: true
    }).addTo(map);

    // Biểu tượng theo loại
    const iconMap = {
      vanhoa: L.divIcon({ className: 'custom-icon', html: '<span style="background:#ef4444;width:14px;height:14px;display:inline-block;border-radius:50%"></span>', iconSize: [14,14] }),
      cho: L.divIcon({ className: 'custom-icon', html: '<span style="background:#f59e0b;width:14px;height:14px;display:inline-block;border-radius:50%"></span>', iconSize: [14,14] }),
      banlang: L.divIcon({ className: 'custom-icon', html: '<span style="background:#3b82f6;width:14px;height:14px;display:inline-block;border-radius:50%"></span>', iconSize: [14,14] }),
      thiennhien: L.divIcon({ className: 'custom-icon', html: '<span style="background:#10b981;width:14px;height:14px;display:inline-block;border-radius:50%"></span>', iconSize: [14,14] })
    };

    // Dữ liệu điểm (tọa độ gần đúng—hãy thay bằng tọa độ chuẩn nếu có)
    const places = [
      {
        name: 'Chợ Bắc Hà',
        type: 'cho',
        coords: [22.3339, 104.2869],
        address: 'TT. Bắc Hà, Bắc Hà, Lào Cai',
        desc: 'Chợ phiên nổi tiếng vùng cao, họp chủ nhật, nơi giao lưu văn hóa các dân tộc.',
        link: 'https://vi.wikipedia.org/wiki/Ch%E1%BB%A3_B%E1%BA%AFc_H%C3%A0'
      },
      {
        name: 'Dinh Hoàng A Tưởng',
        type: 'vanhoa',
        coords: [22.3349, 104.2879],
        address: 'TT. Bắc Hà, Bắc Hà, Lào Cai',
        desc: 'Biệt dinh kiến trúc Á–Âu đầu thế kỷ 20, di tích văn hóa nổi bật.',
        link: 'https://vi.wikipedia.org/wiki/Dinh_Ho%C3%A0ng_A_T%C6%B0%E1%BB%9Fng'
      },
      {
        name: 'Bản Na Hối',
        type: 'banlang',
        coords: [22.3455, 104.3005],
        address: 'Na Hối, Bắc Hà, Lào Cai',
        desc: 'Bản làng người Tày/Dao, cảnh quan ruộng bậc thang và nếp sống bản địa.',
        link: 'https://laocai.gov.vn'
      },
      {
        name: 'Bản Tả Van Chư',
        type: 'banlang',
        coords: [22.392, 104.256],
        address: 'Tả Van Chư, Bắc Hà, Lào Cai',
        desc: 'Vùng cao nguyên mát mẻ, hoa mận mùa xuân, trải nghiệm homestay.',
        link: 'https://laocai.gov.vn'
      },
      {
        name: 'Đồi hoa mận Bắc Hà',
        type: 'thiennhien',
        coords: [22.36, 104.29],
        address: 'Khu vực Bắc Hà',
        desc: 'Thiên nhiên đặc trưng với mùa hoa mận trắng, đẹp nhất từ tháng 1–3.',
        link: 'https://laocai.gov.vn'
      }
    ];

    // Tạo nhóm cluster
    const markerCluster = L.markerClusterGroup({ showCoverageOnHover: false });

    // Lưu tham chiếu marker để lọc/tìm kiếm
    const markers = [];

    // Hàm tạo popup HTML
    function popupHTML(p) {
      return `
        <div class="popup">
          <div class="popup-title">${p.name}</div>
          <div class="popup-meta">${p.address}</div>
          <div class="popup-desc">${p.desc}</div>
          <div class="popup-actions">
            <a class="popup-link primary" href="${p.link}" target="_blank" rel="noopener">Xem chi tiết</a>
            <a class="popup-link" href="https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(p.name + ' ' + p.address)}" target="_blank" rel="noopener">Mở trên Google Maps</a>
          </div>
        </div>
      `;
    }

    // Thêm marker
    places.forEach(p => {
      const m = L.marker(p.coords, { icon: iconMap[p.type] })
        .bindPopup(popupHTML(p));
      m.feature = { properties: { type: p.type, name: p.name } };
      markers.push(m);
      markerCluster.addLayer(m);
    });
    map.addLayer(markerCluster);

    // Bộ lọc theo loại
    const filterInputs = Array.from(document.querySelectorAll('.filter'));
    function applyFilters() {
      const activeTypes = new Set(
        filterInputs.filter(i => i.checked).map(i => i.value)
      );
      markerCluster.clearLayers();
      markers.forEach(m => {
        const t = m.feature.properties.type;
        if (activeTypes.has(t)) markerCluster.addLayer(m);
      });
    }
    filterInputs.forEach(i => i.addEventListener('change', applyFilters));

    // Tìm kiếm nhanh theo tên
    const searchInput = document.getElementById('searchInput');
    searchInput.addEventListener('input', () => {
      const q = searchInput.value.trim().toLowerCase();
      if (!q) { applyFilters(); return; }
      markerCluster.clearLayers();
      markers.forEach(m => {
        const name = (m.feature.properties.name || '').toLowerCase();
        const type = (m.feature.properties.type || '').toLowerCase();
        if (name.includes(q) || type.includes(q)) markerCluster.addLayer(m);
      });
    });

    // Tuyến gợi ý (polyline)
    const routeCoords = [
      [22.3339, 104.2869], // Chợ Bắc Hà
      [22.3349, 104.2879], // Dinh Hoàng A Tưởng
      [22.3455, 104.3005]  // Bản Na Hối
    ];
    let routeLayer = null;

    function showRoute() {
      if (routeLayer) return;
      routeLayer = L.polyline(routeCoords, {
        color: '#8b5cf6',
        weight: 4,
        opacity: 0.9
      }).addTo(map);
      map.fitBounds(routeLayer.getBounds(), { padding: [20, 20] });
    }
    function clearRoute() {
      if (routeLayer) {
        map.removeLayer(routeLayer);
        routeLayer = null;
      }
    }
    document.getElementById('showRouteBtn').addEventListener('click', showRoute);
    document.getElementById('clearRouteBtn').addEventListener('click', clearRoute);

    // Mẹo: phóng to về Bắc Hà khi double-click nền
    map.on('dblclick', () => map.setView([22.333, 104.286], 13));
  </script>
</body>
</html>
