<script>
  import { onMount, onDestroy } from 'svelte';

  const ball = new URL('./assets/ball.png', import.meta.url).href;
  const bluebone = new URL('./assets/bluebone.png', import.meta.url).href;
  const cake = new URL('./assets/cake.png', import.meta.url).href;
  const cbirb = new URL('./assets/cbirb.png', import.meta.url).href;
  const doggo = new URL('./assets/doggo.png', import.meta.url).href;
  const doggohat = new URL('./assets/doggohat.png', import.meta.url).href;
  const hat = new URL('./assets/hat.png', import.meta.url).href;
  const meowl = new URL('./assets/meowl.png', import.meta.url).href;
  const mouse = new URL('./assets/mouse.png', import.meta.url).href;
  const stor = new URL('./assets/stor.png', import.meta.url).href;
  const wotor = new URL('./assets/wotor.png', import.meta.url).href;



  // Sprites are randomly sized between these values (rem).
  const SPRITE_MIN_SIZE = 14;
  const SPRITE_MAX_SIZE = 24;

  // Reach this aura score to win.
  const WIN_SCORE = 67;

  // Chance a spawned sprite is a trap (costs aura instead of giving it).
  const TRAP_CHANCE = 0.12;

  // How long a combo stays alive after a hit, in ms.
  const COMBO_WINDOW = 700;

  // Max bonus aura a combo streak can add per hit.
  const COMBO_MAX_BONUS = 2;

  // Assumed px-per-rem for sizing math (root font-size is never changed here).
  const PX_PER_REM = 16;

  const sprites = [
    ball,
    bluebone,
    cake,
    cbirb,
    doggo,
    doggohat,
    hat,
    meowl,
    mouse,
    stor,
    wotor
  ];

  let containerEl;
  let containerWidth = 0;
  let containerHeight = 0;
  let resizeObserver;

  let falling = [];
  let score = 0;
  let combo = 0;
  let bestCombo = 0;

  let gameStarted = false;
  let gameOver = false;
  let gameWon = false;

  let nextId = 0;
  let recentX = [];

  let spawnTimeout;
  let comboTimeout;
  let rafId;
  let lastFrameAt = 0;


  function getRandomIntInclusive(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);

    return Math.floor(
      Math.random() * (max - min + 1)
    ) + min;
  }


  function difficultyProgress() {
    return Math.min(score / WIN_SCORE, 1);
  }


  function getSpawnDelay() {
    return 1000 - difficultyProgress() * 550;
  }


  function pickLeft(sizePx) {
    const maxLeft = Math.max(0, containerWidth - sizePx);
    const gap = sizePx * 0.9;

    let left;
    let attempts = 0;

    do {
      left = Math.random() * maxLeft;
      attempts++;
    } while (
      recentX.some(x => Math.abs(x - left) < gap) &&
      attempts < 6
    );

    recentX = [...recentX.slice(-3), left];

    return left;
  }


  function spawnSprite() {
    if (!gameStarted || gameOver || gameWon) return;
    if (containerWidth < 10 || containerHeight < 10) return;

    const sizeRem = getRandomIntInclusive(
      SPRITE_MIN_SIZE,
      SPRITE_MAX_SIZE
    );

    // Never let a sprite be wider than the viewport lets it be.
    const sizePx = Math.min(
      sizeRem * PX_PER_REM,
      Math.max(containerWidth - 8, 24)
    );

    const speedUp = 1 - difficultyProgress() * 0.2;
    const fallSeconds = (4.5 + Math.random() * 2) * speedUp;
    const travel = containerHeight + sizePx * 2.4;

    const sprite = {
      id: nextId++,

      src: sprites[
        Math.floor(Math.random() * sprites.length)
      ],

      sizeRem,
      sizePx,

      x: pickLeft(sizePx),
      y: -sizePx * 1.2,
      speed: travel / fallSeconds,

      trap: Math.random() < TRAP_CHANCE,
      hit: false
    };

    falling = [...falling, sprite];
  }


  function scheduleSpawn() {
    spawnTimeout = setTimeout(() => {
      spawnSprite();

      if (gameStarted) scheduleSpawn();
    }, getSpawnDelay());
  }


  function tick(now) {
    const dt = lastFrameAt ? (now - lastFrameAt) / 1000 : 0;
    lastFrameAt = now;

    let missedId = null;
    const removeIds = new Set();

    for (const sprite of falling) {
      if (sprite.hit) continue;

      sprite.y += sprite.speed * dt;

      if (sprite.y > containerHeight + 40) {
        /*
         * Red/trap sprites reaching the bottom are harmless.
         * Remove them without triggering a lose condition.
         */
        if (sprite.trap) {
          removeIds.add(sprite.id);
        } else if (missedId === null) {
          missedId = sprite.id;
        }
      }
    }

    if (removeIds.size > 0) {
      falling = falling.filter(
        sprite => !removeIds.has(sprite.id)
      );
    } else {
      falling = falling;
    }

    if (missedId !== null) {
      missed(missedId);
      return;
    }

    if (gameStarted) {
      rafId = requestAnimationFrame(tick);
    }
  }


  function startGame() {
    clearTimeout(spawnTimeout);
    clearTimeout(comboTimeout);
    cancelAnimationFrame(rafId);

    falling = [];
    score = 0;
    combo = 0;
    bestCombo = 0;
    recentX = [];
    lastFrameAt = 0;

    gameOver = false;
    gameWon = false;
    gameStarted = true;

    spawnSprite();
    spawnSprite();

    scheduleSpawn();
    rafId = requestAnimationFrame(tick);
  }


  function hit(id) {
    if (gameOver || gameWon) return;

    const sprite = falling.find(s => s.id === id);
    if (!sprite || sprite.hit) return;

    sprite.hit = true;
    falling = [...falling];

    if (sprite.trap) {
      score = Math.max(0, score - 8);
      combo = 0;
    } else {
      combo += 1;
      bestCombo = Math.max(bestCombo, combo);

      clearTimeout(comboTimeout);
      comboTimeout = setTimeout(() => { combo = 0; }, COMBO_WINDOW);

      const bonus = Math.min(combo - 1, COMBO_MAX_BONUS);

      score += getRandomIntInclusive(3, 5) + bonus;
    }

    setTimeout(() => {
      falling = falling.filter(s => s.id !== id);
    }, 150);

    // WIN CONDITION
    if (score >= WIN_SCORE) {
      score = WIN_SCORE;

      gameWon = true;
      gameStarted = false;

      falling = [];

      clearTimeout(spawnTimeout);
      cancelAnimationFrame(rafId);
    }
  }


  function missed(id) {
    if (gameOver || gameWon) return;

    falling = falling.filter(
      sprite => sprite.id !== id
    );

    combo = 0;

    gameOver = true;
    gameStarted = false;

    clearTimeout(spawnTimeout);
    cancelAnimationFrame(rafId);
  }


  function restart() {
    startGame();
  }


  onMount(() => {
    resizeObserver = new ResizeObserver(entries => {
      const rect = entries[0].contentRect;

      containerWidth = rect.width;
      containerHeight = rect.height;
    });

    resizeObserver.observe(containerEl);
  });


  onDestroy(() => {
    resizeObserver?.disconnect();

    clearTimeout(spawnTimeout);
    clearTimeout(comboTimeout);
    cancelAnimationFrame(rafId);
  });
