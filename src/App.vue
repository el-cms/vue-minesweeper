<template>
  <div>
    <!-- New game and options -->
    <div v-if="!started">
      <label for="size">Size: </label><input id="size" type="number" v-model="size" name="size" size="3"/> x {{size}} -
      <label for="bombs">Mines: </label><input id="bombs" type="number" v-model="bombs" name="bombs" size="3"/>
      <button @click="start">Commencer <i class="i-play"></i></button>
    </div>
    <!-- /New game and options -->

    <!-- Board section -->
    <div class="board" v-else>

      <!-- Grid -->
      <div class="grid" :class="{debug}">
        <!-- Stats -->
        <div class="stats">
          <div>Flags set: {{flagsSet}}/{{flags}}</div>
          <div>Blocks left: {{ left - flagsSet}}/{{size * size}}</div>
        </div>

        <!-- Line -->
        <div class="line" v-for="line in grid">
          <!-- Cell -->
          <div class="cell"
               :class="cellClass(c)"
               @click="trigger(c)"
               @contextmenu.prevent="flag(c)"
               v-for="c in line">
            {{cellContent(c)}}
          </div>
          <!-- /Cell -->
        </div>
        <!-- /Line -->

        <!-- Status -->
        <div class="status">
          <div v-if="gameOver" :class="{won, lost}" class="game-over">
            <p>Game over</p>
            <small>{{won ? 'Vous avez gagné !' : 'Vous avez perdu'}}</small>
            <button @click="started=false">Recommencer</button>
          </div>
        </div>
        <!-- /Status -->

      </div>
      <!-- /Grid -->

    </div>
    <!-- /Board section -->
  </div>
</template>

