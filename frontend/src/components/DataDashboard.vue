<template>
  <div class="data-dashboard">
    <div class="gauges-section">
      <div class="section-header">
        <div class="section-title-wrap">
          <h3 class="section-title">实时仪表</h3>
          <span class="section-count">{{ visibleGauges.length }}/{{ allGauges.length }}</span>
        </div>
        <el-popover
          placement="bottom-end"
          :width="280"
          trigger="click"
          popper-class="dashboard-settings-popover"
          @show="onGaugeSettingsShow"
        >
          <template #reference>
            <el-button type="primary" plain size="small" class="settings-btn-enhanced">
              <el-icon><Setting /></el-icon>
              <span>自定义指标</span>
            </el-button>
          </template>
          <div class="settings-panel">
            <div class="settings-panel-header">
              <span class="settings-panel-title">仪表指标设置</span>
              <span class="settings-panel-count">已选 {{ visibleGauges.length }} 项</span>
            </div>
            <div class="settings-list-draggable">
              <div
                v-for="(g, index) in gaugeOrderList"
                :key="g.id"
                class="settings-drag-item"
                :class="{
                  'is-dragging': settingsDragGaugeId === g.id,
                  'drag-over': settingsDragOverGaugeId === g.id && settingsDragGaugeId !== g.id
                }"
                draggable="true"
                @dragstart="onSettingsGaugeDragStart(g.id, $event)"
                @dragenter="onSettingsGaugeDragEnter(g.id)"
                @dragover.prevent
                @drop="onSettingsGaugeDrop(g.id)"
                @dragend="onSettingsGaugeDragEnd"
              >
                <el-icon class="settings-drag-handle"><Rank /></el-icon>
                <el-checkbox
                  :model-value="isGaugeVisible(g.id)"
                  @change="(val) => setGaugeVisible(g.id, Boolean(val))"
                >
                  {{ g.label }}
                </el-checkbox>
              </div>
            </div>
            <div class="settings-tips">
              <el-icon><InfoFilled /></el-icon>
              <span>拖拽左侧图标可调整顺序，勾选控制是否显示</span>
            </div>
            <div class="settings-panel-footer">
              <el-button text size="small" @click="resetGauges">
                <el-icon><Refresh /></el-icon>
                <span>恢复默认</span>
              </el-button>
            </div>
          </div>
        </el-popover>
      </div>
      <div class="gauges-grid">
        <div
          v-for="widget in visibleGauges"
          :key="widget.id"
          class="gauge-card"
          :class="{
            'is-dragging': dragGaugeId === widget.id,
            'drag-over': dragOverGaugeId === widget.id
          }"
          draggable="true"
          @dragstart="onGaugeDragStart(widget.id)"
          @dragenter="onGaugeDragEnter(widget.id)"
          @dragover.prevent
          @drop="onGaugeDrop(widget.id)"
          @dragend="onGaugeDragEnd"
        >
          <div class="gauge-card-header">
            <span class="gauge-label">{{ widget.label }}</span>
            <el-icon class="drag-handle" title="拖拽排序"><Rank /></el-icon>
          </div>

          <template v-if="widget.kind === 'gauge-numeric'">
            <div class="gauge-value" :class="getGaugeClass(widget, getWidgetValue(widget))">
              {{ formatNumber(widget, getWidgetValue(widget) as number) }}
              <span class="gauge-unit">{{ widget.unit }}</span>
            </div>
            <el-progress
              :percentage="getProgress(widget, getWidgetValue(widget))"
              :color="getGaugeColor(widget, getWidgetValue(widget))"
              :stroke-width="8"
            />
          </template>

          <template v-else>
            <div class="gauge-value" :class="getWidgetValue(widget) ? 'text-green-400' : 'text-red-400'">
              {{ getWidgetValue(widget) ? '运行中' : '已停止' }}
            </div>
            <div class="pump-indicator" :class="getWidgetValue(widget) ? 'pump-on' : 'pump-off'">
              <el-icon :size="32">
                <CircleCheckFilled v-if="getWidgetValue(widget)" />
                <CircleCloseFilled v-else />
              </el-icon>
            </div>
          </template>

          <div class="gauge-quality">
            <span class="quality-dot" :class="getQualityClass(widget.nodeId)"></span>
            {{ getNodeQuality(widget.nodeId) }}
          </div>
        </div>

        <div v-if="visibleGauges.length === 0" class="empty-placeholder">
          <el-empty description="未选择任何仪表指标，点击右上角「自定义指标」添加" :image-size="64" />
        </div>
      </div>
    </div>

    <div class="charts-section">
      <div class="section-header">
        <div class="section-title-wrap">
          <h3 class="section-title">数据趋势</h3>
          <span class="section-count">{{ visibleCharts.length }}/{{ allCharts.length }}</span>
        </div>
        <el-popover
          placement="bottom-end"
          :width="280"
          trigger="click"
          popper-class="dashboard-settings-popover"
          @show="onChartSettingsShow"
        >
          <template #reference>
            <el-button type="primary" plain size="small" class="settings-btn-enhanced">
              <el-icon><Setting /></el-icon>
              <span>自定义指标</span>
            </el-button>
          </template>
          <div class="settings-panel">
            <div class="settings-panel-header">
              <span class="settings-panel-title">趋势指标设置</span>
              <span class="settings-panel-count">已选 {{ visibleCharts.length }} 项</span>
            </div>
            <div class="settings-list-draggable">
              <div
                v-for="(c, index) in chartOrderList"
                :key="c.id"
                class="settings-drag-item"
                :class="{
                  'is-dragging': settingsDragChartId === c.id,
                  'drag-over': settingsDragOverChartId === c.id && settingsDragChartId !== c.id
                }"
                draggable="true"
                @dragstart="onSettingsChartDragStart(c.id, $event)"
                @dragenter="onSettingsChartDragEnter(c.id)"
                @dragover.prevent
                @drop="onSettingsChartDrop(c.id)"
                @dragend="onSettingsChartDragEnd"
              >
                <el-icon class="settings-drag-handle"><Rank /></el-icon>
                <el-checkbox
                  :model-value="isChartVisible(c.id)"
                  @change="(val) => setChartVisible(c.id, Boolean(val))"
                >
                  {{ c.label }}
                </el-checkbox>
              </div>
            </div>
            <div class="settings-tips">
              <el-icon><InfoFilled /></el-icon>
              <span>拖拽左侧图标可调整顺序，勾选控制是否显示</span>
            </div>
            <div class="settings-panel-footer">
              <el-button text size="small" @click="resetCharts">
                <el-icon><Refresh /></el-icon>
                <span>恢复默认</span>
              </el-button>
            </div>
          </div>
        </el-popover>
      </div>
      <div class="charts-grid">
        <div
          v-for="widget in visibleCharts"
          :key="widget.id"
          class="chart-card"
          :class="{
            'is-dragging': dragChartId === widget.id,
            'drag-over': dragOverChartId === widget.id
          }"
          draggable="true"
          @dragstart="onChartDragStart(widget.id)"
          @dragenter="onChartDragEnter(widget.id)"
          @dragover.prevent
          @drop="onChartDrop(widget.id)"
          @dragend="onChartDragEnd"
        >
          <div class="chart-card-header">
            <span class="chart-card-title">{{ widget.label }}</span>
            <el-icon class="drag-handle" title="拖拽排序"><Rank /></el-icon>
          </div>
          <v-chart :option="buildChartOption(widget.nodeId, widget.chartColor!, widget.unit!)" autoresize class="chart" />
        </div>
        <div v-if="visibleCharts.length === 0" class="empty-placeholder">
          <el-empty description="未选择任何趋势指标，点击右上角「自定义指标」添加" :image-size="64" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import VChart from 'vue-echarts'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart } from 'echarts/charts'
