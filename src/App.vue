<script setup lang="ts">
import { ref, reactive, h } from 'vue'
import { Sparkles, Search, Folder, Monitor, BookOpen, Code, Sun, Globe, LogIn, Plus, MousePointer2, Zap, ArrowRight, RefreshCw, Image as ImageIcon } from 'lucide-vue-next'
import { VueFlow, useVueFlow, Position, MarkerType } from '@vue-flow/core'
import { Background, BackgroundVariant } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'

// 导入 VueFlow 样式
import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'

// API 配置
const API_KEY = import.meta.env.VITE_ZHIPU_AI_API_KEY

// VueFlow 实例
const { addNodes, addEdges, onConnect, setNodes, setEdges, nodes: flowNodes, edges: flowEdges, updateNode } = useVueFlow()

// 状态管理
const ideaInput = ref('')
const isLoading = ref(false)

/**
 * 调用智谱AI生成图片
 */
const generateNodeImage = async (nodeId: string, prompt: string) => {
    const node = flowNodes.value.find(n => n.id === nodeId)
    if (!node || node.data.isImageLoading) return

    updateNode(nodeId, { data: { ...node.data, isImageLoading: true } })

    try {
        const response = await fetch('https://open.bigmodel.cn/api/paas/v4/images/generations', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${API_KEY}`
            },
            body: JSON.stringify({
                model: 'cogview-3-flash',
                prompt: `一张精美的、极简插画风格的图片，表现主题：${prompt}。要求：构图简洁，色彩明快，适合作为思维导图的视觉辅助。`
            })
        })

        if (!response.ok) throw new Error('Image request failed')
        const data = await response.json()
        const imageUrl = data.data[0].url

        updateNode(nodeId, { data: { ...node.data, imageUrl, isImageLoading: false } })
    } catch (error) {
        console.error('Image Generation Error:', error)
        updateNode(nodeId, { data: { ...node.data, isImageLoading: false } })
    }
}

/**
 * 调用智谱AI生成思维发散节点
 */
const expandIdea = async (param?: any) => {
    // 判断是点击了节点上的按钮还是主输入框
    const parentNode = param && param.id ? param : undefined
    const text = parentNode ? parentNode.data.label : ideaInput.value

    if (!text || isLoading.value) return
    isLoading.value = true

    // 如果是第一次生成，清空画布
    if (!parentNode) {
        setNodes([])
        setEdges([])
    }

    const systemPrompt = `你是一个思维发散专家。给定一个想法，请将其发散出 3-5 个更深层或相关维度的子想法。
    每个子想法包含：简短的名称 (text) 和 极简的描述 (description)。
    返回格式必须为 JSON: { "nodes": [ { "text": "...", "description": "..." } ] }`

    try {
        const response = await fetch('https://open.bigmodel.cn/api/paas/v4/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${API_KEY}`
            },
            body: JSON.stringify({
                model: 'glm-4-flash',
                messages: [
                    { role: 'system', content: systemPrompt },
                    { role: 'user', content: text }
                ],
                response_format: { type: 'json_object' },
                temperature: 0.8
            })
        })

        if (!response.ok) throw new Error('AI request failed')
        const data = await response.json()
        const result = JSON.parse(data.choices[0].message.content)

        // 起始坐标
        const startX = parentNode ? parentNode.position.x + 450 : 50
        const startY = parentNode ? parentNode.position.y : 300

        // 创建根节点 (如果是第一次)
        if (!parentNode) {
            const rootId = 'root-' + Date.now()
            addNodes({
                id: rootId,
                type: 'window',
                position: { x: startX, y: startY },
                data: { label: text, description: '核心想法', type: 'root' }
            })

            // 为后续子节点计算位置
            processSubNodes(result.nodes, rootId, startX, startY)
        } else {
            processSubNodes(result.nodes, parentNode.id, startX, startY)
        }
    } catch (error) {
        console.error('Expansion Error:', error)
    } finally {
        isLoading.value = false
    }
}

const processSubNodes = (subNodes: any[], parentId: string, baseX: number, baseY: number) => {
    subNodes.forEach((item: any, index: number) => {
        const childId = `node-${Date.now()}-${index}`
        const offsetX = 400
        const offsetY = (index - (subNodes.length - 1) / 2) * 250

        addNodes({
            id: childId,
            type: 'window',
            position: { x: baseX + offsetX, y: baseY + offsetY },
            data: { label: item.text, description: item.description, type: 'child' }
        })

        addEdges({
            id: `e-${parentId}-${childId}`,
            source: parentId,
            target: childId,
            animated: true,
            style: { stroke: '#fed7aa', strokeWidth: 2 },
            markerEnd: MarkerType.ArrowClosed
        })
    })
}

const startNewSession = () => {
    ideaInput.value = ''
    setNodes([])
    setEdges([])
}
</script>

