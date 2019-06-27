---
layout: post
title: Vuex stores - binding to state
---

I worked to clean up a vue application's view/viewmodels today.
The work mainly consisted of extracting the js script from single file `.vue` files into a typescript backing viewmodel file.
One particularly onerous item was the App.vue.  It was hooking into the lifecycle of the user profile (after being authenticated) in order to determine if it should display a user section, and what the displayed username should be.

This originally looked like this:

```js
// export default {
  data() {
    return {
      currentUser: this.oidcUser,
      // more state...
    };
  },
  computed: {
    ...mapGetters("oidcStore", ["oidcUser"]),
  },
  methods: {
    userLoaded: function(e) {
      this.currentUser = this.oidcUser || e.detail.profile;
    },
    // more mappings, more methods...
  },
  mounted() {
    window.addEventListener("vuexoidc:userLoaded", this.userLoaded);
  },
  destroyed() {
    window.removeEventListener("vuexoidc:userLoaded", this.userLoaded);
  }
  
// lots more stuff...
// };
```

Converted to Typescript it looked like
```typescript
// @Component(<ComponentOptions<App>>{
//      name: "App"
// })
// export default class App extends Vue {
    get currentUser() {
        return this.$store.state.oidcStore.user; //computed property - gets refreshed with oidcStore.user is changed
    }

// more state...
// more actions...
// }
```

The big pieces to catch are -
by using a **Vuex store** we're able to make `currentUser` a *computed property* that's bound to a *vuex property*.
Property changed notification bubbles up from the vuex state through the computed property to the property - and the visual element is immediately updated.
This removes all the complexity attaching and removing event listeners and tracking superfluous local state.
Much cleaner.
