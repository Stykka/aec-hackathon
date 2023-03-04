<template>
  <div
    class="flex flex-col mx-auto justify-between h-screen w-screen max-h-screen overflow-hidden"
  >
    <h1>Speckle Viewer</h1>
    <label for="objectUrl">Object URL: </label>
    <input type="text" id="objectUrl" :value="objectUrl">
    <h2>Select material</h2>
    <select v-model="selectedMaterial">
      <option disabled value="">Please select one</option>
      <option v-for="material in Object.keys(materialList)">
        {{ material }}
      </option>
    </select>
    <span>Selected material: {{ selectedMaterial }}</span>
    <button @click="assignMaterial(selectedMaterial, $event)">Assign</button>
    <table>
      <th>Material
        <td>{{ material }}</td>
      </th>
      <th>Volume
        <td>{{ volume }}</td>
      </th>
      <th>ID
        <td>{{ id }}</td>
      </th>
    </table>
  </div>
 
  <div ref="canvas" class="canvas p-2"></div>

</template>

<script lang="ts">
import { SelectionEvent, SpeckleObject, Viewer, ViewerEvent } from "@speckle/viewer";
import { onMounted, ref } from "vue";
import materials from '../materials.json'

export default {
  data() {
    return {
      materialList: [],
      selectedMaterial: 'A'
    }
  },
  created() {
    this.materialList = materials
  },
  setup() {
    const canvas = ref<HTMLElement>();
    const objectUrl = ref("https://dev.stykka.com/streams/d73703f251/objects/edfb08a14c58f39d0d9d23a5d675b299");
    const material = ref()
    const volume = ref()
    const id = ref()

    function assignMaterial(material, event) {
      //TODO update speckle object?
      console.log(material)
    }

    onMounted(async () => {
      const viewer = new Viewer(canvas.value!)
      let allObjects = viewer.getDataTree()

      let propertyData = viewer.getObjectProperties()
      propertyData.forEach(function(item) {
        if (item['key'] === 'volume') {
          console.log("item: ", item)
        }
      })

      viewer.on(ViewerEvent.ObjectClicked, (selectionInfo: SelectionEvent) => {
        if (selectionInfo) {
          let objectId = [selectionInfo.hits[0].object.id as string]
          // Object was clicked. Focus in on it
          viewer.zoom(objectId)
          viewer.highlightObjects(objectId)
          let obj = selectionInfo.hits[0].object
          id.value = obj.id
          volume.value = obj.volume
          material.value = obj.material
        }
        else {
          // No object clicked. Restore focus to entire scene
          viewer.zoom()	
          viewer.resetHighlight()
          id.value = null
          volume.value = null
          material.value = null
        }
      })
      await viewer.loadObject(objectUrl.value)
    })

    return {
      assignMaterial,
      canvas,
      objectUrl,
      volume,
      id,
      material
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