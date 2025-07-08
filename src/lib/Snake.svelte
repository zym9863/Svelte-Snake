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

  // 粒子系统
  let particles: Array<{
    x: number;
    y: number;
    vx: number;
    vy: number;
    life: number;
    maxLife: number;
    color: string;
  }> = [];

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

  // 创建粒子效果
  function createParticles(x: number, y: number, color: string) {
    for (let i = 0; i < 8; i++) {
      particles.push({
        x: x + GRID_SIZE / 2,
        y: y + GRID_SIZE / 2,
        vx: (Math.random() - 0.5) * 6,
        vy: (Math.random() - 0.5) * 6,
        life: 30,
        maxLife: 30,
        color: color
      });
    }
  }

  // 更新粒子
  function updateParticles() {
    particles = particles.filter(particle => {
      particle.x += particle.vx;
      particle.y += particle.vy;
      particle.vy += 0.2; // 重力
      particle.life--;
      return particle.life > 0;
    });
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
      
      // 创建粒子效果
      createParticles(food.x * GRID_SIZE, food.y * GRID_SIZE, '#ff6666');
      
      generateFood();
    } else {
      // 移除尾部
      snake = snake.slice(0, -1);
    }

    // 更新粒子
    updateParticles();
  }

  // 渲染游戏
  function render() {
    if (!ctx) return;

    // 创建渐变背景
    const gradient = ctx.createLinearGradient(0, 0, CANVAS_SIZE, CANVAS_SIZE);
    gradient.addColorStop(0, '#1a1a2e');
    gradient.addColorStop(0.5, '#16213e');
    gradient.addColorStop(1, '#0f3460');
    
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

    // 绘制网格线（可选，增强视觉效果）
    ctx.strokeStyle = 'rgba(255, 255, 255, 0.05)';
    ctx.lineWidth = 0.5;
    for (let i = 0; i <= CANVAS_SIZE; i += GRID_SIZE) {
      ctx.beginPath();
      ctx.moveTo(i, 0);
      ctx.lineTo(i, CANVAS_SIZE);
      ctx.stroke();
      
      ctx.beginPath();
      ctx.moveTo(0, i);
      ctx.lineTo(CANVAS_SIZE, i);
      ctx.stroke();
    }

    // 绘制蛇
    snake.forEach((segment, index) => {
      const x = segment.x * GRID_SIZE;
      const y = segment.y * GRID_SIZE;
      
      if (index === 0) {
        // 蛇头 - 使用渐变和阴影
        const headGradient = ctx.createRadialGradient(
          x + GRID_SIZE/2, y + GRID_SIZE/2, 0,
          x + GRID_SIZE/2, y + GRID_SIZE/2, GRID_SIZE/2
        );
        headGradient.addColorStop(0, '#00ff88');
        headGradient.addColorStop(1, '#00cc66');
        
        ctx.fillStyle = headGradient;
        ctx.beginPath();
        ctx.roundRect(x + 1, y + 1, GRID_SIZE - 2, GRID_SIZE - 2, 6);
        ctx.fill();
        
        // 添加眼睛
        ctx.fillStyle = '#000';
        const eyeSize = 3;
        ctx.beginPath();
        ctx.arc(x + 6, y + 6, eyeSize, 0, Math.PI * 2);
        ctx.fill();
        ctx.beginPath();
        ctx.arc(x + GRID_SIZE - 6, y + 6, eyeSize, 0, Math.PI * 2);
        ctx.fill();
      } else {
        // 蛇身 - 使用渐变效果
        const bodyGradient = ctx.createRadialGradient(
          x + GRID_SIZE/2, y + GRID_SIZE/2, 0,
          x + GRID_SIZE/2, y + GRID_SIZE/2, GRID_SIZE/2
        );
        const alpha = Math.max(0.6, 1 - index * 0.05);
        bodyGradient.addColorStop(0, `rgba(0, 255, 136, ${alpha})`);
        bodyGradient.addColorStop(1, `rgba(0, 204, 102, ${alpha * 0.8})`);
        
        ctx.fillStyle = bodyGradient;
        ctx.beginPath();
        ctx.roundRect(x + 1, y + 1, GRID_SIZE - 2, GRID_SIZE - 2, 4);
        ctx.fill();
      }
    });

    // 绘制食物 - 使用动画效果
    const foodX = food.x * GRID_SIZE;
    const foodY = food.y * GRID_SIZE;
    const time = Date.now() * 0.005;
    const pulse = 1 + Math.sin(time) * 0.1;
    
    // 食物发光效果
    ctx.shadowColor = '#ff4444';
    ctx.shadowBlur = 15;
    
    const foodGradient = ctx.createRadialGradient(
      foodX + GRID_SIZE/2, foodY + GRID_SIZE/2, 0,
      foodX + GRID_SIZE/2, foodY + GRID_SIZE/2, GRID_SIZE/2 * pulse
    );
    foodGradient.addColorStop(0, '#ff6666');
    foodGradient.addColorStop(0.7, '#ff3333');
    foodGradient.addColorStop(1, '#cc0000');
    
    ctx.fillStyle = foodGradient;
    ctx.beginPath();
    ctx.arc(
      foodX + GRID_SIZE/2, 
      foodY + GRID_SIZE/2, 
      (GRID_SIZE/2 - 1) * pulse, 
      0, 
      Math.PI * 2
    );
    ctx.fill();
    
    // 重置阴影
    ctx.shadowBlur = 0;

    // 绘制粒子效果
    particles.forEach(particle => {
      const alpha = particle.life / particle.maxLife;
      ctx.fillStyle = particle.color.replace(')', `, ${alpha})`).replace('rgb', 'rgba');
      ctx.beginPath();
      ctx.arc(particle.x, particle.y, 3 * alpha, 0, Math.PI * 2);
      ctx.fill();
    });
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
    
    // Polyfill for roundRect if not available
    if (!ctx.roundRect) {
      ctx.roundRect = function(x: number, y: number, width: number, height: number, radii?: number | DOMPointInit | (number | DOMPointInit)[] | Iterable<number | DOMPointInit>) {
        let radius = 0;
        if (typeof radii === 'number') {
          radius = radii;
        } else if (Array.isArray(radii) && radii.length > 0) {
          radius = typeof radii[0] === 'number' ? radii[0] : 0;
        }
        
        this.beginPath();
        this.moveTo(x + radius, y);
        this.lineTo(x + width - radius, y);
        this.quadraticCurveTo(x + width, y, x + width, y + radius);
        this.lineTo(x + width, y + height - radius);
        this.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
        this.lineTo(x + radius, y + height);
        this.quadraticCurveTo(x, y + height, x, y + height - radius);
        this.lineTo(x, y + radius);
        this.quadraticCurveTo(x, y, x + radius, y);
        this.closePath();
      };
    }
    
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
  <div class="game-card">
    <div class="game-header">
      <h1 class="game-title">🐍 贪吃蛇游戏</h1>
      <div class="score-display">
        <span class="score-label">分数</span>
        <span class="score-value">{score}</span>
      </div>
    </div>
    
    <div class="game-area">
      <div class="canvas-container">
        <canvas 
          bind:this={canvas}
          width={CANVAS_SIZE} 
          height={CANVAS_SIZE}
          class="game-canvas"
        ></canvas>
        {#if !gameRunning && !gameOver}
          <div class="start-overlay">
            <div class="start-content">
              <div class="start-icon">🎮</div>
              <p>按空格键开始游戏</p>
            </div>
          </div>
        {/if}
      </div>
    </div>

    <div class="game-controls">
      {#if !gameRunning && !gameOver}
        <button on:click={startGame} class="btn btn-start">
          <span class="btn-icon">▶️</span>
          开始游戏
        </button>
      {:else if gameRunning}
        <button on:click={pauseGame} class="btn btn-pause">
          <span class="btn-icon">⏸️</span>
          暂停游戏
        </button>
      {:else if gameOver}
        <div class="game-over">
          <div class="game-over-icon">💀</div>
          <h2>游戏结束!</h2>
          <p class="final-score">最终分数: <span>{score}</span></p>
          <button on:click={resetGame} class="btn btn-restart">
            <span class="btn-icon">🔄</span>
            重新开始
          </button>
        </div>
      {/if}
    </div>

    <div class="instructions">
      <h3>🎯 游戏说明</h3>
      <div class="instruction-grid">
        <div class="instruction-item">
          <span class="instruction-icon">⌨️</span>
          <span>方向键或 WASD 控制移动</span>
        </div>
        <div class="instruction-item">
          <span class="instruction-icon">⏯️</span>
          <span>空格键开始/暂停游戏</span>
        </div>
        <div class="instruction-item">
          <span class="instruction-icon">🍎</span>
          <span>吃红色食物增长获得分数</span>
        </div>
        <div class="instruction-item">
          <span class="instruction-icon">⚠️</span>
          <span>避免撞墙或撞到自己</span>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  .game-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding: 20px;
    font-family: 'Inter', system-ui, sans-serif;
  }

  .game-card {
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(20px);
    border-radius: 24px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
    padding: 32px;
    max-width: 600px;
    width: 100%;
    text-align: center;
  }

  .game-header {
    margin-bottom: 24px;
  }

  .game-title {
    margin: 0 0 16px 0;
    font-size: 2.5rem;
    font-weight: 700;
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  }

  .score-display {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 12px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 12px 24px;
    border-radius: 50px;
    box-shadow: 0 8px 32px rgba(102, 126, 234, 0.3);
    max-width: 200px;
    margin: 0 auto;
  }

  .score-label {
    font-size: 1rem;
    font-weight: 500;
    color: rgba(255, 255, 255, 0.9);
  }

  .score-value {
    font-size: 1.5rem;
    font-weight: 700;
    color: #fff;
    min-width: 40px;
  }

  .game-area {
    margin-bottom: 24px;
  }

  .canvas-container {
    position: relative;
    display: inline-block;
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 0 16px 32px rgba(0, 0, 0, 0.4);
  }

  .game-canvas {
    display: block;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  }

  .start-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    backdrop-filter: blur(4px);
  }

  .start-content {
    text-align: center;
    color: white;
  }

  .start-icon {
    font-size: 3rem;
    margin-bottom: 12px;
    animation: bounce 2s infinite;
  }

  @keyframes bounce {
    0%, 20%, 50%, 80%, 100% {
      transform: translateY(0);
    }
    40% {
      transform: translateY(-10px);
    }
    60% {
      transform: translateY(-5px);
    }
  }

  .game-controls {
    margin-bottom: 24px;
  }

  .btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 14px 28px;
    font-size: 1.1rem;
    font-weight: 600;
    border: none;
    border-radius: 50px;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  }

  .btn::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
    transition: left 0.5s;
  }

  .btn:hover::before {
    left: 100%;
  }

  .btn-icon {
    font-size: 1.2rem;
  }

  .btn-start {
    background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
    color: white;
  }

  .btn-start:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(79, 172, 254, 0.4);
  }

  .btn-pause {
    background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
    color: #8b4513;
  }

  .btn-pause:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(252, 182, 159, 0.4);
  }

  .btn-restart {
    background: linear-gradient(135deg, #ff758c 0%, #ff7eb3 100%);
    color: white;
  }

  .btn-restart:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(255, 117, 140, 0.4);
  }

  .game-over {
    text-align: center;
    animation: fadeInUp 0.5s ease-out;
  }

  .game-over-icon {
    font-size: 4rem;
    margin-bottom: 16px;
    animation: shake 0.5s ease-in-out;
  }

  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
    20%, 40%, 60%, 80% { transform: translateX(5px); }
  }

  .game-over h2 {
    font-size: 2rem;
    margin: 0 0 12px 0;
    color: #ff6b6b;
    font-weight: 700;
  }

  .final-score {
    font-size: 1.2rem;
    margin-bottom: 20px;
    color: rgba(255, 255, 255, 0.9);
  }

  .final-score span {
    font-weight: 700;
    color: #4ecdc4;
    font-size: 1.4rem;
  }

  .instructions {
    text-align: left;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 16px;
    padding: 20px;
    border: 1px solid rgba(255, 255, 255, 0.1);
  }

  .instructions h3 {
    margin: 0 0 16px 0;
    font-size: 1.3rem;
    font-weight: 600;
    color: rgba(255, 255, 255, 0.95);
    text-align: center;
  }

  .instruction-grid {
    display: grid;
    gap: 12px;
  }

  .instruction-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 8px 0;
    color: rgba(255, 255, 255, 0.8);
    font-size: 0.95rem;
  }

  .instruction-icon {
    font-size: 1.2rem;
    min-width: 24px;
    text-align: center;
  }

  /* 响应式设计 */
  @media (max-width: 768px) {
    .game-card {
      padding: 20px;
      margin: 10px;
    }

    .game-title {
      font-size: 2rem;
    }

    .canvas-container {
      transform: scale(0.8);
      margin: -40px;
    }

    .instruction-grid {
      grid-template-columns: 1fr;
    }
  }

  @media (max-width: 480px) {
    .canvas-container {
      transform: scale(0.6);
      margin: -80px;
    }

    .game-title {
      font-size: 1.8rem;
    }
  }

  /* 添加一些动画效果 */
  .game-card {
    animation: slideIn 0.6s ease-out;
  }

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(50px) scale(0.9);
    }
    to {
      opacity: 1;
      transform: translateY(0) scale(1);
    }
  }

  /* Canvas polyfill for roundRect */
  .game-canvas {
    image-rendering: pixelated;
  }
</style>
