<script setup>
import { ref } from 'vue'

const displayValue = ref('0')

function inputDigit(digit) {
  displayValue.value = displayValue.value === '0' ? digit : `${displayValue.value}${digit}`
}

function inputDecimal() {
  if (!displayValue.value.includes('.')) {
    displayValue.value += '.'
  }
}

function clearDisplay() {
  displayValue.value = '0'
}
</script>

<template>
  <div class="calculator-window">
    <header class="title-bar">
      <div class="title-bar__identity">
        <button class="title-bar__back" type="button" aria-label="返回" hidden>
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
      <section class="calculator-page" aria-label="标准计算器">
        <header class="calculator-toolbar">
          <button class="toolbar-button" type="button" aria-label="打开导航菜单">
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

          <button class="toolbar-button history-button" type="button" aria-label="历史记录">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M4.4 7.1A9 9 0 1 1 3 12" />
              <path d="M3 4v5h5M12 7v5l3 2" />
            </svg>
          </button>
        </header>

        <div class="display-panel" aria-label="显示区">
          <div class="display-panel__expression" aria-hidden="true">&nbsp;</div>
          <output class="display-panel__value">{{ displayValue }}</output>
        </div>

        <div class="memory-bar" aria-label="Memory 按钮栏">
          <button type="button" disabled>MC</button>
          <button type="button" disabled>MR</button>
          <button type="button">M+</button>
          <button type="button">M−</button>
          <button type="button">MS</button>
          <button class="memory-list-button" type="button" disabled>
            <span>M</span>
            <svg viewBox="0 0 12 8" aria-hidden="true">
              <path d="m1 1 5 5 5-5" />
            </svg>
          </button>
        </div>

        <div class="keypad" aria-label="计算器按键">
          <button type="button">%</button>
          <button type="button" @click="clearDisplay">CE</button>
          <button type="button" @click="clearDisplay">C</button>
          <button type="button" aria-label="退格">
            <svg class="backspace-icon" viewBox="0 0 28 22" aria-hidden="true">
              <path d="M10 3h13.5A2.5 2.5 0 0 1 26 5.5v11a2.5 2.5 0 0 1-2.5 2.5H10l-8-8 8-8Z" />
              <path d="m14 8 6 6m0-6-6 6" />
            </svg>
          </button>

          <button class="function-key" type="button"><span><sup>1</sup>⁄<sub>x</sub></span></button>
          <button class="function-key" type="button"><span>x<sup>2</sup></span></button>
          <button class="function-key" type="button" aria-label="平方根">
            <svg class="square-root-icon" viewBox="0 0 42 32" aria-hidden="true">
              <text class="square-root-icon__index" x="2" y="13">2</text>
              <path d="M9 15.5h4l3.5 11L22 4h18" />
              <text class="square-root-icon__variable" x="24" y="26">x</text>
            </svg>
          </button>
          <button type="button">÷</button>

          <button type="button" @click="inputDigit('7')">7</button>
          <button type="button" @click="inputDigit('8')">8</button>
          <button type="button" @click="inputDigit('9')">9</button>
          <button type="button">×</button>

          <button type="button" @click="inputDigit('4')">4</button>
          <button type="button" @click="inputDigit('5')">5</button>
          <button type="button" @click="inputDigit('6')">6</button>
          <button type="button">−</button>

          <button type="button" @click="inputDigit('1')">1</button>
          <button type="button" @click="inputDigit('2')">2</button>
          <button type="button" @click="inputDigit('3')">3</button>
          <button type="button">＋</button>

          <button type="button">＋/−</button>
          <button type="button" @click="inputDigit('0')">0</button>
          <button type="button" @click="inputDecimal">.</button>
          <button class="equals-key" type="button">＝</button>
        </div>
      </section>

      <aside class="navigation-pane" aria-label="计算器导航菜单" hidden>
        <div class="navigation-pane__header">
          <button class="navigation-menu-button" type="button" aria-label="关闭导航菜单">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M3 6.5h18M3 12h18M3 17.5h18" />
            </svg>
          </button>
        </div>

        <div class="navigation-pane__body">
          <div class="navigation-section-label">计算器</div>

          <button class="navigation-item navigation-item--selected" type="button" aria-current="page">
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
          <button class="navigation-item navigation-settings-item" type="button">
            <svg class="navigation-item__icon" viewBox="0 0 28 28" aria-hidden="true">
              <path d="M11.5 3.5h5l.8 3a9 9 0 0 1 2.1 1.2l3-.8 2.5 4.3-2.2 2.2a8 8 0 0 1 0 2.4l2.2 2.2-2.5 4.3-3-.8a9 9 0 0 1-2.1 1.2l-.8 3h-5l-.8-3a9 9 0 0 1-2.1-1.2l-3 .8L3.1 18l2.2-2.2a8 8 0 0 1 0-2.4l-2.2-2.2L5.6 7l3 .8a9 9 0 0 1 2.1-1.2l.8-3.1Z" />
              <circle cx="14" cy="14.5" r="3.5" />
            </svg>
            <span>设置</span>
          </button>
        </div>
      </aside>

      <section class="settings-page" aria-labelledby="settings-title" hidden>
        <h1 id="settings-title">设置</h1>

        <section class="settings-group" aria-labelledby="appearance-title">
          <h2 id="appearance-title">外观</h2>
          <div class="settings-card theme-card">
            <div>
              <div class="settings-card__title">应用程序主题</div>
              <div class="settings-card__description">选择要显示的应用主题</div>
            </div>
            <svg class="settings-chevron" viewBox="0 0 16 10" aria-hidden="true">
              <path d="m1 1 7 7 7-7" />
            </svg>
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
