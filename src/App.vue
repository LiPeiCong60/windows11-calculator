<template>
  <div class="calculator-window">
    <header class="title-bar">
      <div class="title-bar__identity">
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

    <main class="app-content" aria-label="计算器内容区域"></main>
  </div>
</template>
