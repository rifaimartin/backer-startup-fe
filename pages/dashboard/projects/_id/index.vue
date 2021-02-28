<template>
  <div class="project-page">
    <section class="dashboard-header pt-5">
      <div class="container mx-auto relative">
        <Navbar />
      </div>
    </section>
    <section class="container mx-auto pt-8">
      <div class="flex justify-between items-center">
        <div class="w-full mr-6">
          <h2 class="text-4xl text-gray-900 mb-2 font-medium">Dashboard</h2>
        </div>
      </div>
      <div class="flex justify-between items-center">
        <div class="w-3/4 mr-6">
          <h3 class="text-2xl text-gray-900 mb-4">Campaign Details</h3>
        </div>
        <div class="w-1/4 text-right">
          <nuxt-link
            :to="{
              name: 'dashboard-projects-id-edit',
              params: { id: campaign.data.data.id },
            }"
            class="bg-green-button hover:bg-green-button text-white font-bold px-4 py-1 rounded inline-flex items-center"
          >
            Edit
          </nuxt-link>
        </div>
      </div>
      <div class="block mb-2">
        <div class="w-full lg:max-w-full lg:flex mb-4">
          <div
            class="border w-full border-gray-400 bg-white rounded p-8 flex flex-col justify-between leading-normal"
          >
            <div>
              <div class="text-gray-900 font-bold text-xl mb-2">
                {{ campaign.data.data.name }}
              </div>
              <p class="text-sm font-bold flex items-center mb-1">
                Short Description
              </p>
              <p class="text-gray-700 text-base">
                {{ campaign.data.data.short_description }}
              </p>

              <p class="text-sm font-bold flex items-center mb-1 mt-3">
                Description
              </p>
              <p class="text-gray-700 text-base">
                {{ campaign.data.data.description }}
              </p>

              <p class="text-sm font-bold flex items-center mb-1 mt-4">
                What will Funders Get
              </p>
              <ul class="list-disc ml-5">
                <li v-for="perk in campaign.data.data.perks" :key="perk">
                  {{ perk }}
                </li>
              </ul>
              <p class="text-sm font-bold flex items-center mb-1 mt-4">
                Goal Amount
              </p>
              <p class="text-4xl text-gray-700 text-base">
                {{
                  new Intl.NumberFormat().format(campaign.data.data.goal_amount)
                }}
              </p>
            </div>
          </div>
        </div>
      </div>
      <div class="flex justify-between items-center">
        <div class="w-3/4 mr-6">
          <h3 class="text-2xl text-gray-900 mb-4 mt-5">Gallery</h3>
        </div>
        <div class="w-2/4 text-right">
          <input
            type="file"
            ref="file"
            @change="selectFile"
            class="border p-1 rounded overflow-hidden"
          />
          <button
            v-if="!loading"
            @click="upload"
            class="bg-green-button hover:bg-green-button text-white font-bold px-4 py-2 rounded inline-flex items-center"
          >
            Upload
          </button>
        </div>
      </div>
      <div class="grid grid-cols-4 -mx-2">
        <div
          class="relative w-full bg-white m-2 p-2 border border-gray-400 rounded"
          v-for="image in campaign.data.data.images"
          :key="image.image_url"
        >
          <figure class="item-thumbnail">
            <img
              :src="$axios.defaults.baseURL + '/' + image.image_url"
              alt=""
              class="rounded w-full"
            />
          </figure>
        </div>
      </div>
      <div class="flex justify-between items-center">
        <div class="w-3/4 mr-6">
          <h3 class="text-2xl text-gray-900 mb-4 mt-5">Transaction History</h3>
        </div>
      </div>
      <div class="block mb-2" v-for="i in 5" :key="i">
        <div
          class="w-full lg:max-w-full lg:flex mb-4"
          v-for="transaction in transactions.data.data"
          :key="transaction.id"
        >
          <div
            class="w-full border border-gray-400 lg:border-gray-400 bg-white rounded p-8 flex flex-col justify-between leading-normal"
          >
            <div>
              <div class="text-gray-900 font-bold text-xl mb-1">
                {{ transaction.name }}
              </div>
              <p class="text-sm text-gray-600 flex items-center mb-2">
                Rp.
                {{ new Intl.NumberFormat().format(transaction.amount) }}
                &middot; {{ transaction.created_at }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </section>
    <div class="cta-clip -mt-20"></div>
    <section class="call-to-action bg-purple-progress pt-64 pb-10"></section>
    <Footer />
  </div>
</template>

<script>
export default {
  middleware: 'auth',
  async asyncData({ $axios, params }) {
    const campaign = await $axios.get('api/v1/campaigns/' + params.id)
    const transactions = await $axios.get(
      'api/v1/campaigns/' + params.id + '/transactions'
    )

    return { campaign, transactions }
  },

  data() {
    return {
      selectedFile: undefined,
      loading: false,
    }
  },

  methods: {
    selectFile() {
      this.selectedFile = this.$refs.file.files
    },

    async load() {
      const campaign = await this.$axios.$get(
        'api/v1/campaigns/' + this.$route.params.id
      )

      this.campaign = campaign
    },

    async upload(file) {
      let formData = new FormData()

      formData.append('campaign_id', this.$route.params.id)
      formData.append('file', this.selectedFile.item(0))
      formData.append('is_primary', true)

      try {
        this.loading = true

        let response = await this.$axios.post(
          '/api/v1/campaign-images',
          formData,
          {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          }
        )
        console.log(response)
        this.loading = false
        this.load()
        this.selectedFiles = undefined
      } catch (error) {
        console.log(error)
      }
    },
  },
}
</script>
