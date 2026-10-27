<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sổ Tay WebGIS Y Tế & Du Lịch Đà Nẵng v3</title>
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Arial, sans-serif;
        }

        body {
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: #f0f2f5;
        }

        /* Header */
        header {
            background-color: #00695c;
            color: white;
            padding: 14px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.15);
            z-index: 1000;
        }

        header h1 {
            font-size: 1.25rem;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .header-actions {
            display: flex;
            gap: 10px;
        }

        .btn-action {
            padding: 8px 16px;
            font-size: 0.95rem;
            font-weight: bold;
            border-radius: 20px;
            border: none;
            cursor: pointer;
            text-decoration: none;
            display: inline-flex;
            align-items: center;
            gap: 6px;
        }

        .btn-gps {
            background-color: #0288d1;
            color: white;
        }

        .btn-sos {
            background-color: #d32f2f;
            color: white;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        /* Container */
        .container {
            display: flex;
            flex: 1;
            overflow: hidden;
        }

        /* Sidebar */
        .sidebar {
            width: 380px;
            background: white;
            padding: 18px;
            display: flex;
            flex-direction: column;
            gap: 14px;
            box-shadow: 2px 0 12px rgba(0,0,0,0.1);
            z-index: 900;
            overflow-y: auto;
        }

        .filter-group {
            display: flex;
            gap: 8px;
        }

        .filter-btn {
            flex: 1;
            padding: 10px;
            border: 2px solid #00695c;
            background: white;
            color: #00695c;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
        }

        .filter-btn.active, .filter-btn:hover {
            background: #00695c;
            color: white;
        }

        .location-list {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .location-card {
            border: 1px solid #e0e0e0;
            border-left: 5px solid #00695c;
            border-radius: 8px;
            padding: 12px 14px;
            background: #fafafa;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .location-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 10px rgba(0,0,0,0.08);
            background: #e0f2f1;
        }

        .location-card h3 {
            font-size: 1.05rem;
            color: #004d40;
            margin-bottom: 6px;
        }

        .location-card p {
            font-size: 0.9rem;
            color: #424242;
            margin-bottom: 4px;
        }

        .tag {
            display: inline-block;
            font-size: 0.8rem;
            padding: 3px 10px;
            border-radius: 12px;
            font-weight: bold;
            margin-top: 6px;
        }

        .tag-medical { background: #ffebee; color: #c62828; }
        .tag-tourism { background: #e8f5e9; color: #2e7d32; }

        /* Map */
        #map {
            flex: 1;
            height: 100%;
        }

        @media (max-width: 768px) {
            .container { flex-direction: column-reverse; }
            .sidebar { width: 100%; height: 45%; }
            #map { height: 55%; }
        }
    </style>
</head>
<body>

    <header>
        <h1>🏥 WebGIS Sổ Tay Số Đà Nẵng (v3)</h1>
        <div class="header-actions">
            <button class="btn-action btn-gps" onclick="locateUser()">📍 Vị trí của tôi</button>
            <a href="tel:115" class="btn-action btn-sos">🚨 CẤP CỨU 115</a>
        </div>
    </header>

    <div class="container">
        <div class="sidebar">
            <h2>Bộ Lọc Địa Điểm</h2>
            <div class="filter-group">
                <button class="filter-btn active" onclick="filterLocations('all', event)">Tất cả</button>
                <button class="filter-btn" onclick="filterLocations('Y tế', event)">🏥 Y tế</button>
                <button class="filter-btn" onclick="filterLocations('Du lịch', event)">🏨 Du lịch</button>
            </div>

            <div class="location-list" id="location-list"></div>
        </div>

        <div id="map"></div>
    </div>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        // Dữ liệu danh sách địa điểm
        const locations = [
            {
                name: "Bệnh viện C Đà Nẵng",
                category: "Y tế",
                lat: 16.0681,
                lng: 108.2145,
                phone: "0236 3821 118",
                note: "Cấp cứu 24/7, có lối xe lăn"
            },
            {
                name: "Bệnh viện Đa khoa Đà Nẵng",
                category: "Y tế",
                lat: 16.0718,
                lng: 108.2132,
                phone: "0236 3821 115",
                note: "Trung tâm cấp cứu hàng đầu Đà Nẵng"
            },
            {
                name: "Bệnh viện Y học Cổ truyền Đà Nẵng",
                category: "Y tế",
                lat: 16.0335,
                lng: 108.2120,
                phone: "0236 3696 123",
                note: "Phục hồi chức năng và điều trị cho người cao tuổi"
            },
            {
                name: "Công viên APEC Đà Nẵng",
                category: "Du lịch",
                lat: 16.0602,
                lng: 108.2232,
                phone: "Không có",
                note: "Lối đi bằng phẳng, nhiều ghế nghỉ chân"
            },
            {
                name: "Danh thắng Ngũ Hành Sơn",
                category: "Du lịch",
                lat: 16.0042,
                lng: 108.2633,
                phone: "0236 3961 114",
                note: "Có hệ thống thang máy hỗ trợ người cao tuổi"
            }
        ];

        // Khởi tạo bản đồ tập trung tại Đà Nẵng
        const map = L.map('map').setView([16.0544, 108.2022], 13);

        // Sử dụng lớp nền Bản đồ tiêu chuẩn sạch sẽ, hỗ trợ hiển thị tiếng Việt chuẩn hóa
        L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
            subdomains: 'abcd'
        }).addTo(map);

        let markers = [];
        let userMarker = null;

        function displayLocations(data) {
            markers.forEach(m => map.removeLayer(m));
            markers = [];

            const listContainer = document.getElementById('location-list');
            listContainer.innerHTML = '';

            data.forEach((loc) => {
                const marker = L.marker([loc.lat, loc.lng]).addTo(map);
                marker.bindPopup(`
                    <div style="font-size: 14px;">
                        <b style="color: #00695c;">${loc.name}</b><br>
                        <b>Loại:</b> ${loc.category}<br>
                        <b>SĐT:</b> <a href="tel:${loc.phone}">${loc.phone}</a><br>
                        <b>Ghi chú:</b> ${loc.note}
                    </div>
                `);
                markers.push(marker);

                const card = document.createElement('div');
                card.className = 'location-card';
                card.innerHTML = `
                    <h3>${loc.name}</h3>
                    <p><b>SĐT:</b> ${loc.phone}</p>
                    <p><b>Tiện ích:</b> ${loc.note}</p>
                    <span class="tag ${loc.category === 'Y tế' ? 'tag-medical' : 'tag-tourism'}">${loc.category}</span>
                `;

                card.onclick = () => {
                    map.flyTo([loc.lat, loc.lng], 16);
                    marker.openPopup();
                };

                listContainer.appendChild(card);
            });
        }

        function filterLocations(cat, event) {
            document.querySelectorAll('.filter-btn').forEach(btn => btn.classList.remove('active'));
            if(event) event.target.classList.add('active');

            if (cat === 'all') {
                displayLocations(locations);
            } else {
                const filtered = locations.filter(l => l.category === cat);
                displayLocations(filtered);
            }
        }

        function locateUser() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition((position) => {
                    const lat = position.coords.latitude;
                    const lng = position.coords.longitude;

                    if (userMarker) map.removeLayer(userMarker);

                    userMarker = L.circleMarker([lat, lng], {
                        color: '#0288d1',
                        fillColor: '#29b6f6',
                        fillOpacity: 0.8,
                        radius: 10
                    }).addTo(map);

                    userMarker.bindPopup("<b>Vị trí hiện tại của bạn</b>").openPopup();
                    map.flyTo([lat, lng], 15);
                }, () => {
                    alert("Không thể lấy vị trí hiện tại. Vui lòng bật GPS!");
                });
            } else {
                alert("Trình duyệt không hỗ trợ định vị GPS.");
            }
        }

        displayLocations(locations);
    </script>
</body>
</html>
