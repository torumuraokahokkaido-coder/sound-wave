<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>音波ビジュアライザ＆かえるの歌ゲーム</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { margin: 0; overflow: hidden; background-color: #0f172a; color: white; font-family: sans-serif; }
        .canvas-container { cursor: grab; }
        .canvas-container:active { cursor: grabbing; }
        .neon-text { text-shadow: 0 0 10px rgba(56, 189, 248, 0.8), 0 0 20px rgba(56, 189, 248, 0.5); }
        .tab-active { background-color: #3b82f6; color: white; border-bottom: 2px solid #93c5fd; }
        .tab-inactive { background-color: #1e293b; color: #94a3b8; }
        .tab-inactive:hover { background-color: #334155; }
        
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #1e293b; }
        ::-webkit-scrollbar-thumb { background: #475569; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #64748b; }
    </style>
</head>
<body class="flex flex-col h-[100dvh] overflow-hidden">

    <!-- ヘッダー＆タブナビゲーション -->
    <header class="p-2 md:p-3 bg-slate-900 border-b border-slate-700 flex justify-between items-center z-20 shadow-lg shrink-0">
        <div class="flex items-center space-x-3">
            <span class="text-3xl">🐸</span>
            <div>
                <h1 class="text-xl md:text-2xl font-black text-blue-400 neon-text leading-none">SOUND LAB</h1>
                <p class="text-[10px] md:text-xs text-gray-400 mt-1">タイムトラベル対応オシロスコープ＆ピッチゲーム</p>
            </div>
        </div>
        
        <div class="flex space-x-2 items-center">
            <button id="タブ_シミュ" class="px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-active transition-colors">波形観察</button>
            <button id="タブ_ゲーム" class="px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-inactive transition-colors">かえるの歌ゲーム</button>
            <button id="ボタン_ヘルプ" class="ml-2 md:ml-4 px-3 py-2 bg-yellow-500 hover:bg-yellow-400 text-slate-900 rounded-full font-black shadow-[0_0_15px_rgba(234,179,8,0.5)] transition-transform hover:scale-110">？</button>
        </div>
    </header>

    <!-- メインコンテンツ -->
    <div class="flex-1 relative flex flex-col md:flex-row overflow-hidden">
        
        <!-- 左側：コントロールパネル -->
        <aside class="w-full md:w-72 lg:w-80 bg-slate-800 border-r border-slate-700 p-3 flex flex-col gap-3 overflow-y-auto z-10 shadow-xl shrink-0">
            
            <!-- マイク入力制御 -->
            <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 shadow-inner">
                <h2 class="text-xs font-bold text-gray-500 uppercase tracking-wider mb-2">Audio Input (音の入力)</h2>
                <button id="ボタン_マイク" class="w-full py-2.5 bg-red-600 hover:bg-red-500 rounded-lg text-white font-bold shadow-lg transition-all flex items-center justify-center gap-2 mb-2">
                    <svg class="w-5 h-5 animate-pulse" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"></path></svg>
                    マイクを接続する
                </button>
                <button id="ボタン_テスト音" class="w-full py-2 bg-emerald-600 hover:bg-emerald-500 rounded-lg text-white font-bold text-sm shadow-lg transition-all flex justify-center items-center gap-2">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"></path></svg>
                    テスト音(押している間鳴る)
                </button>
                <p class="text-[10px] text-gray-400 mt-2 text-center leading-tight">
                    ※iPadでマイクが使えない場合：<br><span class="text-yellow-400 font-bold">「設定」アプリ ＞「Safari」＞「マイク」を許可</span><br>に設定してページを再読み込みしてください。
                </p>
            </div>

            <!-- ドラッグ操作切り替え -->
            <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 shadow-inner">
                <h2 class="text-xs font-bold text-gray-500 uppercase tracking-wider mb-2">Drag Action (ドラッグ時の操作)</h2>
                <div class="grid grid-cols-2 gap-2">
                    <button id="ボタン_ドラッグ移動" class="py-2 px-3 rounded-lg text-sm font-bold bg-blue-600 text-white border-2 border-blue-400 shadow-lg">
                        ✥ 画面・時間移動
                    </button>
                    <button id="ボタン_ドラッグスケール" class="py-2 px-3 rounded-lg text-sm font-bold bg-slate-800 text-gray-400 hover:bg-slate-700 hover:text-white border-2 border-transparent">
                        🔍 スケール調整
                    </button>
                </div>
                <p class="text-[10px] text-gray-400 mt-2 leading-tight">
                    <b>💡 タイムトラベル機能：</b>一時停止中に「✥ 画面移動」で左右にドラッグすると、<b>過去3秒間</b>の波形を遡って観察できます！
                </p>
            </div>

            <!-- シミュレーション・コントロール -->
            <div id="パネル_シミュ" class="flex flex-col gap-3">
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 shadow-inner">
                    <h2 class="text-xs font-bold text-gray-500 uppercase tracking-wider mb-2">Oscilloscope Control</h2>
                    
                    <div class="mb-2">
                        <div class="flex justify-between text-sm mb-1">
                            <span class="text-gray-300">Y軸 (振幅スケール)</span>
                            <span id="表示_振幅" class="text-blue-400 font-mono">1.0x</span>
                        </div>
                        <input type="range" id="スライダー_振幅" min="0.1" max="5.0" step="0.1" value="1.0" class="w-full accent-blue-500">
                    </div>

                    <div class="mb-2">
                        <div class="flex justify-between text-sm mb-1">
                            <span class="text-gray-300">X軸 (表示時間スケール)</span>
                            <span id="表示_時間幅" class="text-blue-400 font-mono">1.0x</span>
                        </div>
                        <input type="range" id="スライダー_時間幅" min="0.1" max="5.0" step="0.1" value="1.0" class="w-full accent-blue-500">
                    </div>

                    <div class="flex gap-2 mt-2">
                        <button id="ボタン_一時停止" class="flex-1 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-bold transition-colors">一時停止</button>
                        <button id="ボタン_リセット" class="flex-1 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-bold transition-colors">リセット</button>
                    </div>
                </div>
            </div>

            <!-- ゲーム・コントロール -->
            <div id="パネル_ゲーム" class="flex flex-col gap-3 hidden">
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 shadow-inner text-center">
                    <h2 class="text-xs font-bold text-yellow-500 uppercase tracking-wider mb-2 text-left flex items-center gap-1">
                        <span>🐸</span> <span>かえるの合唱ミッション</span>
                    </h2>
                    
                    <label class="block text-left text-xs text-gray-400 mb-1">難易度</label>
                    <select id="セレクト_難易度" class="w-full p-2 bg-slate-800 text-white rounded-lg border border-slate-600 mb-2 text-sm focus:outline-none focus:border-blue-500">
                        <option value="1">🟢 初級 (音程が合わせやすい)</option>
                        <option value="2">🟡 中級 (標準テンポ)</option>
                        <option value="3">🔴 上級 (ハイスピード！)</option>
                    </select>

                    <button id="ボタン_ゲーム開始" class="w-full py-2.5 bg-blue-600 hover:bg-blue-500 rounded-lg text-white font-black shadow-[0_0_15px_rgba(37,99,235,0.4)] transition-all transform active:scale-95 mb-2">
                        🎵 演奏を始める！
                    </button>

                    <div class="grid grid-cols-2 gap-2 mt-1 border-t border-slate-700 pt-2">
                        <div>
                            <p class="text-[10px] text-gray-400 mb-1">SCORE</p>
                            <p id="表示_スコア" class="text-2xl font-black text-yellow-400">0</p>
                        </div>
                        <div>
                            <p class="text-[10px] text-gray-400 mb-1">TIME LEFT</p>
                            <p id="表示_残り時間" class="text-2xl font-black text-white">0s</p>
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- デジタルリアルタイム表示 -->
            <div class="bg-slate-900/80 p-3 rounded-xl border border-slate-700 shadow-inner flex flex-col gap-2">
                <h3 class="text-xs font-bold text-cyan-400 uppercase tracking-wider border-b border-slate-700 pb-1">📊 音響解析データ</h3>
                <div class="grid grid-cols-2 gap-2 text-xs">
                    <div>
                        <span class="text-gray-400">周波数 [Hz]:</span>
                        <div id="デジ_周波数" class="text-xl font-bold text-white font-mono">0 Hz</div>
                    </div>
                    <div>
                        <span class="text-gray-400">音圧 [dB]:</span>
                        <div id="デジ_デシベル" class="text-xl font-bold text-emerald-400 font-mono">-60.0 dB</div>
                    </div>
                    <div class="col-span-2 mt-1 border-t border-slate-800 pt-2 flex justify-between items-center">
                        <span class="text-gray-400">物理的な振幅変位:</span>
                        <span id="デジ_振幅" class="text-sm font-bold text-blue-300 font-mono">0.00</span>
                    </div>
                </div>
            </div>
            
        </aside>

        <!-- 右側：キャンバス領域 -->
        <div class="flex-1 relative bg-slate-950 canvas-container overflow-hidden" id="キャンバスコンテナ">
            <canvas id="メインキャンバス" class="absolute top-0 left-0 w-full h-full"></canvas>
            
            <!-- キャンバス上HUD表示 -->
            <div id="表示_ステータス" class="absolute top-4 left-4 text-xs font-mono text-gray-400 pointer-events-none bg-slate-900/70 p-3 rounded-lg border border-slate-700/50 backdrop-blur-sm space-y-1 z-10">
                <div>マイク: <span id="ステータス_マイク" class="text-red-500 font-bold">未接続</span></div>
                <div>スケール: <span id="ステータス_スケール" class="text-blue-400">X: 1.0x / Y: 1.0x</span></div>
                <div>ドラッグ操作: <span id="ステータス_ドラッグモード" class="text-yellow-400 font-bold">画面移動</span></div>
            </div>

            <!-- 超巨大周波数・音圧デジタル表示 -->
            <div class="absolute top-4 left-1/2 transform -translate-x-1/2 pointer-events-none flex flex-col items-center bg-slate-900/40 px-6 py-2 rounded-full border border-slate-700/30 backdrop-blur-sm z-10">
                <span class="text-[10px] text-gray-400 font-bold tracking-widest uppercase">REALTIME PITCH</span>
                <span id="超巨大_周波数" class="text-3xl md:text-5xl font-black text-cyan-400 font-mono neon-text">0 Hz</span>
            </div>
            
            <!-- ゲームオーバー表示 -->
            <div id="オーバーレイ_ゲーム結果" class="absolute inset-0 bg-slate-900/90 hidden flex flex-col items-center justify-center z-30 backdrop-blur-sm">
                <span class="text-6xl mb-2">🐸🎵</span>
                <h2 class="text-4xl font-black text-white mb-2">演奏終了！</h2>
                <p id="ゲーム評価メッセージ" class="text-xl text-yellow-400 font-bold mb-4">お見事！素晴らしい音感です！</p>
                <p class="text-sm text-gray-400 mb-2">あなたの最終演奏スコア</p>
                <p id="表示_最終スコア" class="text-6xl font-black text-yellow-400 neon-text mb-8">0</p>
                <button id="ボタン_リトライ" class="px-8 py-3 bg-blue-600 hover:bg-blue-500 rounded-full text-white font-bold shadow-lg transition-transform hover:scale-105">
                    もう一度演奏する
                </button>
            </div>
        </div>
    </div>

    <!-- 埋め込み環境でのマイクエラー対策UI -->
    <div id="モーダル_マイクエラー" class="fixed inset-0 bg-black/95 hidden items-center justify-center z-50 p-4 backdrop-blur-md overflow-y-auto">
        <div class="bg-slate-800 border-2 border-red-500 rounded-2xl p-6 max-w-lg w-full text-white shadow-2xl my-auto">
            <span class="text-5xl block text-center mb-2">⚠️</span>
            <h3 class="text-xl font-bold text-center text-red-400 mb-4">埋め込み環境のセキュリティ制限</h3>
            <p class="text-xs text-gray-300 leading-relaxed text-left bg-slate-900 p-3 rounded-lg border border-slate-700 mb-5">
                現在表示されているiframe環境ではブラウザの仕様によりマイクが起動できません。<br>以下のURLをコピーしてSafariやChromeのアドレスバーに直接貼り付けて開してください。
            </p>
            <div class="mb-5 text-left border-b border-slate-700 pb-4">
                <div class="flex gap-2">
                    <input type="text" id="入力_アプリURL" readonly class="flex-1 bg-slate-900 border border-slate-600 rounded px-2 py-1.5 text-xs text-gray-400 font-mono focus:outline-none" value="">
                    <button id="ボタン_コピー" class="bg-blue-600 hover:bg-blue-500 text-white font-bold px-4 py-1.5 rounded text-xs transition-colors whitespace-nowrap">
                        URLをコピー
                    </button>
                </div>
                <p id="表示_コピー完了" class="text-[10px] text-green-400 mt-1 hidden font-bold">✓ コピーしました！ブラウザで開いてください。</p>
            </div>
            <div class="flex flex-col sm:flex-row gap-3">
                <button id="ボタン_エラー閉じる" class="flex-1 py-2.5 bg-slate-700 hover:bg-slate-600 rounded-xl text-xs font-bold text-gray-300 transition-colors">
                    閉じる (テスト音で遊ぶ)
                </button>
            </div>
        </div>
    </div>

    <!-- 解説モーダル -->
    <div id="モーダル_解説" class="fixed inset-0 bg-black/80 hidden items-center justify-center z-50 p-4 backdrop-blur-sm transition-opacity">
        <div class="bg-slate-800 border-2 border-slate-600 rounded-2xl p-6 max-w-3xl w-full text-white shadow-2xl relative overflow-hidden">
            <div class="absolute -top-20 -right-20 w-64 h-64 bg-blue-500/10 rounded-full blur-3xl pointer-events-none"></div>
            <button id="ボタン_モーダル閉じる" class="absolute top-4 right-5 text-gray-400 hover:text-white text-3xl font-black transition-colors">&times;</button>
            <h2 class="text-2xl md:text-3xl font-black mb-4 text-blue-400 border-b border-slate-600 pb-3 flex items-center gap-3">
                <span class="bg-blue-500/20 p-2 rounded-lg">🎓</span> 物理法則：音波・デシベル・音高
            </h2>
            <div class="space-y-6 max-h-[70vh] overflow-y-auto pr-2 text-sm text-gray-300 leading-relaxed">
                <p><b>■ 振幅と音の大きさ (dB)</b><br>振幅は波の揺れの最大値です。アプリ内では取得した波形の実効値(RMS)を計算し、<code>dB = 20 × log₁₀(RMS)</code> の対数公式を使ってデシベルにリアルタイム変換しています。</p>
                <p><b>■ 振動数と音階</b><br>1秒間に波が振動する回数が周波数(Hz)です。かえるの合唱ゲームでは、「ド=約261Hz」「ラ=440Hz」などの実際の音階の物理周波数にターゲットを配置しています。</p>
                <p><b>■ タイムトラベル（波形履歴）機能</b><br>このオシロスコープは、裏側で常に「過去3秒間」の波形データを録音（リングバッファに記憶）し続けています。一時停止中に画面を左右にドラッグすると、過去に遡って一瞬の波の形を詳細に観察できます。</p>
            </div>
        </div>
    </div>

    <!-- メインロジック（Vanilla JS） -->
    <script>
        // ==========================================
        // 1. 変数と状態管理
        // ==========================================
        
        const キャンバス = document.getElementById('メインキャンバス');
        const コンテキスト = キャンバス.getContext('2d');
        const コンテナ = document.getElementById('キャンバスコンテナ');
        
        const UI = {
            タブ_シミュ: document.getElementById('タブ_シミュ'),
            タブ_ゲーム: document.getElementById('タブ_ゲーム'),
            パネル_シミュ: document.getElementById('パネル_シミュ'),
            パネル_ゲーム: document.getElementById('パネル_ゲーム'),
            ボタン_ヘルプ: document.getElementById('ボタン_ヘルプ'),
            モーダル_解説: document.getElementById('モーダル_解説'),
            ボタン_モーダル閉じる: document.getElementById('ボタン_モーダル閉じる'),
            ボタン_マイク: document.getElementById('ボタン_マイク'),
            ボタン_テスト音: document.getElementById('ボタン_テスト音'),
            ボタン_ドラッグ移動: document.getElementById('ボタン_ドラッグ移動'),
            ボタン_ドラッグスケール: document.getElementById('ボタン_ドラッグスケール'),
            スライダー_振幅: document.getElementById('スライダー_振幅'),
            スライダー_時間幅: document.getElementById('スライダー_時間幅'),
            表示_振幅: document.getElementById('表示_振幅'),
            表示_時間幅: document.getElementById('表示_時間幅'),
            ボタン_一時停止: document.getElementById('ボタン_一時停止'),
            ボタン_リセット: document.getElementById('ボタン_リセット'),
            セレクト_難易度: document.getElementById('セレクト_難易度'),
            ボタン_ゲーム開始: document.getElementById('ボタン_ゲーム開始'),
            表示_スコア: document.getElementById('表示_スコア'),
            表示_残り時間: document.getElementById('表示_残り時間'),
            オーバーレイ_ゲーム結果: document.getElementById('オーバーレイ_ゲーム結果'),
            表示_最終スコア: document.getElementById('表示_最終スコア'),
            ゲーム評価メッセージ: document.getElementById('ゲーム評価メッセージ'),
            ボタン_リトライ: document.getElementById('ボタン_リトライ'),
            ステータス_マイク: document.getElementById('ステータス_マイク'),
            ステータス_スケール: document.getElementById('ステータス_スケール'),
            ステータス_ドラッグモード: document.getElementById('ステータス_ドラッグモード'),
            超巨大_周波数: document.getElementById('超巨大_周波数'),
            デジ_周波数: document.getElementById('デジ_周波数'),
            デジ_デシベル: document.getElementById('デジ_デシベル'),
            デジ_振幅: document.getElementById('デジ_振幅'),
            モーダル_マイクエラー: document.getElementById('モーダル_マイクエラー'),
            入力_アプリURL: document.getElementById('入力_アプリURL'),
            ボタン_コピー: document.getElementById('ボタン_コピー'),
            表示_コピー完了: document.getElementById('表示_コピー完了'),
            ボタン_エラー閉じる: document.getElementById('ボタン_エラー閉じる'),
        };

        // --- 画面論理サイズ (高DPIリサイズ対応で自動算出・動的更新されます) ---
        let 画面幅 = 800;
        let 画面高さ = 600;

        // --- Web Audio API 関連 ---
        let オーディオコンテキスト = null;
        let アナライザー = null;
        let マイクストリーム = null;
        let 時間領域データ = null;
        let 周波数データ = null;
        let テスト用オシレータ = null;
        
        // --- 過去3秒間ドライブレコーダー（リングバッファ） ---
        const 過去記録秒数 = 3;
        let サンプリングレート = 44100;
        let 波形履歴バッファ = null;
        let バッファ書き込み位置 = 0;
        let プロセッサノード = null;
        let ミュートノード = null;

        // --- アプリケーション状態 ---
        const 状態 = {
            モード: 'シミュレーション',
            マイク接続済み: false,
            一時停止中: false,
            テスト音要求中: false,
            
            ドラッグモード: 'move',
            ドラッグ中: false,
            直前のマウス位置: { x: 0, y: 0 },
            
            カメラ: { x: 0, y: 0, スケール: 1.0 },
            
            // 履歴スクロール用
            一時停止時の書き込み位置: 0,
            履歴スクロールオフセット: 0, // 過去へ遡るサンプル数

            現在のピッチHz: 0,
            滑らかなピッチHz: 0, 
            現在のデシベル: -60.0,
            滑らかなデシベル: -60.0,
            現在の最大振幅: 0.0,
            波の音量: 0,
            
            ゲーム: {
                実行中: false, スコア: 0, 残り時間: 0,
                ターゲット一覧: [], パーティクル一覧: [],
                難易度: 1, 現在のメロディインデックス: 0, 最終更新時間: 0
            }
        };

        const かえるの合唱音階 = [
            { 音名: "ド", 周波数: 261.63 }, { 音名: "レ", 周波数: 293.66 }, { 音名: "ミ", 周波数: 329.63 }, { 音名: "ファ", 周波数: 349.23 },
            { 音名: "ミ", 周波数: 329.63 }, { 音名: "レ", 周波数: 293.66 }, { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 }, 
            { 音名: "ミ", 周波数: 329.63 }, { 音名: "ファ", 周波数: 349.23 }, { 音名: "ソ", 周波数: 392.00 }, { 音名: "ラ", 周波数: 440.00 },
            { 音名: "ソ", 周波数: 392.00 }, { 音名: "ファ", 周波数: 349.23 }, { 音名: "ミ", 周波数: 329.63 }, { 音名: "・", 周波数: 0 }, 
            { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 }, { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 },
            { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 }, { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 },
            { 音名: "ド", 周波数: 261.63 }, { 音名: "レ", 周波数: 293.66 }, { 音名: "ミ", 周波数: 329.63 }, { 音名: "ファ", 周波数: 349.23 },
            { 音名: "ミ", 周波数: 329.63 }, { 音名: "レ", 周波数: 293.66 }, { 音名: "ド", 周波数: 261.63 }, { 音名: "・", 周波数: 0 }
        ];

        // ==========================================
        // 2. 初期化とイベント設定 (DPI・画面回転追従対応)
        // ==========================================

        function キャンバスのリサイズ() {
            // 論理ピクセルでのサイズ（CSS上の大きさ）を算出
            画面幅 = コンテナ.clientWidth;
            画面高さ = コンテナ.clientHeight;

            // 高DPI(Retinaディスプレイ等)用のピクセル比率を取得して補正
            const ピクセル比 = window.devicePixelRatio || 1;

            // キャンバス自身の内部解像度をデバイスピクセル比に合わせて拡大
            キャンバス.width = 画面幅 * ピクセル比;
            キャンバス.height = 画面高さ * ピクセル比;

            // 表示する物理サイズは元の論理幅に指定してフィットさせる
            キャンバス.style.width = 画面幅 + "px";
            キャンバス.style.height = 画面高さ + "px";

            // コンテキストの描画基準比率を設定（以降、すべての描画を論理サイズベースで計算可能）
            コンテキスト.setTransform(1, 0, 0, 1, 0, 0);
            コンテキスト.scale(ピクセル比, ピクセル比);
        }

        window.addEventListener('resize', キャンバスのリサイズ);
        // iOS・iPadの回転処理をさらに滑らかにキャッチするための向き変更リスナー
        window.addEventListener('orientationchange', () => {
            setTimeout(キャンバスのリサイズ, 100);
        });
        キャンバスのリサイズ();

        function モードを切り替える(新モード) {
            状態.モード = 新モード;
            if (新モード === 'シミュレーション') {
                UI.タブ_シミュ.className = "px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-active transition-colors";
                UI.タブ_ゲーム.className = "px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-inactive transition-colors";
                UI.パネル_シミュ.classList.remove('hidden');
                UI.パネル_ゲーム.classList.add('hidden');
            } else {
                UI.タブ_シミュ.className = "px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-inactive transition-colors";
                UI.タブ_ゲーム.className = "px-3 md:px-5 py-2 rounded-t-lg font-bold text-sm md:text-base tab-active transition-colors";
                UI.パネル_シミュ.classList.add('hidden');
                UI.パネル_ゲーム.classList.remove('hidden');
            }
            視点をリセットする();
        }

        UI.タブ_シミュ.addEventListener('click', () => モードを切り替える('シミュレーション'));
        UI.タブ_ゲーム.addEventListener('click', () => モードを切り替える('ゲーム'));

        UI.ボタン_ヘルプ.addEventListener('click', () => UI.モーダル_解説.classList.remove('hidden'));
        UI.モーダル_解説.addEventListener('click', (e) => { if(e.target === UI.モーダル_解説) UI.モーダル_解説.classList.add('hidden'); });
        UI.ボタン_モーダル閉じる.addEventListener('click', () => UI.モーダル_解説.classList.add('hidden'));

        function ドラッグモードを切り替える(モード) {
            状態.ドラッグモード = モード;
            if (モード === 'move') {
                UI.ボタン_ドラッグ移動.className = "py-2 px-3 rounded-lg text-sm font-bold bg-blue-600 text-white border-2 border-blue-400 shadow-lg";
                UI.ボタン_ドラッグスケール.className = "py-2 px-3 rounded-lg text-sm font-bold bg-slate-800 text-gray-400 hover:bg-slate-700 hover:text-white border-2 border-transparent";
                UI.ステータス_ドラッグモード.innerText = "画面移動";
                UI.ステータス_ドラッグモード.className = "text-yellow-400 font-bold";
            } else {
                UI.ボタン_ドラッグ移動.className = "py-2 px-3 rounded-lg text-sm font-bold bg-slate-800 text-gray-400 hover:bg-slate-700 hover:text-white border-2 border-transparent";
                UI.ボタン_ドラッグスケール.className = "py-2 px-3 rounded-lg text-sm font-bold bg-purple-600 text-white border-2 border-purple-400 shadow-lg";
                UI.ステータス_ドラッグモード.innerText = "スケール調整";
                UI.ステータス_ドラッグモード.className = "text-purple-400 font-bold";
            }
        }
        UI.ボタン_ドラッグ移動.addEventListener('click', () => ドラッグモードを切り替える('move'));
        UI.ボタン_ドラッグスケール.addEventListener('click', () => ドラッグモードを切り替える('scale'));

        UI.スライダー_振幅.addEventListener('input', (e) => { UI.表示_振幅.innerText = parseFloat(e.target.value).toFixed(1) + 'x'; 状態更新_表示(); });
        UI.スライダー_時間幅.addEventListener('input', (e) => { UI.表示_時間幅.innerText = parseFloat(e.target.value).toFixed(1) + 'x'; 状態更新_表示(); });

        UI.ボタン_一時停止.addEventListener('click', () => {
            状態.一時停止中 = !状態.一時停止中;
            if (状態.一時停止中) {
                状態.一時停止時の書き込み位置 = バッファ書き込み位置;
                状態.履歴スクロールオフセット = 0;
                UI.ボタン_一時停止.innerText = "▶ 再開する";
                UI.ボタン_一時停止.className = "flex-1 py-2 bg-yellow-600 hover:bg-yellow-500 rounded-lg text-xs font-bold transition-colors text-white";
            } else {
                UI.ボタン_一時停止.innerText = "一時停止";
                UI.ボタン_一時停止.className = "flex-1 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-bold transition-colors";
            }
        });

        function 視点をリセットする() {
            状態.カメラ = { x: 0, y: 0, スケール: 1.0 };
            状態.履歴スクロールオフセット = 0;
            UI.スライダー_振幅.value = 1.0;
            UI.スライダー_時間幅.value = 1.0;
            UI.表示_振幅.innerText = "1.0x";
            UI.表示_時間幅.innerText = "1.0x";
            状態更新_表示();
        }
        UI.ボタン_リセット.addEventListener('click', 視点をリセットする);

        function 状態更新_表示() {
            UI.ステータス_スケール.innerText = `X: ${parseFloat(UI.スライダー_時間幅.value).toFixed(1)}x / Y: ${parseFloat(UI.スライダー_振幅.value).toFixed(1)}x`;
        }

        // ==========================================
        // 3. マウス＆タッチドラッグでの制御（移動・タイムトラベル・スケール）
        // ==========================================
        
        function ドラッグ開始(clientX, clientY) {
            状態.ドラッグ中 = true;
            状態.直前のマウス位置 = { x: clientX, y: clientY };
        }

        function ドラッグ中移動(clientX, clientY) {
            if (!状態.ドラッグ中) return;
            const デルタX = clientX - 状態.直前のマウス位置.x;
            const デルタY = clientY - 状態.直前のマウス位置.y;

            if (状態.ドラッグモード === 'move') {
                状態.カメラ.y += デルタY;
                
                if (状態.一時停止中 && 状態.モード === 'シミュレーション') {
                    // ★ タイムトラベル：左右ドラッグで時間履歴を遡る
                    const 時間幅スケール = parseFloat(UI.スライダー_時間幅.value);
                    const 基準表示サンプル数 = サンプリングレート * 0.15; 
                    const 表示サンプル数 = 基準表示サンプル数 / 時間幅スケール;
                    
                    const pxあたりのサンプル数 = (表示サンプル数 / 画面幅) * 2.0;
                    状態.履歴スクロールオフセット += デルタX * pxあたりのサンプル数;
                    
                    const 最大オフセット = (過去記録秒数 * サンプリングレート) - 表示サンプル数 - 100;
                    状態.履歴スクロールオフセット = Math.max(0, Math.min(最大オフセット, 状態.履歴スクロールオフセット));
                } else {
                    状態.カメラ.x += デルタX;
                }
            } else {
                // スケール調整
                let 新時間幅 = parseFloat(UI.スライダー_時間幅.value) + デルタX * 0.008;
                let 新振幅 = parseFloat(UI.スライダー_振幅.value) - デルタY * 0.008;
                新時間幅 = Math.max(0.1, Math.min(5.0, 新時間幅));
                新振幅 = Math.max(0.1, Math.min(5.0, 新振幅));
                UI.スライダー_時間幅.value = 新時間幅;
                UI.スライダー_振幅.value = 新振幅;
                UI.表示_時間幅.innerText = 新時間幅.toFixed(1) + 'x';
                UI.表示_振幅.innerText = 新振幅.toFixed(1) + 'x';
                状態更新_表示();
            }

            状態.直前のマウス位置 = { x: clientX, y: clientY };
        }

        function ドラッグ終了() { 状態.ドラッグ中 = false; }

        // キャンバスコンテナ内でのドラッグ位置を、論理座標（CSSピクセル）で正しく計算してハンドリングする
        function マウスタッチイベント取得(e, 関数) {
            let x, y;
            if (e.touches && e.touches.length > 0) {
                // 埋め込み環境でも正しく動くようにgetBoundingClientRectを考慮
                constrect = キャンバス.getBoundingClientRect();
                x = e.touches[0].clientX - rect.left;
                y = e.touches[0].clientY - rect.top;
            } else {
                const rect = キャンバス.getBoundingClientRect();
                x = e.clientX - rect.left;
                y = e.clientY - rect.top;
            }
            関数(x, y);
        }

        キャンバス.addEventListener('mousedown', (e) => マウスタッチイベント取得(e, ドラッグ開始));
        window.addEventListener('mousemove', (e) => {
            if (状態.ドラッグ中) {
                const rect = キャンバス.getBoundingClientRect();
                ドラッグ中移動(e.clientX - rect.left, e.clientY - rect.top);
            }
        });
        window.addEventListener('mouseup', ドラッグ終了);
        
        キャンバス.addEventListener('touchstart', (e) => { 
            if (e.touches.length === 1) マウスタッチイベント取得(e, ドラッグ開始); 
        });
        window.addEventListener('touchmove', (e) => { 
            if (状態.ドラッグ中 && e.touches.length === 1) {
                const rect = キャンバス.getBoundingClientRect();
                ドラッグ中移動(e.touches[0].clientX - rect.left, e.touches[0].clientY - rect.top);
            }
        });
        window.addEventListener('touchend', ドラッグ終了);

        キャンバス.addEventListener('wheel', (e) => {
            e.preventDefault();
            const ズーム量 = e.deltaY * -0.001;
            状態.カメラ.スケール = Math.max(0.2, Math.min(5.0, 状態.カメラ.スケール + ズーム量));
        }, { passive: false });

        // ==========================================
        // 4. Web Audio API ＆ リングバッファ（ドライブレコーダー）
        // ==========================================

        let オーディオ初期化完了 = false;

        async function オーディオ初期化() {
            if (オーディオ初期化完了) return; 

            if (!オーディオコンテキスト) {
                window.AudioContext = window.AudioContext || window.webkitAudioContext;
                オーディオコンテキスト = new AudioContext();
            }
            
            if (オーディオコンテキスト.state === 'suspended') {
                 await オーディオコンテキスト.resume();
            }

            if (!アナライザー) {
                サンプリングレート = オーディオコンテキスト.sampleRate;
                
                アナライザー = オーディオコンテキスト.createAnalyser();
                アナライザー.fftSize = 2048; 
                時間領域データ = new Uint8Array(アナライザー.frequencyBinCount);
                周波数データ = new Uint8Array(アナライザー.frequencyBinCount);

                const バッファ長 = サンプリングレート * 過去記録秒数;
                波形履歴バッファ = new Float32Array(バッファ長);
                バッファ書き込み位置 = 0;

                プロセッサノード = オーディオコンテキスト.createScriptProcessor(2048, 1, 1);
                プロセッサノード.onaudioprocess = (e) => {
                    if (状態.一時停止中) return; 
                    const 入力 = e.inputBuffer.getChannelData(0);
                    for (let i = 0; i < 入力.length; i++) {
                        波形履歴バッファ[バッファ書き込み位置] = 入力[i];
                        バッファ書き込み位置 = (バッファ書き込み位置 + 1) % バッファ長;
                    }
                };

                ミュートノード = オーディオコンテキスト.createGain();
                ミュートノード.gain.value = 0;
                プロセッサノード.connect(ミュートノード);
                ミュートノード.connect(オーディオコンテキスト.destination);
            }
            オーディオ初期化完了 = true;
        }

        function アプリURLの初期設定() {
            let 現在のURL = window.location.href;
            if (現在のURL.startsWith('blob:') || 現在のURL.includes('usercontent')) {
                UI.入力_アプリURL.value = "https://sites.google.com/"; 
            } else {
                UI.入力_アプリURL.value = 現在のURL;
            }
        }
        window.addEventListener('load', アプリURLの初期設定);

        UI.ボタン_マイク.addEventListener('click', async () => {
            try {
                window.AudioContext = window.AudioContext || window.webkitAudioContext;
                if (!オーディオコンテキスト) {
                    オーディオコンテキスト = new AudioContext();
                }
                if (オーディオコンテキスト.state === 'suspended') {
                    await オーディオコンテキスト.resume(); 
                }

                マイクストリーム = await navigator.mediaDevices.getUserMedia({ 
                    audio: { echoCancellation: true, noiseSuppression: true } 
                });

                await オーディオ初期化();
                
                if (プロセッサノード && アナライザー) {
                    const マイクソース = オーディオコンテキスト.createMediaStreamSource(マイクストリーム);
                    マイクソース.connect(アナライザー); 
                    マイクソース.connect(プロセッサノード); 
                    
                    状態.マイク接続済み = true;
                    UI.ボタン_マイク.classList.replace('bg-red-600', 'bg-slate-700');
                    UI.ボタン_マイク.classList.replace('hover:bg-red-500', 'hover:bg-slate-600');
                    UI.ボタン_マイク.innerHTML = '🎤 マイク接続中';
                    UI.ステータス_マイク.innerText = "接続済み";
                    UI.ステータス_マイク.className = "text-green-400";
                } else {
                    throw new Error("オーディオノードの初期化に失敗しました。");
                }
            } catch (エラー) {
                console.warn("マイク取得例外を検知:", エラー);
                UI.入力_アプリURL.value = window.location.href;
                UI.モーダル_マイクエラー.classList.remove('hidden'); 
            }
        });

        UI.ボタン_コピー.addEventListener('click', () => {
            const 入力要素 = UI.入力_アプリURL;
            入力要素.select();
            入力要素.setSelectionRange(0, 99999); 
            try {
                if (document.execCommand('copy')) {
                    UI.表示_コピー完了.classList.remove('hidden');
                    setTimeout(() => UI.表示_コピー完了.classList.add('hidden'), 5000);
                } else {
                    alert("自動コピーが制限されています。直接長押ししてコピーしてください！");
                }
            } catch (エラー) { alert("URLを直接選択してコピーしてください。"); }
        });

        UI.ボタン_エラー閉じる.addEventListener('click', () => UI.モーダル_マイクエラー.classList.add('hidden'));

        function テスト音開始() {
            if(状態.マイク接続済み) return; 
            状態.テスト音要求中 = true;
            window.AudioContext = window.AudioContext || window.webkitAudioContext;
            if (!オーディオコンテキスト) オーディオコンテキスト = new AudioContext();
            
            const 初期化プロセス = async () => {
                if (オーディオコンテキスト.state === 'suspended') await オーディオコンテキスト.resume();
                await オーディオ初期化();

                if (!状態.テスト音要求中) return;
                if (テスト用オシレータ) return; 
                
                if (!プロセッサノード || !アナライザー) return;

                テスト用オシレータ = オーディオコンテキスト.createOscillator();
                テスト用オシレータ.type = 'sine';
                テスト用オシレータ.frequency.value = 329.63; 
                
                キャンバス.addEventListener('mousemove', オシレータ音程変更);
                
                テスト用オシレータ.connect(アナライザー);
                テスト用オシレータ.connect(プロセッサノード); 
                アナライザー.connect(オーディオコンテキスト.destination); 
                
                try { テスト用オシレータ.start(); } catch(e) {}
                
                状態.マイク接続済み = true;
                UI.ステータス_マイク.innerText = "テスト発振中";
                UI.ステータス_マイク.className = "text-emerald-400";
            };
            
            初期化プロセス();
        }

        function テスト音停止() {
            状態.テスト音要求中 = false;
            if (テスト用オシレータ) {
                try { テスト用オシレータ.stop(); } catch(e) {}
                テスト用オシレータ.disconnect();
                テスト用オシレータ = null;
                キャンバス.removeEventListener('mousemove', オシレータ音程変更);
                状態.マイク接続済み = false;
                UI.ステータス_マイク.innerText = "未接続";
                UI.ステータス_マイク.className = "text-red-500";
            }
        }

        function オシレータ音程変更(e) {
            if(テスト用オシレータ) {
                // 論理座標のキャンバスの高さを使用
                const 割合 = 1.0 - (e.clientY / 画面高さ);
                const 新周波数 = 100 + (割合 * 800);
                テスト用オシレータ.frequency.setTargetAtTime(新周波数, オーディオコンテキスト.currentTime, 0.05);
            }
        }

        UI.ボタン_テスト音.addEventListener('mousedown', テスト音開始);
        UI.ボタン_テスト音.addEventListener('mouseup', テスト音停止);
        UI.ボタン_テスト音.addEventListener('mouseleave', テスト音停止);
        UI.ボタン_テスト音.addEventListener('touchstart', (e) => { e.preventDefault(); テスト音開始(); });
        UI.ボタン_テスト音.addEventListener('touchend', テスト音停止);

        // ==========================================
        // 5. 高精度シグナル解析（リアルタイム解析用）
        // ==========================================

        function データを解析する() {
            if (!アナライザー || 状態.一時停止中) return; 

            アナライザー.getByteTimeDomainData(時間領域データ);
            アナライザー.getByteFrequencyData(周波数データ);

            // ① デシベル計算
            let 平方和 = 0;
            let 最大変位 = 0;
            for (let i = 0; i < 時間領域データ.length; i++) {
                const 基準値 = (時間領域データ[i] - 128) / 128.0;
                平方和 += 基準値 * 基準値;
                if (Math.abs(基準値) > 最大変位) 最大変位 = Math.abs(基準値);
            }
            const 実効値RMS = Math.sqrt(平方和 / 時間領域データ.length);
            状態.現在の最大振幅 = 最大変位;

            let 計算デシベル = -60.0;
            if (実効値RMS > 0.001) 計算デシベル = 20 * Math.log10(実効値RMS) + 20;
            計算デシベル = Math.max(-60, Math.min(0, 計算デシベル));
            状態.現在のデシベル = 計算デシベル;
            状態.滑らかなデシベル += (状態.現在のデシベル - 状態.滑らかなデシベル) * 0.2;

            // ② ピッチ検出
            const 周波数分解能 = オーディオコンテキスト.sampleRate / アナライザー.fftSize;
            let 最大音量 = 0;
            let ピークインデックス = 0;
            
            const 探索上限インデックス = Math.floor(1500 / 周波数分解能);
            const 探索下限インデックス = Math.floor(80 / 周波数分解能);

            for (let i = 探索下限インデックス; i < 探索上限インデックス; i++) {
                if (周波数データ[i] > 最大音量) {
                    最大音量 = 周波数データ[i];
                    ピークインデックス = i;
                }
            }

            状態.波の音量 = 最大音量;
            if (最大音量 > 90 && 状態.現在のデシベル > -45) {
                状態.現在のピッチHz = ピークインデックス * 周波数分解能;
            } else {
                状態.現在のピッチHz = 0;
            }

            if (状態.現在のピッチHz > 0) {
                状態.滑らかなピッチHz += (状態.現在のピッチHz - 状態.滑らかなピッチHz) * 0.25;
            } else {
                状態.滑らかなピッチHz += (0 - 状態.滑らかなピッチHz) * 0.2;
            }

            // ③ デジタルHUD更新
            const 表示Hz = Math.round(状態.滑らかなピッチHz);
            UI.超巨大_周波数.innerText = 表示Hz > 20 ? 表示Hz + " Hz" : "--- Hz";
            UI.デジ_周波数.innerText = 表示Hz > 20 ? 表示Hz + " Hz" : "0 Hz";
            UI.デジ_デシベル.innerText = 状態.滑らかなデシベル.toFixed(1) + " dB";
            UI.デジ_振幅.innerText = 状態.現在の最大振幅.toFixed(2);
        }

        // ==========================================
        // 6. 描画処理：タイムトラベル・シミュレーション
        // ==========================================

        function 履歴からゼロクロスを探す(開始インデックス, 探索範囲) {
            const バッファ長 = 波形履歴バッファ.length;
            for (let i = 0; i < 探索範囲; i++) {
                const idx1 = (開始インデックス + i + バッファ長) % バッファ長;
                const idx2 = (開始インデックス + i + 1 + バッファ長) % バッファ長;
                const v1 = 波形履歴バッファ[idx1];
                const v2 = 波形履歴バッファ[idx2];
                if (v1 <= 0 && v2 > 0 && (v2 - v1) > 0.01) {
                    return i; 
                }
            }
            return 0;
        }

        function シミュレーションを描画する() {
            // 背景クリア
            コンテキスト.fillStyle = 'rgba(15, 23, 42, 1.0)';
            コンテキスト.fillRect(0, 0, 画面幅, 画面高さ);

            if (!状態.マイク接続済み || !波形履歴バッファ || 波形履歴バッファ.length === 0) {
                コンテキスト.fillStyle = '#64748b';
                コンテキスト.font = 'bold 16px sans-serif';
                コンテキスト.textAlign = 'center';
                コンテキスト.fillText('マイクを接続するか、テスト音を鳴らしてください', 画面幅 / 2, 画面高さ / 2 - 20);
                
                コンテキスト.fillStyle = '#38bdf8';
                コンテキスト.font = '12px sans-serif';
                コンテキスト.fillText('ドラッグでX軸/Y軸スケールのリアルタイム変更が可能です！', 画面幅 / 2, 画面高さ / 2 + 10);
                
                コンテキスト.fillStyle = '#facc15'; 
                コンテキスト.fillText('⚠️ iPadでマイクが反応しない場合：「設定」アプリ ＞「Safari」＞「マイク」を「許可」にしてください', 画面幅 / 2, 画面高さ / 2 + 40);
                return;
            }

            const 振幅スケール = parseFloat(UI.スライダー_振幅.value);
            const 時間幅スケール = parseFloat(UI.スライダー_時間幅.value);

            コンテキスト.save();
            // 常時画面幅の中央・高さ中央を基準に変形する
            コンテキスト.translate(画面幅 / 2 + 状態.カメラ.x, 画面高さ / 2 + 状態.カメラ.y);
            コンテキスト.scale(状態.カメラ.スケール, 状態.カメラ.スケール);

            // グリッド描画
            コンテキスト.strokeStyle = 'rgba(56, 189, 248, 0.1)';
            コンテキスト.lineWidth = 1 / 状態.カメラ.スケール;
            コンテキスト.beginPath();
            const グリッド幅 = 60;
            const 表示境界 = 3000; // iPad横画面でも十分カバーできるように拡大
            for (let i = -表示境界; i <= 表示境界; i += グリッド幅) {
                コンテキスト.moveTo(i, -表示境界); コンテキスト.lineTo(i, 表示境界);
                コンテキスト.moveTo(-表示境界, i); コンテキスト.lineTo(表示境界, i);
            }
            コンテキスト.stroke();

            // 中心軸
            コンテキスト.strokeStyle = 'rgba(56, 189, 248, 0.35)';
            コンテキスト.lineWidth = 2 / 状態.カメラ.スケール;
            コンテキスト.beginPath();
            コンテキスト.moveTo(-表示境界, 0); コンテキスト.lineTo(表示境界, 0);
            コンテキスト.moveTo(0, -表示境界); コンテキスト.lineTo(0, 表示境界);
            コンテキスト.stroke();

            // 振幅の限界表示線（Y軸スケールに合わせて伸縮）
            コンテキスト.strokeStyle = 'rgba(239, 68, 68, 0.2)'; 
            コンテキスト.setLineDash([10, 10]);
            コンテキスト.beginPath();
            const 音圧限界 = 150 * 振幅スケール;
            コンテキスト.moveTo(-表示境界, 音圧限界); コンテキスト.lineTo(表示境界, 音圧限界);
            コンテキスト.moveTo(-表示境界, -音圧限界); コンテキスト.lineTo(表示境界, -音圧限界);
            コンテキスト.stroke();
            コンテキスト.setLineDash([]); 

            // ★ 波形履歴の描画計算 (画面幅 画面幅 に100%追従して波が自動フィットします)
            const 基準表示サンプル数 = サンプリングレート * 0.15;
            const 表示サンプル数 = Math.floor(基準表示サンプル数 / 時間幅スケール);
            const バッファ長 = 波形履歴バッファ.length;

            let 読み取り基準位置 = 0;
            if (状態.一時停止中) {
                読み取り基準位置 = Math.floor(状態.一時停止時の書き込み位置 - 表示サンプル数 - 状態.履歴スクロールオフセット);
            } else {
                読み取り基準位置 = Math.floor(バッファ書き込み位置 - 表示サンプル数);
            }

            読み取り基準位置 = ((読み取り基準位置 % バッファ長) + バッファ長) % バッファ長;

            let ゼロクロス補正 = 0;
            if (!状態.一時停止中) {
                ゼロクロス補正 = 履歴からゼロクロスを探す(読み取り基準位置, Math.floor(表示サンプル数 / 4));
            }

            const 実際の開始位置 = (読み取り基準位置 + ゼロクロス補正) % バッファ長;
            const 実際の描画サンプル数 = 表示サンプル数 - ゼロクロス補正;
            
            // どのような画面の横幅でも画面端から端まで均等に波が広がるようにX刻みを調整
            const X増分 = 画面幅 / 実際の描画サンプル数;

            コンテキスト.lineWidth = 3 / 状態.カメラ.スケール;
            コンテキスト.strokeStyle = '#38bdf8'; 
            コンテキスト.shadowBlur = 12;
            コンテキスト.shadowColor = '#38bdf8';
            
            コンテキスト.beginPath();
            for (let i = 0; i < 実際の描画サンプル数; i++) {
                const idx = (実際の開始位置 + i) % バッファ長;
                const 正規化振幅 = 波形履歴バッファ[idx]; 
                
                const x = (i * X増分) - (画面幅 / 2);
                const y = 正規化振幅 * -150 * 振幅スケール;

                if (i === 0) コンテキスト.moveTo(x, y);
                else コンテキスト.lineTo(x, y);
            }
            コンテキスト.stroke();
            コンテキスト.restore(); 

            // HUDとメーターの描画
            ダブルメーターを描画する();

            // タイムトラベル（過去遡り）インジケーター
            if (状態.一時停止中) {
                const 遡り秒数 = 状態.履歴スクロールオフセット / サンプリングレート;
                コンテキスト.fillStyle = 'rgba(250, 204, 21, 0.9)'; 
                コンテキスト.font = 'bold 16px monospace';
                コンテキスト.textAlign = 'center';
                コンテキスト.shadowBlur = 10;
                コンテキスト.shadowColor = '#000';
                
                if (遡り秒数 > 0.01) {
                    コンテキスト.fillText(`◀◀ 過去 ${遡り秒数.toFixed(2)} 秒の波形を観察中`, 画面幅 / 2, 画面高さ - 30);
                } else {
                    コンテキスト.fillText(`⏸ 一時停止中 (左右ドラッグで最大3秒前へタイムトラベル)`, 画面幅 / 2, 画面高さ - 30);
                }
                コンテキスト.shadowBlur = 0;
            }
        }

        // --- アナログダブルメーター描画 (画面の幅と位置に自動フィット) ---
        function ダブルメーターを描画する() {
            const メーター幅 = 130;
            const メーター高さ = 75;
            const Y開始 = 15;
            
            // 常に右上にきれいに整列するように画面幅(論理ピクセル)を基準に配置
            const メーター2_X = 画面幅 - メーター幅 - 15; 
            const メーター1_X = メーター2_X - メーター幅 - 15;      

            メーター基礎を描画("PITCH (Hz)", メーター1_X, Y開始, メーター幅, メーター高さ);
            const Hz割合 = Math.max(0, Math.min(1.0, (状態.滑らかなピッチHz - 100) / 900));
            針を描画(メーター1_X + メーター幅/2, Y開始 + メーター高さ - 5, メーター高さ - 15, Hz割合);

            メーター基礎を描画("LEVEL (dB)", メーター2_X, Y開始, メーター幅, メーター高さ);
            const dB割合 = Math.max(0, Math.min(1.0, (状態.滑らかなデシベル + 60) / 60));
            針を描画(メーター2_X + メーター幅/2, Y開始 + メーター高さ - 5, メーター高さ - 15, dB割合);
        }

        function メーター基礎を描画(タイトル, x, y, w, h) {
            コンテキスト.save();
            コンテキスト.fillStyle = 'rgba(30, 41, 59, 0.85)'; 
            コンテキスト.strokeStyle = '#475569';
            コンテキスト.lineWidth = 2;
            コンテキスト.beginPath();
            コンテキスト.roundRect(x, y, w, h, 10);
            コンテキスト.fill();
            コンテキスト.stroke();

            const 中軸X = x + w / 2;
            const 下軸Y = y + h - 5;
            const 半径 = h - 20;

            コンテキスト.strokeStyle = '#64748b';
            コンテキスト.lineWidth = 3;
            コンテキスト.beginPath();
            コンテキスト.arc(中軸X, 下軸Y, 半径, Math.PI, 0);
            コンテキスト.stroke();

            コンテキスト.strokeStyle = '#22c55e'; 
            コンテキスト.lineWidth = 4;
            コンテキスト.beginPath();
            コンテキスト.arc(中軸X, 下軸Y, 半径, Math.PI, Math.PI * 1.4);
            コンテキスト.stroke();

            コンテキスト.strokeStyle = '#eab308'; 
            コンテキスト.beginPath();
            コンテキスト.arc(中軸X, 下軸Y, 半径, Math.PI * 1.4, Math.PI * 1.8);
            コンテキスト.stroke();

            コンテキスト.strokeStyle = '#ef4444';
            コンテキスト.beginPath();
            コンテキスト.arc(中軸X, 下軸Y, 半径, Math.PI * 1.8, Math.PI * 2.0);
            コンテキスト.stroke();

            コンテキスト.fillStyle = '#94a3b8';
            コンテキスト.font = 'bold 9px sans-serif';
            コンテキスト.textAlign = 'center';
            コンテキスト.fillText(タイトル, 中軸X, y + 15);
            コンテキスト.restore();
        }

        function 針を描画(中心X, 中心Y, 長さ, 割合) {
            const 角度 = Math.PI + 割合 * Math.PI;
            コンテキスト.save();
            コンテキスト.strokeStyle = '#f43f5e'; 
            コンテキスト.lineWidth = 2;
            コンテキスト.shadowBlur = 5;
            コンテキスト.shadowColor = '#f43f5e';
            
            コンテキスト.beginPath();
            コンテキスト.moveTo(中心X, 中心Y);
            コンテキスト.lineTo(中心X + Math.cos(角度) * 長さ, 中心Y + Math.sin(角度) * 長さ);
            コンテキスト.stroke();

            コンテキスト.beginPath();
            コンテキスト.arc(中心X, 中心Y, 5, 0, Math.PI * 2);
            コンテキスト.fillStyle = '#cbd5e1';
            コンテキスト.shadowBlur = 0;
            コンテキスト.fill();
            コンテキスト.restore();
        }

        // ==========================================
        // 7. ゲームロジック
        // ==========================================

        class 音波ターゲット {
            constructor(メロディ定義, インデックス, 難易度) {
                this.x = 画面幅 + 100; // 常に最新の画面幅の外側から登場
                this.幅 = 75;
                this.音名 = メロディ定義.音名;
                this.目標周波数 = メロディ定義.周波数;
                this.通過済み = false;
                this.休符 = メロディ定義.周波数 === 0;

                // 画面解像度(画面幅)の違いによるゲーム進行速度の不公平感をなくす補正
                const 速度比率 = 画面幅 / 1000;
                if (難易度 == 1) { this.許容範囲 = 100; this.速度 = 2.5 * 速度比率; } 
                else if (難易度 == 2) { this.許容範囲 = 60; this.速度 = 4.0 * 速度比率; } 
                else { this.許容範囲 = 30; this.速度 = 6.5 * 速度比率; }
            }
            更新() { this.x -= this.速度; }
            描画(ctx) {
                if (this.休符) return; 
                const y = 周波数をY座標に変換(this.目標周波数);
                const 半値幅 = Math.abs(周波数をY座標に変換(this.目標周波数) - 周波数をY座標に変換(this.目標周波数 + this.許容範囲));
                
                ctx.save();
                if (this.通過済み) {
                    ctx.fillStyle = 'rgba(71, 85, 105, 0.15)'; ctx.strokeStyle = '#475569'; ctx.shadowBlur = 0;
                } else {
                    ctx.fillStyle = 'rgba(34, 197, 94, 0.1)'; ctx.strokeStyle = '#22c55e'; ctx.shadowBlur = 10; ctx.shadowColor = '#22c55e';
                }
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.roundRect(this.x, y - 半値幅, this.幅, 半値幅 * 2, 10);
                ctx.fill(); ctx.stroke();

                if (!this.通過済み) {
                    ctx.strokeStyle = 'rgba(34, 197, 94, 0.5)'; ctx.setLineDash([4, 4]);
                    ctx.beginPath(); ctx.moveTo(this.x, y); ctx.lineTo(this.x + this.幅, y); ctx.stroke();
                }
                ctx.restore();
                ctx.fillStyle = this.通過済み ? '#94a3b8' : '#ffffff';
                ctx.font = 'bold 20px sans-serif'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
                ctx.fillText(this.音名, this.x + this.幅 / 2, y);
                ctx.font = '10px monospace'; ctx.fillStyle = this.通過済み ? '#64748b' : '#a7f3d0';
                ctx.fillText(Math.round(this.目標周波数) + "Hz", this.x + this.幅 / 2, y + 18);
            }
        }

        class 歌唱パーティクル {
            constructor(x, y, 色) {
                this.x = x; this.y = y; this.色 = 色;
                const 角度 = Math.random() * Math.PI * 2; const 速度 = Math.random() * 6 + 3;
                this.vx = Math.cos(角度) * 速度; this.vy = Math.sin(角度) * 速度;
                this.寿命 = 1.0; this.減衰 = Math.random() * 0.04 + 0.02;
            }
            更新() { this.x += this.vx; this.y += this.vy; this.寿命 -= this.減衰; }
            描画(ctx) {
                ctx.save(); ctx.globalAlpha = Math.max(0, this.寿命); ctx.fillStyle = this.色;
                ctx.beginPath(); ctx.arc(this.x, this.y, 5, 0, Math.PI * 2); ctx.fill(); ctx.restore();
            }
        }

        function 周波数をY座標に変換(hz) {
            const 最小Hz = 100; const 最大Hz = 1000;
            const クランプ = Math.max(最小Hz, Math.min(最大Hz, hz));
            // 常に変動する論理画面高さを元にマッピング
            return 画面高さ - (((クランプ - 最小Hz) / (最大Hz - 最小Hz)) * (画面高さ - 130) + 65);
        }

        function ゲームを開始する() {
            状態.ゲーム = { 実行中: true, スコア: 0, 残り時間: 40, ターゲット一覧: [], パーティクル一覧: [], 難易度: parseInt(UI.セレクト_難易度.value), 現在のメロディインデックス: 0, 最終更新時間: performance.now() };
            UI.表示_スコア.innerText = "0"; UI.表示_残り時間.innerText = "40s"; UI.オーバーレイ_ゲーム結果.classList.add('hidden');
            メロディターゲットを投下する();
        }

        function メロディターゲットを投下する() {
            const idx = 状態.ゲーム.現在のメロディインデックス;
            if (idx < かえるの合唱音階.length) {
                状態.ゲーム.ターゲット一覧.push(new 音波ターゲット(かえるの合唱音階[idx], idx, 状態.ゲーム.難易度));
                状態.ゲーム.現在のメロディインデックス++;
            }
        }

        UI.ボタン_ゲーム開始.addEventListener('click', ゲームを開始する);
        UI.ボタン_リトライ.addEventListener('click', ゲームを開始する);

        function ゲームロジックを更新(現在時刻) {
            if (!状態.ゲーム.実行中) return;
            const デルタ秒 = (現在時刻 - 状態.ゲーム.最終更新時間) / 1000;
            if (デルタ秒 >= 1.0) {
                状態.ゲーム.残り時間--; UI.表示_残り時間.innerText = 状態.ゲーム.残り時間 + "s";
                状態.ゲーム.最終更新時間 = 現在時刻;
                if (状態.ゲーム.残り時間 <= 0) ゲーム終了();
            }

            const 最終ターゲット = 状態.ゲーム.ターゲット一覧[状態.ゲーム.ターゲット一覧.length - 1];
            // 画面の幅に連動して新しいターゲットの発生間隔(距離)を最適化
            if (!最終ターゲット || 最終ターゲット.x < 画面幅 - 240) {
                if (状態.ゲーム.現在のメロディインデックス < かえるの合唱音階.length) メロディターゲットを投下する();
                else if (状態.ゲーム.ターゲット一覧.length === 0) ゲーム終了();
            }

            // 自機のX位置を論理画面幅の22%の位置に設定
            const プレイヤーX = 画面幅 * 0.22;
            const プレイヤーY = 周波数をY座標に変換(状態.滑らかなピッチHz);

            for (let i = 状態.ゲーム.ターゲット一覧.length - 1; i >= 0; i--) {
                const ターゲット = 状態.ゲーム.ターゲット一覧[i];
                ターゲット.更新();

                if (ターゲット.休符) {
                    if (ターゲット.x < プレイヤーX) 状態.ゲーム.ターゲット一覧.splice(i, 1);
                    continue;
                }

                if (!ターゲット.通過済み && ターゲット.x < プレイヤーX + 25 && ターゲット.x + ターゲット.幅 > プレイヤーX - 25) {
                    const 誤差 = Math.abs(状態.滑らかなピッチHz - ターゲット.目標周波数);
                    if (誤差 <= ターゲット.許容範囲 && 状態.現在のピッチHz > 0 && 状態.滑らかなデシベル > -45) {
                        ターゲット.通過済み = true;
                        状態.ゲーム.スコア += Math.max(15, Math.floor((ターゲット.許容範囲 - 誤差) * 12 * 状態.ゲーム.難易度));
                        UI.表示_スコア.innerText = 状態.ゲーム.スコア;

                        const 色 = ['#4ade80', '#60a5fa', '#facc15', '#f87171'][Math.floor(Math.random() * 4)];
                        for (let j = 0; j < 25; j++) 状態.ゲーム.パーティクル一覧.push(new 歌唱パーティクル(プレイヤーX, プレイヤーY, 色));
                    }
                }
                if (ターゲット.x + ターゲット.幅 < 0) 状態.ゲーム.ターゲット一覧.splice(i, 1);
            }

            for (let i = 状態.ゲーム.パーティクル一覧.length - 1; i >= 0; i--) {
                const p = 状態.ゲーム.パーティクル一覧[i];
                p.更新();
                if (p.寿命 <= 0) 状態.ゲーム.パーティクル一覧.splice(i, 1);
            }
        }

        function ゲーム終了() {
            状態.ゲーム.実行中 = false; UI.表示_最終スコア.innerText = 状態.ゲーム.スコア;
            let メッセージ = "素晴らしい喉を持っていますね！🐸";
            if (状態.ゲーム.スコア > 2500) メッセージ = "もはやプロの合唱団！神がかった正確さです！👑🐸";
            else if (状態.ゲーム.スコア > 1200) メッセージ = "素晴らしい！かえる達も喜んで大合唱しています！🎵";
            else if (状態.ゲーム.スコア < 400) メッセージ = "もう少し声を大きく、はっきりと音程を合わせてみよう！🍀";
            UI.ゲーム評価メッセージ.innerText = メッセージ; UI.オーバーレイ_ゲーム結果.classList.remove('hidden');
        }

        function ゲームを描画する() {
            コンテキスト.fillStyle = '#0b0f19'; 
            コンテキスト.fillRect(0, 0, 画面幅, 画面高さ);
            
            コンテキスト.strokeStyle = 'rgba(255, 255, 255, 0.05)'; 
            コンテキスト.fillStyle = 'rgba(148, 163, 184, 0.35)'; 
            コンテキスト.font = 'bold 12px monospace';
            
            [{名前: "ド(C4)", f: 261.63}, {名前: "レ(D4)", f: 293.66}, {名前: "ミ(E4)", f: 329.63}, {名前: "ファ(F4)", f: 349.23}, {名前: "ソ(G4)", f: 392.00}, {名前: "ラ(A4)", f: 440.00}].forEach(音 => {
                const y = 周波数をY座標に変換(音.f);
                コンテキスト.beginPath(); コンテキスト.moveTo(0, y); コンテキスト.lineTo(画面幅, y); コンテキスト.stroke();
                コンテキスト.textAlign = 'left'; コンテキスト.fillText(音.名前, 15, y - 4);
            });

            状態.ゲーム.ターゲット一覧.forEach(ターゲット => ターゲット.描画(コンテキスト));

            const プレイヤーX = 画面幅 * 0.22; 
            const プレイヤーY = 周波数をY座標に変換(状態.滑らかなピッチHz);
            const 歌唱中 = 状態.現在のピッチHz > 0 && 状態.滑らかなデシベル > -45;

            if (歌唱中) {
                コンテキスト.save();
                const グラデーション = コンテキスト.createLinearGradient(プレイヤーX, プレイヤーY, 画面幅, プレイヤーY);
                グラデーション.addColorStop(0, `rgba(56, 189, 248, ${Math.max(0.2, (状態.滑らかなデシベル + 60) / 60)})`); 
                グラデーション.addColorStop(1, 'rgba(56, 189, 248, 0)');
                コンテキスト.beginPath(); コンテキスト.moveTo(プレイヤーX, プレイヤーY); コンテキスト.lineTo(画面幅, プレイヤーY);
                コンテキスト.strokeStyle = グラデーション; 
                コンテキスト.lineWidth = 14 * ((状態.滑らかなデシベル + 60) / 60);
                コンテキスト.shadowBlur = 15; 
                コンテキスト.shadowColor = '#38bdf8'; 
                コンテキスト.stroke(); 
                コンテキスト.restore();
            }

            コンテキスト.save(); 
            コンテキスト.beginPath();
            const 半径 = 歌唱中 ? 18 : 12; 
            コンテキスト.arc(プレイヤーX, プレイヤーY, 半径, 0, Math.PI * 2);
            コンテキスト.fillStyle = 歌唱中 ? '#22c55e' : '#475569'; 
            if (歌唱中) { コンテキスト.shadowBlur = 15; コンテキスト.shadowColor = '#22c55e'; }
            コンテキスト.fill();
            
            コンテキスト.beginPath(); 
            コンテキスト.arc(プレイヤーX, プレイヤーY, 半径 * 0.5, 0, Math.PI * 2); 
            コンテキスト.fillStyle = '#ffffff'; 
            コンテキスト.fill(); 
            コンテキスト.restore();

            状態.ゲーム.パーティクル一覧.forEach(p => p.描画(コンテキスト));

            if (!状態.ゲーム.実行中 && !UI.オーバーレイ_ゲーム結果.classList.contains('hidden')) {
                 コンテキスト.fillStyle = 'rgba(15, 23, 42, 0.75)'; 
                 コンテキスト.fillRect(0, 0, 画面幅, 画面高さ);
                 コンテキスト.fillStyle = '#ffffff'; 
                 コンテキスト.font = 'bold 20px sans-serif'; 
                 コンテキスト.textAlign = 'center';
                 コンテキスト.fillText('「演奏を始める！」ボタンを押してスタート！', 画面幅 / 2, 画面高さ / 2);
            }
            ダブルメーターを描画する();
        }

        // ==========================================
        // 8. アニメーションメインループ
        // ==========================================
        function 描画ループ(現在時刻) {
            データを解析する();
            if (状態.モード === 'シミュレーション') シミュレーションを描画する();
            else { ゲームロジックを更新(現在時刻); ゲームを描画する(); }
            requestAnimationFrame(描画ループ);
        }
        requestAnimationFrame(描画ループ);

    </script>
</body>
</html>
