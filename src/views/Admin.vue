<template>
  <div class="container-lg mx-auto px-4">
    <div v-if="error">
      Sorry there was a problem determining if you are an admin user ... please try again.
    </div>

    <div v-else-if="loading">
      Loading ...
    </div>

    <div v-else-if="isAdmin">
      <div>
        Unapproved Spaces:
      </div>

      <div class="mt-4">
        <ul v-if="unapprovedSpaces.length > 0" style="list-style: none;">
          <li
            class="mt-2 d-flex flex-items-center"
            v-for="space in unapprovedSpaces"
            :key="space.id">
            <UiButton
              @click="approve(space)"
              class="button--outline">
              Approve
            </UiButton>
            <span class="ml-4 f2">{{ space.id }}</span>
          </li>
        </ul>

        <div v-else>
          Looks like all spaces have been already approved!
        </div>
      </div>
    </div>

    <div v-else-if="!isAdmin">
      Sorry, you are not an admin.
    </div>
  </div>
</template>

<script>
import { mapActions } from 'vuex'

const hubUrl = process.env.VUE_APP_HUB_URL

const checkIsAdmin = async ({ account }) => {
  return await fetch(`${hubUrl}/api/admins/${account}`, {
    headers: {
      'Content-Type': 'application/json'
    }
  })
    .then(x => x.json())
}

const getUnapprovedSpaces = async () => {
  return await fetch(`${hubUrl}/api/spaces/unapproved`, {
    headers: {
      'Content-Type': 'application/json'
    }
  })
    .then(x => x.json())
}

const approveSpace = async ({ space, account }) => {
  const message = JSON.stringify({ approve: space.id })
  const signatureResponse = await window.ethereum.send('personal_sign', [message, account])
  const signature = signatureResponse.result

  return await fetch(`${hubUrl}/api/spaces/${space.id}/approve`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ account, signature, message })
  })
    .then(x => x.json())
}

export default {
  data () {
    return {
      loading: true,
      error: false,
      unapprovedSpaces: [],
      isAdmin: false,
    }
  },

  computed: {
    account: function () {
      return this.$store.state.web3.account
    }
  },

  watch: {
    account: async function (newValue) {
      if (!newValue) return

      this.isAdmin = await checkIsAdmin({ account: newValue })

      this.updateView()
    }
  },

  async mounted () {
    this.isAdmin = await checkIsAdmin({ account: this.account })
    await this.updateView()
  },

  methods: {
    ...mapActions(['notify']),

    async approve (space) {
      try {
        const account = this.$store.state.web3.account

        await approveSpace({ space, account })

        this.unapprovedSpaces = this.unapprovedSpaces
          .filter(x => x.id !== space.id)

        await this.$store.dispatch('getSpaces')
      } catch (err) {
        console.error(err)

        this.notify([
          'red',
          'Sorry there was a problem approving the space, please try again.'
        ])
      }
    },

    async updateView () {
      try {
        this.loading = true

        if (!this.isAdmin) {
          this.loading = false
          return
        }

        this.unapprovedSpaces = await getUnapprovedSpaces()

        this.loading = false
      } catch (err) {
        console.error(err)

        this.error = true
        this.loading = false
      }
    }
  }
}
</script>
