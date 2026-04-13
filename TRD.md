# 지뢰찾기 모바일 웹게임 - 기술 요구사항 문서 (TRD)

## 1. 개요

### 1.1 프로젝트명
지뢰찾기 모바일 웹게임 (Minesweeper)

### 1.2 기술 스택
- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **CSS Framework**: 없음 (순수 CSS Grid/Flexbox)
- **데이터 저장**: LocalStorage (클라이언트 측)
- **배포**: GitHub Pages / Netlify

### 1.3 기술적 목표
1. **모바일 최적화**: 터치 이벤트, 긴 터치, 더블 탭 지원
2. **빠른 로딩**: 3초 이내 초기 로딩
3. **원활한 반응**: 셀 클릭 반응 0.1초 이내
4. **접근성**: 웹 접근성 표준 준수

---

## 2. 시스템 아키텍처

### 2.1 전체 구조

```text
┌─────────────────────────────────────────────────────┐
│                   Frontend Layer                     │
│  ┌───────────────┐  ┌───────────────┐  ┌──────────┐ │
│  │  Game Engine  │  │   UI Layer    │  │  Event   │ │
│  │               │  │               │  │ Handler  │ │
│  │  - Game Logic │  │  - DOM Update │  │          │ │
│  │  - Grid Mgmt  │  │  - Animation  │  │ - Touch  │ │
│  │  - State Mgmt │  │  - Responsive │  │ - Click  │ │
│  └───────────────┘  └───────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────┘
                         ▲
                         │
┌─────────────────────────────────────────────────────┐
│                  Data Layer                         │
│  ┌───────────────┐  ┌───────────────┐              │
│  │  LocalStorage │  │  Game State   │              │
│  │               │  │               │              │
│  │  - Best Times │  │  - Grid       │              │
│  │  - Settings   │  │  - Mines      │              │
│  └───────────────┘  └───────────────┘              │
└─────────────────────────────────────────────────────┘
```

### 2.2 주요 컴포넌트

#### 2.2.1 Game Engine
- 게임 로직
- 그리드 관리
- 상태 관리
- 지뢰 배치

#### 2.2.2 UI Layer
- DOM 업데이트
- 애니메이션
- 반응형 디자인

#### 2.2.3 Event Handler
- 터치 이벤트
- 클릭 이벤트
- 긴 터치 이벤트

---

## 3. 상세 기술 사양

### 3.1 Frontend

#### 3.1.1 HTML5
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>지뢰찾기</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div id="app">
        <header>
            <h1>지뢰찾기</h1>
            <div id="difficulty-selector">
                <button data-difficulty="beginner">초급</button>
                <button data-difficulty="intermediate">중급</button>
                <button data-difficulty="expert">고급</button>
            </div>
        </header>

        <div id="game-info">
            <div id="mines-count">💣 10</div>
            <div id="timer">000</div>
        </div>

        <div id="face-button">😊</div>

        <div id="grid"></div>

        <div id="controls">
            <button id="reset-btn">리셋</button>
            <button id="help-btn">도움말</button>
        </div>

        <div id="result-modal" class="modal hidden">
            <div class="modal-content">
                <h2 id="result-title"></h2>
                <div id="result-content"></div>
                <button id="close-modal">닫기</button>
            </div>
        </div>
    </div>
    <script src="js/game.js"></script>
</body>
</html>
```

#### 3.1.2 CSS3
```css
/* 기본 스타일 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Noto Sans KR', sans-serif;
    background-color: #f0f0f0;
    color: #333;
    display: flex;
    justify-content: center;
    min-height: 100vh;
}

#app {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    width: 100%;
    max-width: 600px;
}

/* 헤더 */
header {
    text-align: center;
    margin-bottom: 20px;
    width: 100%;
}

h1 {
    font-size: 2rem;
    margin-bottom: 15px;
}

#difficulty-selector {
    display: flex;
    gap: 10px;
    justify-content: center;
}

#difficulty-selector button {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    background-color: #4CAF50;
    color: white;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

#difficulty-selector button:hover {
    background-color: #45a049;
}

#difficulty-selector button.active {
    background-color: #2e7d32;
}

/* 게임 정보 */
#game-info {
    display: flex;
    justify-content: space-between;
    width: 100%;
    padding: 15px;
    background-color: #e0e0e0;
    border-radius: 10px;
    margin-bottom: 15px;
    font-size: 1.5rem;
    font-family: 'Courier New', monospace;
}

