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
    <VueBlockProperty :property="selectedBlockProperty" @save="saveProperty"/>
    <label>
      <select name="type" v-model="selectedType">
        <template v-for="type in selectBlocksType">
          <optgroup :label="type">
            <option v-for="block in filteredBlocks(type)" :value="block.name">{{block.title || block.name}}</option>
          </optgroup>
        </template>
      </select>
    </label>
    <button @click.stop="changeTitle">Change</button>
    <button @click.stop="addBlock">Add</button>
    |
    <label for="useContextMenu">
      <input type="checkbox" v-model="useContextMenu" id="useContextMenu">Use right click for Add blocks
    </label>

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
        blocks: [
          {
            name: 'text',
            title: 'Text',
            family: 'Animations',
            description: 'Show text',
            fields: [
              {
                name: 'text',
                label: 'Text',
                type: 'string',
                attr: 'property'
              },
              {
                name: 'delay',
                label: 'Delay (s)',
                type: 'number',
                attr: 'property'
              },
              {
                name: 'Show',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'Hide',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'onShow',
                type: 'event',
                attr: 'output'
              },
              {
                name: 'onHide',
                type: 'event',
                attr: 'output'
              }
            ]
          },
          {
            name: 'animation',
            title: 'Animation',
            family: 'Animations',
            description: 'Show animation',
            fields: [
              {
                name: 'animation',
                label: 'Animation',
                type: 'animation',
                attr: 'property'
              },
              {
                name: 'Play',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'Stop',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'onEnd',
                type: 'event',
                attr: 'output'
              }
            ]
          },
          {
            name: 'Chat message',
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
                name: 'onMessage',
                type: 'event',
                attr: 'output'
              }
            ]
          },
          {
            name: 'delay',
            title: 'Delay',
            family: 'Time',
            description: '',
            fields: [
              {
                name: 'delay',
                label: 'Delay (s)',
                type: 'number',
                attr: 'property',
                value: 1.0
              },
              {
                name: 'input',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'output',
                type: 'event',
                attr: 'output'
              }
            ]
          },
          {
            name: 'shortcuts',
            title: 'Shortcuts',
            family: 'Events',
            description: 'Press shortcut for call event',
            fields: [
              {
                name: 'keys',
                label: 'Activation keys',
                type: 'keys',
                attr: 'property'
              },
              {
                name: 'onPress',
                type: 'event',
                attr: 'output'
              }
            ]
          },
          {
            name: 'splitter',
            title: 'Splitter',
            family: 'Helpers',
            description: 'Press shortcut for call event',
            fields: [
              {
                name: 'input',
                type: 'event',
                attr: 'input'
              },
              {
                name: 'output',
                type: 'event',
                attr: 'output'
              },
              {
                name: 'output',
                type: 'event',
                attr: 'output'
              },
              {
                name: 'output',
                type: 'event',
                attr: 'output'
              },
              {
                name: 'output',
                type: 'event',
                attr: 'output'
              }
            ]
          }
        ],
        scene: {
          blocks: [
            {
              id: 2,
              x: -700,
              y: -69,
              name: 'Chat message',
              title: 'Chat message',
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
              x: -600,
              y: 40,
              name: 'Chat message',
              title: 'Chat message',
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
              id: 4,
              x: -157,
              y: -68.5,
              name: 'text',
              title: 'Text',
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
              originID: 2,
              originSlot: 0,
              targetID: 4,
              targetSlot: 0
            },
            {
              id: 2,
              originID: 2,
              originSlot: 0,
              targetID: 4,
              targetSlot: 1
            },
            {
              id: 3,
              originID: 3,
              originSlot: 0,
              targetID: 4,
              targetSlot: 1
            }
          ],
          container: {
            centerX: 1042,
            centerY: 140,
            scale: 1
          }
        },
        selectedBlock: null,
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
        console.log(this.selectedType)
        this.$refs.container.addNewBlock(this.selectedType)
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
        let offset = domHelper.getOffsetRect(this.$refs.container.$el)
        let x = this.contextMenu.mouseX - offset.left
        let y = this.contextMenu.mouseY - offset.top

        this.$refs.container.addNewBlock(name, x, y)
        this.closeContextMenu()
      },
      closeContextMenu () {
        this.contextMenu.isShow = false
      },
      changeTitle () {
        // console.log('change')
        // console.log(this.scene.blocks[0].title)
        this.scene.blocks[0].title = 'こんちは'
        console.log(this.scene.links)
        // console.log(this.scene.blocks[0].title)
        let scene = this.scene
        this.scene = merge({}, scene)
      }
    },
    watch: {
      blocks (newValue) {
        console.log('blocks', JSON.stringify(newValue))
        console.log('ブロックだよ')
      },
      scene (newValue) {
        console.log('scene', JSON.stringify(newValue))
        console.log('シーンだよ')
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
