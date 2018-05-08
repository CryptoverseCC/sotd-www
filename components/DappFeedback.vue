<template>
<div class="component-DappFeedback">
  <div class="wrapper">
    <h3 class="title-3">Would you recommend this ÐApp to a friend?</h3>
    <ul v-if="!hasSubmitted" class="list">
      <li v-for="(option, index) in options" :key="index" class="item"><span class="submit" @click="trackDappFeedback(option)"><component :is="svgFeedbackComponent(option)" :width="25" :height="25"></component></span></li>
    </ul>
    <transition name="fade">
      <p v-if="hasSubmitted" class="confirmation">Thanks for your feedback!</p>
    </transition>
  </div>
</div>
</template>

<script>
import { feedbackComponentMap } from '~/helpers/constants'
import { mapGetters } from 'vuex'
import { trackDappFeedback } from '~/helpers/mixpanel'
import SvgFeedbackNegative from './SvgFeedbackNegative'
import SvgFeedbackNeutral from './SvgFeedbackNeutral'
import SvgFeedbackPositive from './SvgFeedbackPositive'

import core from '@userfeeds/core'
const web3 = import('web3').then((Web3) => {
  if (typeof window.web3 !== 'undefined') {
    return new Web3(window.web3.currentProvider)
  }

  return Promise.reject('No web3 available')
})

export default {
  components: {
    SvgFeedbackNegative,
    SvgFeedbackNeutral,
    SvgFeedbackPositive
  },
  computed: {
    ...mapGetters('dapps/detail', {
      dapp: 'item'
    })
  },
  data () {
    return {
      options: [
        'positive',
        'neutral',
        'negative'
      ],
      hasSubmitted: false
    }
  },
  methods: {
    svgFeedbackComponent (option) {
      const feedbackComponent = feedbackComponentMap[option]
      return feedbackComponent
    },
    trackDappFeedback (feedback) {
      const action = trackDappFeedback(this.dapp.slug, feedback)
      this.$mixpanel.track(action.name, action.data)

      if (feedback === 'positive') {
        web3.then(async (web3Instance) => {
          const [address] = await web3Instance.eth.getAccounts()
          if (!address) {
            return alert('MetaMask is locked')
          }
          if (process.env.userfeedsFeedbackNetwork !== (await core.utils.getCurrentNetworkName(web3Instance))) {
            return alert(`Switch to ${process.env.userfeedsFeedbackNetwork} network`)
          }

          // Show loader
          return core.ethereum.claims.sendClaimValueTransfer(web3Instance, process.env.userfeedsFeedbackAddress, process.env.feedbackFee, {
            claim: {
              target: `stateofthedapps:dapps:${this.dapp.slug}`
            },
            credits: [{
              type: 'interface',
              value: 'https://www.stateofthedapps.com'
            }]
          }).then(({ promiEvent }) => {
            promiEvent.on('transactionHash', () => (this.hasSubmitted = true))
          })
        }).catch((e) => {
          alert('No web3 available')
        })
      } else {
        this.hasSubmitted = true
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import '~assets/css/settings';

.component-DappFeedback {
  margin-bottom: 20px;
  @include tweakpoint('min-width', 1100px) {
    margin-bottom: 0;
  }
}

.confirmation {
  transition: all .5s ease;
  line-height: 25px;
  height: 25px;
  margin: 10px 0;
  @include tweakpoint('min-width', 1100px) {
    margin: 0;
  }
}

.list {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 25px;
  margin: 10px 0;
  @include tweakpoint('min-width', 1100px) {
    margin: 0;
  }
}

.submit {
  display: flex;
  padding: 0 5px;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  @include tweakpoint('min-width', 1250px) {
    padding: 0;
    padding-left: 10px;
  }
}

.title-3 {
  font-weight: 300;
  font-size: 1rem;
  margin: 0;
  @include tweakpoint('min-width', 1100px) {
    margin-right: 5px;
  }
}

.wrapper {
  @include margin-wrapper-main;
  padding: 20px 0;
  text-align: center;
  @include tweakpoint('min-width', 1100px) {
    display: flex;
    align-items: center;
  }
  @include tweakpoint('min-width', 1250px) {
    margin-right: 0;
  }
}
</style>
