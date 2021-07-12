<script lang="js">
// Note: "voice over INSPECTOR" might be a better name for this component.
// import applyConverters from "axios-case-converter";
// import axios from "axios";
// import * as helpers from "../modules/helpers";
// import AudioRecorder from "../audio_recorder";
// import Video from "../models/Video";
// import VoiceOver from "../models/VoiceOver";
// import AudioRecording from "../models/AudioRecording";

export default {
  components: {},
  props: {
    voiceOverId: {
      type: String,
      required: true,
    },
    // isRecordingAudio: {
    //   type: Boolean,
    //   required: true,
    // },
    // isRecordingBrowser: {
    //   type: Boolean,
    //   required: true,
    // },
  },
  emits: ["click", "will-start-count-down", "started-recording-audio", "stopped-recording-audio"],
  data() {
    return {
      countDown: null,
      audioStream: null,
      audioRecorder: null,
      isSavingAudioRecording: false,
      text: null,
      audioRecordings: [],
      // isRecordingAudio: false,
    };
  },
  computed: {
  },
  watch: {
  },
  created() {
  },
  mounted() {
  },
  beforeUnmount() {
  },
  methods: {
    selectVoiceOverId(voiceOverId) {
      // console.log("select-voice-over-id", voiceOverId);
      this.$emit("select-voice-over-id", voiceOverId);
    },

  }
};
</script>

<template>
  <div class="voice-over-inspector inspector" @click="$emit('click', $event)">
    <button @click="selectVoiceOverId('abcd')"></button>
    <div v-if="countDown" class="count-down">{{ countDown }}</div>

    <textarea
      ref="teleprompter"
      v-model="text"
      class="teleprompter"
      placeholder="Write your voice-over here and use it as your teleprompter…"
    />

    <div class="audio" @click="$refs.teleprompter.focus()">
      <ul class="recordings">
        <li v-for="audioRecording in audioRecordings" :key="audioRecording.id">
          <label
            class="recording button"
            :class="{
              'is-selected primary': audioRecording.selected,
              secondary: !audioRecording.selected,
            }"
            :title="audioRecording.name"
            @click="selectAudioRecording(audioRecording)"
          >
            <font-awesome-icon icon="microphone" />
            <div class="duration">{{ audioRecording.formattedDuration }}</div>
            <button
              class="remove-button"
              title="Remove this audio recording"
              @click.stop="removeAudioRecording(audioRecording)"
            >
              <font-awesome-icon icon="times" />
            </button>
          </label>
        </li>
        <li v-if="isSavingAudioRecording">
          <label class="loading button secondary">
            <font-awesome-icon icon="cog" spin />
            <div class="name">Saving…</div>
          </label>
        </li>
      </ul>

      <div v-if="audioRecordings.length < 3" class="actions">
        <button
          v-if="isRecordingAudio"
          class="stop-button button is-recording"
          @click="stopRecordingAudio"
        >
          <font-awesome-icon icon="stop-circle" />
          <span>Stop</span>
        </button>
        <template v-else-if="!isSavingAudioRecording">
          <button
            class="start-button button"
            :class="{
              primary: audioRecordings.length === 0,
              secondary: audioRecordings.length > 0,
            }"
            :disabled="isRecordingBrowser"
            @click="startRecordingAudio(3)"
          >
            <font-awesome-icon icon="microphone" />
            <span>Record audio</span>
          </button>

          <label
            class="uploader button secondary"
            :disabled="isRecordingBrowser || isRecordingAudio"
          >
            <!--
              In the future we’ll probably have to transcode files for browser playback
              compatibility.
              But we should probably use the original files for the pipeline.
            -->
            + Add audio file
            <input
              class="upload-field"
              type="file"
              accept="audio/*"
              :disabled="isRecordingBrowser || isRecordingAudio"
              @change="uploadAudioFile($event.target.files[0])"
            />
          </label>
        </template>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import "../../assets/stylesheets/variables.scss";
@import "../../assets/stylesheets/editor_buttons.scss";
@import "../../assets/stylesheets/inspector.scss";

.voice-over-inspector {
  height: 100%;
  display: flex;
  flex-direction: column;
  transition: all 0.1s;
  $border-color-while-hovered: blue;
  &:hover {
    box-shadow: 0 0 0 1px rgba($primary-button-color, 1) inset,
      0 5px 15px rgba($primary-button-color, 0.1);
  }
  &:focus-within {
    box-shadow: 0 0 0 3px rgba($primary-button-color, 1) inset,
      0 5px 15px rgba($primary-button-color, 0.1);
  }
}

.teleprompter {
  flex: 1;
  display: flex;
  flex-direction: column;
  color: white;
  background: transparent;
  outline: none;
  border: none;
  border-radius: 1rem;
  padding: 1rem 1.25rem;
  font-size: 2vw;
  // text-shadow: 0 1px 2px black;
  text-align: center;
  resize: none;
}

.count-down {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  font-size: 10vw;
  background: rgba(255, 0, 0, 0.5);
  color: rgba(255, 255, 255, 1);

  @keyframes pulsating-count-down {
    0% {
      font-size: 6vw;
      color: rgba(255, 255, 255, 0);
    }

    10% {
      font-size: 14vw;
      color: rgba(255, 255, 255, 1);
    }

    60% {
      font-size: 6vw;
      color: rgba(255, 255, 255, 0);
    }

    100% {
      font-size: 6vw;
      color: rgba(255, 255, 255, 0);
    }
  }

  animation: 1s ease-in-out infinite both pulsating-count-down;
  transform: rotateZ(
    360deg
  ); // Reduce GPU load: https://stackoverflow.com/questions/13176746/css-keyframe-animation-cpu-usage-is-high-should-it-be-this-way
}

.audio {
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: auto;
  text-align: center;

  .recordings {
    display: flex;
    list-style: none;
    text-align: left;

    &:not(:last-child) {
      margin-right: 1rem;
    }
  }

  .recording,
  .button,
  .uploader {
    height: 2rem;
    padding: 0.25em;
  }

  .recording,
  .loading {
    input[type="radio"] {
      display: none;
    }

    > * {
      display: block;
    }

    overflow: hidden;
    text-overflow: ellipsis;
    padding-left: 0.5rem;

    &.loading {
      padding-right: 1rem;
    }

    .name {
      width: max-content;
      max-width: 0.2 * 65vw;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .duration,
    > svg {
      opacity: 0.5;
      display: block;
    }

    .duration {
      margin-right: 0.5em;
      font-size: 0.8rem;
      width: 3em;
      text-align: center;
    }

    .remove-button {
      border-radius: 2em;
      background: rgba(0, 0, 0, 0.2);
      // box-shadow: 0 0 0 1px rgba(0,0,0,.1) inset;
      border: 0;
      padding: 0.25em;
      display: block;
      justify-content: center;
      cursor: pointer;

      svg {
        color: white;
        display: block;
        width: 1em;
        height: 1em;
      }
    }
  }

  li:first-child .recording,
  li:first-child .loading {
    padding-left: 1em;
  }

  li:not(:last-child) .recording,
  li:not(:last-child) .loading {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
    margin-right: 1px;
  }

  li:not(:first-child) .recording,
  li:not(:first-child) .loading {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }

  .actions {
    display: flex;
    padding: 0.25rem 0;
    align-items: center;

    .button {
      width: 21ch;
      margin-right: 2px;

      &:last-child {
        margin-right: 0;
      }
    }

    .uploader {
      position: relative;

      .upload-field {
        opacity: 0; // Invisible but it’s there!
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        cursor: pointer;
      }
    }
  }
}
</style>
