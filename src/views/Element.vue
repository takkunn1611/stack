<template>
  <div class="element">
    <el-button @click="openDialog">HELLO</el-button>
    <el-dialog modal title="Hello" center :visible.sync="open" :close-on-click-modal="false" :close-on-press-escape="false" :show-close="false">
      <div class="dialog-content">
        This is message.
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button primary @click="closeDialog">OK</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Button, Dialog } from "element-ui";

@Component({
  components: {
    Button,
    Dialog
  }
})
export default class Element extends Vue {
  open = false

  openDialog() {
    this.open = true
  }

  closeDialog() {
    this.open = false
  }
}
</script>
