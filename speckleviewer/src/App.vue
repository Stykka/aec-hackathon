<template>
  <p>Speckle Viewer</p>
  <div
    class="flex flex-col mx-auto justify-between h-screen w-screen max-h-screen overflow-hidden"
  >
    <label for="objectUrl">Object URL: </label>
    <input type="text" id="objectUrl" :value="objectUrl">
    <h2>Select</h2>
    <select v-model="selected">
      <option disabled value="">Please select one</option>
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
    <span>Selected: {{ selected }}</span>
  </div>
 
  <div ref="canvas" class="canvas"></div>

</template>

<script lang="ts">
import { SelectionEvent, Viewer, ViewerEvent } from "@speckle/viewer"
import { onMounted, ref } from "vue";

export default {
  data() {
    return {
      selected: 'A'
    }
  },
  setup() {
    const canvas = ref<HTMLElement>();
    const objectUrl = ref("https://dev.stykka.com/streams/aa6b496028/objects/65a8fa46c214142b2ff426f886ffcaff");

    onMounted(async () => {
      const viewer = new Viewer(canvas.value!)
      viewer.on(ViewerEvent.ObjectClicked, (selectionInfo: SelectionEvent) => {
        if (selectionInfo) {
          // Object was clicked. Focus in on it
          viewer.zoom([selectionInfo.hits[0].object.id as string])
          viewer.highlightObjects([selectionInfo.hits[0].object.id as string])
        }
        else {
          // No object clicked. Restore focus to entire scene
          viewer.zoom()	
        }
      })
      await viewer.loadObject(objectUrl.value)
    })

    return {
      canvas,
      objectUrl
    }
  }
}
</script>

<style>
  .canvas {
    width: 800px;
    height: 600px;
    background-color: cornflowerblue;
  }
  #objectUrl {
    width: 500px;
  }
</style>