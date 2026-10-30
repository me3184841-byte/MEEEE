<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>A&M Code | لوحة التحكم والإدارة السحابية</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Firebase SDK (Compat Version) -->
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

    <style>
        :root {
            --bg-deep: #050508;
            --bg-card: #0e0e14;
            --burgundy: #990033;
            --gold-accent: #d4af37;
            --gold-glow: rgba(212, 175, 55, 0.35);
            --text-white: #f8fafc;
            --text-muted: #94a3b8;
            --border-glow: rgba(212, 175, 55, 0.2);
            --success-color: #10b981;
            --danger-color: #ef4444;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Cairo', sans-serif; }
        body { background: var(--bg-deep); color: var(--text-white); padding: 30px 20px; min-height: 100vh; }
        
        .admin-container { max-width: 1200px; margin: 0 auto; }
        
        .admin-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding-bottom: 25px;
            border-bottom: 1px solid var(--border-glow);
            margin-bottom: 30px;
            flex-wrap: wrap;
            gap: 20px;
        }

        .admin-title { font-size: 1.8rem; font-weight: 800; color: var(--gold-accent); display: flex; align-items: center; gap: 12px; }
        
        /* Remote Control Toggle Box */
        .status-card {
            background: var(--bg-card);
            border: 1.5px solid var(--border-glow);
            border-radius: 20px;
            padding: 25px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 35px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }

        .status-info h3 { font-size: 1.2rem; margin-bottom: 4px; }
        .status-info p { color: var(--text-muted); font-size: 0.9rem; }

        .toggle-switch {
            position: relative;
            width: 70px;
            height: 36px;
        }

        .toggle-switch input { opacity: 0; width: 0; height: 0; }
        .slider {
            position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0;
            background-color: #334155; transition: .4s; border-radius: 34px;
        }
        .slider:before {
            position: absolute; content: ""; height: 26px; width: 26px; left: 5px; bottom: 5px;
            background-color: white; transition: .4s; border-radius: 50%;
        }
        input:checked + .slider { background-color: var(--success-color); }
        input:checked + .slider:before { transform: translateX(34px); }

        /* Orders Grid */
        .orders-section-title { font-size: 1.4rem; font-weight: 700; margin-bottom: 20px; display: flex; align-items: center; gap: 10px; }
        .orders-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 20px; }
        
        .order-card {
            background: var(--bg-card);
            border: 1px solid var(--border-glow);
            border-radius: 18px;
            padding: 22px;
            position: relative;
            transition: 0.3s ease;
        }

        .order-card:hover { border-color: var(--gold-accent); transform: translateY(-4px); }
        .order-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 10px; }
        .client-name { font-weight: 800; font-size: 1.1rem; color: var(--text-white); }
        .order-time { font-size: 0.8rem; color: var(--text-muted); }
        
        .order-body p { margin-bottom: 8px; font-size: 0.92rem; color: var(--text-muted); }
        .order-body strong { color: var(--text-white); }

        .btn-delete {
            background: rgba(239, 68, 68, 0.15);
            border: 1px solid var(--danger-color);
            color: var(--danger-color);
            padding: 8px 16px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: 700;
            font-size: 0.85rem;
            margin-top: 15px;
            width: 100%;
            transition: 0.3s;
        }

        .btn-delete:hover { background: var(--danger-color); color: #fff; }
    </style>
</head>
<body>

    <div class="admin-container">
        <!-- Header -->
        <div class="admin-header">
            <h1 class="admin-title">
                <i class="fa-solid fa-gauge-high"></i> لوحة تحكم A&M Code
            </h1>
            <span style="color: var(--gold-accent); font-weight: 700;">Firebase Live System</span>
        </div>

        <!-- Remote Service Switch -->
        <div class="status-card">
            <div class="status-info">
                <h3>حالة استلام الطلبات للموقع (Turn ON / Turn OFF)</h3>
                <p id="status-text">جاري فحص حالة السيرفر...</p>
            </div>
            <label class="toggle-switch">
                <input type="checkbox" id="service-toggle" onchange="toggleServiceStatus(this.checked)">
                <span class="slider"></span>
            </label>
        </div>

        <!-- Orders List -->
        <h2 class="orders-section-title"><i class="fa-solid fa-list-check" style="color:var(--gold-accent);"></i> الطلبات الواردة مباشرة:</h2>
        <div class="orders-grid" id="orders-container">
            <p style="color:var(--text-muted);">جاري تحميل الطلبات من Firebase...</p>
        </div>
    </div>

    <script>
        // 1. Firebase Configuration (am-code-project)
        const firebaseConfig = {
            apiKey: "AIzaSyBWD_zh4NHJveukQtV_AACX8sZnlr_UYHY",
            authDomain: "am-code-project.firebaseapp.com",
            projectId: "am-code-project",
            storageBucket: "am-code-project.firebasestorage.app",
            messagingSenderId: "86082667576",
            appId: "1:86082667576:web:7022195d92a3bb26ca12cd",
            measurementId: "G-E2M0WF57LT"
        };

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // 2. Fetch & Listen to Service Availability Switch (Turn ON / OFF)
        db.collection("settings").doc("service_status").onSnapshot(doc => {
            if (doc.exists) {
                const isOpen = doc.data().isOpen;
                document.getElementById('service-toggle').checked = isOpen;
                document.getElementById('status-text').innerText = isOpen ? "الخدمة مفعّلة ومفتوحة لاستلام الطلبات (ON)" : "الخدمة متوقفة مؤقتاً عن استلام الطلبات (OFF)";
                document.getElementById('status-text').style.color = isOpen ? "var(--success-color)" : "var(--danger-color)";
            } else {
                db.collection("settings").doc("service_status").set({ isOpen: true });
            }
        });

        function toggleServiceStatus(status) {
            db.collection("settings").doc("service_status").set({ isOpen: status });
        }

        // 3. Real-time Orders Synchronization
        db.collection("orders").orderBy("timestamp", "desc").onSnapshot(snapshot => {
            const container = document.getElementById('orders-container');
            if (snapshot.empty) {
                container.innerHTML = '<p style="color:var(--text-muted); grid-column: 1/-1; text-align: center;">لا توجد طلبات جديدة حالياً.</p>';
                return;
            }

            container.innerHTML = '';
            snapshot.forEach(doc => {
                const data = doc.data();
                const timeStr = data.timestamp ? new Date(data.timestamp.toDate()).toLocaleString('ar-EG') : 'الآن';

                const card = `
                    <div class="order-card">
                        <div class="order-header">
                            <span class="client-name">${data.name || 'عميل'}</span>
                            <span class="order-time">${timeStr}</span>
                        </div>
                        <div class="order-body">
                            <p><strong>الهاتف:</strong> ${data.phone || 'غير محدد'}</p>
                            <p><strong>نوع الموقع:</strong> ${data.type || 'عام'}</p>
                            <p><strong>المميزات:</strong> ${data.features ? data.features.join(', ') : 'بدون'}</p>
                            ${data.notes ? `<p><strong>ملاحظات:</strong> ${data.notes}</p>` : ''}
                        </div>
                        <button class="btn-delete" onclick="deleteOrder('${doc.id}')"><i class="fa-solid fa-trash"></i> مسح الطلب</button>
                    </div>
                `;
                container.innerHTML += card;
            });
        });

        function deleteOrder(id) {
            if (confirm('هل أنت تأكد من مسح هذا الطلب؟')) {
                db.collection("orders").doc(id).delete();
            }
        }
    </script>
</body>
</html>
