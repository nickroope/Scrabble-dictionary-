<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    
    <!-- iOS / PWA Meta Tags -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#4f46e5">
    <meta name="application-name" content="Scrabble">
    
    <!-- Embedded Manifest with SVG Icon (Zero External Dependencies) -->
    <link rel="manifest" href='data:application/manifest+json;base64,eyJuYW1lIjoiU2NyYWJibGUgU3VpdGUiLCJzaG9ydF9uYW1lIjoiU2NyYWJibGUiLCJzdGFydF91cmwiOiIuIiwiZGlzcGxheSI6InN0YW5kYWxvbmUiLCJiYWNrZ3JvdW5kX2NvbG9yIjoiI2YzZjRmNiIsInRoZW1lX2NvbG9yIjoiIzRmNDZlNSIsImljb25zIjpbeyJzcmMiOiJkYXRhOmltYWdlL3N2Zyt4bWw7YmFzZTY0LFBITjJaZyw9Iiwic2l6ZXMiOiIxOTJ4MTkyIiwidHlwZSI6ImltYWdlL3N2Zyt4bWwifV19'>

    <title>Scrabble Suite</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        
        html, body {
            height: 100%;
            width: 100%;
            overflow: hidden; /* Prevent elastic scrolling of the whole page */
            overscroll-behavior: none;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }
        
        input { user-select: text; }

        .pt-safe { padding-top: env(safe-area-inset-top, 0px); }
        .pb-safe { padding-bottom: env(safe-area-inset-bottom, 0px); }

        .custom-scroll::-webkit-scrollbar { width: 4px; }
        .custom-scroll::-webkit-scrollbar-track { background: transparent; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 2px; }

        .loader {
            border: 2px solid #e0e7ff;
            border-top: 2px solid #4f46e5;
            border-radius: 50%;
            width: 14px;
            height: 14px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="flex flex-col bg-slate-100 text-slate-800">

    <!-- Super Compact Header -->
    <div class="bg-indigo-600 pt-safe px-3 pb-2 text-white shadow-md z-20 shrink-0">
        <!-- Row 1: Title & Controls -->
        <div class="flex justify-between items-center h-10">
            <div class="flex items-center gap-2">
                <i class="fa-solid fa-layer-group text-yellow-300 text-sm"></i>
                <h1 class="font-bold text-sm tracking-wide">Scrabble Suite</h1>
            </div>
            
            <div class="flex gap-2">
                <!-- Fullscreen Toggle (Workaround for local files) -->
                <button onclick="toggleFullScreen()" class="text-white/70 hover:text-white" title="Toggle Fullscreen">
                    <i id="fs-icon" class="fa-solid fa-expand text-xs"></i>
                </button>
                
                <div class="flex bg-indigo-800 rounded-md p-0.5 text-[10px] font-bold">
                    <button id="btn-uk" onclick="switchDict('UK')" class="px-2 py-1 rounded bg-white text-indigo-900 shadow-sm">UK</button>
                    <button id="btn-us" onclick="switchDict('US')" class="px-2 py-1 rounded text-indigo-200">US</button>
                </div>
            </div>
        </div>

        <!-- Row 2: Tabs (Very Slim) -->
        <div class="flex space-x-1 mt-1">
            <button onclick="switchTab('check')" id="tab-btn-check" class="flex-1 py-1.5 text-xs font-semibold rounded bg-white text-indigo-900 shadow-sm flex items-center justify-center gap-1">
                <i class="fa-solid fa-check"></i> Check
            </button>
            <button onclick="switchTab('list')" id="tab-btn-list" class="flex-1 py-1.5 text-xs font-semibold rounded text-indigo-100 hover:bg-indigo-500/50 flex items-center justify-center gap-1">
                <i class="fa-solid fa-list-ol"></i> 2-Letter
            </button>
            <button onclick="switchTab('find')" id="tab-btn-find" class="flex-1 py-1.5 text-xs font-semibold rounded text-indigo-100 hover:bg-indigo-500/50 flex items-center justify-center gap-1">
                <i class="fa-solid fa-search"></i> Find
            </button>
        </div>
    </div>

    <!-- Micro Status Bar -->
    <div id="status-bar" class="bg-white border-b border-slate-200 px-3 py-1 text-[10px] text-slate-500 flex justify-between items-center shrink-0 z-10 h-6">
        <span id="dict-name">Loading...</span>
        <div id="loading-indicator" class="flex items-center gap-2">
            <span id="offline-badge" class="hidden text-green-600 font-bold">OFFLINE</span>
            <div id="spinner" class="loader"></div>
        </div>
    </div>

    <!-- Main Content -->
    <div class="flex-1 relative overflow-hidden bg-slate-50">
        
        <!-- TAB 1: CHECKER -->
        <div id="tab-check" class="absolute inset-0 flex flex-col p-2 overflow-y-auto custom-scroll pb-safe">
            <div class="w-full max-w-md mx-auto flex flex-col gap-2 pt-2">
                
                <!-- Input Group -->
                <div class="relative group">
                    <input type="text" id="check-input" 
                        class="w-full bg-white border-2 border-slate-200 text-slate-800 text-2xl font-bold rounded-xl py-3 px-4 text-center uppercase tracking-widest focus:outline-none focus:border-indigo-500 focus:ring-4 focus:ring-indigo-100 transition-all placeholder-slate-300 shadow-sm"
                        placeholder="TYPE WORD" 
                        autocomplete="off" autocorrect="off" spellcheck="false"
                        oninput="handleCheckInput()"
                        onfocus="ensureVisibility()">
                    <button onclick="clearCheck()" id="check-clear-btn" class="absolute right-3 top-1/2 -translate-y-1/2 text-slate-300 hover:text-slate-500 hidden p-2">
                        <i class="fa-solid fa-circle-xmark text-lg"></i>
                    </button>
                </div>

                <!-- Result Card -->
                <div id="check-result" class="hidden flex-row items-center justify-between p-3 rounded-xl border-2 transition-all duration-300 animate-[pop_0.2s_ease-out]">
                    <div class="flex items-center gap-3">
                        <div id="check-icon" class="text-3xl"></div>
                        <div class="flex flex-col">
                            <h2 id="check-text" class="text-lg font-bold tracking-wide leading-tight"></h2>
                            <span class="text-[10px] opacity-70">in dictionary</span>
                        </div>
                    </div>
                    <div id="check-score-box" class="bg-white/60 px-3 py-1 rounded-lg text-sm font-bold border border-black/5 hidden shadow-sm">
                        <span id="check-score">0</span> pts
                    </div>
                </div>

                <!-- Empty State -->
                <div id="check-empty" class="text-center text-slate-400 mt-4 opacity-50">
                    <i class="fa-regular fa-keyboard text-2xl mb-1"></i>
                    <p class="text-xs">Type to check</p>
                    
                    <!-- Install Hint (Only shows if valid PWA) -->
                    <button id="install-btn" class="hidden mt-4 bg-indigo-100 text-indigo-700 px-3 py-1 rounded text-xs font-bold" onclick="installPWA()">
                        <i class="fa-solid fa-download mr-1"></i> Install App
                    </button>
                </div>
            </div>
        </div>

        <!-- TAB 2: 2-LETTER LIST -->
        <div id="tab-list" class="absolute inset-0 hidden flex-col bg-slate-50 overflow-hidden pb-safe">
            <div class="p-2 bg-white border-b border-slate-200 shadow-sm shrink-0">
                <div class="flex justify-between items-center text-[10px] text-slate-500 px-2">
                    <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-rose-200"></span> Consonants</span>
                    <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-yellow-200"></span> Vowels</span>
                    <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-blue-200"></span> Mixed</span>
                    <span>in <strong id="list-dict-label" class="text-slate-700">UK</strong></span>
                </div>
            </div>
            <div class="flex-1 overflow-y-auto p-2 custom-scroll">
                <!-- Changed from grid to flex-col for sections -->
                <div id="two-letter-container" class="flex flex-col pb-10">
                    <!-- Populated via JS -->
                </div>
            </div>
        </div>

        <!-- TAB 3: FINDER -->
        <div id="tab-find" class="absolute inset-0 hidden flex flex-col bg-slate-50 overflow-hidden pb-safe">
             <div class="p-2 bg-white border-b border-slate-200 shadow-sm shrink-0 z-10">
                <div class="flex gap-2">
                    <input type="text" id="find-input" 
                        class="flex-1 bg-slate-100 border border-slate-300 text-slate-800 text-base font-bold rounded py-2 px-3 uppercase tracking-widest focus:outline-none focus:border-indigo-500"
                        placeholder="RACK (use ?)" 
                        maxlength="15"
                        autocomplete="off">
                    <button onclick="findWords()" class="bg-indigo-600 text-white font-bold px-4 rounded shadow-sm text-sm">
                        GO
                    </button>
                </div>
            </div>

            <div class="flex-1 overflow-y-auto p-2 custom-scroll relative">
                <div id="find-results" class="flex flex-col gap-1 pb-20">
                    <div class="flex flex-col items-center justify-center h-20 text-slate-400">
                        <p class="text-xs">Enter letters to find top words</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- Fullscreen Toggle Logic ---
        function toggleFullScreen() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch((e) => {
                    console.log(e);
                });
                document.getElementById('fs-icon').className = "fa-solid fa-compress text-xs";
            } else {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                    document.getElementById('fs-icon').className = "fa-solid fa-expand text-xs";
                }
            }
        }

        // --- PWA Install Logic ---
        let deferredPrompt;
        const installBtn = document.getElementById('install-btn');

        window.addEventListener('beforeinstallprompt', (e) => {
            // Prevent Chrome 67 and earlier from automatically showing the prompt
            e.preventDefault();
            // Stash the event so it can be triggered later.
            deferredPrompt = e;
            // Update UI to notify the user they can add to home screen
            installBtn.classList.remove('hidden');
        });

        async function installPWA() {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                deferredPrompt = null;
                installBtn.classList.add('hidden');
            }
        }

        // --- Core Scrabble Logic ---
        const DICTIONARIES = {
            UK: {
                name: 'Collins (UK)',
                url: 'https://raw.githubusercontent.com/jesstess/Scrabble/master/scrabble/sowpods.txt',
                loaded: false,
                words: new Set(),
                list: [] 
            },
            US: {
                name: 'TWL06 (US)',
                url: 'https://raw.githubusercontent.com/zeisler/scrabble/master/db/dictionary.csv',
                loaded: false,
                words: new Set(),
                list: []
            }
        };

        const LETTER_SCORES = {
            'A': 1, 'B': 3, 'C': 3, 'D': 2, 'E': 1, 'F': 4, 'G': 2, 'H': 4, 'I': 1, 'J': 8, 'K': 5, 'L': 1, 'M': 3,
            'N': 1, 'O': 1, 'P': 3, 'Q': 10, 'R': 1, 'S': 1, 'T': 1, 'U': 1, 'V': 4, 'W': 4, 'X': 8, 'Y': 4, 'Z': 10,
            '?': 0
        };

        let currentDict = 'UK';
        let activeTab = 'check';
        const CACHE_NAME = 'scrabble-dict-v1';

        window.addEventListener('DOMContentLoaded', () => {
            loadDictionary('UK');
        });

        function ensureVisibility() {
            setTimeout(() => {
                document.getElementById('check-input').scrollIntoView({behavior: "smooth", block: "center"});
            }, 300);
        }

        function switchTab(tabName) {
            activeTab = tabName;
            ['check', 'list', 'find'].forEach(t => {
                const btn = document.getElementById(`tab-btn-${t}`);
                const div = document.getElementById(`tab-${t}`);
                if (t === tabName) {
                    btn.className = "flex-1 py-1.5 text-xs font-semibold rounded bg-white text-indigo-900 shadow-sm flex items-center justify-center gap-1";
                    div.classList.remove('hidden');
                    div.classList.add('flex');
                } else {
                    btn.className = "flex-1 py-1.5 text-xs font-semibold rounded text-indigo-100 hover:bg-indigo-500/50 flex items-center justify-center gap-1";
                    div.classList.add('hidden');
                    div.classList.remove('flex');
                }
            });
            if (tabName === 'list') renderTwoLetterList();
            if (tabName === 'check') document.getElementById('check-input').focus();
        }

        async function loadDictionary(type) {
            const statusLabel = document.getElementById('dict-name');
            const spinner = document.getElementById('spinner');
            const offlineBadge = document.getElementById('offline-badge');
            
            spinner.classList.remove('hidden');
            offlineBadge.classList.add('hidden');
            statusLabel.innerText = `Loading ${type}...`;
            
            // Toggle Button Styles
            const activeClass = "px-2 py-1 rounded bg-white text-indigo-900 shadow-sm";
            const inactiveClass = "px-2 py-1 rounded text-indigo-200 hover:text-white";
            document.getElementById('btn-uk').className = type === 'UK' ? activeClass : inactiveClass;
            document.getElementById('btn-us').className = type === 'US' ? activeClass : inactiveClass;

            currentDict = type;

            if (DICTIONARIES[type].loaded) {
                finishLoading();
                return;
            }

            try {
                let text = '';
                if ('caches' in window) {
                    const cache = await caches.open(CACHE_NAME);
                    const cachedResponse = await cache.match(DICTIONARIES[type].url);
                    if (cachedResponse) {
                        text = await cachedResponse.text();
                        offlineBadge.classList.remove('hidden'); 
                    }
                }

                if (!text) {
                    const response = await fetch(DICTIONARIES[type].url);
                    if (!response.ok) throw new Error("Network error");
                    text = await response.text();
                    if ('caches' in window) {
                        const cache = await caches.open(CACHE_NAME);
                        const responseToCache = new Response(text, { status: 200, headers: { 'Content-Type': 'text/plain' } });
                        await cache.put(DICTIONARIES[type].url, responseToCache);
                        offlineBadge.classList.remove('hidden');
                    }
                }
                
                const lines = text.split(/\r?\n/);
                DICTIONARIES[type].words = new Set();
                DICTIONARIES[type].list = [];

                lines.forEach(line => {
                    const word = line.trim().toUpperCase();
                    if (word && /^[A-Z]+$/.test(word)) {
                        DICTIONARIES[type].words.add(word);
                        DICTIONARIES[type].list.push(word);
                    }
                });

                DICTIONARIES[type].loaded = true;
                finishLoading();

            } catch (error) {
                console.error(error);
                statusLabel.innerText = "Failed";
                spinner.classList.add('hidden');
            }
        }

        function finishLoading() {
            document.getElementById('spinner').classList.add('hidden');
            document.getElementById('dict-name').innerText = DICTIONARIES[currentDict].name;
            document.getElementById('list-dict-label').innerText = currentDict;
            if (activeTab === 'check') handleCheckInput();
            if (activeTab === 'list') renderTwoLetterList();
        }

        function switchDict(type) {
            if (currentDict === type) return;
            loadDictionary(type);
        }

        function handleCheckInput() {
            const input = document.getElementById('check-input');
            input.value = input.value.toUpperCase().replace(/[^A-Z]/g, '');
            const word = input.value;
            const clearBtn = document.getElementById('check-clear-btn');

            if (word.length > 0) {
                clearBtn.classList.remove('hidden');
                document.getElementById('check-empty').classList.add('hidden');
                if (word.length >= 2) validateWord(word);
                else {
                    document.getElementById('check-result').classList.add('hidden');
                    document.getElementById('check-result').classList.remove('flex');
                }
            } else {
                clearCheck();
            }
        }

        function clearCheck() {
            document.getElementById('check-input').value = '';
            document.getElementById('check-clear-btn').classList.add('hidden');
            document.getElementById('check-result').classList.add('hidden');
            document.getElementById('check-result').classList.remove('flex');
            document.getElementById('check-empty').classList.remove('hidden');
            document.getElementById('check-input').focus();
        }

        function validateWord(word) {
            const isValid = DICTIONARIES[currentDict].words.has(word);
            const card = document.getElementById('check-result');
            const icon = document.getElementById('check-icon');
            const text = document.getElementById('check-text');
            const scoreBox = document.getElementById('check-score-box');
            
            card.classList.remove('hidden');
            card.classList.add('flex');

            if (isValid) {
                card.className = "flex flex-row items-center justify-between p-3 rounded-xl border-2 transition-all duration-300 bg-green-50 border-green-200 text-green-800 shadow-sm";
                icon.innerHTML = '<i class="fa-solid fa-circle-check text-green-500"></i>';
                text.innerText = "VALID";
                scoreBox.classList.remove('hidden');
                document.getElementById('check-score').innerText = getScore(word);
            } else {
                card.className = "flex flex-row items-center justify-between p-3 rounded-xl border-2 transition-all duration-300 bg-red-50 border-red-200 text-red-800 shadow-sm";
                icon.innerHTML = '<i class="fa-solid fa-circle-xmark text-red-500"></i>';
                text.innerText = "INVALID";
                scoreBox.classList.add('hidden');
            }
        }

        function renderTwoLetterList() {
            const container = document.getElementById('two-letter-container');
            if (!DICTIONARIES[currentDict].loaded) return;

            const allWords = DICTIONARIES[currentDict].list.filter(w => w.length === 2);
            const vowels = new Set(['A', 'E', 'I', 'O', 'U']);
            let htmlContent = '';

            // Loop A to Z
            for (let i = 65; i <= 90; i++) {
                const char = String.fromCharCode(i);
                const wordsStartingWithChar = allWords.filter(w => w.startsWith(char)).sort();
                
                htmlContent += `<div class="mb-3 pb-3 border-b border-slate-200 last:border-0">`;
                
                if (wordsStartingWithChar.length === 0) {
                     htmlContent += `<div class="text-[10px] text-slate-400 pl-1">No two letter words starting with ${char}</div>`;
                } else {
                    htmlContent += `<div class="grid grid-cols-5 gap-2 text-center font-bold text-slate-700 text-sm">`;
                    htmlContent += wordsStartingWithChar.map(w => {
                        let vCount = 0;
                        for (let c of w) {
                            if (vowels.has(c)) vCount++;
                        }
                        
                        let styleClass = "";
                        if (vCount === 2) {
                            // 2 Vowels - Yellow
                            styleClass = "bg-yellow-100 border-yellow-200 text-yellow-800";
                        } else if (vCount === 0) {
                            // 2 Consonants - Red (Rose)
                            styleClass = "bg-rose-100 border-rose-200 text-rose-800";
                        } else {
                            // Mixed (1 Vowel) - Blue
                            styleClass = "bg-blue-100 border-blue-200 text-blue-800";
                        }

                        return `<div class="${styleClass} border rounded-md py-1.5 shadow-sm">${w}</div>`;
                    }).join('');
                    htmlContent += `</div>`;
                }
                htmlContent += `</div>`;
            }

            container.innerHTML = htmlContent;
        }

        function getScore(word) {
            return word.split('').reduce((acc, char) => acc + (LETTER_SCORES[char] || 0), 0);
        }

        function findWords() {
            const input = document.getElementById('find-input');
            const resultsDiv = document.getElementById('find-results');
            let rawInput = input.value.toUpperCase().replace(/[^A-Z?]/g, '');
            input.value = rawInput;
            
            if (rawInput.length < 2) {
                resultsDiv.innerHTML = `<div class="text-center py-4 text-slate-400 text-xs">Enter 2+ letters</div>`;
                return;
            }

            if (!DICTIONARIES[currentDict].loaded) return;
            resultsDiv.innerHTML = '<div class="text-center py-4"><div class="loader mx-auto"></div></div>';

            setTimeout(() => {
                const foundWords = solveRack(rawInput, DICTIONARIES[currentDict].list);
                if (foundWords.length === 0) {
                    resultsDiv.innerHTML = `<div class="text-center py-4 text-slate-400 text-xs">No words found</div>`;
                } else {
                    const limitedResults = foundWords.slice(0, 100);
                    resultsDiv.innerHTML = limitedResults.map(item => `
                        <div class="bg-white px-3 py-2 rounded-lg border border-slate-200 flex justify-between items-center shadow-sm">
                            <span class="text-base font-bold text-slate-800 tracking-wider">${item.word}</span>
                            <div class="flex items-center gap-2">
                                <span class="text-[10px] text-slate-400 bg-slate-100 px-1.5 py-0.5 rounded">${item.word.length}L</span>
                                <div class="flex items-center gap-1 text-amber-600 font-bold text-sm"><span>${item.score}</span></div>
                            </div>
                        </div>
                    `).join('');
                }
            }, 50);
        }

        function solveRack(rack, dictionaryList) {
            const rackArr = rack.split('');
            const validWords = [];
            const getFreq = (arr) => { const map = {}; arr.forEach(c => map[c] = (map[c] || 0) + 1); return map; };
            const rackFreq = getFreq(rackArr);
            const blankCount = rackFreq['?'] || 0;

            for (let i = 0; i < dictionaryList.length; i++) {
                const word = dictionaryList[i];
                if (word.length > rackArr.length) continue;
                let tempBlanks = blankCount;
                let possible = true;
                const tempRack = {...rackFreq};
                for (let char of word) {
                    if (tempRack[char] && tempRack[char] > 0) tempRack[char]--;
                    else if (tempBlanks > 0) tempBlanks--;
                    else { possible = false; break; }
                }
                if (possible) validWords.push({ word: word, score: getScore(word) });
            }
            return validWords.sort((a, b) => {
                if (b.word.length !== a.word.length) return b.word.length - a.word.length;
                return b.score - a.score;
            });
        }
    </script>
</body>
</html>