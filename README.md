<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>注音符號視覺比對練習 (手機排版優化版)</title>
    <style>
        /* 整體介面與視覺化設定 */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f4f8;
            margin: 0; padding: 10px;
            display: flex; flex-direction: column; align-items: center;
            /* 使用 dvh 確保精準貼合手機實際可視範圍 */
            height: 100dvh; box-sizing: border-box;
            touch-action: manipulation; user-select: none;
        }

        #game-container {
            width: 100%; max-width: 700px;
            background-color: #ffffff; border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            padding: 15px; box-sizing: border-box;
            display: flex; flex-direction: column; gap: 15px;
            flex: 1; overflow: hidden;
        }

        /* 題目顯示區 */
        #question-area {
            display: flex; flex-direction: column; align-items: center;
            background-color: #e3f2fd; border-radius: 15px; padding: 15px 10px;
            border: 4px solid #90caf9;
            flex-shrink: 0; /* 確保題目區不會被過度擠壓 */
        }

        /* 圖片容器：設定彈性高度限制 */
        #question-img-container { 
            height: 12vh; min-height: 80px; max-height: 120px;
            display: flex; justify-content: center; align-items: center;
            margin-bottom: 10px; 
        }

        #word-display-container {
            display: flex; justify-content: center; gap: 20px;
        }

        .char-group { display: flex; align-items: center; }

        .char-text {
            font-size: 4rem; font-weight: bold; color: #1565c0;
            margin-right: 10px; line-height: 1;
        }

        /* 雙欄位注音排版 (左主注音、右聲調) */
        .zhuyin-layout { display: flex; gap: 4px; height: 100%; }
        .zhuyin-main { display: flex; flex-direction: column; justify-content: center; gap: 4px; }
        .zhuyin-tone { display: flex; flex-direction: column; gap: 4px; justify-content: flex-end; padding-bottom: 5px; }

        .slot {
            width: 40px; height: 40px;
            border: 3px dashed #90caf9; border-radius: 8px;
            display: flex; justify-content: center; align-items: center;
            font-size: 1.6rem; font-weight: bold; color: #bbdefb;
            background-color: #ffffff;
        }
        .slot.filled { border-style: solid; border-color: #4caf50; background-color: #e8f5e9; color: #2e7d32; }
        .slot.target { border-color: #ff9800; background-color: #fff3e0; color: #ffb74d; }

        /* 虛擬鍵盤區：加入捲動功能 */
        #keyboard-area { 
            flex: 1; display: flex; flex-wrap: wrap; justify-content: center; gap: 6px; align-content: flex-start; 
            overflow-y: auto; /* 允許上下滑動 */
            padding-bottom: 10px; /* 底部預留空間 */
        }
        
        .key-btn {
            width: calc(14% - 6px); min-width: 38px; height: 48px;
            font-size: 1.5rem; font-weight: bold; color: #333;
            background-color: #f5f5f5; border: 2px solid #e0e0e0; border-radius: 8px;
            cursor: pointer; box-shadow: 0 4px 0 #bdbdbd; transition: 0.1s;
            display: flex; justify-content: center; align-items: center;
        }
        .key-btn:active { transform: translateY(4px); box-shadow: 0 0 0 #bdbdbd; }
        
        .space-btn {
            width: 90%; max-width: 400px; height: 55px;
            margin-top: 5px; font-size: 1.5rem; background-color: #e0e0e0;
        }

        /* 國字選擇區 */
        #selection-area { flex: 1; display: none; flex-direction: column; justify-content: center; align-items: center; gap: 15px; }
        .selection-instruction { font-size: 1.5rem; font-weight: bold; color: #f57c00; }
        .word-btn {
            width: 80%; padding: 15px; font-size: 2.5rem; font-weight: bold;
            background-color: #fff9c4; border: 4px solid #fbc02d; border-radius: 15px;
            cursor: pointer; box-shadow: 0 5px 0 #f57f17; color: #333; transition: 0.1s;
        }
        .word-btn:active { transform: translateY(5px); box-shadow: 0 0 0 #f57f17; }

        /* 視覺動畫 */
        #visual-feedback {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            display: none; justify-content: center; align-items: center;
            z-index: 100; font-size: 8rem; background-color: rgba(255,255,255,0.9);
        }
        
        .shake-error { animation: shake 0.4s; border-color: #f44336 !important; background-color: #ffebee !important; }
        @keyframes shake { 0%, 100% {transform: translateX(0);} 25% {transform: translateX(-10px);} 75% {transform: translateX(10px);} }

        /* === 手機版專屬縮放設定 === */
        @media (max-width: 400px) {
            .char-text { font-size: 3rem; margin-right: 5px; }
            .slot { width: 32px; height: 32px; font-size: 1.3rem; }
            .key-btn { height: 42px; font-size: 1.3rem; }
            .space-btn { height: 48px; font-size: 1.2rem; }
            #question-img-container { height: 10vh; min-height: 60px; }
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- 上半部：題目區 -->
        <div id="question-area">
            <div id="question-img-container"></div>
            <div id="word-display-container"></div>
        </div>

        <!-- 下半部：注音鍵盤區 -->
        <div id="keyboard-area">
            <div id="normal-keys" style="display: flex; flex-wrap: wrap; justify-content: center; gap: 6px; width: 100%;"></div>
            <button class="key-btn space-btn" onclick="handleZhuyinClick('空白')">空白鍵 (一聲)</button>
        </div>

        <!-- 下半部：國字選擇區 -->
        <div id="selection-area">
            <div class="selection-instruction">請選出正確的國字：</div>
            <div id="word-options" style="display: flex; flex-direction: column; width: 100%; gap: 15px; align-items: center;"></div>
        </div>
    </div>

    <!-- 視覺回饋覆蓋層 -->
    <div id="visual-feedback"></div>

    <script>
        // ==========================================
        // 老師出題區 (極簡題庫引擎)
        // ==========================================
        const rawQuestions = [
            ["學校", "ㄒㄩㄝˊ ㄒㄧㄠˋ", "🏫"],
            ["老師", "ㄌㄠˇ ㄕ", "👩‍🏫"],
            ["同學", "ㄊㄨㄥˊ ㄒㄩㄝˊ", "🧑‍🎓"],
            ["書包", "ㄕㄨ ㄅㄠ", "🎒"],
            ["鉛筆", "ㄑㄧㄢ ㄅㄧˇ", "✏️"],
            ["爸爸", "ㄅㄚˋ ㄅㄚ˙", "👨"],
            ["媽媽", "ㄇㄚ ㄇㄚ˙", "👩"],
            ["衣服", "ㄧ ㄈㄨˊ", "👕"],
            ["褲子", "ㄎㄨˋ ㄗ˙", "👖"],
            ["水果", "ㄕㄨㄟˇ ㄍㄨㄛˇ", "🍎"]
        ];

        const zhuyinSymbols = [
            "ㄅ","ㄆ","ㄇ","ㄈ","ㄉ","ㄊ","ㄋ","ㄌ","ㄍ","ㄎ","ㄏ","ㄐ","ㄑ","ㄒ",
            "ㄓ","ㄔ","ㄕ","ㄖ","ㄗ","ㄘ","ㄙ","ㄧ","ㄨ","ㄩ","ㄚ","ㄛ","ㄜ","ㄝ",
            "ㄞ","ㄟ","ㄠ","ㄡ","ㄢ","ㄣ","ㄤ","ㄥ","ㄦ","ˊ","ˇ","ˋ","˙"
        ];
        
        const toneSymbols = ['ˊ', 'ˇ', 'ˋ', '˙', '空白'];

        function buildQuestionBank(data) {
            return data.map(item => {
                const fullWord = item[0];
                const zhuyinParts = item[1].split(' ');
                const img = item[2];
                const characters = [];

                for (let i = 0; i < fullWord.length; i++) {
                    const char = fullWord[i];
                    const zyRaw = zhuyinParts[i] || "";
                    const zySymbols = Array.from(zyRaw);
                    
                    if (zySymbols.length > 0 && !['ˊ', 'ˇ', 'ˋ', '˙'].includes(zySymbols[zySymbols.length - 1])) {
                        zySymbols.push('空白');
                    }
                    
                    characters.push({ char: char, zhuyin: zySymbols });
                }
                return { img, fullWord, characters };
            });
        }

        const questionBank = buildQuestionBank(rawQuestions);

        // ==========================================
        // 系統邏輯區
        // ==========================================
        let currentQuestion = null;
        let flatTargetSequence = []; 
        let currentZhuyinIndex = 0; 
        
        const qImgContainer = document.getElementById('question-img-container');
        const wordDisplayContainer = document.getElementById('word-display-container');
        const normalKeysArea = document.getElementById('normal-keys');
        const keyboardArea = document.getElementById('keyboard-area');
        const selectionArea = document.getElementById('selection-area');
        const wordOptions = document.getElementById('word-options');
        const visualFeedback = document.getElementById('visual-feedback');

        function playZhuyinAudio(symbol) {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                let textToSpeak = symbol;
                if (symbol === '空白') textToSpeak = '一聲';
                if (symbol === '˙') textToSpeak = '輕聲';

                const utterance = new SpeechSynthesisUtterance(textToSpeak);
                utterance.lang = 'zh-TW';
                utterance.rate = 0.9; 
                window.speechSynthesis.speak(utterance);
            }
        }

        function initKeyboard() {
            zhuyinSymbols.forEach(symbol => {
                const btn = document.createElement('button');
                btn.className = 'key-btn';
                btn.innerText = symbol;
                btn.onclick = () => handleZhuyinClick(symbol);
                normalKeysArea.appendChild(btn);
            });
        }

        function loadNewQuestion() {
            keyboardArea.style.display = 'flex';
            selectionArea.style.display = 'none';
            currentZhuyinIndex = 0;
            flatTargetSequence = [];
            wordDisplayContainer.innerHTML = '';

            const randomIndex = Math.floor(Math.random() * questionBank.length);
            currentQuestion = questionBank[randomIndex];

            if (currentQuestion.img.includes('.') || currentQuestion.img.includes('http')) {
                qImgContainer.innerHTML = `<img src="${currentQuestion.img}" alt="${currentQuestion.fullWord}" style="max-width: 100%; max-height: 100%; border-radius: 12px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">`;
            } else {
                qImgContainer.innerHTML = `<div style="font-size: 4rem;">${currentQuestion.img}</div>`;
            }

            let globalSlotIndex = 0;

            currentQuestion.characters.forEach(charObj => {
                const charGroup = document.createElement('div');
                charGroup.className = 'char-group';

                const charText = document.createElement('div');
                charText.className = 'char-text';
                charText.innerText = charObj.char;

                const zhuyinLayout = document.createElement('div');
                zhuyinLayout.className = 'zhuyin-layout';
                
                const mainCol = document.createElement('div');
                mainCol.className = 'zhuyin-main';
                
                const toneCol = document.createElement('div');
                toneCol.className = 'zhuyin-tone';

                charObj.zhuyin.forEach(zy => {
                    const isTone = toneSymbols.includes(zy);
                    flatTargetSequence.push(zy); 

                    const slot = document.createElement('div');
                    slot.className = 'slot';
                    slot.id = `slot-${globalSlotIndex}`;
                    slot.innerText = (zy === '空白') ? '␣' : zy; 
                    
                    if(globalSlotIndex === 0) slot.classList.add('target');
                    
                    if (isTone) {
                        if (zy === '˙') {
                            toneCol.style.justifyContent = 'flex-start';
                            toneCol.style.paddingTop = '5px';
                        }
                        toneCol.appendChild(slot);
                    } else {
                        mainCol.appendChild(slot);
                    }
                    globalSlotIndex++;
                });

                zhuyinLayout.appendChild(mainCol);
                zhuyinLayout.appendChild(toneCol);
                charGroup.appendChild(charText);
                charGroup.appendChild(zhuyinLayout);
                wordDisplayContainer.appendChild(charGroup);
            });
        }

        function handleZhuyinClick(clickedSymbol) {
            playZhuyinAudio(clickedSymbol);

            const targetSymbol = flatTargetSequence[currentZhuyinIndex];
            const currentSlot = document.getElementById(`slot-${currentZhuyinIndex}`);

            if (clickedSymbol === targetSymbol) {
                currentSlot.classList.remove('target');
                currentSlot.classList.add('filled');
                currentSlot.innerText = (clickedSymbol === '空白') ? '' : clickedSymbol; 
                
                currentZhuyinIndex++;

                if (currentZhuyinIndex < flatTargetSequence.length) {
                    document.getElementById(`slot-${currentZhuyinIndex}`).classList.add('target');
                } else {
                    setTimeout(enterSelectionPhase, 500);
                }
            } else {
                const qArea = document.getElementById('question-area');
                qArea.classList.add('shake-error');
                setTimeout(() => qArea.classList.remove('shake-error'), 400);
            }
        }

        function enterSelectionPhase() {
            keyboardArea.style.display = 'none';
            selectionArea.style.display = 'flex';
            
            wordDisplayContainer.innerHTML = '<div class="char-text" style="font-size: 4rem;">???</div>';

            let options = [currentQuestion.fullWord];
            const allWords = questionBank.map(q => q.fullWord).filter(w => w !== currentQuestion.fullWord);
            
            allWords.sort(() => Math.random() - 0.5);
            options.push(allWords[0], allWords[1]);
            options.sort(() => Math.random() - 0.5);

            wordOptions.innerHTML = '';
            options.forEach(word => {
                if (word) {
                    const btn = document.createElement('button');
                    btn.className = 'word-btn';
                    btn.innerText = word;
                    btn.onclick = () => handleWordSelection(word);
                    wordOptions.appendChild(btn);
                }
            });
        }

        function handleWordSelection(selectedWord) {
            visualFeedback.style.display = 'flex';
            playZhuyinAudio(selectedWord); 

            if (selectedWord === currentQuestion.fullWord) {
                visualFeedback.innerHTML = '<span style="color:#4caf50;">✅</span>';
                setTimeout(() => {
                    visualFeedback.style.display = 'none';
                    loadNewQuestion(); 
                }, 1500);
            } else {
                visualFeedback.innerHTML = '<span style="color:#f44336;">❌</span>';
                setTimeout(() => {
                    visualFeedback.style.display = 'none';
                }, 1000);
            }
        }

        initKeyboard();
        loadNewQuestion();
    </script>
</body>
</html>
