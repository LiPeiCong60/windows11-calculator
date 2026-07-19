<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const displayValue = ref('0')
const expressionValue = ref('')
const storedOperand = ref(null)
const pendingOperator = ref(null)
const lastOperator = ref(null)
const lastOperand = ref(null)
const shouldResetDisplay = ref(false)
const hasError = ref(false)
const memoryValues = ref([])
const isMemoryPanelOpen = ref(false)
const historyEntries = ref([])
const isHistoryPanelOpen = ref(false)
const currentPage = ref('calculator')
const isNavigationOpen = ref(false)
const selectedTheme = ref('system')
const systemPrefersDark = ref(false)
const isThemeOptionsOpen = ref(false)
let nextHistoryId = 1
let systemThemeMediaQuery
const MAX_INPUT_DIGITS = 16

const themeLabels = {
  light: '浅色',
  dark: '深色',
  system: '使用系统设置',
}

const selectedThemeLabel = computed(() => themeLabels[selectedTheme.value])
const resolvedTheme = computed(() => {
  if (selectedTheme.value === 'system') {
    return systemPrefersDark.value ? 'dark' : 'light'
  }

  return selectedTheme.value
})

const operatorSymbols = {
  add: '+',
  subtract: '−',
  multiply: '×',
  divide: '÷',
}

function inputDigit(digit) {
  if (hasError.value) {
    clearAll()
  }

  if (shouldResetDisplay.value) {
    if (pendingOperator.value === null) {
      expressionValue.value = ''
      lastOperator.value = null
      lastOperand.value = null
    }

    displayValue.value = digit
    shouldResetDisplay.value = false
    return
  }

  if (displayValue.value.replace(/\D/g, '').length >= MAX_INPUT_DIGITS) {
    return
  }

  displayValue.value = displayValue.value === '0' ? digit : `${displayValue.value}${digit}`
}

function inputDecimal() {
  if (hasError.value) {
    clearAll()
  }

  if (shouldResetDisplay.value) {
    if (pendingOperator.value === null) {
      expressionValue.value = ''
      lastOperator.value = null
      lastOperand.value = null
    }

    displayValue.value = '0.'
    shouldResetDisplay.value = false
    return
  }

  if (!displayValue.value.includes('.')) {
    displayValue.value += '.'
  }
}

function clearEntry() {
  displayValue.value = '0'
  shouldResetDisplay.value = false
  hasError.value = false
}

function clearAll() {
  clearEntry()
  expressionValue.value = ''
  storedOperand.value = null
  pendingOperator.value = null
  lastOperator.value = null
  lastOperand.value = null
}

function deleteLastDigit() {
  if (shouldResetDisplay.value || hasError.value) {
    return
  }

  const nextValue = displayValue.value.slice(0, -1)
  displayValue.value = nextValue && nextValue !== '-' ? nextValue : '0'
}

function toggleSign() {
  if (hasError.value) {
    return
  }

  if (shouldResetDisplay.value && pendingOperator.value !== null) {
    return
  }

  shouldResetDisplay.value = false

  if (displayValue.value === '0') {
    return
  }

  displayValue.value = displayValue.value.startsWith('-')
    ? displayValue.value.slice(1)
    : `-${displayValue.value}`
}

function selectOperator(operator) {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value !== null) {
    if (shouldResetDisplay.value) {
      pendingOperator.value = operator
      expressionValue.value = `${storedOperand.value} ${operatorSymbols[operator]}`
    } else {
      calculateResult()
      selectOperator(operator)
    }

    return
  }

  storedOperand.value = Number(displayValue.value)
  pendingOperator.value = operator
  shouldResetDisplay.value = true
  expressionValue.value = `${displayValue.value} ${operatorSymbols[operator]}`
}

function formatResult(value) {
  const roundedValue = Number(value.toPrecision(15))
  return Object.is(roundedValue, -0) ? '0' : String(roundedValue)
}

function showError(message, expression = '') {
  displayValue.value = message
  expressionValue.value = expression
  storedOperand.value = null
  pendingOperator.value = null
  lastOperator.value = null
  lastOperand.value = null
  shouldResetDisplay.value = true
  hasError.value = true
}

function formatFiniteResult(value, expression = '') {
  if (!Number.isFinite(value)) {
    showError('溢出', expression)
    return null
  }

  return formatResult(value)
}

