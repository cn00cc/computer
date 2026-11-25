        <div class="roast-box">
            <p id="roast-comment" class="roast-text"></p>
            <p id="diff-detail" class="diff-detail"></p>
        </div>

        <button onclick="copyShareText()" id="copy-btn" class="btn-copy">📋 複製鑑定結果文案（貼群組嘲諷）</button>
        <button onclick="resetForm()" class="btn-reset">再鑑定下一個商品 🔄</button>
    </section>

    <script>
        let compressedBase64 = "";
        let latestShareText = "";

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

            document.getElementById('loading-text').innerText = "Gemini AI 正在辨識商品型號並搜尋台灣市價...";
            try {
                const promptText = `你是一個精通台灣電商（蝦皮、momo、PChome、酷澎、實體通路）行情的比價專家。
請辨識這張圖片中的商品品牌、完整名稱與型號，並估算出該商品在台灣市場通常能買到的「合理最低入手價（新台幣整數）」。
請嚴格只回傳以下 JSON 格式，不要包含任何 Markdown 標記或多餘說明文字：
{
  "productName": "品牌 + 商品完整名稱與型號",
  "marketPrice": 1200,
  "priceNote": "簡短說明行情依據（例如：蝦皮/momo 常見特價區間）"
}`;

                const response = await fetch(`https://googleapis.com{encodeURIComponent(apiKey)}`, {
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
                const cleanJson = rawText.replace(/```json|```/g, '').trim();
                const parsed = JSON.parse(cleanJson);

                const detectedName = parsed.productName || "未知商品";
                const detectedPrice = Math.max(1, parseInt(parsed.marketPrice, 10) || Math.round(userPrice * 0.8));
                const priceNote = parsed.priceNote || "依台灣常見電商行情估算";

                renderResult(detectedName, detectedPrice, priceNote, userPrice);
            } catch (err) {
                alert(`辨識失敗：${err.message}`);
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

            document.getElementById('product-name').innerText = `辨識商品：${productName}`;
            document.getElementById('product-note').innerText = priceNote;
            document.getElementById('display-user-price').innerText = `$${userPrice.toLocaleString()}`;
            document.getElementById('display-market-price').innerText = `$${marketPrice.toLocaleString()}`;

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
                    : `比市場參考價還省了 $${Math.abs(diff).toLocaleString()}（多賺 ${bobaCount} 杯珍奶）！`;
            } else if (percent <= 15) {
                card.style.borderColor = '#fbbf24';
                badge.style.backgroundColor = 'rgba(245, 158, 11, 0.2)';
                badge.style.color = '#fbbf24';
                badge.innerText = '微盤認證';
                rankString = '精緻小盤子 🍽️';
                roast.style.color = '#fde68a';
                roastString = '「稍微買貴了一點點，沒關係，就當作花錢買快樂跟早享受！」';
                detailString = `比市場參考價貴了 $${diff.toLocaleString()}（多付 ${percent}%，少喝 ${bobaCount} 杯珍奶）`;
            } else {
                card.style.borderColor = '#ef4444';
                badge.style.backgroundColor = 'rgba(239, 68, 68, 0.2)';
                badge.style.color = '#f87171';
                badge.innerText = '警報！發現肥羊';
                rankString = '頂級黃金大盤 🏆';
                roast.style.color = '#fca5a5';
                roastString = '「笑死，這價格你都買得下去？店員現在應該在後面開香檳慶祝遇到大慈善家了！」';
                detailString = `比市場參考價足足貴了 $${diff.toLocaleString()}（貴了 ${percent}%，直接飛走 ${bobaCount} 杯珍奶！）`;
            }

            title.innerText = rankString;
            roast.innerText = roastString;
            detail.innerText = detailString;

            latestShareText = `【🍽️ 盤子鑑定機報告】\n📦 鑑定商品：${productName}\n💸 他的入手價：$${userPrice.toLocaleString()}\n🏷️ 市場參考價：$${marketPrice.toLocaleString()}\n🏆 鑑定稱號：${rankString}\n📊 價差分析：${detailString}\n💬 AI 評語：${roastString}`;

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
    </script>
</body>
</html>
