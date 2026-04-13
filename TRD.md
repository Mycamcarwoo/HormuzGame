# 호르무즈 지로찾기 - 기술 요구사항 문서 (TRD)

## 1. 개요

### 1.1 프로젝트명
호르무즈 지로찾기 (Hormuz Spot Finder)

### 1.2 기술 스택
- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **지도 라이브러리**: Leaflet.js 1.9.4+
- **데이터 저장**: LocalStorage (클라이언트 측)
- **배포**: GitHub Pages / Netlify

### 1.3 기술적 목표
1. **모바일 최적화**: 터치 이벤트 지원, 반응형 디자인
2. **빠른 로딩**: 3초 이내 초기 로딩
3. **오프라인 지원**: Service Worker로 오프라인 플레이 가능
4. **접근성**: 웹 접근성 표준 준수

---

## 2. 시스템 아키텍처

### 2.1 전체 구조

```text
┌─────────────────────────────────────────────────────┐
│                   Frontend Layer                     │
│  ┌───────────────┐  ┌───────────────┐  ┌──────────┐ │
│  │  Game Engine  │  │   UI Layer    │  │   Map    │ │
│  │               │  │               │  │  Engine  │ │
│  │  - Game Logic │  │  - DOM Update │  │          │ │
│  │  - Scoring    │  │  - Events     │  │ Leaflet  │ │
│  │  - State Mgmt │  │  - Animations │  │  .js     │ │
│  └───────────────┘  └───────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────┘
                         ▲
                         │
┌─────────────────────────────────────────────────────┐
│                  Data Layer                         │
│  ┌───────────────┐  ┌───────────────┐  ┌──────────┐ │
│  │  LocalStorage │  │  OpenStreet   │  │  Game    │ │
│  │               │  │      Map      │  │  Data    │ │
│  │  - User Score │  │  - Tiles      │  │  - Clues │ │
│  │  - Settings   │  │  - Layers    │  │  - Info  │ │
│  └───────────────┘  └───────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────┘
```

### 2.2 주요 컴포넌트

#### 2.2.1 Game Engine
- 게임 로직
- 점수 계산
- 상태 관리

#### 2.2.2 UI Layer
- DOM 업데이트
- 이벤트 처리
- 애니메이션

#### 2.2.3 Map Engine
- 지도 렌더링
- 클릭 이벤트
- 마커 표시

#### 2.2.4 Data Layer
- LocalStorage 관리
- 지도 데이터 로드
- 게임 데이터 저장

---

## 3. 상세 기술 사양

### 3.1 Frontend

#### 3.1.1 HTML5
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>호르무즈 지로찾기</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div id="app">
        <header>
            <h1>호르무즈 지로찾기</h1>
            <div id="score">점수: 0</div>
        </header>
        <main>
            <div id="map"></div>
            <div id="clues">
                <div class="clue-card" id="clue-1">단서 1</div>
                <div class="clue-card" id="clue-2">단서 2</div>
                <div class="clue-card" id="clue-3">단서 3</div>
            </div>
            <div id="controls">
                <button id="hint-btn">힌트</button>
                <button id="submit-btn">정답 제출</button>
                <button id="restart-btn">다시하기</button>
            </div>
        </main>
        <div id="result-modal" class="modal hidden">
            <h2>결과</h2>
            <div id="result-content"></div>
            <button id="close-modal">닫기</button>
        </div>
    </div>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="js/game.js"></script>
</body>
</html>
```

#### 3.1.2 CSS3
```css
/* 반응형 디자인 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Noto Sans KR', sans-serif;
    background-color: #f5f5f5;
    color: #333;
}

#app {
    display: grid;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

/* 헤더 */
header {
    background-color: #2c3e50;
    color: white;
    padding: 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* 지도 */
#map {
    height: 50vh;
    min-height: 300px;
}

/* 단서 카드 */
#clues {
    padding: 1rem;
    display: grid;
    gap: 1rem;
}

.clue-card {
    background-color: white;
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s;
}

.clue-card:hover {
    transform: translateY(-2px);
}

/* 컨트롤 */
#controls {
    padding: 1rem;
    display: grid;
    gap: 0.5rem;
}

button {
    padding: 0.75rem;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

button:active {
    transform: scale(0.98);
}

/* 모바일 최적화 */
@media (max-width: 768px) {
    #map {
        height: 40vh;
    }

    .clue-card {
        font-size: 0.9rem;
    }

    button {
        padding: 0.5rem;
        font-size: 0.9rem;
    }
}

/* 모달 */
.modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal.hidden {
    display: none;
}