function applyPercentage() {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value === null) {
    const originalValue = displayValue.value
    const completedExpression = `${originalValue}% =`
    const formattedResult = formatFiniteResult(Number(displayValue.value) / 100, completedExpression)

    if (formattedResult === null) {
      return
    }

    displayValue.value = formattedResult
    expressionValue.value = completedExpression
    lastOperator.value = null
    lastOperand.value = null
    shouldResetDisplay.value = true
    return
  }

  if (shouldResetDisplay.value) {
    return
  }

  const currentValue = Number(displayValue.value)
  const percentageValue = ['add', 'subtract'].includes(pendingOperator.value)
    ? (storedOperand.value * currentValue) / 100
    : currentValue / 100

  const formattedResult = formatFiniteResult(percentageValue, expressionValue.value)

  if (formattedResult === null) {
    return
  }

  displayValue.value = formattedResult
  expressionValue.value = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} ${displayValue.value}`
}

function applySquare() {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value !== null && shouldResetDisplay.value) {
    return
  }

  const originalValue = displayValue.value
  const completedExpression = `sqr(${originalValue}) =`
  const formattedResult = formatFiniteResult(Number(displayValue.value) ** 2, completedExpression)

  if (formattedResult === null) {
    return
  }

  displayValue.value = formattedResult

  if (pendingOperator.value !== null) {
    expressionValue.value = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} sqr(${originalValue})`
    shouldResetDisplay.value = false
    return
  }

  expressionValue.value = completedExpression
  lastOperator.value = null
  lastOperand.value = null
  shouldResetDisplay.value = true
}

function applySquareRoot() {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value !== null && shouldResetDisplay.value) {
    return
  }

  const originalValue = displayValue.value
  if (Number(originalValue) < 0) {
    showError('输入无效', `√(${originalValue})`)
    return
  }

  displayValue.value = formatResult(Math.sqrt(Number(displayValue.value)))

  if (pendingOperator.value !== null) {
    expressionValue.value = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} √(${originalValue})`
    shouldResetDisplay.value = false
    return
  }

  expressionValue.value = `√(${originalValue}) =`
  lastOperator.value = null
  lastOperand.value = null
  shouldResetDisplay.value = true
}

function applyReciprocal() {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value !== null && shouldResetDisplay.value) {
    return
  }

  const originalValue = displayValue.value
  if (Number(originalValue) === 0) {
    showError('无法除以零', `1/(${originalValue})`)
    return
  }

  displayValue.value = formatResult(1 / Number(displayValue.value))

  if (pendingOperator.value !== null) {
    expressionValue.value = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} 1/(${originalValue})`
    shouldResetDisplay.value = false
    return
  }

  expressionValue.value = `1/(${originalValue}) =`
  lastOperator.value = null
  lastOperand.value = null
  shouldResetDisplay.value = true
}

function clearMemory() {
  memoryValues.value = []
  isMemoryPanelOpen.value = false
}

function recallMemoryValue(memoryValue) {
  displayValue.value = formatResult(memoryValue)
  hasError.value = false

  if (pendingOperator.value === null) {
    expressionValue.value = ''
    lastOperator.value = null
    lastOperand.value = null
  }

  shouldResetDisplay.value = false
  isMemoryPanelOpen.value = false
}

function recallMemory() {
  if (memoryValues.value.length > 0) {
    recallMemoryValue(memoryValues.value[0])
  }
}

function storeMemory() {
  if (hasError.value) {
    return
  }

  memoryValues.value.unshift(Number(displayValue.value))
}

function addToMemory() {
  if (hasError.value) {
    return
  }

  const currentValue = Number(displayValue.value)

  if (memoryValues.value.length === 0) {
    memoryValues.value.push(currentValue)
    return
  }

  memoryValues.value[0] = Number(formatResult(memoryValues.value[0] + currentValue))
}

function subtractFromMemory() {
  if (hasError.value) {
    return
  }

  const currentValue = Number(displayValue.value)

  if (memoryValues.value.length === 0) {
    memoryValues.value.push(-currentValue)
    return
  }

  memoryValues.value[0] = Number(formatResult(memoryValues.value[0] - currentValue))
}

function toggleMemoryPanel() {
  if (memoryValues.value.length > 0) {
    isMemoryPanelOpen.value = !isMemoryPanelOpen.value
    isHistoryPanelOpen.value = false
  }
}

function addHistoryEntry(expression, result) {
  historyEntries.value.unshift({
    id: nextHistoryId,
    expression,
    result,
  })
  nextHistoryId += 1
}

function toggleHistoryPanel() {
  isHistoryPanelOpen.value = !isHistoryPanelOpen.value
  isMemoryPanelOpen.value = false
}

