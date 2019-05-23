<template>
  <div>
    <editor
      ref="editor"
      v-model="content" 
      :isClear="isClear"
    ></editor>
  </div>
</template>

<script>
import editor from "./editor";
export default {
  data() {
    return {
      isClear: false,
      directory: "photo",
      content: ""
    }
  },
  components: {
    editor
  },
}
</script>
