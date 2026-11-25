<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>盤子鑑定機 🔍 買貴直接笑</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            background-color: #0f172a;
            color: #ffffff;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans TC", sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px 16px;
        }
        header { text-align: center; margin: 16px 0 20px; }
        header h1 { font-size: 28px; font-weight: 900; color: #fbbf24; letter-spacing: 1px; }
        header p { color: #94a3b8; font-size: 14px; margin-top: 6px; }

        .card {
            width: 100%;
            max-width: 420px;
            background-color: #1e293b;
            border-radius: 16px;
            padding: 24px;
            border: 1px solid #334155;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
        }

        /* API 設定折疊區 */
        .api-settings {
            margin-bottom: 18px;
            background-color: rgba(15, 23, 42, 0.6);
            border: 1px solid #334155;
            border-radius: 10px;
            padding: 10px 14px;
            font-size: 13px;
        }
        .api-settings summary {
            cursor: pointer;
            color: #94a3b8;
            font-weight: bold;
            user-select: none;
        }
        .api-settings summary:hover { color: #fbbf24; }
        .api-input-group {
            display: flex;
            gap: 8px;
            margin-top: 10px;
        }
        .api-input-group input {
            flex: 1;
            background-color: #0f172a;
            border: 1px solid #475569;
            border-radius: 8px;
            padding: 8px 10px;
            color: #ffffff;
            font-size: 12px;
            outline: none;
        }
        .api-input-group input:focus { border-color: #fbbf24; }
        .btn-save-key {
            background-color: #334155;
            color: #fbbf24;
            border: 1px solid #475569;
            border-radius: 8px;
            padding: 0 12px;
            font-size: 12px;
            font-weight: bold;
            cursor: pointer;
        }
        .api-hint {
            font-size: 11px;
            color: #64748b;
            margin-top: 6px;
            line-height: 1.4;
        }

        .form-group { margin-bottom: 20px; }
        label { display: block; font-size: 14px; font-weight: bold; color: #cbd5e1; margin-bottom: 8px; }

        .upload-box {
            border: 2px dashed #475569;
            border-radius: 12px;
            padding: 28px 16px;
            text-align: center;
            cursor: pointer;
            background-color: rgba(15, 23, 42, 0.5);
            transition: border-color 0.2s;
        }
        .upload-box:active { border-color: #fbbf24; }
        .upload-icon { font-size: 36px; display: block; margin-bottom: 8px; }
        .upload-title { font-size: 15px; font-weight: bold; color: #e2e8f0; }
        .upload-sub { font-size: 12px; color: #64748b; margin-top: 4px; }

        #image-input { display: none; }
        #image-preview { max-height: 220px; max-width: 100%; border-radius: 8px; margin: 0 auto; display: none; }

        .input-wrapper { position: relative; }
        .currency { position: absolute; left: 14px; top: 50%; transform: translateY(-50%); color: #94a3b8; font-weight: bold; font-size: 18px; }
        input[type="number"] {
            width: 100%;
            background-color: #0f172a;
            border: 1px solid #475569;
            border-radius: 12px;
            padding: 14px 14px 14px 34px;
            color: #ffffff;
            font-size: 18px;
            font-weight: bold;
            outline: none;
        }
        input[type="number"]:focus { border-color: #fbbf24; }

        .btn-primary {
            width: 100%;
            background-color: #f59e0b;
            color: #0f172a;
            font-size: 18px;
            font-weight: 900;
            padding: 14px;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            box-shadow: 0 4px 14px rgba(245, 158, 11, 0.3);
        }
        .btn-primary:active { transform: scale(0.98); background-color: #fbbf24; }

        .loading { display: none; text-align: center; margin-top: 30px; }
        .spinner {
            width: 38px; height: 38px;
            border: 4px solid #fbbf24;
            border-top-color: transparent;
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
            margin: 0 auto;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
        .loading p { color: #fbbf24; font-weight: bold; margin-top: 12px; font-size: 15px; }

        /* 結果卡片樣式 */
        .result-card {
            display: none;
            width: 100%;
            max-width: 420px;
            margin-top: 24px;
            border-radius: 16px;
            padding: 24px;
            background-color: #1e293b;
            border: 2px solid #ef4444;
            box-shadow: 0 12px 30px rgba(0, 0, 0, 0.5);
        }
        .result-header { text-align: center; border-bottom: 1px solid #334155; padding-bottom: 16px; }
        .badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 999px;
            font-size: 12px;
            font-weight: 900;
            margin-bottom: 8px;
        }
        .rank-title { font-size: 26px; font-weight: 900; margin-top: 4px; }
        .product-name { font-size: 14px; color: #e2e8f0; font-weight: bold; margin-top: 8px; }
        .product-note { font-size: 12px; color: #94a3b8; margin-top: 4px; }

        .price-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin: 18px 0; text-align: center; }
        .price-box { background-color: rgba(15, 23, 42, 0.8); padding: 12px; border-radius: 12px; border: 1px solid #334155; }
        .price-label { font-size: 12px; color: #94a3b8; }
        .price-val { font-size: 20px; font-weight: 900; margin-top: 4px; }

        .roast-box { background-color: rgba(2, 6, 23, 0.7); border-radius: 12px; padding: 16px; border: 1px solid #1e293b; }
        .roast-text { font-size: 15px; font-weight: bold; line-height: 1.5; }
        .diff-detail { font-size: 12px; color: #94a3b8; margin-top: 8px; }

        .btn-copy {
            width: 100%;
            margin-top: 16px;
            background-color: #10b981;
            color: #0f172a;
            font-weight: 900;
            padding: 13px;
            border: none;
            border-radius: 12px;
            font-size: 15px;
            cursor: pointer;
            box-shadow: 0 4px 12px rgba(16, 185, 129, 0.25);
        }
        .btn-copy:active { transform: scale(0.98); }

        .btn-reset {
            width: 100%;
            margin-top: 10px;
            background-color: #334155;
            color: #ffffff;
            font-weight: bold;
            padding: 12px;
            border: none;
            border-radius: 12px;
            font-size: 14px;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <header>
        <h1>🍽️ 盤子鑑定機</h1>
        <p>拍照比價，看看你朋友是不是財富自由大慈善家</p>
    </header>

    <main class="card">
        <details class="api-settings" id="api-details">
            <summary id="api-status-text">⚙️ Gemini AI 真實辨識設定（未設定，目前為模擬模式）</summary>
            <div class="api-input-group">
                <input type="password" id="gemini-api-key" placeholder="貼上你的 Gemini API Key">
                <button type="button" class="btn-save-key" onclick="saveApiKey()">儲存</button>
            </div>
            <p class="api-hint">填寫 Google AI Studio 免費申請的 API Key，即可讓 AI 自動辨識商品型號並估算台灣市場最低價（金鑰僅儲存在你的手機/瀏覽器本機）。</p>
        </details>

        <div class="form-group">
            <label>1. 拍下商品或上傳照片</label>
            <div class="upload-box" onclick="document.getElementById('image-input').click()">
                <input type="file" id="image-input" accept="image/*" capture="environment" onchange="previewImage(event)">
                <div id="upload-placeholder">
                    <span class="upload-icon">📸</span>
                    <div class="upload-title">點擊開啟相機 / 選擇照片</div>
                    <div class="upload-sub">支援拍商品外觀、包裝或條碼</div>
                </div>
                <img id="image-preview" alt="商品預覽">
            </div>
        </div>

        <div class="form-group">
            <label for="user-price">2. 他買多少錢？（新台幣）</label>
            <div class="input-wrapper">
                <span class="currency">$</span>
                <input type="number" id="user-price" placeholder="例如：1500">
            </div>
        </div>

        <button onclick="analyzeItem()" id="submit-btn" class="btn-primary">
            開始鑑定是不是盤子 🔍
        </button>
    </main>

    <div id="loading" class="loading">
        <div class="spinner"></div>
        <p id="loading-text">AI 正在全網搜刮最低價...</p>
    </div>

    <section id="result-card" class="result-card">
        <div class="result-header">
            <span id="badge-tag" class="badge">鑑定報告</span>
            <h2 id="rank-title" class="rank-title">頂級黃金大盤 🏆</h2>
            <p id="product-name" class="product-name">辨識商品：範例商品</p>
            <p id="product-note" class="product-note"></p>
        </div>

        <div class="price-grid">
            <div class="price-box">
                <div class="price-label">他的入手價</div>
                <div id="display-user-price" class="price-val" style="color: #f87171;">$0</div>
            </div>
            <div class="price-box">
                <div class="price-label">市場參考最低價</div>
                <div id="display-market-price" class="price-val" style="color: #34d399;">$0</div>
            </div>
        </div>
📋 複製鑑定結果文案（貼群組嘲諷）
再鑑定下一個商品 🔄
let compressedBase64 = "";
let latestShareText = "";
// 頁面載入時讀取已儲存的 API Key
window.addEventListener('DOMContentLoaded', () => {
const savedKey = localStorage.getItem('gemini_api_key');
if (savedKey) {
document.getElementById('gemini-api-key').value = savedKey;
document.getElementById('api-status-text').innerText = "✅ Gemini AI 真實辨識已就緒（點擊展開修改）";
document.getElementById('api-status-text').style.color = "#34d399";
}
});
function saveApiKey() {
const key = document.getElementById('gemini-api-key').value.trim();
if (key) {
localStorage.setItem('gemini_api_key', key);
document.getElementById('api-status-text').innerText = "✅ Gemini AI 真實辨識已就緒（點擊展開修改）";
document.getElementById('api-status-text').style.color = "#34d399";
document.getElementById('api-details').removeAttribute('open');
alert('API Key 已儲存至本機瀏覽器！');
} else {
localStorage.removeItem('gemini_api_key');
document.getElementById('api-status-text').innerText = "⚙️ Gemini AI 真實辨識設定（未設定，目前為模擬模式）";
document.getElementById('api-status-text').style.color = "#94a3b8";
alert('已清除 API Key，切換回模擬模式。');
}
}
// 預覽並自動壓縮圖片（避免手機照片過大導致 API 超時）
function previewImage(event) {
const file = event.target.files[0];
if (!file) return;
const reader = new FileReader();
reader.onload = function(e) {
const img = new Image();
img.onload = function() {
const canvas = document.createElement('canvas');
const maxDim = 1024;
let width = img.width;
let height = img.height;
if (width > maxDim || height > maxDim) {
if (width > height) {
height = Math.round((height * maxDim) / width);
width = maxDim;
} else {
width = Math.round((width * maxDim) / height);
height = maxDim;
}
}
canvas.width = width;
canvas.height = height;
const ctx = canvas.getContext('2d');
ctx.drawImage(img, 0, 0, width, height);
const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.85);
compressedBase64 = compressedDataUrl.split(',')[1];
const preview = document.getElementById('image-preview');
const placeholder = document.getElementById('upload-placeholder');
preview.src = compressedDataUrl;
preview.style.display = 'block';
placeholder.style.display = 'none';
};
img.src = e.target.result;
};
reader.readAsDataURL(file);
}
async function analyzeItem() {
const imageInput = document.getElementById('image-input');
const userPrice = parseFloat(document.getElementById('user-price').value);
const apiKey = document.getElementById('gemini-api-key').value.trim();
if (!imageInput.files.length || !compressedBase64) {
alert('請先拍一張照片或上傳圖片！');
return;
}
if (!userPrice || userPrice <= 0) {
alert('請輸入有效的購買金額！');
return;
}
document.getElementById('submit-btn').style.display = 'none';
document.getElementById('result-card').style.display = 'none';
document.getElementById('loading').style.display = 'block';
// 若未填寫 API Key，使用模擬模式
if (!apiKey) {
document.getElementById('loading-text').innerText = "模擬模式比價中（填入 API Key 可真實辨識）...";
setTimeout(() => {
document.getElementById('loading').style.display = 'none';
document.getElementById('submit-btn').style.display = 'block';
const mockMarketPrice = Math.max(1, Math.round(userPrice * 0.72));
renderResult("測試辨識商品（尚未設定 Gemini API Key）", mockMarketPrice, "目前為模擬估算價格", userPrice);
}, 1200);
return;
}
// 呼叫 Gemini Vision API 辨識商品與台灣市價
document.getElementById('loading-text').innerText = "Gemini AI 正在辨識商品型號並搜尋台灣市價...";
try {
const promptText = 你是一個精通台灣電商（蝦皮、momo、PChome、酷澎、實體通路）行情的比價專家。 請辨識這張圖片中的商品品牌、完整名稱與型號，並估算出該商品在台灣市場通常能買到的「合理最低入手價（新台幣整數）」。 請嚴格只回傳以下 JSON 格式，不要包含任何 Markdown 標記或多餘說明文字： { "productName": "品牌 + 商品完整名稱與型號", "marketPrice": 1200, "priceNote": "簡短說明行情依據（例如：蝦皮/momo 常見特價區間）" };
const response = await fetch(https://googleapis.com{encodeURIComponent(apiKey)}, {
method: 'POST',
headers: { 'Content-Type': 'application/json' },
body: JSON.stringify({
contents: [{
parts: [
{ text: promptText },
{
inlineData: {
mimeType: "image/jpeg",
data: compressedBase64
}
}
]
}],
generationConfig: {
temperature: 0.2,
responseMimeType: "application/json"
}
})
});
const data = await response.json();
if (!response.ok) {
throw new Error(data.error?.message || "API 呼叫失敗，請檢查 API Key 是否正確");
}
const rawText = data.candidates[0].content.parts[0].text;
const cleanJson = rawText.replace(/json|/g, '').trim();
const parsed = JSON.parse(cleanJson);
const detectedName = parsed.productName || "未知商品";
const detectedPrice = Math.max(1, parseInt(parsed.marketPrice, 10) || Math.round(userPrice * 0.8));
const priceNote = parsed.priceNote || "依台灣常見電商行情估算";
renderResult(detectedName, detectedPrice, priceNote, userPrice);
} catch (err) {
alert(辨識失敗：${err.message});
} finally {
document.getElementById('loading').style.display = 'none';
document.getElementById('submit-btn').style.display = 'block';
}
}
function renderResult(productName, marketPrice, priceNote, userPrice) {
const diff = userPrice - marketPrice;
const percent = Math.round((diff / marketPrice) * 100);
const bobaCount = Math.floor(Math.abs(diff) / 65);
const card = document.getElementById('result-card');
const badge = document.getElementById('badge-tag');
const title = document.getElementById('rank-title');
const roast = document.getElementById('roast-comment');
const detail = document.getElementById('diff-detail');
document.getElementById('product-name').innerText = 辨識商品：${productName};
document.getElementById('product-note').innerText = priceNote;
document.getElementById('display-user-price').innerText = $${userPrice.toLocaleString()};
document.getElementById('display-market-price').innerText = $${marketPrice.toLocaleString()};
let rankString = "";
let roastString = "";
let detailString = "";
if (diff <= 0) {
card.style.borderColor = '#10b981';
badge.style.backgroundColor = 'rgba(16, 185, 129, 0.2)';
badge.style.color = '#34d399';
badge.innerText = '省錢大師';
rankString = '超級精算師 🧠';
roast.style.color = '#6ee7b7';
roastString = '「太神啦！你買得比網路最低價還便宜，老闆看到你都要虧錢哭出來了！」';
detailString = diff === 0
? '剛好買在全網最低價！'
: 比市場參考價還省了 $${Math.abs(diff).toLocaleString()}（多賺 ${bobaCount} 杯珍奶）！;
} else if (percent <= 15) {
card.style.borderColor = '#fbbf24';
badge.style.backgroundColor = 'rgba(245, 158, 11, 0.2)';
badge.style.color = '#fbbf24';
badge.innerText = '微盤認證';
rankString = '精緻小盤子 🍽️';
roast.style.color = '#fde68a';
roastString = '「稍微買貴了一點點，沒關係，就當作花錢買快樂跟早享受！」';
detailString = 比市場參考價貴了 $${diff.toLocaleString()}（多付 ${percent}%，少喝 ${bobaCount} 杯珍奶）;
} else {
card.style.borderColor = '#ef4444';
badge.style.backgroundColor = 'rgba(239, 68, 68, 0.2)';
badge.style.color = '#f87171';
badge.innerText = '警報！發現肥羊';
rankString = '頂級黃金大盤 🏆';
roast.style.color = '#fca5a5';
roastString = '「笑死，這價格你都買得下去？店員現在應該在後面開香檳慶祝遇到大慈善家了！」';
detailString = 比市場參考價足足貴了 $${diff.toLocaleString()}（貴了 ${percent}%，直接飛走 ${bobaCount} 杯珍奶！）;
}
title.innerText = rankString;
roast.innerText = roastString;
detail.innerText = detailString;
// 組合一鍵複製的 LINE 群組嘲諷文案
latestShareText = 【🍽️ 盤子鑑定機報告】 📦 鑑定商品：${productName} 💸 他的入手價：$${userPrice.toLocaleString()} 🏷️ 市場參考價：$${marketPrice.toLocaleString()} 🏆 鑑定稱號：${rankString} 📊 價差分析：${detailString} 💬 AI 評語：${roastString};
const copyBtn = document.getElementById('copy-btn');
copyBtn.innerText = "📋 複製鑑定結果文案（貼群組嘲諷）";
copyBtn.style.backgroundColor = "#10b981";
card.style.display = 'block';
card.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
}
function copyShareText() {
if (!latestShareText) return;
navigator.clipboard.writeText(latestShareText).then(() => {
const copyBtn = document.getElementById('copy-btn');
copyBtn.innerText = "✅ 已複製！快去群組貼上嘲諷朋友";
copyBtn.style.backgroundColor = "#34d399";
setTimeout(() => {
copyBtn.innerText = "📋 複製鑑定結果文案（貼群組嘲諷）";
copyBtn.style.backgroundColor = "#10b981";
}, 2500);
}).catch(() => {
alert("複製失敗，請手動選取文字複製：\n\n" + latestShareText);
});
}
function resetForm() {
document.getElementById('image-input').value = '';
document.getElementById('user-price').value = '';
compressedBase64 = '';
document.getElementById('image-preview').style.display = 'none';
document.getElementById('upload-placeholder').style.display = 'block';
document.getElementById('result-card').style.display = 'none';
}