/* 표정 버튼 */
#face-button {
    font-size: 3rem;
    margin-bottom: 15px;
    cursor: pointer;
    user-select: none;
    transition: transform 0.1s;
}

#face-button:active {
    transform: scale(0.95);
}

/* 그리드 */
#grid {
    display: grid;
    gap: 2px;
    background-color: #999;
    padding: 5px;
    border-radius: 5px;
    margin-bottom: 20px;
}

.cell {
    background-color: #c0c0c0;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.2rem;
    font-weight: bold;
    cursor: pointer;
    user-select: none;
    transition: background-color 0.1s;
}

.cell:active {
    background-color: #a0a0a0;
}

.cell.open {
    background-color: #e0e0e0;
    cursor: default;
}

.cell.mine {
    background-color: #ffcccc;
}

.cell.flagged {
    background-color: #ffcc80;
}

.cell.questioned {
    background-color: #80d8ff;
}

/* 숫자 색상 */
.cell[data-number="1"] { color: #0000ff; }
.cell[data-number="2"] { color: #008000; }
.cell[data-number="3"] { color: #ff0000; }
.cell[data-number="4"] { color: #000080; }
.cell[data-number="5"] { color: #800000; }
.cell[data-number="6"] { color: #008080; }
.cell[data-number="7"] { color: #000000; }
.cell[data-number="8"] { color: #808080; }

/* 컨트롤 */
#controls {
    display: flex;
    gap: 10px;
}

#controls button {
    padding: 15px 30px;
    border: none;
    border-radius: 5px;
    background-color: #2196F3;
    color: white;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

#controls button:hover {
    background-color: #0b7dda;
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
    text-align: center;
}

/* 모바일 최적화 */
@media (max-width: 768px) {
    #grid {
        padding: 2px;
        gap: 1px;
    }

    .cell {
        font-size: 1rem;
    }

    h1 {
        font-size: 1.5rem;
    }

    #game-info {
        font-size: 1.2rem;
        padding: 10px;
    }

    #face-button {
        font-size: 2rem;
    }
}

@media (max-width: 480px) {
    .cell {
        font-size: 0.8rem;
    }

    h1 {
        font-size: 1.2rem;
    }

    #game-info {
        font-size: 1rem;
        padding: 8px;
    }
}
```

#### 3.1.3 JavaScript (ES6+)

##### 3.1.3.1 Game Engine
```javascript
class MinesweeperGame {
    constructor() {
        this.rows = 9;
        this.cols = 9;
        this.mines = 10;
        this.grid = [];
        this.gameOver = false;
        this.won = false;
        this.firstClick = true;
        this.flagsUsed = 0;
        this.seconds = 0;
        this.timerInterval = null;
        this.difficulty = 'beginner';
    }

    init(difficulty = 'beginner') {
        this.difficulty = difficulty;
        this.setupDifficulty(difficulty);
        this.reset();
    }

    setupDifficulty(difficulty) {
        switch (difficulty) {
            case 'beginner':
                this.rows = 9;
                this.cols = 9;
                this.mines = 10;
                break;
            case 'intermediate':
                this.rows = 16;
                this.cols = 16;
                this.mines = 40;
                break;
            case 'expert':
                this.rows = 16;
                this.cols = 30;
                this.mines = 99;
                break;
        }
    }

    reset() {
        this.grid = this.createGrid();
        this.gameOver = false;
        this.won = false;
        this.firstClick = true;
        this.flagsUsed = 0;
        this.seconds = 0;
        this.stopTimer();
        this.updateDisplay();
        this.renderGrid();
    }

    createGrid() {
        const grid = [];
        for (let row = 0; row < this.rows; row++) {
            const rowData = [];
            for (let col = 0; col < this.cols; col++) {
                rowData.push({
                    row,
                    col,
                    isMine: false,
                    isOpen: false,
                    isFlagged: false,
                    isQuestioned: false,
                    neighborMines: 0
                });
            }
            grid.push(rowData);
        }
        return grid;
    }

    placeMines(excludeRow, excludeCol) {
        let minesPlaced = 0;
        while (minesPlaced < this.mines) {
            const row = Math.floor(Math.random() * this.rows);
            const col = Math.floor(Math.random() * this.cols);

            // 첫 클릭 위치와 주변에는 지뢰를 배치하지 않음
            if (!this.grid[row][col].isMine &&
                !(Math.abs(row - excludeRow) <= 1 && Math.abs(col - excludeCol) <= 1)) {
                this.grid[row][col].isMine = true;
                minesPlaced++;
            }
        }

        // 주변 지뢰 수 계산
        this.calculateNeighborMines();
    }

    calculateNeighborMines() {
        for (let row = 0; row < this.rows; row++) {
            for (let col = 0; col < this.cols; col++) {
                if (!this.grid[row][col].isMine) {
                    this.grid[row][col].neighborMines = this.countNeighborMines(row, col);
                }
            }
        }
    }

    countNeighborMines(row, col) {
        let count = 0;
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                const newRow = row + dr;
                const newCol = col + dc;

                if (newRow >= 0 && newRow < this.rows &&
                    newCol >= 0 && newCol < this.cols &&
                    this.grid[newRow][newCol].isMine) {
                    count++;
                }
            }
        }
        return count;
    }

    openCell(row, col) {
        if (this.gameOver || this.won) return;
        if (this.grid[row][col].isOpen) return;
        if (this.grid[row][col].isFlagged) return;

        // 첫 클릭은 안전
        if (this.firstClick) {
            this.firstClick = false;
            this.startTimer();
            this.placeMines(row, col);
        }

        const cell = this.grid[row][col];
        cell.isOpen = true;

        // 지뢰 클릭
        if (cell.isMine) {
            this.gameOver = true;
            this.stopTimer();
            this.revealMines();
            this.showGameOver(false);
            return;
        }

        // 0인 경우 주변 셀 자동 열기
        if (cell.neighborMines === 0) {
            this.openNeighbors(row, col);
        }

        // 승리 확인
        this.checkWin();
        this.updateDisplay();
    }

    openNeighbors(row, col) {
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                const newRow = row + dr;
                const newCol = col + dc;

                if (newRow >= 0 && newRow < this.rows &&
                    newCol >= 0 && newCol < this.cols &&
                    !this.grid[newRow][newCol].isOpen &&
                    !this.grid[newRow][newCol].isFlagged) {
                    this.openCell(newRow, newCol);
                }
            }
        }
    }

    toggleFlag(row, col) {
        if (this.gameOver || this.won) return;
        if (this.grid[row][col].isOpen) return;

        const cell = this.grid[row][col];

        if (cell.isFlagged) {
            cell.isFlagged = false;
            cell.isQuestioned = true;
            this.flagsUsed--;
        } else if (cell.isQuestioned) {
            cell.isQuestioned = false;
        } else {
            cell.isFlagged = true;
            this.flagsUsed++;
        }

        this.updateDisplay();
    }

    revealMines() {
        for (let row = 0; row < this.rows; row++) {
            for (let col = 0; col < this.cols; col++) {
                if (this.grid[row][col].isMine) {
                    this.grid[row][col].isOpen = true;
                }
            }
        }
    }

    checkWin() {
        let openCount = 0;
        for (let row = 0; row < this.rows; row++) {
            for (let col = 0; col < this.cols; col++) {
                if (this.grid[row][col].isOpen && !this.grid[row][col].isMine) {
                    openCount++;
                }
            }
        }

        const totalSafeCells = this.rows * this.cols - this.mines;
        if (openCount === totalSafeCells) {
            this.won = true;
            this.gameOver = true;
            this.stopTimer();
            this.showGameOver(true);
        }
    }

    startTimer() {
        this.timerInterval = setInterval(() => {
            this.seconds++;
            if (this.seconds > 999) {
                this.stopTimer();
            }
            this.updateDisplay();
        }, 1000);
    }

    stopTimer() {
        if (this.timerInterval) {
            clearInterval(this.timerInterval);
            this.timerInterval = null;
        }
    }

    updateDisplay() {
        const minesCount = document.getElementById('mines-count');
        const timer = document.getElementById('timer');
        const faceButton = document.getElementById('face-button');

        minesCount.textContent = `💣 ${this.mines - this.flagsUsed}`;
        timer.textContent = this.seconds.toString().padStart(3, '0');

        if (this.gameOver) {
            faceButton.textContent = this.won ? '🥺' : '😵';
        } else {
            faceButton.textContent = '😊';
        }
    }

    renderGrid() {
        const gridElement = document.getElementById('grid');
        gridElement.innerHTML = '';
        gridElement.style.gridTemplateColumns = `repeat(${this.cols}, 1fr)`;

        for (let row = 0; row < this.rows; row++) {
            for (let col = 0; col < this.cols; col++) {
                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.dataset.row = row;
                cell.dataset.col = col;

                if (this.grid[row][col].isOpen) {
                    cell.classList.add('open');
                    if (this.grid[row][col].isMine) {
                        cell.classList.add('mine');
                        cell.textContent = '💣';
                    } else if (this.grid[row][col].neighborMines > 0) {
                        cell.textContent = this.grid[row][col].neighborMines;
                        cell.dataset.number = this.grid[row][col].neighborMines;
                    }
                } else if (this.grid[row][col].isFlagged) {
                    cell.classList.add('flagged');
                    cell.textContent = '🚩';
                } else if (this.grid[row][col].isQuestioned) {
                    cell.classList.add('questioned');
                    cell.textContent = '❓';
                }

                gridElement.appendChild(cell);
            }
        }
    }

    showGameOver(won) {
        this.renderGrid();

        const modal = document.getElementById('result-modal');
        const title = document.getElementById('result-title');
        const content = document.getElementById('result-content');

        title.textContent = won ? '🎉 승리!' : '💥 게임 오버';

        content.innerHTML = `
            <p>시간: ${this.seconds}초</p>
            <p>난이도: ${this.difficulty}</p>
            ${won ? '<p>축하합니다! 🎉</p>' : '<p>다시 도전하세요! 💪</p>'}
        `;

        modal.classList.remove('hidden');

        // 최고 기록 저장
        if (won) {
            this.saveBestTime();
        }
    }

    saveBestTime() {
        const key = `minesweeper_${this.difficulty}_best`;
        const currentBest = localStorage.getItem(key);

        if (!currentBest || this.seconds < parseInt(currentBest)) {
            localStorage.setItem(key, this.seconds);
            alert(`🎉 새로운 최고 기록! ${this.seconds}초`);
        }
    }

    getBestTime(difficulty) {
        const key = `minesweeper_${difficulty}_best`;
        const bestTime = localStorage.getItem(key);
        return bestTime ? parseInt(bestTime) : null;
    }
}
```

##### 3.1.3.2 Event Handler
```javascript
class EventHandler {
    constructor(game) {
        this.game = game;
        this.longPressTimeout = null;
        this.longPressDuration = 500; // 500ms
        this.init();
    }

