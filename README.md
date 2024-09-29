<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>BlockGame</title>
    <style>
        body { background: #000; margin: 0; overflow: hidden; }
        canvas { display: block; margin: 0 auto; background: #111; }
    </style>
</head>
<body>
<canvas id="game" width="200" height="400"></canvas>
<script>
    const canvas = document.getElementById('game');
    const context = canvas.getContext('2d');
    const grid = 20;
    const shapes = {
        'I': [[1, 1, 1, 1]],
        'J': [[1, 0, 0], [1, 1, 1]],
        'L': [[0, 0, 1], [1, 1, 1]],
        'O': [[1, 1], [1, 1]],
        'S': [[0, 1, 1], [1, 1, 0]],
        'T': [[0, 1, 0], [1, 1, 1]],
        'Z': [[1, 1, 0], [0, 1, 1]]
    };
    let field = [];
    for (let row = -2; row < 20; row++) {
        field[row] = [];
        for (let col = 0; col < 10; col++) {
            field[row][col] = 0;
        }
    }
    let sequence = [];
    function generateSequence() {
        const pieces = ['I', 'J', 'L', 'O', 'S', 'T', 'Z'];
        while (pieces.length) {
            const rand = Math.floor(Math.random() * pieces.length);
            const name = pieces.splice(rand, 1)[0];
            sequence.push(name);
        }
    }
    const colors = {
        'I': 'cyan',
        'O': 'yellow',
        'T': 'purple',
        'S': 'green',
        'Z': 'red',
        'J': 'blue',
        'L': 'orange'
    };
    let count = 0;
    let piece = getNextPiece();
    let rAF = null;
    let gameOver = false;
    function getNextPiece() {
        if (sequence.length === 0) {
            generateSequence();
        }
        const name = sequence.pop();
        const matrix = shapes[name];
        const col = Math.floor(field[0].length / 2) - Math.ceil(matrix[0].length / 2);
        const row = -2;
        return {
            name: name,
            matrix: matrix,
            row: row,
            col: col
        };
    }
    function rotate(matrix) {
        return matrix[0].map((_, index) =>
            matrix.map(row => row[index]).reverse()
        );
    }
    function isValidMove(matrix, cellRow, cellCol) {
        for (let row = 0; row < matrix.length; row++) {
            for (let col = 0; col < matrix[row].length; col++) {
                if (matrix[row][col] && (
                    cellCol + col < 0 ||
                    cellCol + col >= field[0].length ||
                    cellRow + row >= field.length ||
                    field[cellRow + row][cellCol + col])
                ) {
                    return false;
                }
            }
        }
        return true;
    }
    function placePiece() {
        for (let row = 0; row < piece.matrix.length; row++) {
            for (let col = 0; col < piece.matrix[row].length; col++) {
                if (piece.matrix[row][col]) {
                    if (piece.row + row < 0) {
                        return showGameOver();
                    }
                    field[piece.row + row][piece.col + col] = piece.name;
                }
            }
        }
        for (let row = field.length - 1; row >= 0; ) {
            if (field[row].every(cell => !!cell)) {
                for (let r = row; r >= 0; r--) {
                    for (let c = 0; c < field[r].length; c++) {
                        field[r][c] = field[r - 1][c];
                    }
                }
            } else {
                row--;
            }
        }
        piece = getNextPiece();
    }
    function showGameOver() {
        cancelAnimationFrame(rAF);
        gameOver = true;
        context.fillStyle = 'black';
        context.globalAlpha = 0.75;
        context.fillRect(0, canvas.height / 2 - 30, canvas.width, 60);
        context.globalAlpha = 1;
        context.fillStyle = 'white';
        context.font = '36px monospace';
        context.textAlign = 'center';
        context.textBaseline = 'middle';
        context.fillText('GAME OVER', canvas.width / 2, canvas.height / 2);
    }
    function loop() {
        rAF = requestAnimationFrame(loop);
        context.clearRect(0, 0, canvas.width, canvas.height);
        for (let row = 0; row < field.length; row++) {
            for (let col = 0; col < field[row].length; col++) {
                if (field[row][col]) {
                    const name = field[row][col];
                    context.fillStyle = colors[name];
                    context.fillRect(col * grid, row * grid, grid - 1, grid - 1);
                }
            }
        }
        if (piece) {
            if (++count > 35) {
                piece.row++;
                count = 0;
                if (!isValidMove(piece.matrix, piece.row, piece.col)) {
                    piece.row--;
                    placePiece();
                }
            }
            context.fillStyle = colors[piece.name];
            for (let row = 0; row < piece.matrix.length; row++) {
                for (let col = 0; col < piece.matrix[row].length; col++) {
                    if (piece.matrix[row][col]) {
                        context.fillRect((piece.col + col) * grid, (piece.row + row) * grid, grid - 1, grid - 1);
                    }
                }
            }
        }
    }
    document.addEventListener('keydown', function(e) {
        if (gameOver) return;
        if (e.key === 'ArrowLeft' || e.key === 'Left') {
            const col = piece.col - 1;
            if (isValidMove(piece.matrix, piece.row, col)) {
                piece.col = col;
            }
        } else if (e.key === 'ArrowRight' || e.key === 'Right') {
            const col = piece.col + 1;
            if (isValidMove(piece.matrix, piece.row, col)) {
                piece.col = col;
            }
        } else if (e.key === 'ArrowUp' || e.key === 'Up') {
            const matrix = rotate(piece.matrix);
            if (isValidMove(matrix, piece.row, piece.col)) {
                piece.matrix = matrix;
            }
        } else if (e.key === 'ArrowDown' || e.key === 'Down') {
            const row = piece.row + 1;
            if (!isValidMove(piece.matrix, row, piece.col)) {
                piece.row = row - 1;
                placePiece();
                return;
            }
            piece.row = row;
        }
    });
    rAF = requestAnimationFrame(loop);
</script>
</body>
</html>
