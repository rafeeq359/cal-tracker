<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>متتبع السعرات اليومي</title>
    <style>
        :root {
            --primary: #22c55e;
            --primary-dark: #16a34a;
            --bg: #f8fafc;
            --card: #ffffff;
            --text: #1e293b;
            --muted: #64748b;
            --border: #e2e8f0;
        }
        * {
            box-sizing: border-box;
            font-family: system-ui, -apple-system, sans-serif;
        }
        body {
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
        }
        .app-container {
            width: 100%;
            max-width: 480px;
            min-height: 100vh;
            background: var(--bg);
            padding: 20px;
            padding-bottom: 40px;
        }
        .screen {
            display: none;
        }
        .screen.active {
            display: block;
        }
        h1, h2, h3 {
            margin-top: 0;
        }
        .card {
            background: var(--card);
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.04);
            margin-bottom: 20px;
            border: 1px solid var(--border);
        }
        .btn {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 14px 20px;
            border-radius: 14px;
            font-size: 16px;
            cursor: pointer;
            width: 100%;
            font-weight: bold;
            transition: background 0.2s;
            text-align: center;
            display: block;
        }
        .btn:hover {
            background-color: var(--primary-dark);
        }
        input, select {
            width: 100%;
            padding: 14px;
            border-radius: 12px;
            border: 1px solid var(--border);
            font-size: 16px;
            margin-bottom: 15px;
            background: #fff;
        }
        label {
            display: block;
            margin-bottom: 6px;
            font-weight: 600;
            color: var(--text);
            font-size: 14px;
        }
        .header-top {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }
        .date-display {
            font-size: 14px;
            color: var(--muted);
            font-weight: 500;
        }
        .progress-card {
            text-align: center;
            position: relative;
        }
        .progress-circle {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            background: conic-gradient(var(--primary) 0deg, #e2e8f0 0deg);
            margin: 0 auto 15px auto;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }
        .progress-inner {
            width: 125px;
            height: 125px;
            background: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-top: 15px;
        }
        .stat-box {
            background: #f8fafc;
            padding: 12px 8px;
            border-radius: 12px;
            border: 1px solid var(--border);
            text-align: center;
        }
        .stat-box span {
            font-size: 16px;
            font-weight: bold;
            color: var(--text);
        }
        .stat-box small {
            color: var(--muted);
            font-size: 12px;
        }
        .meal-item {
            display: flex;
            align-items: center;
            background: var(--card);
            padding: 12px;
            border-radius: 14px;
            margin-bottom: 12px;
            border: 1px solid var(--border);
        }
        .meal-item img {
            width: 60px;
            height: 60px;
            border-radius: 10px;
            object-fit: cover;
            margin-left: 15px;
        }
        .meal-info {
            flex-grow: 1;
        }
        .meal-info h4 {
            margin: 0 0 5px 0;
            font-size: 16px;
        }
        .meal-info p {
            margin: 0;
            font-size: 13px;
            color: var(--muted);
        }
        .delete-btn {
            background: none;
            border: none;
            color: #ef4444;
            cursor: pointer;
            font-size: 18px;
            padding: 5px;
        }
        #loading-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 999;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            color: white;
        }
        .spinner {
            width: 50px; height: 50px;
            border: 5px solid #f3f3f3;
            border-top: 5px solid var(--primary);
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-bottom: 15px;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        input[type="file"] { display: none; }
    </style>
</head>
<body>

<div class="app-container">

    <!-- شاشة الإعداد الأولية -->
    <div id="setup-screen" class="screen active">
        <div style="text-align: center; margin: 30px 0;">
            <h1 style="color: var(--primary);">🎯 متتبع السعرات</h1>
            <p style="color: var(--muted);">حدد هدفك اليومي من السعرات لنبدأ</p>
        </div>
        <div class="card">
            <label for="target-input">هدف السعرات اليومي (سعرة)</label>
            <input type="number" id="target-input" placeholder="مثال: 2000" value="2000">
            
            <label for="api-key-input">مفتاح Gemini API</label>
            <input type="password" id="api-key-input" placeholder="AIzaSy...">
            <small style="color: var(--muted); display: block; margin-top: -10px; margin-bottom: 20px;">يتم حفظ بياناتك ومفتاحك محلياً على جهازك فقط.</small>
            
            <button class="btn" onclick="saveSetup()">ابدأ التطبيق 🚀</button>
        </div>
    </div>

    <!-- الشاشة الرئيسية -->
    <div id="dashboard-screen" class="screen">
        <div class="header-top">
            <div>
                <h2 style="margin:0;">🍽️ سجل اليوم</h2>
                <div class="date-display" id="current-date"></div>
            </div>
            <button onclick="resetData()" style="background:none; border:1px solid var(--border); padding:8px 12px; border-radius:10px; cursor:pointer; color:var(--muted); font-size:13px;">⚙️ الإعدادات</button>
        </div>

        <!-- ملخص السعرات والتقدم -->
        <div class="card progress-card">
            <div class="progress-circle" id="progress-circle">
                <div class="progress-inner">
                    <span id="consumed-calories" style="font-size: 24px; font-weight: bold; color: var(--primary);">0</span>
                    <small style="color: var(--muted);">من <span id="target-display">2000</span></small>
                </div>
            </div>
            <p id="remaining-text" style="margin: 0; font-weight: 500; color: var(--muted);">المتبقي: 2000 سعرة</p>

            <div class="stats-grid">
                <div class="stat-box">
                    <small>بروتين</small><br>
                    <span id="total-protein">0</span>g
                </div>
                <div class="stat-box">
                    <small>كارْب</small><br>
                    <span id="total-carbs">0</span>g
                </div>
                <div class="stat-box">
                    <small>دهون</small><br>
                    <span id="total-fat">0</span>g
                </div>
            </div>
        </div>

        <!-- زر تصوير وجبة -->
        <div class="card" style="text-align: center; padding: 15px;">
            <label for="camera-input" class="btn" style="margin: 0; cursor: pointer;">
                📷 تصوير وجبة جديدة
            </label>
            <input type="file" id="camera-input" accept="image/*" capture="environment" onchange="handleImageCapture(event)">
        </div>

        <!-- سجل الوجبات -->
        <h3>الوجبات المسجلة</h3>
        <div id="meals-container">
            <div style="text-align: center; color: var(--muted); padding: 20px;" id="empty-state">
                لم تسجل أي وجبات اليوم بعد. ابدأ بتصوير وجبتك الأولى!
            </div>
        </div>
    </div>

</div>

<!-- شاشة التحميل -->
<div id="loading-overlay">
    <div class="spinner"></div>
    <div style="font-weight: bold; font-size: 16px;">جاري تحليل الوجبة بالذكاء الاصطناعي...</div>
    <div style="font-size: 13px; color: #cbd5e1; margin-top: 5px;">يرجى الانتظار ثوانٍ معدودة</div>
</div>

<script>
    let state = {
        targetCalories: 2000,
        apiKey: '',
        lastDate: '',
        consumedCalories: 0,
        totalProtein: 0,
        totalCarbs: 0,
        totalFat: 0,
        meals: []
    };

    window.onload = function() {
        loadFromLocalStorage();
        checkNewDay();
        
        if (state.apiKey && state.targetCalories) {
            showScreen('dashboard-screen');
            updateUI();
        } else {
            showScreen('setup-screen');
        }

        const options = { weekday: 'long', year: 'numeric', month: 'short', day: 'numeric' };
        document.getElementById('current-date').innerText = new Date().toLocaleDateString('ar-SA', options);
    };

    function showScreen(screenId) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(screenId).classList.add('active');
    }

    function saveSetup() {
        const target = parseInt(document.getElementById('target-input').value);
        const key = document.getElementById('api-key-input').value.trim();

        if (!target || target <= 0) {
            alert('الرجاء إدخال هدف سعرات صحيح.');
            return;
        }
        if (!key) {
            alert('الرجاء إدخال مفتاح Gemini API.');
            return;
        }

        state.targetCalories = target;
        state.apiKey = key;
        state.lastDate = new Date().toDateString();
        saveToLocalStorage();

        showScreen('dashboard-screen');
        updateUI();
    }

    function resetData() {
        document.getElementById('target-input').value = state.targetCalories;
        document.getElementById('api-key-input').value = state.apiKey;
        showScreen('setup-screen');
    }

    function checkNewDay() {
        const today = new Date().toDateString();
        if (state.lastDate && state.lastDate !== today) {
            // يوم جديد: تصفير العدادات اليومية تلقائياً
            state.consumedCalories = 0;
            state.totalProtein = 0;
            state.totalCarbs = 0;
            state.totalFat = 0;
            state.meals = [];
            state.lastDate = today;
            saveToLocalStorage();
        } else if (!state.lastDate) {
            state.lastDate = today;
            saveToLocalStorage();
        }
    }

    function saveToLocalStorage() {
        localStorage.setItem('cal_tracker_state', JSON.stringify(state));
    }

    function loadFromLocalStorage() {
        const saved = localStorage.getItem('cal_tracker_state');
        if (saved) {
            state = JSON.parse(saved);
        }
    }

    async function handleImageCapture(event) {
        const file = event.target.files[0];
        if (!file) return;

        document.getElementById('loading-overlay').style.display = 'flex';

        const reader = new FileReader();
        reader.onloadend = async function () {
            const base64Image = reader.result.split(',')[1];
            try {
                const nutritionData = await callGeminiAPI(base64Image, file.type);
                if (nutritionData) {
                    addMeal(reader.result, nutritionData);
                }
            } catch (error) {
    console.error(error);
    alert("الخطأ بالتفصيل: " + error.message);
}
            } finally {
                document.getElementById('loading-overlay').style.display = 'none';
                event.target.value = '';
            }
        };
        reader.readAsDataURL(file);
    }

   async function callGeminiAPI(base64Data, mimeType) {
    // تحديث رابط الموديل ليكون بالإصدار القياسي المدعوم حالياً
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${state.apiKey}`;
    
    const prompt = `حلل صورة وجبة الطعام هذه بدقة واستخرج القيم الغذائية فقط على شكل كود JSON خالص بهذا الشكل ودون أي كود إضافي أو علامات ترميز:
    {
      "name": "اسم الوجبة باختصار باللغة العربية",
      "calories": إجمالي السعرات الحرارية (رقم صحيح فقط),
      "protein": جرامات البروتين (رقم صحيح فقط),
      "carbs": جرامات الكربوهيدرات (رقم صحيح فقط),
      "fat": جرامات الدهون (رقم صحيح فقط)
    }`;

    const payload = {
        contents: [{
            parts: [
                { text: prompt },
                { inline_data: { mime_type: mimeType, data: base64Data } }
            ]
        }]
    };

    const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
    });

    const result = await response.json();
    
    // فحص إذا كان هناك خطأ راجع من سيرفر جوجل نفسه
    if (result.error) {
        console.error("Gemini API Error:", result.error);
        alert("خطأ من جوجل API: " + result.error.message);
        throw new Error(result.error.message);
    }

    try {
        const textResponse = result.candidates[0].content.parts[0].text;
        // تنظيف النص بدقة لاستخراج الـ JSON
        const cleanJson = textResponse.replace(/```json/g, '').replace(/```/g, '').trim();
        return JSON.parse(cleanJson);
    } catch (e) {
        console.error("Parsing Error. Raw response was:", result);
        throw new Error("فشل في قراءة بيانات الوجبة من الذكاء الاصطناعي.");
    }
}
    function addMeal(imageSrc, data) {
        const mealObj = {
            id: Date.now(),
            image: imageSrc,
            name: data.name,
            calories: data.calories,
            protein: data.protein,
            carbs: data.carbs,
            fat: data.fat
        };

        state.meals.unshift(mealObj);
        state.consumedCalories += data.calories;
        state.totalProtein += data.protein;
        state.totalCarbs += data.carbs;
        state.totalFat += data.fat;

        saveToLocalStorage();
        updateUI();
    }

    function deleteMeal(id) {
        const index = state.meals.findIndex(m => m.id === id);
        if (index !== -1) {
            const meal = state.meals[index];
            state.consumedCalories -= meal.calories;
            state.totalProtein -= meal.protein;
            state.totalCarbs -= meal.carbs;
            state.totalFat -= meal.fat;
            
            state.meals.splice(index, 1);
            saveToLocalStorage();
            updateUI();
        }
    }

    function updateUI() {
        document.getElementById('consumed-calories').innerText = state.consumedCalories;
        document.getElementById('target-display').innerText = state.targetCalories;
        
        const remaining = Math.max(0, state.targetCalories - state.consumedCalories);
        document.getElementById('remaining-text').innerText = `المتبقي: ${remaining} سعرة`;

        document.getElementById('total-protein').innerText = state.totalProtein;
        document.getElementById('total-carbs').innerText = state.totalCarbs;
        document.getElementById('total-fat').innerText = state.totalFat;

        let percentage = Math.min(100, (state.consumedCalories / state.targetCalories) * 100);
        let degrees = (percentage / 100) * 360;
        document.getElementById('progress-circle').style.background = `conic-gradient(var(--primary) ${degrees}deg, #e2e8f0 ${degrees}deg)`;

        const container = document.getElementById('meals-container');
        if (state.meals.length === 0) {
            container.innerHTML = `<div style="text-align: center; color: var(--muted); padding: 20px;">لم تسجل أي وجبات اليوم بعد. ابدأ بتصوير وجبتك الأولى!</div>`;
            return;
        }

        let html = '';
        state.meals.forEach(m => {
            html += `
                <div class="meal-item">
                    <img src="${m.image}" alt="وجبة">
                    <div class="meal-info">
                        <h4>${m.name}</h4>
                        <p>${m.calories} سعرة | P: ${m.protein}g | C: ${m.carbs}g | F: ${m.fat}g</p>
                    </div>
                    <button class="delete-btn" onclick="deleteMeal(${m.id})" title="حذف">🗑️</button>
                </div>
            `;
        });
        container.innerHTML = html;
    }
</script>

</body>
</html>
