<script>
  import VueDraggable from 'vuedraggable'

  import BaseInteractiveTitle from '../common/BaseInteractiveTitle'
  import { getOzariaAssetUrl } from '../../../../common/ozariaUtils'

  export default {
    components: {
      'base-interactive-title': BaseInteractiveTitle,
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
      }
    }
  }
</script>

<template>
  <div class="draggable-ordering-container">
    <base-interactive-title
      :interactive="interactive"
    />

    <div class="prompt-row">
      <div class="prompt">
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

      <div
        v-if="artUrl"
        class="art-container"
      >
        <img
          :src="artUrl"
          alt="Art!"
        >
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .draggable-ordering-container {

    display: flex;
    flex-direction: column;

    background-color: #FFF;
  }

  .prompt-row {
    display: flex;
    flex-direction: row;

    align-items: stretch;
    justify-content: center;

    .prompt {
      flex-grow: 1;

      padding: 20px;

      display: flex;
      flex-direction: row;

      align-items: center;
      justify-content: center;
    }

    .art-container {
      display: flex;

      align-items: center;
      justify-content: center;

      background-color: #9b9b9b;

      img {
        width: 100%;
        max-width: 1000px;

        height: auto;
      }
    }
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

    /deep/ .draggable-slot {
      height: 53px;
      border: 1px solid black;

      padding: 0;
      list-style: none;
      width: 100%;
      li {
        text-align: center;
      }
    }
  }

</style>
