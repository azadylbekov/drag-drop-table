<template>
  <div class="vh-100" @mousemove="trackMouse" @mouseup="mouseUpHandle">
    <table ref="tableRef">
      <thead>
        <tr>
          <th class="person-col">Person</th>
          <th v-for="day in 30" :key="day" class="day-col">
            {{ day }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr class="entry-row" v-for="item in list" :key="item.id">
          <td ref="personColRef" class="person-col">{{ item.name }}</td>
          <td v-for="(day, idx) in item.days" :key="idx" class="day-col">
            <div class="h-100 position-relative">
              <div
                v-if="
                  item.id === hoveredCells.rowId &&
                  hoveredCells.cellIndexes.includes(idx)
                "
                class="hovered-cell"
              >
                <div :class="hoveredCellHandleClass(idx)"></div>
              </div>
              <div
                class="day-marker h-100"
                :class="day != 0 ? 'day-filled' : ''"
                v-if="
                  day != 0 &&
                  !(
                    hoveredCells.rowId == item.id &&
                    hoveredCells.cellIndexes.includes(idx)
                  )
                "
              >
                <!-- hoveredCells.direction == 'right' -->
                <div
                  v-if="item.days[idx - 1] == 0 || item.days[idx - 1] == null"
                  :class="!hoveredCells.active ? 'day-left-handle' : ''"
                  @mousedown="
                    mouseDownLeftHandle($event, {
                      itemId: item.id,
                      day: day,
                      index: idx,
                    })
                  "
                ></div>
                <span>{{ day }}</span>
              </div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    msg: String,
  },
  data() {
    return {
      headers: ["person", "date"],
      list: [
        {
          id: 0,
          name: "Azamat Adylbekov",
          days: [
            0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
        {
          id: 1,
          name: "Dmitry Fedorov",
          days: [
            1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
      ],
      hoveredCells: {
        rowId: -1,
        active: false,
        direction: "left",
        cellIndexes: [],
      },
      cellWidth: 0,
      cellHeight: 0,
      mouseStartPosX: -1,
      mouseLastPosX: -1,
      activeCellObj: { itemId: -1, day: -1, index: -1 },
      timesMovedLeftByOneCell: 0,
      timesMovedRightByOneCell: 0,
      activeDirection: "",
      initialDirection: "",
      tableDaysWidth: 0,
      isInitDirectionSet: false,
    };
  },
  mounted() {
    this.calculateTableDaysWidth();
    // window.addEventListener(
    //   "resize",
    //   () => {
    //     this.calculateTableDaysWidth();
    //   },
    //   true
    // );
  },
  methods: {
    calculateTableDaysWidth() {
      let tableWidth = this.$refs.tableRef.offsetWidth;
      let personColWidth = this.$refs.personColRef[0].offsetWidth;
      this.tableDaysWidth = Math.floor(tableWidth - personColWidth);
    },
    hoveredCellHandleClass(idx) {
      return this.hoveredCells.active &&
        this.hoveredCells.cellIndexes[0] === idx
        ? "day-left-handle"
        : "";
      // if (this.hoveredCells.direction === "right") {
      //   let lastCellIndex = this.hoveredCells.cellIndexes.length - 1;
      //   return this.hoveredCells.active &&
      //     this.hoveredCells.cellIndexes[lastCellIndex] === idx
      //     ? "day-left-handle"
      //     : "";
      // }
    },
    trackMouse(e) {
      e.preventDefault();
      // console.log("tracking mouse");
      if (this.cellWidth) {
        let mouseXPosition = e.clientX;
        let movedLeftByOneCell;
        let movedRightByOneCell;
        let mouseCheckpointPosX;

        if (!this.initialDirection) {
          movedLeftByOneCell =
            mouseXPosition <= this.mouseStartPosX - this.cellWidth;
          movedRightByOneCell =
            mouseXPosition >= this.mouseStartPosX + this.cellWidth;
        }

        if (movedLeftByOneCell) {
          this.initialDirection = "left";
        }

        if (movedRightByOneCell) {
          this.initialDirection = "right";
        }

        if (this.initialDirection === "left") {
          if (this.mouseLastPosX !== -1) {
            mouseCheckpointPosX =
              mouseXPosition > this.mouseStartPosX - this.cellWidth
                ? this.mouseStartPosX
                : this.mouseLastPosX;
          } else {
            mouseCheckpointPosX = this.mouseStartPosX;
          }
          movedLeftByOneCell =
            mouseXPosition <= mouseCheckpointPosX - this.cellWidth;
          movedRightByOneCell =
            mouseXPosition >= mouseCheckpointPosX + this.cellWidth;

          if (movedLeftByOneCell) {
            this.mouseLastPosX = e.clientX;
            if (this.hoveredCells.direction === "right") {
              this.timesMovedLeftByOneCell = 0;
            }
            this.timesMovedLeftByOneCell++;

            this.hoveredCells.active = true;
            this.hoveredCells.direction = "left";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            if (this.activeCellObj.index - this.timesMovedLeftByOneCell < 0) {
              return;
            }
            this.hoveredCells.cellIndexes.unshift(
              this.activeCellObj.index - this.timesMovedLeftByOneCell
            );
          }

          if (movedRightByOneCell) {
            this.mouseLastPosX = e.clientX;

            this.hoveredCells.active = true;
            this.hoveredCells.direction = "right";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            if (this.hoveredCells.cellIndexes.length > 0) {
              this.hoveredCells.cellIndexes.shift();
            }
          }
        }

        if (this.initialDirection === "right") {
          if (this.mouseLastPosX !== -1) {
            mouseCheckpointPosX =
              mouseXPosition < this.mouseStartPosX - this.cellWidth
                ? this.mouseStartPosX
                : this.mouseLastPosX;
          } else {
            mouseCheckpointPosX = this.mouseStartPosX;
          }

          movedLeftByOneCell =
            mouseXPosition <= mouseCheckpointPosX - this.cellWidth;
          movedRightByOneCell =
            mouseXPosition >= mouseCheckpointPosX + this.cellWidth;

          if (movedLeftByOneCell) {
            this.mouseLastPosX = e.clientX;

            this.hoveredCells.active = true;
            this.hoveredCells.direction = "left";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            if (this.hoveredCells.cellIndexes.length > 0) {
              this.hoveredCells.cellIndexes.pop();
            }
          }
          if (movedRightByOneCell) {
            this.mouseLastPosX = e.clientX;
            if (this.hoveredCells.direction === "left") {
              this.timesMovedRightByOneCell = 0;
            }

            this.hoveredCells.active = true;
            this.hoveredCells.direction = "right";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            let activeItemIndex = this.list.findIndex(
              (item) => item.id === this.activeCellObj.itemId
            );

            let isLastActiveCell =
              this.list[activeItemIndex].days[
                this.activeCellObj.index + this.timesMovedRightByOneCell + 1
              ] === 0;
            if (isLastActiveCell) {
              return;
            }
            this.hoveredCells.cellIndexes.push(
              this.activeCellObj.index + this.timesMovedRightByOneCell
            );
            this.timesMovedRightByOneCell++;
          }
        }
      }
    },
    mouseDownLeftHandle(e, cellObj) {
      e.preventDefault();
      this.changeMouseCursor("ew-resize");
      this.cellWidth = e.target.parentElement.offsetWidth;
      this.cellHeight = e.target.parentElement.offsetHeight;
      this.mouseStartPosX = e.clientX;
      this.activeCellObj = cellObj;
    },
    mouseUpHandle() {
      this.changeMouseCursor("auto");
      if (!this.activeCellObj) {
        return;
      }
      let activeItemIndex = this.list.findIndex(
        (item) => item.id === this.activeCellObj.itemId
      );

      if (this.initialDirection === "left") {
        this.hoveredCells.cellIndexes.forEach((index) => {
          this.list[activeItemIndex].days[index] = 1;
        });
      }

      if (this.initialDirection === "right") {
        this.hoveredCells.cellIndexes.forEach((index) => {
          this.list[activeItemIndex].days[index] = 0;
        });
      }

      this.resetDragState();
    },
    changeMouseCursor(cursorType) {
      document.body.style.cursor = cursorType;
    },
    resetDragState() {
      this.cellWidth = 0;
      this.cellHeight = 0;
      this.activeCellObj = null;
      this.timesMovedLeftByOneCell = 0;
      this.timesMovedRightByOneCell = 0;
      this.hoveredCells = { rowId: -1, active: false, cellIndexes: [] };
      this.mouseLastPosX = -1;
      this.mouseStartPosX = -1;
      this.initialDirection = null;
    },
  },
};
</script>

<style scoped>
table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
  table-layout: fixed;
}

td,
th {
  border: 1px solid #000;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #ddd;
}

.person-col {
  width: 20%;
}

.day-col {
  text-align: center;
  padding: 0;
  height: inherit;
}

.day-filled {
  background: rgb(24, 79, 233);
  color: #fff;
}

.entry-row {
  height: 1px;
}

.day-marker {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.day-left-handle {
  width: 5px;
  height: 100%;
  background-color: red;
  position: absolute;
  left: 0;
  cursor: ew-resize;
  transition: 0.3s;
}

.day-left-handle:hover {
  box-shadow: 0px 0px 15px red;
}

.hovered-cell {
  background-color: rgba(24, 79, 233, 0.5);
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
</style>