.modal-content {
    background-color: white;
    padding: 2rem;
    border-radius: 12px;
    max-width: 90%;
    max-height: 80vh;
    overflow-y: auto;
}
```

#### 3.1.3 JavaScript (ES6+)
```javascript
// Game Engine
class GameEngine {
    constructor() {
        this.score = 0;
        this.timeUsed = 0;
        this.hintsUsed = 0;
        this.startTime = null;
        this.userGuess = null;
        this.correctAnswer = { lat: 26.5833, lng: 56.25 };
        this.state = 'ready'; // ready, playing, finished
    }

    start() {
        this.startTime = Date.now();
        this.state = 'playing';
        this.startTimer();
    }

    startTimer() {
        setInterval(() => {
            if (this.state === 'playing') {
                this.timeUsed = Math.floor((Date.now() - this.startTime) / 1000);
                this.updateTimerDisplay();
            }
        }, 1000);
    }

    makeGuess(lat, lng) {
        if (this.state !== 'playing') return;

        this.userGuess = { lat, lng };
        this.calculateScore();
        this.state = 'finished';
        this.showResult();
    }

    calculateScore() {
        // 거리 계산 (Haversine 공식)
        const distance = this.calculateDistance(
            this.userGuess.lat,
            this.userGuess.lng,
            this.correctAnswer.lat,
            this.correctAnswer.lng
        );

        // 정확도 점수
        let accuracyScore;
        if (distance <= 50) accuracyScore = 100;
        else if (distance <= 100) accuracyScore = 80;
        else if (distance <= 200) accuracyScore = 60;
        else if (distance <= 500) accuracyScore = 40;
        else accuracyScore = 20;

        // 시간 보너스
        let timeBonus;
        if (this.timeUsed <= 60) timeBonus = 50;
        else if (this.timeUsed <= 120) timeBonus = 30;
        else if (this.timeUsed <= 180) timeBonus = 10;
        else timeBonus = 0;

        // 힌트 페널티
        const hintPenalty = this.hintsUsed * 20;

        // 총점
        this.score = accuracyScore + timeBonus - hintPenalty;
        this.score = Math.max(0, this.score); // 0점 이하로 내려가지 않음
    }

    calculateDistance(lat1, lng1, lat2, lng2) {
        const R = 6371; // 지구 반지름 (km)
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLng = (lng2 - lng1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLng/2) * Math.sin(dLng/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }

    showResult() {
        const modal = document.getElementById('result-modal');
        const content = document.getElementById('result-content');

        const distance = this.calculateDistance(
            this.userGuess.lat,
            this.userGuess.lng,
            this.correctAnswer.lat,
            this.correctAnswer.lng
        );

        content.innerHTML = `
            <h3>최종 점수: ${this.score}점</h3>
            <p>거리: ${distance.toFixed(2)} km</p>
            <p>사용 시간: ${this.timeUsed}초</p>
            <p>힌트 사용: ${this.hintsUsed}회</p>
            <div class="educational-info">
                <h4>호르무즈 해협이란?</h4>
                <p>
                    호르무즈 해협은 페르시아만과 오만만을 연결하는 좁은 해협으로,
                    세계 석유 수송의 20%가 통과하는 전략적으로 중요한 위치입니다.
                    이란과 아랍에미리트 사이에 위치하며, 폭은 약 54km입니다.
                </p>
            </div>
        `;

        modal.classList.remove('hidden');
    }

    updateTimerDisplay() {
        const timer = document.getElementById('timer');
        const minutes = Math.floor(this.timeUsed / 60);
        const seconds = this.timeUsed % 60;
        timer.textContent = `${minutes}:${seconds.toString().padStart(2, '0')}`;
    }
}

// Map Engine
class MapEngine {
    constructor(gameEngine) {
        this.gameEngine = gameEngine;
        this.map = null;
        this.userMarker = null;
        this.answerMarker = null;
    }

    init() {
        this.map = L.map('map').setView([20, 60], 3);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(this.map);

        // 클릭 이벤트
        this.map.on('click', (e) => {
            this.handleMapClick(e.latlng);
        });
    }

    handleMapClick(latlng) {
        // 기존 마커 제거
        if (this.userMarker) {
            this.map.removeLayer(this.userMarker);
        }

        // 새 마커 추가
        this.userMarker = L.marker(latlng).addTo(this.map);
    }

    submitAnswer() {
        if (!this.userMarker) {
            alert('지도를 클릭하여 위치를 선택해주세요.');
            return;
        }

        const latlng = this.userMarker.getLatLng();
        this.gameEngine.makeGuess(latlng.lat, latlng.lng);
    }

