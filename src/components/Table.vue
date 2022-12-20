<template>
  <div>
    <table @mouseover="trackMouse" @mouseup="mouseUpHandle">
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
          <td class="person-col">{{ item.name }}</td>
          <td v-for="(day, idx) in item.days" :key="idx" class="day-col">
            <div class="h-100">
              <div
                class="day-marker h-100"
                :class="day != 0 ? 'day-filled' : ''"
                v-if="day != 0"
              >
                <div
                  @mousedown="
                    mouseDownHandle($event, {
                      itemId: item.id,
                      day: day,
                      index: idx,
                    })
                  "
                  v-if="item.days[idx - 1] == 0"
                  class="day-left-handle"
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
            0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0,
          ],
        },
      ],
      cellWidth: 0,
      cellHeight: 0,
      mouseStartPosX: 0,
      activeCellObj: null,
      timesMovedByOneCell: 0,
    };
  },
  methods: {
    trackMouse(e) {
      e.preventDefault();
      if (this.cellWidth) {
        let mouseXPosition = e.clientX;
        let movedLeftByOneCell = mouseXPosition <= (this.mouseStartPosX - this.cellWidth);
        // let movedRightByOneCell = mouseXPosition >= (this.mouseStartPosX + this.cellWidth);
        console.log("active cell obj", this.activeCellObj);
        if (movedLeftByOneCell) {
          this.mouseStartPosX = e.clientX;
          this.timesMovedByOneCell++;
          let activeItemIndex = this.list.findIndex(item => item.id === this.activeCellObj.itemId);
          this.list[activeItemIndex].days.splice(this.activeCellObj.index - this.timesMovedByOneCell, 1, 1);
        }
      }
    },
    mouseDownHandle(e, cellObj) {
      e.preventDefault();
      // console.log('event', e);
      // console.log('event target', e.target);
      // console.log('event target parent', e.target.parentElement);
      // console.log('event target parent width', e.target.parentElement.offsetWidth);
      // console.log('event target parent height', e.target.parentElement.offsetHeight);
      this.cellWidth = e.target.parentElement.offsetWidth;
      this.cellHeight = e.target.parentElement.offsetHeight;
      this.mouseStartPosX = e.clientX;
      this.activeCellObj = cellObj;
    },
    mouseUpHandle() {
      console.log("mouse up");
      this.cellWidth = 0;
      this.cellHeight = 0;
      this.activeCellObj = null;
      this.timesMovedByOneCell = 0;
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
  background: rgb(126, 160, 251);
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
</style>
