<script>
  import VueDraggable from 'vuedraggable'

  import BaseInteractiveLayout from '../common/BaseInteractiveLayout'

  import { putSession } from 'ozaria/site/api/interactive'
  import { getOzariaAssetUrl } from '../../../../common/ozariaUtils'

  import BaseButton from '../common/BaseButton'
  import ModalInteractive from '../common/ModalInteractive.vue'

  export default {
    components: {
      BaseButton,
      ModalInteractive,
      BaseInteractiveLayout,

      'draggable': VueDraggable
    },

    props: {
      interactive: {
        type: Object,
        required: true
      },

      localizedInteractiveConfig: {
        type: Object,
        required: true
      },

      interactiveSession: {
        type: Object,
        default: undefined
      },

      codeLanguage: {
        type: String,
        required: true
      }
    },

    data () {
      return {
        showModal: false,

        draggableGroup: Math.random().toString(),

        promptSlots: (this.localizedInteractiveConfig.elements || [])
          .map(({ elementId, ...rest }) => ({
            ...rest,
            id: elementId
          }))
      }
    },

    computed: {
      labels () {
        return (this.localizedInteractiveConfig.labels || []).map((label) => {
          if (typeof label === 'string') {
            return { text: label }
          }

          return label
        })
      },

      artUrl () {
        if (this.interactive.defaultArtAsset) {
          return getOzariaAssetUrl(this.interactive.defaultArtAsset)
        }

        return undefined
      },

      solution () {
        return this.promptSlots.map((s) => s.choiceId)
      },

      solutionCorrect () {
        for (let i = 0; i < this.solution.length; i++) {
          if (this.solution[i] !== this.localizedInteractiveConfig.solution[i]) {
            return false
          }
        }

        return true
      }
    },

    methods: {
      async submitSolution () {
        this.showModal = true

        // TODO save through vuex and block progress until save is successful
        // TODO fix submission format
        await putSession(this.interactive._id, {
          json: {
            codeLanguage: this.codeLanguage,
            submission: this.solution
          }
        })
      },

      closeModal () {
        if (this.solution.correct) {
          this.$emit('completed')
        } else {
          this.resetAnswer()
        }

        this.showModal = false
      },

      resetAnswer () {
        // TODO consolidate with initial state setting
        this.promptSlots = (this.localizedInteractiveConfig.elements || [])
          .map(({ elementId, ...rest }) => ({
            ...rest,
            id: elementId
          }))
      }
    }
  }
</script>

<template>
  <base-interactive-layout
    :interactive="interactive"
    :art-url="artUrl"
  >
    <div class="draggable-ordering-content">
      <div class="draggable-ordering-lists">
        <draggable
          :list="promptSlots"
          class="slots-container prompt-slots"
          ghost-class="ghost-slot"
          tag="ul"
          :force-fallback="true"
          fallback-class="dragging-slot"
        >
          <li
            v-for="prompt in promptSlots"
            :key="prompt.id"
            :class="{ 'prompt': true, 'monospaced': (prompt.textStyleCode === true) }"
          >
            {{ prompt.text }}
          </li>
        </draggable>

        <ul
          class="slots-container"
        >
          <li
            v-for="(label, index) in labels"
            :key="index"
            :class="{ 'prompt-label': true, 'monospaced': (label.textStyleCode === true) }"
          >
            {{ label.text }}
          </li>
        </ul>
      </div>

      <base-button
        class="submit"
        :on-click="submitSolution"
      >
        {{ $t('common.submit') }}
      </base-button>

      <modal-interactive
        v-if="showModal"
        @close="closeModal"
      >
        <template v-slot:body>
          <h1>{{ solutionCorrect ? "Did it!" : "Try Again!" }}</h1>
        </template>
      </modal-interactive>
    </div>
  </base-interactive-layout>
</template>

<style lang="scss" scoped>
  .draggable-ordering-content {
    padding: 20px;

    display: flex;
    flex-direction: column;

    .draggable-ordering-lists {
      flex-grow: 1;

      width: 100%;

      display: flex;
      flex-direction: row;

      align-items: center;
      justify-content: center;
    }

    height: 100%;
  }

  .submit {
    justify-content: flex-end;

    margin: 0px auto;
    margin-top: auto;
  }

  ul.slots-container {
    height: 100%;
    width: 50%;

    max-width: 500px;

    padding: 0;

    margin: 0;
    margin-right: 10px;

    display: flex;
    flex-direction: column;

    align-items: center;
    justify-content: space-evenly;

    li {
      margin-bottom: 15px;

      width: 100%;

      display: flex;

      justify-content: center;
      align-items: center;

      text-align: center;
      font-size: 15px;

      min-height: 50px;
    }

    li.prompt {
      border: 2px solid #acb9fa;
    }

    li.prompt-label {
      background-color: #acb9fa;
      border: 2px solid #acb9fa;
    }

    li.monospaced {
      font-family: monospace;
    }

    li.dragging-slot {
      // TODO this doesn't work because vue-draggable also uses transforms for positioing
      transform: rotate(5deg);
    }
  }
</style>
