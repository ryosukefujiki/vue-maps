<template>
  <div id="app">
    <VueBlocksContainer
      @contextmenu.native="showContextMenu"
      @click.native="closeContextMenu"
      ref="container"
      :blocksContent="blocks"
      :scene.sync="scene"
      @blockSelect="selectBlock"
      @blockDeselect="deselectBlock"
      class="container"/>
    <label>
      <select name="type"  v-model="selectedId">
            <option v-for="block in scene.blocks" :value="block.id" v-model="blockId">
              {{block.id}}
            </option>
      </select>
    </label>
    <input v-model="changetitle" placeholder="change-title">
    <button @click.stop="changeTitle">Change</button>

    |

    <label>
      <select name="type" v-model="selectedType">
        <template v-for="type in selectBlocksType">
          <optgroup :label="type">
            <option v-for="block in filteredBlocks(type)" :value="block.name">{{block.title || block.name}}</option>
          </optgroup>
        </template>
      </select>
    </label>

    <input v-model="addCard" placeholder="title">
    <button @click.stop="addBlock">Add</button>
    |
    <label for="useContextMenu">
      <input type="checkbox" v-model="useContextMenu" id="useContextMenu">Use right click for Add blocks
    </label>

    <button @click.stop="scaleEnlarge">Enlarge</button>
    <button @click.stop="scaleShrink">Shrink</button>

    <ul id="contextMenu" ref="contextMenu" tabindex="-1" v-show="contextMenu.isShow"
        @blur="closeContextMenu"
        :style="{top: contextMenu.top + 'px', left: contextMenu.left + 'px'}">
      <template v-for="type in selectBlocksType">
        <li class="label">{{type}}</li>
        <li v-for="block in filteredBlocks(type)"
            @click="addBlockContextMenu(block.name)">{{block.title || block.name}}
        </li>
      </template>
    </ul>
  </div>
</template>

