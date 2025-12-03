<script lang="ts">
  import Board, { type BoardState, type Player } from '$lib/components/Board.svelte';

  // PUBLIC_INTERFACE
  function calculateWinner(board: BoardState): { winner: Player; line: number[] } | null {
    /** Checks all winning lines and returns the winner and line if found. */
    const lines = [
      [0,1,2],[3,4,5],[6,7,8], // rows
      [0,3,6],[1,4,7],[2,5,8], // cols
      [0,4,8],[2,4,6],         // diagonals
    ];
    for (const [a,b,c] of lines) {
      if (board[a] && board[a] === board[b] && board[a] === board[c]) {
        return { winner: board[a] as Player, line: [a,b,c] };
      }
    }
    return null;
  }

  // PUBLIC_INTERFACE
  function isDraw(board: BoardState): boolean {
    /** Returns true when all squares are filled and there is no winner. */
    return board.every((s) => s !== '');
  }

  let board: BoardState = Array(9).fill('');
  let xIsNext: boolean = true;
  let winner: Player | null = null;
  let winningLine: number[] | null = null;
  let message: string = 'X starts';
  let score = { X: 0, O: 0 };

  function iconFor(p: Player): string {
    return p === 'X' ? '♞' : '♛';
  }

  function labelFor(p: Player): string {
    return p === 'X' ? 'X knight' : 'O queen';
  }

  function handleMove(index: number) {
    if (board[index] || winner) return;

    const next: BoardState = board.slice();
    next[index] = xIsNext ? 'X' : 'O';
    board = next;

    const result = calculateWinner(board);
    if (result) {
      winner = result.winner;
      winningLine = result.line;
      score[winner] += 1;
      message = `${labelFor(result.winner)} wins!`;
      return;
    }

    if (isDraw(board)) {
      message = 'Draw game.';
      return;
    }

    xIsNext = !xIsNext;
    message = `${labelFor(xIsNext ? 'X' : 'O')} to move`;
  }

  // PUBLIC_INTERFACE
  function resetBoard() {
    /** Clears the board and state to start a new round; preserves running score. */
    board = Array(9).fill('');
    xIsNext = true;
    winner = null;
    winningLine = null;
    message = 'X starts';
  }

  // PUBLIC_INTERFACE
  function newMatch() {
    /** Starts a new match and resets running scores. */
    resetBoard();
    score = { X: 0, O: 0 };
  }

  const envPort = import.meta.env?.VITE_PORT ?? '3000';
</script>

<svelte:head>
  <title>Tic Tac Toe — Ocean Pro</title>
  <meta name="description" content="Two-player Tic Tac Toe with a modern Ocean Professional theme." />
</svelte:head>

<div class="wrap">
  <header class="topbar" aria-label="Tic Tac Toe header">
    <div class="title">
      <div class="logo" title="Tic Tac Toe">
        ♛♞
      </div>
      <div class="text">
        <h1>Tic Tac Toe</h1>
        <p>Ocean Professional • Local play • Port {envPort}</p>
      </div>
    </div>
    <div class="actions">
      <button class="btn ghost" on:click={newMatch} aria-label="Start a new match (reset scores)">
        New Match
      </button>
      <button class="btn primary" on:click={resetBoard} aria-label="Start a new round">
        New Game
      </button>
    </div>
  </header>

  <section class="content">
    <div class="panel status" role="status" aria-live="polite">
      <div class="turn">
        <span class={`badge ${xIsNext ? 'x' : 'o'}`} aria-label={labelFor(xIsNext ? 'X' : 'O')} title={labelFor(xIsNext ? 'X' : 'O')}>
          {iconFor(xIsNext ? 'X' : 'O')}
        </span>
        <span class="label">{winner ? 'Winner' : isDraw(board) ? 'Draw' : 'Turn'}</span>
      </div>
      <div class="message">{message}</div>
      <div class="score">
        <div class="score-card" aria-label="Score for X knight">
          <span class="player x" title="X knight">♞</span>
          <span class="value">{score.X}</span>
        </div>
        <div class="score-card" aria-label="Score for O queen">
          <span class="player o" title="O queen">♛</span>
          <span class="value">{score.O}</span>
        </div>
      </div>
    </div>

    <div class="board-wrap" aria-label="Game area">
      <Board
        board={board}
        onMove={handleMove}
        disabled={!!winner}
        winningLine={winningLine}
      />
    </div>
  </section>
</div>