<script>

  export default {
    name: 'app',
    data () {
      return {
        grid: [],       // Game grid
        lost: false,    // Has the player lost the game ?
        won: false,     // Has he won ?
        started: false, // Has the game started ?
        flags: 0,       // Number of flags available
        bombs: 10,      // Number of bombs
        size: 10,       // Grid width/height, in cells
        debug: true     // Enable debugging ?
      }
    },
    computed: {
      /**
       * Calculates the number of uncovered cells
       */
      left () {
        var left = 0
        const size = this.size
        for (let i = 0; i < size; i++) {
          for (let j = 0; j < size; j++) {
            if (!this.grid[i][j].revealed) {
              left++
            }
          }
        }
        return left
      },
      /**
       * Is the game over ?
       */
      gameOver () {
        return this.won || this.lost
      },
      /**
       * Number of flags set
       */
      flagsSet () {
        var nbFlags = 0
        const size = this.size
        for (let i = 0; i < size; i++) {
          for (let j = 0; j < size; j++) {
            if (this.grid[i][j].flagged) {
              nbFlags++
            }
          }
        }
        return nbFlags
      }
    },
    methods: {
      /**
       * Board initialization
       */
      start () {
        const grid = []           // Final grid
        const cells = []          // Cells list for easy loopings
        const size = this.size    // Grid side size
        const bombs = this.bombs  // Number of bombs

        // Cell creation
        for (let i = 0; i < size; i++) {
          for (let j = 0; j < size; j++) {
            cells.push({
              // Is the bomb flagged (simpleclick)
              flagged: false,
              // Is the bomb triggered (dblclick)
              triggered: false,
              // Is there a bomb on it ?
              bomb: false,
              // Nb of bombs around
              bombsAround: 0,
              // Is the cell content visible ?
              revealed: false,
              // Position
              x: j,
              y: i
            })
          }
        }

        // Placing bombs
        const nbCells = cells.length
        for (let i = 0; i < bombs; i++) {
          let selectedCell = cells[Math.floor(Math.random() * nbCells)]
          let alreadyBomb = selectedCell.bomb
          while (alreadyBomb === true) {
            let selectedCell = cells[Math.floor(Math.random() * nbCells)]
            alreadyBomb = selectedCell.bomb
          }
          selectedCell.bomb = true
        }

        // Grid creation
        for (let i = 0; i < size; i++) {
          let line = []
          for (let j = 0; j < size; j++) {
            // Finding cell in list:
            line.push(cells.shift())
          }
          grid.push(line)
        }

        // Computing number of mines around cells
        for (let i = 0; i < size; i++) {
          for (let j = 0; j < size; j++) {
            let x = grid[i][j].x
            let y = grid[i][j].y
            let sum = 0
            // ^ y
            //   +----+----+----+
            //   | NW | N  | NE |
            //   +----+----+----+
            //   | W  |    | E  |
            //   +----+----+----+
            //   | SW | S  | SE |
            //   +----+----+----+
            //                   >x

            // NW
            sum += (x > 0 && y > 0) ? (grid[y - 1][x - 1].bomb ? 1 : 0) : 0
            // N
            sum += (y > 0) ? grid[y - 1][x].bomb : 0
            // NE
            sum += (x + 1 < size && y > 0) ? grid[y - 1][x + 1].bomb : 0
            // W
            sum += (x > 0) ? grid[y][x - 1].bomb : 0
            // E
            sum += (x + 1 < size) ? grid[y][x + 1].bomb : 0
            // SW
            sum += (x > 0 && y + 1 < size) ? grid[y + 1][x - 1].bomb : 0
            // S
            sum += (y + 1 < size) ? grid[y + 1][x].bomb : 0
            // SE
            sum += (x + 1 < size && y + 1 < size) ? grid[y + 1][x + 1].bomb : 0

            grid[i][j].bombsAround = sum
          }
        }

        // Initializ game vars
        this.grid = grid
        this.bombs = bombs
        this.flags = bombs
        this.points = 0
        this.lost = false
        this.won = false
        this.started = true
      },
      /**
       * Triggered on a simple click
       * @param cell
       */
      flag (cell) {
        if (!cell.flagged && this.flagsSet < this.flags && !cell.revealed) {
          cell.flagged = true
        } else if (cell.flagged) {
          cell.flagged = false
        }
        this.testWin()
      },
      /**
       * Checks if the game should end
       */
      testWin () {
        if (this.left - this.flagsSet === 0) {
          this.end(true)
        }
      },
      /**
       * Triggered on a click
       * @param cell
       */
      trigger (cell) {
        cell.flagged = false
        cell.triggered = true
        cell.revealed = true
        if (cell.bomb) {
          cell.exploded = true
          this.end(false)
          this.lost = true
        } else if (cell.bombsAround === 0) {
          this.reveal(cell)
        }

        this.testWin()
      },
      /**
       * Computes the cell content depending on the state
       * @param cell
       * @returns {String}
       */
      cellContent (cell) {
        return cell.revealed && !cell.bomb && cell.bombsAround > 0 ? cell.bombsAround : ''
      },
      /**
       * Uncover blank cells
       * @param startingCell
       * @param previousCellBombCount
       */
      reveal (startingCell, previousCellBombCount = 0) {
        if (previousCellBombCount === 0) {
          // Current cell position
          const x = startingCell.x
          const y = startingCell.y
          // Testing north cell:
          const cells = [
            y - 1 >= 0 ? this.grid[y - 1][x] : null,        // North
            y + 1 < this.size ? this.grid[y + 1][x] : null, // South
            x + 1 < this.size ? this.grid[y][x + 1] : null, // East
            x - 1 >= 0 ? this.grid[y][x - 1] : null,        // West
            x - 1 >= 0 && y - 1 >= 0 ? this.grid[y - 1][x - 1] : null,              // North-West
            x + 1 < this.size && y - 1 >= 0 ? this.grid[y - 1][x + 1] : null,       // North-East
            x - 1 >= 0 && y + 1 < this.size ? this.grid[y + 1][x - 1] : null,       // South-West
            x + 1 < this.size && y + 1 < this.size ? this.grid[y + 1][x + 1] : null // South-East
          ]
          for (let c in cells) {
            let cell = cells[c]
            if (cell
              && !cell.revealed
              && !cell.bomb
              && !cell.flagged
            ) {
              cell.revealed = true
              // Running north
              this.reveal(cell, cell.bombsAround)
            }
          }
        }
      },
      /**
       * Determines cell classes
       * @param cell
       * @returns {{nobomb: boolean, flagged: boolean|computed.gameOver, bomb: boolean, notFoundBomb: boolean|*, exploded: boolean}}
       */
      cellClass (cell) {
        const debugString = `x-${cell.x}--y-${cell.y}--bomb-${cell.bomb}--revealed-${cell.revealed}--bombsaround-${cell.bombsAround}`
        const classes = {
          // Discovered, safe cell
          nobomb: cell.revealed && !cell.bomb,
          // Undiscovered, Flagged cell
          flagged: cell.flagged && (!cell.revealed || this.gameOver),
          // Bomb cell, when the game is over
          bomb: cell.bomb && this.lost && cell.triggered && cell.exploded,
          // Visible cell: when the game is over
          notFoundBomb: cell.bomb && cell.revealed && !cell.exploded && this.lost,
          //visible: this.running && (this.lost || this.won),
          exploded: cell.exploded
        }
        classes[debugString] = this.debug
        return classes
      },
      /**
       * Ends the game
       * @param status
       */
      end (status) {
        if (status && !this.lost) {
          console.log('WON')
          this.won = true
        } else {
          console.log('LOST')
          this.lost = true
        }
        for (let i = 0; i < this.size; i++) {
          for (let j = 0; j < this.size; j++) {
            this.grid[j][i].revealed = true
            //this.grid[j][i].flagged = false
          }
        }
      }
    }
  }
</script>

<style lang="scss">
  @import "./stylesheets/app";
</style>
