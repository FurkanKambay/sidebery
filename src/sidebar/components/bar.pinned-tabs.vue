<template lang="pug">
.PinnedTabsBar(
  tabindex="-1"
  :data-empty="pinnedTabs.length === 0"
  :data-dnd-end="dropToEnd"
  data-dnd-type="pinned-bar"
  :data-dnd-id="panel?.id ?? NOID"
  @wheel="onWheel"
  @drop="onDrop")
  .tab-wrapper(v-for="id in pinnedTabs" :key="id" :data-targeted="dstMatches && dropId === id")
    Tab(:tabId="id")
</template>

<script lang="ts" setup>
import { computed } from 'vue'
import type * as T from 'src/types'
import * as E from 'src/enums'
import * as Settings from 'src/services/settings'
import * as Tabs from 'src/services/tabs.fg'
import * as Mouse from 'src/services/mouse.fg'
import * as DnD from 'src/services/drag-and-drop.fg'
import Tab from './tab.vue'
import { NOID } from 'src/defaults'

const props = defineProps<{ panel?: T.TabsPanel }>()
const pinnedTabs = computed(() => {
  if (props.panel) return props.panel.reactive.anchoredTabIds
  else return Tabs.reactive.pinnedIds
})
const dropId = computed(() => {
  const tab = Tabs.list[DnD.reactive.dstIndex]
  if (!tab || !tab.rank || (props.panel && tab.panelId !== props.panel.id)) return NOID
  else return tab.id
})
const dropToEnd = computed(() => dstMatches && dropId.value === NOID)
const dstMatches = computed(() =>
  (props.panel && DnD.reactive.dstRank === E.TabRank.Anchored) ||
  (!props.panel && DnD.reactive.dstRank === E.TabRank.Pinned))

const onWheel = Mouse.getWheelDebouncer(E.WheelDirection.Vertical, (e: WheelEvent) => {
  if (!props.panel && Settings.state.scrollThroughTabs !== 'none') {
    const globaly = (Settings.state.scrollThroughTabs === 'global') !== e.shiftKey
    const cyclic = Settings.state.scrollThroughTabsCyclic !== e.ctrlKey

    const globPin = !props.panel && Settings.state.scrollThroughTabsGlobPinIsolate ? true : undefined
    if (e.deltaY > 0) Tabs.switchTab(globaly, cyclic, 1, globPin)
    else if (e.deltaY < 0) Tabs.switchTab(globaly, cyclic, -1, globPin)
  }
})

function onDrop(): void {
  DnD.reactive.dstType = E.DropType.Tabs
  DnD.reactive.dstRank = props.panel ? E.TabRank.Anchored : E.TabRank.Pinned
}
</script>
