<template>
  <div>
    <nav
      class="w-full fixed top-0 bg-green-800 p-3 flex flex-wrap bg-opacity-95 z-50"
    >
      <div class="ml-auto flex gap-x-4 font-semibold">
        <nuxt-link to="/">
          About
        </nuxt-link>
        <nuxt-link to="#users">
          Users
        </nuxt-link>
        <!--
        <nuxt-link to="#portfolio">
          Portfolio
        </nuxt-link>
        -->
        <nuxt-link to="#result">
          Result
        </nuxt-link>
      </div>
    </nav>
    <div class="mt-20 container mx-auto px-5 relative">
      <!-- About Section -->
      <section id="about" class="-mt-16 pt-24 min-h-screen max-h-screen">
        <div class="py-16">
          <div class="flex flex-col justify-center">
            <span class="block text-5xl text-center font-semibold">
              Easy Shares
            </span>
            <span class="block text-gray-300 text-center mt-3 italic">
              Your own NEPSE helper!
            </span>
          </div>
          <div class="mt-16 text-left">
            <span class="block text-2xl font-semibold underline">
              Features
            </span>
            <ul class="ml-5 mt-5 list-disc">
              <li v-for="(item, i) of features" :key="i">
                {{ item }}
              </li>
            </ul>
          </div>
          <div class="flex justify-center py-14">
            <nuxt-link
              to="#users"
              class="px-3 py-2 bg-indigo-600 rounded-md animate-bounce shadow-lg"
            >
              Explore
            </nuxt-link>
          </div>
        </div>
      </section>

      <!-- Users Section -->
      <section id="users" class="-mt-16 pt-24 min-h-screen text-center">
        <Observer
          :active="!capitalList.length"
          @intersect="loadAvailableCapital"
        />
        <span class="text-3xl uppercase font-semibold"> Users </span>
        <div class="mt-10" align="center">
          <!-- Users listing -->
          <div class="px-0 md:px-10 flex justify-center">
            <div>
              <div class="flex bg-gray-900 py-1 px-2">
                <span class="w-48 text-left"> Name </span>
                <span class="w-40 text-left"> BOID </span>
                <span class="w-40 hidden md:block text-center"> Actions </span>
              </div>
              <div v-for="(user, i) of users" :key="user.boid" class="flex flex-col py-1 px-2" :class="{ 'bg-gray-700': i % 2 === 0 }">
                <div class="flex">
                  <span class="w-48 text-left font-semibold"> {{ user.name }} </span>
                  <span class="w-40 text-left"> {{ user.boid }} </span>
                  <span class="w-40 hidden md:flex justify-center gap-x-2">
                    <!--
                    <span class="text-blue-400 cursor-pointer" @click="editUser(user.boid)"> Edit </span>
                    -->
                    <span class="text-red-400 cursor-pointer" @click="removeUser(user.boid)"> Remove </span>
                  </span>
                </div>
                <div class="flex md:hidden mt-2 justify-center gap-x-2 bg-gray-600">
                  <!--
                  <span class="text-blue-400 cursor-pointer" @click="editUser(user.boid)"> Edit </span>
                  -->
                  <span class="text-red-400 cursor-pointer" @click="removeUser(user.boid)"> Remove </span>
                </div>
              </div>
            </div>
          </div>
          <!-- New user form -->
          <div class="mt-20 max-w-xs w-full font-semibold">
            <p class="text-sm mb-1">
              Add new User
            </p>
            <form @submit.stop.prevent="addUser">
              <input
                id="boid-id"
                v-model="newUser.boid"
                type="text"
                name="boid-id"
                required
                maxlength="16"
                class="appearance-none w-full px-3 py-2 border font-semibold text-gray-900 border-gray-300 placeholder-gray-500 rounded-t-md focus:outline-none focus:z-10 sm:text-sm"
                placeholder="Enter 16 digit BOID"
              >
              <input
                id="full-name"
                v-model="newUser.name"
                type="text"
                name="full-name"
                required
                class="appearance-none w-full px-3 py-2 border font-semibold text-gray-900 border-gray-300 placeholder-gray-500 rounded-b-md focus:outline-none focus:z-10 sm:text-sm"
                placeholder="Enter Name associated"
              >
              <span v-if="hasError" class="block mt-2 text-sm text-red-400">
                {{ errorMsg }}
              </span>
              <button
                class="rounded-md font-semibold bg-blue-500 mt-5 px-3 py-2 sm:text-sm focus:outline-none"
                type="submit"
              >
                Submit
              </button>
            </form>
          </div>
        </div>
      </section>

      <!-- Portfolio Section -->
      <!--
      <section id="portfolio" class="-mt-16 pt-24 min-h-screen text-center">
        <span class="text-3xl uppercase font-semibold"> Portfolio </span>
      </section>
      -->

      <!-- Result Section -->
      <section id="result" class="-mt-16 pt-24 min-h-screen text-center">
        <Observer
          :active="!resultCompanies.length && !!users.length"
          @intersect="loadAvailableResults"
        />
        <span class="text-3xl uppercase font-semibold"> Result </span>
        <div class="mt-10" align="center">
          <select
            id="companyID"
            v-model="resultCompany"
            class="text-gray-800 px-2 py-1 focus:outline-none rounded-sm w-96 block tracking-tighter"
            :disabled="!users.length"
          >
            <option value="" disabled>
              Please Select one
            </option>
            <option
              v-for="comp of resultCompanies"
              :key="comp.id"
              :value="comp.id"
              class="text-gray-800 text-sm"
            >
              {{ comp.name }}
            </option>
          </select>
          <button
            :disabled="!resultCompanies.length || !users.length || resultCompany === ''"
            class="rounded-md font-semibold bg-blue-500 mt-5 px-3 py-2 sm:text-sm focus:outline-none"
            :class="{ 'cursor-not-allowed opacity-50' : !resultCompanies.length || !users.length || resultCompany === '' }"
            @click="checkResult"
          >
            Check
          </button>
          <div v-if="result.length" class="mt-16 text-left">
            <div v-for="(res, i) of result" :key="i" class="mt-5">
              <span class="font-semibold"> • {{ res.name }} - {{ res.boid }} </span>
              <div class="ml-5">
                <div v-if="res.success">
                  <span> Congratulations!! 🎉 </span>
                  <span class="block">
                    <span class="font-bold text-green-300"> {{ res.units }} </span>
                    <span> units alloted </span>
                  </span>
                </div>
                <div v-else>
                  <span> Seems like you are not lucky 😢😭 </span>
                  <span class="block text-sm"> Better luck next time! </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>

    <!-- Footer -->
    <div class="mt-10 w-full bg-green-800 p-3 text-center">
      <p class="my-3">
        <span> © 2021 </span>
        <a
          href="https://santoshb.com.np/"
          target="_blank"
          rel="noreferrer noopener"
          class="text-blue-200 font-semibold"
        >
          Santosh Bhandari
        </a>
      </p>
      <p class="my-3">
        <span> All the date available in the website is thanks to </span>
        <a
          href="https://nepalstock.com/"
          target="_blank"
          rel="noreferrer noopener"
          class="text-blue-200 font-semibold"
        >
          NEPSE!
        </a>
      </p>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Observer from '~/components/Observer.vue'

