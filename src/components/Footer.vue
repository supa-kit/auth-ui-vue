<template>
  <footer
    class="mt-16 pb-4 w-full flex justify-center items-center text-foreground"
    text="slate-50 dark:slate-300 opacity-60 sm"
  >
    <div class="copyright flex flex-col justify-center items-center">
      <p class="text-neutral-500 dark:text-neutral-400">
        Code with ❤ & ☕️ by
        <a class="text-neon" href="https://github.com/xiaoluoboding">
          @xiaoluoboding
        </a>
        <span> © {{ new Date().getFullYear() }}</span>
      </p>
      <div
        class="flex flex-col sm:flex-row items-center space-x-1 space-y-2 sm:space-y-0 mt-2"
      >
        <div class="flex items-center space-x-1">
          <carbon:logo-twitter class="text-blue-300" />
          <span>
            <a
              href="https://twitter.com/robert_shaw_x"
              class="text-neon"
              target="_blank"
            >
              Follow me on Twitter
            </a>
          </span>
        </div>
        <span class="px-2 text-emerald-300 hidden sm:block">|</span>
        <div class="flex items-center space-x-1">
          <carbon:cafe class="text-amber-700" />
          <span>
            <a
              href="https://www.buymeacoffee.com/xlbd"
              target="_blank"
              class="text-neon"
            >
              Buy me a coffee
            </a>
          </span>
        </div>
        <span class="px-2 text-emerald-300 hidden sm:block">|</span>
        <div class="flex items-center space-x-1">
          <mdi:heart class="text-rose-500" />
          <span>
            <a
              href="https://github.com/sponsors/xiaoluoboding"
              target="_blank"
              class="text-neon"
            >
              Sponsor me on GitHub
            </a>
          </span>
        </div>
      </div>
    </div>
  </footer>
</template>
