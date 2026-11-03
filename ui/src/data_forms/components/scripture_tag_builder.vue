<template>
  <div class="scripture-tag-builder">
    <Spin
      v-if="isLoading"
      fix
    />
    <p
      v-if="errorMessage"
      class="scripture-tag-error"
    >
      {{ errorMessage }}
    </p>
    <VForm label-position="top">
      <FormItem
        v-if="needsLanguage"
        label="Language code"
      >
        <VInput
          v-model="languageCode"
          placeholder="en"
          @on-enter="loadOptions"
          @on-blur="loadOptions"
        />
      </FormItem>
      <FormItem label="Version">
        <VSelect
          v-model="version"
          clearable
          placeholder="Network / language default"
        >
          <VOption
            v-for="option in versions"
            :key="option.abbreviation"
            :value="option.abbreviation"
            :label="versionLabel(option)"
          />
        </VSelect>
      </FormItem>
      <FormItem label="Book">
        <VSelect
          v-model="book"
          filterable
          placeholder="Select a book"
        >
          <VOption
            v-for="option in books"
            :key="option.code"
            :value="option.names[0]"
            :label="option.names.slice(0, 2).join(' / ')"
          />
        </VSelect>
      </FormItem>
      <div class="scripture-tag-numbers">
        <FormItem label="Chapter">
          <InputNumber
            v-model="chapter"
            :min="1"
          />
        </FormItem>
        <FormItem label="From verse">
          <InputNumber
            v-model="verseFrom"
            :min="1"
          />
        </FormItem>
        <FormItem label="To verse">
          <InputNumber
            v-model="verseTo"
            :min="verseFrom || 1"
          />
        </FormItem>
      </div>
    </VForm>
    <p class="scripture-tag-preview">
      <code v-if="tag">{{ tag }}</code>
      <span v-else>Pick a book and chapter to build a tag.</span>
    </p>
    <div class="scripture-tag-actions">
      <VButton
        :disabled="!tag"
        @click="copyTag"
      >
        Copy
      </VButton>
      <VButton
        type="primary"
        :disabled="!tag"
        @click="onInsert(tag)"
      >
        Insert into content
      </VButton>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { basePath } from 'utils/scripts/configs'

export default {
  name: 'ScriptureTagBuilder',
  props: {
    sectionableParams: {
      type: Object,
      required: false,
      default: () => ({})
    },
    onInsert: {
      type: Function,
      required: true
    }
  },
  data () {
    return {
      isLoading: false,
      errorMessage: null,
      needsLanguage: false,
      languageCode: '',
      versions: [],
      books: [],
      version: null,
      book: null,
      chapter: null,
      verseFrom: null,
      verseTo: null
    }
  },
  computed: {
    reference () {
      if (!this.book || !this.chapter) return null

      let reference = `${this.book} ${this.chapter}`

      if (this.verseFrom) {
        reference += `:${this.verseFrom}`

        if (this.verseTo && this.verseTo > this.verseFrom) {
          reference += `-${this.verseTo}`
        }
      }

      return reference
    },
    tag () {
      if (!this.reference) return null

      return this.version ? `[[verse ${this.reference} | ${this.version}]]` : `[[verse ${this.reference}]]`
    }
  },
  mounted () {
    this.loadOptions()
  },
  methods: {
    versionLabel (option) {
      return `${option.abbreviation} — ${option.title}${option.default ? ' (default)' : ''}`
    },
    loadOptions () {
      this.isLoading = true
      this.errorMessage = null

      const params = this.languageCode
        ? { language_code: this.languageCode }
        : this.sectionableParams

      axios.get(basePath + 'scripture_tags/options', { params }).then((result) => {
        this.versions = result.data.versions
        this.books = result.data.books
        this.languageCode = result.data.language_code
        if (!this.versions.some((option) => option.abbreviation === this.version)) {
          this.version = null
        }
      }).catch((error) => {
        if (error.response?.status === 422) {
          this.needsLanguage = true
        } else {
          this.errorMessage = error.response?.data?.errors?.join(' ') || error.message
        }
      }).finally(() => {
        this.isLoading = false
      })
    },
    copyTag () {
      navigator.clipboard.writeText(this.tag).then(() => {
        this.$Message.info('Tag copied')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.scripture-tag-builder {
  position: relative;
}

.scripture-tag-numbers {
  display: flex;
  gap: 12px;
}

.scripture-tag-preview {
  margin: 4px 0 12px;
}

.scripture-tag-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}

.scripture-tag-error {
  color: #ed4014;
  margin-bottom: 8px;
}
</style>
