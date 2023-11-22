<template>
	<transition name="active" duration="400">
		<div v-if="win_team" :class="[`round-winner --${win_team} --${this.half}`,  { '--active': active }]">
			<div class="diagonal-wrapper --left">
				<div :class="`diagonal --${win_team}`"></div>
			</div>

			<div class="inner-wrapper">
				<div
					:class="`flag ${flagStyle(team.flag)} --${win_team}`"
					:style="{ backgroundImage: team.flag && `url('https://flagcdn.com/${team.flag.toLowerCase()}.svg')` }"
				></div>

				<div class="background"></div>

				<div :class="`inner --${win_team}`">
					<div class="phrase">
						WINS THE ROUND!
					</div>
				</div>
			</div>

			<div class="diagonal-wrapper --right">
				<div :class="`diagonal --${win_team}`"></div>
			</div>
		</div>
	</transition>
</template>

<script>
import { mapGetters } from 'vuex'
import flagStyle from '../flag-style'

export default {
	data() {
		return {
			win_team: false,
		}
	},

	props: [
		'directionalSides',
	],

	methods: {
		flagStyle,

		sync() {
			this.win_team = this.round.win_team ? this.round.win_team.toLowerCase() : this.round.win_team
			this.half = this.map.round > 15
		},
	},

	computed: {
		...mapGetters([
			'map',
			'round',
		]),

		active() {
			return this.round && this.round.phase === 'over'
		},

        left() {
			return this.map[`team_${this.directionalSides[0]}`]
		},

        right() {
            return this.map[`team_${this.directionalSides[1]}`]
        },

		team() {
			return this.map[`team_${this.win_team}`]
		},

		reason() {
			let latestRound = -1

			for (const round in this.map.round_wins) {
				const roundNum = Number(round)
				if (roundNum > latestRound) latestRound = roundNum
			}

			if (latestRound === -1) return
			if (this.map.round_wins[latestRound].endsWith('_elimination')) return 'by Eliminating the Enemy Team'
			if (this.map.round_wins[latestRound].endsWith('_bomb')) return 'by Detonating the Bomb'
			if (this.map.round_wins[latestRound].endsWith('_defuse')) return 'by Defusing the Bomb'
			if (this.map.round_wins[latestRound].endsWith('_time')) return 'by Running down the Clock'
		},
	},

	watch: {
		round(now) {
			if (now.phase === 'over') this.sync()
		},

		active(now, previously) {
			if (now) this.sync()
			else setTimeout(this.sync, 1000)
		},
	},
}
</script>