    showAnswer() {
        // 정답 마커 표시
        this.answerMarker = L.marker(
            [this.gameEngine.correctAnswer.lat, this.gameEngine.correctAnswer.lng],
            { color: 'red' }
        ).addTo(this.map);

        // 두 마커 사이에 선 그리기
        const latlngs = [
            this.userMarker.getLatLng(),
            [this.gameEngine.correctAnswer.lat, this.gameEngine.correctAnswer.lng]
        ];
        L.polyline(latlngs, { color: 'red' }).addTo(this.map);

        // 지도를 두 마커가 모두 보이도록 조정
        this.map.fitBounds([this.userMarker.getLatLng(), this.answerMarker.getLatLng()], {
            padding: [50, 50]
        });
    }
}

// UI Manager
class UIManager {
    constructor(gameEngine, mapEngine) {
        this.gameEngine = gameEngine;
        this.mapEngine = mapEngine;
    }

    init() {
        document.getElementById('submit-btn').addEventListener('click', () => {
            this.mapEngine.submitAnswer();
            this.mapEngine.showAnswer();
        });

        document.getElementById('restart-btn').addEventListener('click', () => {
            location.reload();
        });

        document.getElementById('close-modal').addEventListener('click', () => {
            document.getElementById('result-modal').classList.add('hidden');
        });

        document.getElementById('hint-btn').addEventListener('click', () => {
            this.showHint();
        });
    }

