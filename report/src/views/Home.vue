<template>
  <div>
    <a-layout id="components-layout-demo-fixed">
      <a-layout-header :style="{position: 'fixed',zIndex: 10000,width: '100%',background: '#0867c1'}">
        <svg id="logo" viewBox="0 0 180 24" xmlns:xlink="http://www.w3.org/1999/xlink" width="7.5em" height="1em"
             class="fs6b9dl">
          <defs>
            <path id="a" d="M0 .214v23.539h14.417V.214H0z"></path>
          </defs>
          <g fill="none" fill-rule="evenodd">
            <path fill="#FFF"
                  d="M19.992 16.038l-3.937-3.937 3.937-3.936 3.937 3.936zM84.353 12.07a2.784 2.784 0 1 1 5.567.001 2.784 2.784 0 0 1-5.567 0M82.995.304h-5.25l-8.5 8.502 2.623 2.623zM71.226 12.07l-2.622-2.623L59.46.305h-5.248L65.978 12.07 54.21 23.842h5.246l9.149-9.146 9.146 9.146h5.248z"></path>
            <g transform="translate(0 .09)">
              <mask id="b" fill="#fff">
                <use xlink:href="#a"></use>
              </mask>
              <path fill="#FFF" mask="url(#b)"
                    d="M14.369 23.753L2.623 12.009 14.417.214H9.17L0 9.382v5.252l9.12 9.119z"></path>
            </g>
            <path fill="#FFF"
                  d="M25.615 23.842h5.248l9.118-9.118V9.471L30.814.304h-5.248L37.358 12.1zM122.748 6.822A6.505 6.505 0 0 0 116.25.325H91.258l2.771 3.958h22.221a2.54 2.54 0 0 1 2.539 2.539 2.542 2.542 0 0 1-2.539 2.538H96.545l-1.243 3.42-.197.54-3.83 10.522h3.95l3.832-10.523h9.036l7.369 10.523h4.832l-7.367-10.523h3.323a6.505 6.505 0 0 0 6.498-6.497M141.072.304l-.032-.057-.032.057h-4.22l-.033-.057-.032.057-13.591 23.538h4.285l11.48-19.883 11.481 19.883h4.285zM168.292 12.071L156.524.304h-5.247l11.767 11.767 1.537 1.537v10.234h3.71zM174.812.304l-8.5 8.502 2.622 2.623L180.062.304z"></path>
          </g>
        </svg>
      </a-layout-header>
      <a-layout-content :style="{ padding: '0 50px', marginTop: '96px' }">
<!--        <a-card style="width: 100%; margin-bottom: 24px;">-->
<!--          <h3 slot="title">Load Data</h3>-->
<!--          <div slot="extra">-->
<!--            <a-button type="primary" @click="loadFalsePositive">Load</a-button>-->
<!--          </div>-->
<!--          <a-textarea-->
<!--            v-model="falseData"-->
<!--            placeholder="Paste message"-->
<!--            :autoSize="{ minRows: 6, maxRows: 10 }"-->
<!--          />-->
<!--        </a-card>-->
        <web-vulnerability v-if="webData.length"
                           :data="webData"
                           :loading="loading"
                           @feedback="openFeedback"
                           @download="download"
                           style="margin-bottom: 48px;">

        </web-vulnerability>
        <service-vulnerability v-if="serviceData.length"
                               @feedback="openFeedback"
                               @download="download"
                               :data="serviceData"
                               style="margin-bottom: 48px;"
                               :loading="loading">
        </service-vulnerability>
        <subdomain v-if="subdomainData.length" :data="subdomainData" :loading="loading"></subdomain>
      </a-layout-content>
      <!--??????????????????????????????????????????????????????????????????????????????-->
      <a-layout-footer :style="{ textAlign: 'center' }">
        <a href="https://xray.cool" target="_blank">Powered by XRay Team</a>
      </a-layout-footer>
    </a-layout>

    <a-modal title="????????????"
             @ok="submitSentry"
             :confirmLoading="confirmLoading"
             @cancel="modalVisible=false"
             :visible="modalVisible">
      <p>?????????????????????<span style="color: red">??????</span>??????????????? xray ?????????????????????<span style="color: red">?????????????????????????????????</span></p>
      <p>?????????????????? <a href="https://xray.cool/xray/#/guide/feedback" target="_blank">https://xray.cool/xray/#/guide/feedback</a>
        ?????????????????????????????????</p>
      <p>??????????????????????????????????????????????????????????????????????????? xss ??? sqldet ????????????????????????????????????????????? sqlmap ???????????????????????????????????????????????????</p>
      <p>????????????????????????<span style="color: red">????????????</span>?????????????????????????????????????????????????????????????????????</p>
      <textarea style="width: 100%;" autoSize v-model="comment"></textarea>
    </a-modal>
  </div>
