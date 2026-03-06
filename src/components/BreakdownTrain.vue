// Xpilot Copyright 2025 [Fyisvia Virell] — https://mj.fyisvia.com
// Licensed under AGPL-3.0 with Additional Terms (see LICENSE).
// Note: Certain non-code assets (including datasets, content sets, or media files)
// are excluded from the AGPL license and may NOT be publicly published or redistributed
// without written permission from the author. (See LICENSE for details)

<template>
    <ul class="list bg-base-100 sm:rounded-box sm:shadow-md w-[100%] px-2 sm:px-8">

        <li aria-hidden="true" role="presentation" class="p-0 m-0 sm:h-4"></li>

        <li class="p-4 pb-2 text-lg font-semibold opacity-100 tracking-wide">
            {{ t('breakdownTrain.title') }}
        </li>
        <li aria-hidden="true" role="presentation" class="p-0 m-0 h-2"></li>

        <li class="p-4 pb-2 text-base opacity-100 tracking-wide">
            <div class="flex flex-col sm:flex-row sm:items-center sm:gap-8">
                <fieldset class="fieldset ml-0 flex-1">
                    <legend class="text-base">{{ t('breakdownTrain.settings.questionLegend') }}</legend>
                    <div class="flex items-center gap-2 flex-wrap">
                        <span class="text-sm font-medium">{{ currentIndex + 1 }} / {{ problems.length }}</span>
                        <input type="number" v-model.number="jumpInputValue" min="1" :max="problems.length" 
                            class="input input-bordered input-sm w-16" 
                            @keyup.enter="handleJumpToQuestion" />
                        <button class="btn btn-sm btn-outline" @click="handleJumpToQuestion">
                            {{ t('breakdownTrain.buttons.jump') }}
                        </button>
                    </div>
                </fieldset>
            </div>
        </li>

        <li class="p-4 pb-2 opacity-100 tracking-wide text-base sm:text-lg font-semibold" :style="contentWidthStyle">
            <div class="flex items-center gap-4">
                <span>{{ t('breakdownTrain.ui.handTitle') }}</span>
                <span class="badge badge-lg badge-primary">{{ t('breakdownTrain.ui.shanten') }}: {{ currentShanten }}</span>
            </div>
        </li>

        <li class="p-2 sm:p-4 pb-2 text-xs sm:text-sm md:text-base opacity-100 tracking-wide"
            :style="contentWidthStyle">
            <div class="grid w-full" :style="{
                gridTemplateColumns: `repeat(${handTiles.length || 1}, 1fr)`,
                gap: '0px'
            }">
                <template v-for="(tile, index) in handTiles" :key="tile + '-' + index">
                    <img :src="tileSrc(tile)" :alt="tile" @click="handleTileClick(tile)"
                        @mouseenter="hoveredIndex = index" @mouseleave="hoveredIndex = null"
                        class="tile-img transition-transform duration-150 cursor-pointer" :style="{
                            borderRadius: '5px',
                            transform: hoveredIndex === index ? 'translateY(-5px)' : 'none',
                            filter: (selectedAnswer === tile) ? 'grayscale(100%) brightness(0.9)' : undefined,
                            opacity: (selectedAnswer === tile) ? '0.8' : undefined,
                            boxShadow: (selectedAnswer === tile) ? '0 0 10px rgba(34, 197, 94, 0.5)' : undefined
                        }" />
                </template>
            </div>
        </li>

        <li class="p-4 pb-2 text-xs md:text-base opacity-80 tracking-wide flex items-center gap-4 mx-2">
            <div class="flex items-center gap-3">
                <span v-if="selectedAnswer" class="flex items-center gap-1">
                    {{ t('breakdownTrain.ui.selected') }}:
                    <img :src="tileSrc(selectedAnswer)" :alt="selectedAnswer" class="tile-img-tiny" />
                </span>
                <span v-else class="opacity-60">
                    {{ t('breakdownTrain.ui.selectHint') }}
                </span>
            </div>
        </li>

        <li v-if="selectedAnswer" class="p-4 pb-2 text-xs md:text-base opacity-100">
            <button class="btn btn-primary btn-sm md:btn-md" @click="handleSubmitAnswer">
                {{ t('breakdownTrain.buttons.submit') }}
            </button>
            <button class="btn btn-outline btn-sm md:btn-md ml-2" @click="handleSkip">
                {{ t('breakdownTrain.buttons.skip') }}
            </button>
        </li>

        <li v-if="lastResult" aria-hidden="true" role="presentation" class="p-0 m-0 h-3"></li>

        <li v-if="lastResult" class="p-4 pb-2 text-base font-semibold">
            <div class="alert" :class="lastResult.isCorrect ? 'alert-success' : 'alert-error'">
                <span>{{ lastResult.isCorrect ? t('breakdownTrain.result.correct') : t('breakdownTrain.result.wrong')
                    }}</span>
                <span v-if="!lastResult.isCorrect" class="flex items-center gap-1">
                    {{ t('breakdownTrain.result.correctAnswer') }}:
                    <img :src="tileSrc(lastResult.correctAnswer)" :alt="lastResult.correctAnswer" class="tile-img-small" />
                </span>
            </div>
        </li>

        <li aria-hidden="true" role="presentation" class="p-0 m-0 h-3"></li>

        <li class="p-4 pb-2 flex items-center gap-2 mx-2">
            <div class="flex items-center gap-2 flex-wrap">
                <button class="btn btn-outline btn-sm md:btn-md" @click="handlePrevQuestion"
                    :disabled="currentIndex === 0">
                    {{ t('breakdownTrain.buttons.prev') }}
                </button>
                <button class="btn btn-outline btn-sm md:btn-md" @click="handleNextQuestion">
                    {{ t('breakdownTrain.buttons.next') }}
                </button>
            </div>
        </li>

        <li aria-hidden="true" role="presentation" class="p-0 m-0 h-2"></li>
        <!-- 本题解析 -->
        <li class="p-4 pb-2 flex items-center gap-2 mx-2">
            <div class="flex items-center gap-2 flex-wrap w-full">
                <div class="collapse collapse-arrow bg-base-100 border-base-300 border flex-1">
                    <input type="checkbox" v-model="showAnalysis" />
                    <div class="collapse-title text-base sm:text-lg font-semibold text-center pl-12">
                        {{ t('breakdownTrain.analysis.title') }}
                    </div>
                    <div class="collapse-content text-sm sm:text-base md:text-lg mb-0">
                        <div class="prose prose-sm max-w-none">
                            {{ currentProblem.tips }}
                        </div>
                    </div>
                </div>
            </div>
        </li>

        <li aria-hidden="true" role="presentation" class="p-0 m-0 h-2"></li>


        <!-- 进张数分析 -->
        <li class="p-4 pb-2 flex items-center gap-2 mx-2">
            <div class="flex items-center gap-2 flex-wrap w-full">
                <div class="collapse collapse-arrow bg-base-100 border-base-300 border flex-1">
                    <input type="checkbox" v-model="showEfficiencyAnalysis" />
                    <div class="collapse-title text-base sm:text-lg font-semibold text-center pl-12">
                        {{ t('breakdownTrain.efficiencyAnalysis.title') }}
                    </div>
                    <div class="collapse-content text-sm sm:text-base md:text-lg mb-0">
                        <div class="overflow-x-auto">
                            <table class="table table-sm w-full bg-base-100 rounded-lg">
                                <thead>
                                    <tr>
                                        <th class="text-center">{{ t('breakdownTrain.table.cut') }}</th>
                                        <th class="text-center">{{ t('breakdownTrain.table.improvements') }}</th>
                                        <th class="text-center">{{ t('breakdownTrain.table.goodShapeRate') }}</th>
                                        <th class="text-center">{{ t('breakdownTrain.table.total') }}</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="([tile, result], idx) in sortedImprovementResults" :key="tile"
                                        :class="idx % 2 === 1 ? 'hover:bg-base-300' : ''">
                                        <td class="font-bold text-center">
                                            <img :src="tileSrc(tile)" :alt="tile" class="tile-img-small inline-block" />
                                        </td>
                                        <td class="text-center">
                                            <div class="flex flex-wrap gap-2 justify-center">
                                                <div v-for="(count, improveTile) in result.improvements" :key="improveTile" class="inline-flex items-center gap-0.5">
                                                    <img :src="tileSrc(improveTile)" :alt="improveTile" class="tile-img-tiny" />
                                                    <span class="bg-red-300 text-white text-xs font-bold rounded-full w-5 h-5 flex items-center justify-center flex-shrink-0">{{ count }}</span>
                                                </div>
                                            </div>
                                        </td>
                                        <td class="font-bold text-center">{{ result.goodShapeRate.toFixed(0) }}%</td>
                                        <td class="font-bold text-center">{{ result.totalCount }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </li>

    </ul>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { Shanten } from '../utils/shanten'
import { breakdownProblems } from '../data/breakdownProblems'

const { t } = useI18n()

// 统一的图片路径函数
const tileSrc = (tile) => `/mahjongfiles/${tile}.png`

// -------------------- 响应式状态 --------------------
const problems = ref(breakdownProblems)
const currentIndex = ref(0)
const handTiles = ref([])
const selectedAnswer = ref(null)
const lastResult = ref(null)
const hoveredIndex = ref(null)
const showAnalysis = ref(false)
const showEfficiencyAnalysis = ref(false)
const jumpInputValue = ref(1)
const STORAGE_KEY = 'breakdownTrain_lastQuestionIndex'

// 当题目索引改变时保存到本地存储
import { watch } from 'vue'

watch(currentIndex, (newIndex) => {
    localStorage.setItem(STORAGE_KEY, newIndex.toString())
})

const contentWidth = ref(100)
const contentWidthStyle = computed(() => ({
    width: `${contentWidth.value}%`,
    marginLeft: 'auto',
    marginRight: 'auto'
}))

// 进张分析相关状态
const improvementResults = ref({})
let goodShapeCache = new Map()
const baseCountMap = ref({})
const uniqueTiles = computed(() => Object.keys(baseCountMap.value))

// 当前问题
const currentProblem = computed(() => {
    if (currentIndex.value >= 0 && currentIndex.value < problems.value.length) {
        return problems.value[currentIndex.value]
    }
    return null
})

// 当前向听数
const currentShanten = computed(() => {
    if (handTiles.value.length > 0) {
        const arr34 = convertToTiles34Arr(handTiles.value)
        return calcShanten(arr34)
    }
    return 0
})

// 手牌花色权重
const TYPE_ORDER = { m: 0, p: 1, s: 2, z: 3 }
// 牌面数值
const numVal = (n) => (n === '0' ? 5.5 : +n)

// 手牌排序
const sortTiles = (tiles) => {
    return [...tiles].sort((a, b) => {
        const ta = a.slice(-1), tb = b.slice(-1)
        if (TYPE_ORDER[ta] !== TYPE_ORDER[tb]) return TYPE_ORDER[ta] - TYPE_ORDER[tb]
        const na = numVal(a[0]), nb = numVal(b[0])
        return na - nb
    })
}

// 将卡牌字符串转换为数组
const parseCardsString = (cardsStr) => {
    const tiles = []
    let currentNumber = ''

    for (let i = 0; i < cardsStr.length; i++) {
        const char = cardsStr[i]
        if (char === 'm' || char === 'p' || char === 's' || char === 'z') {
            // 花色字符
            for (const num of currentNumber) {
                tiles.push(num + char)
            }
            currentNumber = ''
        } else {
            // 数字字符
            currentNumber += char
        }
    }

    return sortTiles(tiles)
}

// -------------------- Shanten 计算器 --------------------
const calcShanten = (arr) => new Shanten().calculateShanten(arr)

// -------------------- 牌面处理 --------------------
// 生成所有可用牌
const generateAllTiles = () => {
    const suits = ['m', 'p', 's']
    const tiles = {}

    for (const suit of suits) {
        for (let i = 1; i <= 9; i++) {
            if (i === 5) {
                tiles[`5${suit}`] = 3
                tiles[`0${suit}`] = 1
            } else {
                tiles[`${i}${suit}`] = 4
            }
        }
    }

    return tiles
}

// 手牌 -> 34进制计数数组
const convertToTiles34Arr = (tiles) => {
    const arr = Array(34).fill(0)
    for (const t of tiles) {
        const n = t[0] === '0' ? 5 : +t[0]
        const tp = t.slice(-1)
        let idx
        if (tp === 'm') idx = n - 1
        else if (tp === 'p') idx = 9 + (n - 1)
        else if (tp === 's') idx = 18 + (n - 1)
        else if (tp === 'z') idx = 27 + (n - 1)
        if (idx !== undefined) arr[idx]++
    }
    return arr
}

// 构建手牌计数
const buildHandCountMap = (hand) => {
    const cnt = {}
    for (const t of hand) cnt[t] = (cnt[t] || 0) + 1
    return cnt
}

// 计算剩余牌
const remainingCounts = (handCount) => {
    const res = {}
    for (const tile of uniqueTiles.value) {
        const left = (baseCountMap.value[tile] || 0) - (handCount[tile] || 0)
        if (left > 0) res[tile] = left
    }
    return res
}

// -------------------- 好型率计算 --------------------
const MAX_GOOD_SHAPE_DEPTH = 3
const calculateGoodShapeRate = (hand, arr34, depth = 0) => {
    const key = arr34.join(',') + '|' + depth
    if (goodShapeCache.has(key)) return goodShapeCache.get(key)
    if (depth > MAX_GOOD_SHAPE_DEPTH) return 0

    const shanten = calcShanten(arr34)

    if (shanten === 0) {
        const handCount = buildHandCountMap(hand)
        const rem = remainingCounts(handCount)
        const waitTypes = new Set()
        let waitCount = 0
        for (const [tile, count] of Object.entries(rem)) {
            const tempHand = [...hand, tile]
            const temp34 = convertToTiles34Arr(tempHand)
            if (calcShanten(temp34) === -1) {
                waitTypes.add(tile)
                waitCount += count
            }
        }
        const res = (waitTypes.size > 1 && waitCount > 4) ? 1 : 0
        goodShapeCache.set(key, res)
        return res
    }

    const handCount = buildHandCountMap(hand)
    const rem = remainingCounts(handCount)
    let total = 0
    let good = 0

    for (const [tile, count] of Object.entries(rem)) {
        const tempHand = [...hand, tile]
        const temp34 = convertToTiles34Arr(tempHand)
        const newShanten = calcShanten(temp34)
        if (newShanten < shanten) {
            let bestPath = 0
            const seen = new Set()
            for (let i = 0; i < tempHand.length; i++) {
                const d = tempHand[i]
                if (seen.has(d)) continue
                seen.add(d)
                const discardHand = tempHand.slice()
                discardHand.splice(i, 1)
                const discard34 = convertToTiles34Arr(discardHand)
                if (calcShanten(discard34) < shanten) {
                    bestPath = Math.max(bestPath, calculateGoodShapeRate(discardHand, discard34, depth + 1))
                }
            }
            total += count
            good += count * bestPath
        }
    }

    const res = total === 0 ? 0 : good / total
    goodShapeCache.set(key, res)
    return res
}

// -------------------- 进张分析 --------------------
const analyzeImprovement = (currentHand, current34) => {
    const results = {}
    const originalShanten = calcShanten(current34)
    const globalHandCount = buildHandCountMap(currentHand)

    currentHand.forEach((tileToDiscard, discardIndex) => {
        const newHand = currentHand.filter((_, i) => i !== discardIndex)
        const newHandCount = { ...globalHandCount }
        newHandCount[tileToDiscard]--
        if (newHandCount[tileToDiscard] === 0) delete newHandCount[tileToDiscard]

        const new34 = convertToTiles34Arr(newHand)
        const rem = remainingCounts(newHandCount)

        const improvements = {}
        let totalCount = 0

        for (const [tile, count] of Object.entries(rem)) {
            const tempHand = [...newHand, tile]
            const temp34 = convertToTiles34Arr(tempHand)
            if (calcShanten(temp34) < originalShanten) {
                improvements[tile] = count
                totalCount += count
            }
        }

        if (totalCount > 0) {
            const goodShapeRate = Math.max(0, calculateGoodShapeRate(newHand, new34) * 100)
            results[tileToDiscard] = { improvements, totalCount, goodShapeRate }
        }
    })

    return results
}

// 格式化进张显示
const formatImprovements = (improvements) =>
    sortTiles(Object.keys(improvements)).join(', ')

// 排序进张结果
const sortedImprovementResults = computed(() => {
    return Object.entries(improvementResults.value || {}).sort(
        (a, b) => ((b[1].totalCount || 0) - (a[1].totalCount || 0))
    )
})

// 加载当前问题的手牌
const loadCurrentProblem = () => {
    if (currentProblem.value) {
        handTiles.value = parseCardsString(currentProblem.value.cards)
        selectedAnswer.value = null
        lastResult.value = null
        showAnalysis.value = false
        showEfficiencyAnalysis.value = false

        // 初始化牌堆计数（所有可用牌）
        baseCountMap.value = generateAllTiles()

        // 计算进张分析
        goodShapeCache = new Map()
        const arr34 = convertToTiles34Arr(handTiles.value)
        improvementResults.value = analyzeImprovement(handTiles.value, arr34)
    }
}

// 处理牌的点击
const handleTileClick = (tile) => {
    selectedAnswer.value = selectedAnswer.value === tile ? null : tile
}

// 处理提交答案
const handleSubmitAnswer = () => {
    if (!selectedAnswer.value || !currentProblem.value) return

    const isCorrect = selectedAnswer.value === currentProblem.value.answer

    lastResult.value = {
        isCorrect,
        correctAnswer: currentProblem.value.answer,
        selectedAnswer: selectedAnswer.value
    }

    // 保存当前题目index到localStorage
    localStorage.setItem(STORAGE_KEY, currentIndex.value.toString())

    showAnalysis.value = true
}

// 跳过当前问题
const handleSkip = () => {
    selectedAnswer.value = null
    lastResult.value = null
    showAnalysis.value = false
}

// 下一题
const handleNextQuestion = () => {
    if (currentIndex.value < problems.value.length - 1) {
        currentIndex.value += 1
        loadCurrentProblem()
        jumpInputValue.value = currentIndex.value + 1
    }
}

// 上一题
const handlePrevQuestion = () => {
    if (currentIndex.value > 0) {
        currentIndex.value -= 1
        loadCurrentProblem()
        jumpInputValue.value = currentIndex.value + 1
    }
}

// 跳转到指定题目
const handleJumpToQuestion = () => {
    const targetIndex = jumpInputValue.value - 1
    if (targetIndex >= 0 && targetIndex < problems.value.length) {
        currentIndex.value = targetIndex
        loadCurrentProblem()
        jumpInputValue.value = currentIndex.value + 1
    }
}

// 初始化
onMounted(() => {
    // 从localStorage读取上次答题的index
    const savedIndex = localStorage.getItem(STORAGE_KEY)
    if (savedIndex !== null) {
        const index = parseInt(savedIndex, 10)
        if (index >= 0 && index < problems.value.length) {
            currentIndex.value = index
        }
    }
    loadCurrentProblem()
    jumpInputValue.value = currentIndex.value + 1
})
</script>

<style scoped>
/* 牌面图片：自适应容器宽度，保持纵横比 */
.tile-img {
    width: 100%;
    height: auto;
    object-fit: contain;
}

.tile-preview {
    height: 24px;
    width: auto;
    display: inline-block;
    margin-left: 4px;
    vertical-align: middle;
}

/* 小牌面图片（进张表格中使用） */
.tile-img-tiny {
    height: 36px;
    width: auto;
    object-fit: contain;
    display: inline-block;
}

.tile-img-small {
    height: 44px;
    width: auto;
    object-fit: contain;
    display: inline-block;
}

.prose {
    line-height: 1.6;
}
</style>
