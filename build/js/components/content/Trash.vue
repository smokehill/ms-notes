<template>
  <div class="content" v-bind:class="{ 'lg': isSidebarMini }">
    <div class="content-header fixed">
      <h4 class="title">{{ "list.title_trash"|i18n }}</h4>
      <div class="controls">
        <ol class="breadcrumb">
          <li v-on:click="selectAll($event)" :data-tooltip="'list.breadcrumb_select'|i18n">
            <i class="fa" v-bind:class="{ 'fa-checkbox-o': selectedAll, 'fa-checkbox': !selectedAll }"></i>
          </li>
          <li v-on:click="restoreSelected($event)" :data-tooltip="'list.breadcrumb_restore'|i18n">
            <i class="fa fa-restore"></i>
          </li>
          <li v-on:click="deleteSelected($event)" :data-tooltip="'list.breadcrumb_delete_forever'|i18n">
            <i class="fa fa-trash-r"></i>
          </li>
          <li>
            <div class="search">
              <input type="text" v-model="search" :placeholder="'list.breadcrumb_search'|i18n">
            </div>
          </li>
        </ol>
        <span class="total">{{ "list.breadcrumb_total"|i18n }}: {{ total }}</span>
      </div>
    </div>
    <div class="content-body fixed">
      <list ref="list" v-bind:type="type"></list>
    </div>
  </div>
</template>

<script>
  import List from './inc/List.vue';
  import { lsGet, lsSet, i18n } from './../../helpers.js';

  export default {
    data() {
      return {
        type: 'trash',
        total: 0,
        search: '',
        selectedAll: false,
        isSidebarMini: false
      }
    },
    components: {
      'list': List
    },
    watch: {
      search: function (val) {
        this.selectedAll = false;
        this.$refs.list.search(val);
        this.total = this.$refs.list.count();
      }
    },
    mounted: function () {
      this.isSidebarMini = lsGet('sidebar_mini');
      this.total = this.$refs.list.count();
    },
    methods: {
      /**
       * Select all notes.
       * @param {object} e
       */
      selectAll(e) {
        e.preventDefault();
        let status = 'on';
        this.selectedAll = (!this.selectedAll) ? true : false;
        if (!this.selectedAll) {
          status = 'off';
        }
        // reset checked values
        for (let i = 0; i < this.$refs.list.items.length; i++) {
          if (status == 'on' && !this.$refs.list.items[i].is_hidden && !this.$refs.list.items[i].is_checked) {
            this.$refs.list.items[i].is_checked = true;
          } else if (status == 'off' && !this.$refs.list.items[i].is_hidden && this.$refs.list.items[i].is_checked) {
            this.$refs.list.items[i].is_checked = false;
          }
        }
      },
      /**
       * Restore selected notes.
       * @param {object} e
       */
      restoreSelected(e) {
        e.preventDefault();
        let selected = [];
        let lsNotes = lsGet('notes');
        this.search = '';

        for (let i = 0; i < this.$refs.list.items.length; i++) {
          if (this.$refs.list.items[i].is_checked) {
            selected.push(this.$refs.list.items[i].id);
          }
        }

        if (selected.length > 0 && lsNotes != null) {
          for (let i = 0; i < lsNotes.length; i++) {
            if (selected.indexOf(lsNotes[i].id) != -1) {
              lsNotes[i].type = 'notes';
              this.$eventBus.$emit('modal.changeHeader', 'notes');
            }
          }
          lsSet('notes', lsNotes);
          this.$refs.list.init();
        }

        this.selectedAll = false;
        this.total = this.$refs.list.count();
      },
      /**
       * Drop selected notes from Local Storage.
       * @param {object} e
       */
      deleteSelected(e) {
        e.preventDefault();
        let selected = [];
        let lsNotes = lsGet('notes');
        this.search = '';

        for (let i = 0; i < this.$refs.list.items.length; i++) {
          if (this.$refs.list.items[i].is_checked) {
            selected.push(this.$refs.list.items[i].id);
          }
        }

        if (selected.length > 0 && lsNotes != null) {
          for (let i = lsNotes.length - 1; i >= 0; i--) {
            if (selected.indexOf(lsNotes[i].id) != -1) {
              if (lsNotes[i].id == this.$store.state.noteId) {
                this.$eventBus.$emit('modal.close');
              }
              lsNotes.splice(i, 1);
            }
          }
          lsSet('notes', lsNotes);
          this.$refs.list.init();
        }

        this.selectedAll = false;
        this.total = this.$refs.list.count();
      }
    }
  }
</script>