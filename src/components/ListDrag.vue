<style scoped>
.list
{
  border: 1px solid black;
  padding: 8px;
}

.list-item
{
  border: 1px solid transparent;
}
.drag-out
{
  color: rgba(0, 0, 0, 0.2);
}
.hovering
{
  background-color: rgba(0, 0, 0, 0.1);
}
.hovering-top { border-top: 1px solid lightseagreen; }
.hovering-bottom { border-bottom: 1px solid lightseagreen; }

</style>

<template>
<div>
  <h3>List Drag</h3>
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


function wrapItem({
    title = '',
    values = {},
    image = null,
})
{
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
])
const Protocol = 'application/dd-item'
function onDragStart(event, item, indexItem)
{
  item.isDragOut = true
  const { dataTransfer } = event
  dataTransfer.effectAllowed = 'move'
  if(item.values != null) for(const key in item.values)
    dataTransfer.setData(key, item.values[key])
  dataTransfer.setData(Protocol, indexItem)
  if(item.image?.value != null)
    dataTransfer.setDragImage(item.image?.value ?? null, item.image?.offsetX ?? 0, item.image?.offsetY ?? 0)
}
function onDragEnd(event, item, indexItem, isInitItem)
{
  console.log('onDragEnd', event, item, indexItem, isInitItem)
  const { dataTransfer } = event
  if(isInitItem)
  {
    item.isDragOut = false
  }
  else // 拖放到的元素
  {
    // 清除样式
    item.isHovering = false
    item.isDragHoveringTop = false
    item.isDragHoveringBottom = false

    let index = parseInt(dataTransfer.getData(Protocol))
    if(isNaN(index)) index = -1

    if(index === indexItem) return

    // 交换元素位置
    const item = list.value[index]
    list.value.splice(index, 1)
    list.value.splice(indexItem, 0, item)

  }
}
function onDragEnter(event, item)
{
  const { dataTransfer } = event
  console.log(dataTransfer.getData(Protocol))
  item.isHovering = !item.isDragOut
}
function onDragLeave(event, item)
{
  item.isHovering = false
  item.isDragHoveringTop = false
  item.isDragHoveringBottom = false
}
function onDragOver(event, item)
{
  // console.log('onDragOver', event, item, indexItem)
  const isHovering = !item.isDragOut

  const { clientX, clientY, target } = event
  const bbox = target.getBoundingClientRect()
  const isHoveringTop = isHovering && clientY < bbox.top + bbox.height / 2
  const isHoveringBottom = isHovering && clientY >= bbox.top + bbox.height / 2

  item.isHovering = isHovering
  item.isDragHoveringTop = isHoveringTop
  item.isDragHoveringBottom = isHoveringBottom
}

</script>
