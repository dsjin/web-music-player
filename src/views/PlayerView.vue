<template>
  <div>
    <h1>Local Music Player</h1>
    <input type="file" @change="handleFileUpload" />
    <div>
      Selected mode: {{ mode }}
    </div>
    <div>
      <input v-model="mode" type="radio" id="nonRepeat" name="mode" value="nonRepeat"/>
      <label for="nonRepeat">Non Repeat</label>
    </div>
    <div>
      <input v-model="mode" type="radio" id="repeatAll" name="mode" value="repeatAll" />
      <label for="repeatAll">Repeat All</label>
    </div>
    <div>
      <input v-model="mode" type="radio" id="repeatOnce" name="mode" value="repeatOnce" />
      <label for="repeatOnce">Repeat Once</label>
    </div>
    <div v-for="(song, index) in playlist" :key="index">
      <div
        :style="{
          'background-color': currentSongIndex === index ? 'white' : 'unset',
          'color': currentSongIndex === index ? 'black' : 'unset'
        }"
        @click="handleChangingSong(index)"
      >
        {{ song.metadata.title ?? song.filename }}
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
import * as mmb from 'music-metadata-browser'
import type { ICommonTagsResult, IAudioMetadata } from 'music-metadata/lib/type'

type Playlist = {
  filename: string
  url: string
  metadata: ICommonTagsResult
}

interface data {
  playlist: Playlist[]
  currentSongIndex?: number
  sound?: Howl
  mode: string
}

export default {
  data(): data {
    return {
      playlist: [],
      currentSongIndex: undefined,
      sound: undefined,
      mode: 'nonRepeat'
    }
  },
  methods: {
    async handleFileUpload(event: any) {
      const file: File = event.target.files[0]
      const blob = new Blob([file], {type: file.type})
      const reader = new FileReader()
      reader.addEventListener(
        'load',
        async () => {
          const url = reader.result as string
          const metadata: IAudioMetadata = await mmb.parseBlob(blob)
          this.playlist.push({ filename: file.name, url, metadata: metadata.common})
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

      this.sound.on('end', this.songEndHandler)
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
    },
    songEndHandler () {
      if (this.currentSongIndex !== undefined) {
        if (this.mode === 'repeatOnce') {
          this.playSong(this.currentSongIndex)
          return
        }
        const isFinal = this.currentSongIndex === (this.playlist.length - 1)
        this.currentSongIndex = (this.currentSongIndex + 1) % this.playlist.length
        this.sound?.unload()
        this.sound = undefined
        if ((this.mode === 'nonRepeat' && !isFinal ) || this.mode === 'repeatAll') {
          this.playSong(this.currentSongIndex)
        }
      }
    }
  },
  beforeDestroy() {
    if (this.sound) {
      this.sound.stop()
      this.sound.unload()
      this.sound = undefined
    }
  }
}
</script>