import { GridComponent, TooltipComponent, TitleComponent } from 'echarts/components'
import { CircleCheckFilled, CircleCloseFilled, Rank, Setting, InfoFilled, Refresh } from '@element-plus/icons-vue'
import { useOpcuaStore } from '../store/opcua'

use([CanvasRenderer, LineChart, GridComponent, TooltipComponent, TitleComponent])

const store = useOpcuaStore()

type WidgetKind = 'gauge-numeric' | 'gauge-boolean' | 'chart'
type ColorKey = 'temp' | 'pressure' | 'speed' | 'flow' | 'valve'

interface WidgetDef {
  id: string
  label: string
  kind: WidgetKind
  nodeId: string
  unit?: string
  fallback?: number | boolean
  progressMax?: number
  decimals?: number
  colorKey?: ColorKey
  chartColor?: string
}

const WIDGET_REGISTRY: WidgetDef[] = [
  { id: 'gauge_temp', label: '温度', kind: 'gauge-numeric', nodeId: 'temp_sensor', unit: '°C', fallback: 25, progressMax: 50, decimals: 1, colorKey: 'temp' },
  { id: 'gauge_pressure', label: '压力', kind: 'gauge-numeric', nodeId: 'pressure_transmitter', unit: 'MPa', fallback: 3.5, progressMax: 6, decimals: 2, colorKey: 'pressure' },
  { id: 'gauge_flow', label: '流量', kind: 'gauge-numeric', nodeId: 'flow_meter', unit: 'L/min', fallback: 150, progressMax: 300, decimals: 1, colorKey: 'flow' },
  { id: 'gauge_valve', label: '阀门开度', kind: 'gauge-numeric', nodeId: 'valve_position', unit: '%', fallback: 75, progressMax: 100, decimals: 0, colorKey: 'valve' },
  { id: 'gauge_motor', label: '电机转速', kind: 'gauge-numeric', nodeId: 'motor_speed', unit: 'RPM', fallback: 1480, progressMax: 2000, decimals: 0, colorKey: 'speed' },
  { id: 'gauge_pump', label: '泵运行状态', kind: 'gauge-boolean', nodeId: 'pump_status', fallback: false },
  { id: 'chart_temp', label: '温度趋势', kind: 'chart', nodeId: 'temp_sensor', unit: '°C', chartColor: '#67c23a' },
  { id: 'chart_pressure', label: '压力趋势', kind: 'chart', nodeId: 'pressure_transmitter', unit: 'MPa', chartColor: '#06b6d4' },
  { id: 'chart_flow', label: '流量趋势', kind: 'chart', nodeId: 'flow_meter', unit: 'L/min', chartColor: '#60a5fa' }
]

