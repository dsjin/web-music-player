<template>
  <div>
    <h1>Local Music Player</h1>
    <input type="file" @change="handleFileUpload" />
    <div v-for="(song, index) in playlist" :key="index">
      <div
        :style="{
          'background-color': currentSongIndex === index ? 'white' : 'unset',
          'color': currentSongIndex === index ? 'black' : 'unset'
        }"
        @click="handleChangingSong(index)"
      >
        {{ song.name }}
      </div>
    </div>
    <div>
      <button :disabled="currentSongIndex === undefined" @click="currentSongIndex !== undefined ? playSong(currentSongIndex) : null">Play</button>
      <button :disabled="currentSongIndex === undefined" @click="currentSongIndex !== undefined ? pauseSong() : null">Pause</button>
      <button :disabled="currentSongIndex === undefined" @click="currentSongIndex !== undefined ? stopSong() : null">Stop</button>
    </div>
  </div>
</template>

<script lang="ts">
import { Howl, Howler } from 'howler'

type Playlist = {
  name: string
  url: string
}

interface data {
  playlist: Playlist[]
  currentSongIndex?: number
  sound?: Howl
}

export default {
  data(): data {
    return {
      playlist: [],
      currentSongIndex: undefined,
      sound: undefined,
    }
  },
  methods: {
    handleFileUpload(event: any) {
      const file = event.target.files[0]
      const reader = new FileReader()
      reader.addEventListener(
        'load',
        () => {
          const url = reader.result as string
          this.playlist.push({ name: file.name, url })
        }
      )
      reader.readAsDataURL(file)
    },
    handleChangingSong(index: number) {
      if (this.sound) {
        this.sound.unload()
        this.sound = undefined
      }
      this.currentSongIndex = index
    },
    playSong(index: number) {
      if (this.sound) {
        this.sound.play()
        return
      }

      this.currentSongIndex = index
      const song = this.playlist[index]

      this.sound = new Howl({
        src: [song.url],
      })

      this.sound.play()
    },
    pauseSong() {
      if (this.sound) {
        this.sound.pause()
      }
    },
    stopSong() {
      if (this.sound) {
        this.sound.stop()
      }
    }
  },
  beforeDestroy() {
    if (this.sound) {
      this.sound.unload()
    }
  },
}
</script>