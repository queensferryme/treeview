<template>
  <div id="app">
    <div id="side-bar">
      <header>
        <h1>TOC</h1>
      </header>
      <tree-view
        id="tree-view"
        :active.sync="active"
        :nodes="nodes"
      >
      </tree-view>
    </div>
    <user-content
      id="user-content"
      :active.sync="active"
      :nodes.sync="nodes"
    >
    </user-content>
  </div>
</template>

<script>
import TreeView from './TreeView'
import UserContent from './UserContent'

export default {
  name: 'app',
  components: {
    TreeView,
    UserContent
  },
  data () {
    return {
      active: [0, 0],
      nodes: []
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
#app {
  display: flex;
  height: 100%;
  overflow: hidden;
}
#side-bar {
  box-shadow: 2px 0 5px #aaa;
  height: 100%;
  width: 20%;
}
#side-bar header {
  color: #01BB16;
  height: 10%;
  text-align: center;
  width: 100%;
}
#side-bar #tree-view {
  height: 90%;
}
#user-content {
  width: 80%;
}
</style>