const allGauges = WIDGET_REGISTRY.filter(w => w.kind !== 'chart')
const allCharts = WIDGET_REGISTRY.filter(w => w.kind === 'chart')
const DEFAULT_GAUGE_ORDER = allGauges.map(w => w.id)
const DEFAULT_CHART_ORDER = allCharts.map(w => w.id)

interface SectionLayout {
  order: string[]
  hidden: string[]
}

interface DashboardLayout {
  gauges: SectionLayout
  charts: SectionLayout
}

const STORAGE_KEY = 'opcua-dashboard-layout'

function mergeOrder(stored: string[] | undefined, defaults: string[]): string[] {
  if (!Array.isArray(stored)) return [...defaults]
  const valid = stored.filter(id => defaults.includes(id))
  defaults.forEach(id => {
    if (!valid.includes(id)) valid.push(id)
  })
  return valid
}

function mergeSection(stored: Partial<SectionLayout> | undefined, defaults: string[]): SectionLayout {
  const order = mergeOrder(stored?.order, defaults)
  const hidden = Array.isArray(stored?.hidden)
    ? stored!.hidden.filter(id => order.includes(id))
    : []
  return { order, hidden }
}

function loadLayout(): DashboardLayout {
  const fallback: DashboardLayout = {
    gauges: { order: [...DEFAULT_GAUGE_ORDER], hidden: [] },
    charts: { order: [...DEFAULT_CHART_ORDER], hidden: [] }
  }
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (!raw) return fallback
    const parsed = JSON.parse(raw) as Partial<DashboardLayout>
    return {
      gauges: mergeSection(parsed.gauges, DEFAULT_GAUGE_ORDER),
      charts: mergeSection(parsed.charts, DEFAULT_CHART_ORDER)
    }
  } catch {
    return fallback
  }
}

