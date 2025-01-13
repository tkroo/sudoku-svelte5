<script lang="ts">
  import { getSudoku } from 'sudoku-gen';
  
  type Cell = { value: any; enabled: boolean; candidates: number[] };
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
  let candidatesMode = $state(false);

  function generateBoard(level: Difficulty) {
    sudoku = getSudoku(level);
    grid = makeGrid(sudoku.puzzle);
    solution = makeGrid(sudoku.solution);
  }

  generateBoard('easy');

  let solved = $derived(grid.flat().map(x => x.value).join('') == solution.flat().map(x => x.value).join(''));

  let curRow = $state(0);
  let curCol = $state(0);
  let cellSelected = $derived(grid[curRow][curCol].value);

  const nums: number[] = [7,8,9,4,5,6,1,2,3];
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
        row.push({value:parseInt(puzzle[i * 9 + j]), enabled:parseInt(puzzle[i * 9 + j]) == 0 ? true : false, candidates: [ ]});
      }
      b.push(row);
    }
    return b;
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
    if (e.key=='d') {
      showDebug = !showDebug;
    }
    if (e.key=='c') {
      candidatesMode = !candidatesMode;
    }
    if (e.key=='h') {
      showHighlight = !showHighlight;
    }
    if (e.key=='e') {
      showErrors = !showErrors;
    }
    if (e.key === 'Delete') {
      if(candidatesMode) {
        grid[curRow][curCol].candidates.pop();
      } else {
        grid[curRow][curCol].value = 0;

      }
    }
    
    if (parseInt(e.key) >= 1 && parseInt(e.key) <= 9) {
      let num: number = parseInt(e.key);
      if(grid[curRow][curCol].enabled) {
        if(candidatesMode) {
          let c = grid[curRow][curCol].candidates;
          if(c.includes(num)) {
            grid[curRow][curCol].candidates = c.filter(x => x != num)
          } else {
            grid[curRow][curCol].candidates.push(num)
          }
        } else {
          grid[curRow][curCol].value = num;
        }
      }
    }

  }

  // function handleClick(v: number,r,c) {
  //   curRow = r;
  //   curCol = c;
  // }
  function handleNumsClick(value) {
    if(grid[curRow][curCol].enabled) {
      grid[curRow][curCol].value = value;
    }
  }

  function handleButtonClick(level) {
      generateBoard(level);
  }

  function solvePuzzle() {
    // console.log('not implemented yet')
  }

</script>

<svelte:body on:keydown={handleKeyDown} />
<main>
  <h1>sudoku</h1>
  <div class="controls">
    {#each difficultyLevel as level}
      <button
        class:highlight={level == sudoku.difficulty && !solved}
        onclick={() => handleButtonClick(level)}
      >{level}</button>
    {/each}
    <!-- <button onclick={solvePuzzle}>solve</button> -->
    <!-- <button onclick={() => showHighlight = !showHighlight} class:highlight={showHighlight}>highlight similar</button> -->
    <!-- <button onclick={() => showErrors = !showErrors} class:highlight={showErrors}>toggle show errors</button> -->
    <label for="showHighlight">highlight (h) <input type="checkbox" id="showHighlight" bind:checked={showHighlight}></label>
    <label for="showErrors">errors (e) <input type="checkbox" id="showErrors" bind:checked={showErrors}></label>
    <label for="candidatesMode">candidate entry (c) <input type="checkbox" id="candidatesMode" bind:checked={candidatesMode}></label>
  </div>

  <div class="main-cols">
  
    <div class="board" class:solved={solved}>
      {#each grid as row, r}
          {#each row as cell, c}
            <button class="cell"
              onclick={() => {curRow = r; curCol = c}}
              class:highlight={showHighlight && cellSelected==cell.value && cell.value != 0}
              class:active={r==curRow && c==curCol}
              class:error={showErrors && (solution[r][c].value != cell.value) && (cell.value != 0)}
            >
              <span class="candidates" class:activecandy={candidatesMode && r==curRow && c==curCol}>{cell.candidates.join('')}</span>
              <span class="value" class:given={!grid[r][c].enabled} class:hidden={cell.value==0}>{cell.value}</span>
            </button>
          {/each}
      {/each}
    </div>

    <div class="nums-wrap">
      {#each nums as value, index}
        <button
          class="num"
          class:done={countAppearances[value] == 9}
          onclick={() => handleNumsClick(value)}>{value}
        </button>
      {/each}
      <button class="num num-wide" onclick={() => handleNumsClick(0)}>clear</button>
    </div>

  </div>
</main>
<footer>
  <p><a href="https://github.com/tkroo/sudoku-svelte5">source</a></p>
  <p>puzzles generated by <a href="https://github.com/petewritescode/sudoku-gen">sudoku-gen</a></p>
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
