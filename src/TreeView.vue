<template>
  <div class="nodes">
    <div
      class="node"
      v-for="(node, index) in nodes"
      :key="index"
    >
      <div
        class="node-head"
        :class="active.toString() === node.$path.toString() ? 'active' : ''"
        @click="$emit('update:active', node.$path)"
      >
        {{ node.$value }}
      </div>
      <tree-view
        class="node-content"
        :active="active"
        :nodes="node.$children"
        @update:active="$emit('update:active', $event)"
      >
      </tree-view>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TreeView',
  props: {
    active: { require: true },
    nodes: { require: true },
  }
}
</script>

<style scoped>
.nodes {
  overflow-y: auto;
}
.node {
  padding-left: 10px;
}
.node-head {
  cursor: pointer;
  height: 2rem;
  line-height: 2rem;
  overflow-x: hidden;
  padding-left: 5%;
  text-overflow: ellipsis;
  transition: background-color .2s ease;
  white-space: nowrap;
}
.node-head:hover {
  background-color: #D6F5D6;
}
.node-head.active {
  background-color: #D6F5D6;
  color: #01BB16;
}
</style>
