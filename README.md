<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>متجر فك الاحتكار</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    body {
      background-color: #0f172a;
      color: #ffffff;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      margin: 0;
      padding: 16px;
    }
    header {
      text-align: center;
      margin-bottom: 20px;
    }
    h1 {
      font-size: 22px;
      color: #38bdf8;
      margin: 0;
    }
    .status {
      font-size: 13px;
      color: #94a3b8;
      margin-top: 5px;
    }
    .search-box {
      width: 100%;
      box-sizing: border-box;
      padding: 12px;
      border-radius: 10px;
      border: 1px solid #334155;
      background: #1e293b;
      color: #fff;
      font-size: 14px;
      margin-bottom: 16px;
    }
    .app-card {
      background: #1e293b;
      border-radius: 12px;
      padding: 14px;
      margin-bottom: 12px;
      border: 1px solid #334155;
    }
    .app-name {
      font-size: 16px;
      font-weight: bold;
      color: #f8fafc;
    }
    .app-category {
      display: inline-block;
      background: #0284c7;
      color: #fff;
      font-size: 11px;
      padding: 2px 8px;
      border-radius: 6px;
      margin: 6px 0;
    }
    .app-desc {
      font-size: 13px;
      color: #cbd5e1;
      margin-bottom: 10px;
    }
    .btn-action {
      background: #38bdf8;
      color: #0f172a;
      border: none;
      padding: 10px;
      width: 100%;
      border-radius: 8px;
      font-weight: bold;
      font-size: 14px;
      cursor: pointer;
    }
    .empty-state {
      text-align: center;
      padding: 40px 10px;
      color: #94a3b8;
    }
  </style>
</head>
<body>

  <header>
    <h1>🛍️ متجر فك الاحتكار</h1>
    <div class="status" id="status-text">جاري جلب التطبيقات...</div>
  </header>

  <input type="text" id="search-input" class="search-box" placeholder="🔍 ابحث عن تطبيق..." onkeyup="filterApps()">

  <div id="apps-container"></div>

  <script>
    const tg = window.Telegram?.WebApp;
    if (tg) {
      tg.expand();
      tg.ready();
    }

    let allApps = [];

    async function loadApps() {
      const status = document.getElementById('status-text');
      const container = document.getElementById('apps-container');

      try {
        const res = await fetch('https://telegram-appstore-8cac3-default-rtdb.firebaseio.com/apps.json');
        const data = await res.json();

        if (!data || Object.keys(data).length === 0) {
          status.innerText = "المتجر متصل وقيد العمل";
          container.innerHTML = `
            <div class="empty-state">
              <h3>لا توجد تطبيقات معروضة بعد! 📦</h3>
              <p>قم برفع تطبيق من بوت المطورين واعتماده من الإدارة ليظهر هنا فوراً.</p>
            </div>`;
          return;
        }

        allApps = Object.entries(data)
          .filter(([id, app]) => app && app.status === 'approved')
          .map(([id, app]) => ({ id, ...app }));

        if (allApps.length === 0) {
          status.innerText = "المتجر متصل";
          container.innerHTML = `
            <div class="empty-state">
              <h3>بانتظار موافقة الإدارة ⏳</h3>
              <p>تم استلام التطبيقات وهي قيد المراجعة في بوت الإدارة.</p>
            </div>`;
        } else {
          status.innerText = `عدد التطبيقات المتاحة: ${allApps.length}`;
          renderApps(allApps);
        }

      } catch (err) {
        status.innerText = "تعذر الاتصال بقاعدة البيانات";
        container.innerHTML = `<div class="empty-state">⚠️ حدث خطأ في جلب البيانات</div>`;
      }
    }

    function renderApps(apps) {
      const container = document.getElementById('apps-container');
      container.innerHTML = '';

      apps.forEach(app => {
        const card = document.createElement('div');
        card.className = 'app-card';
        card.innerHTML = `
          <div class="app-name">${app.name}</div>
          <span class="app-category">${app.category || 'عام'}</span>
          <div class="app-desc">${app.description || 'لا يوجد وصف متاح'}</div>
          <button class="btn-action" onclick="openApp('${app.id}')">تنزيل وتفاصيل 📥</button>
        `;
        container.appendChild(card);
      });
    }

    function filterApps() {
      const query = document.getElementById('search-input').value.toLowerCase();
      const filtered = allApps.filter(app => 
        (app.name && app.name.toLowerCase().includes(query)) ||
        (app.category && app.category.toLowerCase().includes(query))
      );
      renderApps(filtered);
    }

    function openApp(appId) {
      if (tg) {
        tg.sendData(JSON.stringify({ action: "view", app_id: appId }));
        tg.close();
      } else {
        alert("يتم الفتح داخل تيليجرام فقط");
      }
    }

    loadApps();
  </script>
</body>
</html>