<script>
  import merge from 'deepmerge'

  import VueBlocksContainer from './components/VueBlocksContainer'
  import VueBlockProperty from './components/VueBlockProperty'
  import domHelper from './helpers/dom'

  export default {
    name: 'App',
    components: {
      VueBlocksContainer,
      VueBlockProperty
    },
    data: function () {
      return {
        blockId: this.blockId,
        message: this.message,
        changetitle: this.changetitle,
        addCard: this.addCard,
        blocks: [
          {
            name: 'Card',
            family: 'Events',
            description: '',
            fields: [
              {
                name: 'message',
                label: 'Activation message',
                type: 'string',
                attr: 'property'
              },
              {
                name: 'input',
                type: '',
                attr: 'input'
              },
              {
                name: 'output',
                type: '',
                attr: 'output'
              }
            ]
          }
        ],
        scene: {
          blocks: [
            {
              id: 1,
              x: -700,
              y: -69,
              name: 'Card',
              title: 'Title',
              values: {
                property: [
                  {
                    name: 'message',
                    type: 'string'
                  }
                ]
              }
            },
            {
              id: 2,
              x: -600,
              y: 40,
              name: 'Card',
              title: 'Title',
              values: {
                property: [
                  {
                    name: 'message',
                    type: 'string'
                  }
                ]
              }
            },
            {
              id: 3,
              x: -157,
              y: -68.5,
              name: 'Card',
              title: 'Title',
              values: {
                property: {
                  text: {
                    label: 'Text',
                    type: 'string'
                  }
                }
              }
            }
          ],
          links: [
            {
              id: 1,
              originID: 1,
              originSlot: 0,
              targetID: 3,
              targetSlot: 0
            },
            {
              id: 2,
              originID: 1,
              originSlot: 0,
              targetID: 2,
              targetSlot: 0
            },
            {
              id: 3,
              originID: 2,
              originSlot: 0,
              targetID: 2,
              targetSlot: 0
            }
          ],
          container: {
            centerX: 1042,
            centerY: 140,
            scale: 1
          }
        },
        selectedBlock: null,
        selectedId: this.selectedId,
        selectedType: 'delay',
        useContextMenu: true,
        contextMenu: {
          isShow: false,
          mouseX: 0,
          mouseY: 0,
          top: 0,
          left: 0
        }
      }
    },
    computed: {
      selectedBlockProperty () {
        if (!this.selectedBlock || !this.selectedBlock.values || !this.selectedBlock.values.property) {
          return null
        }

        return this.selectedBlock.values.property
      },
      selectBlocksType () {
        return this.blocks.map(b => {
          return b.family
        }).filter((value, index, array) => {
          return array.indexOf(value) === index
        })
      }
    },
    methods: {
      selectBlock (block) {
        console.log('select', block)
        this.selectedBlock = block
      },
      deselectBlock (block) {
        console.log('deselect', block)
        this.selectedBlock = null
      },
      filteredBlocks (type) {
        return this.blocks.filter(value => {
          return value.family === type
        })
      },
      addBlock () {
        // console.log(this.selectedType)
        // this.$refs.container.addNewBlock(this.selectedType)
        this.$refs.container.addNewBlock('Card', this.addCard)
      },
      saveProperty (val) {
        console.log(val)

        let scene = this.scene
        let block = scene.blocks.find(b => {
          return b.id === this.selectedBlock.id
        })
        block.values.property = val

        this.scene = merge({}, scene)
      },
      showContextMenu (e) {
        if (!this.useContextMenu) return
        if (e.preventDefault) e.preventDefault()

        this.contextMenu.isShow = true
        this.contextMenu.mouseX = e.x
        this.contextMenu.mouseY = e.y

        this.$nextTick(function () {
          this.setMenu(e.y, e.x)
          this.$refs.contextMenu.focus()
        })
      },
      setMenu (top, left) {
        let border = 5
        let contextMenuEl = this.$refs.contextMenu
        let containerElRect = this.$refs.container.$el.getBoundingClientRect()
        let largestWidth = containerElRect.right - contextMenuEl.offsetWidth - border
        let largestHeight = containerElRect.bottom - contextMenuEl.offsetHeight - border

        console.log(this.$refs.container)
        console.log(containerElRect)

        if (left > largestWidth) left = largestWidth
        if (top > largestHeight) top = largestHeight

        this.contextMenu.top = top
        this.contextMenu.left = left
      },
      addBlockContextMenu (name) {
        console.log(this.addCard)
        let offset = domHelper.getOffsetRect(this.$refs.container.$el)
        let x = this.contextMenu.mouseX - offset.left
        let y = this.contextMenu.mouseY - offset.top

        this.$refs.container.addNewBlock(name, this.addCard,x, y)
        this.closeContextMenu()
      },
      closeContextMenu () {
        this.contextMenu.isShow = false
      },
      changeTitle () {
        // console.log('change')
        // console.log(this.scene.blocks[0].title)
        for (var i = 0; i < this.scene.blocks.length; i++) {
          if (this.scene.blocks[i].id == this.selectedId) {
            console.log(this.selectedId)
            console.log('を変更します')
            this.scene.blocks[i].title = this.changetitle
          }
            // console.log('hoge')
        }
        // this.scene.blocks[0].title = this.message
        // console.log(this.scene.links)
        // console.log(this.scene.blocks[0].title)
        let scene = this.scene
        this.scene = merge({}, scene)
      },
      scaleEnlarge () {
        this.scene.container.scale += 0.2
        let scene = this.scene
        this.scene = merge({}, scene)
      },
      scaleShrink () {
        this.scene.container.scale -= 0.2
        let scene = this.scene
        this.scene = merge({}, scene)
      }
    },
    watch: {
      blocks (newValue) {
        console.log('blocks', JSON.stringify(newValue))
        // console.log('ブロックだよ')
      },
      scene (newValue) {
        console.log('scene', JSON.stringify(newValue))
        // console.log('シーンだよ')
      }
    }
  }
</script>

<style lang="less">
  html, body {
    margin: 0;
    padding: 0;
  }

  html {
    width: 100vw;
    height: 100vh;
  }

  body {
    width: 100%;
    height: 100%;
  }

  #app {
    width: ~"calc(100% - 40px)";
    height: ~"calc(100% - 40px)";
    padding: 20px 0 0 20px;
  }

  .container {
    width: 100%;
    height: ~"calc(100% - 50px)";
    border: 1px solid black;
  }

  #contextMenu {
    position: absolute;
    z-index: 1000;
    background: white;
    border: 1px solid black;
    padding: 5px;
    margin: 0;

    li {
      &.label {
        color: gray;
        font-size: 90%;
      }
      list-style: none;
    }

    &:focus {
      outline: none;
    }
  }
</style>