    init() {
        // 난이도 버튼
        document.querySelectorAll('#difficulty-selector button').forEach(button => {
            button.addEventListener('click', (e) => {
                const difficulty = e.target.dataset.difficulty;
                this.game.init(difficulty);
                this.updateActiveButton(difficulty);
            });
        });

        // 리셋 버튼
        document.getElementById('reset-btn').addEventListener('click', () => {
            this.game.reset();
        });

        // 도움말 버튼
        document.getElementById('help-btn').addEventListener('click', () => {
            this.showHelp();
        });

        // 표정 버튼
        document.getElementById('face-button').addEventListener('click', () => {
            this.game.reset();
        });

        // 그리드 이벤트
        const grid = document.getElementById('grid');
        grid.addEventListener('click', (e) => {
            if (e.target.classList.contains('cell')) {
                const row = parseInt(e.target.dataset.row);
                const col = parseInt(e.target.dataset.col);
                this.handleLeftClick(row, col);
            }
        });

        // 긴 터치 (우클릭 대체)
        grid.addEventListener('touchstart', (e) => {
            if (e.target.classList.contains('cell')) {
                this.longPressTimeout = setTimeout(() => {
                    const row = parseInt(e.target.dataset.row);
                    const col = parseInt(e.target.dataset.col);
                    this.game.toggleFlag(row, col);
                    this.game.renderGrid();
                    if (navigator.vibrate) {
                        navigator.vibrate(100);
                    }
                }, this.longPressDuration);
            }
        });

        grid.addEventListener('touchend', () => {
            if (this.longPressTimeout) {
                clearTimeout(this.longPressTimeout);
                this.longPressTimeout = null;
            }
        });

        grid.addEventListener('touchmove', () => {
            if (this.longPressTimeout) {
                clearTimeout(this.longPressTimeout);
                this.longPressTimeout = null;
            }
        });

        // 모바일에서 더블 탭 자동 열기
        let lastTap = 0;
        grid.addEventListener('touchend', (e) => {
            if (e.target.classList.contains('cell') &&
                this.game.grid[e.target.dataset.row][e.target.dataset.col].isOpen) {
                const currentTime = new Date().getTime();
                const tapLength = currentTime - lastTap;
                if (tapLength < 300 && tapLength > 0) {
                    const row = parseInt(e.target.dataset.row);
                    const col = parseInt(e.target.dataset.col);
                    this.handleAutoOpen(row, col);
                    e.preventDefault();
                }
                lastTap = currentTime;
            }
        });

        // 모달 닫기
        document.getElementById('close-modal').addEventListener('click', () => {
            document.getElementById('result-modal').classList.add('hidden');
        });
    }

