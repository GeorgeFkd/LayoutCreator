<template>
  <PropsAccordion class="area-props" :accordion="accordion">
    <div>
      <AreaAccordionItem v-for="a in areaPath" :key="a.id" :area="a" :name="a.name" :accordion="accordion" />

      <!--
    <div class="area-type">{{ area.type === 'div' ? area.display : area.type }} props</div>
    <PropsAccordionItem name="display" :heading="`Display (${area.display})`" :accordion="accordion">
      <AreaContentProps :area="area" />
    </PropsAccordionItem>
      -->

      <template v-if="area.display === 'grid'">
        <PropsAccordionItem
          name="explicit-grid"
          heading="Explicit Grid"
          :independent="true"
          :start-opened="true"
          :accordion="accordion"
        >
          <AreaGridTemplateProps :area="area" />
          <AreaGridGapProps :area="area" />
        </PropsAccordionItem>
      </template>
      <button @click="exportLayouts">Export All Layouts</button>
      <button @click="saveLayouts">Send all Layouts to Backend</button>

      <PropsAccordionItem
        v-if="area.display === 'flex'"
        name="flex-props"
        heading="Flex Props"
        :independent="true"
        :accordion="accordion"
      >
        <AreaFlexProps :area="area" />
      </PropsAccordionItem>

      <PropsAccordionItem
        v-if="area.parent && area.parent.display === 'grid'"
        name="self-grid"
        heading="Self Placement"
        :accordion="accordion"
        :independent="true"
      >
        <AreaSelfGridProps :area="area" />
      </PropsAccordionItem>

      <PropsAccordionItem
        v-if="area.parent && area.parent.display === 'flex'"
        name="self-flex"
        heading="Self Flex"
        :accordion="accordion"
        :independent="true"
      >
        <AreaSelfFlexProps :area="area" />
      </PropsAccordionItem>

      <!--
      <PropsAccordionItem
        name="tree"
        :independent="true"
        :heading="`Area Tree`"
        :closed-heading="`${area.name}`"
        :accordion="accordion"
        class="tree-item"
      >
        <AreaTree :area="mainArea" />
      </PropsAccordionItem>
      -->
      <!--     
      <PropsAccordionItem v-if="area.parent" name="type" heading="Type" :accordion="accordion">
        <AreaTypeSelect v-if="area.parent" :model-value="area.type" @update:modelValue="onUpdateType" />
      </PropsAccordionItem> 
      -->

      <PropsAccordionItem
        v-if="area.children.length"
        name="children"
        :independent="true"
        :start-opened="true"
        heading="Children"
        :accordion="accordion"
        class="child-accordion"
      >
        <AreaAccordionItem v-for="a in area.children" :key="`child-${a.id}`" :area="a" />
      </PropsAccordionItem>
    </div>
  </PropsAccordion>
</template>

<script setup lang="ts">
// import AreaTree from './AreaTree.vue'

import { areaToReactCSS } from '../../generateCode'
import { useAppState, setCurrentArea, removeArea, layouts } from '../../store.js'
import JSZip from 'jszip'
import { saveAs } from 'file-saver'
import { computed } from 'vue'
let { currentArea, mainArea } = $(useAppState())

const { area } = defineProps<{ area }>()

let accordion = ref({ active: 'explicit-grid' })
let currentGrid = computed(() => area.grid)
let currentFlex = computed(() => area.flex)

function sendFilesToBackend(fileStrings) {}

function saveLayouts() {
  const { _rawValue } = layouts
  console.log('The raw value of layouts: ', _rawValue)
  const layoutsInXml = _rawValue.map((layout) => areaToReactCSS(layout, {}))
  const wrappedLayouts = `
  <layouts>
    ${layoutsInXml.reduce((acc, layout) => acc + layout + '\n', '')}
    </layouts>
  `
  console.log('Wrapped Layouts', wrappedLayouts)
  //get the value of the layouts stored in the store
  //convert them to xml files
  //send them to the backend
}

//it works
async function exportLayouts() {
  console.log('====Exporting Layouts=====')
  console.log('The layouts being exported: ', layouts)
  //get the value of the layouts stored in the store
  const { _rawValue } = layouts
  console.log('The raw value of layouts: ', _rawValue)
  //convert them to XML
  const layoutsInExportFormat = _rawValue.map((layout) => areaToReactCSS(layout, {}))
  console.log('Layouts in export format: ', layoutsInExportFormat)
  //save as zip to PC
  const zip = new JSZip()
  layoutsInExportFormat.forEach((layoutXml, index) => {
    const num = index + 1
    zip.file('layout' + num + '.xml', layoutXml)
  })
  const content = await zip.generateAsync({ type: 'blob' })
  console.log('Content Generated, now saving to Zip')
  saveAs(content, 'layouts.zip')

  console.log('=====Zip saved======')
}

function concatenateParents(area, list = [area]) {
  const { parent } = area
  return parent ? concatenateParents(parent, [parent, ...list]) : list
}

let areaPath = computed(() => concatenateParents(area))

watch($$(currentArea), () => {
  if (accordion.active !== 'tree') {
    if (currentArea.display === 'grid') {
      accordion.active = 'explicit-grid'
    } else {
      const { parent } = currentArea
      if (parent && parent.display === 'grid') {
        accordion.active = 'self-grid'
      } else if (parent && parent.display === 'flex') {
        accordion.active = 'self-flex'
      } else {
        accordion.active = 'box'
      }
    }
  }
})

function onUpdateType(type: keyof HTMLElementTagNameMap) {
  area.type = type
  if (type === 'p') {
    if (!area.text) {
      area.text = 'Paragraph'
      area.display = 'block'
    }
  }
  if (type === 'button') {
    if (!area.text) {
      area.text = 'Action'
      area.display = 'block'
    }
  }
}
</script>

<style scoped lang="postcss">
.props-accordion-item h1 {
  width: 100%;
  border-top: 1px solid rgba(var(--color-gray-dark-rgb), 0.5);

  color: var(--color-white);
  font-size: 15px;
  margin: 0;
  padding: 10px 15px;
  cursor: pointer;
  span {
    font-weight: normal;
    font-size: 13px;
    text-shadow: none;
    color: var(--color-golden);
    font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  }
}

.area-actions-sidebar h1 {
  color: var(--color-golden);
  font-size: 13px;
  text-shadow: none;
  font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  direction: ltr;
  text-align: left;
  white-space: pre;
  word-spacing: normal;
  word-break: normal;
  line-height: 1.5;
  font-weight: normal;
}

.area-type {
  font-size: 14px;
  font-weight: 700;
  color: black;
  margin-bottom: 10px;
  text-align: center;
  padding: 5px;
  border-radius: 4px;
  background: var(--color-gray-light);
  border-bottom: 1px solid black;
}

.area-props {
  overflow: auto;
  overflow: overlay;
}

* {
  scrollbar-width: thin;
  scrollbar-color: var(--color-purple-middle) var(--color-purple-dark);
}
::-webkit-scrollbar {
  width: 5px;
}
::-webkit-scrollbar:horizontal {
  height: 5px;
}
::-webkit-scrollbar-track {
  background: var(--color-gray-darkest);
  border-radius: 10px;
}
::-webkit-scrollbar-thumb {
  background: var(--color-gray-middle);
  border-radius: 10px;
}
::-webkit-scrollbar-thumb:hover {
  background: var(--color-purple-dark);
}

.items {
  font-size: 11px;
  margin-bottom: 10px;
}
</style>
