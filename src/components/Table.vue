<template>
  <div class="vh-100 py-5" @mousemove="trackMouse" @mouseup="mouseUpHandle">
    <table>
      <thead>
        <tr>
          <th ref="tablePersonRef" class="person-col">Person</th>
          <th v-for="day in 30" :key="day" class="day-col">
            {{ day }}
          </th>
        </tr>
      </thead>
      <tbody ref="tableBodyRef">
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
                ref="hoveredCellRef"
              >
                <div :class="hoveredCellHandleClass(idx)"></div>
              </div>
              <div
                @mousedown="
                  filledCellMouseDown($event, { itemId: item.id, index: idx })
                "
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
                <div
                  v-if="item.days[idx - 1] == 0 || item.days[idx - 1] == null"
                  :class="!hoveredCells.active ? 'day-left-handle' : ''"
                  @mousedown.stop="
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
        {
          id: 2,
          name: "Dmitry Stoyanov",
          days: [
            0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
        {
          id: 3,
          name: "Renaud Viot",
          days: [
            1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
        {
          id: 4,
          name: "Renaud Viot",
          days: [
            0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
        {
          id: 5,
          name: "Renaud Viot",
          days: [
            0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
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
      mouseStartPosY: -1,
      activeCellObj: { itemId: -1, index: -1 },
      timesMovedLeftByOneCell: 0,
      timesMovedRightByOneCell: 0,
      initialDirection: "",
      isMoving: false,
      isDragging: false,
      newRowId: -1,
      clickCellCoord: {
        topY: -1,
        bottomY: -1,
        leftX: -1,
        rightX: -1,
      },
      entryBodyCoord: {
        topY: -1,
        bottomY: -1,
        leftX: -1,
        rightX: -1,
      },
    };
  },
  mounted() {
    this.initTableBodyCoord();
  },
  methods: {
    initTableBodyCoord() {
      let tableBodyCoordTemp = this.$refs.tableBodyRef.getBoundingClientRect();
      let tablePersonCoordTemp =
        this.$refs.tablePersonRef.getBoundingClientRect();
      this.entryBodyCoord.topY = tablePersonCoordTemp.bottom;
      this.entryBodyCoord.bottomY = tableBodyCoordTemp.bottom;
      this.entryBodyCoord.leftX = Math.floor(
        tableBodyCoordTemp.x + tablePersonCoordTemp.width
      );
      this.entryBodyCoord.rightX = tableBodyCoordTemp.right;
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
      let mouseXPosition = e.clientX;
      let mouseYPosition = e.clientY;
      let movedLeftByOneCell;
      let movedRightByOneCell;
      let movedTopByOneCell;
      let movedDownByOneCell;

      // console.log('mouse x position', mouseXPosition);
      // console.log('mouse y position', mouseYPosition);
      

      if (this.isDragging && this.cellWidth) {
        if (!this.initialDirection) {
          movedLeftByOneCell =
            mouseXPosition < this.clickCellCoord.leftX - this.cellWidth;
          movedRightByOneCell =
            mouseXPosition > this.clickCellCoord.rightX + this.cellWidth;
        }

        if (movedLeftByOneCell) {
          this.initialDirection = "left";
          this.hoveredCells.active = true;
        }

        if (movedRightByOneCell) {
          this.initialDirection = "right";
          this.hoveredCells.active = true;
        }

        if (this.initialDirection === "left") {
          movedLeftByOneCell =
            mouseXPosition < this.clickCellCoord.leftX - this.cellWidth &&
            mouseXPosition > this.entryBodyCoord.leftX;
          movedRightByOneCell =
            mouseXPosition > this.clickCellCoord.rightX + this.cellWidth &&
            mouseXPosition < this.mouseStartPosX + this.cellWidth;

          if (movedLeftByOneCell) {
            this.clickCellCoord.leftX -= this.cellWidth;
            this.clickCellCoord.rightX -= this.cellWidth;

            if (this.hoveredCells.direction === "right") {
              this.timesMovedLeftByOneCell = 0;
            }
            this.timesMovedLeftByOneCell++;

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
            this.clickCellCoord.rightX += this.cellWidth;
            this.clickCellCoord.leftX += this.cellWidth;

            this.hoveredCells.direction = "right";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            if (this.hoveredCells.cellIndexes.length > 0) {
              this.hoveredCells.cellIndexes.shift();
            }
          }
        }

        if (this.initialDirection === "right") {
          movedRightByOneCell =
            mouseXPosition > this.clickCellCoord.rightX + this.cellWidth;
          movedLeftByOneCell =
            mouseXPosition < this.clickCellCoord.leftX - this.cellWidth &&
            mouseXPosition > this.mouseStartPosX - this.cellWidth;

          if (movedLeftByOneCell) {
            this.clickCellCoord.leftX -= this.cellWidth;
            this.clickCellCoord.rightX -= this.cellWidth;

            this.hoveredCells.direction = "left";
            this.hoveredCells.rowId = this.activeCellObj.itemId;

            if (this.hoveredCells.cellIndexes.length > 0) {
              this.hoveredCells.cellIndexes.pop();
            }
          }
          if (movedRightByOneCell) {
            if (this.hoveredCells.direction === "left") {
              this.timesMovedRightByOneCell = 0;
            }

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
            this.clickCellCoord.rightX += this.cellWidth;
            this.clickCellCoord.leftX += this.cellWidth;

            this.hoveredCells.cellIndexes.push(
              this.activeCellObj.index + this.timesMovedRightByOneCell
            );
            this.timesMovedRightByOneCell++;
          }
        }
      }
      if (this.isMoving && this.cellWidth && this.cellHeight) {
        let hoveredCellsRef = this.$refs.hoveredCellRef;
        let hoveredCellRightPos;
        let hoveredCellLeftPos;

        // console.log('hoveredCellsRef 0', hoveredCellsRef[0].getBoundingClientRect().left);
        // console.log('hoveredCellsRef 1', hoveredCellsRef[1].getBoundingClientRect().left);
        // console.log('hoveredCellsRef 2', hoveredCellsRef[2].getBoundingClientRect().left);


        if (hoveredCellsRef.length > 1) {
          let left = this.entryBodyCoord.rightX;
          let right = 0;
          hoveredCellsRef.forEach(hovCell => {
            if (hovCell.getBoundingClientRect().left < left) {
              left = hovCell.getBoundingClientRect().left;
            }
            if (hovCell.getBoundingClientRect().right > right) {
              right = hovCell.getBoundingClientRect().right;
            }
          })
          hoveredCellRightPos = right;
          hoveredCellLeftPos = left;
        } else {
          hoveredCellRightPos =
            hoveredCellsRef[0].getBoundingClientRect().right;
          hoveredCellLeftPos = hoveredCellsRef[0].getBoundingClientRect().left;
        }

        // console.log('hoveredCellRightPos', hoveredCellRightPos);
        // console.log('hoveredCellLeftPos', hoveredCellLeftPos);
        // console.log('this.entryBodyCoord.leftX', this.entryBodyCoord.leftX);

        movedLeftByOneCell =
          mouseXPosition < this.clickCellCoord.leftX - this.cellWidth &&
          hoveredCellLeftPos > this.entryBodyCoord.leftX + this.cellWidth / 3;
        movedRightByOneCell =
          mouseXPosition > this.clickCellCoord.rightX + this.cellWidth &&
          hoveredCellRightPos < this.entryBodyCoord.rightX - this.cellWidth / 3;

        movedTopByOneCell =
          mouseYPosition < this.clickCellCoord.topY &&
          mouseYPosition > this.entryBodyCoord.topY + this.cellHeight / 2;
        movedDownByOneCell =
          mouseYPosition > this.clickCellCoord.bottomY &&
          mouseYPosition < this.entryBodyCoord.bottomY - this.cellHeight / 2;        

        this.newRowId = this.hoveredCells.rowId;

        if (movedLeftByOneCell) {
          this.clickCellCoord.leftX -= this.cellWidth;
          this.clickCellCoord.rightX -= this.cellWidth;

          this.hoveredCells.cellIndexes = this.hoveredCells.cellIndexes.map(
            (cell) => cell - 1
          );
        }

        if (movedRightByOneCell) {
          this.clickCellCoord.leftX += this.cellWidth;
          this.clickCellCoord.rightX += this.cellWidth;

          this.hoveredCells.cellIndexes = this.hoveredCells.cellIndexes.map(
            (cell) => cell + 1
          );
        }

        if (movedTopByOneCell) {
          this.clickCellCoord.topY -= this.cellHeight;
          this.clickCellCoord.bottomY -= this.cellHeight;

          this.hoveredCells.rowId =
            this.hoveredCells.rowId - 1 < 0 ? 0 : this.hoveredCells.rowId - 1;
        }

        if (movedDownByOneCell) {
          this.clickCellCoord.topY += this.cellHeight;
          this.clickCellCoord.bottomY += this.cellHeight;

          this.hoveredCells.rowId = this.hoveredCells.rowId + 1;
        }
      }
    },
    mouseDownLeftHandle(e, cellObj) {
      e.preventDefault();
      let targetCoord = e.target.getBoundingClientRect();

      this.changeMouseCursor("ew-resize");
      this.cellWidth = e.target.parentElement.offsetWidth;
      this.cellHeight = e.target.parentElement.offsetHeight;
      this.mouseStartPosX = e.clientX;
      this.activeCellObj = cellObj;
      this.isDragging = true;

      this.clickCellCoord = {
        topY: targetCoord.top,
        bottomY: targetCoord.bottom,
        leftX: targetCoord.x,
        rightX: targetCoord.right,
      };
    },
    filledCellMouseDown(e, cellObj) {
      e.preventDefault();

      let targetCoord = e.target.getBoundingClientRect();

      this.changeMouseCursor("grabbing");
      this.cellWidth = e.target.parentElement.offsetWidth;
      this.cellHeight = e.target.parentElement.offsetHeight;
      this.mouseStartPosX = e.clientX;
      this.mouseStartPosY = e.clientY;
      this.activeCellObj = cellObj;
      this.isMoving = true;

      this.clickCellCoord = {
        topY: targetCoord.top,
        bottomY: targetCoord.bottom,
        leftX: targetCoord.x,
        rightX: targetCoord.right,
      };

      this.hoveredCells.active = true;
      this.hoveredCells.rowId = this.activeCellObj.itemId;

      let listItem = this.list.find(
        (item) => item.id == this.activeCellObj.itemId
      );

      let cellIndexesToFill = [];

      let i = this.activeCellObj.index;

      if (i > 0) {
        while (listItem.days[i] != 0 && i >= 0) {
          if (!cellIndexesToFill.includes(i)) {
            cellIndexesToFill.push(i);
          }
          console.log("in loop: i = ", i);
          console.log("in loop: listItem.days[i] = ", listItem.days[i]);
          i--;
        }
      }

      i = this.activeCellObj.index;

      while (listItem.days[i] != 0 && i <= 29) {
        if (!cellIndexesToFill.includes(i)) {
          cellIndexesToFill.push(i);
        }
        console.log("in loop: i = ", i);
        console.log("in loop: listItem.days[i] = ", listItem.days[i]);
        i++;
      }

      cellIndexesToFill.sort();

      this.hoveredCells.cellIndexes = cellIndexesToFill;
    },
    mouseUpHandle() {
      this.changeMouseCursor("auto");
      if (!this.activeCellObj) {
        return;
      }
      let activeItemIndex = this.list.findIndex(
        (item) => item.id === this.activeCellObj.itemId
      );

      if (this.isDragging) {
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
      }

      if (this.isMoving) {
        let newActiveItemIndex = this.list.findIndex(
          (item) => item.id === this.newRowId
        );

        let listItem = this.list.find(
          (item) => item.id == this.activeCellObj.itemId
        );

        console.log("list item", listItem);

        let cellIndexesToFill = [];

        let i = this.activeCellObj.index;

        if (i > 0) {
          while (listItem.days[i] != 0 && i >= 0) {
            if (!cellIndexesToFill.includes(i)) {
              cellIndexesToFill.push(i);
            }
            console.log("in loop: i = ", i);
            console.log("in loop: listItem.days[i] = ", listItem.days[i]);
            i--;
          }
        }

        i = this.activeCellObj.index;

        while (listItem.days[i] != 0 && i <= 29) {
          if (!cellIndexesToFill.includes(i)) {
            cellIndexesToFill.push(i);
          }
          console.log("in loop: i = ", i);
          console.log("in loop: listItem.days[i] = ", listItem.days[i]);
          i++;
        }

        cellIndexesToFill.sort();

        if (newActiveItemIndex >= 0) {
          cellIndexesToFill.forEach((index) => {
            this.list[activeItemIndex].days[index] = 0;
          });

          this.hoveredCells.cellIndexes.forEach((index) => {
            this.list[newActiveItemIndex].days[index] = 1;
          });
        }
      }

      this.resetDragState();
    },
    resetDragState() {
      this.cellWidth = 0;
      this.cellHeight = 0;
      this.activeCellObj = null;
      this.timesMovedLeftByOneCell = 0;
      this.timesMovedRightByOneCell = 0;
      this.hoveredCells = { rowId: -1, active: false, cellIndexes: [] };
      this.mouseStartPosX = -1;
      this.mouseStartPosY = -1;
      this.initialDirection = "";
      this.isMoving = false;
      this.isDragging = false;
      this.newRowId = -1;
      this.clickCellCoord = {};
    },
    changeMouseCursor(cursorType) {
      document.body.style.cursor = cursorType;
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

.day-marker span {
  height: 100%;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
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
