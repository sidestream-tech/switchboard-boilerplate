<script setup lang="ts">
import scrollIntoView from 'scroll-into-view-if-needed'

const { data: documentation } = await useAsyncData('blogToc', () =>
  queryContent('/documentation').findOne()
)

const currentlyActiveId = ref(null as string | null)
const content = ref<HTMLDivElement>()
const tocLinks = computed(() => documentation.value?.body.toc.links ?? [])

const scrollToId = function (id: string) {
  const navigationElement = document.querySelector(`nav [href="#${id}"]`)
  if (navigationElement) {
    scrollIntoView(navigationElement, {
      scrollMode: 'if-needed',
      behavior: 'smooth',
      block: 'center',
      inline: 'start'
    })
  }
}

const trackCurrentlyActiveId = function () {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const id = entry.target.getAttribute('id')
        if (id) {
          currentlyActiveId.value = id
          scrollToId(id)
        }
      }
    })
  }, {
    root: document,
    rootMargin: '0px 0px -90%',
    threshold: 0
  })
  document
    .querySelectorAll('.markdown-body h2[id], .markdown-body h3[id]')
    .forEach((section) => {
      observer.observe(section)
    })
}

onMounted(trackCurrentlyActiveId)

useHead({
  title: 'Makerdao Document Model Documentation'
})
</script>

<template>
  <div ref="content" class="w-full max-w-screen-2xl m-auto">
    <div class="flex flex-row w-full px-4">
      <div class="flex-shrink-0 w-56 py-3 hidden md:block h-screen overflow-x-scroll sticky top-14">
        <DocTableOfContents :toc-links="tocLinks" :currently-active-id="currentlyActiveId" class="pb-14" />
      </div>
      <div class="flex-grow !w-[calc(100%-18rem)] p-4 pt-2 mt-4 bg-white w-full">
        <ContentRendererMarkdown v-if="documentation" :value="documentation" class="markdown-body w-full" />
      </div>
    </div>
  </div>
</template>