const initialLayout = loadLayout()
const gaugeLayout = ref<SectionLayout>(initialLayout.gauges)
const chartLayout = ref<SectionLayout>(initialLayout.charts)

const visibleGauges = computed(() =>
  gaugeLayout.value.order
    .filter(id => !gaugeLayout.value.hidden.includes(id))
    .map(id => allGauges.find(w => w.id === id))
    .filter((w): w is WidgetDef => Boolean(w))
)

const visibleCharts = computed(() =>
  chartLayout.value.order
    .filter(id => !chartLayout.value.hidden.includes(id))
    .map(id => allCharts.find(w => w.id === id))
    .filter((w): w is WidgetDef => Boolean(w))
)

const gaugeOrderList = computed(() =>
  gaugeLayout.value.order
    .map(id => allGauges.find(w => w.id === id))
    .filter((w): w is WidgetDef => Boolean(w))
)

const chartOrderList = computed(() =>
  chartLayout.value.order
    .map(id => allCharts.find(w => w.id === id))
    .filter((w): w is WidgetDef => Boolean(w))
)

function isGaugeVisible(id: string): boolean {
  return !gaugeLayout.value.hidden.includes(id)
}

function setGaugeVisible(id: string, visible: boolean): void {
  const hidden = [...gaugeLayout.value.hidden]
  if (visible) {
    gaugeLayout.value.hidden = hidden.filter(x => x !== id)
  } else if (!hidden.includes(id)) {
    hidden.push(id)
    gaugeLayout.value.hidden = hidden
  }
}

function resetGauges(): void {
  gaugeLayout.value = { order: [...DEFAULT_GAUGE_ORDER], hidden: [] }
}

function isChartVisible(id: string): boolean {
  return !chartLayout.value.hidden.includes(id)
}

function setChartVisible(id: string, visible: boolean): void {
  const hidden = [...chartLayout.value.hidden]
  if (visible) {
    chartLayout.value.hidden = hidden.filter(x => x !== id)
  } else if (!hidden.includes(id)) {
    hidden.push(id)
    chartLayout.value.hidden = hidden
  }
}

function resetCharts(): void {
  chartLayout.value = { order: [...DEFAULT_CHART_ORDER], hidden: [] }
}

const dragGaugeId = ref<string | null>(null)
const dragOverGaugeId = ref<string | null>(null)
const dragChartId = ref<string | null>(null)
const dragOverChartId = ref<string | null>(null)

const settingsDragGaugeId = ref<string | null>(null)
const settingsDragOverGaugeId = ref<string | null>(null)
const settingsDragChartId = ref<string | null>(null)
const settingsDragOverChartId = ref<string | null>(null)

function reorderSection(section: SectionLayout, fromId: string, targetId: string): SectionLayout {
  if (fromId === targetId) return section
  const order = [...section.order]
  const from = order.indexOf(fromId)
  const to = order.indexOf(targetId)
  if (from < 0 || to < 0) return section
  order.splice(from, 1)
  order.splice(to, 0, fromId)
  return { order, hidden: [...section.hidden] }
}

function onGaugeDragStart(id: string): void {
  dragGaugeId.value = id
}

function onGaugeDragEnter(id: string): void {
  dragOverGaugeId.value = id
}

function onGaugeDrop(targetId: string): void {
  const fromId = dragGaugeId.value
  if (fromId) {
    gaugeLayout.value = reorderSection(gaugeLayout.value, fromId, targetId)
  }
  dragGaugeId.value = null
  dragOverGaugeId.value = null
}

function onGaugeDragEnd(): void {
  dragGaugeId.value = null
  dragOverGaugeId.value = null
}

function onChartDragStart(id: string): void {
  dragChartId.value = id
}

function onChartDragEnter(id: string): void {
  dragOverChartId.value = id
}

function onChartDrop(targetId: string): void {
  const fromId = dragChartId.value
  if (fromId) {
    chartLayout.value = reorderSection(chartLayout.value, fromId, targetId)
  }
  dragChartId.value = null
  dragOverChartId.value = null
}