interface User {
  boid: string;
  name: string;
}

export default Vue.extend({
  components: { Observer },
  data () {
    return {
      features: [
        'Portfolio management',
        'View IPO result',
        'Full family support',
        'Easy to use',
        'Complete control of your data',
        'Open source'
      ],
      users: [] as User[],
      resultCompanies: [] as any[],
      resultCompany: '',
      result: [] as any[],
      capitalList: [] as any[],
      newUser: {
        boid: '',
        name: ''
      },
      hasError: false,
      errorMsg: ''
    }
  },
  mounted () {
    const users = this.getLocalStorage('saved-nepse-users')
    if (users) { this.users = users as User[] }
  },
  methods: {
    // Local storage related helpers
    setLocalStorage<V> (key: string, value: V): V {
      localStorage.setItem(key, JSON.stringify(value))
      return value
    },
    getLocalStorage (key: string): unknown {
      const value = localStorage.getItem(key) as string
      return value ? JSON.parse(value) : null
    },
    removeLocaleStorage (key: string) {
      localStorage.removeItem(key)
    },
    // General helpers methods
    async loadAvailableResults () {
      try {
        const { data } = await this.$axios.get(
          '/ipo-result/result/companyShares/fileUploaded'
        )
        this.resultCompanies = data.body.sort((a: any, b: any) => b.id - a.id)
      } catch (err) {
        this.resultCompanies = []
      }
    },
    async loadAvailableCapital () {
      try {
        const { data } = await this.$axios.get('https://backend.cdsc.com.np/api/meroShare/capital')
        this.capitalList = data
      } catch (err) {
        this.capitalList = []
      }
    },
    async checkResult () {
      const users = this.users.map(usr => ({ boid: usr.boid, name: usr.name }))
      this.result = []
      for (const user of users) {
        try {
          const { data } = await this.$axios.post('/ipo-result/result/result/check', {
            companyShareId: this.resultCompany,
            boid: user.boid
          })
          const obj = {
            success: data.success,
            name: user.name,
            boid: user.boid,
            units: 0
          }
          if (data.success === true) {
            const regexMatch = data.message.match(/(\d+)/)
            if (regexMatch) { obj.units = parseInt(regexMatch[0]) }
          }
          this.result.push(obj)
        } catch (err) {
          this.result.push({
            success: false,
            boid: user.boid,
            name: user.name
          })
        }
      }
    },
    addUser (e: Event) {
      const boid = this.newUser.boid
      if (boid === '') { return }
      if (boid.length < 16) { return this.showError('Please Make sure the BOID is 16 digit long!') }
      // validate boid
      if (!boid.startsWith('130')) { return this.showError('Invalid BOID! Please check it again...') }
      if (this.capitalList.length && !this.capitalList.map(cap => cap.code).includes(boid.slice(3, 8))) { return this.showError('Invalid BOID! Please check the first 8 digits...') }

      const currentUsers = this.users
      if (currentUsers.filter(usr => usr.boid === boid).length) { return this.showError('User with same BOID already exists!') }

      currentUsers.push({
        boid,
        name: this.newUser.name
      })

      this.setLocalStorage('saved-nepse-users', currentUsers)

      this.newUser.boid = ''
      this.newUser.name = ''
      this.hasError = false

      e.preventDefault()
    },
    removeUser (boid: string) {
      const user = this.users.find(usr => usr.boid === boid)
      if (!user) { return }
      const yes = confirm(`Are you sure you want to remove ${user.name} with BOID ${user.boid} ??`)
      if (yes === false) { return }
      const remainingUser = this.users.filter(usr => usr.boid !== boid)
      this.setLocalStorage('saved-nepse-users', remainingUser)
      this.users = remainingUser
      alert(`${user.name} with BOID ${user.boid} successfully removed!`)
    },
    showError (msg: string) {
      this.hasError = true
      this.errorMsg = msg
    }
  }
})
</script>
