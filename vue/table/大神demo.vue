<script>
  class Store {
    constructor (parent, code, name, leaf) {
      this.isLeaf = leaf
      this.parent = parent
      this.code = code
      this.name = name
      this.children = []
      this.data = null
    }

    size () {
      return this.isLeaf ? 1 : this.children.reduce((pre, curr) => pre + curr.size(), 0)
    }

    push (child) {
      this.children.push(child)
    }

    find (name) {
      return this.children.find(child => child.name === name)
    }

    setData (data) {
      this.data = data
    }

    setLeaf (leaf) {
      this.isLeaf = leaf
    }

    getChildren () {
      return this.children
    }
  }

  function formatData (datas, headers, group) {
    let store = new Store(null, null, null, false)
    datas.forEach(data => dueMap(data, group, store))
    const ret = []
    const buf = []
    const done = []
    dueArray(store, ret, group, headers, buf, done)
    return ret
  }

  function dueArray (store, arr, group, headers, rearray, darray) {
    store.getChildren().forEach(child => {
      if (child.isLeaf) {
        let ret = []
        const data = child.data
        for (let header of headers) {
          if (group.findIndex(g => g === header.code) >= 0) {
            const ss = rearray.find(r => r.code === header.code)
            const done = darray.find(r => r.code === header.code)
            if (!done.done) {
              ret.push({
                v: ss.name,
                r: ss.size()
              })
              done.done = true
            }
          } else {
            ret.push({
              v: data[header.code],
              r: 1
            })
          }
        }
        arr.push(ret)
      } else {
        rearray.push(child)
        darray.push({code: child.code, done: false})
        dueArray(child, arr, group, headers, rearray, darray)
        darray.pop()
        rearray.pop()
      }
    })
  }

  function dueMap (data, group, store) {
    let copyStore = store
    for (let item of group) {
      const v = data[item]
      const child = copyStore.find(v)
      if (child) {
        copyStore = child
      } else {
        const childStore = new Store(copyStore, item, v, false)
        copyStore.push(childStore)
        copyStore = childStore
      }
    }
    const childStore = new Store(copyStore, null, null, true)
    childStore.setData(data)
    copyStore.push(childStore)
  }

  export default {
    name: 'row-span-demo',
    data () {
      return {
        headers: [
          {code: 'prodCode'},
          {code: 'schCode'},
          {code: 'orderCode'},
          {code: 'ctrCode'}
        ],
        group: ['prodCode', 'schCode'],
        data: [
          {
            prodCode: 'PROJ1',
            schCode: 'SCH1',
            orderCode: 'PO1',
            ctrCode: 'CTR1'
          },
          {
            prodCode: 'PROJ1',
            schCode: 'SCH2',
            orderCode: 'PO2',
            ctrCode: ''
          },
          {
            prodCode: 'PROJ1',
            schCode: 'SCH1',
            orderCode: 'PO3',
            ctrCode: 'CTR3'
          }, {
            prodCode: 'PROJ2',
            schCode: 'SCH4',
            orderCode: 'PO4',
            ctrCode: ''
          }, {
            prodCode: 'PROJ1',
            schCode: 'SCH2',
            orderCode: 'PO5',
            ctrCode: 'CTR5'
          }, {
            prodCode: 'PROJ1',
            schCode: 'SCH1',
            orderCode: 'PO6',
            ctrCode: 'CTR6'
          }
        ]
      }
    },
    computed: {
      computedDate () {
        return formatData(this.data, this.headers, this.group)
      }
    }
  }
</script>
<template>
  <div>
    {{computedDate}}
    <table style="border: 1px solid">
      <tr v-for="row in computedDate">
        <td  style="border: 1px solid" v-for="cell in row" :rowspan="cell.r">{{cell.v}}</td>
      </tr>
    </table>
  </div>
</template>
