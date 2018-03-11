<template>
  <div id="app">
    <div class="sidebar">
      <modal @confirm="delTemplate" :id="delModalId" is-confirm>
        <template slot="title">Confirm</template>
        <template slot="subtitle">Do you confirm delete this template ?</template>
      </modal>
      <div class="row btn-group flex-spaces">
        <button class="btn-small col-4" @click="addTemplate">+</button>
        <label ref="del-btn" class="paper-btn btn-small col-4" :for="delModalId" style="text-align: center">-</label>
      </div>
      <ol class="tpl-list">
        <li v-for="item in tplList" :key="item.id" class="list-item"
          @click.self="selectTpl(item)"
        >
          <h6 class="heading-6" v-if="!item.editing"
            @dblclick="editTplName(item)"
            @click="selectTpl(item)"
          >
            <span v-show="item.active" class="badge secondary">&gt;</span>
            <span class="item-content">{{item.title}}</span>
          </h6>
          <input v-if="item.editing" type="text" v-model="item.title" placeholder="Filename"
            @keydown.enter="item.editing = false; item.active = true" v-focus
          >
        </li>
      </ol>
    </div>
    <div class="content">
      <table-view ref="tableView" :current-tpl="currentTpl"></table-view>
    </div>
  </div>
</template>

<script>
import uid from 'uid'
import Modal from '@/components/Modal'
import TableView from '@/components/TableView'

const {ipcRenderer} = require('electron')

export default {
  name: 'md-table-generator',
  data () {
    return {
      delModalId: 'modal-del',
      tplList: []
    }
  },
  computed: {
    currentTpl () {
      return this.tplList.find(item => item.active)
    }
  },
  components: {
    Modal,
    TableView
  },
  directives: {
    focus: {
      inserted (el) {
        el.focus()
      }
    }
  },
  watch: {
    tplList: {
      handler (val) {
        localStorage.setItem('tplList', JSON.stringify(val))
      },
      deep: true
    }
  },
  methods: {
    editTplName (tpl) {
      const index = this.tplList.findIndex(item => item === tpl)
      this.tplList.forEach((item, _index) => {
        item.active = _index === index
        item.editing = _index === index
      })
    },
    selectTpl (tpl) {
      const index = this.tplList.findIndex(item => item === tpl)
      if (this.tplList[index].active) return
      this.tplList.forEach((item, _index) => {
        item.active = _index === index
        item.editing = false
      })
      this.$refs.tableView.loadTpl(this.currentTpl)
    },
    addTemplate () {
      this.tplList.push({title: 'Untitled', id: uid(4), active: false, editing: false})
      this.currentItem = this.editingIndex = this.tplList.length - 1
    },
    delTemplate () {
      if (this.tplList.length === 1) return
      const index = this.tplList.findIndex(item => item.active)
      localStorage.removeItem(this.tplList[index].id)
      this.tplList.splice(index, 1)
      this.tplList[0].active = true
      this.$refs.tableView.loadTpl(this.currentTpl)
    }
  },
  created () {
    let defaultTplList = [{title: 'Untitled', id: uid(4), active: true, editing: false}]
    let storage = localStorage.tplList
    if (!storage) {
      localStorage.tplList = JSON.stringify(defaultTplList)
      this.tplList = defaultTplList
    } else {
      this.tplList = JSON.parse(storage)
    }
  },
  mounted () {
    this.$refs.tableView.loadTpl(this.currentTpl)
    ipcRenderer.on('del-tpl', () => {
      this.$refs['del-btn'].click()
    })
  }
}
</script>

<style lang="less">
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html, body{
  height: 100%;
  padding-top: 20px;
  background: #fff;
}
html{
  -webkit-app-region: drag;
}
#app{
  height: 100%;
  display: flex;
  flex-flow: row nowrap;
  padding-right: 5px;
}
.sidebar{
  height: 100%;
  width: 240px;
  overflow-y: auto;
  position: relative;
  border-right: 2px dashed;
  .btn-group{
    padding-top: 10px;
    position: fixed;
    width: 238px;
    background: #fff;
  }
  .tpl-list{
    padding-top: 60px;
    max-height: 100%;
    .list-item{
      margin-top: 4px;
      padding-left: 4px;
      height: 24px;
      cursor: pointer;
      &:hover{
        color: #0071de;
      }
      h6, input{
        display: inline-block;
        height: 100%;
      }
      .heading-6{
        display: inline-flex;
        line-height: 1.5;
        width: 100%;
      }
      input{
        font-size: .8em;
      }
      .item-content{
        flex: 1;
        text-indent: 4px;
        max-height: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        word-wrap: break-all;
        white-space: pre;
      }
    }
  }
}
.content{
  height: 100%;
  flex: 1;
  display: flex;
  flex-flow: column nowrap;
  overflow-x: auto;
}
</style>
