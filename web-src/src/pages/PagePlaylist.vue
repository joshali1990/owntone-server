<template>
  <content-with-heading>
    <template #heading-left>
      <div class="title is-4" v-text="playlist.name" />
    </template>
    <template #heading-right>
      <div class="buttons is-centered">
        <a
          class="button is-small is-light is-rounded"
          @click="show_playlist_details_modal = true"
        >
          <span class="icon"><mdicon name="dots-horizontal" size="16" /></span>
        </a>
        <a class="button is-small is-dark is-rounded" @click="play">
          <span class="icon"><mdicon name="shuffle" size="16" /></span>
          <span v-text="$t('page.playlist.shuffle')" />
        </a>
      </div>
    </template>
    <template #content>
      <p
        class="heading has-text-centered-mobile"
        v-text="$t('page.playlist.length', { length: tracks.length })"
      />
      <list-tracks :tracks="tracks" :uris="uris" />
      <modal-dialog-playlist
        :show="show_playlist_details_modal"
        :playlist="playlist"
        :uris="uris"
        @close="show_playlist_details_modal = false"
      />
    </template>
  </content-with-heading>
</template>

<script>
import ContentWithHeading from '@/templates/ContentWithHeading.vue'
import ListTracks from '@/components/ListTracks.vue'
import ModalDialogPlaylist from '@/components/ModalDialogPlaylist.vue'
import webapi from '@/webapi'

const dataObject = {
  load: function (to) {
    return Promise.all([
      webapi.library_playlist(to.params.playlist_id),
      webapi.library_playlist_tracks(to.params.playlist_id)
    ])
  },

  set: function (vm, response) {
    vm.playlist = response[0].data
    vm.tracks = response[1].data.items
  }
}

export default {
  name: 'PagePlaylist',
  components: { ContentWithHeading, ListTracks, ModalDialogPlaylist },

  beforeRouteEnter(to, from, next) {
    dataObject.load(to).then((response) => {
      next((vm) => dataObject.set(vm, response))
    })
  },
  beforeRouteUpdate(to, from, next) {
    const vm = this
    dataObject.load(to).then((response) => {
      dataObject.set(vm, response)
      next()
    })
  },

  data() {
    return {
      playlist: {},
      tracks: [],

      show_playlist_details_modal: false
    }
  },

  computed: {
    uris() {
      if (this.playlist.random) {
        return this.tracks.map((a) => a.uri).join(',')
      }
      return this.playlist.uri
    }
  },

  methods: {
    play: function () {
      webapi.player_play_uri(this.uris, true)
    }
  }
}
</script>

<style></style>