function onChartDragEnd(): void {
  dragChartId.value = null
  dragOverChartId.value = null
}

function onGaugeSettingsShow(): void {
  // Reset drag states when popover opens
  settingsDragGaugeId.value = null
  settingsDragOverGaugeId.value = null
}

function onChartSettingsShow(): void {
  settingsDragChartId.value = null
  settingsDragOverChartId.value = null
}

function onSettingsGaugeDragStart(id: string, event: DragEvent): void {
  settingsDragGaugeId.value = id
  if (event.dataTransfer) {
    event.dataTransfer.effectAllowed = 'move'
  }
}

function onSettingsGaugeDragEnter(id: string): void {
  if (settingsDragGaugeId.value && settingsDragGaugeId.value !== id) {
    settingsDragOverGaugeId.value = id
  }
}

function onSettingsGaugeDrop(targetId: string): void {
  const fromId = settingsDragGaugeId.value
  if (fromId && fromId !== targetId) {
    gaugeLayout.value = reorderSection(gaugeLayout.value, fromId, targetId)
  }
  settingsDragGaugeId.value = null
  settingsDragOverGaugeId.value = null
}

function onSettingsGaugeDragEnd(): void {
  settingsDragGaugeId.value = null
  settingsDragOverGaugeId.value = null
}

function onSettingsChartDragStart(id: string, event: DragEvent): void {
  settingsDragChartId.value = id
  if (event.dataTransfer) {
    event.dataTransfer.effectAllowed = 'move'
  }
}

function onSettingsChartDragEnter(id: string): void {
  if (settingsDragChartId.value && settingsDragChartId.value !== id) {
    settingsDragOverChartId.value = id
  }
}

function onSettingsChartDrop(targetId: string): void {
  const fromId = settingsDragChartId.value
  if (fromId && fromId !== targetId) {
    chartLayout.value = reorderSection(chartLayout.value, fromId, targetId)
  }
  settingsDragChartId.value = null
  settingsDragOverChartId.value = null
}

function onSettingsChartDragEnd(): void {
  settingsDragChartId.value = null
  settingsDragOverChartId.value = null
}

watch(
  [gaugeLayout, chartLayout],
  ([gauges, charts]) => {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify({ gauges, charts }))
    } catch {
      // ignore storage errors
    }
  },
  { deep: true }
)

function findNodeById(id: string) {
  function search(nodes: any[]): any {
    for (const node of nodes) {
      if (node.id === id) return node
      if (node.children) {
        const found = search(node.children)
        if (found) return found
      }
    }
    return null
  }
  return search(store.nodeTree)
}

function getNodeQuality(nodeId: string): string {
  const data = store.realTimeData.get(nodeId)
  if (data) return data.quality
  const node = findNodeById(nodeId)
  return node?.quality ?? 'Unknown'
}

function getQualityClass(nodeId: string): string {
  const quality = getNodeQuality(nodeId)
  return quality === 'Good' ? 'quality-good' : quality === 'Bad' ? 'quality-bad' : 'quality-uncertain'
}

function getWidgetValue(widget: WidgetDef): number | boolean {
  const data = store.realTimeData.get(widget.nodeId)
  if (data) return data.value as number | boolean
  const node = findNodeById(widget.nodeId)
  if (node && node.value !== undefined) return node.value as number | boolean
  return widget.fallback as number | boolean
}

function formatNumber(widget: WidgetDef, value: number): string {
  return value.toFixed(widget.decimals ?? 1)
}

function getTempClass(val: number) {
  if (val > 30) return 'text-red-400'
  if (val > 28) return 'text-yellow-400'
  return 'text-green-400'
}

function getTempColor(val: number) {
  if (val > 30) return '#f56c6c'
  if (val > 28) return '#e6a23c'
  return '#67c23a'
}

function getPressureClass(val: number) {
  if (val > 4.5) return 'text-red-400'
  if (val > 4.0) return 'text-yellow-400'
  return 'text-cyan-400'
}

