const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

// Game constants
const PADDLE_WIDTH = 10;
const PADDLE_HEIGHT = 80;
const BALL_SIZE = 12;

let leftPaddle = {
  x: 20,
  y: canvas.height / 2 - PADDLE_HEIGHT / 2,
  width: PADDLE_WIDTH,
  height: PADDLE_HEIGHT,
  color: '#fff'
};

let rightPaddle = {
  x: canvas.width - 20 - PADDLE_WIDTH,
  y: canvas.height / 2 - PADDLE_HEIGHT / 2,
  width: PADDLE_WIDTH,
  height: PADDLE_HEIGHT,
  color: '#fff',
  speed: 4
};

let ball = {
  x: canvas.width / 2 - BALL_SIZE / 2,
  y: canvas.height / 2 - BALL_SIZE / 2,
  size: BALL_SIZE,
  speedX: 5 * (Math.random() < 0.5 ? 1 : -1),
  speedY: 3 * (Math.random() < 0.5 ? 1 : -1),
  color: '#fff'
};

function drawRect(x, y, w, h, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x, y, w, h);
}

function drawBall(ball) {
  ctx.fillStyle = ball.color;
  ctx.beginPath();
  ctx.arc(ball.x + ball.size / 2, ball.y + ball.size / 2, ball.size / 2, 0, Math.PI * 2);
  ctx.fill();
}

function draw() {
  // Clear
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Draw paddles & ball
  drawRect(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height, leftPaddle.color);
  drawRect(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height, rightPaddle.color);
  drawBall(ball);

  // Middle line
  ctx.strokeStyle = "#fff";
  ctx.setLineDash([8, 8]);
  ctx.beginPath();
  ctx.moveTo(canvas.width / 2, 0);
  ctx.lineTo(canvas.width / 2, canvas.height);
  ctx.stroke();
  ctx.setLineDash([]);
}

function clamp(val, min, max) {
  return Math.max(min, Math.min(max, val));
}

// Mouse controls for left paddle
canvas.addEventListener('mousemove', function(evt) {
  const rect = canvas.getBoundingClientRect();
  const mouseY = evt.clientY - rect.top;
  leftPaddle.y = clamp(mouseY - leftPaddle.height / 2, 0, canvas.height - leftPaddle.height);
});

// AI controls for right paddle
function aiMove() {
  // Follow the ball's y position, but not too fast (basic AI)
  let paddleCenter = rightPaddle.y + rightPaddle.height / 2;
  if (ball.y + ball.size / 2 < paddleCenter - 8) {
    rightPaddle.y -= rightPaddle.speed;
  } else if (ball.y + ball.size / 2 > paddleCenter + 8) {
    rightPaddle.y += rightPaddle.speed;
  }
  // Clamp to canvas
  rightPaddle.y = clamp(rightPaddle.y, 0, canvas.height - rightPaddle.height);
}

// Ball and paddle collision
function collide(paddle, ball) {
  return (
    ball.x < paddle.x + paddle.width &&
    ball.x + ball.size > paddle.x &&
    ball.y < paddle.y + paddle.height &&
    ball.y + ball.size > paddle.y
  );
}

function resetBall() {
  ball.x = canvas.width / 2 - BALL_SIZE / 2;
  ball.y = canvas.height / 2 - BALL_SIZE / 2;
  ball.speedX = 5 * (Math.random() < 0.5 ? 1 : -1);
  ball.speedY = 3 * (Math.random() < 0.5 ? 1 : -1);
}

function update() {
  // Move ball
  ball.x += ball.speedX;
  ball.y += ball.speedY;

  // Wall collision (top/bottom)
  if (ball.y <= 0 || ball.y + ball.size >= canvas.height) {
    ball.speedY *= -1;
    ball.y = clamp(ball.y, 0, canvas.height - ball.size);
  }

  // Paddle collision (left)
  if (collide(leftPaddle, ball)) {
    ball.x = leftPaddle.x + leftPaddle.width;
    ball.speedX *= -1;
    // Add a bit of "english" based on where it hit the paddle
    let impact = (ball.y + ball.size / 2 - (leftPaddle.y + leftPaddle.height / 2)) / (leftPaddle.height / 2);
    ball.speedY = impact * 5;
  }

  // Paddle collision (right)
  if (collide(rightPaddle, ball)) {
    ball.x = rightPaddle.x - ball.size;
    ball.speedX *= -1;
    let impact = (ball.y + ball.size / 2 - (rightPaddle.y + rightPaddle.height / 2)) / (rightPaddle.height / 2);
    ball.speedY = impact * 5;
  }

  // Scoring (ball goes off left or right)
  if (ball.x < 0 || ball.x + ball.size > canvas.width) {
    resetBall();
  }

  aiMove();
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

// Start the game
gameLoop();