<template>
    <div class="h-screen w-screen bg-white font-mono text-slate-800 relative overflow-hidden flex flex-col selection:bg-orange-100">
        <!-- 顶部导航栏 -->
        <nav class="flex-none bg-white/80 backdrop-blur-md border-b border-slate-200 px-6 py-2 flex items-center justify-between shadow-sm z-50">
            <div class="flex items-center gap-6">
                <div class="flex items-center gap-2">
                    <div class="w-2.5 h-2.5 bg-emerald-400 rounded-full animate-pulse"></div>
                    <span class="text-slate-400">ready</span>
                    <span class="text-slate-300 mx-1">~/</span>
                    <span class="font-bold text-slate-900 tracking-tight">ThinkFlow AI</span>
                    <span class="w-1.5 h-5 bg-orange-200 ml-1"></span>
                </div>
            </div>

            <div class="hidden lg:flex items-center gap-2">
                <button class="nav-btn group">
                    <span class="text-orange-500 font-bold">$</span>
                    <span class="text-slate-600">ai</span>
                    <span class="text-slate-300 ml-1">--search</span>
                </button>
                <button class="nav-btn">
                    <span class="text-emerald-500 font-bold">$</span>
                    <span class="text-slate-600">cd</span>
                    <span class="text-slate-300 ml-1">/categories</span>
                </button>
                <button class="nav-btn">
                    <span class="text-blue-500 font-bold">$</span>
                    <span class="text-slate-600">watch</span>
                    <span class="font-bold ml-1">stats</span>
                </button>
                <div class="h-4 w-[1px] bg-slate-200 mx-2"></div>
                <button class="p-2 hover:bg-slate-100 rounded-md transition-colors text-slate-400 font-bold text-xs flex items-center gap-1">
                    <Globe class="w-3.5 h-3.5" /> ZH
                </button>
                <button
                    class="flex items-center gap-2 px-4 py-1.5 bg-emerald-50 text-emerald-600 rounded-md text-xs font-bold border border-emerald-100 hover:bg-emerald-100 transition-all"
                >
                    <span class="text-emerald-500">$</span> 登录 <LogIn class="w-3.5 h-3.5 ml-1" />
                </button>
            </div>
        </nav>

        <!-- 主内容区：VueFlow 画布 -->
        <div class="flex-grow relative">
            <VueFlow :default-edge-options="{ type: 'smoothstep' }" :fit-view-on-init="true" class="bg-white">
                <Background :variant="BackgroundVariant.Lines" pattern-color="#f2f2f2" :gap="24" :size="0.5" />
                <Controls />

                <!-- 自定义节点插槽 -->
                <template #node-window="{ id, data }">
                    <div class="window-node group">
                        <!-- Window Header -->
                        <div class="window-header">
                            <div class="flex gap-1.5">
                                <div class="w-2 h-2 rounded-full bg-red-400/80"></div>
                                <div class="w-2 h-2 rounded-full bg-amber-400/80"></div>
                                <div class="w-2 h-2 rounded-full bg-emerald-400/80"></div>
                            </div>
                            <span class="window-title">
                                {{ data.type === 'root' ? 'main.ts' : 'module.tsx' }}
                            </span>
                        </div>

                        <!-- Window Content -->
                        <div class="window-content">
                            <!-- Image Display -->
                            <div
                                v-if="data.imageUrl || data.isImageLoading"
                                class="mb-4 rounded-lg overflow-hidden bg-slate-50 border border-slate-100 aspect-video flex items-center justify-center relative group/img"
                            >
                                <img v-if="data.imageUrl" :src="data.imageUrl" class="w-full h-full object-cover" />
                                <div v-if="data.isImageLoading" class="absolute inset-0 flex flex-col items-center justify-center bg-white/80 backdrop-blur-sm">
                                    <RefreshCw class="w-6 h-6 text-orange-500 animate-spin mb-2" />
                                    <span class="text-[8px] font-bold text-slate-400 uppercase">Generating...</span>
                                </div>
                                <div
                                    v-if="data.imageUrl"
                                    class="absolute inset-0 bg-black/40 opacity-0 group-hover/img:opacity-100 transition-opacity flex items-center justify-center"
                                >
                                    <button @click="generateNodeImage(id, data.label)" class="p-2 bg-white/20 hover:bg-white/40 rounded-full backdrop-blur-md transition-all">
                                        <RefreshCw class="w-4 h-4 text-white" />
                                    </button>
                                </div>
                            </div>

                            <div class="flex items-center gap-2 mb-2">
                                <span class="text-orange-500 font-bold">></span>
                                <h3 class="font-black text-slate-900 tracking-tight truncate">{{ data.label }}</h3>
                            </div>
                            <p class="text-[10px] text-slate-500 leading-relaxed font-medium line-clamp-3">
                                {{ data.description }}
                            </p>

                            <!-- Node Actions -->
                            <div class="pt-3 mt-3 border-t border-slate-50 flex items-center justify-between gap-2">
                                <div class="flex items-center gap-1.5 shrink-0">
                                    <div class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse"></div>
                                    <span class="text-[9px] font-bold text-slate-400 uppercase tracking-tighter">active</span>
                                </div>

                                <div class="flex items-center gap-2 overflow-hidden">
                                    <button
                                        v-if="!data.imageUrl && !data.isImageLoading"
                                        @click.stop="generateNodeImage(id, data.label)"
                                        class="action-btn text-blue-500 hover:bg-blue-50"
                                    >
                                        <ImageIcon class="w-2.5 h-2.5" />
                                        <span>IMG</span>
                                    </button>

                                    <button
                                        @click.stop="expandIdea({ id, data, position: flowNodes.find(n => n.id === id)?.position })"
                                        class="action-btn text-orange-500 hover:bg-orange-50"
                                    >
                                        <ArrowRight class="w-2.5 h-2.5" />
                                        <span>EXPAND</span>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </template>
            </VueFlow>

            <!-- 浮动 UI 层 -->
            <div class="absolute inset-0 pointer-events-none z-10 p-12"></div>

            <!-- 加载状态遮罩 -->
            <div v-if="isLoading" class="absolute inset-0 z-[100] flex items-center justify-center bg-white/5 backdrop-blur-[1px] pointer-events-none">
                <div class="bg-white/90 p-4 rounded-full shadow-2xl border border-orange-100 animate-bounce">
                    <Zap class="w-8 h-8 text-orange-500" />
                </div>
            </div>
        </div>

        <!-- 底部全局操作栏 -->
        <div class="fixed bottom-10 left-1/2 -translate-x-1/2 z-50 flex items-center gap-4">
            <!-- 核心输入框 -->
            <div
                class="bg-slate-900 rounded-2xl p-2 pl-6 flex items-center gap-4 shadow-2xl shadow-slate-300 border border-slate-800 w-[600px] group transition-all focus-within:ring-2 focus-within:ring-orange-500/20"
            >
                <span class="text-orange-500 font-black text-xl select-none">></span>
                <input
                    v-model="ideaInput"
                    @keyup.enter="expandIdea"
                    placeholder="Input your idea to expand..."
                    class="bg-transparent border-none outline-none text-white placeholder:text-slate-500 flex-grow font-bold tracking-tight"
                />
                <button
                    @click="expandIdea"
                    :disabled="isLoading"
                    class="p-3 bg-orange-500 hover:bg-orange-600 text-white rounded-xl transition-all active:scale-90 disabled:opacity-50"
                >
                    <Zap v-if="!isLoading" class="w-5 h-5" />
                    <RefreshCw v-else class="w-5 h-5 animate-spin" />
                </button>
            </div>

            <!-- 清空按钮 -->
            <button
                @click="startNewSession"
                class="bg-white text-slate-900 p-4 rounded-2xl shadow-2xl border border-slate-200 hover:bg-slate-50 transition-all active:scale-95 group"
                title="Clear Canvas"
            >
                <Plus class="w-6 h-6 group-hover:rotate-90 transition-transform text-slate-400" />
            </button>
        </div>
    </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700;800&display=swap');