</script>


<main class="blur-panel vignette">

  {#if gameStarted}
    <div class="score retro-border">
      aura: {score} / {WIN_SCORE}

      {#if combo > 1}
        <span class="combo">x{combo} combo!</span>
      {/if}
    </div>
  {/if}


  <div class="game-area" bind:this={containerEl}>

    {#if !gameStarted && !gameOver && !gameWon}
      <div class="start-screen">

        <h1>
          click and defeat the evil minions
          before they steal your gift 🔥
          <br />
          <span class="subtext">(do not click on red glowing sprites)</span>
        </h1>

        <button
          class="start-btn retro-border"
          onclick={startGame}
        >
          alr
        </button>

      </div>
    {/if}


    {#each falling as sprite (sprite.id)}
      <img
        class={`falling-sprite ${sprite.trap ? 'trap' : ''} ${sprite.hit ? 'hit' : ''}`}
        src={sprite.src}
        alt=""
        draggable="false"
        style={`
          width: ${sprite.sizeRem}rem;
          height: ${sprite.sizeRem}rem;
          transform: translate3d(${sprite.x}px, ${sprite.y}px, 0);
        `}
        onclick={() => hit(sprite.id)}
      />
    {/each}


    {#if gameOver}
      <div class="game-over">

        <div class="game-over-box retro-border">

          <h1>twin 💔</h1>

          <p>aura: {score}</p>

          {#if bestCombo > 1}
            <p class="combo-stat">best combo: x{bestCombo}</p>
          {/if}

          <button
            class="start-btn"
            onclick={restart}
          >
            TRY AGAIN
          </button>

        </div>

      </div>
    {/if}


    {#if gameWon}
      <div class="game-over">

        <div class="game-over-box win-box retro-border">

          <h1>YOU WON</h1>

          <p>
            you got the gift
          </p>

          <p>
            aura: {score}
          </p>

          {#if bestCombo > 1}
            <p class="combo-stat">best combo: x{bestCombo}</p>
          {/if}

          <button  class="start-btn" onclick={() => {
               window.location.href = '/megh/#/4';
          }}>
            GIFT TIME
          </button>

        </div>

      </div>
    {/if}

  </div>

</main>


<style>
  :global(html),
  :global(body) {
    width: 100%;
    height: 100%;
    margin: 0;
    overflow: hidden;
  }


  :global(body) {
    background: conic-gradient(
      violet 90deg,
      aqua 90deg 180deg,
      violet 180deg 270deg,
      aqua 270deg
    );

    background-repeat: repeat;
    background-size: 80px 80px;
    background-position: 0 0;

    animation: scroll-diag 6s linear infinite;
  }


  /* FULLSCREEN */

  main {
    position: fixed;
    inset: 0;

    width: 100%;
    height: 100%;

    overflow: hidden;
  }


  /* BLUR */

  .blur-panel {
    background: rgba(255, 255, 255, 0.12);

    backdrop-filter: blur(0.5px);
    -webkit-backdrop-filter: blur(0.5px);

    z-index: 2;
  }


  /* VIGNETTE */

  .vignette {
    --vignette-color: rgba(0, 0, 0, 0.3);
    --vignette-inner: rgba(0, 0, 0, 0);

    position: fixed;
    inset: 0;

    overflow: hidden;

    border-radius: 0;
  }


  .vignette::before {
    content: "";

    position: absolute;
    inset: 0;

    pointer-events: none;

    background-image: radial-gradient(
      circle at center,
      var(--vignette-inner) 50%,
      var(--vignette-color) 100%
    );

    mix-blend-mode: multiply;

    z-index: 50;
  }


  /* GAME */

  .game-area {
    position: absolute;
    inset: 0;

    overflow: hidden;

    z-index: 10;
  }


  /* SCORE */

  .score {
    position: absolute;

    top: 2rem;
    left: 2rem;

    padding: 0.75rem 1.25rem;

    font-size: 1.5rem;
    font-weight: bold;

    color: #000;

    z-index: 100;
  }


  .combo {
    margin-left: 0.75rem;

    color: #cc2222;

    font-size: 1rem;
  }


  .combo-stat {
    color: #555;

    font-size: 1rem !important;
  }


  /* FALLING SPRITES */

  .falling-sprite {
    position: absolute;

    top: 0;
    left: 0;

    object-fit: contain;

    cursor: pointer;

    user-select: none;

    image-rendering: pixelated;

    z-index: 20;

    box-sizing: border-box;

    /*
     * Glow via layered drop-shadows, kept small so it doesn't
     * balloon out the hitbox.
     */
    filter:
      drop-shadow(0 0 0.35rem rgba(255, 255, 255, 0.95))
      drop-shadow(0 0 0.8rem rgba(255, 255, 255, 0.55))
      drop-shadow(0 0 1.5rem rgba(255, 255, 255, 0.3));

    /*
     * Position is driven by JS every frame via transform,
     * which stays GPU-friendly and doesn't trigger layout.
     */
    will-change: transform;
  }


  .falling-sprite.trap {
    filter:
      sepia(1)
      saturate(6)
      hue-rotate(-50deg)
      brightness(0.9)
      drop-shadow(0 0 0.4rem rgba(255, 40, 40, 0.95))
      drop-shadow(0 0 0.9rem rgba(255, 40, 40, 0.6))
      drop-shadow(0 0 1.6rem rgba(255, 40, 40, 0.35));

    animation: warn-pulse 0.5s ease-in-out infinite;
  }


  .falling-sprite.hit {
    /*
     * Do NOT animate transform here.
     * transform is controlled by JS and contains the sprite's
     * translate3d position. Overriding it causes the sprite to
     * jump toward the origin when clicked.
     */
    animation: pop 150ms ease-out forwards;

    pointer-events: none;
  }


  @keyframes warn-pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.6; }
  }


  @keyframes pop {
    from {
      opacity: 1;
      filter: brightness(1);
    }

    to {
      opacity: 0;
      filter: brightness(1.8);
    }
  }


  /* START SCREEN */

  .start-screen {
    position: absolute;
    inset: 0;

    display: flex;
    flex-direction: column;

    justify-content: center;
    align-items: center;

    gap: 1.5rem;

    padding: 2rem;

    z-index: 100;
  }


  .start-screen h1 {
    max-width: 60rem;

    margin: 0;

    font-size: clamp(
      2.5rem,
      6vw,
      5rem
    );

    font-weight: bold;

    color: greenyellow;

    text-align: center;

    text-shadow:
      0.25rem 0.25rem 0 #000,
      0.5rem 0.5rem 0 rgba(
        0,
        0,
        0,
        0.5
      );
  }


  .subtext {
    font-size: 0.7em;
    border: none;
    box-shadow: none;
    color: #ff6666;
  }


  /* GAME OVER / WIN */

  .game-over {
    position: absolute;
    inset: 0;

    display: flex;

    justify-content: center;
    align-items: center;

    z-index: 100;
  }


  .game-over-box {
    min-width: 18rem;

    padding: 2rem;

    display: flex;
    flex-direction: column;

    align-items: center;

    gap: 1rem;

    text-align: center;
  }


  .game-over-box h1 {
    margin: 0;

    font-size: 2.5rem;
  }


  .game-over-box p {
    margin: 0;

    font-size: 1.5rem;
    font-weight: bold;
  }


  /* WIN */

  .win-box h1 {
    color: greenyellow;

    text-shadow:
      0.2rem 0.2rem 0 #000;
  }


  /* RETRO UI */

  .retro-border {
    background: #c0c0c0;

    border: 0.5rem solid #000;

    box-shadow:
      0.5rem 0.5rem 0 #000;
  }


  /* BUTTON */

  .start-btn {
    font-size: 1.25rem;
    font-weight: bold;

    background: #5555ff;

    color: #fff;

    border: 0.25rem solid #000;

    padding: 1rem 2.5rem;

    cursor: pointer;

    box-shadow:
      0.4rem 0.4rem 0 #000;
  }


  .start-btn:hover {
    background: #6666ff;
  }


  .start-btn:active {
    background: #2222aa;

    transform: translate(
      0.2rem,
      0.2rem
    );

    box-shadow:
      0.2rem 0.2rem 0 #000;
  }


  /* BACKGROUND */

  @keyframes scroll-diag {
    from {
      background-position: 0 0;
    }

    to {
      background-position: 80px 80px;
    }
  }


  /* MOBILE */

  @media (max-width: 600px) {

    .falling-sprite {
      max-width: 12rem;
      max-height: 12rem;
    }


    .score {
      top: 1rem;
      left: 1rem;

      font-size: 1rem;
    }


    .combo {
      display: block;

      margin-left: 0;
    }
  }
</style>