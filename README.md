# TeoDev
Cờ Caro
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cờ Caro - Đấu với máy nâng cao</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f5f5f5;
            margin: 0;
            padding: 20px;
        }
        
        h1 {
            color: #333;
            margin-bottom: 10px;
        }
        
        .game-info {
            margin-bottom: 20px;
            font-size: 18px;
        }
        
        .board {
            display: grid;
            grid-template-columns: repeat(10, 40px);
            grid-template-rows: repeat(10, 40px);
            gap: 1px;
            background-color: #333;
            border: 2px solid #333;
        }
        
        .cell {
            width: 40px;
            height: 40px;
            background-color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 24px;
            font-weight: bold;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        
        .cell:hover {
            background-color: #f0f0f0;
        }
        
        .cell.X {
            color: #e74c3c;
        }
        
        .cell.O {
            color: #3498db;
        }
        
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        
        .modal-content {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }
        
        .modal h2 {
            margin-top: 0;
            color: #333;
        }
        
        .modal button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 16px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 20px;
        }
        
        .modal button:hover {
            background-color: #2980b9;
        }
        
        .difficulty {
            margin: 15px 0;
        }
        
        .difficulty button {
            padding: 8px 15px;
            margin: 0 5px;
            background-color: #2ecc71;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        
        .difficulty button.active {
            background-color: #27ae60;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Cờ Caro</h1>
    <div class="difficulty">
        <button id="easy-btn" class="active">Dễ</button>
        <button id="medium-btn">Trung bình</button>
        <button id="hard-btn">Khó</button>
    </div>
    <div class="game-info">
        Bạn là: <span id="player-symbol"></span> | Lượt đi: <span id="current-turn"></span>
    </div>
    <div class="board" id="board"></div>
    
    <div class="modal" id="modal">
        <div class="modal-content">
            <h2 id="modal-message"></h2>
            <button id="restart-button">Chơi lại</button>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const boardSize = 10;
            const boardElement = document.getElementById('board');
            const playerSymbolElement = document.getElementById('player-symbol');
            const currentTurnElement = document.getElementById('current-turn');
            const modal = document.getElementById('modal');
            const modalMessage = document.getElementById('modal-message');
            const restartButton = document.getElementById('restart-button');
            const easyBtn = document.getElementById('easy-btn');
            const mediumBtn = document.getElementById('medium-btn');
            const hardBtn = document.getElementById('hard-btn');
            
            let board = [];
            let currentPlayer = 'X';
            let playerSymbol = Math.random() < 0.5 ? 'X' : 'O';
            let computerSymbol = playerSymbol === 'X' ? 'O' : 'X';
            let gameOver = false;
            let difficulty = 'easy'; // 'easy', 'medium', 'hard'
            
            // Khởi tạo bảng
            function initBoard() {
                board = Array(boardSize).fill().map(() => Array(boardSize).fill(''));
                boardElement.innerHTML = '';
                
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j < boardSize; j++) {
                        const cell = document.createElement('div');
                        cell.className = 'cell';
                        cell.dataset.row = i;
                        cell.dataset.col = j;
                        cell.addEventListener('click', handleCellClick);
                        boardElement.appendChild(cell);
                    }
                }
                
                updateGameInfo();
                
                // Nếu máy đi trước
                if (currentPlayer === computerSymbol) {
                    setTimeout(computerMove, 500);
                }
            }
            
            // Cập nhật thông tin game
            function updateGameInfo() {
                playerSymbolElement.textContent = playerSymbol;
                currentTurnElement.textContent = currentPlayer === playerSymbol ? 'Bạn' : 'Máy';
                currentTurnElement.style.color = currentPlayer === 'X' ? '#e74c3c' : '#3498db';
            }
            
            // Xử lý khi click vào ô
            function handleCellClick(e) {
                if (gameOver || currentPlayer !== playerSymbol) return;
                
                const row = parseInt(e.target.dataset.row);
                const col = parseInt(e.target.dataset.col);
                
                if (board[row][col] !== '') return;
                
                makeMove(row, col, playerSymbol);
                
                if (!gameOver) {
                    setTimeout(computerMove, 500);
                }
            }
            
            // Thực hiện nước đi
            function makeMove(row, col, symbol) {
                board[row][col] = symbol;
                const cell = document.querySelector(`.cell[data-row="${row}"][data-col="${col}"]`);
                cell.textContent = symbol;
                cell.classList.add(symbol);
                
                if (checkWin(row, col, symbol)) {
                    gameOver = true;
                    const winner = symbol === playerSymbol ? 'Bạn' : 'Máy';
                    showModal(`${winner} đã chiến thắng!`);
                    return;
                }
                
                if (checkDraw()) {
                    gameOver = true;
                    showModal('Hòa! Không còn nước đi nào.');
                    return;
                }
                
                currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
                updateGameInfo();
            }
            
            // Máy thực hiện nước đi với AI thông minh hơn
            function computerMove() {
                if (gameOver) return;
                
                let move;
                
                switch(difficulty) {
                    case 'easy':
                        move = findRandomMove();
                        break;
                    case 'medium':
                        // Ưu tiên tấn công hoặc phòng thủ cơ bản
                        move = findWinningMove(computerSymbol) || 
                               findWinningMove(playerSymbol) || 
                               findRandomMove();
                        break;
                    case 'hard':
                        // AI mạnh với đánh giá nước đi
                        move = findBestMove();
                        break;
                    default:
                        move = findRandomMove();
                }
                
                if (move) {
                    makeMove(move.row, move.col, computerSymbol);
                }
            }
            
            // Tìm nước đi tốt nhất (AI mạnh)
            function findBestMove() {
                // 1. Kiểm tra nếu có thể thắng ngay
                const winningMove = findWinningMove(computerSymbol);
                if (winningMove) return winningMove;
                
                // 2. Chặn nếu người chơi có thể thắng
                const blockingMove = findWinningMove(playerSymbol);
                if (blockingMove) return blockingMove;
                
                // 3. Tìm các mẫu cờ nguy hiểm để tấn công hoặc phòng thủ
                const attackPatterns = [
                    { pattern: [computerSymbol, computerSymbol, '', computerSymbol], score: 1000 },
                    { pattern: [computerSymbol, '', computerSymbol, computerSymbol], score: 1000 },
                    { pattern: ['', computerSymbol, computerSymbol, computerSymbol, ''], score: 800 },
                    { pattern: [playerSymbol, playerSymbol, '', playerSymbol], score: 900 },
                    { pattern: [playerSymbol, '', playerSymbol, playerSymbol], score: 900 },
                    { pattern: ['', playerSymbol, playerSymbol, playerSymbol, ''], score: 700 },
                    { pattern: [computerSymbol, computerSymbol, ''], score: 100 },
                    { pattern: [computerSymbol, '', computerSymbol], score: 100 },
                    { pattern: ['', computerSymbol, computerSymbol], score: 100 },
                    { pattern: [playerSymbol, playerSymbol, ''], score: 90 },
                    { pattern: [playerSymbol, '', playerSymbol], score: 90 },
                    { pattern: ['', playerSymbol, playerSymbol], score: 90 }
                ];
                
                let bestScore = -1;
                let bestMove = null;
                
                // Duyệt qua tất cả các ô trống để đánh giá
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j < boardSize; j++) {
                        if (board[i][j] === '') {
                            let score = evaluatePosition(i, j, attackPatterns);
                            
                            // Ưu tiên ô ở trung tâm và gần các nước đi khác
                            const centerDist = Math.abs(i - boardSize/2) + Math.abs(j - boardSize/2);
                            score += (boardSize - centerDist) * 5;
                            
                            if (score > bestScore) {
                                bestScore = score;
                                bestMove = { row: i, col: j };
                            }
                        }
                    }
                }
                
                return bestMove || findRandomMove();
            }
            
            // Đánh giá điểm số của một vị trí
            function evaluatePosition(row, col, patterns) {
                let totalScore = 0;
                
                // Kiểm tra 4 hướng: ngang, dọc, chéo chính, chéo phụ
                const directions = [
                    { dr: 0, dc: 1 },  // ngang
                    { dr: 1, dc: 0 },  // dọc
                    { dr: 1, dc: 1 },   // chéo chính
                    { dr: 1, dc: -1 }  // chéo phụ
                ];
                
                for (const dir of directions) {
                    for (const pattern of patterns) {
                        const patternLength = pattern.pattern.length;
                        
                        // Kiểm tra tất cả các vị trí có thể chứa mẫu này
                        for (let k = 0; k <= patternLength; k++) {
                            let match = true;
                            
                            for (let l = 0; l < patternLength; l++) {
                                const r = row + dir.dr * (l - k);
                                const c = col + dir.dc * (l - k);
                                
                                if (r < 0 || r >= boardSize || c < 0 || c >= boardSize) {
                                    match = false;
                                    break;
                                }
                                
                                const cellValue = (r === row && c === col) ? '' : board[r][c];
                                if (pattern.pattern[l] !== '' && cellValue !== pattern.pattern[l]) {
                                    match = false;
                                    break;
                                }
                            }
                            
                            if (match) {
                                totalScore += pattern.score;
                                break;
                            }
                        }
                    }
                }
                
                return totalScore;
            }
            
            // Tìm nước đi để thắng hoặc chặn
            function findWinningMove(symbol) {
                // Kiểm tra các hàng
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j <= boardSize - 5; j++) {
                        let count = 0;
                        let empty = null;
                        for (let k = 0; k < 5; k++) {
                            if (board[i][j + k] === symbol) {
                                count++;
                            } else if (board[i][j + k] === '') {
                                empty = { row: i, col: j + k };
                            } else {
                                count = 0;
                                break;
                            }
                        }
                        if (count === 4 && empty) {
                            return empty;
                        }
                    }
                }
                
                // Kiểm tra các cột
                for (let j = 0; j < boardSize; j++) {
                    for (let i = 0; i <= boardSize - 5; i++) {
                        let count = 0;
                        let empty = null;
                        for (let k = 0; k < 5; k++) {
                            if (board[i + k][j] === symbol) {
                                count++;
                            } else if (board[i + k][j] === '') {
                                empty = { row: i + k, col: j };
                            } else {
                                count = 0;
                                break;
                            }
                        }
                        if (count === 4 && empty) {
                            return empty;
                        }
                    }
                }
                
                // Kiểm tra đường chéo chính
                for (let i = 0; i <= boardSize - 5; i++) {
                    for (let j = 0; j <= boardSize - 5; j++) {
                        let count = 0;
                        let empty = null;
                        for (let k = 0; k < 5; k++) {
                            if (board[i + k][j + k] === symbol) {
                                count++;
                            } else if (board[i + k][j + k] === '') {
                                empty = { row: i + k, col: j + k };
                            } else {
                                count = 0;
                                break;
                            }
                        }
                        if (count === 4 && empty) {
                            return empty;
                        }
                    }
                }
                
                // Kiểm tra đường chéo phụ
                for (let i = 0; i <= boardSize - 5; i++) {
                    for (let j = 4; j < boardSize; j++) {
                        let count = 0;
                        let empty = null;
                        for (let k = 0; k < 5; k++) {
                            if (board[i + k][j - k] === symbol) {
                                count++;
                            } else if (board[i + k][j - k] === '') {
                                empty = { row: i + k, col: j - k };
                            } else {
                                count = 0;
                                break;
                            }
                        }
                        if (count === 4 && empty) {
                            return empty;
                        }
                    }
                }
                
                return null;
            }
            
            // Tìm nước đi ngẫu nhiên
            function findRandomMove() {
                const emptyCells = [];
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j < boardSize; j++) {
                        if (board[i][j] === '') {
                            emptyCells.push({ row: i, col: j });
                        }
                    }
                }
                
                if (emptyCells.length > 0) {
                    return emptyCells[Math.floor(Math.random() * emptyCells.length)];
                }
                return null;
            }
            
            // Kiểm tra thắng
            function checkWin(row, col, symbol) {
                // Kiểm tra hàng
                let count = 0;
                for (let j = Math.max(0, col - 4); j <= Math.min(boardSize - 1, col + 4); j++) {
                    if (board[row][j] === symbol) {
                        count++;
                        if (count === 5) return true;
                    } else {
                        count = 0;
                    }
                }
                
                // Kiểm tra cột
                count = 0;
                for (let i = Math.max(0, row - 4); i <= Math.min(boardSize - 1, row + 4); i++) {
                    if (board[i][col] === symbol) {
                        count++;
                        if (count === 5) return true;
                    } else {
                        count = 0;
                    }
                }
                
                // Kiểm tra đường chéo chính
                count = 0;
                let i = row - Math.min(row, col);
                let j = col - Math.min(row, col);
                while (i < boardSize && j < boardSize) {
                    if (board[i][j] === symbol) {
                        count++;
                        if (count === 5) return true;
                    } else {
                        count = 0;
                    }
                    i++;
                    j++;
                }
                
                // Kiểm tra đường chéo phụ
                count = 0;
                i = row - Math.min(row, boardSize - 1 - col);
                j = col + Math.min(row, boardSize - 1 - col);
                while (i < boardSize && j >= 0) {
                    if (board[i][j] === symbol) {
                        count++;
                        if (count === 5) return true;
                    } else {
                        count = 0;
                    }
                    i++;
                    j--;
                }
                
                return false;
            }
            
            // Kiểm tra hòa
            function checkDraw() {
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j < boardSize; j++) {
                        if (board[i][j] === '') {
                            return false;
                        }
                    }
                }
                return true;
            }
            
            // Hiển thị modal thông báo
            function showModal(message) {
                modalMessage.textContent = message;
                modal.style.display = 'flex';
            }
            
            // Khởi động lại game
            restartButton.addEventListener('click', () => {
                modal.style.display = 'none';
                gameOver = false;
                currentPlayer = 'X'; // X luôn đi trước
                playerSymbol = Math.random() < 0.5 ? 'X' : 'O';
                computerSymbol = playerSymbol === 'X' ? 'O' : 'X';
                initBoard();
            });
            
            // Thiết lập độ khó
            easyBtn.addEventListener('click', () => {
                difficulty = 'easy';
                easyBtn.classList.add('active');
                mediumBtn.classList.remove('active');
                hardBtn.classList.remove('active');
            });
            
            mediumBtn.addEventListener('click', () => {
                difficulty = 'medium';
                easyBtn.classList.remove('active');
                mediumBtn.classList.add('active');
                hardBtn.classList.remove('active');
            });
            
            hardBtn.addEventListener('click', () => {
                difficulty = 'hard';
                easyBtn.classList.remove('active');
                mediumBtn.classList.remove('active');
                hardBtn.classList.add('active');
            });
            
            // Bắt đầu game
            initBoard();
        });
    </script>
</body>
</html>
