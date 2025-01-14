<script lang="ts">
  import { getSudoku } from 'sudoku-gen';
  import { humanReadableTime } from './lib/humanReadableTime';
  
  type Cell = { value: any; enabled: boolean; notes: number[] };
  type Grid = Cell[][];
  type Difficulty = 'easy' | 'medium' | 'hard' | 'expert';
  interface SudokuPuzzle {
    puzzle: string;
    solution: string;
    difficulty: Difficulty;
  }
  
  const difficultyLevel: string[] = ['easy', 'medium', 'hard', 'expert'];
  let sudoku = $state<SudokuPuzzle>();
  let showDebug = $state(false);
  let grid = $state<Grid>([])
  let solution = $state<Grid>([])
  let showHighlight = $state(true);
  let showErrors = $state(false);
  let notesMode = $state(false);

  // TIMER
  let myInterval: ReturnType<typeof setInterval>;
  let timeElapsed = $state(0);
  let paused = false;
  function startInterval() {
    clearInterval(myInterval)
		
    myInterval = setInterval(()=> {
      timeElapsed += 1;
      if(solved) {
        clearInterval(myInterval);
      }
    }, 1000)
		
	}
  startInterval();
  

  function generateBoard(level: Difficulty) {
    sudoku = getSudoku(level);
    grid = makeGrid(sudoku.puzzle);
    solution = makeGrid(sudoku.solution);
    timeElapsed = 0;
    startInterval();
  }

  generateBoard('easy');

  let solved = $derived(grid.flat().map(x => x.value).join('') == solution.flat().map(x => x.value).join(''));

  let curRow = $state(0);
  let curCol = $state(0);
  let cellSelected = $derived(grid[curRow][curCol].value);

  // const nums: number[] = [7,8,9,4,5,6,1,2,3];
  const nums: number[] = [1,2,3,4,5,6,7,8,9];
  let countAppearances = $derived.by(() => {
    const b = grid.flat().map(x => x.value).filter(x => x != '0');
    const counts: {[key: number]: number} = {};
    for (const num of b) {
      counts[num] = counts[num] ? counts[num] + 1 : 1;
    }
    return counts;
  });

  function makeGrid(puzzle: string) {
    puzzle = puzzle.replaceAll('-', '0');
    let b = [];
    for (let i = 0; i < 9; i++) {
      let row = [];
      for (let j = 0; j < 9; j++) {
        row.push({value:parseInt(puzzle[i * 9 + j]), enabled:parseInt(puzzle[i * 9 + j]) == 0 ? true : false, notes: [ ]});
      }
      b.push(row);
    }
    return b;
  }

  function focusCell() {
    document.getElementById(`cell_${curRow}${curCol}`).focus();
  }

  function handleKeyDown(e: KeyboardEvent) {
    // console.log(e.key)
    if (e.key === 'ArrowRight') {
      curCol = (curCol + 1) % 9;
    } else if (e.key === 'ArrowLeft') {
      curCol = (curCol + 8) % 9;
    } else if (e.key === 'ArrowDown') {
      curRow = (curRow + 1) % 9;
    } else if (e.key === 'ArrowUp') {
      curRow = (curRow + 8) % 9;
    }
    focusCell();
    if (e.key=='d') {
      showDebug = !showDebug;
    }
    if (e.key=='-') {
      notesMode = !notesMode;
    }
    if (e.key=='h') {
      showHighlight = !showHighlight;
    }
    if (e.key=='e') {
      showErrors = !showErrors;
    }
    if (e.key === 'Delete') {
      if(notesMode) {
        grid[curRow][curCol].notes.pop();
      } else {
        grid[curRow][curCol].value = 0;
      }
    }
    
    if (parseInt(e.key) >= 1 && parseInt(e.key) <= 9) {
      let num: number = parseInt(e.key);
      updateCellorNotes(num);
    }

  }

  function updateCellorNotes(value) {
    if(grid[curRow][curCol].enabled) {
      if(notesMode) {
        let c = grid[curRow][curCol].notes;
          if(c.includes(value)) {
            grid[curRow][curCol].notes = c.filter(x => x != value)
          } else {
            grid[curRow][curCol].notes.push(value)
          }
      } else {
        grid[curRow][curCol].value = value;
      }
    }
  }

  function handleButtonClick(level) {
      generateBoard(level);
  }

  function solvePuzzle() {
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        grid[i][j].value = solution[i][j].value
      }
    }
  }

