<script>
  import { getInteractive, getSession } from '../../../api/interactive'
  import draggableOrderingComponent from './draggableOrdering/index'
  import insertCodeComponent from './insertCode'
  import draggableStatementCompletionComponent from './draggableStatementCompletion'

  module.exports = Vue.extend({
    components: {
      'draggable-ordering': draggableOrderingComponent,
      'insert-code': insertCodeComponent,
      'draggable-statement-completion': draggableStatementCompletionComponent
    },

    props: {
      interactiveIdOrSlug: {
        type: String,
        required: true,
        default: ''
      },

      introLevelId: {
        type: String,
        required: true,
        default: ''
      },

      courseInstanceId: {
        type: String,
        default: undefined
      }
    },

    data: () => ({
      loading: false,
      interactive: {},
      interactiveSession: {},
      interactiveType: ''
    }),

    computed: {
      interactiveComponent () {
        switch (this.interactive.interactiveType) {
        case 'draggable-statement-completion':
          return draggableStatementCompletionComponent

        case 'insert-code':
          return insertCodeComponent

        case 'draggable-ordering':
          return draggableOrderingComponent

        default:
          noty({ text: 'Interactive type is not set for the interactive', type: 'error', timeout: '2000' })
          throw new Error('Invalid interactive type provided for interactive')
        }
      }
    },

    async created () {
      this.loading = true

      if (!me.hasInteractiveAccess()) {
        alert('You must be logged in as an admin to use this page.')
        return application.router.navigate('/', { trigger: true })
      }

      try {
        this.interactive = await getInteractive(this.interactiveIdOrSlug)
        this.interactiveType = this.interactive.interactiveType
        if (!this.interactiveType) {
          console.error('Interactive type is not set for the interactive', this.interactiveIdOrSlug)
          noty({ text: 'Interactive type is not set for the interactive', type: 'error', timeout: '2000' })
          return
        }

        const getSessionOptions = {
          introLevelId: this.introLevelId,
          courseInstanceId: this.courseInstanceId
        }

        this.interactiveSession = await getSession(this.interactiveIdOrSlug, getSessionOptions)
      } catch (err) {
        console.error('Error:', err)
        noty({ text: 'Error occured in getting interactives data.', type: 'error', timeout: '2000' })
      } finally {
        this.loading = false
      }
    },

    methods: {
      onCompleted () {
        this.$emit('completed')
      }
    }
  })
</script>

<template>
  <div class="interactive-container">
    <h1 v-if="loading">
      LOADING
    </h1>
    <component
      :is="interactiveComponent"
      v-else
      :interactive="interactive"
      :intro-level-id="introLevelId"
      :interactive-session="interactiveSession"
      :course-instance-id="courseInstanceId"
      @completed="onCompleted"
    />
  </div>
</template>

<style scoped>

  .interactive-container {
    background-color: #FFF;
  }
</style>