function clearHistory() {
  historyEntries.value = []
}

function recallHistoryResult(result) {
  displayValue.value = result
  hasError.value = false
  expressionValue.value = ''
  storedOperand.value = null
  pendingOperator.value = null
  lastOperator.value = null
  lastOperand.value = null
  shouldResetDisplay.value = true
  isHistoryPanelOpen.value = false
}

function showCalculator() {
  currentPage.value = 'calculator'
  isNavigationOpen.value = false
  isMemoryPanelOpen.value = false
  isHistoryPanelOpen.value = false
}

function showSettings() {
  currentPage.value = 'settings'
  isNavigationOpen.value = false
  isMemoryPanelOpen.value = false
  isHistoryPanelOpen.value = false
}

function toggleNavigation() {
  isNavigationOpen.value = !isNavigationOpen.value
  isMemoryPanelOpen.value = false
  isHistoryPanelOpen.value = false
}

function selectTheme(theme) {
  selectedTheme.value = theme
  isThemeOptionsOpen.value = false
}

function updateSystemTheme(event) {
  systemPrefersDark.value = event.matches
}

function calculateResult() {
  if (hasError.value) {
    return
  }

  if (pendingOperator.value === null) {
    if (lastOperator.value === null || !shouldResetDisplay.value) {
      return
    }

    const firstOperand = Number(displayValue.value)
    let repeatedResult

    switch (lastOperator.value) {
      case 'add':
        repeatedResult = firstOperand + lastOperand.value
        break
      case 'subtract':
        repeatedResult = firstOperand - lastOperand.value
        break
      case 'multiply':
        repeatedResult = firstOperand * lastOperand.value
        break
      case 'divide':
        if (lastOperand.value === 0) {
          const repeatedExpression = `${displayValue.value} ${operatorSymbols[lastOperator.value]} ${lastOperand.value} =`
          showError(firstOperand === 0 ? '结果未定义' : '无法除以零', repeatedExpression)
          return
        }
        repeatedResult = firstOperand / lastOperand.value
        break
    }

    const repeatedExpression = `${displayValue.value} ${operatorSymbols[lastOperator.value]} ${lastOperand.value} =`
    const formattedRepeatedResult = formatFiniteResult(repeatedResult, repeatedExpression)

    if (formattedRepeatedResult === null) {
      return
    }

    expressionValue.value = repeatedExpression
    displayValue.value = formattedRepeatedResult
    addHistoryEntry(repeatedExpression, formattedRepeatedResult)
    return
  }

  if (shouldResetDisplay.value) {
    return
  }

  const secondOperand = Number(displayValue.value)
  let result

  switch (pendingOperator.value) {
    case 'add':
      result = storedOperand.value + secondOperand
      break
    case 'subtract':
      result = storedOperand.value - secondOperand
      break
    case 'multiply':
      result = storedOperand.value * secondOperand
      break
    case 'divide':
      if (secondOperand === 0) {
        const completedExpression = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} ${displayValue.value} =`
        showError(storedOperand.value === 0 ? '结果未定义' : '无法除以零', completedExpression)
        return
      }
      result = storedOperand.value / secondOperand
      break
  }

  lastOperator.value = pendingOperator.value
  lastOperand.value = secondOperand
  const completedExpression = `${storedOperand.value} ${operatorSymbols[pendingOperator.value]} ${displayValue.value} =`
  const formattedResult = formatFiniteResult(result, completedExpression)

  if (formattedResult === null) {
    return
  }

  expressionValue.value = completedExpression
  displayValue.value = formattedResult
  addHistoryEntry(completedExpression, formattedResult)
  storedOperand.value = null
  pendingOperator.value = null
  shouldResetDisplay.value = true
}

function handleKeyboardInput(event) {
  if (currentPage.value !== 'calculator') {
    return
  }

  if (event.ctrlKey) {
    const memoryShortcut = event.key.toLowerCase()

    switch (memoryShortcut) {
      case 'l':
        clearMemory()
        break
      case 'm':
        storeMemory()
        break
      case 'p':
        addToMemory()
        break
      case 'q':
        subtractFromMemory()
        break
      case 'r':
        recallMemory()
        break
      default:
        return
    }

    event.preventDefault()
    return
  }

  if (/^[0-9]$/.test(event.key)) {
    inputDigit(event.key)
  } else {
    switch (event.key) {
      case '.':
      case ',':
        inputDecimal()
        break
      case '+':
        selectOperator('add')
        break
      case '-':
        selectOperator('subtract')
        break
      case '*':
        selectOperator('multiply')
        break
      case '/':
        selectOperator('divide')
        break
      case '=':
      case 'Enter':
        calculateResult()
        break
      case '%':
        applyPercentage()
        break
      case 'Backspace':
        deleteLastDigit()
        break
      case 'Delete':
        clearEntry()
        break
      case 'Escape':
        clearAll()
        break
      case 'F9':
        toggleSign()
        break
      case 'q':
      case 'Q':
        applySquare()
        break
      case '@':
        applySquareRoot()
        break
      case 'r':
      case 'R':
        applyReciprocal()
        break
      default:
        return
    }
  }

  event.preventDefault()
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyboardInput)
  systemThemeMediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
  updateSystemTheme(systemThemeMediaQuery)
  systemThemeMediaQuery.addEventListener('change', updateSystemTheme)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyboardInput)
  systemThemeMediaQuery?.removeEventListener('change', updateSystemTheme)
})
</script>

<template>
  <div class="calculator-window" :data-theme="resolvedTheme">
    <header class="title-bar">
      <div class="title-bar__identity">
        <button
          v-if="currentPage === 'settings'"
          class="title-bar__back"
          type="button"
          aria-label="返回"
          @click="showCalculator"
        >
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <path d="m10 5-7 7 7 7M3 12h18" />
          </svg>
        </button>
        <svg
          class="app-icon"
          viewBox="0 0 24 24"
          aria-hidden="true"
        >
          <rect x="3" y="1.5" width="18" height="21" rx="1.5" fill="#4f6675" />
          <rect x="5.5" y="4" width="13" height="5" rx="0.5" fill="#72c9db" />
          <g fill="#d9e1e5">
            <rect x="5.5" y="11" width="3" height="3" rx="0.4" />
            <rect x="10.5" y="11" width="3" height="3" rx="0.4" />
            <rect x="15.5" y="11" width="3" height="3" rx="0.4" />
            <rect x="5.5" y="16" width="3" height="3" rx="0.4" />
            <rect x="10.5" y="16" width="3" height="3" rx="0.4" />
            <rect x="15.5" y="16" width="3" height="3" rx="0.4" />
          </g>
        </svg>
        <span class="title-bar__title">计算器</span>
      </div>

      <div class="window-controls" aria-label="窗口控件">
        <span class="window-control" aria-label="最小化">
          <svg viewBox="0 0 16 16" aria-hidden="true">
            <path d="M3 8.5h10" />
          </svg>
        </span>
        <span class="window-control" aria-label="最大化">
          <svg viewBox="0 0 16 16" aria-hidden="true">
            <rect x="3.5" y="3.5" width="9" height="9" rx="0.8" />
          </svg>
        </span>
        <span class="window-control window-control--close" aria-label="关闭">
          <svg viewBox="0 0 16 16" aria-hidden="true">
            <path d="m3.5 3.5 9 9m0-9-9 9" />
          </svg>
        </span>
      </div>
    </header>

    <main class="app-content">
      <section v-if="currentPage === 'calculator'" class="calculator-page" aria-label="标准计算器">
        <header class="calculator-toolbar">
          <button class="toolbar-button" type="button" aria-label="打开导航菜单" @click="toggleNavigation">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M3 6.5h18M3 12h18M3 17.5h18" />
            </svg>
          </button>

          <h1>标准</h1>

          <button class="compact-overlay-button" type="button" aria-label="保持在最前面">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M4 9V5a3 3 0 0 1 3-3h11a3 3 0 0 1 3 3v13a3 3 0 0 1-3 3h-4" />
              <path d="M12 2v9h9M4 12h8v8M12 12l-8 8" />
            </svg>
          </button>

          <button
            class="toolbar-button history-button"
            type="button"
            aria-label="历史记录"
            :aria-expanded="isHistoryPanelOpen"
            @click="toggleHistoryPanel"
          >
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M4.4 7.1A9 9 0 1 1 3 12" />
              <path d="M3 4v5h5M12 7v5l3 2" />
            </svg>
          </button>
        </header>

        <div class="display-panel" aria-label="显示区">
          <div class="display-panel__expression" aria-hidden="true">{{ expressionValue || '\u00a0' }}</div>
          <output class="display-panel__value">{{ displayValue }}</output>
        </div>

        <div class="memory-bar" aria-label="Memory 按钮栏">
          <button type="button" :disabled="memoryValues.length === 0" @click="clearMemory">MC</button>
          <button type="button" :disabled="memoryValues.length === 0" @click="recallMemory">MR</button>
          <button type="button" @click="addToMemory">M+</button>
          <button type="button" @click="subtractFromMemory">M−</button>
          <button type="button" @click="storeMemory">MS</button>
          <button
            class="memory-list-button"
            type="button"
            :disabled="memoryValues.length === 0"
            :aria-expanded="isMemoryPanelOpen"
            @click="toggleMemoryPanel"
          >
            <span>M</span>
            <svg viewBox="0 0 12 8" aria-hidden="true">
              <path d="m1 1 5 5 5-5" />
            </svg>
          </button>
        </div>

        <div class="keypad" aria-label="计算器按键">
          <button type="button" @click="applyPercentage">%</button>
          <button type="button" @click="clearEntry">CE</button>
          <button type="button" @click="clearAll">C</button>
          <button type="button" aria-label="退格" @click="deleteLastDigit">
            <svg class="backspace-icon" viewBox="0 0 28 22" aria-hidden="true">
              <path d="M10 3h13.5A2.5 2.5 0 0 1 26 5.5v11a2.5 2.5 0 0 1-2.5 2.5H10l-8-8 8-8Z" />
              <path d="m14 8 6 6m0-6-6 6" />
            </svg>
          </button>

          <button class="function-key" type="button" aria-label="倒数" @click="applyReciprocal">
            <span><sup>1</sup>⁄<sub>x</sub></span>
          </button>
          <button class="function-key" type="button" aria-label="平方" @click="applySquare">
            <span>x<sup>2</sup></span>
          </button>
          <button class="function-key" type="button" aria-label="平方根" @click="applySquareRoot">
            <svg class="square-root-icon" viewBox="0 0 42 32" aria-hidden="true">
              <text class="square-root-icon__index" x="2" y="13">2</text>
              <path d="M9 15.5h4l3.5 11L22 4h18" />
              <text class="square-root-icon__variable" x="24" y="26">x</text>
            </svg>
          </button>
          <button type="button" @click="selectOperator('divide')">÷</button>

          <button type="button" @click="inputDigit('7')">7</button>
          <button type="button" @click="inputDigit('8')">8</button>
          <button type="button" @click="inputDigit('9')">9</button>
          <button type="button" @click="selectOperator('multiply')">×</button>

          <button type="button" @click="inputDigit('4')">4</button>
          <button type="button" @click="inputDigit('5')">5</button>
          <button type="button" @click="inputDigit('6')">6</button>
          <button type="button" @click="selectOperator('subtract')">−</button>

          <button type="button" @click="inputDigit('1')">1</button>
          <button type="button" @click="inputDigit('2')">2</button>
          <button type="button" @click="inputDigit('3')">3</button>
          <button type="button" @click="selectOperator('add')">＋</button>

          <button type="button" @click="toggleSign">＋/−</button>
          <button type="button" @click="inputDigit('0')">0</button>
          <button type="button" @click="inputDecimal">.</button>
          <button class="equals-key" type="button" @click="calculateResult">＝</button>
        </div>

        <aside v-if="isMemoryPanelOpen" class="memory-panel" aria-label="内存列表">
          <header class="memory-panel__header">
            <h2>内存</h2>
            <button type="button" aria-label="关闭内存列表" @click="isMemoryPanelOpen = false">×</button>
          </header>
          <div class="memory-panel__list">
            <button
              v-for="(memoryValue, index) in memoryValues"
              :key="`${memoryValue}-${index}`"
              type="button"
              :aria-label="`调用内存 ${formatResult(memoryValue)}`"
              @click="recallMemoryValue(memoryValue)"
            >
              {{ formatResult(memoryValue) }}
            </button>
          </div>
        </aside>

        <aside v-if="isHistoryPanelOpen" class="history-panel" aria-label="历史记录面板">
          <header class="history-panel__header">
            <h2>历史记录</h2>
            <button type="button" aria-label="关闭历史记录" @click="isHistoryPanelOpen = false">×</button>
          </header>

          <p v-if="historyEntries.length === 0" class="history-panel__empty">尚无历史记录</p>

          <div v-else class="history-panel__list">
            <button
              v-for="entry in historyEntries"
              :key="entry.id"
              type="button"
              :aria-label="`调用历史结果 ${entry.result}`"
              @click="recallHistoryResult(entry.result)"
            >
              <span>{{ entry.expression }}</span>
              <strong>{{ entry.result }}</strong>
            </button>
          </div>

          <button
            v-if="historyEntries.length > 0"
            class="history-panel__clear"
            type="button"
            aria-label="清除历史记录"
            @click="clearHistory"
          >
            清除
          </button>
        </aside>
      </section>

      <aside v-if="isNavigationOpen" class="navigation-pane" aria-label="计算器导航菜单">
        <div class="navigation-pane__header">
          <button class="navigation-menu-button" type="button" aria-label="关闭导航菜单" @click="toggleNavigation">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M3 6.5h18M3 12h18M3 17.5h18" />
            </svg>
          </button>
        </div>

        <div class="navigation-pane__body">
          <div class="navigation-section-label">计算器</div>

          <button
            class="navigation-item navigation-item--selected"
            type="button"
            aria-current="page"
            @click="showCalculator"
          >
            <svg class="navigation-item__icon navigation-calculator-icon" viewBox="0 0 28 28" aria-hidden="true">
              <rect x="4" y="2.5" width="20" height="23" rx="3" />
              <rect x="7.5" y="6" width="13" height="5" rx="0.8" />
              <circle cx="9" cy="15.5" r="1" />
              <circle cx="14" cy="15.5" r="1" />
              <circle cx="19" cy="15.5" r="1" />
              <circle cx="9" cy="20.5" r="1" />
              <circle cx="14" cy="20.5" r="1" />
              <circle cx="19" cy="20.5" r="1" />
            </svg>
            <span>标准</span>
          </button>
        </div>

        <div class="navigation-pane__footer">
          <button class="navigation-item navigation-settings-item" type="button" @click="showSettings">
            <svg class="navigation-item__icon" viewBox="0 0 28 28" aria-hidden="true">
              <path d="M11.5 3.5h5l.8 3a9 9 0 0 1 2.1 1.2l3-.8 2.5 4.3-2.2 2.2a8 8 0 0 1 0 2.4l2.2 2.2-2.5 4.3-3-.8a9 9 0 0 1-2.1 1.2l-.8 3h-5l-.8-3a9 9 0 0 1-2.1-1.2l-3 .8L3.1 18l2.2-2.2a8 8 0 0 1 0-2.4l-2.2-2.2L5.6 7l3 .8a9 9 0 0 1 2.1-1.2l.8-3.1Z" />
              <circle cx="14" cy="14.5" r="3.5" />
            </svg>
            <span>设置</span>
          </button>
        </div>
      </aside>

      <section v-if="currentPage === 'settings'" class="settings-page" aria-labelledby="settings-title">
        <h1 id="settings-title">设置</h1>

        <section class="settings-group" aria-labelledby="appearance-title">
          <h2 id="appearance-title">外观</h2>
          <button
            class="settings-card theme-card"
            type="button"
            aria-label="应用程序主题"
            :aria-expanded="isThemeOptionsOpen"
            @click="isThemeOptionsOpen = !isThemeOptionsOpen"
          >
            <div>
              <div class="settings-card__title">应用程序主题</div>
              <div class="settings-card__description">{{ selectedThemeLabel }}</div>
            </div>
            <svg class="settings-chevron" viewBox="0 0 16 10" aria-hidden="true">
              <path d="m1 1 7 7 7-7" />
            </svg>
          </button>

          <div v-if="isThemeOptionsOpen" class="theme-options" role="radiogroup" aria-label="主题选项">
            <button
              v-for="(label, theme) in themeLabels"
              :key="theme"
              type="button"
              role="radio"
              :aria-checked="selectedTheme === theme"
              @click="selectTheme(theme)"
            >
              {{ label }}
            </button>
          </div>
        </section>

        <section class="settings-group about-group" aria-labelledby="about-title">
          <h2 id="about-title">关于</h2>
          <div class="settings-card about-card">
            <div>
              <div class="settings-card__title">计算器</div>
              <div class="settings-card__description">© 2025 Microsoft。保留所有权利。</div>
              <div class="settings-version">11.2605.9.0</div>
            </div>
            <svg class="settings-chevron" viewBox="0 0 16 10" aria-hidden="true">
              <path d="m1 1 7 7 7-7" />
            </svg>
          </div>

          <div class="feedback-link">发送反馈</div>

          <p class="about-description">
            若要了解如何参与 Windows 计算器，请在
            <span class="github-link">GitHub</span>
            上查看该项目。
          </p>
        </section>
      </section>
    </main>
  </div>
</template>
