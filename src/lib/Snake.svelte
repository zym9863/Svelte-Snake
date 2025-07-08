<script lang="ts">
  import { onMount, onDestroy } from 'svelte';

  // 游戏配置
  const GRID_SIZE = 20;
  const CANVAS_SIZE = 400;
  const INITIAL_SPEED = 150;

  // 游戏状态
  let gameRunning = false;
  let gameOver = false;
  let score = 0;
  let speed = INITIAL_SPEED;

  // 蛇的状态
  let snake = [{ x: 10, y: 10 }];
  let direction = { x: 1, y: 0 };
  let nextDirection = { x: 1, y: 0 };

  // 食物位置
  let food = { x: 15, y: 15 };

  // Canvas 引用
  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let gameLoop: number;

  // 生成随机食物位置
  function generateFood() {
    const maxPos = CANVAS_SIZE / GRID_SIZE - 1;
    let newFood;
    do {
      newFood = {
        x: Math.floor(Math.random() * (maxPos + 1)),
        y: Math.floor(Math.random() * (maxPos + 1))
      };
    } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));
    food = newFood;
  }

  // 检查碰撞
  function checkCollision(head: {x: number, y: number}) {
    const maxPos = CANVAS_SIZE / GRID_SIZE - 1;
    
    // 检查墙壁碰撞
    if (head.x < 0 || head.x > maxPos || head.y < 0 || head.y > maxPos) {
      return true;
    }
    
    // 检查自身碰撞
    return snake.some(segment => segment.x === head.x && segment.y === head.y);
  }

  // 游戏主循环
  function update() {
    if (!gameRunning || gameOver) return;

    // 更新方向
    direction = { ...nextDirection };

    // 计算新的头部位置
    const head = { 
      x: snake[0].x + direction.x, 
      y: snake[0].y + direction.y 
    };

    // 检查碰撞
    if (checkCollision(head)) {
      gameOver = true;
      gameRunning = false;
      return;
    }

    // 添加新头部
    snake = [head, ...snake];

    // 检查是否吃到食物
    if (head.x === food.x && head.y === food.y) {
      score += 10;
      speed = Math.max(50, speed - 2); // 增加速度
      generateFood();
    } else {
      // 移除尾部
      snake = snake.slice(0, -1);
    }
  }

  // 渲染游戏
  function render() {
    if (!ctx) return;

    // 清空画布
    ctx.fillStyle = '#000';
    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

    // 绘制蛇
    ctx.fillStyle = '#0f0';
    snake.forEach((segment, index) => {
      if (index === 0) {
        // 蛇头用不同颜色
        ctx.fillStyle = '#0a0';
      } else {
        ctx.fillStyle = '#0f0';
      }
      ctx.fillRect(
        segment.x * GRID_SIZE, 
        segment.y * GRID_SIZE, 
        GRID_SIZE - 1, 
        GRID_SIZE - 1
      );
    });

    // 绘制食物
    ctx.fillStyle = '#f00';
    ctx.fillRect(
      food.x * GRID_SIZE, 
      food.y * GRID_SIZE, 
      GRID_SIZE - 1, 
      GRID_SIZE - 1
    );
  }

  // 游戏循环
  function gameStep() {
    update();
    render();
    if (gameRunning && !gameOver) {
      gameLoop = setTimeout(gameStep, speed);
    }
  }

  // 开始游戏
  function startGame() {
    if (gameOver) {
      resetGame();
    }
    gameRunning = true;
    gameStep();
  }

  // 暂停游戏
  function pauseGame() {
    gameRunning = false;
    if (gameLoop) {
      clearTimeout(gameLoop);
    }
  }

  // 重置游戏
  function resetGame() {
    gameRunning = false;
    gameOver = false;
    score = 0;
    speed = INITIAL_SPEED;
    snake = [{ x: 10, y: 10 }];
    direction = { x: 1, y: 0 };
    nextDirection = { x: 1, y: 0 };
    generateFood();
    if (gameLoop) {
      clearTimeout(gameLoop);
    }
    render();
  }

  // 键盘控制
  function handleKeyPress(event: KeyboardEvent) {
    if (!gameRunning && !gameOver && event.code === 'Space') {
      startGame();
      return;
    }

    if (gameOver && event.code === 'Space') {
      resetGame();
      return;
    }

    if (!gameRunning) return;

    switch (event.code) {
      case 'ArrowUp':
      case 'KeyW':
        if (direction.y === 0) nextDirection = { x: 0, y: -1 };
        break;
      case 'ArrowDown':
      case 'KeyS':
        if (direction.y === 0) nextDirection = { x: 0, y: 1 };
        break;
      case 'ArrowLeft':
      case 'KeyA':
        if (direction.x === 0) nextDirection = { x: -1, y: 0 };
        break;
      case 'ArrowRight':
      case 'KeyD':
        if (direction.x === 0) nextDirection = { x: 1, y: 0 };
        break;
      case 'Space':
        pauseGame();
        break;
    }
  }

  onMount(() => {
    ctx = canvas.getContext('2d')!;
    generateFood();
    render();
    
    // 添加键盘事件监听
    window.addEventListener('keydown', handleKeyPress);
  });

  onDestroy(() => {
    if (gameLoop) {
      clearTimeout(gameLoop);
    }
    window.removeEventListener('keydown', handleKeyPress);
  });
