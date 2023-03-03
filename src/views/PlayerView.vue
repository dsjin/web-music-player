<template>
  <div>
    <h1>Local Music Player</h1>
    <input type="file" @change="handleFileUpload" />
    <div v-for="(song, index) in playlist" :key="index">
      <span>{{ song.name }}</span>
      <button @click="playSong(index)">Play</button>
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
    playSong(index: number) {
      if (this.sound) {
        this.sound.stop()
      }

      this.currentSongIndex = index
      const song = this.playlist[index]

      this.sound = new Howl({
        src: [song.url],
      })

      this.sound.play()
    },
  },
  beforeDestroy() {
    if (this.sound) {
      this.sound.unload()
    }
  },
}
</script>