<template>
	<div :class="`sidebar --${direction} --${side}`">
		<div :class="`iem --${direction}`">
			<img class="iem-logo" :src="require(`../../img/assets/iem.png`).default">
		</div>
		<div :class="`esl --${direction}`">
			<img class="esl-logo" :src="require(`../../img/assets/esl.png`).default">
			<div class="esl-text">
				IEM Sydney 2018<br>Quarter Finals 2    
			</div>
		</div>
	
		<SidebarPlayer
			v-for="(player, index) in players"
			:adr="adr[player.steamid]"
			:direction="direction"
			:freezetime="freezetime"
			:key="player.steamid"
			:observerSlot="observerSlotSortingEnabled ? (index + (direction === 'right' ? 6 : 1)) : player.observer_slot"
			:player="player"
			:side="side"
		/>
	</div>
</template>

<script>
import { mapGetters } from 'vuex'
import SidebarPlayer from './SidebarPlayer'
import WeaponIcon from './WeaponIcon'

export default {
	components: {
		SidebarPlayer,
		WeaponIcon,
	},

	props: [
		'adr',
		'direction',
		'freezetime',
		'players',
		'side',
	],

	methods: {
		image(str) {
			return decodeURIComponent(str.replace(/^data:image\/svg\+xml,/, ''))
		},
	},

	computed: {
		...mapGetters([
			'map',
			'observerSlotSortingEnabled',
		]),

		equipmentValue() {
			return this.players.reduce((sum, player) => sum + player.state.equip_value, 0)
		},

		teamMoney() {
			return this.players.reduce((sum, player) => sum + player.state.money, 0)
		},

		lossBonusLevel() {
			return this.map[`team_${this.side}`].consecutive_round_losses
		},

		grenades() {
			const grenades = { smoke: 0, molotov: 0, flash: 0, he: 0 }

			for (const player of this.players) {
				for (const weapon in player.weapons) {
					const { name } = player.weapons[weapon]

					switch (name) {
						case 'weapon_smokegrenade':
							grenades.smoke++
							break

						case 'weapon_molotov':
						case 'weapon_incgrenade':
							grenades.molotov++
							break

						case 'weapon_flashbang':
							grenades.flash++
							break

						case 'weapon_hegrenade':
							grenades.he++
							break
					}
				}
			}

			return grenades
		},

		grenadeTotal() {
			return Object.values(this.grenades).reduce((carry, item) => {
				return carry + item
			})
		},
	},
}
</script>
