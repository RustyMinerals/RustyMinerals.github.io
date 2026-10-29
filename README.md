<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>나만의 스마트 단어장</title>
    <!-- 실시간 기기 동기화를 위한 Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-database-compat.js"></script>
    <style>
        :root {
            --primary-color: #4CAF50;
            --primary-hover: #45a049;
            --secondary-color: #2196F3;
            --danger-color: #f44336;
            --warning-color: #ffc107;
            --bg-color: #f4f6f9;
            --card-bg: #ffffff;
            --text-color: #333;
        }
        
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif;
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            padding-bottom: 80px;
        }

        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }

        .view {
            display: none;
        }
        .view.active {
            display: block;
        }

        .header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 20px;
            padding-bottom: 12px;
            border-bottom: 2px solid #e2e8f0;
        }
        
        .btn-back {
            background: #e2e8f0;
            border: none;
            font-size: 14px;
            font-weight: bold;
            cursor: pointer;
            color: #4a5568;
            padding: 8px 14px;
            border-radius: 6px;
            transition: background 0.2s;
        }
        .btn-back:hover { background: #cbd5e0; }

        .backup-controls {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-bottom: 15px;
        }
        .btn-sub {
            background-color: #edf2f7;
            border: 1px solid #cbd5e0;
            padding: 6px 12px;
            border-radius: 6px;
            font-size: 13px;
            cursor: pointer;
            font-weight: bold;
            color: #4a5568;
        }
        .btn-sub:hover { background-color: #e2e8f0; }

        .btn-sync-active {
            background-color: #e8f5e9;
            border-color: #4CAF50;
            color: #2e7d32;
        }

        h2 { margin: 0; font-size: 20px; }

        ul { list-style: none; padding: 0; margin: 0; }

        .list-item {
            background-color: var(--card-bg);
            padding: 16px;
            margin-bottom: 12px;
            border-radius: 10px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.04);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        #home-view .list-item,
        #file-view #word-list .list-item { cursor: pointer; transition: transform 0.1s, box-shadow 0.1s; }
        #home-view .list-item:hover,
        #file-view #word-list .list-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 10px rgba(0,0,0,0.08);
        }

        .draggable-item {
            cursor: grab;
            transition: transform 0.1s, box-shadow 0.1s, background-color 0.2s;
        }
        .draggable-item:active { cursor: grabbing; }
        .draggable-item.dragging {
            opacity: 0.5;
            background-color: #e2e8f0;
            transform: scale(0.98);
        }

        .word-content { flex-grow: 1; }
        .word-title-group { display: flex; align-items: center; gap: 8px; margin-bottom: 4px; }
        .word-text { font-weight: bold; font-size: 18px; }
        .meaning-text { color: #666; font-size: 14px; }

        .badge-wrong {
            background-color: var(--danger-color);
            color: white;
            font-size: 11px;
            padding: 3px 6px;
            border-radius: 12px;
            margin-left: 8px;
            vertical-align: middle;
            font-weight: normal;
        }

        .badge-auto {
            background-color: #ff9800;
            color: white;
            font-size: 11px;
            padding: 3px 6px;
            border-radius: 12px;
            margin-left: 8px;
            vertical-align: middle;
            font-weight: normal;
        }
        .special-file-item {
            background: linear-gradient(135deg, #fff8ec, #ffedd5);
            border: 1px solid #ffb74d;
        }
        .blue-file-item {
            background: linear-gradient(135deg, #e3f2fd, #bbdefb);
            border: 1px solid #64b5f6;
        }

        .group-divider {
            margin-top: 26px;
            padding-top: 18px;
            border-top: 2px dashed #cbd5e0;
            position: relative;
        }
        .group-divider::before {
            content: '헷갈리는 단어 구분선';
            position: absolute;
            top: -10px;
            left: 50%;
            transform: translateX(-50%);
            background: var(--bg-color);
            padding: 0 8px;
            font-size: 11px;
            color: #a0aec0;
            white-space: nowrap;
        }

        .search-highlight {
            outline: 3px solid var(--secondary-color);
            background-color: #e8f4ff;
        }
        #search-bar-container input,
        #home-search-bar-container input {
            width: 100%;
            padding: 10px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            font-size: 14px;
            box-sizing: border-box;
        }
        #search-bar-container input:focus,
        #home-search-bar-container input:focus { outline: 2px solid var(--secondary-color); border-color: transparent; }

        .btn-tts {
            background: none;
            border: none;
            cursor: pointer;
            font-size: 18px;
            padding: 2px 6px;
            border-radius: 4px;
        }
        .btn-tts:hover { background-color: #edf2f7; }

        .action-buttons button {
            margin-left: 4px;
            padding: 6px 10px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 13px;
            font-weight: bold;
        }
        .btn-edit { background-color: var(--warning-color); color: #000; }
        .btn-delete { background-color: var(--danger-color); color: #fff; }
        
        .btn-test {
            display: block;
            width: 100%;
            padding: 14px;
            background-color: var(--primary-color);
            color: white;
            text-align: center;
            border: none;
            border-radius: 10px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            margin-bottom: 20px;
            box-shadow: 0 3px 8px rgba(76,175,80,0.3);
        }
        .btn-test:hover { background-color: var(--primary-hover); }

        .add-form {
            display: flex;
            gap: 10px;
            margin-top: 20px;
            background-color: var(--card-bg);
            padding: 16px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.08);
        }
        
        .input-wrapper {
            position: relative;
            flex: 1;
            display: flex;
            align-items: center;
        }
        .add-form input {
            width: 100%;
            padding: 10px;
            padding-right: 36px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            font-size: 14px;
            box-sizing: border-box;
        }
        .add-form input:focus { outline: 2px solid var(--primary-color); border-color: transparent; }
        
        .btn-paren {
            display: none;
            position: absolute;
            right: 6px;
            background: #e2e8f0;
            border: none;
            border-radius: 4px;
            padding: 4px 8px;
            font-size: 13px;
            font-weight: bold;
            cursor: pointer;
            color: #4a5568;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
        .btn-paren:hover { background: #cbd5e0; }

        .btn-add {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 10px 18px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }

        .btn-wrong-test {
            display: block;
            width: 100%;
            padding: 12px;
            background-color: #fff;
            color: var(--danger-color);
            text-align: center;
            border: 2px solid var(--danger-color);
            border-radius: 10px;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            margin-bottom: 20px;
        }
        .btn-wrong-test:hover { background-color: #fff5f5; }

        .btn-listen {
            display: block;
            width: 100%;
            padding: 12px;
            background-color: #fff;
            color: var(--secondary-color);
            text-align: center;
            border: 2px solid var(--secondary-color);
            border-radius: 10px;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            margin-bottom: 20px;
        }
        .btn-listen:hover { background-color: #eef6ff; }
        #listen-slider {
            width: 100%;
            margin: 20px 0 8px 0;
            accent-color: var(--secondary-color);
        }

        .modal-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.45);
            z-index: 2000;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        .modal-box {
            background: #fff;
            border-radius: 14px;
            padding: 24px;
            max-width: 420px;
            width: 100%;
            box-shadow: 0 8px 24px rgba(0,0,0,0.2);
            box-sizing: border-box;
        }
        .modal-box h3 { margin: 0 0 14px 0; font-size: 17px; }
        .modal-box p { margin: 0 0 12px 0; }
        .modal-box button { margin-bottom: 10px; }
        .modal-box button:last-child { margin-bottom: 0; }
        #bulk-add-textarea {
            width: 100%;
            box-sizing: border-box;
            padding: 10px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            font-size: 14px;
            font-family: inherit;
            resize: vertical;
        }

        .fab {
            position: fixed;
            bottom: 30px;
            right: 30px;
            width: 56px;
            height: 56px;
            background-color: var(--primary-color);
            color: white;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 28px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.25);
            cursor: pointer;
            border: none;
            z-index: 1000;
        }
        .fab:hover { background-color: var(--primary-hover); }

        .fab-divider {
            position: fixed;
            bottom: 30px;
            right: 96px;
            width: 56px;
            height: 56px;
            background-color: var(--secondary-color);
            color: white;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 22px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.25);
            cursor: pointer;
            border: none;
            z-index: 1000;
        }
        .fab-divider:hover { background-color: #1976d2; }

        .fab-search-third {
            position: fixed;
            bottom: 30px;
            right: 162px;
            width: 56px;
            height: 56px;
            background-color: var(--secondary-color);
            color: white;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 22px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.25);
            cursor: pointer;
            border: none;
            z-index: 1000;
        }
        .fab-search-third:hover { background-color: #1976d2; }

        .nav-arrows-container {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 12px;
            margin-bottom: 20px;
        }
        .btn-nav-arrow {
            background: #e2e8f0;
            border: none;
            font-size: 18px;
            color: #4a5568;
            cursor: pointer;
            width: 44px;
            height: 44px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background 0.2s, transform 0.1s;
        }
        .btn-nav-arrow:hover { background: #cbd5e0; transform: scale(1.05); }
        .btn-nav-arrow:disabled { color: #a0aec0; background: transparent; cursor: not-allowed; transform: scale(1); }
        
        .test-card {
            background-color: var(--card-bg);
            padding: 40px 20px;
            text-align: center;
            border-radius: 14px;
            box-shadow: 0 4px 16px rgba(0,0,0,0.08);
            min-height: 220px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: relative;
            flex: 1;
        }
        .test-word { font-size: 32px; font-weight: bold; margin-bottom: 6px; word-break: break-all; }
        .test-phonetic {
            font-size: 16px;
            color: #64748b;
            margin-bottom: 14px;
            font-weight: normal;
            min-height: 22px;
            word-break: break-all;
        }
        .test-meaning { font-size: 22px; color: #e91e63; font-weight: 500; min-height: 32px; word-break: break-all; }
        .hidden-text { visibility: hidden; }
        
        .test-controls {
            display: flex;
            gap: 12px;
            justify-content: center;
            flex-direction: column;
        }
        .test-controls button {
            padding: 12px 20px;
            font-size: 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        .btn-reveal { background-color: var(--secondary-color); color: white; width: 100%; }
        .btn-correct { background-color: var(--primary-color); color: white; flex: 1; }
        .btn-wrong { background-color: var(--danger-color); color: white; flex: 1; }

        .result-box {
            text-align: center;
            background: white;
            padding: 30px 20px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.08);
        }
        .result-score { font-size: 48px; font-weight: bold; color: var(--primary-color); margin: 15px 0; }

        .empty-message { text-align: center; color: #888; padding: 30px 10px; }

        /* 매칭 퀴즈 스타일 */
        .matching-modal-box {
            max-width: 520px;
            max-height: 88vh;
            overflow-y: auto;
        }
        .matching-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin: 14px 0;
        }
        .matching-col {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .match-card {
            padding: 12px 10px;
            border: 2px solid #cbd5e0;
            border-radius: 8px;
            background: #fff;
            font-size: 14px;
            font-weight: 600;
            color: #333;
            cursor: pointer;
            text-align: center;
            word-break: break-word;
            transition: all 0.15s ease;
            min-height: 44px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-sizing: border-box;
        }
        .match-card:hover:not(:disabled) {
            border-color: var(--secondary-color);
            background: #f0f8ff;
        }
        .match-card.selected {
            border-color: var(--secondary-color);
            background-color: #e3f2fd;
            color: #0d47a1;
            transform: scale(1.02);
        }
        .match-card.matched {
            border-color: var(--primary-color);
            background-color: #e8f5e9;
            color: #2e7d32;
            opacity: 0.6;
            cursor: default;
        }
        .match-card.wrong {
            border-color: var(--danger-color);
            background-color: #ffebee;
            color: #c62828;
        }

        /* 예문 추가/보기/수정 모달 스타일 */
        .example-modal-box {
            max-width: 500px;
            max-height: 86vh;
            overflow-y: auto;
        }
        .example-word-banner {
            background: #f1f5f9;
            padding: 10px 14px;
            border-radius: 8px;
            margin-bottom: 14px;
            font-size: 14px;
            color: #334155;
            display: flex;
            flex-wrap: wrap;
            align-items: center;
            gap: 8px;
        }
        .example-list-container {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 16px;
            max-height: 260px;
            overflow-y: auto;
        }
        .example-item-card {
            background: #f8fafc;
            border: 1px solid #e2e8f0;
            border-left: 4px solid var(--secondary-color);
            border-radius: 8px;
            padding: 12px;
            text-align: left;
        }
        .example-item-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 6px;
            gap: 6px;
        }
        .example-meaning-badge {
            background: #e0f2fe;
            color: #0369a1;
            font-size: 11px;
            font-weight: bold;
            padding: 3px 8px;
            border-radius: 12px;
        }
        .example-item-en {
            font-size: 14px;
            font-weight: bold;
            color: #1e293b;
            margin-bottom: 4px;
            line-height: 1.4;
            word-break: break-word;
        }
        .example-item-ko {
            font-size: 13px;
            color: #64748b;
            line-height: 1.4;
            word-break: break-word;
        }
        .example-form-box {
            background: #ffffff;
            border: 1px solid #cbd5e0;
            border-radius: 10px;
            padding: 14px;
            margin-bottom: 12px;
        }
        .example-form-title {
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 10px;
            color: #2d3748;
        }
        .example-form-field {
            width: 100%;
            box-sizing: border-box;
            padding: 9px 10px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            font-size: 13px;
            margin-bottom: 8px;
            font-family: inherit;
        }
        .example-form-field:focus {
            outline: 2px solid var(--secondary-color);
            border-color: transparent;
        }
        .example-item-actions {
            display: flex;
            gap: 4px;
        }
        .example-item-actions button {
            margin: 0 !important;
            padding: 4px 8px;
            font-size: 11px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <!-- 1. 메인 화면 (단어장 목록) -->
    <div id="home-view" class="view active container">
        <div class="header">
            <h2>내 단어장 목록</h2>
        </div>

        <div id="home-search-bar-container" style="display: none; margin-bottom: 15px;">
            <input type="text" id="home-search-word-input" placeholder="모든 단어장에서 단어 또는 뜻을 검색하세요" oninput="searchWordInAllFiles()">
        </div>
        <ul id="home-search-results" style="display: none;"></ul>

        <div class="backup-controls">
            <button class="btn-sub" onclick="exportData()">💾 백업 내보내기</button>
            <button class="btn-sub" onclick="document.getElementById('import-file').click()">📂 백업 불러오기</button>
            <button id="btn-cloud-sync" class="btn-sub" onclick="openCloudSyncModal()">☁️ 실시간 동기화</button>
            <input type="file" id="import-file" style="display: none;" accept=".json" onchange="importData(event)">
        </div>

        <!-- 실시간 클라우드 동기화 설정 모달 -->
        <div id="cloud-sync-overlay" class="modal-overlay">
            <div class="modal-box">
                <h3>☁️ 기기 간 실시간 동기화</h3>
                <p style="font-size: 12px; color: #666; line-height: 1.5;">
                    Firebase Realtime Database 주소와 본인만의 동기화 키를 입력하면 모든 기기에서 단어·오답·예문이 실시간으로 함께 업데이트됩니다.
                </p>
                <div style="margin-bottom: 10px;">
                    <label style="font-size: 12px; font-weight: bold; color: #4a5568; display: block; margin-bottom: 4px;">Firebase DB 주소 (URL)</label>
                    <input type="text" id="cloud-db-url-input" class="example-form-field" placeholder="https://프로젝트명-default-rtdb.firebaseio.com">
                </div>
                <div style="margin-bottom: 12px;">
                    <label style="font-size: 12px; font-weight: bold; color: #4a5568; display: block; margin-bottom: 4px;">동기화 암호키 (모든 기기 동일하게 입력)</label>
                    <input type="text" id="cloud-sync-key-input" class="example-form-field" placeholder="예: wooseok-vocab-1234">
                </div>
                <div id="cloud-sync-status-text" style="font-size: 12px; margin-bottom: 12px; color: #64748b; font-weight: bold;">상태: 미연결</div>
                <button class="btn-test" style="margin-bottom: 8px; padding: 12px;" onclick="saveAndConnectCloudSync()">🔗 연동 시작 (실시간 동기화)</button>
                <div style="display: flex; gap: 6px; margin-bottom: 8px;">
                    <button class="btn-sub" style="flex: 1; padding: 10px;" onclick="forcePushToCloud()">⬆️ 현재 기기 데이터 업로드</button>
                    <button class="btn-sub" style="flex: 1; padding: 10px;" onclick="forcePullFromCloud()">⬇️ 클라우드 데이터 내려받기</button>
                </div>
                <button class="btn-sub" style="width: 100%; padding: 10px; color: var(--danger-color); margin-bottom: 8px;" onclick="disconnectCloudSync()">🔌 연동 해제</button>
                <button class="btn-sub" style="width: 100%; padding: 10px;" onclick="closeCloudSyncModal()">닫기</button>
            </div>
        </div>

        <ul id="file-list"></ul>
        <button class="fab-divider" onclick="toggleHomeSearchBar()" title="전체 단어장에서 단어 검색">🔍</button>
        <button class="fab" onclick="createNewFile()">+</button>
    </div>

    <!-- 2. 단어장 상세 화면 -->
    <div id="file-view" class="view container">
        <div class="header">
            <button class="btn-back" onclick="navigate('home')">⬅ 뒤로가기</button>
            <h2 id="current-file-name">파일명</h2>
            <div style="width: 70px;"></div>
        </div>

        <div id="search-bar-container" style="display: none; margin-bottom: 15px;">
            <input type="text" id="search-word-input" placeholder="찾고 싶은 단어를 입력하세요" oninput="searchWordInList()">
        </div>
        
        <button class="btn-test" onclick="startTest()">📝 테스트 시작</button>
        <!-- 오답 학습 및 테스트 버튼 (동적 업데이트됨) -->
        <button id="btn-wrong-test-dynamic" class="btn-wrong-test" onclick="handleWrongTestButton()" style="display: none;">🔁 오답 학습하기 (0/2)</button>
        <button class="btn-listen" onclick="startListening()">🔊 발음 듣기</button>
        
        <ul id="word-list"></ul>
        
        <div class="add-form" id="add-form-container">
            <div class="input-wrapper">
                <input type="text" id="input-word" placeholder="단어 (예: apple)" onkeypress="handleEnter(event)">
                <button class="btn-paren" onmousedown="insertParentheses(event, 'input-word')">()</button>
            </div>
            <div class="input-wrapper">
                <input type="text" id="input-meaning" placeholder="뜻 (예: 사과)" onkeypress="handleEnter(event)">
                <button class="btn-paren" onmousedown="insertParentheses(event, 'input-meaning')">()</button>
            </div>
            <button class="btn-add" onclick="openAddChoiceModal()">추가</button>
        </div>

        <!-- 모달들 -->
        <div id="add-choice-overlay" class="modal-overlay">
            <div class="modal-box">
                <h3>어떤 방식으로 추가할까요?</h3>
                <button class="btn-test" style="margin-bottom: 10px;" onclick="confirmSingleAdd()">✏️ 단일 추가</button>
                <button class="btn-test" style="background-color: var(--secondary-color); box-shadow: none;" onclick="openBulkAddModal()">📋 일괄 추가</button>
                <button class="btn-sub" style="width: 100%; padding: 12px;" onclick="closeAddChoiceModal()">취소</button>
            </div>
        </div>

        <div id="bulk-add-overlay" class="modal-overlay">
            <div class="modal-box">
                <h3>📋 일괄 추가</h3>
                <p style="font-size: 13px; color: #666;">한 줄에 하나씩, "단어, 뜻" 형식으로 입력하세요.<br>예) apple, 사과</p>
                <textarea id="bulk-add-textarea" rows="8" placeholder="apple, 사과&#10;banana, 바나나"></textarea>
                <button class="btn-add" style="width: 100%; margin-top: 10px;" onclick="submitBulkAdd()">일괄 추가하기</button>
                <button class="btn-sub" style="width: 100%; padding: 12px;" onclick="closeBulkAddModal()">취소</button>
            </div>
        </div>

        <div id="segment-select-overlay" class="modal-overlay">
            <div class="modal-box">
                <h3>🚩 테스트할 구간을 선택하세요</h3>
                <div id="segment-checkbox-list" style="margin-bottom: 14px;"></div>
                <button class="btn-test" onclick="confirmSegmentTest()">선택한 구간으로 테스트 시작</button>
                <button class="btn-sub" style="width: 100%; padding: 12px;" onclick="closeSegmentSelectModal()">취소</button>
            </div>
        </div>

        <button class="fab-search-third" onclick="toggleSearchBar()" title="단어 검색">🔍</button>
        <button class="fab-divider" onclick="addDividerAtEnd()" title="경계선 추가">🚩</button>
        <button class="fab" onclick="scrollToAddForm()">+</button>
    </div>

    <!-- 3. 테스트 및 학습 화면 -->
    <div id="test-view" class="view container">
        <div class="header">
            <button class="btn-back" onclick="handleExitTestView()">⬅ 중단</button>
            <h2 id="test-progress-text">단어 테스트</h2>
            <div style="width: 70px;"></div>
        </div>

        <div style="text-align: right; margin-bottom: 12px;">
            <button id="btn-toggle-mode" onclick="toggleTestMode()" class="btn-sub">모드: 단어 제시 🔄</button>
        </div>

        <div id="test-content">
            <!-- 화살표 네비게이션 적용된 테스트 카드 영역 -->
            <div class="nav-arrows-container">
                <button id="btn-test-prev" class="btn-nav-arrow" onclick="prevTestWord()">◀</button>
                
                <div class="test-card">
                    <div id="test-word-container" style="display: flex; align-items: center; gap: 8px;">
                        <div id="test-word-display" class="test-word">단어</div>
                        <button class="btn-tts" id="test-tts-btn" onclick="speakCurrentTestWord()">🔊</button>
                    </div>
                    <!-- [추가된 기능]: 영단어 발음 기호 표시 영역 -->
                    <div id="test-phonetic-display" class="test-phonetic"></div>
                    <div id="test-meaning-display" class="test-meaning hidden-text">뜻</div>
                    
                    <!-- [수정된 기능]: 예문 추가 및 보기/수정 버튼 (클릭 시 별도 모달 창 오픈) -->
                    <button id="btn-show-example" class="btn-sub" style="display: none; margin-top: 15px; font-size: 13px;" onclick="openExampleModal()">➕ 예문 추가</button>
                </div>

                <button id="btn-test-next" class="btn-nav-arrow" onclick="nextTestWord()">▶</button>
            </div>

            <div class="test-controls">
                <button id="btn-reveal-meaning" class="btn-reveal" onclick="revealMeaning()">정답 확인하기</button>
                <div id="score-buttons" style="display: none; width: 100%; gap: 10px;">
                    <button class="btn-correct" onclick="recordAnswer(true)">⭕ 알아요</button>
                    <button class="btn-wrong" onclick="recordAnswer(false)">❌ 몰라요</button>
                </div>
                <!-- 오답 학습 모드 시 표시되는 '다음' 버튼 -->
                <div id="study-next-btn-container" style="display: none; width: 100%;">
                    <button id="btn-study-next" class="btn-correct" style="width: 100%;" onclick="nextStudyWord()">다음 단어</button>
                </div>
            </div>
        </div>

        <div id="test-result" class="result-box" style="display: none;">
            <h3>🎉 테스트 완료!</h3>
            <div id="result-score-display" class="result-score">0%</div>
            <p id="result-detail-text" style="color: #666;">총 0문제 중 0개 맞춤</p>
            <button class="btn-test" onclick="startTest()" style="margin-top: 20px;">🔄 다시 테스트하기</button>
            <button class="btn-sub" onclick="navigate('file')" style="width: 100%; padding: 12px;">단어장으로 돌아가기</button>
        </div>

        <!-- [추가된 기능]: 예문 추가/보기/수정 전용 모달 창 -->
        <div id="example-modal-overlay" class="modal-overlay">
            <div class="modal-box example-modal-box">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
                    <h3 style="margin: 0;">📝 예문 보기 및 관리</h3>
                    <button class="btn-sub" style="margin: 0; padding: 4px 10px;" onclick="closeExampleModal()">✕ 닫기</button>
                </div>

                <div class="example-word-banner">
                    <strong id="example-modal-word" style="font-size: 16px; color: #0f172a;"></strong>
                    <span id="example-modal-meaning" style="color: #e91e63; font-weight: 600;"></span>
                </div>

                <!-- 등록된 예문 목록 -->
                <div id="example-list-container" class="example-list-container"></div>

                <!-- 예문 작성 / 수정 폼 -->
                <div class="example-form-box">
                    <div id="example-form-title" class="example-form-title">➕ 새 예문 추가</div>
                    
                    <div id="example-meaning-select-wrap" style="display: none;">
                        <select id="example-input-meaning" class="example-form-field"></select>
                    </div>

                    <input type="text" id="example-input-en" class="example-form-field" placeholder="영어 예문을 입력하세요 (예: I eat an apple.)">
                    <input type="text" id="example-input-ko" class="example-form-field" placeholder="한글 해석을 입력하세요 (예: 나는 사과를 먹는다.)">

                    <div style="display: flex; gap: 8px; margin-top: 4px;">
                        <button id="btn-save-example" class="btn-add" style="flex: 1; padding: 10px; margin: 0;" onclick="saveExampleItem()">예문 저장</button>
                        <button id="btn-cancel-edit-example" class="btn-sub" style="display: none; padding: 10px; margin: 0;" onclick="resetExampleForm()">수정 취소</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- [추가된 기능]: 누적 10개(및 나머지) 매칭 퀴즈 모달 -->
        <div id="matching-quiz-overlay" class="modal-overlay">
            <div class="modal-box matching-modal-box">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px;">
                    <h3 style="margin: 0;">🧩 단어 매칭 퀴즈</h3>
                    <span id="matching-quiz-progress" style="font-size: 13px; font-weight: bold; color: var(--secondary-color);">0 / 0</span>
                </div>
                <p style="font-size: 13px; color: #666; margin-bottom: 10px;">방금 학습한 단어와 뜻을 알맞게 짝지어 주세요.</p>
                <div class="matching-grid">
                    <div id="matching-words-col" class="matching-col"></div>
                    <div id="matching-meanings-col" class="matching-col"></div>
                </div>
                <button id="btn-matching-continue" class="btn-test" style="display: none; margin-top: 12px; margin-bottom: 0;" onclick="closeMatchingQuizAndContinue()">🎉 퀴즈 완료! 계속하기</button>
            </div>
        </div>
    </div>

    <!-- 4. 발음 듣기 화면 -->
    <div id="listen-view" class="view container">
        <div class="header">
            <button class="btn-back" onclick="stopListening(); navigate('file')">⬅ 중단</button>
            <h2 id="listen-progress-text">발음 듣기</h2>
            <div style="width: 70px;"></div>
        </div>

        <div class="test-card" style="margin-bottom: 20px;">
            <div style="display: flex; align-items: center; gap: 8px;">
                <div id="listen-word-display" class="test-word">단어</div>
                <button class="btn-tts" onclick="speakCurrentListenWord()">🔊</button>
            </div>
            <div id="listen-phonetic-display" class="test-phonetic"></div>
            <div id="listen-meaning-display" class="test-meaning hidden-text">뜻</div>
            <div id="listen-play-count" style="margin-top: 10px; font-size: 13px; color: #999;"></div>
        </div>

        <input type="range" id="listen-slider" min="0" max="0" value="0" oninput="onListenSliderChange(this.value)">
        <p style="text-align: center; font-size: 12px; color: #999; margin-top: 6px;">슬라이더를 옮기면 해당 단어부터 다시 3번 재생됩니다.</p>
    </div>

    <script>
        let vocabData = JSON.parse(localStorage.getItem('vocabApp')) || {};
        let wrongStudyCounts = JSON.parse(localStorage.getItem('vocabWrongStudy')) || {};
        let wrongStudyProgress = JSON.parse(localStorage.getItem('vocabWrongStudyProgress')) || {};
        let savedSessions = JSON.parse(localStorage.getItem('vocabSavedSessions')) || {};
        let currentFile = null;
        
        let testWords = [];
        let currentTestIndex = 0;
        let testMode = 'word';
        let correctCount = 0;
        let isStudyMode = false;
        let isWrongOnlyTest = false;
        let currentSessionAnswers = {};
        let currentDividerIndex = null;

        let listenWords = [];
        let listenIndex = 0;
        let listenPlayCount = 0;
        let listenTimeoutId = null;

        let currentTestSegments = [];

        // 매칭 퀴즈 상태 변수
        let matchingQuizCallback = null;
        let selectedMatchWordBtn = null;
        let selectedMatchMeaningBtn = null;
        let matchedPairsCount = 0;
        let totalMatchPairs = 0;

        // 예문 수정 상태 변수
        let editingExampleIndex = null;

        // 실시간 클라우드 동기화 상태 변수
        let cloudSyncConfig = JSON.parse(localStorage.getItem('vocabCloudConfig')) || { dbUrl: '', syncKey: '' };
        let firebaseAppInstance = null;
        let firebaseDbRef = null;
        let isApplyingRemoteUpdate = false;
        let clientDeviceId = localStorage.getItem('vocabDeviceId') || ('dev-' + Math.random().toString(36).substring(2, 10));
        localStorage.setItem('vocabDeviceId', clientDeviceId);

        const SPECIAL_FILE_NAME = '헷갈릴 수 있는 단어 모음';
        const SAME_MEANING_FILE_NAME = '같은 뜻 단어 모음';
        const OVERLAP_WORD_FILE_NAME = '겹치는 영단어 모음';
        const BLUE_FILE_NAME = '파랭이';
        
        const PREPOSITIONS = new Set([
            'in','out','on','off','up','down','over','under','through',
            'about','around','along','across','into','onto','upon',
            'with','without','for','from','by','at','to','of'
        ]);

        window.onload = () => {
            if (vocabData['비슷한 뜻 단어 모음']) {
                delete vocabData['비슷한 뜻 단어 모음'];
                saveData();
            }

            if (!vocabData[BLUE_FILE_NAME]) {
                vocabData[BLUE_FILE_NAME] = [];
                saveData();
            }

            consolidateOverlappingEnglishWords();
            navigate('home');

            if (cloudSyncConfig.dbUrl && cloudSyncConfig.syncKey) {
                initCloudSyncConnection(false);
            }
            
            document.querySelectorAll('.add-form input').forEach(input => {
                input.addEventListener('focus', function() {
                    this.nextElementSibling.style.display = 'block';
                });
                input.addEventListener('blur', function() {
                    this.nextElementSibling.style.display = 'none';
                });
            });

            if (typeof initListenSwipeGesture === 'function') {
                initListenSwipeGesture();
            }
        };

        window.addEventListener('beforeunload', () => {
            const testViewEl = document.getElementById('test-view');
            if (testViewEl && testViewEl.classList.contains('active')) {
                saveCurrentSessionState();
            }
        });

        // ==========================================
        // 실시간 클라우드 동기화 (Firebase Realtime DB)
        // ==========================================
        function openCloudSyncModal() {
            document.getElementById('cloud-db-url-input').value = cloudSyncConfig.dbUrl || '';
            document.getElementById('cloud-sync-key-input').value = cloudSyncConfig.syncKey || '';
            document.getElementById('cloud-sync-overlay').style.display = 'flex';
        }

        function closeCloudSyncModal() {
            document.getElementById('cloud-sync-overlay').style.display = 'none';
        }

        function updateCloudSyncUI(connected, message) {
            const btn = document.getElementById('btn-cloud-sync');
            const statusText = document.getElementById('cloud-sync-status-text');
            if (connected) {
                btn.classList.add('btn-sync-active');
                btn.innerText = '🟢 실시간 동기화중';
                if (statusText) {
                    statusText.style.color = '#2e7d32';
                    statusText.innerText = message || '상태: 실시간 동기화 연결됨';
                }
            } else {
                btn.classList.remove('btn-sync-active');
                btn.innerText = '☁️ 실시간 동기화';
                if (statusText) {
                    statusText.style.color = '#64748b';
                    statusText.innerText = message || '상태: 미연결';
                }
            }
        }

        function hasMeaningfulLocalData() {
            const keys = Object.keys(vocabData || {});
            if (keys.length === 0) return false;
            if (keys.length === 1 && keys[0] === BLUE_FILE_NAME && (vocabData[BLUE_FILE_NAME] || []).length === 0) {
                return false;
            }
            return true;
        }

        function saveAndConnectCloudSync() {
            let dbUrl = document.getElementById('cloud-db-url-input').value.trim();
            const syncKey = document.getElementById('cloud-sync-key-input').value.trim();

            if (!dbUrl || !syncKey) {
                alert('Firebase DB 주소와 동기화 암호키를 모두 입력해주세요.');
                return;
            }
            if (dbUrl.endsWith('/')) {
                dbUrl = dbUrl.slice(0, -1);
            }

            cloudSyncConfig = { dbUrl, syncKey };
            localStorage.setItem('vocabCloudConfig', JSON.stringify(cloudSyncConfig));
            initCloudSyncConnection(true);
        }

        function initCloudSyncConnection(showAlert = false) {
            if (!cloudSyncConfig.dbUrl || !cloudSyncConfig.syncKey) return;

            try {
                if (firebaseDbRef) {
                    firebaseDbRef.off();
                }

                const safeKey = cloudSyncConfig.syncKey.replace(/[.#$\[\]\/]/g, '_');
                if (!firebase.apps.length) {
                    firebaseAppInstance = firebase.initializeApp({ databaseURL: cloudSyncConfig.dbUrl });
                } else {
                    firebaseAppInstance = firebase.app();
                }

                firebaseDbRef = firebase.database().ref('vocabSync/' + safeKey);

                firebaseDbRef.once('value').then(snapshot => {
                    const remoteVal = snapshot.val();
                    if (remoteVal && remoteVal.vocabDataJson) {
                        applyCloudSnapshot(remoteVal);
                    } else if (hasMeaningfulLocalData()) {
                        pushAllDataToCloud();
                    }
                    updateCloudSyncUI(true, `상태: '${cloudSyncConfig.syncKey}' 실시간 연동중`);
                    if (showAlert) {
                        alert('실시간 동기화가 연결되었습니다! 이제 모든 기기에서 실시간으로 반영됩니다.');
                        closeCloudSyncModal();
                    }

                    firebaseDbRef.on('value', snap => {
                        const data = snap.val();
                        if (!data || !data.vocabDataJson) return;
                        if (data.senderId === clientDeviceId) return;
                        applyCloudSnapshot(data);
                    });
                }).catch(err => {
                    updateCloudSyncUI(false, '상태: 연결 오류 (DB 주소 또는 규칙 확인)');
                    if (showAlert) {
                        alert('DB 연결에 실패했습니다. Firebase Realtime Database 주소와 규칙(테스트 모드)을 확인해주세요.');
                    }
                });
            } catch (err) {
                updateCloudSyncUI(false, '상태: 설정 오류');
                if (showAlert) {
                    alert('연동 초기화 중 오류가 발생했습니다: ' + err.message);
                }
            }
        }

        function applyCloudSnapshot(data) {
            isApplyingRemoteUpdate = true;
            try {
                if (data.vocabDataJson) {
                    vocabData = JSON.parse(data.vocabDataJson) || {};
                    localStorage.setItem('vocabApp', JSON.stringify(vocabData));
                }
                if (data.wrongStudyCountsJson) {
                    wrongStudyCounts = JSON.parse(data.wrongStudyCountsJson) || {};
                    localStorage.setItem('vocabWrongStudy', JSON.stringify(wrongStudyCounts));
                }
                if (data.wrongStudyProgressJson) {
                    wrongStudyProgress = JSON.parse(data.wrongStudyProgressJson) || {};
                    localStorage.setItem('vocabWrongStudyProgress', JSON.stringify(wrongStudyProgress));
                }
                if (data.savedSessionsJson) {
                    savedSessions = JSON.parse(data.savedSessionsJson) || {};
                    localStorage.setItem('vocabSavedSessions', JSON.stringify(savedSessions));
                }

                if (!vocabData[BLUE_FILE_NAME]) {
                    vocabData[BLUE_FILE_NAME] = [];
                }

                if (document.getElementById('home-view').classList.contains('active')) {
                    renderFileList();
                } else if (document.getElementById('file-view').classList.contains('active')) {
                    if (currentFile && vocabData[currentFile]) {
                        renderWordList();
                    } else {
                        navigate('home');
                    }
                }
            } catch (e) {
                console.error('클라우드 데이터 반영 오류:', e);
            } finally {
                isApplyingRemoteUpdate = false;
            }
        }

        function pushAllDataToCloud() {
            if (isApplyingRemoteUpdate || !firebaseDbRef) return;
            try {
                firebaseDbRef.set({
                    vocabDataJson: JSON.stringify(vocabData),
                    wrongStudyCountsJson: JSON.stringify(wrongStudyCounts),
                    wrongStudyProgressJson: JSON.stringify(wrongStudyProgress),
                    savedSessionsJson: JSON.stringify(savedSessions),
                    updatedAt: Date.now(),
                    senderId: clientDeviceId
                });
            } catch (e) {
                console.error('클라우드 업로드 오류:', e);
            }
        }

        function forcePushToCloud() {
            if (!firebaseDbRef) {
                alert('먼저 연동 시작 버튼을 눌러 연결해주세요.');
                return;
            }
            pushAllDataToCloud();
            alert('현재 기기의 데이터를 클라우드에 업로드했습니다.');
        }

        function forcePullFromCloud() {
            if (!firebaseDbRef) {
                alert('먼저 연동 시작 버튼을 눌러 연결해주세요.');
                return;
            }
            firebaseDbRef.once('value').then(snap => {
                const data = snap.val();
                if (data && data.vocabDataJson) {
                    applyCloudSnapshot(data);
                    alert('클라우드에서 최신 데이터를 불러왔습니다.');
                } else {
                    alert('클라우드에 저장된 데이터가 없습니다.');
                }
            });
        }

        function disconnectCloudSync() {
            if (firebaseDbRef) {
                firebaseDbRef.off();
                firebaseDbRef = null;
            }
            cloudSyncConfig = { dbUrl: '', syncKey: '' };
            localStorage.removeItem('vocabCloudConfig');
            updateCloudSyncUI(false, '상태: 미연결');
            alert('실시간 동기화 연동이 해제되었습니다.');
        }

        function insertParentheses(event, inputId) {
            event.preventDefault();
            const input = document.getElementById(inputId);
            const start = input.selectionStart;
            const end = input.selectionEnd;
            const text = input.value;
            input.value = text.substring(0, start) + "()" + text.substring(end);
            input.setSelectionRange(start + 1, start + 1);
            input.focus();
        }

        function escapeHtml(str) {
            return String(str)
                .replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;")
                .replace(/"/g, "&quot;")
                .replace(/'/g, "&#039;");
        }

        function speakWord(text) {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                window.speechSynthesis.speak(utterance);
            } else {
                alert('현재 브라우저는 음성 재생을 지원하지 않습니다.');
            }
        }

        function speakCurrentTestWord() {
            if (testWords[currentTestIndex]) {
                speakWord(testWords[currentTestIndex].word);
            }
        }

        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        function saveData() {
            localStorage.setItem('vocabApp', JSON.stringify(vocabData));
            pushAllDataToCloud();
        }

        function saveWrongStudy() {
            localStorage.setItem('vocabWrongStudy', JSON.stringify(wrongStudyCounts));
            pushAllDataToCloud();
        }

        function saveWrongStudyProgress() {
            localStorage.setItem('vocabWrongStudyProgress', JSON.stringify(wrongStudyProgress));
            pushAllDataToCloud();
        }

        function saveSessionsToStorage() {
            localStorage.setItem('vocabSavedSessions', JSON.stringify(savedSessions));
            pushAllDataToCloud();
        }

        function getSessionKey() {
            if (isStudyMode) return 'study';
            if (isWrongOnlyTest) return 'wrongTest';
            return 'normal';
        }

        function saveCurrentSessionState() {
            if (!currentFile || !testWords || testWords.length === 0) return;
            if (!savedSessions[currentFile]) {
                savedSessions[currentFile] = {};
            }
            const key = getSessionKey();
            savedSessions[currentFile][key] = {
                testWords: testWords,
                currentTestIndex: currentTestIndex,
                testMode: testMode,
                answers: currentSessionAnswers,
                dividerIndex: currentDividerIndex
            };
            saveSessionsToStorage();

            if (isStudyMode) {
                wrongStudyProgress[currentFile] = currentTestIndex;
                saveWrongStudyProgress();
            }
        }

        function clearCurrentSessionState(fileName, key) {
            if (savedSessions[fileName] && savedSessions[fileName][key]) {
                delete savedSessions[fileName][key];
                saveSessionsToStorage();
            }
        }

        // ==========================================
        // 단어 수정/삭제 시 진행중인 테스트 세션 동기화
        // ==========================================
        function syncEditWordInSessions(fileName, oldItem, newItem) {
            if (savedSessions[fileName]) {
                ['normal', 'study', 'wrongTest'].forEach(key => {
                    const session = savedSessions[fileName][key];
                    if (!session || !Array.isArray(session.testWords)) return;
                    session.testWords.forEach((tw, idx) => {
                        if (
                            (tw.word === oldItem.word && tw.meaning === oldItem.meaning) ||
                            tw.word.toLowerCase().trim() === oldItem.word.toLowerCase().trim()
                        ) {
                            session.testWords[idx] = {
                                ...tw,
                                word: newItem.word,
                                meaning: newItem.meaning,
                                phonetic: newItem.phonetic,
                                examples: newItem.examples,
                                example: newItem.example
                            };
                        }
                    });
                });
                saveSessionsToStorage();
            }

            if (currentFile === fileName && Array.isArray(testWords) && testWords.length > 0) {
                testWords.forEach((tw, idx) => {
                    if (
                        (tw.word === oldItem.word && tw.meaning === oldItem.meaning) ||
                        tw.word.toLowerCase().trim() === oldItem.word.toLowerCase().trim()
                    ) {
                        testWords[idx] = {
                            ...tw,
                            word: newItem.word,
                            meaning: newItem.meaning,
                            phonetic: newItem.phonetic,
                            examples: newItem.examples,
                            example: newItem.example
                        };
                    }
                });
            }
        }

        function syncDeleteWordInSessions(fileName, deletedItem, deletedIndexInFile) {
            if (savedSessions[fileName]) {
                ['normal', 'study', 'wrongTest'].forEach(key => {
                    const session = savedSessions[fileName][key];
                    if (!session || !Array.isArray(session.testWords)) return;

                    if (
                        key === 'normal' &&
                        session.dividerIndex !== null &&
                        session.dividerIndex !== undefined &&
                        deletedIndexInFile !== undefined &&
                        deletedIndexInFile < session.dividerIndex
                    ) {
                        session.dividerIndex--;
                    }

                    let delIdx = session.testWords.findIndex(
                        tw => tw.word === deletedItem.word && tw.meaning === deletedItem.meaning
                    );
                    if (delIdx === -1) {
                        delIdx = session.testWords.findIndex(
                            tw => tw.word.toLowerCase().trim() === deletedItem.word.toLowerCase().trim()
                        );
                    }

                    while (delIdx !== -1) {
                        session.testWords.splice(delIdx, 1);

                        if (session.answers) {
                            const newAnswers = {};
                            Object.keys(session.answers).forEach(k => {
                                const numK = parseInt(k, 10);
                                if (numK < delIdx) {
                                    newAnswers[numK] = session.answers[k];
                                } else if (numK > delIdx) {
                                    newAnswers[numK - 1] = session.answers[k];
                                }
                            });
                            session.answers = newAnswers;
                        }

                        if (delIdx < session.currentTestIndex) {
                            session.currentTestIndex--;
                        }

                        if (session.testWords.length === 0) {
                            delete savedSessions[fileName][key];
                            if (key === 'study') {
                                wrongStudyProgress[fileName] = 0;
                                saveWrongStudyProgress();
                            }
                            break;
                        }

                        if (session.currentTestIndex >= session.testWords.length) {
                            session.currentTestIndex = session.testWords.length - 1;
                        }

                        if (key === 'study') {
                            wrongStudyProgress[fileName] = session.currentTestIndex;
                            saveWrongStudyProgress();
                        }

                        delIdx = session.testWords.findIndex(
                            tw => tw.word === deletedItem.word && tw.meaning === deletedItem.meaning
                        );
                    }
                });
                saveSessionsToStorage();
            }
        }

        function navigate(viewName) {
            document.querySelectorAll('.view').forEach(el => el.classList.remove('active'));
            document.getElementById(`${viewName}-view`).classList.add('active');
            
            if (viewName === 'home') {
                renderFileList();
                currentFile = null;
                const homeSearchContainer = document.getElementById('home-search-bar-container');
                homeSearchContainer.style.display = 'none';
                document.getElementById('home-search-word-input').value = '';
                document.getElementById('home-search-results').style.display = 'none';
                document.getElementById('home-search-results').innerHTML = '';
                document.getElementById('file-list').style.display = '';
            } else if (viewName === 'file') {
                const searchContainer = document.getElementById('search-bar-container');
                searchContainer.style.display = 'none';
                document.getElementById('search-word-input').value = '';
                renderWordList();
            }
        }

        // ==========================================
        // 발음 기호(Phonetic) 조회 및 캐싱 함수
        // ==========================================
        async function updatePhoneticDisplay(wordItem, elementId) {
            const el = document.getElementById(elementId);
            if (!el || !wordItem) return;

            if (wordItem.phonetic !== undefined) {
                el.innerText = wordItem.phonetic;
                return;
            }

            el.innerText = '';
            const targetWord = wordItem.word.trim();

            try {
                const response = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${encodeURIComponent(targetWord)}`);
                if (!response.ok) {
                    wordItem.phonetic = '';
                    return;
                }
                const data = await response.json();
                let foundPhonetic = '';

                if (Array.isArray(data) && data.length > 0) {
                    if (data[0].phonetic) {
                        foundPhonetic = data[0].phonetic;
                    } else if (Array.isArray(data[0].phonetics)) {
                        const pObj = data[0].phonetics.find(p => p.text && p.text.trim() !== '');
                        if (pObj) foundPhonetic = pObj.text.trim();
                    }
                }

                wordItem.phonetic = foundPhonetic;

                if (currentFile && vocabData[currentFile]) {
                    const originalIndex = vocabData[currentFile].findIndex(
                        item => !item.isDivider && item.word === wordItem.word && item.meaning === wordItem.meaning
                    );
                    if (originalIndex !== -1) {
                        vocabData[currentFile][originalIndex].phonetic = foundPhonetic;
                        saveData();
                    }
                }

                // 현재 화면에 떠 있는 단어가 동일한 경우에만 표시 업데이트
                if (elementId === 'test-phonetic-display' && testWords[currentTestIndex] && testWords[currentTestIndex].word === wordItem.word) {
                    el.innerText = foundPhonetic;
                } else if (elementId === 'listen-phonetic-display' && listenWords[listenIndex] && listenWords[listenIndex].word === wordItem.word) {
                    el.innerText = foundPhonetic;
                }
            } catch (e) {
                // 네트워크 오류 시 빈칸 유지 (추후 재시도 가능하도록 undefined 유지)
            }
        }

        // ==========================================
        // 백업 및 복원
        // ==========================================
        function exportData() {
            if (Object.keys(vocabData).length === 0) {
                alert('내보낼 단어장 데이터가 없습니다.');
                return;
            }
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(vocabData, null, 2));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", `단어장_백업_${new Date().toISOString().slice(0,10)}.json`);
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        }

        function importData(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const importedData = JSON.parse(e.target.result);
                    if (typeof importedData === 'object' && importedData !== null) {
                        if (confirm('기존 데이터에 덮어쓰시겠습니까?')) {
                            vocabData = importedData;
                            consolidateOverlappingEnglishWords();
                            saveData();
                            renderFileList();
                            alert('성공적으로 불러왔습니다.');
                        }
                    } else {
                        alert('올바르지 않은 단어장 파일 형식입니다.');
                    }
                } catch (err) {
                    alert('파일을 읽는 중 오류가 발생했습니다.');
                }
            };
            reader.readAsText(file);
            event.target.value = '';
        }

        // ==========================================
        // 단어장 목록 기능
        // ==========================================
        function renderFileList() {
            const listEl = document.getElementById('file-list');
            listEl.innerHTML = '';
            const files = Object.keys(vocabData);
            
            if (files.length === 0) {
                listEl.innerHTML = '<div class="empty-message">등록된 단어장이 없습니다.<br>오른쪽 아래 + 버튼을 눌러 생성해보세요!</div>';
                return;
            }

            files.forEach(fileName => {
                const li = document.createElement('li');
                li.className = 'list-item draggable-item';
                
                const isAutoFile = (fileName === SPECIAL_FILE_NAME || fileName === SAME_MEANING_FILE_NAME);
                const isOverlapFile = (fileName === OVERLAP_WORD_FILE_NAME);
                const isBlueFile = (fileName === BLUE_FILE_NAME);
                
                if (isAutoFile || isOverlapFile) li.classList.add('special-file-item');
                else if (isBlueFile) li.classList.add('blue-file-item');
                
                li.draggable = true;
                li.dataset.name = fileName;

                const autoBadge = isAutoFile
                    ? `<span class="badge-auto">자동 생성</span>`
                    : isOverlapFile
                        ? `<span class="badge-auto">중복 통합</span>`
                        : '';

                const actionButtonsHtml = isAutoFile
                    ? `<button class="btn-sub" onclick="refreshAutoFile(event, '${escapeHtml(fileName)}')">🔄 새로고침</button>`
                    : isOverlapFile
                        ? `<button class="btn-sub" onclick="refreshAutoFile(event, '${escapeHtml(fileName)}')">🔄 새로고침</button>
                           <button class="btn-delete" onclick="deleteFile(event, '${escapeHtml(fileName)}')">삭제</button>`
                        : isBlueFile
                            ? ''
                            : `<button class="btn-edit" onclick="renameFile(event, '${escapeHtml(fileName)}')">이름 수정</button>
                               <button class="btn-delete" onclick="deleteFile(event, '${escapeHtml(fileName)}')">삭제</button>`;
                
                li.innerHTML = `
                    <div class="word-text">
                        <span style="color:#aaa; margin-right:8px; font-size:14px;">☰</span> 
                        ${escapeHtml(fileName)} 
                        <span style="font-size:13px; color:#888; font-weight:normal;">(${vocabData[fileName].length}개)</span>
                        ${autoBadge}
                    </div>
                    <div class="action-buttons">
                        ${actionButtonsHtml}
                    </div>
                `;
                
                li.onclick = (e) => {
                    if(e.target.tagName !== 'BUTTON') openFile(fileName);
                };

                li.addEventListener('dragstart', () => li.classList.add('dragging'));
                li.addEventListener('dragend', () => li.classList.remove('dragging'));

                listEl.appendChild(li);
            });

            listEl.addEventListener('dragover', e => {
                e.preventDefault();
                const afterElement = getDragAfterElement(listEl, e.clientY);
                const dragging = document.querySelector('.dragging');
                if (!dragging || !listEl.contains(dragging)) return;
                
                if (afterElement == null) {
                    listEl.appendChild(dragging);
                } else {
                    listEl.insertBefore(dragging, afterElement);
                }
            });

            listEl.addEventListener('drop', () => {
                const dragging = document.querySelector('.dragging');
                if (!dragging || !listEl.contains(dragging)) return;
                
                const newVocabData = {};
                document.querySelectorAll('#file-list .list-item').forEach(li => {
                    const name = li.dataset.name;
                    newVocabData[name] = vocabData[name];
                });
                vocabData = newVocabData;
                saveData();
            });
        }

        function getDragAfterElement(container, y) {
            const draggableElements = [...container.querySelectorAll('.draggable-item:not(.dragging)')];
            return draggableElements.reduce((closest, child) => {
                const box = child.getBoundingClientRect();
                const offset = y - box.top - box.height / 2;
                if (offset < 0 && offset > closest.offset) {
                    return { offset: offset, element: child };
                } else {
                    return closest;
                }
            }, { offset: Number.NEGATIVE_INFINITY }).element;
        }

        function createNewFile() {
            const fileName = prompt('새로운 단어장의 이름을 입력하세요:');
            if (fileName !== null) {
                const trimmedName = fileName.trim();
                if (!trimmedName) {
                    alert('단어장 이름을 입력해주세요.');
                    return;
                }
                if (vocabData[trimmedName]) {
                    alert('이미 존재하는 단어장 이름입니다.');
                    return;
                }
                vocabData[trimmedName] = [];
                saveData();
                openFile(trimmedName);
            }
        }

        function deleteFile(event, fileName) {
            event.stopPropagation();
            if (confirm(`'${fileName}' 단어장을 정말 삭제하시겠습니까?`)) {
                delete vocabData[fileName];
                delete wrongStudyCounts[fileName];
                delete wrongStudyProgress[fileName];
                delete savedSessions[fileName];
                saveData();
                saveWrongStudy();
                saveWrongStudyProgress();
                saveSessionsToStorage();
                renderFileList();
            }
        }

        function refreshAutoFile(event, fileName) {
            event.stopPropagation();

            if (fileName === OVERLAP_WORD_FILE_NAME) {
                consolidateOverlappingEnglishWords();
                saveData();
                renderFileList();
                if (currentFile === fileName) {
                    renderWordList();
                }
                alert(`'${fileName}' 파일을 최신 상태로 새로고침했습니다.`);
                return;
            }

            vocabData[fileName] = [];

            if (fileName === SPECIAL_FILE_NAME) {
                const allWords = getAllWordsAcrossFiles();
                allWords.forEach(item => {
                    handleConfusingWords(item, allWords);
                });
            } else {
                Object.keys(vocabData).forEach(otherFileName => {
                    if (otherFileName === SPECIAL_FILE_NAME || otherFileName === SAME_MEANING_FILE_NAME) return;
                    const words = (vocabData[otherFileName] || []).filter(item => !item.isDivider);
                    words.forEach(item => {
                        if (fileName === SAME_MEANING_FILE_NAME) {
                            handleSameMeaningWords(item, words);
                        }
                    });
                });
            }

            saveData();
            renderFileList();
            if (currentFile === fileName) {
                renderWordList();
            }
            alert(`'${fileName}' 파일을 최신 상태로 새로고침했습니다.`);
        }

        function renameFile(event, fileName) {
            event.stopPropagation();

            if (fileName === SPECIAL_FILE_NAME || fileName === SAME_MEANING_FILE_NAME || fileName === OVERLAP_WORD_FILE_NAME) {
                alert('자동 생성된 파일은 이름을 수정할 수 없습니다.');
                return;
            }

            const newName = prompt('새로운 단어장 이름을 입력하세요:', fileName);
            if (newName === null) return;

            const trimmedName = newName.trim();
            if (!trimmedName) {
                alert('단어장 이름을 입력해주세요.');
                return;
            }
            if (trimmedName === fileName) return;
            if (vocabData[trimmedName]) {
                alert('이미 존재하는 단어장 이름입니다.');
                return;
            }

            const newVocabData = {};
            Object.keys(vocabData).forEach(key => {
                if (key === fileName) {
                    newVocabData[trimmedName] = vocabData[key];
                } else {
                    newVocabData[key] = vocabData[key];
                }
            });
            vocabData = newVocabData;

            if (wrongStudyCounts[fileName] !== undefined) {
                wrongStudyCounts[trimmedName] = wrongStudyCounts[fileName];
                delete wrongStudyCounts[fileName];
                saveWrongStudy();
            }
            if (wrongStudyProgress[fileName] !== undefined) {
                wrongStudyProgress[trimmedName] = wrongStudyProgress[fileName];
                delete wrongStudyProgress[fileName];
                saveWrongStudyProgress();
            }
            if (savedSessions[fileName] !== undefined) {
                savedSessions[trimmedName] = savedSessions[fileName];
                delete savedSessions[fileName];
                saveSessionsToStorage();
            }

            if (currentFile === fileName) {
                currentFile = trimmedName;
                document.getElementById('current-file-name').innerText = trimmedName;
            }

            saveData();
            renderFileList();
        }

        function openFile(fileName) {
            currentFile = fileName;
            document.getElementById('current-file-name').innerText = fileName;
            navigate('file');
        }

        // ==========================================
        // 단어 상세, 추가, 수정, 이동, 오답버튼 상태 업데이트
        // ==========================================
        function updateWrongTestButton() {
            const wrongWords = (vocabData[currentFile] || []).filter(w => w.isWrong);
            const btn = document.getElementById('btn-wrong-test-dynamic');
            
            if (wrongWords.length === 0) {
                btn.style.display = 'none';
                if (wrongStudyProgress[currentFile]) {
                    wrongStudyProgress[currentFile] = 0;
                    saveWrongStudyProgress();
                }
                clearCurrentSessionState(currentFile, 'study');
                clearCurrentSessionState(currentFile, 'wrongTest');
                return;
            }
            
            btn.style.display = 'block';
            let count = wrongStudyCounts[currentFile] || 0;

            if (count < 2) {
                const studySession = savedSessions[currentFile] && savedSessions[currentFile].study;
                if (studySession && Array.isArray(studySession.testWords) && studySession.testWords.length > 0 && studySession.currentTestIndex > 0) {
                    btn.innerText = `🔁 오답 학습하기 (${count}/2 · ${studySession.currentTestIndex + 1}/${studySession.testWords.length} 학습중)`;
                } else {
                    btn.innerText = `🔁 오답 학습하기 (${count}/2)`;
                }
            } else {
                const wrongTestSession = savedSessions[currentFile] && savedSessions[currentFile].wrongTest;
                const hasWrongProgress = wrongTestSession && Array.isArray(wrongTestSession.testWords) && wrongTestSession.testWords.length > 0 &&
                    (wrongTestSession.currentTestIndex > 0 || Object.keys(wrongTestSession.answers || {}).length > 0);
                if (hasWrongProgress) {
                    btn.innerText = `🔁 오답만 다시 테스트 (${wrongTestSession.currentTestIndex + 1}/${wrongTestSession.testWords.length} 진행중)`;
                } else {
                    btn.innerText = `🔁 오답만 다시 테스트`;
                }
            }
        }

        function startTestFromWord(itemIndex) {
            const words = vocabData[currentFile] || [];
            const targetItem = words[itemIndex];
            if (!targetItem || targetItem.isDivider) return;

            const realWords = [];
            let targetRealIndex = 0;
            words.forEach((w, idx) => {
                if (!w.isDivider) {
                    if (idx === itemIndex) {
                        targetRealIndex = realWords.length;
                    }
                    realWords.push(w);
                }
            });

            if (realWords.length === 0) return;

            isStudyMode = false;
            isWrongOnlyTest = false;
            currentDividerIndex = null;
            beginTestWithWords(realWords, false, targetRealIndex);
        }

        function renderWordList() {
            const listEl = document.getElementById('word-list');
            listEl.innerHTML = '';
            const words = vocabData[currentFile] || [];
            
            updateWrongTestButton();

            if (words.length === 0) {
                listEl.innerHTML = '<div class="empty-message">등록된 단어가 없습니다.<br>아래에서 새 단어를 추가해보세요.</div>';
                return;
            }

            const isSpecialFile = (currentFile === SPECIAL_FILE_NAME || currentFile === SAME_MEANING_FILE_NAME);

            words.forEach((item, index) => {
                const li = document.createElement('li');

                if (item.isDivider) {
                    li.className = 'list-item user-divider-item draggable-item';
                    li.draggable = true;
                    li.dataset.index = index;
                    li.innerHTML = `
                        <div class="divider-title">🚩 ${escapeHtml(item.title)}</div>
                        <div class="action-buttons" style="display: flex; gap: 4px;">
                            <button style="background-color: var(--secondary-color); color: white;" onclick="testDividerSegment(${index})">테스트</button>
                            <button class="btn-edit" onclick="editDividerTitle(${index})">이름 수정</button>
                            <button class="btn-delete" onclick="deleteDivider(${index})">삭제</button>
                        </div>
                    `;
                    li.addEventListener('dragstart', () => li.classList.add('dragging'));
                    li.addEventListener('dragend', () => li.classList.remove('dragging'));
                    listEl.appendChild(li);
                    return;
                }

                li.className = 'list-item draggable-item';
                li.draggable = true;
                li.dataset.index = index;
                li.dataset.word = item.word.toLowerCase();
                li.dataset.meaning = item.meaning.toLowerCase();

                if (isSpecialFile && index > 0 && !words[index - 1].isDivider && words[index - 1].groupId !== item.groupId) {
                    li.classList.add('group-divider');
                }
                
                const wrongBadge = item.isWrong ? `<span class="badge-wrong">오답</span>` : '';
                
                li.innerHTML = `
                    <div class="word-content">
                        <div class="word-title-group">
                            <div class="word-text"><span style="color:#aaa; margin-right:8px; font-size:14px; cursor:grab;">☰</span>${escapeHtml(item.word)} ${wrongBadge}</div>
                            <button class="btn-tts" onclick="speakWord('${escapeHtml(item.word)}')">🔊</button>
                        </div>
                        <div class="meaning-text">${escapeHtml(item.meaning)}</div>
                    </div>
                    <div class="action-buttons" style="display: flex; gap: 4px;">
                        <button class="btn-edit" onclick="editWord(${index})">수정</button>
                        <button class="btn-delete" onclick="deleteWord(${index})">삭제</button>
                    </div>
                `;
                li.onclick = (e) => {
                    if (e.target.closest('button')) return;
                    startTestFromWord(index);
                };
                li.addEventListener('dragstart', () => li.classList.add('dragging'));
                li.addEventListener('dragend', () => li.classList.remove('dragging'));
                listEl.appendChild(li);
            });

            if (!listEl.dataset.dragInit) {
                listEl.addEventListener('dragover', e => {
                    e.preventDefault();
                    const dragging = document.querySelector('.dragging');
                    if (!dragging || !listEl.contains(dragging)) return;
                    
                    const afterElement = getDragAfterElement(listEl, e.clientY);
                    if (afterElement == null) {
                        listEl.appendChild(dragging);
                    } else {
                        listEl.insertBefore(dragging, afterElement);
                    }
                });

                listEl.addEventListener('drop', () => {
                    const dragging = document.querySelector('.dragging');
                    if (!dragging || !listEl.contains(dragging)) return;

                    const currentList = [...vocabData[currentFile]];
                    const newWordList = [];
                    document.querySelectorAll('#word-list .list-item').forEach(li => {
                        const originalIndex = li.dataset.index;
                        if (originalIndex !== undefined) {
                            newWordList.push(currentList[originalIndex]);
                        }
                    });
                    vocabData[currentFile] = newWordList;
                    saveData();
                    renderWordList();
                });
                listEl.dataset.dragInit = 'true';
            }
        }

        function addDividerAtEnd() {
            if (currentFile === SPECIAL_FILE_NAME || currentFile === SAME_MEANING_FILE_NAME) {
                alert('자동 생성된 파일에는 경계선을 추가할 수 없습니다.');
                return;
            }
            const title = prompt('경계선 이름을 입력하세요:', '');
            if (title === null) return;
            const dividerTitle = title.trim() || '구분선';

            if (!vocabData[currentFile]) {
                vocabData[currentFile] = [];
            }
            vocabData[currentFile].push({
                isDivider: true,
                title: dividerTitle
            });
            saveData();
            renderWordList();
        }

        function editDividerTitle(index) {
            const item = vocabData[currentFile][index];
            if (!item || !item.isDivider) return;

            const newTitle = prompt('경계선 이름을 수정하세요:', item.title);
            if (newTitle === null) return;
            const trimmed = newTitle.trim();
            if (!trimmed) {
                alert('경계선 이름을 입력해주세요.');
                return;
            }
            item.title = trimmed;
            saveData();
            renderWordList();
        }

        function deleteDivider(index) {
            if (confirm('이 경계선을 삭제하시겠습니까? (단어는 삭제되지 않습니다)')) {
                vocabData[currentFile].splice(index, 1);
                saveData();
                renderWordList();
            }
        }

        function testDividerSegment(index) {
            isStudyMode = false;
            isWrongOnlyTest = false;

            const savedNormal = savedSessions[currentFile] && savedSessions[currentFile].normal;
            if (savedNormal && savedNormal.dividerIndex === index && Array.isArray(savedNormal.testWords) && savedNormal.testWords.length > 0 && savedNormal.currentTestIndex < savedNormal.testWords.length) {
                currentDividerIndex = index;
                resumeSavedSession(savedNormal);
                return;
            }

            const words = vocabData[currentFile];
            let segmentWords = [];
            for (let i = index + 1; i < words.length; i++) {
                if (words[i].isDivider) break;
                segmentWords.push(words[i]);
            }
            
            if (segmentWords.length === 0) {
                alert('해당 경계선 아래에 테스트할 단어가 없습니다.');
                return;
            }
            currentDividerIndex = index;
            beginTestWithWords(pickTestPool(segmentWords));
        }

        function getWordSegments(words) {
            const segments = [];
            let current = { title: '기본 구간', items: [] };

            words.forEach(item => {
                if (item.isDivider) {
                    segments.push(current);
                    current = { title: item.title, items: [] };
                } else {
                    current.items.push(item);
                }
            });
            segments.push(current);
            return segments;
        }

        function getBaseForm(text) {
            const tokens = text.toLowerCase().trim().split(/\s+/).filter(Boolean);
            if (tokens.length !== 2) return null;
            const [verb, particle] = tokens;
            if (!PREPOSITIONS.has(particle)) return null;
            return verb;
        }

        function getAllWordsAcrossFiles() {
            let allWords = [];
            Object.keys(vocabData).forEach(fileName => {
                if (fileName === SPECIAL_FILE_NAME || fileName === SAME_MEANING_FILE_NAME) return;
                const words = (vocabData[fileName] || []).filter(item => !item.isDivider);
                allWords = allWords.concat(words);
            });
            return allWords;
        }

        // ==========================================
        // 겹치는 영단어 통합 및 파일 이동 로직
        // ==========================================
        function mergeMeaningsText(m1, m2) {
            const list1 = (m1 || '').split(/,\s*/).map(s => s.trim()).filter(Boolean);
            const list2 = (m2 || '').split(/,\s*/).map(s => s.trim()).filter(Boolean);
            const combined = [...list1];
            list2.forEach(item => {
                if (!combined.some(existing => existing.toLowerCase() === item.toLowerCase())) {
                    combined.push(item);
                }
            });
            return combined.join(', ');
        }

        function mergeExamplesList(ex1, ex2) {
            const arr1 = Array.isArray(ex1) ? ex1 : [];
            const arr2 = Array.isArray(ex2) ? ex2 : [];
            const combined = [...arr1];
            arr2.forEach(e => {
                if (!e || (!e.en && !e.ko)) return;
                const exists = combined.some(
                    c => c.en === e.en && c.ko === e.ko && (c.meaningTag || '') === (e.meaningTag || '')
                );
                if (!exists) combined.push(e);
            });
            return combined;
        }

        function mergeAndMoveWordToOverlapFile(word, newMeaning) {
            const excludedFiles = [SPECIAL_FILE_NAME, SAME_MEANING_FILE_NAME];
            const normWord = (word || '').toLowerCase().trim();
            const inputWords = normWord.split(/,\s*/).map(w => w.trim()).filter(Boolean);

            if (!vocabData[OVERLAP_WORD_FILE_NAME]) {
                vocabData[OVERLAP_WORD_FILE_NAME] = [];
            }

            let canonicalWord = word.trim();
            let mergedMeaning = '';
            let mergedIsWrong = false;
            let mergedPhonetic = undefined;
            let mergedExamples = [];
            let existingOverlapIndex = -1;
            let oldOverlapItemSnapshot = null;

            const toRemove = [];

            Object.keys(vocabData).forEach(fileName => {
                if (excludedFiles.includes(fileName)) return;
                const list = vocabData[fileName] || [];
                list.forEach((item, idx) => {
                    if (item.isDivider) return;
                    const itemWordNorm = (item.word || '').toLowerCase().trim();
                    const itemWords = itemWordNorm.split(/,\s*/).map(w => w.trim()).filter(Boolean);
                    const isWordDup = (normWord && itemWordNorm === normWord) ||
                                      inputWords.some(w => itemWords.includes(w));

                    if (isWordDup) {
                        if (canonicalWord.toLowerCase() === itemWordNorm) {
                            canonicalWord = item.word.trim();
                        } else if (item.word && item.word.trim()) {
                            canonicalWord = mergeMeaningsText(canonicalWord, item.word.trim());
                        }
                        mergedMeaning = mergeMeaningsText(mergedMeaning, item.meaning);
                        if (item.isWrong) mergedIsWrong = true;
                        if (mergedPhonetic === undefined && item.phonetic !== undefined) {
                            mergedPhonetic = item.phonetic;
                        }
                        mergedExamples = mergeExamplesList(mergedExamples, item.examples);

                        if (fileName === OVERLAP_WORD_FILE_NAME && existingOverlapIndex === -1) {
                            existingOverlapIndex = idx;
                            oldOverlapItemSnapshot = { ...item };
                        } else {
                            toRemove.push({ fileName, index: idx, item });
                        }
                    }
                });
            });

            if (newMeaning && newMeaning.trim()) {
                mergedMeaning = mergeMeaningsText(mergedMeaning, newMeaning.trim());
            }

            toRemove.sort((a, b) => b.index - a.index).forEach(({ fileName, index, item }) => {
                vocabData[fileName].splice(index, 1);
                syncDeleteWordInSessions(fileName, item, index);
            });

            const mergedItem = {
                word: canonicalWord,
                meaning: mergedMeaning,
                isWrong: mergedIsWrong,
                phonetic: mergedPhonetic,
                examples: mergedExamples
            };

            if (existingOverlapIndex !== -1 && vocabData[OVERLAP_WORD_FILE_NAME][existingOverlapIndex]) {
                vocabData[OVERLAP_WORD_FILE_NAME][existingOverlapIndex] = mergedItem;
                if (oldOverlapItemSnapshot) {
                    syncEditWordInSessions(OVERLAP_WORD_FILE_NAME, oldOverlapItemSnapshot, mergedItem);
                }
            } else {
                vocabData[OVERLAP_WORD_FILE_NAME].push(mergedItem);
            }

            return mergedItem;
        }

        function consolidateOverlappingEnglishWords() {
            const excludedFiles = [SPECIAL_FILE_NAME, SAME_MEANING_FILE_NAME];
            const wordMap = new Map();

            Object.keys(vocabData).forEach(fileName => {
                if (excludedFiles.includes(fileName)) return;
                (vocabData[fileName] || []).forEach(item => {
                    if (item.isDivider) return;
                    const normWord = (item.word || '').toLowerCase().trim();
                    if (!normWord) return;
                    if (!wordMap.has(normWord)) {
                        wordMap.set(normWord, []);
                    }
                    wordMap.get(normWord).push({ fileName, item });
                });
            });

            let changed = false;
            wordMap.forEach((occurrences) => {
                if (occurrences.length >= 2) {
                    const mergedItem = mergeAndMoveWordToOverlapFile(occurrences[0].item.word, '');
                    if (mergedItem) {
                        handleConfusingWords(mergedItem, getAllWordsAcrossFiles());
                        handleSameMeaningWords(mergedItem, vocabData[OVERLAP_WORD_FILE_NAME]);
                        changed = true;
                    }
                }
            });

            if (changed) {
                saveData();
            }
            return changed;
        }

        function findOverlapsAcrossAllFiles(word, meaning) {
            const excludedFiles = [SPECIAL_FILE_NAME, SAME_MEANING_FILE_NAME];
            const normWord = (word || '').toLowerCase().trim();
            const inputWords = normWord.split(/,\s*/).map(w => w.trim()).filter(Boolean);
            const inputMeanings = (meaning || '').toLowerCase().split(/,\s*/).map(m => m.trim()).filter(Boolean);

            const overlaps = [];

            Object.keys(vocabData).forEach(fileName => {
                if (excludedFiles.includes(fileName)) return;
                (vocabData[fileName] || []).forEach(item => {
                    if (item.isDivider) return;
                    const itemWordNorm = (item.word || '').toLowerCase().trim();
                    const itemWords = itemWordNorm.split(/,\s*/).map(w => w.trim()).filter(Boolean);
                    const itemMeanings = (item.meaning || '').toLowerCase().split(/,\s*/).map(m => m.trim()).filter(Boolean);

                    const isWordDup = (normWord && itemWordNorm === normWord) ||
                                      inputWords.some(w => itemWords.includes(w));
                    const isMeaningDup = inputMeanings.length > 0 &&
                                         inputMeanings.some(m => itemMeanings.includes(m));

                    if (isWordDup || isMeaningDup) {
                        let typeText = '';
                        if (isWordDup && isMeaningDup) typeText = '영어·뜻 겹침';
                        else if (isWordDup) typeText = '영어 겹침';
                        else typeText = '뜻 겹침';

                        overlaps.push({
                            fileName,
                            word: item.word,
                            meaning: item.meaning,
                            typeText,
                            isWordDup,
                            isMeaningDup
                        });
                    }
                });
            });

            return overlaps;
        }

        function findSimilarWords(words, newWord) {
            const newBase = getBaseForm(newWord);
            if (!newBase) return [];
            const normalizedNew = newWord.toLowerCase().trim();
            return words.filter(w =>
                w.word.toLowerCase().trim() !== normalizedNew &&
                getBaseForm(w.word) === newBase
            );
        }

        function reorderSpecialListByGroup() {
            const specialList = vocabData[SPECIAL_FILE_NAME];
            if (!specialList) return;
            const groupOrder = [];
            const seen = new Set();
            specialList.forEach(item => {
                const gid = item.groupId || 'ungrouped';
                if (!seen.has(gid)) {
                    seen.add(gid);
                    groupOrder.push(gid);
                }
            });
            const sorted = [];
            groupOrder.forEach(gid => {
                specialList.filter(item => (item.groupId || 'ungrouped') === gid)
                    .forEach(item => sorted.push(item));
            });
            vocabData[SPECIAL_FILE_NAME] = sorted;
        }

        function handleConfusingWords(newItem, sourceList) {
            const similarItems = findSimilarWords(sourceList, newItem.word);
            if (similarItems.length === 0) return;

            if (!vocabData[SPECIAL_FILE_NAME]) {
                vocabData[SPECIAL_FILE_NAME] = [];
            }
            const specialList = vocabData[SPECIAL_FILE_NAME];
            const newBase = getBaseForm(newItem.word);

            similarItems.forEach(similarItem => {
                let groupId;
                const existingGroupEntry = specialList.find(w => getBaseForm(w.word) === newBase);
                if (existingGroupEntry && existingGroupEntry.groupId) {
                    groupId = existingGroupEntry.groupId;
                } else {
                    groupId = 'group-' + Date.now() + '-' + Math.floor(Math.random() * 1000);
                }

                const existingIdx = specialList.findIndex(w => w.word === similarItem.word && w.meaning === similarItem.meaning);
                if (existingIdx === -1) {
                    specialList.push({ word: similarItem.word, meaning: similarItem.meaning, isWrong: false, groupId, examples: similarItem.examples });
                } else {
                    specialList[existingIdx].groupId = groupId;
                }

                const newAlreadyIn = specialList.some(w => w.word === newItem.word && w.meaning === newItem.meaning);
                if (!newAlreadyIn) {
                    specialList.push({ word: newItem.word, meaning: newItem.meaning, isWrong: false, groupId, examples: newItem.examples });
                }
            });

            reorderSpecialListByGroup();
        }

        function findSameMeaningWords(words, newMeaning) {
            const normalizedMeaning = newMeaning.toLowerCase().trim();
            return words.filter(w =>
                !w.isDivider && w.meaning.toLowerCase().trim() === normalizedMeaning
            );
        }

        function reorderSameMeaningListByGroup() {
            const specialList = vocabData[SAME_MEANING_FILE_NAME];
            if (!specialList) return;
            const groupOrder = [];
            const seen = new Set();
            specialList.forEach(item => {
                const gid = item.groupId || 'ungrouped';
                if (!seen.has(gid)) {
                    seen.add(gid);
                    groupOrder.push(gid);
                }
            });
            const sorted = [];
            groupOrder.forEach(gid => {
                specialList.filter(item => (item.groupId || 'ungrouped') === gid)
                    .forEach(item => sorted.push(item));
            });
            vocabData[SAME_MEANING_FILE_NAME] = sorted;
        }

        function handleSameMeaningWords(newItem, sourceList) {
            const normalizedMeaning = newItem.meaning.toLowerCase().trim();
            const sameMeaningItems = findSameMeaningWords(sourceList, newItem.meaning)
                .filter(item => item.word.toLowerCase().trim() !== newItem.word.toLowerCase().trim());
            if (sameMeaningItems.length === 0) return;

            if (!vocabData[SAME_MEANING_FILE_NAME]) {
                vocabData[SAME_MEANING_FILE_NAME] = [];
            }
            const specialList = vocabData[SAME_MEANING_FILE_NAME];

            sameMeaningItems.forEach(sameItem => {
                let groupId;
                const existingGroupEntry = specialList.find(w => w.meaning.toLowerCase().trim() === normalizedMeaning);
                if (existingGroupEntry && existingGroupEntry.groupId) {
                    groupId = existingGroupEntry.groupId;
                } else {
                    groupId = 'meaning-group-' + Date.now() + '-' + Math.floor(Math.random() * 1000);
                }

                const existingIdx = specialList.findIndex(w => w.word === sameItem.word && w.meaning === sameItem.meaning);
                if (existingIdx === -1) {
                    specialList.push({ word: sameItem.word, meaning: sameItem.meaning, isWrong: false, groupId, examples: sameItem.examples });
                } else {
                    specialList[existingIdx].groupId = groupId;
                }

                const newAlreadyIn = specialList.some(w => w.word === newItem.word && w.meaning === newItem.meaning);
                if (!newAlreadyIn) {
                    specialList.push({ word: newItem.word, meaning: newItem.meaning, isWrong: false, groupId, examples: newItem.examples });
                }
            });

            reorderSameMeaningListByGroup();
        }

        function addWord() {
            const wordInput = document.getElementById('input-word');
            const meaningInput = document.getElementById('input-meaning');
            const word = wordInput.value.trim();
            const meaning = meaningInput.value.trim();

            if (!word || !meaning) {
                alert('단어와 뜻을 모두 입력해주세요.');
                return;
            }

            const overlaps = findOverlapsAcrossAllFiles(word, meaning);
            const hasWordOverlap = overlaps.some(o => o.isWordDup);

            if (overlaps.length > 0) {
                const overlapMsg = overlaps.map(o => `• [${o.fileName}] ${o.word} : ${o.meaning} (${o.typeText})`).join('\n');
                if (hasWordOverlap) {
                    alert(`⚠️ 전체 단어장 범위에서 겹치는 영단어가 있습니다!\n기존 단어와 추가할 단어를 합쳐 '${OVERLAP_WORD_FILE_NAME}' 파일로 이동합니다.\n\n${overlapMsg}`);
                } else {
                    alert(`⚠️ 전체 단어장 범위에서 겹치는 뜻이 있습니다!\n\n${overlapMsg}`);
                }
            }

            if (hasWordOverlap) {
                const mergedItem = mergeAndMoveWordToOverlapFile(word, meaning);
                handleConfusingWords(mergedItem, getAllWordsAcrossFiles());
                handleSameMeaningWords(mergedItem, vocabData[OVERLAP_WORD_FILE_NAME]);
                saveData();

                wordInput.value = '';
                meaningInput.value = '';

                renderWordList();
                scrollToAddForm();
                wordInput.focus();
                return;
            }

            const list = vocabData[currentFile];
            const newItem = { word, meaning, isWrong: false };
            list.push(newItem);

            handleConfusingWords(newItem, getAllWordsAcrossFiles());
            handleSameMeaningWords(newItem, list);

            saveData();
            
            wordInput.value = '';
            meaningInput.value = '';
            
            renderWordList();
            scrollToAddForm();
            wordInput.focus();
        }

        function handleEnter(event) {
            if (event.key === 'Enter') {
                addWord();
            }
        }

        function openAddChoiceModal() {
            document.getElementById('add-choice-overlay').style.display = 'flex';
        }

        function closeAddChoiceModal() {
            document.getElementById('add-choice-overlay').style.display = 'none';
        }

        function confirmSingleAdd() {
            closeAddChoiceModal();
            addWord();
        }

        function openBulkAddModal() {
            closeAddChoiceModal();
            document.getElementById('bulk-add-textarea').value = '';
            document.getElementById('bulk-add-overlay').style.display = 'flex';
            setTimeout(() => document.getElementById('bulk-add-textarea').focus(), 100);
        }

        function closeBulkAddModal() {
            document.getElementById('bulk-add-overlay').style.display = 'none';
        }

        function isEnglishSegment(text) {
            return /^[a-zA-Z][a-zA-Z\s'\-\.]*$/.test(text);
        }

        function submitBulkAdd() {
            const textarea = document.getElementById('bulk-add-textarea');
            const rawLines = textarea.value.split('\n');

            const meaningOptional = (currentFile === BLUE_FILE_NAME);
            const parsedItems = [];
            const invalidLines = [];

            rawLines.forEach((rawLine, idx) => {
                const line = rawLine.trim();
                if (!line) return;

                const segments = line.split(/,|\t/).map(s => s.trim()).filter(s => s.length > 0);
                if (segments.length < 2 && !meaningOptional) {
                    invalidLines.push({ lineNumber: idx + 1, content: rawLine });
                    return;
                }
                if (segments.length === 0) {
                    invalidLines.push({ lineNumber: idx + 1, content: rawLine });
                    return;
                }

                const wordParts = [];
                const meaningParts = [];
                segments.forEach(seg => {
                    if (isEnglishSegment(seg)) {
                        wordParts.push(seg);
                    } else {
                        meaningParts.push(seg);
                    }
                });

                const word = wordParts.join(', ');
                const meaning = meaningParts.join(', ');

                if (!word || (!meaning && !meaningOptional)) {
                    invalidLines.push({ lineNumber: idx + 1, content: rawLine });
                    return;
                }

                parsedItems.push({ word, meaning });
            });

            if (parsedItems.length === 0 && invalidLines.length === 0) {
                alert('추가할 단어를 입력해주세요.');
                return;
            }

            if (invalidLines.length > 0) {
                const lineList = invalidLines.map(l => `${l.lineNumber}번째 줄: "${l.content}"`).join('\n');
                alert(`형식이 올바르지 않은 줄이 있어 추가하지 않았습니다.\n아래 줄을 "단어, 뜻" 형식에 맞게 직접 수정한 뒤 다시 시도해주세요.\n\n${lineList}`);
                textarea.focus();
                return;
            }

            const list = vocabData[currentFile];
            let addedCount = 0;
            let movedCount = 0;
            const bulkOverlapMessages = [];

            parsedItems.forEach(({ word, meaning }) => {
                const overlaps = findOverlapsAcrossAllFiles(word, meaning);
                const hasWordOverlap = overlaps.some(o => o.isWordDup);

                if (overlaps.length > 0) {
                    const detail = overlaps.map(o => `[${o.fileName}] ${o.word}:${o.meaning}(${o.typeText})`).join(', ');
                    bulkOverlapMessages.push(`• ${word} (${meaning}) ➔ ${detail}`);
                }

                if (hasWordOverlap) {
                    const mergedItem = mergeAndMoveWordToOverlapFile(word, meaning);
                    handleConfusingWords(mergedItem, getAllWordsAcrossFiles());
                    handleSameMeaningWords(mergedItem, vocabData[OVERLAP_WORD_FILE_NAME]);
                    movedCount++;
                } else {
                    const newItem = { word, meaning, isWrong: false };
                    list.push(newItem);
                    handleConfusingWords(newItem, getAllWordsAcrossFiles());
                    handleSameMeaningWords(newItem, list);
                }
                addedCount++;
            });

            saveData();
            renderWordList();
            closeBulkAddModal();

            if (bulkOverlapMessages.length > 0) {
                const moveNotice = movedCount > 0 ? `\n(겹치는 영단어 ${movedCount}개는 기존 단어와 합쳐 '${OVERLAP_WORD_FILE_NAME}' 파일로 이동되었습니다.)` : '';
                alert(`⚠️ 전체 단어장 범위에서 겹치는 영어 또는 뜻이 있습니다!${moveNotice}\n\n${bulkOverlapMessages.join('\n')}`);
            }

            alert(`${addedCount}개의 단어를 추가했습니다.`);
        }

        function editWord(index) {
            const item = vocabData[currentFile][index];
            const oldItem = { ...item };
            const newWord = prompt('단어를 수정하세요:', item.word);
            if (newWord === null) return;
            
            const newMeaning = prompt('뜻을 수정하세요:', item.meaning);
            if (newMeaning === null) return;

            if (newWord.trim() && newMeaning.trim()) {
                const wordChanged = (newWord.trim() !== item.word);
                const updatedItem = { 
                    word: newWord.trim(), 
                    meaning: newMeaning.trim(),
                    isWrong: item.isWrong,
                    groupId: item.groupId,
                    testedThisRound: item.testedThisRound,
                    example: wordChanged ? undefined : item.example,
                    examples: wordChanged ? [] : item.examples,
                    phonetic: wordChanged ? undefined : item.phonetic
                };
                vocabData[currentFile][index] = updatedItem;
                syncEditWordInSessions(currentFile, oldItem, updatedItem);
                saveData();
                renderWordList();
            } else {
                alert('단어와 뜻을 비워둘 수 없습니다.');
            }
        }

        function deleteWord(index) {
            if (confirm('이 단어를 삭제하시겠습니까?')) {
                const deletedItem = vocabData[currentFile][index];
                vocabData[currentFile].splice(index, 1);
                if (deletedItem && !deletedItem.isDivider) {
                    syncDeleteWordInSessions(currentFile, deletedItem, index);
                }
                saveData();
                renderWordList();
            }
        }

        function scrollToAddForm() {
            const formContainer = document.getElementById('add-form-container');
            formContainer.scrollIntoView({ behavior: 'smooth', block: 'end' });
            setTimeout(() => {
                document.getElementById('input-word').focus();
            }, 300);
        }

        function toggleSearchBar() {
            const container = document.getElementById('search-bar-container');
            const isHidden = (container.style.display === 'none' || container.style.display === '');
            if (isHidden) {
                container.style.display = 'block';
                const input = document.getElementById('search-word-input');
                input.value = '';
                container.scrollIntoView({ behavior: 'smooth', block: 'center' });
                input.focus();
            } else {
                container.style.display = 'none';
                resetWordListDisplay();
            }
        }

        function resetWordListDisplay() {
            document.querySelectorAll('#word-list .list-item').forEach(li => {
                li.style.display = '';
            });
        }

        function searchWordInList() {
            const query = document.getElementById('search-word-input').value.trim().toLowerCase();

            if (!query) {
                resetWordListDisplay();
                return;
            }

            const items = document.querySelectorAll('#word-list .list-item');
            items.forEach(li => {
                const wordText = li.dataset.word || '';
                const meaningText = li.dataset.meaning || '';
                const isMatch = wordText.includes(query) || meaningText.includes(query);
                li.style.display = isMatch ? '' : 'none';
            });
        }

        function toggleHomeSearchBar() {
            const container = document.getElementById('home-search-bar-container');
            const isHidden = (container.style.display === 'none' || container.style.display === '');
            if (isHidden) {
                container.style.display = 'block';
                const input = document.getElementById('home-search-word-input');
                input.value = '';
                container.scrollIntoView({ behavior: 'smooth', block: 'center' });
                input.focus();
            } else {
                container.style.display = 'none';
                document.getElementById('home-search-results').style.display = 'none';
                document.getElementById('home-search-results').innerHTML = '';
                document.getElementById('file-list').style.display = '';
            }
        }

        function searchWordInAllFiles() {
            const query = document.getElementById('home-search-word-input').value.trim().toLowerCase();
            const resultsEl = document.getElementById('home-search-results');
            const fileListEl = document.getElementById('file-list');

            if (!query) {
                resultsEl.style.display = 'none';
                resultsEl.innerHTML = '';
                fileListEl.style.display = '';
                return;
            }

            fileListEl.style.display = 'none';
            resultsEl.style.display = 'block';

            const excludedFiles = [SPECIAL_FILE_NAME, SAME_MEANING_FILE_NAME];
            const matches = [];

            Object.keys(vocabData).forEach(fileName => {
                if (excludedFiles.includes(fileName)) return;
                (vocabData[fileName] || []).forEach(item => {
                    if (item.isDivider) return;
                    const wordText = (item.word || '').toLowerCase();
                    const meaningText = (item.meaning || '').toLowerCase();
                    if (wordText.includes(query) || meaningText.includes(query)) {
                        matches.push({ fileName, word: item.word, meaning: item.meaning });
                    }
                });
            });

            if (matches.length === 0) {
                resultsEl.innerHTML = '<div class="empty-message">일치하는 단어가 없습니다.</div>';
                return;
            }

            resultsEl.innerHTML = matches.map(m => `
                <li class="list-item" style="cursor: pointer;" data-goto-file="${escapeHtml(m.fileName)}" data-goto-word="${escapeHtml(m.word)}" data-goto-meaning="${escapeHtml(m.meaning)}">
                    <div class="word-content">
                        <div class="word-text">${escapeHtml(m.word)}</div>
                        <div class="meaning-text">${escapeHtml(m.meaning)}</div>
                    </div>
                    <div style="font-size: 12px; color: #888; white-space: nowrap; margin-left: 10px;">${escapeHtml(m.fileName)}</div>
                </li>
            `).join('');

            resultsEl.querySelectorAll('.list-item').forEach(li => {
                li.addEventListener('click', () => {
                    goToWordInFile(li.dataset.gotoFile, li.dataset.gotoWord, li.dataset.gotoMeaning);
                });
            });
        }

        function goToWordInFile(fileName, word, meaning) {
            openFile(fileName);

            const normalizedWord = (word || '').toLowerCase();
            const normalizedMeaning = (meaning || '').toLowerCase();

            setTimeout(() => {
                const targetLi = Array.from(document.querySelectorAll('#word-list .list-item')).find(li =>
                    li.dataset.word === normalizedWord && li.dataset.meaning === normalizedMeaning
                );
                if (targetLi) {
                    targetLi.scrollIntoView({ behavior: 'smooth', block: 'center' });
                    targetLi.classList.add('search-highlight');
                    setTimeout(() => targetLi.classList.remove('search-highlight'), 2000);
                }
            }, 100);
        }

        // ==========================================
        // 테스트 기능 & 채점 & 학습 시스템 & 예문 추가/관리
        // ==========================================
        function handleExitTestView() {
            cancelTestSpeech();
            saveCurrentSessionState();
            document.getElementById('matching-quiz-overlay').style.display = 'none';
            document.getElementById('example-modal-overlay').style.display = 'none';
            navigate('file');
        }

        function resumeSavedSession(sessionObj) {
            testWords = [...sessionObj.testWords];
            currentTestIndex = Math.min(Math.max(0, sessionObj.currentTestIndex || 0), testWords.length - 1);
            testMode = sessionObj.testMode || 'word';
            currentSessionAnswers = sessionObj.answers ? { ...sessionObj.answers } : {};
            correctCount = Object.values(currentSessionAnswers).filter(Boolean).length;

            document.getElementById('test-content').style.display = 'block';
            document.getElementById('test-result').style.display = 'none';
            document.getElementById('matching-quiz-overlay').style.display = 'none';
            document.getElementById('example-modal-overlay').style.display = 'none';

            saveCurrentSessionState();
            updateTestUI();
            navigate('test');
        }

        function startTest() {
            isStudyMode = false;
            isWrongOnlyTest = false;
            currentDividerIndex = null;

            const savedNormal = savedSessions[currentFile] && savedSessions[currentFile].normal;
            if (savedNormal && savedNormal.dividerIndex === null && Array.isArray(savedNormal.testWords) && savedNormal.testWords.length > 0 && savedNormal.currentTestIndex < savedNormal.testWords.length) {
                resumeSavedSession(savedNormal);
                return;
            }

            const words = vocabData[currentFile] || [];
            const hasDividers = words.some(item => item.isDivider);

            if (hasDividers) {
                openSegmentSelectModal();
            } else {
                const realWords = words.filter(item => !item.isDivider);
                beginTestWithWords(pickTestPool(realWords));
            }
        }

        function pickTestPool(words) {
            if (words.length === 0) return [];
            return [...words];
        }

        function openSegmentSelectModal() {
            const words = vocabData[currentFile] || [];
            currentTestSegments = getWordSegments(words).filter(seg => seg.items.length > 0);

            const listEl = document.getElementById('segment-checkbox-list');
            listEl.innerHTML = currentTestSegments.map((seg, idx) => `
                <label style="display: flex; align-items: center; gap: 8px; margin-bottom: 10px; font-size: 14px;">
                    <input type="checkbox" class="segment-checkbox" data-segment-index="${idx}" checked>
                    <span>${escapeHtml(seg.title)} (${seg.items.length}개)</span>
                </label>
            `).join('');

            document.getElementById('segment-select-overlay').style.display = 'flex';
        }

        function closeSegmentSelectModal() {
            document.getElementById('segment-select-overlay').style.display = 'none';
        }

        function confirmSegmentTest() {
            const checkboxes = document.querySelectorAll('.segment-checkbox');
            let selectedWords = [];
            checkboxes.forEach(cb => {
                if (cb.checked) {
                    const idx = parseInt(cb.dataset.segmentIndex, 10);
                    selectedWords = selectedWords.concat(currentTestSegments[idx].items);
                }
            });

            closeSegmentSelectModal();

            if (selectedWords.length === 0) {
                alert('선택된 구간에 단어가 없습니다. 다른 구간을 선택해주세요.');
                return;
            }

            currentDividerIndex = null;
            beginTestWithWords(pickTestPool(selectedWords));
        }

        function handleWrongTestButton() {
            let count = wrongStudyCounts[currentFile] || 0;
            if (count < 2) {
                startStudyMode();
            } else {
                startWrongOnlyTest();
            }
        }

        function startStudyMode() {
            isStudyMode = true;
            isWrongOnlyTest = false;
            currentDividerIndex = null;

            const savedStudy = savedSessions[currentFile] && savedSessions[currentFile].study;
            if (savedStudy && Array.isArray(savedStudy.testWords) && savedStudy.testWords.length > 0 && savedStudy.currentTestIndex < savedStudy.testWords.length) {
                resumeSavedSession(savedStudy);
                return;
            }

            const words = (vocabData[currentFile] || []).filter(item => item.isWrong);
            if (words.length === 0) return;

            let resumeIdx = wrongStudyProgress[currentFile] || 0;
            if (resumeIdx < 0 || resumeIdx >= words.length) {
                resumeIdx = 0;
                wrongStudyProgress[currentFile] = 0;
                saveWrongStudyProgress();
            }

            beginTestWithWords(words, false, resumeIdx);
        }

        function startWrongOnlyTest() {
            isStudyMode = false;
            isWrongOnlyTest = true;
            currentDividerIndex = null;

            const savedWrongTest = savedSessions[currentFile] && savedSessions[currentFile].wrongTest;
            if (savedWrongTest && Array.isArray(savedWrongTest.testWords) && savedWrongTest.testWords.length > 0 && savedWrongTest.currentTestIndex < savedWrongTest.testWords.length) {
                resumeSavedSession(savedWrongTest);
                return;
            }

            const words = (vocabData[currentFile] || []).filter(item => item.isWrong);
            if (words.length === 0) {
                alert('오답으로 표시된 단어가 없습니다.');
                return;
            }
            beginTestWithWords(words, true, 0);
        }

        function beginTestWithWords(words, doShuffle = true, startIndex = 0) {
            if (words.length === 0) {
                alert('진행할 단어가 없습니다. 먼저 단어를 추가해주세요.');
                return;
            }
            
            testWords = [...words];
            if (doShuffle) shuffleArray(testWords);
            
            currentTestIndex = startIndex;
            correctCount = 0;
            currentSessionAnswers = {};
            testMode = 'word';
            
            document.getElementById('test-content').style.display = 'block';
            document.getElementById('test-result').style.display = 'none';
            document.getElementById('matching-quiz-overlay').style.display = 'none';
            document.getElementById('example-modal-overlay').style.display = 'none';
            
            saveCurrentSessionState();
            updateTestUI();
            navigate('test');
        }

        function toggleTestMode() {
            testMode = (testMode === 'word') ? 'meaning' : 'word';
            saveCurrentSessionState();
            updateTestUI();
        }

        function prevTestWord() {
            if (currentTestIndex > 0) {
                currentTestIndex--;
                saveCurrentSessionState();
                updateTestUI();
            }
        }

        function nextTestWord() {
            if (currentTestIndex < testWords.length - 1) {
                currentTestIndex++;
                saveCurrentSessionState();
                updateTestUI();
            }
        }

        function getNormalizedExamples(wordItem) {
            if (!wordItem) return [];
            if (!Array.isArray(wordItem.examples)) {
                wordItem.examples = [];
            }
            return wordItem.examples;
        }

        function updateExampleButtonUI() {
            const btnShowExample = document.getElementById('btn-show-example');
            if (!btnShowExample || !testWords[currentTestIndex]) return;
            const examples = getNormalizedExamples(testWords[currentTestIndex]);
            if (examples.length > 0) {
                btnShowExample.innerText = `📖 예문 보기 / 수정 (${examples.length}개)`;
            } else {
                btnShowExample.innerText = `➕ 예문 추가`;
            }
        }

        function updateTestUI() {
            const wordData = testWords[currentTestIndex];
            
            const titleType = isStudyMode ? '학습' : '테스트';
            document.getElementById('test-progress-text').innerText = `${titleType} (${currentTestIndex + 1}/${testWords.length})`;

            document.getElementById('btn-test-prev').disabled = (currentTestIndex === 0);
            document.getElementById('btn-test-next').disabled = (currentTestIndex === testWords.length - 1);

            const wordDisplay = document.getElementById('test-word-display');
            const phoneticDisplay = document.getElementById('test-phonetic-display');
            const meaningDisplay = document.getElementById('test-meaning-display');
            const toggleBtn = document.getElementById('btn-toggle-mode');
            const revealBtn = document.getElementById('btn-reveal-meaning');
            const scoreButtons = document.getElementById('score-buttons');
            const studyNextBtnContainer = document.getElementById('study-next-btn-container');
            
            wordDisplay.innerText = wordData.word;
            meaningDisplay.innerText = wordData.meaning;
            updatePhoneticDisplay(wordData, 'test-phonetic-display');
            
            if (testMode === 'word') {
                wordDisplay.classList.remove('hidden-text');
                phoneticDisplay.classList.remove('hidden-text');
                meaningDisplay.classList.add('hidden-text');
                toggleBtn.innerText = '모드: 단어 제시 🔄';
            } else {
                meaningDisplay.classList.remove('hidden-text');
                wordDisplay.classList.add('hidden-text');
                phoneticDisplay.classList.add('hidden-text');
                toggleBtn.innerText = '모드: 뜻 제시 🔄';
            }

            revealBtn.style.display = 'block';
            scoreButtons.style.display = 'none';
            studyNextBtnContainer.style.display = 'none';

            // 예문 버튼 초기화 (정답 확인 전에는 숨김)
            const btnShowExample = document.getElementById('btn-show-example');
            if (btnShowExample) {
                btnShowExample.style.display = 'none';
                updateExampleButtonUI();
            }

            playTestWordThreeTimes(wordData.word);
        }

        let testPlayTimeoutId = null;

        function playTestWordThreeTimes(word) {
            if (!('speechSynthesis' in window)) return;

            cancelTestSpeech();

            let playCount = 0;
            function playOnce() {
                playCount++;
                const utterance = new SpeechSynthesisUtterance(word);
                utterance.lang = 'en-US';
                utterance.onend = () => {
                    if (playCount < 3) {
                        testPlayTimeoutId = setTimeout(playOnce, 400);
                    }
                };
                window.speechSynthesis.speak(utterance);
            }
            playOnce();
        }

        function cancelTestSpeech() {
            if (testPlayTimeoutId) {
                clearTimeout(testPlayTimeoutId);
                testPlayTimeoutId = null;
            }
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
            }
        }

        function revealMeaning() {
            document.getElementById('test-word-display').classList.remove('hidden-text');
            document.getElementById('test-phonetic-display').classList.remove('hidden-text');
            document.getElementById('test-meaning-display').classList.remove('hidden-text');
            
            document.getElementById('btn-reveal-meaning').style.display = 'none';
            
            // 정답 확인 후 예문 추가/보기 버튼 노출
            updateExampleButtonUI();
            document.getElementById('btn-show-example').style.display = 'inline-block';

            if (isStudyMode) {
                document.getElementById('study-next-btn-container').style.display = 'block';
                const nextBtn = document.getElementById('btn-study-next');
                if (currentTestIndex >= testWords.length - 1) {
                    nextBtn.innerText = '학습 완료하기';
                    nextBtn.onclick = finishStudyRound;
                } else {
                    nextBtn.innerText = '다음 단어';
                    nextBtn.onclick = nextStudyWord;
                }
            } else {
                document.getElementById('score-buttons').style.display = 'flex';
            }
        }

        // ==========================================
        // [수정된 기능] 예문 추가 / 보기 / 수정 모달 로직
        // ==========================================
        function getSplitMeanings(meaningStr) {
            if (!meaningStr) return [];
            return meaningStr.split(/,\s*/).map(m => m.trim()).filter(Boolean);
        }

        function syncExamplesToStorage(wordItem) {
            if (!wordItem) return;
            const examplesCopy = [...getNormalizedExamples(wordItem)];

            if (currentFile && vocabData[currentFile]) {
                vocabData[currentFile].forEach(item => {
                    if (!item.isDivider && item.word === wordItem.word && item.meaning === wordItem.meaning) {
                        item.examples = examplesCopy;
                    }
                });
            }

            // 다른 단어장(원본 또는 자동생성 단어장)에 있는 동일 단어에도 예문 동기화
            Object.keys(vocabData).forEach(fileName => {
                (vocabData[fileName] || []).forEach(item => {
                    if (!item.isDivider && item.word === wordItem.word && item.meaning === wordItem.meaning) {
                        item.examples = examplesCopy;
                    }
                });
            });

            saveData();
            saveCurrentSessionState();
            updateExampleButtonUI();
        }

        function openExampleModal() {
            const currentTestWord = testWords[currentTestIndex];
            if (!currentTestWord) return;

            // 원본 단어장에 저장된 최신 예문이 있으면 동기화
            if (currentFile && vocabData[currentFile]) {
                const orig = vocabData[currentFile].find(
                    item => !item.isDivider && item.word === currentTestWord.word && item.meaning === currentTestWord.meaning
                );
                if (orig && Array.isArray(orig.examples)) {
                    currentTestWord.examples = [...orig.examples];
                }
            }

            document.getElementById('example-modal-word').innerText = currentTestWord.word;
            document.getElementById('example-modal-meaning').innerText = currentTestWord.meaning;

            setupExampleMeaningSelect(currentTestWord.meaning);
            resetExampleForm();
            renderExampleModalList();

            document.getElementById('example-modal-overlay').style.display = 'flex';

            const examples = getNormalizedExamples(currentTestWord);
            if (examples.length === 0) {
                setTimeout(() => document.getElementById('example-input-en').focus(), 100);
            }
        }

        function closeExampleModal() {
            document.getElementById('example-modal-overlay').style.display = 'none';
            resetExampleForm();
            updateExampleButtonUI();
        }

        function setupExampleMeaningSelect(meaningStr) {
            const meanings = getSplitMeanings(meaningStr);
            const wrapEl = document.getElementById('example-meaning-select-wrap');
            const selectEl = document.getElementById('example-input-meaning');
            selectEl.innerHTML = '';

            if (meanings.length > 1) {
                wrapEl.style.display = 'block';
                const defaultOpt = document.createElement('option');
                defaultOpt.value = '';
                defaultOpt.innerText = '전체 / 뜻 구분 없음';
                selectEl.appendChild(defaultOpt);

                meanings.forEach(m => {
                    const opt = document.createElement('option');
                    opt.value = m;
                    opt.innerText = `뜻: ${m}`;
                    selectEl.appendChild(opt);
                });
            } else {
                wrapEl.style.display = 'none';
            }
        }

        function renderExampleModalList() {
            const currentTestWord = testWords[currentTestIndex];
            const listEl = document.getElementById('example-list-container');
            const examples = getNormalizedExamples(currentTestWord);

            if (examples.length === 0) {
                listEl.innerHTML = '<div class="empty-message" style="padding: 16px 10px; font-size: 13px;">등록된 예문이 없습니다.<br>아래에서 영어 예문과 한글 해석을 추가해보세요!</div>';
                return;
            }

            listEl.innerHTML = examples.map((ex, idx) => {
                const badgeHtml = ex.meaningTag
                    ? `<span class="example-meaning-badge">${escapeHtml(ex.meaningTag)}</span>`
                    : `<span style="font-size: 12px; color: #94a3b8; font-weight: bold;">예문 ${idx + 1}</span>`;

                return `
                    <div class="example-item-card">
                        <div class="example-item-header">
                            <div style="display: flex; align-items: center; gap: 6px;">
                                ${badgeHtml}
                                <button class="btn-tts" style="font-size: 15px; padding: 0 4px;" onclick="speakExampleSentence(${idx})" title="예문 발음 듣기">🔊</button>
                            </div>
                            <div class="example-item-actions">
                                <button class="btn-edit" onclick="startEditExample(${idx})">수정</button>
                                <button class="btn-delete" onclick="deleteExampleItem(${idx})">삭제</button>
                            </div>
                        </div>
                        <div class="example-item-en">${escapeHtml(ex.en)}</div>
                        <div class="example-item-ko">${escapeHtml(ex.ko)}</div>
                    </div>
                `;
            }).join('');
        }

        function speakExampleSentence(idx) {
            const currentTestWord = testWords[currentTestIndex];
            const examples = getNormalizedExamples(currentTestWord);
            if (examples[idx] && examples[idx].en) {
                speakWord(examples[idx].en);
            }
        }

        function saveExampleItem() {
            const currentTestWord = testWords[currentTestIndex];
            if (!currentTestWord) return;

            const enInput = document.getElementById('example-input-en');
            const koInput = document.getElementById('example-input-ko');
            const meaningSelect = document.getElementById('example-input-meaning');
            const selectWrap = document.getElementById('example-meaning-select-wrap');

            const enText = enInput.value.trim();
            const koText = koInput.value.trim();
            const meaningTag = (selectWrap.style.display !== 'none') ? meaningSelect.value.trim() : '';

            if (!enText || !koText) {
                alert('영어 예문과 한글 해석을 모두 입력해주세요.');
                return;
            }

            const examples = getNormalizedExamples(currentTestWord);
            const newEntry = {
                en: enText,
                ko: koText,
                meaningTag: meaningTag
            };

            if (editingExampleIndex !== null && editingExampleIndex >= 0 && editingExampleIndex < examples.length) {
                examples[editingExampleIndex] = newEntry;
            } else {
                examples.push(newEntry);
            }

            syncExamplesToStorage(currentTestWord);
            resetExampleForm();
            renderExampleModalList();
        }

        function startEditExample(idx) {
            const currentTestWord = testWords[currentTestIndex];
            const examples = getNormalizedExamples(currentTestWord);
            const target = examples[idx];
            if (!target) return;

            editingExampleIndex = idx;
            document.getElementById('example-form-title').innerText = `✏️ 예문 수정 (#${idx + 1})`;
            document.getElementById('btn-save-example').innerText = '수정 완료';
            document.getElementById('btn-cancel-edit-example').style.display = 'inline-block';

            document.getElementById('example-input-en').value = target.en || '';
            document.getElementById('example-input-ko').value = target.ko || '';

            const selectEl = document.getElementById('example-input-meaning');
            const selectWrap = document.getElementById('example-meaning-select-wrap');
            if (selectWrap.style.display !== 'none') {
                const tagExists = Array.from(selectEl.options).some(opt => opt.value === (target.meaningTag || ''));
                selectEl.value = tagExists ? (target.meaningTag || '') : '';
            }

            document.getElementById('example-input-en').focus();
        }

        function deleteExampleItem(idx) {
            if (!confirm('이 예문을 삭제하시겠습니까?')) return;
            const currentTestWord = testWords[currentTestIndex];
            const examples = getNormalizedExamples(currentTestWord);
            examples.splice(idx, 1);

            if (editingExampleIndex === idx) {
                resetExampleForm();
            } else if (editingExampleIndex !== null && editingExampleIndex > idx) {
                editingExampleIndex--;
            }

            syncExamplesToStorage(currentTestWord);
            renderExampleModalList();
        }

        function resetExampleForm() {
            editingExampleIndex = null;
            document.getElementById('example-form-title').innerText = '➕ 새 예문 추가';
            document.getElementById('btn-save-example').innerText = '예문 저장';
            document.getElementById('btn-cancel-edit-example').style.display = 'none';
            document.getElementById('example-input-en').value = '';
            document.getElementById('example-input-ko').value = '';
            const selectEl = document.getElementById('example-input-meaning');
            if (selectEl) selectEl.value = '';
        }

        // ==========================================
        // 누적 10개(및 나머지) 매칭 퀴즈 로직
        // ==========================================
        function shouldTriggerMatchingQuiz(index) {
            return ((index + 1) % 10 === 0) || (index === testWords.length - 1);
        }

        function getCurrentBatchWords(endIndex) {
            const batchStart = Math.floor(endIndex / 10) * 10;
            return testWords.slice(batchStart, endIndex + 1);
        }

        function openMatchingQuiz(batchWords, onComplete) {
            cancelTestSpeech();
            matchingQuizCallback = onComplete;
            selectedMatchWordBtn = null;
            selectedMatchMeaningBtn = null;
            matchedPairsCount = 0;
            totalMatchPairs = batchWords.length;

            document.getElementById('matching-quiz-progress').innerText = `0 / ${totalMatchPairs}`;
            document.getElementById('btn-matching-continue').style.display = 'none';

            const wordsCol = document.getElementById('matching-words-col');
            const meaningsCol = document.getElementById('matching-meanings-col');
            wordsCol.innerHTML = '';
            meaningsCol.innerHTML = '';

            const wordEntries = batchWords.map((item, idx) => ({
                pairId: idx,
                word: item.word,
                meaning: item.meaning
            }));
            const meaningEntries = batchWords.map((item, idx) => ({
                pairId: idx,
                word: item.word,
                meaning: item.meaning
            }));

            shuffleArray(wordEntries);
            shuffleArray(meaningEntries);

            wordEntries.forEach(entry => {
                const btn = document.createElement('button');
                btn.className = 'match-card';
                btn.innerText = entry.word;
                btn.dataset.pairId = entry.pairId;
                btn.dataset.word = entry.word.toLowerCase().trim();
                btn.dataset.meaning = entry.meaning.toLowerCase().trim();
                btn.onclick = () => selectMatchCard(btn, 'word');
                wordsCol.appendChild(btn);
            });

            meaningEntries.forEach(entry => {
                const btn = document.createElement('button');
                btn.className = 'match-card';
                btn.innerText = entry.meaning;
                btn.dataset.pairId = entry.pairId;
                btn.dataset.word = entry.word.toLowerCase().trim();
                btn.dataset.meaning = entry.meaning.toLowerCase().trim();
                btn.onclick = () => selectMatchCard(btn, 'meaning');
                meaningsCol.appendChild(btn);
            });

            document.getElementById('matching-quiz-overlay').style.display = 'flex';
        }

        function selectMatchCard(btn, type) {
            if (btn.classList.contains('matched') || btn.classList.contains('wrong')) return;

            if (type === 'word') {
                if (selectedMatchWordBtn === btn) {
                    btn.classList.remove('selected');
                    selectedMatchWordBtn = null;
                    return;
                }
                if (selectedMatchWordBtn) selectedMatchWordBtn.classList.remove('selected');
                selectedMatchWordBtn = btn;
                btn.classList.add('selected');
            } else {
                if (selectedMatchMeaningBtn === btn) {
                    btn.classList.remove('selected');
                    selectedMatchMeaningBtn = null;
                    return;
                }
                if (selectedMatchMeaningBtn) selectedMatchMeaningBtn.classList.remove('selected');
                selectedMatchMeaningBtn = btn;
                btn.classList.add('selected');
            }

            if (selectedMatchWordBtn && selectedMatchMeaningBtn) {
                checkMatchPair(selectedMatchWordBtn, selectedMatchMeaningBtn);
            }
        }

        function checkMatchPair(wordBtn, meaningBtn) {
            selectedMatchWordBtn = null;
            selectedMatchMeaningBtn = null;

            const isCorrectMatch =
                wordBtn.dataset.pairId === meaningBtn.dataset.pairId ||
                wordBtn.dataset.meaning === meaningBtn.dataset.meaning ||
                wordBtn.dataset.word === meaningBtn.dataset.word;

            if (isCorrectMatch) {
                wordBtn.classList.remove('selected');
                meaningBtn.classList.remove('selected');
                wordBtn.classList.add('matched');
                meaningBtn.classList.add('matched');
                wordBtn.disabled = true;
                meaningBtn.disabled = true;

                matchedPairsCount++;
                document.getElementById('matching-quiz-progress').innerText = `${matchedPairsCount} / ${totalMatchPairs}`;

                if (matchedPairsCount >= totalMatchPairs) {
                    document.getElementById('btn-matching-continue').style.display = 'block';
                }
            } else {
                wordBtn.classList.remove('selected');
                meaningBtn.classList.remove('selected');
                wordBtn.classList.add('wrong');
                meaningBtn.classList.add('wrong');

                setTimeout(() => {
                    wordBtn.classList.remove('wrong');
                    meaningBtn.classList.remove('wrong');
                }, 400);
            }
        }

        function closeMatchingQuizAndContinue() {
            document.getElementById('matching-quiz-overlay').style.display = 'none';
            if (typeof matchingQuizCallback === 'function') {
                const cb = matchingQuizCallback;
                matchingQuizCallback = null;
                cb();
            }
        }

        function nextStudyWord() {
            const proceedNext = () => {
                currentTestIndex++;
                saveCurrentSessionState();
                updateTestUI();
            };

            if (shouldTriggerMatchingQuiz(currentTestIndex)) {
                saveCurrentSessionState();
                openMatchingQuiz(getCurrentBatchWords(currentTestIndex), proceedNext);
            } else {
                proceedNext();
            }
        }

        function finishStudyRound() {
            const completeRound = () => {
                wrongStudyCounts[currentFile] = (wrongStudyCounts[currentFile] || 0) + 1;
                wrongStudyProgress[currentFile] = 0;
                clearCurrentSessionState(currentFile, 'study');
                saveWrongStudy();
                saveWrongStudyProgress();
                alert(`오답 학습을 1회 완료했습니다. (현재 ${wrongStudyCounts[currentFile]}/2회)`);
                navigate('file');
            };

            if (shouldTriggerMatchingQuiz(currentTestIndex)) {
                saveCurrentSessionState();
                openMatchingQuiz(getCurrentBatchWords(currentTestIndex), completeRound);
            } else {
                completeRound();
            }
        }

        function recordAnswer(isCorrect) {
            const currentTestWord = testWords[currentTestIndex];
            currentSessionAnswers[currentTestIndex] = isCorrect;
            correctCount = Object.values(currentSessionAnswers).filter(Boolean).length;

            if (!isWrongOnlyTest) {
                const originalIndex = vocabData[currentFile].findIndex(
                    item => !item.isDivider && item.word === currentTestWord.word && item.meaning === currentTestWord.meaning
                );

                if (originalIndex !== -1) {
                    vocabData[currentFile][originalIndex].isWrong = !isCorrect;
                    saveData();
                }
            }

            saveCurrentSessionState();

            const proceedAfterAnswer = () => {
                currentTestIndex++;
                if (currentTestIndex >= testWords.length) {
                    showTestResult();
                } else {
                    saveCurrentSessionState();
                    updateTestUI();
                }
            };

            if (shouldTriggerMatchingQuiz(currentTestIndex)) {
                openMatchingQuiz(getCurrentBatchWords(currentTestIndex), proceedAfterAnswer);
            } else {
                proceedAfterAnswer();
            }
        }

        function showTestResult() {
            cancelTestSpeech();

            document.getElementById('test-content').style.display = 'none';
            document.getElementById('test-result').style.display = 'block';

            if (isWrongOnlyTest) {
                testWords.forEach((wordItem, idx) => {
                    const ans = currentSessionAnswers[idx];
                    if (ans !== undefined) {
                        const originalIndex = vocabData[currentFile].findIndex(
                            item => !item.isDivider && item.word === wordItem.word && item.meaning === wordItem.meaning
                        );
                        if (originalIndex !== -1) {
                            vocabData[currentFile][originalIndex].isWrong = !ans;
                        }
                    }
                });
                saveData();

                wrongStudyCounts[currentFile] = 0;
                wrongStudyProgress[currentFile] = 0;
                clearCurrentSessionState(currentFile, 'wrongTest');
                clearCurrentSessionState(currentFile, 'study');
                saveWrongStudy();
                saveWrongStudyProgress();
                isWrongOnlyTest = false;
            } else {
                clearCurrentSessionState(currentFile, 'normal');
            }

            const percentage = Math.round((correctCount / testWords.length) * 100);
            document.getElementById('result-score-display').innerText = `${percentage}%`;
            document.getElementById('result-detail-text').innerText = `총 ${testWords.length}문제 중 ${correctCount}개 정답`;
        }

        // ==========================================
        // 발음 듣기 기능
        // ==========================================
        function startListening() {
            const words = (vocabData[currentFile] || []).filter(item => !item.isDivider);
            if (words.length === 0) {
                alert('발음을 들을 단어가 없습니다. 먼저 단어를 추가해주세요.');
                return;
            }

            listenWords = words;

            const slider = document.getElementById('listen-slider');
            slider.min = 0;
            slider.max = listenWords.length - 1;

            navigate('listen');
            beginListenWord(0);
        }

        function beginListenWord(index) {
            clearListenTimers();

            listenIndex = index;
            listenPlayCount = 0;

            const wordData = listenWords[listenIndex];
            document.getElementById('listen-word-display').innerText = wordData.word;
            updatePhoneticDisplay(wordData, 'listen-phonetic-display');
            document.getElementById('listen-meaning-display').innerText = wordData.meaning;
            document.getElementById('listen-meaning-display').classList.add('hidden-text');
            document.getElementById('listen-progress-text').innerText = `발음 듣기 (${listenIndex + 1}/${listenWords.length})`;
            document.getElementById('listen-slider').value = listenIndex;

            playListenWordOnce();
        }

        function clearListenTimers() {
            if (listenTimeoutId) {
                clearTimeout(listenTimeoutId);
                listenTimeoutId = null;
            }
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
            }
        }

        function stopListening() {
            clearListenTimers();
        }

        function speakCurrentListenWord() {
            if (listenWords[listenIndex]) {
                speakWord(listenWords[listenIndex].word);
            }
        }

        function onListenSliderChange(value) {
            beginListenWord(parseInt(value, 10));
        }

        function playListenWordOnce() {
            if (!('speechSynthesis' in window)) {
                alert('현재 브라우저는 음성 재생을 지원하지 않습니다.');
                return;
            }

            const wordData = listenWords[listenIndex];
            listenPlayCount++;
            
            document.getElementById('listen-play-count').innerText = `재생: ${listenPlayCount}/3`;

            const utterance = new SpeechSynthesisUtterance(wordData.word);
            utterance.lang = 'en-US';
            
            utterance.onend = () => {
                // 두 번째 재생 시 뜻 공개
                if (listenPlayCount === 2) {
                    document.getElementById('listen-meaning-display').classList.remove('hidden-text');
                }
                
                // 총 3번 반복 후 다음 단어로 넘어가기
                if (listenPlayCount < 3) {
                    listenTimeoutId = setTimeout(playListenWordOnce, 800);
                } else {
                    listenTimeoutId = setTimeout(() => {
                        if (listenIndex < listenWords.length - 1) {
                            beginListenWord(listenIndex + 1);
                        } else {
                            alert('해당 단어장의 발음 듣기가 모두 끝났습니다.');
                            navigate('file');
                        }
                    }, 1500);
                }
            };
            
            window.speechSynthesis.speak(utterance);
        }
    </script>
</body>
</html>
