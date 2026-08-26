<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>متبع السعرات بالذكاء الاصطناعي</title>
    <style>
        :root {
            --primary: #10B981;
            --bg: #0F172A;
            --card-bg: #1E293B;
            --text: #F8FAFC;
            --text-secondary: #94A3B8;
        }
        body {
            font-family: system-ui, -apple-system, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container { width: 100%; max-width: 480px; }
        header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; }
        h1 { font-size: 1.5rem; margin: 0; }
        .card { background-color: var(--card-bg); border-radius: 16px; padding: 20px; margin-bottom: 20px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
        .progress-circle { text-align: center; margin: 10px 0; }
        .calories-num { font-size: 2.5rem; font-weight: bold; color: var(--primary); }
        .macros { display: flex; justify-content: space-around; margin-top: 15px; text-align: center; }
        .macro-item span { display: block; color: var(--text-secondary); font-size: 0.85rem; }
        .macro-item strong { font-size: 1.1rem; }
        .btn { background-color: var(--primary); color: white; border: none; width: 100%; padding: 14px; border-radius: 12px; font-size: 1rem; font-weight: bold; cursor: pointer; text-align: center; display: block; box-sizing: border-box; }
        input[type="file"] { display: none; }
        .meal-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #334155; }
        .loading { text-align: center; color: var(--text-secondary); margin: 10px 0; display: none; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>🍽️ متبع السعرات اليومي</h1>
        <button class="btn" style="width: auto; padding: 8px 12px; font-size: 0.85rem;" onclick="setApiKey()">⚙️ الإعدادات</button>
    </header>

    <div class="card">
        <div class="progress-circle">
            <div class="calories-num" id="totalCalories">0</div>
            <div style="color: var(--text-secondary);">من <span id="targetCalories">1600</span> سعرة</div>
        </div>
        <div class="macros">
            <div class="macro-item"><span>بروتين</span><strong id="totalProtein">0</strong>g</div>
            <div class="macro-item"><span>كاربو</span><strong id="totalCarbs">0</strong>g</div>
            <div class="macro-item"><span>دهون</span><strong id="totalFat">0</strong>g</div>
        </div>
    </div>

    <div class="card" style="text-align: center;">
        <label for="imageInput" class="btn">📸 تصوير أو اختيار وجبة للتحليل</label>
        <input type="file" id="imageInput" accept="image/*">
        <div class="loading" id="loadingText">جاري تحليل الوجبة عبر الذكاء الاصطناعي... يرجى الانتظار</div>
    </div>

    <div class="card">
        <h3 style="margin-top: 0; font-size: 1.1rem;">الوجبات المسجلة</h3>
        <div id="mealsList" style="color: var(--text-secondary); text-align: center; font-size: 0.9rem;">
            لم تسجل أي وجبات اليوم بعد.
        </div>
    </div>
</div>

<script>
    let state = {
        apiKey: localStorage.getItem('gemini_api_key') || '',
        goal: 1600,
        consumed: 0,
        protein: 0,
        carbs: 0,
        fat: 0,
        meals: []
    };

    function setApiKey() {
        const key = prompt("أدخل مفتاح Gemini API الخاص بك:", state.apiKey);
        if (key !== null) {
            state.apiKey = key.trim();
            localStorage.setItem('gemini_api_key', state.apiKey);
            alert("تم حفظ المفتاح بنجاح!");
        }
    }

    const imageInput = document.getElementById('imageInput');
    const loadingText = document.getElementById('loadingText');

    imageInput.addEventListener('change', async function(e) {
        const file = e.target.files[0];
        if (!file) return;

        if (!state.apiKey) {
            setApiKey();
            if (!state.apiKey) return;
        }

        loadingText.style.display = 'block';

        try {
            const base64Data = await convertFileToBase64(file);
            const analysisResult = await callGeminiAPI(base64Data, file.type);
            addMeal(analysisResult);
        } catch (error) {
            console.error(error);
            alert("حدث خطأ أثناء تحليل الصورة: " + error.message);
        } finally {
            loadingText.style.display = 'none';
            imageInput.value = '';
        }
    });

    function convertFileToBase64(file) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = () => {
                const base64String = reader.result.split(',')[1];
                resolve(base64String);
            };
            reader.onerror = error => reject(error);
        });
    }

    async function callGeminiAPI(base64Data, mimeType) {
    const url = `https://generativelanguage.googleapis.com/v1/models/gemini-3.5-flash:generateContent?key=${state.apiKey}`;
        
        const prompt = `حلل صورة الوجبة هذه بدقة واستخرج القيم الغذائية بناءً على الكمية الظاهرة في الصورة، وأعطني الرد حصراً على شكل كود JSON بهذا الشكل ودون أي كود إضافي أو نصوص خارجية:
        {
          "name": "اسم الوجبة باختصار بالعربية",
          "calories": إجمالي السعرات (رقم فقط),
          "protein": جرامات البروتين (رقم فقط),
          "carbs": جرامات الكربوهيدرات (رقم فقط),
          "fat": جرامات الدهون (رقم فقط)
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

        if (result.error) {
            throw new Error(result.error.message);
        }

        if (!result.candidates || result.candidates.length === 0) {
            throw new Error("لم يتم استقبال استجابة صحيحة من الخادم.");
        }

        const textResponse = result.candidates[0].content.parts[0].text;
        const cleanJson = textResponse.replace(/```json/g, '').replace(/```/g, '').trim();
        return JSON.parse(cleanJson);
    }

    function addMeal(food) {
        state.consumed += food.calories;
        state.protein += food.protein;
        state.carbs += food.carbs;
        state.fat += food.fat;
        state.meals.push(food);
        updateUI();
    }

    function updateUI() {
        document.getElementById('totalCalories').innerText = state.consumed;
        document.getElementById('totalProtein').innerText = state.protein;
        document.getElementById('totalCarbs').innerText = state.carbs;
        document.getElementById('totalFat').innerText = state.fat;

        const mealsList = document.getElementById('mealsList');
        if (state.meals.length === 0) {
            mealsList.innerHTML = 'لم تسجل أي وجبات اليوم بعد.';
            return;
        }

        mealsList.innerHTML = '';
        state.meals.forEach(meal => {
            const div = document.createElement('div');
            div.className = 'meal-item';
            div.innerHTML = `
                <span>${meal.name}</span>
                <span style="color: var(--primary); font-weight: bold;">${meal.calories} سعرة (${meal.protein}g بروتين)</span>
            `;
            mealsList.appendChild(div);
        });
    }
</script>

</body>
</html>