function getPressureColor(val: number) {
  if (val > 4.5) return '#f56c6c'
  if (val > 4.0) return '#e6a23c'
  return '#06b6d4'
}

function getSpeedClass(val: number) {
  if (val > 1600) return 'text-red-400'
  if (val > 1550) return 'text-yellow-400'
  return 'text-emerald-400'
}

function getSpeedColor(val: number) {
  if (val > 1600) return '#f56c6c'
  if (val > 1550) return '#e6a23c'
  return '#34d399'
}

const colorProfiles: Record<ColorKey, { textClass: (v: number) => string; color: (v: number) => string }> = {
  temp: { textClass: getTempClass, color: getTempColor },
  pressure: { textClass: getPressureClass, color: getPressureColor },
  speed: { textClass: getSpeedClass, color: getSpeedColor },
  flow: { textClass: () => 'text-blue-400', color: () => '#60a5fa' },
  valve: { textClass: () => 'text-purple-400', color: () => '#a78bfa' }
}

function getGaugeClass(widget: WidgetDef, value: number | boolean): string {
  if (typeof value === 'boolean' || !widget.colorKey) return ''
  return colorProfiles[widget.colorKey]?.textClass(value) ?? ''
}

function getGaugeColor(widget: WidgetDef, value: number | boolean): string {
  if (typeof value === 'boolean' || !widget.colorKey) return '#409eff'
  return colorProfiles[widget.colorKey]?.color(value) ?? '#409eff'
}

function getProgress(widget: WidgetDef, value: number | boolean): number {
  if (typeof value === 'boolean' || !widget.progressMax) return 0
  return Math.min((value / widget.progressMax) * 100, 100)
}

function buildChartOption(nodeId: string, color: string, unit: string) {
  const history = store.dataHistory.get(nodeId) || []
  const data = history.map(h => [h.timestamp, h.value])

  return {
    tooltip: { trigger: 'axis' as const },
    grid: { left: 60, right: 20, top: 16, bottom: 30 },
    xAxis: {
      type: 'time' as const,
      axisLabel: { color: '#999', formatter: '{HH}:{mm}:{ss}' },
      axisLine: { lineStyle: { color: '#444' } }
    },
    yAxis: {
      type: 'value' as const,
      axisLabel: { color: '#999', formatter: `{value} ${unit}` },
      splitLine: { lineStyle: { color: '#333' } }
    },
    series: [{
      type: 'line',
      data,
      smooth: true,
      lineStyle: { color, width: 2 },
      areaStyle: { color: { type: 'linear', x: 0, y: 0, x2: 0, y2: 1, colorStops: [{ offset: 0, color: color + '40' }, { offset: 1, color: color + '05' }] } },
      symbol: 'none'
    }]
  }
}
</script>

<style scoped>
.data-dashboard {
  height: 100%;
  overflow-y: auto;
  padding: 12px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.section-title-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #22d3ee;
  margin: 0;
  padding-left: 8px;
  border-left: 3px solid #06b6d4;
}

.section-count {
  font-size: 12px;
  color: #64748b;
  background: rgba(71, 85, 105, 0.3);
  padding: 2px 8px;
  border-radius: 10px;
}

.settings-btn-enhanced {
  border-color: rgba(34, 211, 238, 0.5);
  color: #22d3ee;
}

.settings-btn-enhanced:hover {
  background: rgba(34, 211, 238, 0.1);
  border-color: #22d3ee;
  color: #22d3ee;
}

.gauges-section {
  margin-bottom: 20px;
}

.gauges-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.gauge-card {
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 8px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  cursor: grab;
  transition: opacity 0.15s ease, border-color 0.15s ease, box-shadow 0.15s ease;
}

.gauge-card:active {
  cursor: grabbing;
}

.gauge-card.is-dragging {
  opacity: 0.4;
}

.gauge-card.drag-over {
  border-color: #22d3ee;
  box-shadow: 0 0 0 1px #22d3ee inset;
}

.gauge-card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.gauge-label {
  font-size: 13px;
  color: #94a3b8;
  font-weight: 500;
}

