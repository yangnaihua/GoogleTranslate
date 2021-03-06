<template>
  <Layout @close="view.animationEnd && $router.push('/')">
    <header ref="header" style="transform: translateY(-40px) translateZ(0)">
      <form action method="post" @submit.prevent>
        <div class="search-box">
          <vInput ref="search" type="search" placeholder="搜索语言" v-model="language" @input.native="searchHandler" />
        </div>
      </form>
    </header>
    <div class="languages" style="transform: translateY(40px) translateZ(0)">
      <LanguageList v-if="!isSearch" title="最近使用">
        <LanguageItem 
          v-for="(value, key, index) in recent" 
          :key="key" 
          :country="key" 
          :text="value" 
          :active="key === query.active" 
          @click.native="changeLanguageHandler(key, value, index)" 
        />
      </LanguageList>
      <LanguageList ref="list" :title="isSearch ? `${Object.keys(country).length} 个语言` : '所有语言'">
        <LanguageItem 
          v-for="(value, key, index) in country" 
          :key="key" 
          :country="key" 
          :text="value" 
          :active="key === query.active" 
          @click.native="changeLanguageHandler(key, value, index)" 
        />
      </LanguageList>
    </div>
  </Layout>
</template>
<script>
import anime from 'animejs'
import rawCountry from '../assets/json/languages.js'
import Layout from '@/views/_Layout'
import vInput from '@/components/Input'
import LanguageList from '@/components/LanguageList'
import LanguageItem from '@/components/LanguageItem'
import { WindowHelper, Thread } from '../utils'
import { SAVE_STATE } from '../store/types'
let country = { ...rawCountry }
export default {
  name: 'change-language-page',
  components: { Layout, vInput, LanguageList, LanguageItem },
  data () {
    return {
      query: Object.keys(this.$route.query).length > 0 ? this.$route.query : { from: 'source', active: 'auto' },
      country,
      language: '',
      view: {
        height: 530,
        animeOptions: { duration: 150, easing: 'easeOutQuart' },
        animationEnd: false
      }
    }
  },
  created () {
    if (this.query.from === 'target') delete country.auto
    else this.country = country = { ...rawCountry }
  },
  mounted () {
    WindowHelper.setSize(window.innerWidth, this.view.height, this.view.animeOptions)
    anime({
      targets: [this.$refs.header, '.languages'],
      translateY: 0,
      translateZ: 0,
      complete: () => (this.view.animationEnd = true)
    })
  },
  computed: {
    isSearch () {
      return Object.keys(this.country).length !== Object.keys(country).length
    },
    recent () {
      const recent = { ...this.$store.getters.state.recent }
      if (this.query.from === 'target') delete recent.auto
      return recent
    }
  },
  methods: {
    async searchHandler () {
      await Thread.sleep()
      if (this.$refs.search.isComposition) return // 输入法未上屏
      if (this.language) {
        const result = {}
        Object.entries(country).filter(x => x[1].includes(this.language)).forEach(x => (result[x[0]] = x[1]))
        this.country = result
        await this.$nextTick()
        const outerHeight = el => {
          const style = getComputedStyle(el)
          const height = el.offsetHeight
          const marginTop = Number.parseInt(style.marginTop)
          const marginBottom = Number.parseInt(style.marginBottom)
          return height + Number.parseInt(marginTop + marginBottom)
        }
        const headers = [...document.querySelectorAll('header')]
        const headerHeight = headers.map(x => outerHeight(x)).reduce((prev, next) => prev + next)
        const listHeight = this.$refs.list.$el.offsetHeight
        WindowHelper.setSize(window.innerWidth, headerHeight + listHeight + 19 + 5, this.view.animeOptions)
      } else {
        this.country = country
        WindowHelper.setSize(window.innerWidth, this.view.height, this.view.animeOptions)
      }
    },
    changeLanguageHandler (lang, name) {
      this.$router.push({ path: '/', query: { lang, action: this.query.from } })
      this.$store.commit(SAVE_STATE, {
        ...this.$store.getters.state,
        recent: {
          ...this.recent,
          ...{ [lang]: name }
        }
      })
    }
  }
}
</script>
<style lang="stylus" scoped>
main
  > header
    background #f7fafb
    border-bottom 1px solid #d7dce0
    margin-bottom 10px
    width calc(100% - 2px)
    .search-box
      position relative
      padding 10px 20px
      &::before
        content '\e60d'
        font-family icon
        font-size 22px
        color #aaaeb3
        position absolute
        top 50%
        left 30px
        transform translate3d(0, -50%, 0)
      input[type="search"]
        padding-left 40px
  .languages
    flex 1
    overflow scroll
    box-sizing border-box
</style>
