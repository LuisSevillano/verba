<!-- eslint-disable vue/no-v-html -->
<template>
  <div
    :class="{ modal: true, show: currentResult }"
    tabindex="-1"
    role="dialog"
  >
    <div class="modal-dialog modal-lg" role="document">
      <div v-if="currentResult" class="modal-content">
        <div class="modal-header">
          <!--  eslint-disable-next-line vue/no-v-html -->
          <p class="modal-title" v-html="title"></p>
          <button
            type="button"
            class="close"
            aria-label="Close"
            @click="onClose"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <div v-if="resultContext">
            <p v-for="(item, key) in resultContext" :key="key">
              {{ item.time_start | formatTime }}:
              <span v-html="highlight(item.content)"></span>
            </p>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-background" @click="onClose"></div>
  </div>
</template>

<script>
import Vue from 'vue'
import moment from 'moment'
import { mapActions, mapState } from 'vuex'

export default {
  name: 'ResultContext',
  data() {
    return {
      resultContext: null
    }
  },
  computed: {
    ...mapState(['currentResult', 'queryTerms']),
    title: function() {
      const date = this.currentResult.programme.date
      return `<strong>${moment(date).format(
        'DD/MM/YYYY'
      )}</strong> | TD ${date.substring(
        11,
        13
      )}h <small>(${this.$options.filters.formatTime(
        this.currentResult.time_start
      )})</small>`
    }
  },
  watch: {
    currentResult: function() {
      if (this.currentResult) {
        document.body.classList.add('modal-open')
        this.fetchResultContext()
      } else {
        document.body.classList.remove('modal-open')
      }
    }
  },
  methods: {
    ...mapActions(['setCurrentResult']),
    highlight(content) {
      this.queryTerms.forEach(term => {
        const re = new RegExp('\\b' + term + '\\b', 'gi')
        content = content.replace(re, match => `<mark>${match}</mark>`)
      })
      return content
    },
    onClose: function() {
      this.setCurrentResult(null)
    },
    fetchResultContext() {
      const params = {
        programme_id: this.currentResult.programme.id,
        start_time: this.currentResult.time_start,
        range: 60 // 1 minute context
      }
      Vue.verbaAPI('fetchContext', params, response => {
        this.resultContext = response.data
      })
    }
  }
}
</script>

<style lang="scss">
@import '../scss/_variables.scss';

.modal.show {
  display: block;
}
.modal-dialog {
  z-index: 2;
}
.modal-body {
  padding: 2rem;
  text-align: left;
}
.modal-content {
  border-radius: 0;
  top: 50px;
}
.modal-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #000;
  opacity: 0.5;
  z-index: 1;
  cursor: pointer;
}
mark {
  font-weight: 600;
  background-color: mix($color-highlight-1, white, 85%);
  padding-top: 0;
  padding-bottom: 0;
}
</style>