.drag-handle {
  color: #475569;
  cursor: grab;
}

.drag-handle:active {
  cursor: grabbing;
}

.gauge-value {
  font-size: 28px;
  font-weight: bold;
  font-family: 'Courier New', monospace;
}

.gauge-unit {
  font-size: 14px;
  color: #64748b;
  font-weight: normal;
}

.gauge-quality {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #64748b;
}

.quality-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
}

.quality-good { background: #67c23a; }
.quality-bad { background: #f56c6c; }
.quality-uncertain { background: #e6a23c; }

.pump-indicator {
  display: flex;
  justify-content: center;
  padding: 8px 0;
}

.pump-on { color: #67c23a; }
.pump-off { color: #f56c6c; }

.charts-section {
  margin-top: 16px;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.chart-card {
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 8px;
  padding: 12px;
  cursor: grab;
  transition: opacity 0.15s ease, border-color 0.15s ease, box-shadow 0.15s ease;
}

.chart-card:active {
  cursor: grabbing;
}

.chart-card.is-dragging {
  opacity: 0.4;
}

.chart-card.drag-over {
  border-color: #22d3ee;
  box-shadow: 0 0 0 1px #22d3ee inset;
}

.chart-card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.chart-card-title {
  font-size: 13px;
  color: #e0e0e0;
  font-weight: 500;
}

.chart {
  height: 220px;
  width: 100%;
}

.empty-placeholder {
  grid-column: 1 / -1;
  display: flex;
  justify-content: center;
  padding: 24px 0;
}

@media (max-width: 1200px) {
  .gauges-grid { grid-template-columns: repeat(2, 1fr); }
  .charts-grid { grid-template-columns: 1fr; }
}
</style>

<style>
.dashboard-settings-popover.el-popover.el-popper {
  background: #1e293b;
  border: 1px solid rgba(71, 85, 105, 0.5);
  color: #e0e0e0;
}

.dashboard-settings-popover .el-checkbox__label {
  color: #cbd5e1;
}

.dashboard-settings-popover .settings-panel {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.dashboard-settings-popover .settings-panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.dashboard-settings-popover .settings-panel-title {
  font-size: 13px;
  color: #cbd5e1;
  font-weight: 600;
}

.dashboard-settings-popover .settings-panel-count {
  font-size: 11px;
  color: #22d3ee;
  background: rgba(34, 211, 238, 0.1);
  padding: 2px 6px;
  border-radius: 4px;
}

.dashboard-settings-popover .settings-list-draggable {
  display: flex;
  flex-direction: column;
  gap: 4px;
  max-height: 320px;
  overflow-y: auto;
}

.dashboard-settings-popover .settings-drag-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 8px;
  border-radius: 6px;
  cursor: grab;
  transition: background-color 0.15s ease, border-color 0.15s ease;
  border: 1px solid transparent;
}

.dashboard-settings-popover .settings-drag-item:hover {
  background: rgba(71, 85, 105, 0.3);
}

.dashboard-settings-popover .settings-drag-item:active {
  cursor: grabbing;
}

.dashboard-settings-popover .settings-drag-item.is-dragging {
  opacity: 0.4;
}

.dashboard-settings-popover .settings-drag-item.drag-over {
  border-color: #22d3ee;
  background: rgba(34, 211, 238, 0.1);
}

.dashboard-settings-popover .settings-drag-handle {
  color: #475569;
  font-size: 14px;
  flex-shrink: 0;
}

.dashboard-settings-popover .settings-tips {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
  color: #64748b;
  padding-top: 4px;
  border-top: 1px solid rgba(71, 85, 105, 0.3);
}

.dashboard-settings-popover .settings-tips .el-icon {
  color: #3b82f6;
}

.dashboard-settings-popover .settings-panel-footer {
  display: flex;
  justify-content: flex-end;
  padding-top: 4px;
}

.dashboard-settings-popover .settings-panel-footer .el-button {
  color: #94a3b8;
}

.dashboard-settings-popover .settings-panel-footer .el-button:hover {
  color: #f56c6c;
}
</style>
