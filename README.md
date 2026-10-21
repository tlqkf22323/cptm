<!DOCTYPE html>
<html lang="ko" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>온라인 2인 체스 - GitHub Pages 배포 가능</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome 아이콘 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts: Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chess.js 규칙 검증 라이브러리 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
    <!-- Firebase SDKs (v9 Compat) -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        .chess-board {
            display: grid;
            grid-template-columns: repeat(8, minmax(0, 1fr));
            grid-template-rows: repeat(8, minmax(0, 1fr));
            aspect-ratio: 1 / 1;
        }
        .chess-piece {
            transition: transform 0.15s ease-in-out;
            cursor: grab;
        }
        .chess-piece:active {
            cursor: grabbing;
        }
    </style>
</head>
<body class="h-full bg-slate-900 text-slate-100 flex flex-col justify-between overflow-x-hidden">

    <!-- 상단 헤더 -->
    <header class="bg-slate-800 border-b border-slate-700 px-6 py-4 flex items-center justify-between shadow-md">
        <div class="flex items-center space-x-3">
            <div class="bg-indigo-600 p-2 rounded-xl text-white shadow-lg shadow-indigo-500/30">
                <i class="fa-solid fa-chess text-xl"></i>
            </div>
            <div>
                <h1 class="font-bold text-lg tracking-wide text-white">온라인 체스 그랜드마스터</h1>
                <p class="text-xs text-slate-400">실시간 멀티플레이어 (한국어 지원)</p>
            </div>
        </div>
        <div class="flex items-center space-x-3">
            <button onclick="openConfigModal()" class="bg-slate-700 hover:bg-slate-600 text-slate-200 px-3 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 border border-slate-600">
                <i class="fa-solid fa-gear"></i>
                <span class="hidden sm:inline">파이어베이스 설정</span>
            </button>
            <button onclick="openRulesModal()" class="bg-slate-700 hover:bg-slate-600 text-slate-200 px-3 py-2 rounded-lg text-sm font-medium transition flex items-center space-x-2 border border-slate-600">
                <i class="fa-solid fa-circle-question"></i>
                <span class="hidden sm:inline">사용 방법</span>
            </button>
        </div>
    </header>

    <!-- 메인 컨테이너 -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6 flex flex-col items-center justify-center">

        <!-- 화면 1: 로비 / 방 선택 화면 -->
        <div id="lobby-screen" class="w-full max-w-md bg-slate-800 border border-slate-700 rounded-2xl p-6 md:p-8 shadow-2xl flex flex-col space-y-6">
            <div class="text-center space-y-2">
                <h2 class="text-2xl font-bold text-white">온라인 체스 대국</h2>
                <p class="text-sm text-slate-400">방을 생성하거나 친구의 방 코드를 입력해 접속하세요.</p>
            </div>

            <!-- 상태 배너 -->
            <div id="connection-status-banner" class="p-3 rounded-xl bg-amber-950/50 border border-amber-800/60 text-amber-300 text-xs flex items-center space-x-2">
                <i class="fa-solid fa-triangle-exclamation text-sm shrink-0"></i>
                <span id="status-text">현재 데모(오프라인) 모드입니다. 온라인 대국을 위해 파이어베이스 설정을 완료해주세요!</span>
            </div>

            <div class="space-y-4 pt-2">
                <button onclick="createRoom()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-semibold py-3.5 px-4 rounded-xl shadow-lg shadow-indigo-600/30 transition flex items-center justify-center space-x-2">
                    <i class="fa-solid fa-plus-circle text-lg"></i>
                    <span>새로운 방 만들기</span>
                </button>

                <div class="relative flex py-2 items-center">
                    <div class="flex-grow border-t border-slate-700"></div>
                    <span class="flex-shrink mx-4 text-slate-500 text-xs uppercase tracking-wider">또는</span>
                    <div class="flex-grow border-t border-slate-700"></div>
                </div>

                <div class="space-y-3">
                    <input type="text" id="room-code-input" placeholder="6자리 방 코드 입력" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-3 text-white placeholder-slate-500 focus:outline-none focus:border-indigo-500 text-center tracking-widest font-mono text-lg uppercase">
                    <button onclick="joinRoom()" class="w-full bg-slate-700 hover:bg-slate-600 text-white font-semibold py-3 px-4 rounded-xl transition flex items-center justify-center space-x-2 border border-slate-600">
                        <i class="fa-solid fa-right-to-bracket"></i>
                        <span>방 참가하기</span>
                    </button>
                </div>
            </div>

            <div class="pt-4 border-t border-slate-700/60 text-center">
                <button onclick="startLocalGame()" class="text-xs text-indigo-400 hover:text-indigo-300 underline font-medium">
                    혼자서 연습하기 (오프라인 2인용)
                </button>
            </div>
        </div>

        <!-- 화면 2: 게임 대국실 화면 -->
        <div id="game-screen" class="hidden w-full max-w-6xl grid grid-cols-1 lg:grid-cols-3 gap-6 items-start">
            
            <!-- 좌측/메인 열: 체스판 및 플레이어 정보 -->
            <div class="lg:col-span-2 flex flex-col items-center space-y-4">
                
                <!-- 방 정보 헤더 -->
                <div class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 flex items-center justify-between shadow">
                    <div class="flex items-center space-x-2 text-sm">
                        <span class="text-slate-400">방 코드:</span>
                        <span id="display-room-code" class="font-mono font-bold text-indigo-400 bg-slate-900 px-2.5 py-1 rounded border border-slate-700">----</span>
                        <button onclick="copyRoomCode()" class="text-slate-400 hover:text-white transition p-1" title="코드 복사">
                            <i class="fa-solid fa-copy"></i>
                        </button>
                    </div>
                    <div id="game-status-badge" class="bg-indigo-950 text-indigo-300 border border-indigo-800 text-xs px-3 py-1 rounded-full font-medium">
                        상대방 기다리는 중...
                    </div>
                    <button onclick="leaveRoom()" class="text-rose-400 hover:text-rose-300 text-xs font-semibold px-3 py-1 rounded bg-rose-950/40 border border-rose-900 transition">
                        나가기
                    </button>
                </div>

                <!-- 상대방 플레이어 태그 -->
                <div class="w-full flex items-center justify-between px-2 text-sm text-slate-300">
                    <div class="flex items-center space-x-2">
                        <div class="w-3 h-3 rounded-full bg-slate-600" id="black-status-dot"></div>
                        <span id="black-player-label" class="font-medium">흑 (상대방)</span>
                    </div>
                    <div id="black-captured" class="flex space-x-1 text-xs text-slate-400"></div>
                </div>

                <!-- 체스판 컴포넌트 -->
                <div class="relative w-full max-w-[520px] bg-slate-800 p-3 rounded-2xl border border-slate-700 shadow-2xl">
                    <div id="chessboard" class="chess-board rounded-lg overflow-hidden shadow-inner select-none bg-slate-700">
                        <!-- 동적 생성되는 체스 칸들 -->
                    </div>
                </div>

                <!-- 나(본인) 플레이어 태그 -->
                <div class="w-full flex items-center justify-between px-2 text-sm text-slate-300">
                    <div class="flex items-center space-x-2">
                        <div class="w-3 h-3 rounded-full bg-emerald-500 animate-pulse" id="white-status-dot"></div>
                        <span id="white-player-label" class="font-medium">백 (나)</span>
                    </div>
                    <div id="white-captured" class="flex space-x-1 text-xs text-slate-400"></div>
                </div>
            </div>

            <!-- 우측 열: 사이드바 (기보 기록, 실시간 채팅, 조작) -->
            <div class="w-full bg-slate-800 border border-slate-700 rounded-2xl p-4 md:p-5 shadow-2xl flex flex-col space-y-4 h-[600px]">
                <!-- 탭 선택 -->
                <div class="flex border-b border-slate-700 text-sm">
                    <button onclick="switchTab('moves')" id="tab-moves-btn" class="flex-1 pb-2 font-semibold text-indigo-400 border-b-2 border-indigo-500 transition">기보 기록</button>
                    <button onclick="switchTab('chat')" id="tab-chat-btn" class="flex-1 pb-2 font-medium text-slate-400 hover:text-slate-200 transition">실시간 채팅</button>
                </div>

                <!-- 기보 탭 내용 -->
                <div id="tab-moves-content" class="flex-1 overflow-y-auto pr-1 space-y-1 font-mono text-xs">
                    <table class="w-full text-left">
                        <thead>
                            <tr class="text-slate-500 border-b border-slate-700/50">
                                <th class="pb-2 w-12">#</th>
                                <th class="pb-2">백</th>
                                <th class="pb-2">흑</th>
                            </tr>
                        </thead>
                        <tbody id="move-history-body" class="divide-y divide-slate-700/30">
                            <!-- 이동 내역이 동적으로 추가됩니다 -->
                        </tbody>
                    </table>
                </div>

                <!-- 채팅 탭 내용 -->
                <div id="tab-chat-content" class="hidden flex-1 flex flex-col justify-between">
                    <div id="chat-messages" class="flex-1 overflow-y-auto space-y-2 pr-1 text-xs text-slate-300">
                        <div class="text-slate-500 text-center py-2 italic">채팅방에 연결되었습니다.</div>
                    </div>
                    <div class="pt-2 flex space-x-2">
                        <input type="text" id="chat-input" onkeydown="if(event.key==='Enter') sendChatMessage()" placeholder="메시지를 입력하세요..." class="flex-1 bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-xs text-white focus:outline-none focus:border-indigo-500">
                        <button onclick="sendChatMessage()" class="bg-indigo-600 hover:bg-indigo-500 text-white px-3 py-2 rounded-lg text-xs font-semibold transition">
                            <i class="fa-solid fa-paper-plane"></i>
                        </button>
                    </div>
                </div>

                <!-- 하단 게임 제어 버튼 -->
                <div class="pt-3 border-t border-slate-700 flex space-x-2">
                    <button onclick="offerResign()" class="flex-1 bg-rose-950/50 hover:bg-rose-900/50 text-rose-300 border border-rose-800/60 py-2 rounded-xl text-xs font-semibold transition">
                        기권하기
                    </button>
                    <button onclick="offerDraw()" class="flex-1 bg-slate-700 hover:bg-slate-600 text-slate-200 border border-slate-600 py-2 rounded-xl text-xs font-semibold transition">
                        무승부 제안
                    </button>
                </div>
            </div>

        </div>

    </main>

    <!-- 하단 푸터 -->
    <footer class="text-center py-4 text-xs text-slate-500 border-t border-slate-800">
        온라인 체스 그랜드마스터 &bull; GitHub Pages 및 Firebase로 완벽 호환 호스팅
    </footer>

    <!-- 모달: 파이어베이스 설정 -->
    <div id="config-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="bg-slate-800 border border-slate-700 rounded-2xl max-w-md w-full p-6 space-y-4 shadow-2xl">
            <div class="flex items-center justify-between">
                <h3 class="font-bold text-lg text-white">파이어베이스 설정</h3>
                <button onclick="closeConfigModal()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark text-lg"></i></button>
            </div>
            <p class="text-xs text-slate-400 leading-relaxed">
                다른 기기의 친구와 실시간 온라인 대국을 하려면 무료 파이어베이스(Firebase Realtime Database) 설정을 입력하세요. 빈칸으로 두면 오프라인 2인용 연습 모드로 작동합니다.
            </p>
            <div class="space-y-3">
                <div>
                    <label class="block text-xs font-medium text-slate-300 mb-1">파이어베이스 설정 JSON 입력</label>
                    <textarea id="firebase-config-input" rows="6" placeholder='{
  "apiKey": "AIzaSy...",
  "authDomain": "your-app.firebaseapp.com",
  "databaseURL": "https://your-app-default-rtdb.firebaseio.com",
  "projectId": "your-app"
}' class="w-full bg-slate-900 border border-slate-700 rounded-xl p-3 text-xs font-mono text-white placeholder-slate-600 focus:outline-none focus:border-indigo-500"></textarea>
                </div>
            </div>
            <div class="flex space-x-3 pt-2">
                <button onclick="saveFirebaseConfig()" class="flex-1 bg-indigo-600 hover:bg-indigo-500 text-white py-2.5 rounded-xl text-sm font-semibold transition">저장 및 연결</button>
                <button onclick="clearFirebaseConfig()" class="bg-slate-700 hover:bg-slate-600 text-slate-300 px-4 py-2.5 rounded-xl text-sm font-medium transition">설정 지우기</button>
            </div>
        </div>
    </div>

    <!-- 모달: 사용 가이드 -->
    <div id="rules-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="bg-slate-800 border border-slate-700 rounded-2xl max-w-lg w-full p-6 space-y-4 shadow-2xl">
            <div class="flex items-center justify-between">
                <h3 class="font-bold text-lg text-white">GitHub Pages 호스팅 가이드</h3>
                <button onclick="closeRulesModal()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark text-lg"></i></button>
            </div>
            <div class="space-y-3 text-xs text-slate-300 leading-relaxed">
                <p><strong>1. 싱글 파일 구조:</strong> 이 게임은 하나의 HTML 파일로 구성되어 있어 GitHub 저장소에 `index.html`로 업로드하기만 하면 바로 배포됩니다.</p>
                <p><strong>2. GitHub Pages 활성화:</strong> 저장소 설정(Settings) > Pages에서 배포 소스를 메인 브랜치로 지정하면 누구나 웹 링크로 접속할 수 있습니다.</p>
                <p><strong>3. 실시간 멀티플레이:</strong> 무료 <a href="https://console.firebase.google.com" target="_blank" class="text-indigo-400 underline">Firebase 콘솔</a>에서 실시간 데이터베이스(Realtime Database)를 생성한 뒤 위 설정 버튼을 눌러 JSON을 입력하면 전 세계 어디서든 친구와 실시간 대국이 가능합니다.</p>
            </div>
            <button onclick="closeRulesModal()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white py-2.5 rounded-xl text-sm font-semibold transition mt-2">확인</button>
        </div>
    </div>

    <!-- 모달: 알림 박스 (alert 대체) -->
    <div id="alert-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="bg-slate-800 border border-slate-700 rounded-2xl max-w-sm w-full p-6 space-y-4 shadow-2xl text-center">
            <div id="alert-icon" class="w-12 h-12 rounded-full bg-indigo-950 text-indigo-400 border border-indigo-800 flex items-center justify-center mx-auto text-xl">
                <i class="fa-solid fa-info"></i>
            </div>
            <h3 id="alert-title" class="font-bold text-lg text-white">알림</h3>
            <p id="alert-message" class="text-xs text-slate-300">내용이 여기에 표시됩니다.</p>
            <button onclick="closeAlertModal()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white py-2.5 rounded-xl text-sm font-semibold transition">확인</button>
        </div>
    </div>

    <!-- 모달: 폰 프로모션 선택 -->
    <div id="promotion-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="bg-slate-800 border border-slate-700 rounded-2xl max-w-xs w-full p-6 space-y-4 shadow-2xl text-center">
            <h3 class="font-bold text-lg text-white">폰 승격 (Promotion)</h3>
            <p class="text-xs text-slate-400">변경할 체스 기물을 선택하세요:</p>
            <div id="promotion-options" class="flex justify-center space-x-3 py-2">
                <!-- 동적 생성 -->
            </div>
        </div>
    </div>

    <!-- 자바스크립트 게임 로직 및 파이어베이스 연동 -->
    <script>
        // --- 게임 상태 변수 ---
        let chess = new Chess();
        let playerColor = 'white'; // 'white' 또는 'black'
        let roomCode = null;
        let firebaseDB = null;
        let firebaseConnected = false;
        let dbRef = null;
        let selectedSquare = null;
        let possibleMoves = [];
        let pendingPromotionMove = null;
        let localMode = false;

        // 기물 유니코드 매핑
        const pieceUnicode = {
            'w': { 'p': '♙', 'r': '♖', 'n': '♘', 'b': '♗', 'q': '♕', 'k': '♔' },
            'b': { 'p': '♟', 'r': '♜', 'n': '♞', 'b': '♝', 'q': '♛', 'k': '♚' }
        };

        // --- 초기화 ---
        window.addEventListener('DOMContentLoaded', () => {
            loadSavedFirebaseConfig();
            renderBoard();
        });

        // --- 파이어베이스 설정 관리 ---
        function loadSavedFirebaseConfig() {
            const saved = localStorage.getItem('grandmaster_firebase_config_kr');
            if (saved) {
                try {
                    const config = JSON.parse(saved);
                    document.getElementById('firebase-config-input').value = saved;
                    initFirebase(config);
                } catch (e) {
                    console.error("저장된 설정 파싱 실패");
                }
            }
        }

        function initFirebase(config) {
            try {
                if (!firebase.apps.length) {
                    firebase.initializeApp(config);
                }
                firebaseDB = firebase.database();
                firebaseConnected = true;
                document.getElementById('connection-status-banner').className = "p-3 rounded-xl bg-emerald-950/50 border border-emerald-800/60 text-emerald-300 text-xs flex items-center space-x-2";
                document.getElementById('status-text').innerText = "파이어베이스가 성공적으로 연결되었습니다! 온라인 방 생성이 가능합니다.";
            } catch (e) {
                console.error(e);
                showAlert("연결 오류", "파이어베이스 설정 JSON을 올바르게 입력했는지 확인해주세요.");
                firebaseConnected = false;
            }
        }

        function saveFirebaseConfig() {
            const val = document.getElementById('firebase-config-input').value.trim();
            if (!val) {
                clearFirebaseConfig();
                return;
            }
            try {
                const config = JSON.parse(val);
                localStorage.setItem('grandmaster_firebase_config_kr', val);
                initFirebase(config);
                closeConfigModal();
                showAlert("성공", "파이어베이스 설정이 저장되었습니다!");
            } catch (e) {
                showAlert("잘못된 형식", "올바른 JSON 형식을 입력해주세요.");
            }
        }

        function clearFirebaseConfig() {
            localStorage.removeItem('grandmaster_firebase_config_kr');
            firebaseConnected = false;
            firebaseDB = null;
            document.getElementById('firebase-config-input').value = '';
            document.getElementById('connection-status-banner').className = "p-3 rounded-xl bg-amber-950/50 border border-amber-800/60 text-amber-300 text-xs flex items-center space-x-2";
            document.getElementById('status-text').innerText = "현재 데모(오프라인) 모드입니다. 온라인 대국을 위해 파이어베이스 설정을 완료해주세요!";
            closeConfigModal();
            showAlert("모드 변경", "오프라인 연습 모드로 전환되었습니다.");
        }

        // --- 방 관리 ---
        function generateRoomCode() {
            return Math.random().toString(36).substring(2, 8).toUpperCase();
        }

        function createRoom() {
            if (!firebaseConnected) {
                showAlert("설정 필요", "온라인 방을 생성하려면 먼저 상단의 '파이어베이스 설정' 버튼을 눌러 설정을 완료하거나, 하단의 '혼자서 연습하기'를 이용해주세요.");
                return;
            }
            roomCode = generateRoomCode();
            playerColor = 'white';
            localMode = false;
            setupFirebaseListeners();
            
            dbRef.set({
                fen: chess.fen(),
                turn: 'w',
                whitePlayer: '호스트',
                blackPlayer: null,
                lastMove: null,
                chats: { 0: { sender: '시스템', text: '방이 생성되었습니다. 상대방 접속을 기다리는 중...' } }
            });

            enterGameScreen();
        }

        function joinRoom() {
            const codeInput = document.getElementById('room-code-input').value.trim().toUpperCase();
            if (!codeInput || codeInput.length < 4) {
                showAlert("코드 오류", "올바른 6자리 방 코드를 입력해주세요.");
                return;
            }
            if (!firebaseConnected) {
                showAlert("설정 필요", "온라인 방에 참가하려면 먼저 파이어베이스 설정을 완료해주세요.");
                return;
            }
            roomCode = codeInput;
            playerColor = 'black';
            localMode = false;
            setupFirebaseListeners();

            firebaseDB.ref('rooms/' + roomCode).once('value', (snapshot) => {
                if (snapshot.exists()) {
                    const data = snapshot.val();
                    chess.load(data.fen);
                    firebaseDB.ref('rooms/' + roomCode + '/blackPlayer').set('참가자');
                    firebaseDB.ref('rooms/' + roomCode + '/chats').push({ sender: '시스템', text: '상대방이 입장했습니다.' });
                    enterGameScreen();
                } else {
                    showAlert("방을 찾을 수 없음", "해당 코드로 생성된 방이 없습니다: " + roomCode);
                    leaveRoom();
                }
            });
        }

        function startLocalGame() {
            localMode = true;
            roomCode = "LOCAL";
            playerColor = 'white';
            chess.reset();
            enterGameScreen();
            document.getElementById('display-room-code').innerText = "오프라인";
            document.getElementById('game-status-badge').innerText = "로컬 2인 연습 모드";
            document.getElementById('black-player-label').innerText = "흑 (플레이어 2)";
            document.getElementById('white-player-label').innerText = "백 (플레이어 1)";
        }

        function enterGameScreen() {
            document.getElementById('lobby-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.remove('hidden');
            document.getElementById('display-room-code').innerText = roomCode;
            
            if (!localMode) {
                if (playerColor === 'white') {
                    document.getElementById('white-player-label').innerText = "나 (백)";
                    document.getElementById('black-player-label').innerText = "상대방 (흑)";
                    document.getElementById('game-status-badge').innerText = "상대방 접속 대기 중...";
                } else {
                    document.getElementById('white-player-label').innerText = "상대방 (백)";
                    document.getElementById('black-player-label').innerText = "나 (흑)";
                    document.getElementById('game-status-badge').innerText = "흑 플레이어로 접속 완료";
                }
            }
            renderBoard();
            updateMoveHistoryUI();
        }

        function leaveRoom() {
            if (dbRef && !localMode) {
                dbRef.off();
            }
            roomCode = null;
            chess.reset();
            selectedSquare = null;
            possibleMoves = [];
            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('lobby-screen').classList.remove('hidden');
            document.getElementById('room-code-input').value = '';
            document.getElementById('chat-messages').innerHTML = '<div class="text-slate-500 text-center py-2 italic">채팅방에 연결되었습니다.</div>';
        }

        function setupFirebaseListeners() {
            dbRef = firebaseDB.ref('rooms/' + roomCode);
            
            dbRef.on('value', (snapshot) => {
                if (!snapshot.exists()) return;
                const data = snapshot.val();
                
                if (data.fen && data.fen !== chess.fen()) {
                    chess.load(data.fen);
                    renderBoard();
                    updateMoveHistoryUI();
                    checkGameStatus();
                }

                if (playerColor === 'white' && data.blackPlayer) {
                    document.getElementById('game-status-badge').innerText = "대국 진행 중 (내 차례)";
                }
            });

            firebaseDB.ref('rooms/' + roomCode + '/chats').on('child_added', (snapshot) => {
                const msg = snapshot.val();
                appendChatMessage(msg.sender, msg.text);
            });
        }

        // --- 체스판 렌더링 및 인터랙션 ---
        function renderBoard() {
            const boardEl = document.getElementById('chessboard');
            boardEl.innerHTML = '';

            const isFlipped = (playerColor === 'black' && !localMode);
            const files = isFlipped ? ['h','g','f','e','d','c','b','a'] : ['a','b','c','d','e','f','g','h'];
            const ranks = isFlipped ? ['1','2','3','4','5','6','7','8'] : ['8','7','6','5','4','3','2','1'];

            for (let r = 0; r < 8; r++) {
                for (let f = 0; f < 8; f++) {
                    const squareName = files[f] + ranks[r];
                    const piece = chess.get(squareName);
                    const isDark = (r + f) % 2 === 1;
                    const isSelected = (selectedSquare === squareName);
                    const isPossibleMove = possibleMoves.includes(squareName);

                    const squareDiv = document.createElement('div');
                    squareDiv.className = `relative flex items-center justify-center text-4xl sm:text-5xl font-bold select-none cursor-pointer transition-colors duration-150 ${
                        isDark ? 'bg-slate-600 hover:bg-slate-500' : 'bg-slate-300 hover:bg-slate-200'
                    }`;
                    if (isSelected) squareDiv.classList.add('ring-4', 'ring-indigo-500', 'z-10');

                    squareDiv.onclick = () => handleSquareClick(squareName);

                    if (piece) {
                        const pieceSpan = document.createElement('span');
                        pieceSpan.className = `chess-piece ${piece.color === 'w' ? 'text-amber-100 drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)]' : 'text-slate-900 drop-shadow-[0_1px_2px_rgba(255,255,255,0.4)]'}`;
                        pieceSpan.innerText = pieceUnicode[piece.color][piece.type];
                        squareDiv.appendChild(pieceSpan);
                    }

                    if (isPossibleMove) {
                        const indicator = document.createElement('div');
                        indicator.className = `absolute w-4 h-4 rounded-full ${piece ? 'border-4 border-indigo-500/80 inset-0 m-auto' : 'bg-indigo-500/60'}`;
                        squareDiv.appendChild(indicator);
                    }

                    boardEl.appendChild(squareDiv);
                }
            }
        }

        function handleSquareClick(square) {
            if (!localMode) {
                const myTurn = (chess.turn() === playerColor[0]);
                if (!myTurn) return;
            }

            if (selectedSquare === null) {
                const piece = chess.get(square);
                if (piece && piece.color === chess.turn()) {
                    selectedSquare = square;
                    possibleMoves = chess.moves({ square: square, verbose: true }).map(m => m.to);
                    renderBoard();
                }
            } else {
                if (possibleMoves.includes(square)) {
                    const piece = chess.get(selectedSquare);
                    if (piece && piece.type === 'p' && ((piece.color === 'w' && square[1] === '8') || (piece.color === 'b' && square[1] === '1'))) {
                        pendingPromotionMove = { from: selectedSquare, to: square };
                        showPromotionModal();
                        selectedSquare = null;
                        possibleMoves = [];
                        return;
                    }
                    executeMove({ from: selectedSquare, to: square, promotion: 'q' });
                } else {
                    selectedSquare = null;
                    possibleMoves = [];
                    renderBoard();
                }
            }
        }

        function executeMove(moveObj) {
            const result = chess.move(moveObj);
            selectedSquare = null;
            possibleMoves = [];

            if (result) {
                renderBoard();
                updateMoveHistoryUI();
                checkGameStatus();

                if (!localMode && dbRef) {
                    dbRef.update({
                        fen: chess.fen(),
                        turn: chess.turn(),
                        lastMove: moveObj
                    });
                }
            }
        }

        // --- 프로모션 ---
        function showPromotionModal() {
            const container = document.getElementById('promotion-options');
            container.innerHTML = '';
            const color = chess.turn();
            ['q', 'r', 'b', 'n'].forEach(type => {
                const btn = document.createElement('button');
                btn.className = 'w-14 h-14 bg-slate-700 hover:bg-slate-600 rounded-xl text-3xl flex items-center justify-center text-white border border-slate-600 transition';
                btn.innerHTML = pieceUnicode[color][type];
                btn.onclick = () => {
                    closePromotionModal();
                    if (pendingPromotionMove) {
                        pendingPromotionMove.promotion = type;
                        executeMove(pendingPromotionMove);
                        pendingPromotionMove = null;
                    }
                };
                container.appendChild(btn);
            });
            document.getElementById('promotion-modal').classList.remove('hidden');
            document.getElementById('promotion-modal').classList.add('flex');
        }

        function closePromotionModal() {
            document.getElementById('promotion-modal').classList.add('hidden');
            document.getElementById('promotion-modal').classList.remove('flex');
        }

        // --- 게임 상태 및 기보 관리 ---
        function checkGameStatus() {
            let statusText = "";
            const turnName = chess.turn() === 'w' ? '백' : '흑';

            if (chess.in_checkmate()) {
                const winner = chess.turn() === 'w' ? '흑' : '백';
                statusText = `체크메이트! ${winner} 승리!`;
                showAlert("게임 종료", statusText);
            } else if (chess.in_draw()) {
                statusText = "무승부 (스테일메이트 또는 기물 부족)";
                showAlert("게임 종료", statusText);
            } else if (chess.in_check()) {
                statusText = `${turnName} 체크 상태!`;
            } else {
                statusText = `${turnName} 차례`;
            }

            if (!localMode) {
                const myTurn = (chess.turn() === playerColor[0]);
                document.getElementById('game-status-badge').innerText = myTurn ? "내 차례 (" + statusText + ")" : "상대방 차례 (" + statusText + ")";
            } else {
                document.getElementById('game-status-badge').innerText = statusText;
            }
        }

        function updateMoveHistoryUI() {
            const history = chess.history({ verbose: true });
            const tbody = document.getElementById('move-history-body');
            tbody.innerHTML = '';

            for (let i = 0; i < history.length; i += 2) {
                const tr = document.createElement('tr');
                tr.className = 'border-b border-slate-700/20';
                
                const numTd = document.createElement('td');
                numTd.className = 'py-1.5 text-slate-500';
                numTd.innerText = Math.floor(i / 2) + 1;
                tr.appendChild(numTd);

                const whiteTd = document.createElement('td');
                whiteTd.className = 'py-1.5 text-slate-200 font-medium';
                whiteTd.innerText = history[i].san;
                tr.appendChild(whiteTd);

                const blackTd = document.createElement('td');
                blackTd.className = 'py-1.5 text-slate-200 font-medium';
                blackTd.innerText = history[i + 1] ? history[i + 1].san : '';
                tr.appendChild(blackTd);

                tbody.appendChild(tr);
            }
            const container = document.getElementById('tab-moves-content');
            container.scrollTop = container.scrollHeight;
        }

        // --- 채팅 기능 ---
        function switchTab(tab) {
            const movesBtn = document.getElementById('tab-moves-btn');
            const chatBtn = document.getElementById('tab-chat-btn');
            const movesContent = document.getElementById('tab-moves-content');
            const chatContent = document.getElementById('tab-chat-content');

            if (tab === 'moves') {
                movesBtn.className = "flex-1 pb-2 font-semibold text-indigo-400 border-b-2 border-indigo-500 transition";
                chatBtn.className = "flex-1 pb-2 font-medium text-slate-400 hover:text-slate-200 transition";
                movesContent.classList.remove('hidden');
                chatContent.classList.add('hidden');
            } else {
                chatBtn.className = "flex-1 pb-2 font-semibold text-indigo-400 border-b-2 border-indigo-500 transition";
                movesBtn.className = "flex-1 pb-2 font-medium text-slate-400 hover:text-slate-200 transition";
                chatContent.classList.remove('hidden');
                movesContent.classList.add('hidden');
            }
        }

        function sendChatMessage() {
            const input = document.getElementById('chat-input');
            const text = input.value.trim();
            if (!text) return;

            const senderName = localMode ? "플레이어" : (playerColor === 'white' ? '백' : '흑');

            if (!localMode && dbRef) {
                firebaseDB.ref('rooms/' + roomCode + '/chats').push({
                    sender: senderName,
                    text: text
                });
            } else {
                appendChatMessage(senderName, text);
            }
            input.value = '';
        }

        function appendChatMessage(sender, text) {
            const chatMsgs = document.getElementById('chat-messages');
            const div = document.createElement('div');
            div.className = "bg-slate-900/60 p-2 rounded-lg border border-slate-700/50";
            div.innerHTML = `<span class="font-bold text-indigo-400">${sender}:</span> <span class="text-slate-200">${escapeHtml(text)}</span>`;
            chatMsgs.appendChild(div);
            chatMsgs.scrollTop = chatMsgs.scrollHeight;
        }

        function escapeHtml(text) {
            return text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
        }

        // --- 유틸리티 및 조작 ---
        function offerResign() {
            showAlert("기권", "경기에 기권하였습니다. 상대방이 승리했습니다.");
            chess.reset();
            renderBoard();
        }

        function offerDraw() {
            showAlert("무승부 제안", "상대방에게 무승부 제안을 보냈습니다.");
        }

        function copyRoomCode() {
            navigator.clipboard.writeText(roomCode);
            showAlert("복사 완료", "방 코드가 클립보드에 복사되었습니다!");
        }

        function openConfigModal() {
            document.getElementById('config-modal').classList.remove('hidden');
            document.getElementById('config-modal').classList.add('flex');
        }

        function closeConfigModal() {
            document.getElementById('config-modal').classList.add('hidden');
            document.getElementById('config-modal').classList.remove('flex');
        }

        function openRulesModal() {
            document.getElementById('rules-modal').classList.remove('hidden');
            document.getElementById('rules-modal').classList.add('flex');
        }

        function closeRulesModal() {
            document.getElementById('rules-modal').classList.add('hidden');
            document.getElementById('rules-modal').classList.remove('flex');
        }

        function showAlert(title, message) {
            document.getElementById('alert-title').innerText = title;
            document.getElementById('alert-message').innerText = message;
            document.getElementById('alert-modal').classList.remove('hidden');
            document.getElementById('alert-modal').classList.add('flex');
        }

        function closeAlertModal() {
            document.getElementById('alert-modal').classList.add('hidden');
            document.getElementById('alert-modal').classList.remove('flex');
        }
    </script>
</body>
<script type="module">
  // Import the functions you need from the SDKs you need
  import { initializeApp } from "https://www.gstatic.com/firebasejs/13.0.0/firebase-app.js";
  // TODO: Add SDKs for Firebase products that you want to use
  // https://firebase.google.com/docs/web/setup#available-libraries

  // Your web app's Firebase configuration
  const firebaseConfig = {
    apiKey: "AIzaSyBNLqrgKEGISYsufWI8P1qXsdsiAC6h0d0",
    authDomain: "cotm-26d79.firebaseapp.com",
    projectId: "cotm-26d79",
    storageBucket: "cotm-26d79.firebasestorage.app",
    messagingSenderId: "543223403492",
    appId: "1:543223403492:web:2581f359e527b4c106edf1"
  };

  // Initialize Firebase
  const app = initializeApp(firebaseConfig);
</script>
</html>