<style>
  .wrap {
    width: 100%;
    max-width: 920px;
    margin: 24px;
  }

  .topbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 18px 22px;
    border-radius: var(--radius-lg);
    background: linear-gradient(120deg, rgba(37,99,235,0.10), rgba(243,244,246,0.6));
    border: 1px solid var(--border);
    box-shadow: var(--shadow-sm), 0 12px 28px rgba(37,99,235,0.08);
    margin-bottom: 22px;
    backdrop-filter: saturate(130%) blur(6px);
  }

  .title {
    display: flex;
    align-items: center;
    gap: 14px;
  }

  .logo {
    width: 42px;
    height: 42px;
    display: grid;
    place-items: center;
    border-radius: 12px;
    background: radial-gradient(120px 120px at 20% 20%, rgba(37,99,235,0.22), rgba(255,255,255,0.85));
    color: var(--primary);
    font-weight: 900;
    letter-spacing: -0.02em;
    border: 1px solid var(--border);
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.7);
    font-size: 1.05rem;
  }

  .text h1 {
    font-size: 1.25rem;
    line-height: 1.2;
    margin: 0;
    color: var(--text);
  }
  .text p {
    margin: 2px 0 0 0;
    font-size: 0.85rem;
    color: var(--muted);
  }

  .actions {
    display: flex;
    gap: 10px;
  }

  .btn {
    appearance: none;
    border: 1px solid var(--border);
    background: var(--surface);
    color: var(--text);
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 600;
    letter-spacing: 0.01em;
    transition: transform 120ms ease, box-shadow 200ms ease, background 200ms ease, color 150ms ease, border-color 150ms ease;
    box-shadow: 0 6px 18px rgba(17, 24, 39, 0.06);
  }

  .btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 12px 28px rgba(17, 24, 39, 0.10);
    border-color: var(--primary-300);
  }

  .btn.primary {
    background: linear-gradient(145deg, rgba(37,99,235,0.95), rgba(37,99,235,0.85));
    color: white;
    border-color: rgba(37,99,235,0.6);
    box-shadow: 0 10px 24px rgba(37, 99, 235, 0.28);
  }
  .btn.primary:hover {
    box-shadow: 0 14px 30px rgba(37, 99, 235, 0.38);
  }

  .btn.ghost {
    background: linear-gradient(145deg, rgba(255,255,255,0.9), rgba(243,244,246,0.8));
  }

  .content {
    display: grid;
    grid-template-columns: 1fr;
    gap: 18px;
  }

  .panel.status {
    display: grid;
    grid-template-columns: 1fr auto auto;
    align-items: center;
    gap: 14px;
    padding: 16px 18px;
    border-radius: var(--radius-lg);
    background: var(--surface);
    border: 1px solid var(--border);
    box-shadow: var(--shadow-sm), 0 10px 26px rgba(17, 24, 39, 0.06);
  }

  .turn {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .badge {
    display: inline-grid;
    place-items: center;
    width: 34px;
    height: 34px;
    border-radius: 10px;
    font-weight: 800;
    letter-spacing: 0.02em;
    color: white;
    box-shadow: 0 8px 22px rgba(17, 24, 39, 0.12);
    font-size: 1.1rem; /* ensure icon is visually balanced */
    line-height: 1;
  }
  .badge.x { background: var(--primary); }
  .badge.o { background: var(--secondary); }

  .label {
    font-weight: 700;
    color: var(--muted);
  }

  .message {
    justify-self: center;
    font-weight: 700;
    color: var(--text);
  }

  .score {
    display: flex;
    gap: 10px;
    justify-self: end;
  }

  .score-card {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 8px 10px;
    border-radius: 12px;
    background: linear-gradient(145deg, rgba(255,255,255,0.95), rgba(243,244,246,0.8));
    border: 1px solid var(--border);
  }
  .score-card .player {
    font-weight: 900;
    padding: 2px 8px;
    border-radius: 8px;
    color: white;
    display: inline-grid;
    place-items: center;
    font-size: 0.95rem;
    line-height: 1;
    width: 28px;
    height: 28px;
  }
  .score-card .player.x { background: var(--primary); }
  .score-card .player.o { background: var(--secondary); }
  .score-card .value {
    font-weight: 800;
    color: var(--text);
    min-width: 1.2em;
    text-align: right;
  }

  .board-wrap {
    display: grid;
    place-items: center;
    padding: 10px;
  }

  @media (max-width: 560px) {
    .topbar { padding: 14px 16px; }
    .panel.status {
      grid-template-columns: 1fr;
      text-align: center;
    }
    .message { justify-self: start; }
    .score { justify-self: start; }
  }
</style>
