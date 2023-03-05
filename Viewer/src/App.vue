<template>
  <div
    class="flex flex-col items-start overflow-hidden"
  >
    <h1 class="text-2xl font-semibold text-white">Speckle Viewer</h1>

    <label for="objectUrl" class="pb-1">Object URL</label>
    <input type="text" class="mb-2 hover:select-all text-xs text-gray-500" id="objectUrl" :value="objectUrl">
    
    <label for="selectedMaterial" class="pb-1">Component Material</label>
    <select v-model="selectedMaterial" class="mb-2 hover:select-all text-xs text-gray-500" id="selectedMaterial">
      <option disabled value="">Please select one</option>
      <option v-for="material in Object.keys(materialList)">
        {{ material }}
      </option>
    </select>

    <span class="mt-2 mb-1 hover:select-all font-semibold text-sm text-gray-400">Currently selected: {{ selectedMaterial }}</span>

    <button type="button" class="bg-yellow-500 text-gray-800 rounded-lg p-1 pl-4 pr-4 mt-1 mb-3 ..." @click="assignMaterial(selectedMaterial, $event)">Assign</button>
    
    <table class="justify-between flex flex-auto">
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

  <table class="justify-between flex flex-auto">
      <th>Global Warming Potential
        <td>{{ gwp }}</td>
      </th>
    </table>

</template>

<script lang="ts">
import { SelectionEvent, SpeckleObject, Viewer, ViewerEvent } from "@speckle/viewer";
import ObjectLoader from '@speckle/objectloader';
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
    const selectedObject = ref()
    selectedObject.value = null
    const material = ref()
    const volume = ref()
    const id = ref()

    const gwp = ref(0)

    async function assignMaterial(m, event) {
      //TODO update speckle object?
      console.log(m)
      selectedObject.value["material"] = m
      material.value = m
      gwp.value += materials[m]
      console.log(selectedObject.value)

      const loader = new ObjectLoader( { serverUrl: "https://dev.stykka.com/", streamId: "d73703f251", objectId: selectedObject.value.id } )
      let obj = await loader.getAndConstructObject((e) => console.log('Progress', e))
      obj["material"] = m
      console.log(obj["material"])
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
          selectedObject.value = obj
          id.value = obj.id
          volume.value = obj.volume
          material.value = obj.material
        }
        else {
          // No object clicked. Restore focus to entire scene
          viewer.zoom()	
          viewer.resetHighlight()
          selectedObject.value = null
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
      selectedObject,
      volume,
      id,
      material,
      gwp
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