    handleLeftClick(row, col) {
        this.game.openCell(row, col);
        this.game.renderGrid();
    }

    handleAutoOpen(row, col) {
        if (this.game.gameOver || this.game.won) return;

        const cell = this.game.grid[row][col];
        if (!cell.isOpen || cell.isMine || cell.neighborMines === 0) return;

        // 주변 깃발 수 계산
        let flagCount = 0;
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                const newRow = row + dr;
                const newCol = col + dc;

                if (newRow >= 0 && newRow < this.game.rows &&
                    newCol >= 0 && newCol < this.game.cols &&
                    this.game.grid[newRow][newCol].isFlagged) {
                    flagCount++;
                }
            }
        }

        // 깃발 수와 주변 지뢰 수가 일치하면 주변 셀 열기
        if (flagCount === cell.neighborMines) {
            for (let dr = -1; dr <= 1; dr++) {
                for (let dc = -1; dc <= 1; dc++) {
                    const newRow = row + dr;
                    const newCol = col + dc;

                    if (newRow >= 0 && newRow < this.game.rows &&
                        newCol >= 0 && newCol < this.game.cols &&
                        !this.game.grid[newRow][newCol].isFlagged &&
                        !this.game.grid[newRow][newCol].isOpen) {
                        this.game.openCell(newRow, newCol);
                    }
                }
            }
            this.game.renderGrid();
        }
    }

    updateActiveButton(difficulty) {
        document.querySelectorAll('#difficulty-selector button').forEach(button => {
            button.classList.remove('active');
            if (button.dataset.difficulty === difficulty) {
                button.classList.add('active');
            }
        });

        // 최고 기록 표시
        const bestTime = this.game.getBestTime(difficulty);
        if (bestTime) {
            alert(`최고 기록: ${bestTime}초`);
        }
    }

    showHelp() {
        const modal = document.getElementById('result-modal');
        const title = document.getElementById('result-title');
        const content = document.getElementById('result-content');

        title.textContent = '도움말';

        content.innerHTML = `
            <h3>게임 조작</h3>
            <p>📱 탭: 셀 열기</p>
            <p>📱 긴 터치 (500ms): 깃발 표시</p>
            <p>📱 열린 셀 더블 탭: 주변 셀 자동 열기</p>
            <h3>PC 조작</h3>
            <p>🖱️ 좌클릭: 셀 열기</p>
            <p>🖱️ 우클릭: 깃발 표시</p>
            <p>🖱️ 더블 클릭: 주변 셀 자동 열기</p>
            <h3>게임 목표</h3>
            <p>지뢰를 제외한 모든 셀을 열면 승리합니다!</p>
        `;

        modal.classList.remove('hidden');
    }
}
```

##### 3.1.3.3 메인 실행
```javascript
// 초기화
document.addEventListener('DOMContentLoaded', () => {
    const game = new MinesweeperGame();
    const eventHandler = new EventHandler(game);

    // 기본 난이도로 게임 시작
    game.init('beginner');
    eventHandler.updateActiveButton('beginner');
});
```

---

## 4. 모바일 최적화

### 4.1 터치 이벤트
- **탭**: 셀 열기
- **긴 터치 (500ms)**: 깃발 표시
- **더블 탭 (300ms 이내)**: 주변 셀 자동 열기

### 4.2 뷰포트 설정
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
```

