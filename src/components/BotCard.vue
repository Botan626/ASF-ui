<template>
  <div class="bot" :class="[`status--${bot.status}`, { 'bot--big': selectedButtons.length > 2 }]">
    <a v-if="bot.steamid !== '0'" target="_blank" rel="noreferrer noopener" :href="bot.profileURL">
      <img class="bot__avatar" :src="bot.avatarURL" :alt="bot.name">
    </a>
    <router-link v-else v-slot="{ navigate }" custom :to="{ name: 'bot', params: { bot: bot.name } }">
      <img class="bot__avatar" :src="bot.avatarURL" @click="navigate">
    </router-link>

    <router-link v-slot="{ navigate }" custom :to="{ name: 'bot', params: { bot: bot.name } }">
      <div class="bot__status" @click="navigate">
        <span :title="bot.name" class="bot__status-property bot__status-property--name">{{ bot.viewableName }}</span>
        <span class="bot__status-property bot__status-property--text">{{ bot.statusText }}</span>
      </div>
    </router-link>

    <div class="bot__actions">
      <router-link v-for="button in selectedButtons" v-if="button.name !== 'pause'" :key="button.name" :to="{ name: `bot-${button.name}`, params: { bot: bot.name } }">
        <span class="bot__action"><FontAwesomeIcon :icon="button.icon"></FontAwesomeIcon></span>
      </router-link>

      <span v-if="bot.paused && bot.active && isPauseButtonSelected" class="bot__action" @click="resume"><FontAwesomeIcon icon="play"></FontAwesomeIcon></span>
      <span v-if="!bot.paused && bot.active && isPauseButtonSelected" class="bot__action" @click="pause"><FontAwesomeIcon icon="pause"></FontAwesomeIcon></span>

      <span v-if="!bot.active" class="bot__action" @click="start"><FontAwesomeIcon icon="power-off"></FontAwesomeIcon></span>
      <span v-if="bot.active" class="bot__action" @click="stop"><FontAwesomeIcon icon="power-off"></FontAwesomeIcon></span>
    </div>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex';
  import getUserInputType from '../utils/getUserInputType';

  const quickActionButtons = [
    { name: '2fa', icon: 'lock' },
    { name: 'bgr', icon: 'key' },
    { name: 'config', icon: 'wrench' },
    { name: 'pause', icon: 'none' },
  ];

  export default {
    name: 'BotCard',
    props: {
      bot: Object,
    },
    computed: {
      ...mapGetters({
        favButtons: 'settings/favButtons',
        headless: 'asf/headless',
      }),
      isPauseButtonSelected() {
        return this.selectedButtons.filter(e => e.name === 'pause').length > 0;
      },
      selectedButtons() {
        return Array.from(this.favButtons.toString(2))
          .reverse()
          .reduce((activeButtons, enabled, index) => {
            if (enabled === '1') activeButtons.push(quickActionButtons[index]);
            return activeButtons;
          }, []);
      },
    },
    methods: {
      async pause() {
        try {
          await this.$http.botAction(this.bot.name, 'pause', { permanent: true, resumeInSeconds: 0 });
          await this.$store.dispatch('bots/updateBot', { name: this.bot.name, paused: true });
        } catch (err) {
          this.$error(err.message);
        }
      },
      async resume() {
        try {
          await this.$http.botAction(this.bot.name, 'resume');
          await this.$store.dispatch('bots/updateBot', { name: this.bot.name, paused: false });
        } catch (err) {
          this.$error(err.message);
        }
      },
      async start() {
        try {
          const inputType = getUserInputType(this.bot.requiredInput);

          if (this.headless && inputType !== 'None') {
            this.$router.push({ name: 'bot-input', params: { bot: this.bot.name, type: inputType } });
            return;
          }

          await this.$http.botAction(this.bot.name, 'start');
          await this.$store.dispatch('bots/updateBot', { name: this.bot.name, active: true });
        } catch (err) {
          this.$error(err.message);
        }
      },
      async stop() {
        try {
          await this.$http.botAction(this.bot.name, 'stop');
          await this.$store.dispatch('bots/updateBot', { name: this.bot.name, active: false });
        } catch (err) {
          this.$error(err.message);
        }
      },
    },
  };
</script>

<style lang="scss">
	.bot {
		background: var(--color-background-light);
		border-radius: 0 0 4px 4px;
		border-top: 3px solid var(--color-status);
		display: grid;
		grid-template-areas: 'avatar meta buttons';
		grid-template-columns: min-content 1fr auto;
		padding: 0.5em;
		transition: border .3s;

		&--big {
			grid-template-areas: 'avatar meta' 'buttons buttons';
			grid-template-columns: min-content 1fr;
			grid-template-rows: auto min-content;

			.bot__actions {
				margin-top: 0.5rem;
			}

			.bot__action {
				font-size: 1.25rem;
				padding: 0 0.5rem;
			}
		}
	}

	.bot__avatar {
		cursor: pointer;
		display: block;
		height: 2.25em; // (1em + 0.8em) * 1.25
		padding-right: 0.5em;
		width: 2.25em;
	}

	.bot__status {
		cursor: pointer;
		display: flex;
		flex-direction: column;
		overflow: hidden;
	}

	.bot__status-property {
		display: inline-block;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.bot__status-property--name {
		font-weight: 600;
	}

	.bot__status-property--text {
		font-size: 0.8em;
		font-style: italic;
	}

	.bot__actions {
		align-items: center;
		display: flex;
		grid-area: buttons;
		justify-content: space-around;
	}

	.bot__action {
		color: var(--color-text-disabled);
		cursor: pointer;
		display: block;
		padding: 0.5em;
		transition: color .3s;

		&:hover {
			color: var(--color-text-dark);

			.app--dark-mode & {
				color: var(--color-text);
			}
		}
	}
</style>
