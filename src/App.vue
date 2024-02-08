<script setup lang="ts">
import { ref } from 'vue';

class Grid2D {
  public grid: Point[][] = [];
  public start: Point | null = null;
  public neighbors: Point[] = [];
  public important: Point[] = [];
  constructor(public width: number, public height: number) {
    for (let y = 0; y < height; y++) {
      const row: Point[] = [];
      for (let x = 0; x < width; x++) {
        row.push(new Point(x, y, false));
      }
      this.grid.push(row);
    }
    this.start = this.grid[width / 2][height / 2];
    this.update();
  }
  public update() {
    this.neighbors = [];
    this.important = [];
    {
      const rookPath = (origin: Point, xDir: number, yDir: number) => {
        if (xDir !== 0 && yDir !== 0) {
          throw new Error('Invalid direction');
        }
        const isX = xDir !== 0;
        let offset = isX ? origin.y : origin.x;
        let dir = isX ? xDir : yDir;
        let ob = dir === -1 ? -1 : isX ? this.width : this.height;
        for (let i = origin[isX ? 'x' : 'y'] + dir; i !== ob; i += dir) {
          console.log(i, offset, isX);
          const cell = isX ? this.grid[offset][i] : this.grid[i][offset];
          if (cell.state) {
            break;
          }
          this.neighbors.push(cell);
          const side1 = isX ? this.grid[offset + 1]?.[i] : this.grid[i]?.[offset + 1];
          const side2 = isX ? this.grid[offset - 1]?.[i] : this.grid[i]?.[offset - 1];
          console.log(side1, side2);
          if ((side1 && !side1.state) || (side2 && !side2.state)) {
            this.important.push(cell);
          }
        }
      }

      const bishopPath = (origin: Point, xDir: number, yDir: number) => {
        if (xDir === 0 || yDir === 0) {
          throw new Error('Invalid direction');
        }
        let x = origin.x + xDir;
        let y = origin.y + yDir;
        while (x >= 0 && x < this.width && y >= 0 && y < this.height) {
          const cell = this.grid[y][x];
          if (cell.state) {
            break;
          }
          const intermediate1 = this.grid[y - yDir]?.[x];
          const intermediate2 = this.grid[y]?.[x - xDir];
          if (!(intermediate1 && !intermediate1.state) && !(intermediate2 && !intermediate2.state)) {
            break;
          }
          this.neighbors.push(cell);
          this.important.push(cell); // there will always be an open side due to the intermediate checks
          x += xDir;
          y += yDir;
        }
      }

      const down = rookPath(this.start!, 0, 1);
      const up = rookPath(this.start!, 0, -1);
      const left = rookPath(this.start!, -1, 0);
      const right = rookPath(this.start!, 1, 0);

      const upLeft = bishopPath(this.start!, -1, -1);
      const upRight = bishopPath(this.start!, 1, -1);
      const downLeft = bishopPath(this.start!, -1, 1);
      const downRight = bishopPath(this.start!, 1, 1);
    }
  }
}

class Point {
  constructor(public x: number, public y: number, public state: boolean) { }
}

const grid = ref<Grid2D>(new Grid2D(10, 10));
</script>

<template>
  <main>
    <div class="flex" v-for="(row, ri) in grid.grid" :key="JSON.stringify(row)">
      <button class="w-8 h-8 border border-neutral-800 flex justify-center items-center" v-for="(cell, ci) in row"
        :key="`${cell}|${ci}|${ri}`" @contextmenu.prevent="() => { grid.start = cell; grid.update() }"
        @click="() => { grid.grid[ri][ci].state = !grid.grid[ri][ci].state; grid.update() }" :class="{
          'bg-neutral-800': cell.state == true && cell != grid.start, 'bg-emerald-600': cell == grid.start, 'bg-indigo-900':
            grid.neighbors.includes(cell) && !grid.important.includes(cell), 'bg-indigo-700': grid.important.includes(cell)
        }">
      </button>
    </div>
    {{ JSON.stringify(grid.neighbors) }}
    <br>
    {{ JSON.stringify(grid.important) }}
  </main>
</template>
