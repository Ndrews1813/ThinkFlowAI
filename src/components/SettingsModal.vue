<script setup lang="ts">
/**
 * 设置弹窗（API 配置）
 * - 默认模式：使用内置 DEFAULT_CONFIG + 环境变量 API Key（无需输入）
 * - 自定义模式：允许分别配置“文本生成/图片生成”的 baseUrl、model、apiKey
 * - 写入逻辑不在组件内：apiConfig 是响应式对象，持久化由 useThinkFlow 的 watch 完成
 */

// 图标：分区与输入项标识
import { Activity, Image as ImageIcon, Key, Link as LinkIcon, Settings, Shield, Sparkles, X } from 'lucide-vue-next'

/**
 * props：
 * - show：弹窗显示开关
 * - t：i18n 翻译函数
 * - apiConfig：由 useThinkFlow 提供的响应式配置对象
 */
const props = defineProps<{
    show: boolean
    t: any
    apiConfig: any
}>()

/**
 * close：关闭弹窗（保存按钮也仅触发关闭，真实保存由响应式 + watch 完成）
 */
const emit = defineEmits<{
    (e: 'close'): void
}>()
</script>

<template>
    <div v-if="props.show" class="fixed inset-0 z-[100] flex items-center justify-center bg-slate-900/40 backdrop-blur-sm p-4" @click.self="emit('close')">
        <div class="bg-white rounded-3xl shadow-2xl w-full max-w-2xl overflow-hidden border border-slate-200 animate-in fade-in zoom-in duration-300">
            <div class="px-8 py-6 border-b border-slate-100 flex items-center justify-between bg-slate-50/50">
                <div class="flex items-center gap-3">
                    <div class="p-2 bg-slate-900 rounded-xl text-white">
                        <Settings class="w-5 h-5" />
                    </div>
                    <div>
                        <h3 class="text-lg font-black text-slate-900 tracking-tight">{{ props.t('settings.title') }}</h3>
                        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">{{ props.t('settings.subtitle') }}</p>
                    </div>
                </div>
                <button @click="emit('close')" class="p-2 hover:bg-slate-200 rounded-xl transition-colors">
                    <X class="w-5 h-5 text-slate-500" />
                </button>
            </div>

            <div class="p-8 py-4 space-y-8 max-h-[70vh] overflow-y-auto custom-scrollbar">
                <div class="flex p-1.5 bg-slate-100 rounded-2xl w-fit">
                    <button
                        @click="props.apiConfig.mode = 'default'"
                        class="px-6 py-2 rounded-xl text-xs font-black tracking-widest transition-all"
                        :class="props.apiConfig.mode === 'default' ? 'bg-white text-slate-900 shadow-sm' : 'text-slate-500 hover:text-slate-700'"
                    >
                        {{ props.t('common.default') }}
                    </button>
                    <button
                        @click="props.apiConfig.mode = 'custom'"
                        class="px-6 py-2 rounded-xl text-xs font-black tracking-widest transition-all"
                        :class="props.apiConfig.mode === 'custom' ? 'bg-white text-slate-900 shadow-sm' : 'text-slate-500 hover:text-slate-700'"
                    >
                        {{ props.t('common.custom') }}
                    </button>
                </div>

                <div v-if="props.apiConfig.mode === 'custom'" class="space-y-4 animate-in slide-in-from-top-2 duration-300">
                    <div class="flex items-center gap-2 text-slate-900">
                        <Sparkles class="w-4 h-4 text-orange-500" />
                        <span class="text-sm font-black uppercase tracking-widest">{{ props.t('settings.textGen') }}</span>
                    </div>
                    <div class="grid grid-cols-1 gap-4 p-5 bg-slate-50 rounded-2xl border border-slate-100">
                        <div class="space-y-1.5">
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                <LinkIcon class="w-3 h-3" /> {{ props.t('settings.baseUrl') }}
                            </label>
                            <input
                                v-model="props.apiConfig.chat.baseUrl"
                                type="text"
                                :placeholder="props.t('settings.placeholderUrl')"
                                class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-orange-500/20 focus:border-orange-500 outline-none transition-all font-mono"
                            />
                        </div>
                        <div class="grid grid-cols-2 gap-4">
                            <div class="space-y-1.5">
                                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                    <Shield class="w-3 h-3" /> {{ props.t('settings.modelName') }}
                                </label>
                                <input
                                    v-model="props.apiConfig.chat.model"
                                    type="text"
                                    :placeholder="props.t('settings.placeholderModel')"
                                    class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-orange-500/20 focus:border-orange-500 outline-none transition-all font-mono"
                                />
                            </div>
                            <div class="space-y-1.5">
                                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                    <Key class="w-3 h-3" /> {{ props.t('settings.apiKey') }}
                                </label>
                                <input
                                    v-model="props.apiConfig.chat.apiKey"
                                    type="password"
                                    :placeholder="props.t('settings.placeholderKey')"
                                    class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-orange-500/20 focus:border-orange-500 outline-none transition-all font-mono"
                                />
                            </div>
                        </div>
                    </div>
                </div>

                <div v-if="props.apiConfig.mode === 'custom'" class="space-y-4 animate-in slide-in-from-top-2 duration-300">
                    <div class="flex items-center gap-2 text-slate-900">
                        <ImageIcon class="w-4 h-4 text-blue-500" />
                        <span class="text-sm font-black uppercase tracking-widest">{{ props.t('settings.imageGen') }}</span>
                    </div>
                    <div class="grid grid-cols-1 gap-4 p-5 bg-slate-50 rounded-2xl border border-slate-100">
                        <div class="space-y-1.5">
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                <LinkIcon class="w-3 h-3" /> {{ props.t('settings.baseUrl') }}
                            </label>
                            <input
                                v-model="props.apiConfig.image.baseUrl"
                                type="text"
                                :placeholder="props.t('settings.placeholderUrl')"
                                class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500 outline-none transition-all font-mono"
                            />
                        </div>
                        <div class="grid grid-cols-2 gap-4">
                            <div class="space-y-1.5">
                                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                    <Shield class="w-3 h-3" /> {{ props.t('settings.modelName') }}
                                </label>
                                <input
                                    v-model="props.apiConfig.image.model"
                                    type="text"
                                    :placeholder="props.t('settings.placeholderModel')"
                                    class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500 outline-none transition-all font-mono"
                                />
                            </div>
                            <div class="space-y-1.5">
                                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest flex items-center gap-1">
                                    <Key class="w-3 h-3" /> {{ props.t('settings.apiKey') }}
                                </label>
                                <input
                                    v-model="props.apiConfig.image.apiKey"
                                    type="password"
                                    :placeholder="props.t('settings.placeholderKey')"
                                    class="w-full px-4 py-2.5 bg-white border border-slate-200 rounded-xl text-sm focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500 outline-none transition-all font-mono"
                                />
                            </div>
                        </div>
                    </div>
                </div>

                <div v-if="props.apiConfig.mode === 'default'" class="flex flex-col items-center justify-center py-12 text-center space-y-4 animate-in fade-in duration-500">
                    <div class="w-16 h-16 bg-slate-100 rounded-full flex items-center justify-center">
                        <Activity class="w-8 h-8 text-slate-400" />
                    </div>
                    <div class="space-y-1">
                        <h4 class="text-sm font-black text-slate-900 uppercase">{{ props.t('settings.defaultModeTitle') }}</h4>
                        <p class="text-xs text-slate-500 max-w-[280px]">{{ props.t('settings.defaultModeDesc') }}</p>
                    </div>
                </div>
            </div>

            <div class="px-8 py-6 border-t border-slate-100 bg-slate-50/50 flex justify-end gap-3">
                <button
                    @click="emit('close')"
                    class="px-8 py-2.5 bg-slate-900 text-white rounded-xl text-xs font-black tracking-widest hover:bg-slate-800 transition-all shadow-lg shadow-slate-200 active:scale-95"
                >
                    {{ props.t('common.save') }}
                </button>
            </div>
        </div>
    </div>
</template>