### 4.3 진동 피드백
```javascript
if (navigator.vibrate) {
    navigator.vibrate(100); // 100ms 진동
}
```

### 4.4 셀 크기 조정
- **초급**: 40px
- **중급**: 30px
- **고급**: 25px

### 4.5 화면 크기에 따른 조정
- CSS Grid 사용
- max-width: 600px
- 반응적 크기 조정

---

## 5. 성능 최적화

### 5.1 그리드 렌더링 최적화
```javascript
// 필요한 셀만 업데이트
updateCell(row, col) {
    const cellElement = document.querySelector(`[data-row="${row}"][data-col="${col}"]`);
    // 셀 업데이트 로직
}
```

### 5.2 이벤트 위임
```javascript
// 그리드 컨테이너에 단일 이벤트 리스너 추가
grid.addEventListener('click', (e) => {
    if (e.target.classList.contains('cell')) {
        // 셀 클릭 처리
    }
});
```

### 5.3 타이머 최적화
```javascript
// requestAnimationFrame 사용
startTimer() {
    let startTime = Date.now();
    const updateTimer = () => {
        const elapsed = Math.floor((Date.now() - startTime) / 1000);
        this.seconds = elapsed;
        this.updateDisplay();
        
        if (elapsed < 1000) {
            requestAnimationFrame(updateTimer);
        }
    };
    requestAnimationFrame(updateTimer);
}
```