    showHint() {
        if (this.gameEngine.hintsUsed >= 3) {
            alert('힌트는 게임당 최대 3개까지만 사용할 수 있습니다.');
            return;
        }

        this.gameEngine.hintsUsed++;

        // 힌트 종류 선택
        const hintType = this.gameEngine.hintsUsed;
        let hintText;

        switch (hintType) {
            case 1:
                hintText = '힌트: 페르시아만과 오만만 사이에 위치합니다.';
                break;
            case 2:
                hintText = '힌트: 이란과 아랍에미리트 사이에 있습니다.';
                break;
            case 3:
                hintText = '힌트: 호르무즈 섬 근처입니다.';
                break;
        }

        alert(hintText);
        this.gameEngine.score -= 20;
    }
}

// 초기화
document.addEventListener('DOMContentLoaded', () => {
    const gameEngine = new GameEngine();
    const mapEngine = new MapEngine(gameEngine);
    const uiManager = new UIManager(gameEngine, mapEngine);

    mapEngine.init();
    uiManager.init();
    gameEngine.start();
});
```

### 3.2 지도 라이브러리 (Leaflet.js)

#### 3.2.1 설치
```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

#### 3.2.2 기본 설정
```javascript
const map = L.map('map').setView([20, 60], 3);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors',
    maxZoom: 19,
    minZoom: 2
}).addTo(map);
```

#### 3.2.3 마커 및 이벤트
```javascript
// 마커 추가
const marker = L.marker([26.5833, 56.25]).addTo(map);

// 클릭 이벤트
map.on('click', (e) => {
    console.log('Clicked at:', e.latlng);
});

// 지도 범위 설정
map.fitBounds([
    [20, 50],
    [30, 70]
]);
```

### 3.3 데이터 관리

#### 3.3.1 LocalStorage 사용
```javascript
// 점수 저장
localStorage.setItem('bestScore', 150);

// 점수 불러오기
const bestScore = localStorage.getItem('bestScore');

// 설정 저장
const settings = {
    soundEnabled: true,
    vibrationEnabled: true
};
localStorage.setItem('settings', JSON.stringify(settings));

// 설정 불러오기
const savedSettings = JSON.parse(localStorage.getItem('settings'));
```

#### 3.3.2 게임 데이터
```javascript
// 호르무즈 해협 데이터
const hormuzData = {
    name: '호르무즈 해협',
    coordinates: { lat: 26.5833, lng: 56.25 },
    clues: [
        '페르시아만과 오만만을 연결하는 해협',
        '세계 석유 수송의 20%가 통과',
        '이란과 아랍에미리트 사이에 위치'
    ],
    hints: [
        '페르시아만과 오만만 사이에 위치합니다.',
        '이란과 아랍에미리트 사이에 있습니다.',
        '호르무즈 섬 근처입니다.'
    ],
    educationalInfo: `
        호르무즈 해협은 페르시아만과 오만만을 연결하는 좁은 해협으로,
        세계 석유 수송의 20%가 통과하는 전략적으로 중요한 위치입니다.
        이란과 아랍에미리트 사이에 위치하며, 폭은 약 54km입니다.
    `
};
```

### 3.4 성능 최적화

#### 3.4.1 이미지 최적화
- WebP 형식 사용
- 이미지 크기 적절하게 조정
- 지도 타일 캐싱

#### 3.4.2 JavaScript 최적화
- 코드 분할 (Code Splitting)
- 지연 로딩 (Lazy Loading)
- 이벤트 디바운싱 (Debouncing)

```javascript
// 디바운싱 예시
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

map.on('move', debounce(() => {
    console.log('Map moved');
}, 100));
```

### 3.5 모바일 최적화

#### 3.5.1 터치 이벤트
```javascript
// 터치 이벤트 처리
map.on('click touchstart', (e) => {
    console.log('Touched at:', e.latlng);
});
```

#### 3.5.2 뷰포트 설정
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

#### 3.5.3 진동 피드백
```javascript
// 진동 피드백 (안드로이드)
if (navigator.vibrate) {
    navigator.vibrate(100); // 100ms 진동
}
```

### 3.6 오프라인 지원 (Service Worker)

#### 3.6.1 Service Worker 등록
```javascript
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js')
        .then(registration => {
            console.log('Service Worker registered:', registration);
        })
        .catch(error => {
            console.log('Service Worker registration failed:', error);
        });
}
```

#### 3.6.2 Service Worker 구현
```javascript
const CACHE_NAME = 'hormuz-game-v1';
const urlsToCache = [
    '/',
    '/index.html',
    '/css/style.css',
    '/js/game.js',
    'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css',
    'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js'
];

self.addEventListener('install', event => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then(cache => cache.addAll(urlsToCache))
    );
});

self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request)
            .then(response => {
                // 캐시에 있으면 반환, 없으면 네트워크 요청
                return response || fetch(event.request);
            })
    );
});
```

---

## 4. 보안 및 개인정보

### 4.1 보안 고려사항
- **XSS 방지**: 사용자 입력 검증 및 이스케이프
- **HTTPS 사용**: 모든 통신 암호화
- **Content Security Policy**: CSP 헤더 설정

### 4.2 개인정보 처리
- **데이터 수집 최소화**: 점수 및 설정만 LocalStorage에 저장
- **서버 데이터 없음**: 모든 데이터 클라이언트 측에서 처리

---

## 5. 테스트

### 5.1 단위 테스트
```javascript
// 거리 계산 테스트
test('calculateDistance', () => {
    const distance = gameEngine.calculateDistance(26.5833, 56.25, 26.5, 56.2);
    expect(distance).toBeCloseTo(5.6, 1);
});

// 점수 계산 테스트
test('calculateScore', () => {
    gameEngine.timeUsed = 30;
    gameEngine.hintsUsed = 0;
    gameEngine.userGuess = { lat: 26.6, lng: 56.3 };
    gameEngine.calculateScore();
    expect(gameEngine.score).toBeGreaterThanOrEqual(80);
});
```

### 5.2 통합 테스트
- 게임 플레이 전체 흐름 테스트
- 지도 클릭 및 정답 제출 테스트
- 점수 계산 및 결과 화면 테스트

### 5.3 모바일 테스트
- iOS Safari 테스트
- Android Chrome 테스트
- 다양한 화면 크기 테스트

---

## 6. 배포

### 6.1 GitHub Pages 배포
```bash
# GitHub 리포지토리 생성
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/hormuz-game.git
git push -u origin main

# GitHub Pages 설정
# Settings → Pages → Source: main branch → Save
```

### 6.2 Netlify 배포
```bash
# Netlify CLI 설치
npm install -g netlify-cli

# 배포
netlify deploy --prod
```

### 6.3 CDN 설정
- Cloudflare 또는 AWS CloudFront 사용
- 정적 리소스 캐싱 설정

---

## 7. 성능 모니터링

### 7.1 웹 성능
- **Lighthouse 점수**: 90점 이상
- **FCP (First Contentful Paint)**: 1.5초 이내
- **TTI (Time to Interactive)**: 3초 이내
- **CLS (Cumulative Layout Shift)**: 0.1 이하

### 7.2 사용자 분석
- Google Analytics 통합
- 이벤트 추적 (게임 시작, 정답 제출, 완료)
- 사용자 유입 경로 분석

---

## 8. 개발 일정

### 8.1 Week 1: 기본 기능 구현
- Day 1-2: HTML/CSS 구조 구현
- Day 3-4: JavaScript 로직 구현
- Day 5: 지도 기능 구현

### 8.2 Week 2: UI/UX 개선
- Day 1-2: 반응형 디자인
- Day 3-4: 모바일 최적화
- Day 5: 애니메이션 및 효과

### 8.3 Week 3: 테스트 및 배포
- Day 1-2: 테스트 및 버그 수정
- Day 3-4: 성능 최적화
- Day 5: 배포 및 릴리스

---

## 9. 기술 우선순위

### 9.1 MVP (최소 기능 제품)
- [x] 기본 게임 플레이
- [x] 세계 지도 표시
- [x] 점수 계산
- [x] 모바일 지원

### 9.2 Phase 2 (업데이트)
- [ ] 힌트 시스템
- [ ] 랭킹 시스템
- [ ] Service Worker (오프라인)

### 9.3 Phase 3 (확장)
- [ ] 소셜 공유
- [ ] 여러 위치 도전
- [ ] 멀티플레이어

---

*작성일: 2026-04-13*
*작성자*: nanobot
*버전*: 1.0
