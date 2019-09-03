# Using Hasura Backend Plus with Nuxt

[![Edit Nuxt + HBP](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/codesandbox-nuxt-8zegn?fontsize=14)

Use [nhost.io](https://nhost.io) for zero-config HBP!

## Dependencies

 - [`@nuxtjs/auth`](https://auth.nuxtjs.org)
 - [`@nuxtjs/apollo`](https://github.com/nuxt-community/apollo-module)
 
**Install:**
```bash
# npm

npm install @nuxtjs/auth @nuxtjs/apollo

# yarn

yarn add @nuxtjs/auth @nuxtjs/apollo
```

## Setup

`nuxt.config.js`:

```js
export default {
  modules: [
    '@nuxtjs/apollo',
    '@nuxtjs/auth',
  ],
  
  apollo: {
    tokenName: 'auth._token.local', // set by @nuxtjs/auth module
    authenticationType: '', // auth._token.local contains 'Bearer' prefix already
    clientConfigs: {
      default: {
        httpEndpoint: '', // insert your own graphql link
        httpLinkOptions: {
          includeExtensions: true,
        },
      },
    },
  },

  auth: {
    strategies: {
      local: {
        endpoints: {
          login: {
            url: '', // insert login link (https://backend-xxxxxxxx.nhost.io/auth/login)
            method: 'post',
            propertyName: 'jwt_token',
          },
          logout: false,
          user: {
            url: '', // insert user link (https://backend-xxxxxxxx.nhost.io/auth/user)
            method: 'get',
            propertyName: 'user',
          },
        },
      },
    },
  },
}
```

`./pages/login.vue` or default login page:

```vue
<template>
  <div>
    <input v-model="email" type="email" placeholder="Email"><br>
    <input v-model="password" type="password" placeholder="Password"><br>
    <button @click="login">
      Login
    </button>
  </div>
</template>

<script>
export default {
  name: 'TheLogin',
  data () {
    return {
      username: '',
      password: '',
    }
  },
  methods: {
    async login () {
      try {
        await this.$auth.loginWith('local', {
          username: this.username,
          password: this.password,
        })
        
        this.$store.dispatch('auth-refresh/start')
      } catch (err) {
        console.err(err.message)
      }
    },
  },
}
```

`./store/auth-refresh.js`:

```js
export const state = () => {
  return {
    refreshInterval: null,
  }
}

export const mutations = {
  UPDATE_REFRESH_INTERVAL(state, interval) {
    state.refreshInterval = interval
  },
}

export const actions = {
  start ({ commit, dispatch }) {
    const jwt = parseJwt('') // get jwt from auth module
    const interval = jwt.exp - jwt.iat // Expires - Issued At = lifetime
    commit('UPDATE_REFRESH_INTERVAL', interval)
    
    dispatch('startFetchLoop')
  },
  startFetchLoop({ state }) {
    setInterval(function () {
      // refresh token
    }, state.refreshInterval)
  },
}

// https://stackoverflow.com/a/38552302
function parseJwt (token) {
  const base64Url = token.split('.')[1]
  const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/')
  const jsonPayload = decodeURIComponent(atob(base64).split('').map(function(c) {
    return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2)
  }).join(''))

  return JSON.parse(jsonPayload)
}
```

add the following to the default export on any page you wish to authenticate, as per [`@nuxtjs/auth` docs](https://auth.nuxtjs.org/guide/middleware.html):

```js
export default {
  middleware: ['auth'],
}
```

