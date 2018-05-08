<template>
<div class="component-DappFeaturedList">
  <div class="wrapper">
    <h2 class="title-2"><SvgIconFeatured/>Promoted ÐApps</h2>
    <div class="featured-wrapper">
      <div class="featured-list-wrapper">
        <ul class="featured-list">
          <DappCardListItem v-for="(dapp, index) in dapps"
            :key="index"
            :dapp="dapp"
            :index="index"
          />
        </ul>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import { trackCollectionView } from '~/helpers/mixpanel'
import axios from '~/helpers/axios'
import DappCardListItem from './DappCardListItem'
import Linkexchange from './Linkexchange'
import SvgIconChevron from './SvgIconChevron'
import SvgIconFeatured from './SvgIconFeatured'

export default {
  components: {
    DappCardListItem,
    Linkexchange,
    SvgIconChevron,
    SvgIconFeatured
  },
  data () {
    return {
      scrollIndex: 0,
      dapps: []
    }
  },
  methods: {
    trackCollectionView (slug) {
      const sourceComponent = 'DappPromotedList'
      const sourcePath = this.$route.path
      const targetCollection = slug
      const action = trackCollectionView(sourceComponent, sourcePath, targetCollection)
      this.$mixpanel.track(action.name, action.data)
    }
  },
  mounted () {
    axios
      .get(`https://api-staging.userfeeds.io/ranking/experimental_boost;asset=${process.env.userfeedsFeedbackNetwork};context=${process.env.userfeedsFeedbackAddress.toLowerCase()}`,
)
      .then(response => {
        const dapps = response.data.items.filter(({ id }) => id.startsWith('stateofthedapps:dapps:'))
        // Fetch all at once
        return Promise.all(dapps.slice(0, 6).map(({ id }) => {
          const [,, slug] = id.split(':')
          return axios.get(`/dapps/${slug}`).then(({ data }) => data.item.slug ? data.item : null)
        }))
      }).then(dapps => {
        this.dapps = dapps.filter((v) => !!v)
      })
  }
}
</script>


<style lang="scss" scoped>
@import '~assets/css/settings';

.component-DappFeaturedList {
  margin-bottom: 10px;
}

.component-SvgIconFeatured {
  margin-right: 7px;
}

.featured-list-wrapper {
  position: relative;
  margin-left: -10px;
  margin-right: -10px;
  flex: 1;
}

.featured-list {
  display: flex;
  flex-wrap: wrap;
  &::-webkit-scrollbar {
    display: none;
  }
}

.featured-list-spacer {
  width: 10px;
}

.featured-wrapper {
  @include tweakpoint('min-width', 750px) {
    display: flex;
  }
}

.wrapper {
  @include margin-wrapper-main;
  position: relative;
}
</style>

