<style scoped>
.list
{
  border: 2px solid black;
  padding: 8px;
}

.list-item
{
  border: 2px solid transparent;
}
.drag-out
{
  color: rgba(0, 0, 0, 0.2);
}
.hovering
{
  background-color: rgba(0, 0, 0, 0.1);
}
.hovering-top { border-top: 2px solid lightseagreen; }
.hovering-bottom { border-bottom: 2px solid lightseagreen; }

</style>

<template>
<div>
  <h3>List Drag</h3>
  <p>列表内元素的拖拽排列</p>
  <div class="list">
    <template v-for="(item, indexItem) in list" :key="item.id">
      <div draggable="true"
           @dragstart="onDragStart($event, item, indexItem)"
           @dragend="onDragEnd($event, item, indexItem, true)"
           @drop="onDragEnd($event, item, indexItem, false)"
           @dragenter="onDragEnter($event, item)"
           @dragleave="onDragLeave($event, item)"
           @dragover.prevent="onDragOver($event, item)"
           class="list-item"
           :class="listItemStyle[indexItem]"
      >
        {{ item.title }}
      </div>
    </template>
  </div>
</div>
</template>

<script setup>

import {ref, computed} from 'vue'


const imgDefault = new Image()
imgDefault.src = 'test-item.png'
function wrapItem({
    title = '',
    values = {},
    image = null,
})
{
  if(image == null)
  {
    // const canvas = document.createElementNS('http://www.w3.org/1999/xhtml', 'canvas')
    // canvas.height = canvas.width = 100
    // const ctx = canvas.getContext('2d')
    // ctx.fillStyle = 'lightseagreen'
    // ctx.fillRect(0, 0, 100, 100)
    // // ctx.fillStyle = 'white'
    // // ctx.font = '20px sans-serif'
    // // ctx.textAlign = 'center'
    // // ctx.textBaseline = 'middle'
    // ctx.fillText(title, 50, 50)
    // image = { value: canvas, offsetX: 50, offsetY: 50 }

    image = {
      value: imgDefault,
      offsetX: 15,
      offsetY: 15,
    }
  }

  return {
    id: Math.random() + '-' + Math.random() + '-' + Math.random(),
    title, values, image,
    isDragOut: false,
    isHovering: false,
    isDragHoveringTop: false,
    isDragHoveringBottom: false,
  }
}
const listItemStyle = computed(() => {
  const ret = []
  for(const item of list.value)
  {
    let style = ''
    if(item.isDragOut)
      style += ' drag-out'
    if(item.isHovering)
      style += ' hovering'
    if(item.isDragHoveringTop)
      style += ' hovering-top'
    if(item.isDragHoveringBottom)
      style += ' hovering-bottom'

    ret.push(style.trim())
  }
  return ret
})

const list = ref([
  wrapItem({ title: 'i1', values: {}, image: null }),
  wrapItem({ title: 'i2', values: {}, image: null }),
  wrapItem({ title: 'i3', values: {}, image: null }),
  wrapItem({ title: 'i4', values: {}, image: null }),
  wrapItem({ title: 'i5', values: {}, image: null }),
  wrapItem({ title: 'i6', values: {}, image: null }),
])
const ProtocolIndex = 'application/dd-index'
const DataId = Math.random() + '-' + Math.random() + '-' + Math.random()
const ProtocolId = 'application/dd-id-' + DataId // 用来校验是否是当前列表的元素
function applyId(event)
{
  event.dataTransfer.setData(ProtocolId, '')
}
function checkId(event)
{
  return event.dataTransfer.types.includes(ProtocolId)
}

function onDragStart(event, item, indexItem)
{
  item.isDragOut = true
  applyId(event)
  const { dataTransfer } = event
  dataTransfer.effectAllowed = 'move'
  if(item.values != null) for(const key in item.values)
    dataTransfer.setData(key, item.values[key])
  dataTransfer.setData(ProtocolIndex, indexItem)
  if(item.image?.value != null)
    dataTransfer.setDragImage(item.image?.value ?? null, item.image?.offsetX ?? 0, item.image?.offsetY ?? 0)
}
function onDragEnd(event, item, indexItem, isInitItem)
{
  const { dataTransfer } = event
  if(isInitItem)
  {
    item.isDragOut = false
  }
  else // 拖放到的元素
  {
    // 检查当前元素位置和拖拽到的元素位置
    // 把当前元素挪动到指定元素位置上
    // 这其中还要检查是拖拽到了上半部分还是下半部分
    // 如果拖拽到了指定元素的上半部分, 则把当前元素插入到指定元素的前面
    // 如果拖拽到了指定元素的下半部分, 则把当前元素插入到指定元素的后面

    const isHovering = item.isHovering
    const isDragHoveringTop = item.isDragHoveringTop
    const isDragHoveringBottom = item.isDragHoveringBottom
    // 清除样式
    item.isHovering = false
    item.isDragHoveringTop = false
    item.isDragHoveringBottom = false

    const indexDrag = parseInt(dataTransfer.getData(ProtocolIndex))
    if(isNaN(indexDrag)) return
    let indexDrop = indexItem
    if(indexDrag === indexDrop) return
    if(isDragHoveringBottom) indexDrop++
    if(indexDrag === indexDrop) return

    const itemDrag = list.value[indexDrag]
    if(indexDrop < indexDrag)
    {
      list.value.splice(indexDrag, 1)
      list.value.splice(indexDrop, 0, itemDrag)
    }
    else if(indexDrop > indexDrag)
    {
      list.value.splice(indexDrag, 1)
      list.value.splice(indexDrop - 1, 0, itemDrag)
    }
  }
}
function onDragEnter(event, item)
{
  if(!checkId(event)) return
  const { dataTransfer } = event
  item.isHovering = !item.isDragOut
}
function onDragLeave(event, item)
{
  if(!checkId(event)) return
  item.isHovering = false
  item.isDragHoveringTop = false
  item.isDragHoveringBottom = false
}
function onDragOver(event, item)
{
  if(!checkId(event)) return
  const isHovering = !item.isDragOut

  const { clientX, clientY, target } = event
  const bbox = target.getBoundingClientRect()
  const isHoveringTop = isHovering && clientY < bbox.top + bbox.height / 2
  const isHoveringBottom = isHovering && clientY >= bbox.top + bbox.height / 2

  event.dataTransfer.dropEffect = isHovering ? 'move' : 'none'

  item.isHovering = isHovering
  item.isDragHoveringTop = isHoveringTop
  item.isDragHoveringBottom = isHoveringBottom
}

</script>
