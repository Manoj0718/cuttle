<template>
  <v-container id="lobby-wrapper">
    <v-row>
      <v-col sm="3" md="1" class="my-auto">
        <img id="logo" alt="Cuttle logo" src="/img/logo.png" />
      </v-col>
      <v-col md="8" class="my-auto">
        <h1>
          Lobby for {{ gameName }}
          <small v-if="isRanked" class="lobby-ranked-text">
            (Ranked <v-icon v-if="isRanked" size="medium">mdi-trophy</v-icon>)
          </small>
        </h1>
      </v-col>
    </v-row>
    <!-- Usernames -->
    <v-row>
      <v-col offset="1">
        <lobby-player-indicator
          :player-username="$store.state.auth.username"
          :player-ready="iAmReady"
          data-cy="my-indicator"
        />
      </v-col>
      <v-col offset="1">
        <audio ref="enterLobbySound" src="/sounds/lobby/enter-lobby.mp3"></audio>
        <audio ref="leaveLobbySound" src="/sounds/lobby/leave-lobby.mp3"></audio>
        <lobby-player-indicator
          :player-username="opponentUsername"
          :player-ready="opponentIsReady"
          data-cy="opponent-indicator"
        />
      </v-col>
    </v-row>
    <!-- Buttons -->
    <v-row class="mt-4">
      <v-spacer />
      <v-col cols="3" offset="1">
        <v-btn :disabled="readying" variant="outlined" color="primary" data-cy="exit-button" @click="leave">
          EXIT
        </v-btn>
      </v-col>
      <v-col cols="3">
        <v-btn :loading="readying" contained color="primary" data-cy="ready-button" @click="ready">
          {{ readyButtonText }}
          <v-icon
            v-if="isRanked"
            class="ml-1"
            size="small"
            icon="mdi-trophy"
            data-cy="ready-button-ranked-icon"
          />
        </v-btn>
      </v-col>
      <v-spacer />
    </v-row>
  </v-container>
</template>

<script>
import { mapGetters, mapState } from 'vuex';

import LobbyPlayerIndicator from '@/components/LobbyPlayerIndicator.vue';

export default {
  name: 'LobbyView',
  components: {
    LobbyPlayerIndicator,
  },
  data() {
    return {
      readying: false,
    };
  },
  watch: {
    opponentUsername(newVal) {
      if (newVal) {
        if (this.$refs.enterLobbySound.readyState === 4) {
          this.$refs.enterLobbySound.play();
        }
      } else {
        if (this.$refs.leaveLobbySound.readyState === 4) {
          this.$refs.leaveLobbySound.play();
        }
      }
    },
  },
  computed: {
    ...mapState({
      isRanked: ({ game }) => game.isRanked,
    }),
    ...mapGetters(['opponentIsReady', 'opponentUsername']),
    gameId() {
      return this.$store.state.game.id;
    },
    gameName() {
      return this.$store.state.game.name;
    },
    iAmReady() {
      return this.$store.state.game.myPNum === 0
        ? this.$store.state.game.p0Ready
        : this.$store.state.game.p1Ready;
    },
    readyButtonText() {
      return this.iAmReady ? 'UNREADY' : 'READY';
    },
  },
  mounted() {
    this.$store.dispatch('requestLobbyData');
  },
  methods: {
    async ready() {
      this.readying = true;
      await this.$store.dispatch('requestReady');
      this.readying = false;
    },
    leave() {
      this.$store
        .dispatch('requestLeaveLobby')
        .then(() => {
          this.$router.push('/');
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>

<style scoped lang="scss">
#logo {
  height: 10vh;
  min-height: 64px;
  margin: 0 auto;
}
.lobby-ranked-text {
  color: var(--v-neutral-darken2);
}
</style>