</template>

<script>
import WebVulnerability from "../components/WebVulnerability";
import ServiceVulnerability from "../components/ServiceVulnerability";
import Subdomain from "../components/Subdomain";

export default {
  name: "Home",
  components: {
    WebVulnerability,
    ServiceVulnerability,
    Subdomain,
  },
  created () {
    if (document.readyState === 'complete') {
      this.loadVulns(0)
    } else {
      window.addEventListener("load", () => {
        this.loadVulns(0)
      });
    }
  },
  mounted () {
    if (!window.fetch) {
      alert("Please use modern browser like Chrome, Firefox, Safari to open the report.")
    }
  },
  data () {
    return {
      loading: true,
      modalVisible: false,
      confirmLoading: false,
      falseData: "",
      comment: '',
      dataToSubmit: {},
      serviceData: [],
      webData: [],
      subdomainData: [],
    };
  },
  methods: {
    loadVulns () {
      for (let data of [window.serviceVulns, window.webVulns, window.subdomains]) {
        for (let [i, obj] of data.entries()) {
          obj.id = i
        }
      }
      this.webData = window.webVulns
      this.serviceData = window.serviceVulns
      this.subdomainData = window.subdomains
      this.loading = false
    },
    loadFalsePositive () {
      let obj
      try {
        obj = JSON.parse(this.falseData)
      } catch (e) {
        this.$message.error("invalid json message")
        return
      }
      obj = obj.data
      // console.log(obj)
      obj.id = this.webData.length
      this.webData.push(obj)
      this.$message.success("loaded")
    },
    openFeedback (data) {
      this.modalVisible = true
      this.dataToSubmit = data
    },
    submitSentry () {
      this.confirmLoading = true
      let vulnJson = Object.assign({}, this.dataToSubmit)
      vulnJson.expand = undefined

      let data = {
        version: version,
        gitHash: gitHash,
        comment: this.comment,
        data: vulnJson
      }
      fetch("https://feedback-fc.xray.cool/feedback", {
        method: "POST",
        body: JSON.stringify(data),
      }).then(resp => resp.json()).then(body => {
        if (body.code !== 0) {
          this.$message.error("????????????! " + body.msg)
          return
        }
        this.$message.success("????????????!")
      }).catch(() => {
        this.$message.error("??????????????????????????????")
      }).finally(() => {
        this.modalVisible = false
        this.confirmLoading = false
      })
    },
    download (record) {
      let data = Object.assign({}, record)
      data.expand = undefined
      let link = document.createElement('a')
      link.href = window.URL.createObjectURL(new window.Blob([JSON.stringify(data, null, 2)], {type: "application/json"}))
      link.download = record.plugin
      document.body.appendChild(link)
      link.click()
      window.URL.revokeObjectURL(link.href)
      link.remove()
    }
  }
}
;
</script>

<style lang="less">
.expand-detail *:not(.internal-detail) {
  .ant-descriptions-item-label {
    width: 150px;
  }

  .ant-descriptions-item-content, .ant-descriptions-item-label {
    border-bottom: 1px solid #e8e8e8;;
  }

  pre {
    margin: 8px 0;
  }

  table {
    table-layout: fixed !important;
  }
}

.filter-column {
  .anticon-filter {
    width: 32px !important;
    font-size: 14px !important;
    color: rgba(0, 0, 0, 0.85) !important;

    svg {
      margin-top: -8px !important;
      margin-left: -8px !important;
    }
  }
}
</style>
