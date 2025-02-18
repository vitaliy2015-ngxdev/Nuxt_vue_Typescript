<script lang="ts">
import ClubReservation from '@includes/models/ClubReservation'
import _ from 'lodash'
import { Component, InjectReactive } from 'nuxt-property-decorator'

@Component
export default class ReservationStepHostQuestions extends ClubReservation.Mixin {
	@InjectReactive({ from: 'reservation', default: () => ({}) })
	club_reservation!: any

	host_answers = []

	get host_questions() {
		return this.club_reservation_questions('host')
	}

	get required_questions() {
		return this.club_reservation_questions('host').filter((f) => f.is_required)
	}

	created() {
		this.host_answers = this.club_reservation.form_answers.filter(
			(f: any) => !f.is_per_attendee
		)
	}

	beforeDestroy() {
		this.club_reservation.form_answers = _.uniqBy(
			[...this.club_reservation.form_answers, ...this.host_answers],
			(t) => t.id
		)
	}

	required_questions_answered() {
		if (!this.required_questions) return true
		return this.host_answers.every((a: any) => {
			if (!a.is_required) return true
			return a.is_required && a.value.length
		})
	}
}
</script>

<template>
	<div class="step-page step-4">
		<div class="question">
			<h2>
				Please answer the following question(s) regarding your reservation ...
			</h2>
		</div>

		<div class="divider" />

		<div class="answers">
			<dynamic-form
				v-model="host_answers"
				:questions="host_questions"
				class="reservation-host-form"
			>
				<template #before>
					<div
						v-if="club_event.time_slots && club_event.time_slots.length"
						class="reservation-time-slot form-input form-select expanded flat"
					>
						<label for="time_slot" class="section-title input-label">
							What is your preferred seating time?
							<span class="input-required-mark">*</span>
						</label>

						<div class="input-area">
							<select v-model="club_reservation.time_slot" name="time_slot">
								<template v-for="(time, i) in club_event.time_slots">
									<option
										:key="i"
										:disabled="!time.available_seats"
										:value="time.id"
									>
										{{ time.title }}

										<template v-if="time.available_seats > 0">
											({{ time.available_seats }} seats available):
										</template>
										<template v-else>(no seats available):</template>

										{{ $moment(time.start_date).format('h:mma') }}
										-
										{{ $moment(time.end_date).format('h:mma') }}
									</option>
								</template>
							</select>
						</div>
					</div>
				</template>
			</dynamic-form>

			<div class="step-actions">
				<form-button
					class="ghost next"
					:disabled="!required_questions_answered()"
					@click="$emit('next')"
				>
					<span>Next</span>
				</form-button>

				<form-button
					v-if="!required_questions.length"
					class="ghost no-border skip"
					@click="$emit('next')"
				>
					<span>Skip</span>
				</form-button>
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@import '@sassy';

.step-page.step-4 {
	.question {
		@include below-desktop {
			font-size: 0.75em;
		}
	}

	.answers {
		width: 100%;
		max-height: 100%;

		overflow: hidden;
		overflow-y: auto;

		@include above-tablet {
			min-width: 450px;
			max-width: 450px;
			padding-right: 1em;
		}

		@include on-mobile {
			width: 90%;
			max-width: 600px;
		}
	}
} // step 1 page
</style>
