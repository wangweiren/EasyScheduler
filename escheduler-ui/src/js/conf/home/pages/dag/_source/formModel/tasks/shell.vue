<template>
  <div class="shell-model">
    <m-list-box>
      <div slot="text">{{$t('脚本')}}</div>
      <div slot="content">
        <div class="from-mirror">
          <textarea
                  id="code-shell-mirror"
                  name="code-shell-mirror"
                  style="opacity: 0">
          </textarea>
        </div>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('资源')}}</div>
      <div slot="content">
        <m-resources
                ref="refResources"
                @on-resourcesData="_onResourcesData"
                :resource-list="resourceList">
        </m-resources>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('自定义参数')}}</div>
      <div slot="content">
        <m-local-params
                ref="refLocalParams"
                @on-local-params="_onLocalParams"
                :udp-list="localParams"
                :hide="false">
        </m-local-params>
      </div>
    </m-list-box>
  </div>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import mListBox from './_source/listBox'
  import mResources from './_source/resources'
  import mLocalParams from './_source/localParams'
  import disabledState from '@/module/mixin/disabledState'
  import codemirror from '@/conf/home/pages/resource/pages/file/pages/_source/codemirror'

  let editor

  export default {
    name: 'shell',
    data () {
      return {
        // script
        rawScript: '',
        // Custom parameter
        localParams: [],
        // resource(list)
        resourceList: []
      }
    },
    mixins: [disabledState],
    props: {
      backfillItem: Object
    },
    methods: {
      /**
       * return localParams
       */
      _onLocalParams (a) {
        this.localParams = a
      },
      /**
       * return resourceList
       */
      _onResourcesData (a) {
        this.resourceList = a
      },
      /**
       * verification
       */
      _verification () {
        // rawScript verification
        if (!editor.getValue()) {
          this.$message.warning(`${i18n.$t('请输入rawScript(必填)')}`)
          return false
        }

        if (!this.$refs.refResources._verifResources()) {
          return false
        }

        // localParams Subcomponent verification
        if (!this.$refs.refLocalParams._verifProp()) {
          return false
        }

        // storage
        this.$emit('on-params', {
          resourceList: this.resourceList,
          localParams: this.localParams,
          rawScript: editor.getValue()
        })
        return true
      },
      /**
       * Processing code highlighting
       */
      _handlerEditor () {
        // editor
        editor = codemirror('code-shell-mirror', {
          mode: 'shell',
          readOnly: this.isDetails
        })

        this.keypress = () => {
          if (!editor.getOption('readOnly')) {
            editor.showHint({
              completeSingle: false
            })
          }
        }

        // Monitor keyboard
        editor.on('keypress', this.keypress)

        editor.setValue(this.rawScript)

        return editor
      }
    },
    watch: {},
    created () {
      let o = this.backfillItem

      // Non-null objects represent backfill
      if (!_.isEmpty(o)) {
        this.rawScript = o.params.rawScript

        // backfill resourceList
        let resourceList = o.params.resourceList || []
        if (resourceList.length) {
          this.resourceList = resourceList
        }

        // backfill localParams
        let localParams = o.params.localParams || []
        if (localParams.length) {
          this.localParams = localParams
        }
      }
    },
    mounted () {
      setTimeout(() => {
        this._handlerEditor()
      }, 200)
    },
    destroyed () {
      if (editor) {
        editor.toTextArea() // Uninstall
        editor.off($('.code-shell-mirror'), 'keypress', this.keypress)
      }
    },
    components: { mLocalParams, mListBox, mResources }
  }
</script>
