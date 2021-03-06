<template>
  <div>
    <!-- navigation menu builder -->
    <h3 class="is-size-3">Navigation Menu Builder</h3>
    <div class="box">
      <div class="columns">

        <!-- link form -->
        <div class="column">
          <div class="field">
            <label class="label">Name</label>
            <div class="control">
              <input class="input" type="text" placeholder="Name (ex: Posts)" v-model="name">
            </div>
          </div>
          <div class="field">
            <label class="label">Path</label>
            <div class="control">
              <input class="input" type="text" placeholder="Path (ex: /admin/posts)" v-model="path">
            </div>
          </div>
          <div class="field is-grouped">
            <div class="control">
              <button v-if="action === 'new'" class="button is-info" @click="addLink">Add</button>
              <button v-if="action === 'edit'" class="button is-info" @click="updateLink">
                Update
              </button>
              <button v-if="action === 'sub'" class="button is-info" @click="appendSubLink">
                Add sub-link
              </button>
            </div>
            <div class="control">
              <button class="button" @click="clear">Cancel</button>
            </div>
          </div>
        </div>

        <!-- menu visualization -->
        <div class="column">
          <p class="is-size-4">menu</p>
          <ul v-for="(item, index) in menu" :key="index" class="nav">
            <li>
              {{item.name}}: {{item.path}}

              <span class="link-actions">
                <span class="has-text-danger fa fa-trash" @click="removeLink(item)"></span>
                <span v-if="index !== 0" class="has-text-success fa fa-arrow-up" @click="moveUp(item, menu[index - 1])"></span>
                <span v-if="index !== menu.length - 1" class="fa fa-arrow-down" @click="moveDown(item, menu[index + 1])"></span>
                <span class="has-text-info fa fa-edit" @click="editLink(item)"></span>
                <span class="has-text-primary fa fa-plus" @click="addSubLink(item)"></span>
              </span>

              <!-- render children links -->
              <ul v-if="item.children" class="sub-nav">
                <li v-for="(child, key) in item.children" :key="key">
                  {{child.name}}: {{child.path}}

                  <span class="link-actions">
                    <span class="has-text-danger fa fa-trash" @click="removeSubLink(key, item)"></span>
                  </span>
                </li>
              </ul>
            </li>
          </ul>
        </div>

      </div>

    </div>
  </div>
</template>

<script>
import { settingsRef, navRef } from '@/firebase_config'
export default {
  data () {
    return {
      key: '',
      link: '', // {'.key', name, path}
      name: '',
      path: '',
      action: 'new'
    }
  },
  computed: {
    isAbsolute () {
      return this.path.startsWith('http')
    }
  },
  firebase: {
    settings: {
      source: settingsRef,
      asObject: true
    },
    menu: {
      source: navRef
    }
  },
  methods: {
    addLink () {
      if (this.name && this.path) {
        this.$firebaseRefs.menu.push({
          name: this.name,
          path: this.path,
          isAbsolute: this.isAbsolute
        })
      }
      this.clear()
    },
    editLink (link) {
      this.name = link.name
      this.path = link.path
      this.action = 'edit'
      this.key = link['.key']
      this.link = Object.assign({}, link)
    },
    updateLink () {
      delete this.link['.key']
      this.link.name = this.name
      this.link.path = this.path
      this.link.isAbsolute = this.isAbsolute

      this.$firebaseRefs.menu.child(this.key).set(this.link)
      this.clear()
    },
    removeLink (item) {
      this.$firebaseRefs.menu.child(item['.key']).remove()
    },
    clear () {
      this.name = ''
      this.path = ''
      this.action = 'new'
    },
    moveUp (item, previousItem) {
      let itemCopy = Object.assign({}, item)
      let previousItemCopy = Object.assign({}, previousItem)

      delete itemCopy['.key']
      delete previousItemCopy['.key']

      this.$firebaseRefs.menu.child(item['.key']).set(previousItemCopy)
      this.$firebaseRefs.menu.child(previousItem['.key']).set(itemCopy)
    },
    moveDown (item, nextItem) {
      let itemCopy = Object.assign({}, item)
      let nextItemCopy = Object.assign({}, nextItem)

      delete itemCopy['.key']
      delete nextItemCopy['.key']

      this.$firebaseRefs.menu.child(item['.key']).set(nextItemCopy)
      this.$firebaseRefs.menu.child(nextItem['.key']).set(itemCopy)
    },
    addSubLink (link) {
      this.action = 'sub'
      this.key = link['.key']
    },
    appendSubLink () {
      this.$firebaseRefs.menu.child(this.key).child('children').push({
        name: this.name,
        path: this.path,
        isAbsolute: this.isAbsolute
      })
      this.clear()
    },
    removeSubLink (key, parent) {
      this.$firebaseRefs.menu.child(parent['.key']).child('children').child(key).remove()
    }
  }
}
</script>

<style lang="scss" scoped>
.nav {
  padding-left: 15px;
}
.sub-nav {
  padding-left: 30px;
}
.link-actions {
  display: none;
  span {
    cursor: pointer;
  }
}

li:hover .link-actions {
  display: inline;
}

</style>