body {
    margin: 0;
    overflow: hidden;
    -webkit-font-smoothing: antialiased;
}

.font-mono {
    font-family: 'JetBrains Mono', monospace;
}

.nav-btn {
    @apply flex items-center gap-1 px-3 py-1 bg-white border border-slate-200 rounded-md text-xs font-medium text-slate-600 hover:border-slate-300 hover:shadow-sm transition-all;
}

/* VueFlow Custom Node Styles */
.window-node {
    @apply w-[280px] bg-white rounded-xl border border-slate-200 shadow-xl overflow-hidden transition-all duration-300;
}

.window-node:hover {
    @apply shadow-2xl shadow-orange-100 -translate-y-1 border-orange-200;
}

.window-header {
    @apply bg-slate-50/80 px-3 py-1.5 border-b border-slate-100 flex items-center justify-between;
}

.window-title {
    @apply text-[9px] font-bold text-slate-300 tracking-widest uppercase;
}

.window-content {
    @apply p-4;
}

.action-btn {
    @apply flex items-center gap-1 px-2 py-1 rounded-md text-[9px] font-bold transition-all active:scale-95 uppercase tracking-tighter whitespace-nowrap border border-transparent;
}

.action-btn:hover {
    @apply border-current bg-opacity-10;
}

/* VueFlow Overrides */
.vue-flow__node-window {
    @apply p-0 border-none bg-transparent !important;
}

.vue-flow__edge-path {
    @apply stroke-orange-200 !important;
}

.vue-flow__controls {
    @apply !bg-white !border-slate-200 !shadow-xl !rounded-lg !left-6 !bottom-6;
}

.vue-flow__controls-button {
    @apply !border-slate-100 !fill-slate-400 hover:!bg-slate-50 !transition-colors;
}

.vue-flow__background {
    @apply !bg-white;
}

/* Animation */
@keyframes spin-slow {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

input::placeholder {
    @apply opacity-30;
}

input:focus::placeholder {
    @apply opacity-10;
}

.line-clamp-3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
</style>
