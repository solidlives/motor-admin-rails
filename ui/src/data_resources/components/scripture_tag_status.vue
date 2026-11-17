<template>
  <div
    v-if="results && results.length"
    class="scripture-tag-status"
  >
    <div
      v-for="(result, index) in results"
      :key="index"
      class="scripture-tag-status-row"
    >
      <Tag :color="statusColor(result.status)">
        {{ statusLabel(result.status) }}
      </Tag>
      <code>{{ result.raw_tag }}</code>
      <span class="scripture-tag-status-message">{{ messageFor(result) }}</span>
    </div>
  </div>
  <div
    v-else-if="errorMessage"
    class="scripture-tag-status-error"
  >
    {{ errorMessage }}
  </div>
</template>

<script>
import axios from 'axios'
import { basePath } from 'utils/scripts/configs'

const STATUS_LABELS = { ready: 'Ready', pending: 'Pending', error: 'Error' }
const STATUS_COLORS = { ready: 'success', pending: 'primary', error: 'error' }
const STATUS_MESSAGES = {
  ready: 'Verse text is saved — members see the full passage in the app',
  pending: 'Fetching from YouVersion — members see only the reference until it\'s retrieved'
}

export default {
  name: 'ScriptureTagStatus',
  props: {
    resource: {
      type: Object,
      required: true
    },
    column: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      results: null,
      errorMessage: null
    }
  },
  computed: {
    content () {
      return this.resource[this.column.name] || ''
    }
  },
  watch: {
    content () {
      this.loadStatus()
    }
  },
  mounted () {
    this.loadStatus()
  },
  methods: {
    statusLabel (status) {
      return STATUS_LABELS[status] || status
    },
    statusColor (status) {
      return STATUS_COLORS[status] || 'default'
    },
    messageFor (result) {
      return STATUS_MESSAGES[result.status] || result.message
    },
    loadStatus () {
      this.errorMessage = null

      // No tags, no request.
      if (!this.content.includes('[[verse')) {
        this.results = null

        return
      }

      axios.post(basePath + 'scripture_tags/preview', {
        content: this.content,
        sectionable_type: this.resource.sectionable_type,
        sectionable_id: this.resource.sectionable_id
      }).then((result) => {
        this.results = result.data.tags
      }).catch((error) => {
        this.results = null
        this.errorMessage = error.response?.data?.errors?.join(' ') || error.message
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.scripture-tag-status {
  margin-top: 8px;
  line-height: 1.6;
}

.scripture-tag-status-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 4px;
}

.scripture-tag-status-message {
  color: #808695;
  font-size: 12px;
}

.scripture-tag-status-error {
  margin-top: 8px;
  color: #ed4014;
  font-size: 12px;
}
</style>
