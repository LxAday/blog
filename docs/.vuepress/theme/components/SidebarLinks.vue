<template>
  <ul
    class="sidebar-links"
    v-if="items.length"
  >
    <li v-for="(item, i) in items" :key="i">
      <SidebarGroup
        v-if="item.type === 'group'"
        :item="item"
        :open="i === openGroupIndex"
        :collapsable="item.collapsable || item.collapsible"
        :depth="depth"
        @toggle="toggleGroup(i)"
      />
      <SidebarLink
        v-else
        :sidebarDepth="sidebarDepth"
        :item="item"
      />
    </li>
  </ul>
</template>

<script>
  import SidebarGroup from '@theme/components/SidebarGroup'
  import SidebarLink from '@theme/components/SidebarLink'
  import {isActive} from '@theme/helpers/utils'

  export default {
    name: 'SidebarLinks',

    components: {SidebarGroup, SidebarLink},

    props: [
      'items',
      'depth', // depth of current sidebar links
      'sidebarDepth' // depth of headers to be extracted
    ],

    data() {
      return {
        openGroupIndex: 0,
      }
    },

    created() {
      this.refreshIndex()
    },

    mounted() {
      this.activationLink()
      this.isInViewPortOfOne()
    },

    watch: {
      '$route'() {
        this.refreshIndex()
      }
    },

    updated: function () {
      this.isInViewPortOfOne()
    },

    methods: {
      activationLink() {
        let subtitleName = decodeURIComponent(this.$route.fullPath)
        if (!subtitleName || subtitleName == '') return

        const subtitles = [].slice.call(document.querySelectorAll(AHL_SIDEBAR_LINK_SELECTOR))
        for (let i = 0; i < subtitles.length; i++) {
          if (decodeURIComponent(subtitles[i].getAttribute('href')).indexOf(subtitleName) != -1) {
            subtitles[i].click()
            this.activationAnchor()
            return
          }
        }
      },

      activationAnchor() {
        let anchors = [].slice.call(document.querySelectorAll(AHL_HEADER_ANCHOR_SELECTOR))
          .filter(anchor => decodeURIComponent(this.$route.hash) == decodeURIComponent(anchor.hash))
        if (anchors == null || anchors.length < 1 || anchors[0].offsetTop == undefined) return

        setTimeout(function () {
          window.scrollTo(0, anchors[0].offsetTop + 170)
        }, 300)
      },

      isInViewPortOfOne() {
        let siderbarScroll = document.getElementsByClassName("sidebar")[0]
        let el = document.getElementsByClassName("active sidebar-link")[1]
        if (el == null || el == undefined || el.offsetTop == undefined) {
          el = document.getElementsByClassName("active sidebar-link")[0]
        }
        if (el == null || el == undefined || el.offsetTop == undefined) return

        const viewPortHeight = siderbarScroll.clientHeight || window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight
        let offsetTop = el.offsetTop
        let offsetBottom = el.offsetTop + el.offsetHeight
        let scrollTop = siderbarScroll.scrollTop
        let bottomVisible = (offsetBottom <= viewPortHeight + scrollTop)
        if (!bottomVisible) {
          siderbarScroll.scrollTop = (offsetBottom + 5 - viewPortHeight)
        }
        let topVisible = (offsetTop >= scrollTop)
        if (!topVisible) {
          siderbarScroll.scrollTop = (offsetTop - 5)
        }
      },

      refreshIndex() {
        const index = resolveOpenGroupIndex(
          this.$route,
          this.items
        )
        if (index > -1) {
          this.openGroupIndex = index
        }
      },

      toggleGroup(index) {
        this.openGroupIndex = index === this.openGroupIndex ? -1 : index
      },

      isActive(page) {
        return isActive(this.$route, page.regularPath)
      }
    }
  }

  function resolveOpenGroupIndex(route, items) {
    for (let i = 0; i < items.length; i++) {
      const item = items[i]
      if (item.type === 'group' && item.children.some(c => c.type === 'page' && isActive(route, c.path))) {
        return i
      }
    }
    return -1
  }
</script>
