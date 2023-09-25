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
    <div v-if="sound">
      {{ formatTime(time) }} / {{ formatTime(playlist[currentSongIndex!!].metadata.duration ?? 0) }}
    </div>
  </div>
</template>

<script lang="ts">
import { Howl, Howler } from 'howler'
import * as mmb from 'music-metadata-browser'
import type { ICommonTagsResult, IAudioMetadata,  } from 'music-metadata/lib/type'

type Playlist = {
  filename: string
  url: string
  metadata: ICommonTagsResult & { duration?: number }
}

interface data {
  playlist: Playlist[]
  currentSongIndex?: number
  sound?: Howl
  mode: string,
  time: number
}

export default {
  data(): data {
    return {
      playlist: [],
      currentSongIndex: undefined,
      sound: undefined,
      mode: 'nonRepeat',
      time: 0
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
          this.playlist.push({ filename: file.name, url, metadata: {...metadata.common, duration: metadata.format.duration}})
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
        onplay: this.songPlayingHandler,
        onend: this.songEndHandler
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
        this.time = 0
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
    },
    songPlayingHandler () {
      // console.log(this.sound?.seek(), ((((this.sound?.seek() ?? 0) / (this.sound?.duration() ?? 1)) * 100) || 0))
      this.time = this.sound?.seek() ?? 0
      if (this.sound?.playing()) {
        requestAnimationFrame(this.songPlayingHandler)
      }
    },
    formatTime (secs: number) {
      secs = Math.round(secs)
      let minutes = Math.floor(secs / 60) || 0
      let seconds = (secs - minutes * 60) || 0
      return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`
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