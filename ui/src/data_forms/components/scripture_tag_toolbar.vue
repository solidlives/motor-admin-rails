<template>
  <div class="scripture-tag-toolbar">
    <p class="scripture-tag-help">
      Insert a scripture tag like <code>[[verse John 3:10-16]]</code>
      (or <code>[[verse John 3:16 | KJV]]</code> to pin a version) on its own line in the content.
      Members will see the full passage text there. The tag itself stays in the editor —
      the verse text is filled in automatically in the app.
    </p>
    <div class="scripture-tag-actions">
      <VButton
        size="small"
        icon="ios-book-outline"
        @click="openBuilder"
      >
        Build a tag
      </VButton>
      <VButton
        size="small"
        icon="ios-checkmark-circle-outline"
        :loading="isChecking"
        @click="checkTags"
      >
        Check tags
      </VButton>
    </div>
    <p
      v-if="errorMessage"
      class="scripture-tag-error"
    >
      {{ errorMessage }}
    </p>
    <ul
      v-if="results"
      class="scripture-tag-results"
    >
      <li v-if="!results.length">
        No scripture tags found in the content.
      </li>
      <li
        v-for="(result, index) in results"
        :key="index"
      >
        <Tag :color="statusColor(result.status)">
          {{ statusLabel(result.status) }}
        </Tag>
        <code>{{ result.raw_tag }}</code>
        <span v-if="result.status !== 'ready'"> — {{ result.message }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'
import { basePath } from 'utils/scripts/configs'
import ScriptureTagBuilder from './scripture_tag_builder'

const STATUS_LABELS = { ready: 'Ready', pending: 'Will retrieve on save', error: 'Error' }
const STATUS_COLORS = { ready: 'success', pending: 'primary', error: 'error' }

export default {
  name: 'ScriptureTagToolbar',
  props: {
    content: {
      type: String,
      required: false,
      default: ''
    },
    formData: {
      type: Object,
      required: false,
      default: null
    }
  },
  emits: ['insert'],
  data () {
    return {
      isChecking: false,
      results: null,
      errorMessage: null
    }
  },
  computed: {
    sectionableParams () {
      return {
        sectionable_type: this.formData?.sectionable_type,
        sectionable_id: this.formData?.sectionable_id
      }
    }
  },
  methods: {
    statusLabel (status) {
      return STATUS_LABELS[status] || status
    },
    statusColor (status) {
      return STATUS_COLORS[status] || 'default'
    },
    errorFrom (error) {
      return error.response?.data?.errors?.join(' ') || error.message
    },
    checkTags () {
      this.isChecking = true
      this.errorMessage = null

      axios.post(basePath + 'scripture_tags/preview', {
        content: this.content || '',
        ...this.sectionableParams
      }).then((result) => {
        this.results = result.data.tags
      }).catch((error) => {
        this.errorMessage = this.errorFrom(error)
      }).finally(() => {
        this.isChecking = false
      })
    },
    openBuilder () {
      this.$Modal.open(ScriptureTagBuilder, {
        sectionableParams: this.sectionableParams,
        onInsert: (tag) => {
          this.$emit('insert', tag)
          this.$Modal.remove()
        }
      }, {
        title: 'Build a scripture tag',
        closable: true
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.scripture-tag-toolbar {
  margin-top: 6px;
  line-height: 1.5;
}

.scripture-tag-help {
  color: #808695;
  font-size: 12px;
  margin-bottom: 6px;
}

.scripture-tag-actions {
  display: flex;
  gap: 8px;
}

.scripture-tag-error {
  color: #ed4014;
  margin-top: 6px;
}

.scripture-tag-results {
  list-style: none;
  margin: 8px 0 0;
  padding: 0;

  li {
    margin-bottom: 4px;
  }
}
</style>
