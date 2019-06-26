---
layout: post
title: Vuex stores - binding to state
---

I worked to clean up a vue application's view/viewmodels today.
The work mainly consisted of extracting the js script from single file `.vue` files into a typescript backing viewmodel file.
One particularly onerous item was the App.vue.  It was hooking into the lifecycle of the user profile (after being authenticated) in order to determine if it should display a user section, and what the displayed username should be.

This originally looked like this:

```js
export default {
  data() {
    return {
      currentUser: this.oidcUser,
    };
  },
  computed: {
    ...mapGetters("oidcStore", ["oidcUser"]),
  },
  methods: {
    userLoaded: function(e) {
      this.currentUser = this.oidcUser || e.detail.profile;
    },
    ...mapActions("oidcStore", ["signOutOidc"]),
    logout() {
      try {
        this.currentUser = null;
        this.signOutOidc();
      } catch {}
    }
  },
  mounted() {
    window.addEventListener("vuexoidc:userLoaded", this.userLoaded);
  },
  destroyed() {
    window.removeEventListener("vuexoidc:userLoaded", this.userLoaded);
  }
};
</script>
```

Converted to Typescript it looked like
```typescript
@Component(<ComponentOptions<App>>{
     name: "App"
})
export default class App extends Vue {
    //computed property - gets refreshed with oidcStore.user is changed
    get currentUser() {
        return this.$store.state.oidcStore.user;
    }

    logout() {
      try {
        this.$store.dispatch('oidcStore/signOutOidc');
      } catch {}
    }
}

```

The big pieces to catch are -
by using a <mark>Vuex store</mark> we're able to make `currentUser` a `computed property` that's bound to a vuex property.
Property changed notification bubbles up from the vuex state through the computed property to the property - and the visual element is immediately updated.
This removes all the complexity attaching and removing event listeners and tracking superfluous local state.
Much cleaner.