---

## 6. 보안 및 개인정보

### 6.1 보안 고려사항
- **XSS 방지**: 사용자 입력 없음
- **HTTPS 사용**: 배포 시 HTTPS 사용
- **Content Security Policy**: 불필요 (외부 스크립트 없음)

### 6.2 개인정보 처리
- **데이터 수집 최소화**: 최고 기록만 LocalStorage에 저장
- **서버 데이터 없음**: 모든 데이터 클라이언트 측에서 처리

---

## 7. 테스트

### 7.1 단위 테스트
```javascript
// 그리드 생성 테스트
test('createGrid', () => {
    game.init('beginner');
    expect(game.grid.length).toBe(9);
    expect(game.grid[0].length).toBe(9);
});

// 지뢰 배치 테스트
test('placeMines', () => {
    game.placeMines(4, 4);
    let mineCount = 0;
    for (let row = 0; row < game.rows; row++) {
        for (let col = 0; col < game.cols; col++) {
            if (game.grid[row][col].isMine) mineCount++;
        }
    }
    expect(mineCount).toBe(10);
});
```

### 7.2 통합 테스트
- 게임 전체 플레이 테스트
- 승리/패배 로직 테스트
- 타이머 테스트

### 7.3 모바일 테스트
- iOS Safari 테스트
- Android Chrome 테스트
- 다양한 화면 크기 테스트
- 터치 이벤트 테스트

---

## 8. 배포

### 8.1 GitHub Pages 배포
```bash
# GitHub 리포지토리 생성
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/minesweeper.git
git push -u origin main

# GitHub Pages 설정
# Settings → Pages → Source: main branch → Save
```

### 8.2 Netlify 배포
```bash
# Netlify CLI 설치
npm install -g netlify-cli

# 배포
netlify deploy --prod
```

---

## 9. 성능 모니터링

### 9.1 웹 성능
- **Lighthouse 점수**: 95점 이상
- **FCP (First Contentful Paint)**: 1초 이내
- **TTI (Time to Interactive)**: 2초 이내
- **CLS (Cumulative Layout Shift)**: 0.1 이하

### 9.2 사용자 분석
- Google Analytics 통합 (선택사항)
- 이벤트 추적 (게임 시작, 승리, 패배)
- 난이도 분석

---

## 10. 개발 일정

### 10.1 Week 1: MVP 개발
- Day 1-2: 기본 게임 로직 구현
- Day 3-4: UI/UX 구현
- Day 5: 테스트 및 버그 수정

### 10.2 Week 2: 업데이트
- Day 1-2: 모바일 최적화
- Day 3-4: 최고 기록 저장
- Day 5: 테스트 및 배포

### 10.3 Week 3: 확장
- Day 1-2: 커스텀 난이도
- Day 3-4: 테마 추가
- Day 5: 마케팅

---

## 11. 기술 우선순위

### 11.1 MVP (최소 기능 제품)
- [x] 기본 게임 플레이
- [x] 3개 난이도
- [x] 타이머
- [x] 모바일 지원

### 11.2 Phase 2 (업데이트)
- [ ] 긴 터치 깃발
- [ ] 더블 탭 자동 열기
- [ ] 최고 기록 저장
- [ ] 랭킹 시스템

### 11.3 Phase 3 (확장)
- [ ] 커스텀 난이도
- [ ] 다양한 테마
- [ ] 소셜 공유
- [ ] 오프라인 지원

---

*작성일: 2026-04-13*
*작성자*: nanobot
*버전*: 1.0