</script>

<svelte:body on:keydown={handleKeyDown} />
<main>
  <h1>sudoku</h1>
  <div class="controls">
    <div class="buttons">

      {#each difficultyLevel as level}
      <button
      class:highlight={level == sudoku.difficulty && !solved}
      onclick={() => handleButtonClick(level)}
      >{level}</button>
      {/each}
      <!-- <button onclick={solvePuzzle}>solve</button> -->
      
    </div>
    <div class="settings">
      <label for="showHighlight">highlight <span class="keyshortcut">(h)</span> <input type="checkbox" id="showHighlight" bind:checked={showHighlight}></label>
      <label for="showErrors">errors <span class="keyshortcut">(e)</span> <input type="checkbox" id="showErrors" bind:checked={showErrors}></label>
      <label class="input-note-entry" class:highlight={notesMode} for="notesMode">notes <span class="keyshortcut">(-)</span> <input type="checkbox" id="notesMode" bind:checked={notesMode}></label>
    </div>
  </div>

  <div class="main-cols">
  
    <div class="board" class:solved={solved}>
      {#each grid as row, r}
          {#each row as cell, c}
            <button id={'cell_'+r+''+c} class="cell"
              onclick={() => {curRow = r; curCol = c}}
              class:highlight={showHighlight && cellSelected==cell.value && cell.value != 0}
              class:active={r==curRow && c==curCol}
              class:error={showErrors && (solution[r][c].value != cell.value) && (cell.value != 0)}
            >
              <!-- <span class="notes" class:activecandy={notesMode && r==curRow && c==curCol}>{cell.notes.join('')}</span> -->
              <div class="notes-grid" class:active={r==curRow && c==curCol && notesMode} class:hideme={cell.value != 0}>
                {#each nums as note}
                  <span class:shown={cell.notes.includes(note)}>{note}</span>
                {/each}
              </div>
              <span class="value" class:given={!grid[r][c].enabled} class:hidden={cell.value==0}>
                {cell.value}
              </span>
            </button>
          {/each}
      {/each}
    </div>

    <div class="nums-wrap">
      {#each nums as value, index}
        <button
          class="num"
          class:done={countAppearances[value] == 9}
          onclick={() => updateCellorNotes(value)}>{value}
        </button>
      {/each}
      <button class="num num-wide" onclick={() => updateCellorNotes(0)}>X</button>
    </div>
    <div class="settings-small">
      <label for="showHighlight">highlight <span class="keyshortcut">(h)</span> <input type="checkbox" id="showHighlight" bind:checked={showHighlight}></label>
      <label for="showErrors">errors <span class="keyshortcut">(e)</span> <input type="checkbox" id="showErrors" bind:checked={showErrors}></label>
      <label class="input-note-entry" class:highlight={notesMode} for="notesMode">notes <span class="keyshortcut">(-)</span> <input type="checkbox" id="notesMode" bind:checked={notesMode}></label>
    </div>

  </div>
  <div class="timer" class:solved={solved}>{solved ? 'Solved '+sudoku?.difficulty+' in ' : ''}{humanReadableTime(timeElapsed)}</div>
</main>
<footer>
  <p>puzzles generated by <a href="https://github.com/petewritescode/sudoku-gen">sudoku-gen</a></p>
  <p><a href="https://github.com/tkroo/sudoku-svelte5">source</a></p>
</footer>


{#if showDebug}
<div class="debug">
  countAppearances: {JSON.stringify(countAppearances)}<br>
  curRow: {curRow}, curCol: {curCol}, cellSelected: {cellSelected} solved: {solved}
  <br><br>
  {#each solution.flat() as i, index}
  {i.value}
  {#if (index+1)%3 == 0 && (index+1)%9 != 0}&nbsp;{/if}
  {#if (index+1)%9 == 0}<br>{/if}
  {#if (index+1)%27 == 0}<br>{/if}
  {/each}
  <br><br>
  {JSON.stringify(grid.flat().map(x => parseInt(x.value)))}
  <br><br>
  {JSON.stringify(solution.flat().map(x => parseInt(x.value)))}
</div>
{/if}