</script>

<div class="game-container">
  <div class="game-header">
    <h1>贪吃蛇游戏</h1>
    <div class="score">分数: {score}</div>
  </div>
  
  <div class="game-area">
    <canvas 
      bind:this={canvas}
      width={CANVAS_SIZE} 
      height={CANVAS_SIZE}
      class="game-canvas"
    ></canvas>
  </div>

  <div class="game-controls">
    {#if !gameRunning && !gameOver}
      <button on:click={startGame} class="btn btn-start">开始游戏</button>
    {:else if gameRunning}
      <button on:click={pauseGame} class="btn btn-pause">暂停游戏</button>
    {:else if gameOver}
      <div class="game-over">
        <h2>游戏结束!</h2>
        <p>最终分数: {score}</p>
        <button on:click={resetGame} class="btn btn-restart">重新开始</button>
      </div>
    {/if}
  </div>

  <div class="instructions">
    <h3>游戏说明:</h3>
    <ul>
      <li>使用方向键或 WASD 控制蛇的移动</li>
      <li>按空格键开始/暂停游戏</li>
      <li>吃到红色食物可以增长并获得分数</li>
      <li>避免撞墙或撞到自己</li>
    </ul>
  </div>
</div>

<style>
  .game-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    font-family: Arial, sans-serif;
  }

  .game-header {
    text-align: center;
    margin-bottom: 20px;
  }

  .game-header h1 {
    margin: 0 0 10px 0;
    color: #333;
  }

  .score {
    font-size: 18px;
    font-weight: bold;
    color: #0066cc;
  }

  .game-area {
    margin-bottom: 20px;
  }

  .game-canvas {
    border: 2px solid #333;
    background-color: #000;
  }

  .game-controls {
    margin-bottom: 20px;
    text-align: center;
  }

  .btn {
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .btn-start {
    background-color: #4CAF50;
    color: white;
  }

  .btn-start:hover {
    background-color: #45a049;
  }

  .btn-pause {
    background-color: #ff9800;
    color: white;
  }

  .btn-pause:hover {
    background-color: #e68900;
  }

  .btn-restart {
    background-color: #f44336;
    color: white;
  }

  .btn-restart:hover {
    background-color: #da190b;
  }

  .game-over {
    text-align: center;
  }

  .game-over h2 {
    color: #f44336;
    margin: 0 0 10px 0;
  }

  .instructions {
    max-width: 400px;
    text-align: left;
  }

  .instructions h3 {
    margin-top: 0;
    color: #333;
  }

  .instructions ul {
    padding-left: 20px;
  }

  .instructions li {
    margin-bottom: 5px;
    color: #666;
  }
</style>
