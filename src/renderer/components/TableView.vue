<template>
  <div>
    <div class="operation-bar">
      <button class="btn-small" @click="addColumn">Add Column</button>
      <button class="btn-small" @click="delColumn">Delete Column</button>
      <button class="btn-small" @click="addRow">Add Row</button>
      <button class="btn-small" @click="delRow">Delete Row</button>
      <button class="btn-small btn-secondary" @click="copyContent">Copy Content</button>
    </div>
    <div class="table-view">
      <table id="table">
        <tbody>
          <tr>
            <th class="col-mark"></th>
            <th @click="selectCol(index)" class="col-mark" 
              v-for="(item, index) in columns" :key="index"
              :class="{'active': index === acCol}"
            >{{index}}</th>
          </tr>
          <tr>
            <th class="col-mark"></th>
            <th data-row="0" :data-col="index" v-title="item.text"
              v-for="(item, index) in columns" :key="item.key" contenteditable
              :class="{'active': index === acCol}"
            ></th>
          </tr>
        </tbody>
        <tbody>
          <tr v-for="(row, index) in rows" :key="row.key">
            <td @click="selectRow(index)" class="row-mark" :class="{active: index === acRow}">{{index}}</td>
            <td contenteditable 
              v-for="(item2, index2) in row.data" :key="item2.key"
              :data-row="index" :data-col="index2"
              :class="{active: index === acRow || index2 === acCol}"
            >
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import uid from 'uid'
import { clipboard } from 'electron'

const genArray = (length, placeholder) => {
  return Array.apply(null, new Array(length)).map(e => {
    return {text: placeholder || '', key: uid(4)}
  })
}

export default {
  name: 'table-view',
  data () {
    return {
      columns: genArray(5, 'title'),
      rows: [],
      observer: null,
      acRow: null,
      acCol: null,
      config: { subtree: true, characterData: true, childList: true },
      targetNode: null
    }
  },
  props: {
    currentTpl: Object
  },
  methods: {
    loadTpl (tpl) {
      const storage = localStorage.getItem(tpl.id)
      this.pauseObserve()
      if (!storage) {
        let defaultTpl = genArray(5, 'title')
        localStorage.setItem(tpl.id, JSON.stringify(defaultTpl))
        this.columns = defaultTpl
      } else {
        this.columns = JSON.parse(storage)
      }
      this.resetRow()
    },
    resetRow () {
      this.rows = genArray(4).map(e => ({key: uid(4), data: genArray(this.columns.length)}))
    },
    pauseObserve () {
      this.observer.disconnect()
      setTimeout(() => {
        this.observer.observe(this.targetNode, this.config)
      }, 0)
    },
    addColumn () {
      this.pauseObserve()

      let index = this.acCol === null ? (this.columns.length - 1) : this.acCol
      this.columns.splice(index + 1, 0, {text: 'title', key: uid(4)})
      this.rows.forEach(row => {
        row.data.splice(index + 1, 0, {text: '', key: uid(4)})
      })
    },
    addRow () {
      this.pauseObserve()
      this.rows.push({key: uid(4), data: genArray(this.columns.length)})
    },
    delColumn () {
      if (this.columns.length === 1) return
      this.pauseObserve()
      let index = this.acCol === null ? (this.columns.length - 1) : this.acCol

      this.columns.splice(index, 1)
      this.rows.forEach(row => row.data.splice(index, 1))
      this.acCol = null
    },
    delRow () {
      if (this.rows.length === 1) return
      this.pauseObserve()
      const index = this.acRow === null ? this.rows.length - 1 : this.acRow

      this.rows.splice(index, 1)
      this.acRow = null
    },
    selectCol (index) {
      this.acRow = null
      this.acCol = this.acCol === index ? null : index
    },
    selectRow (index) {
      this.acCol = null
      this.acRow = this.acRow === index ? null : index
    },
    processNormalMutation (mutation) {
      let target = mutation.target
      let parent = target.parentElement
      const {row, col} = parent.dataset
      if (parent.nodeName === 'TH') {
        this.columns[col].text = target.textContent
        localStorage.setItem(this.currentTpl.id, JSON.stringify(this.columns))
      } else {
        this.rows[row].data[col].text = target.textContent
      }
    },
    processBoundaryMutation (mutationList) {
      const childListMutation = mutationList.filter(mutation => mutation.type === 'childList')[0]
      const characterDataMutation = mutationList.filter(mutation => mutation.type === 'characterData')[0]

      let parent = childListMutation.target
      let target = characterDataMutation.target
      const {row, col} = parent.dataset
      if (parent.nodeName === 'TH') {
        this.columns[col].text = target.textContent
      } else {
        this.rows[row].data[col].text = target.textContent
      }
    },
    handleMutation (mutationList) {
      const hasChildList = ~mutationList.findIndex(mutation => mutation.type === 'childList')
      if (!hasChildList) {
        this.processNormalMutation(mutationList[0])
      } else {
        this.processBoundaryMutation(mutationList)
      }
    },
    copyContent () {
      let theadStr = `|${this.columns.map(column => column.text).join('|')}|\n`
      theadStr += `|${this.columns.map(column => '-').join('|')}|\n`

      let tbodyStr = ''
      this.rows.forEach(row => {
        tbodyStr += `|${row.data.map(item => item.text).join('|')}|\n`
      })
      clipboard.writeText(theadStr + tbodyStr)
      // eslint-disable-next-line
      let myNotification = new Notification('Table Generator', {
        body: 'Copy successfully~'
      })
      setTimeout(() => {
        myNotification.close()
      }, 2000)
    }
  },
  watch: {
    columns: {
      handler (val) {
        localStorage.setItem(this.currentTpl.id, JSON.stringify(val))
      }
    }
  },
  directives: {
    title: {
      inserted (el, binding) {
        el.textContent = binding.value
      }
    }
  },
  mounted () {
    this.targetNode = document.getElementById('table')

    this.observer = new MutationObserver(this.handleMutation)
    this.observer.observe(this.targetNode, this.config)
  },
  beforeDestroy () {
    this.observer.disconnect()
    this.observer = null
    this.targetNode = null
  }
}
</script>

<style lang="less">
.table-view{
  padding-top: 60px;
  padding-left: 15px;
  flex: 1;
  table{
    border-collapse: collapse;
    width: auto;
    max-width: auto;
  }
  th, td{
    outline: 0;
    padding: 10px;
    min-width: 100px;
    max-width: 200px;
    font-size: 14px;
    &.active{
      background: #eee;
    }
  }
  .col-mark, .row-mark{
    font-weight: 500;
    cursor: pointer;
    border: 0;
    font-style: italic;
    border-color: #fff;
  }
  .row-mark{
    width: 50px;
  }
}
.operation-bar{
  position: fixed;
  padding-top: 10px;
  padding-left: 15px;
  width: 100%;
  background: #fff;
}
</style>
