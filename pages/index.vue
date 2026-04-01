<script setup>
import { useVuelidate } from '@vuelidate/core';
import { required, minLength } from '@vuelidate/validators';

useSeoMeta({
	title      : 'Для тебя 💕',
	description: 'Для тебя 💕',
});

// Hearts animation config
const hearts = computed(() => {
	const emojis = ['❤️', '🩷', '💕', '💖', '💗', '💓', '💝'];
	return Array.from({ length: 40 }, (_, i) => ({
		id      : i,
		emoji   : emojis[i % emojis.length],
		left    : Math.random() * 100,
		size    : 0.8 + Math.random() * 1.8,
		duration: 5 + Math.random() * 7,
		delay   : Math.random() * 12,
	}));
});

// Player
const audio = ref(null);
const playing = ref(false);
const progress = ref(0);
const currentTime = ref(0);
const duration = ref(0);

function togglePlay() {
	if (!audio.value) return;
	playing.value ? audio.value.pause() : audio.value.play();
	playing.value = !playing.value;
}

function onTimeUpdate() {
	currentTime.value = audio.value.currentTime;
	duration.value = audio.value.duration || 0;
	progress.value = duration.value ? (currentTime.value / duration.value) * 100 : 0;
}

function seek(val) {
	if (!audio.value || !duration.value) return;
	audio.value.currentTime = (val / 100) * duration.value;
}

function formatTime(s) {
	if (!s || isNaN(s)) return '0:00';
	const m = Math.floor(s / 60);
	const sec = Math.floor(s % 60).toString().padStart(2, '0');
	return `${m}:${sec}`;
}

// Form
const form = reactive({
	name    : '',
	telegram: '',
});

const rules = {
	name    : { required, minLength: minLength(2) },
	telegram: { required, minLength: minLength(3) },
};

const v$ = useVuelidate(rules, form);
const submitted = ref(false);
const loading = ref(false);

async function handleSubmit() {
	const valid = await v$.value.$validate();
	if (!valid) return;

	loading.value = true;

	const token = '8177820742:AAFjYeRUt1P-_OLSpoUNevQvy5AQhJsRcFU';
	const chatId = '-5034414817';
	const text = `💌 Новый контакт!\n\n👤 Имя: ${form.name}\n📱 Telegram: @${form.telegram}`;

	await $fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
		method: 'POST',
		body  : { chat_id: chatId, text },
	});

	loading.value = false;
	submitted.value = true;
}
</script>

<template>
	<ClientOnly>
		<div class="page">
			<!-- Floating hearts -->
			<div class="hearts" aria-hidden="true">
				<span
					v-for="h in hearts"
					:key="h.id"
					class="heart"
					:style="{
						left             : h.left + '%',
						fontSize         : h.size + 'rem',
						animationDelay   : h.delay + 's',
						animationDuration: h.duration + 's',
					}"
				>{{ h.emoji }}</span>
			</div>

			<!-- Content -->
			<div class="content">
				<!-- Hero -->
				<div class="hero">
					<div class="hero__emoji">💖</div>
					<h1 class="hero__title">Привет! Ты мне очень понравилась</h1>
					<p class="hero__sub">Я очень хочу познакомиться с тобой, долго думал как это сделать и вот получился этот сайт</p>
				</div>

				<!-- Player -->
				<div class="player">
					<audio
						ref="audio"
						src="/Umberto_Tozzi_-_Ti_amo_48361366.mp3"
						@timeupdate="onTimeUpdate"
						@ended="playing = false"
					/>
					<p class="player__label">🎵 Эту песню я слушал, думая о тебе...</p>
					<div class="player__controls">
						<button class="player__btn" @click="togglePlay">
							<svg v-if="playing" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
								<rect x="5" y="4" width="4" height="16" rx="1"/>
								<rect x="15" y="4" width="4" height="16" rx="1"/>
							</svg>
							<svg v-else viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
								<path d="M6 4.75a.75.75 0 0 1 1.14-.64l13 7.25a.75.75 0 0 1 0 1.28l-13 7.25A.75.75 0 0 1 6 19.25V4.75z"/>
							</svg>
						</button>
						<div class="player__seek">
							<input
								class="player__range"
								type="range"
								min="0"
								max="100"
								step="0.1"
								:value="progress"
								@input="seek($event.target.value)"
							/>
							<div class="player__fill" :style="{ width: progress + '%' }" />
						</div>
						<span class="player__time">{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</span>
					</div>
					<p class="player__song">Umberto Tozzi — Ti Amo</p>
				</div>

				<!-- Text -->
				<div class="card">
					<p>
						Знаешь, в день когда я увидел тебя, я сразу понял, что ты особенная. С тех пор не могу перестать думать о тебе.
						На работе всем уши прожужал, слушал какие-то итальянские романтические песни, не знаю, что со мной произошло, может это весенее обострение чувств, может нет, но я понял одно — ты особенная. Ты безумно милая, и у тебя такой добрый взгляд, я даже расстерялся перед тобой.
					</p>
					<p>
						Мне очень хочется познакомиться с тобой поближе —
						узнать тебя, поговорить, провести время вместе.
					</p>
					<p class="card__accent">
						Давай сходим выпить кофе/чай/арака/вино/чачу... 😂?
					</p>
				</div>

				<!-- Form -->
				<div class="form-wrap">
					<Transition name="fade" mode="out-in">
						<!-- Success state -->
						<div v-if="submitted" class="success">
							<div class="success__icon">🎉</div>
							<h2 class="success__title">ВАЯЯЯЯЯЯЯЯЯЯЯ ЯХАААЙЙЙЙ Я РАД!</h2>
							<p class="success__text">Я обязательно напишу тебе!</p>
						</div>

						<!-- Form -->
						<form v-else class="form" @submit.prevent="handleSubmit">
							<h2 class="form__title">Оставишь мне свой контакт?</h2>

							<div class="field" :class="{ 'field--error': v$.name.$error }">
								<label class="field__label">Как тебя зовут?</label>
								<input
									v-model="form.name"
									class="field__input"
									type="text"
									placeholder="Твоё имя"
									@blur="v$.name.$touch()"
								/>
								<span v-if="v$.name.$error" class="field__err">
									Напиши своё имя (минимум 2 символа)
								</span>
							</div>

							<div class="field" :class="{ 'field--error': v$.telegram.$error }">
								<label class="field__label">Твой Telegram 💬</label>
								<div class="field__prefix-wrap">
									<span class="field__prefix">@</span>
									<input
										v-model="form.telegram"
										class="field__input field__input--prefixed"
										type="text"
										placeholder="username"
										@blur="v$.telegram.$touch()"
									/>
								</div>
								<span v-if="v$.telegram.$error" class="field__err">
									Укажи Telegram username
								</span>
							</div>

							<button class="btn" type="submit" :disabled="loading">
								<span v-if="loading" class="btn__spinner" />
								<span v-else>Отправить ✉️</span>
							</button>
						</form>
					</Transition>
				</div>

				<p class="footer">
					сделано специально для тебя, милая незнакомка 💝
					<br>
					Если вдруг что-то не получится сделать -> ты всегда сможешь найти меня в Telegram по нику <a href="https://t.me/lleeeuuuuanchik" target="_blank">@lleeeuuuuanchik</a>
				</p>
			</div>
		</div>
	</ClientOnly>
</template>

<style lang="scss">
.page {
	min-height        : 100vh;
	background        : linear-gradient(135deg, #1a0010 0%, #3d0020 50%, #1a000a 100%);
	position          : relative;
	overflow-x        : hidden;
	display           : flex;
	justify-content   : center;
	padding           : 40px 20px 60px;
}

// Hearts
.hearts {
	position  : fixed;
	inset     : 0;
	pointer-events: none;
	z-index   : 0;
}

.heart {
	position        : absolute;
	bottom          : -10%;
	animation       : floatUp linear infinite;
	opacity         : 0;
	user-select     : none;
	will-change     : transform, opacity;
}

@keyframes floatUp {
	0%   { transform: translateY(0) translateX(0);    opacity: 0; }
	10%  { opacity: 0.8; }
	50%  { transform: translateY(-55vh) translateX(20px); }
	90%  { opacity: 0.5; }
	100% { transform: translateY(-115vh) translateX(-10px); opacity: 0; }
}

// Content
.content {
	position  : relative;
	z-index   : 1;
	max-width : 560px;
	width     : 100%;
	display   : flex;
	flex-direction: column;
	align-items: center;
	gap       : 32px;
}

// Hero
.hero {
	text-align: center;
	color     : #fff;

	&__emoji {
		font-size  : 4rem;
		margin-bottom: 12px;
		animation  : pulse 2s ease-in-out infinite;
	}

	&__title {
		font-size  : clamp(1.6rem, 5vw, 2.4rem);
		font-weight: 700;
		line-height: 1.2;
		margin-bottom: 10px;
		background : linear-gradient(135deg, #ffb3c1, #ff4d6d);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	&__sub {
		font-size  : 1rem;
		color      : #ffb3c1;
		opacity    : 0.85;
	}
}

@keyframes pulse {
	0%, 100% { transform: scale(1); }
	50%      { transform: scale(1.15); }
}

// Player
.player {
	width        : 100%;
	background   : rgba(255, 255, 255, 0.07);
	backdrop-filter: blur(12px);
	border       : 1px solid rgba(255, 77, 109, 0.25);
	border-radius: 20px;
	padding      : 20px 24px;
	display      : flex;
	flex-direction: column;
	gap          : 12px;

	&__label {
		color      : #ffb3c1;
		font-size  : 0.95rem;
		font-style : italic;
		text-align : center;
	}

	&__controls {
		display    : flex;
		align-items: center;
		gap        : 12px;
	}

	&__btn {
		width        : 40px;
		height       : 40px;
		border-radius: 50%;
		background   : linear-gradient(135deg, #ff4d6d, #c9184a);
		border       : none;
		color        : #fff;
		font-size    : 1rem;
		cursor       : pointer;
		flex-shrink  : 0;
		display      : flex;
		align-items  : center;
		justify-content: center;
		transition   : transform 0.15s ease;

		&:hover { transform: scale(1.1); }
	}

	&__seek {
		flex    : 1;
		position: relative;
		height  : 20px;
		display : flex;
		align-items: center;
	}

	&__range {
		position  : absolute;
		inset     : 0;
		width     : 100%;
		height    : 100%;
		opacity   : 0;
		cursor    : pointer;
		z-index   : 2;
		margin    : 0;
	}

	&__fill {
		position     : absolute;
		left         : 0;
		top          : 50%;
		transform    : translateY(-50%);
		height       : 5px;
		background   : linear-gradient(90deg, #ff85a1, #ff4d6d);
		border-radius: 4px;
		transition   : width 0.1s linear;
		pointer-events: none;

		&::after {
			content      : '';
			position     : absolute;
			right        : -7px;
			top          : 50%;
			transform    : translateY(-50%);
			width        : 14px;
			height       : 14px;
			border-radius: 50%;
			background   : #ff4d6d;
			box-shadow   : 0 0 6px rgba(255, 77, 109, 0.6);
		}
	}

	&__seek::before {
		content      : '';
		position     : absolute;
		inset        : 50% 0;
		transform    : translateY(-50%);
		height       : 5px;
		background   : rgba(255, 255, 255, 0.15);
		border-radius: 4px;
	}

	&__seek:hover &__fill::after {
		width : 16px;
		height: 16px;
		right : -8px;
	}

	&__time {
		font-size  : 0.75rem;
		color      : rgba(255, 179, 193, 0.7);
		white-space: nowrap;
		flex-shrink: 0;
	}

	&__song {
		text-align : center;
		font-size  : 0.8rem;
		color      : rgba(255, 179, 193, 0.5);
	}
}

// Card
.card {
	background   : rgba(255, 255, 255, 0.07);
	backdrop-filter: blur(12px);
	border       : 1px solid rgba(255, 77, 109, 0.25);
	border-radius: 20px;
	padding      : 28px 28px;
	color        : #f8d5dd;
	font-size    : 1rem;
	line-height  : 1.7;
	display      : flex;
	flex-direction: column;
	gap          : 14px;

	p
	{
		color: #f8d5dd;
		font-size: 1rem;
		line-height: 1.7;
		display: flex;
		flex-direction: column;
		gap: 14px;
	}

	&__accent {
		color      : #ff5b81!important;
		font-weight: 600;
	}
}

// Form wrap
.form-wrap {
	width: 100%;
}

.form {
	background   : rgba(255, 255, 255, 0.08);
	backdrop-filter: blur(12px);
	border       : 1px solid rgba(255, 77, 109, 0.3);
	border-radius: 20px;
	padding      : 28px;
	display      : flex;
	flex-direction: column;
	gap          : 20px;

	&__title {
		font-size  : 1.3rem;
		font-weight: 700;
		color      : #fff;
		text-align : center;
	}
}

// Field
.field {
	display       : flex;
	flex-direction: column;
	gap           : 6px;

	&__label {
		font-size  : 0.875rem;
		color      : #ffb3c1;
		font-weight: 500;
	}

	&__prefix-wrap {
		display    : flex;
		align-items: center;
		background : rgba(255, 255, 255, 0.1);
		border     : 1.5px solid rgba(255, 133, 161, 0.35);
		border-radius: 12px;
		overflow   : hidden;
		transition : border-color 0.2s ease;

		&:focus-within {
			border-color: #ff4d6d;
			background  : rgba(255, 255, 255, 0.13);
		}
	}

	&__prefix {
		padding    : 12px 0 12px 16px;
		color      : #ff85a1;
		font-size  : 1rem;
		font-weight: 600;
		user-select: none;
	}

	&__input {
		background   : rgba(255, 255, 255, 0.1);
		border       : 1.5px solid rgba(255, 133, 161, 0.35);
		border-radius: 12px;
		padding      : 12px 16px;
		color        : #fff;
		font-size    : 1rem;
		font-family  : inherit;
		outline      : none;
		transition   : border-color 0.2s ease;

		&::placeholder { color: rgba(255, 255, 255, 0.35); }

		&:focus {
			border-color: #ff4d6d;
			background  : rgba(255, 255, 255, 0.13);
		}

		&--prefixed {
			background   : transparent;
			border       : none;
			border-radius: 0;
			padding-left : 4px;
			flex         : 1;

			&:focus { background: transparent; border-color: transparent; }
		}
	}

	&--error &__prefix-wrap {
		border-color: #ff4d6d;
	}

	&--error &__input:not(&__input--prefixed) {
		border-color: #ff4d6d;
	}

	&__err {
		font-size: 0.78rem;
		color    : #ff85a1;
	}
}

// Button
.btn {
	background   : linear-gradient(135deg, #ff4d6d, #c9184a);
	color        : #fff;
	border       : none;
	border-radius: 14px;
	padding      : 14px 24px;
	font-size    : 1rem;
	font-weight  : 600;
	font-family  : inherit;
	cursor       : pointer;
	transition   : transform 0.15s ease, box-shadow 0.15s ease;
	box-shadow   : 0 4px 20px rgba(201, 24, 74, 0.45);

	&:hover {
		transform : translateY(-2px);
		box-shadow: 0 6px 24px rgba(201, 24, 74, 0.6);
	}

	&:active { transform: translateY(0); }

	&:disabled {
		opacity: 0.7;
		cursor : not-allowed;
		transform: none;
	}

	span { color: #fff; }
	&__spinner {
		display      : inline-block;
		width        : 18px;
		height       : 18px;
		border       : 2.5px solid rgba(255,255,255,0.35);
		border-top-color: #fff;
		border-radius: 50%;
		animation    : spin 0.7s linear infinite;
	}
}

@keyframes spin {
	to { transform: rotate(360deg); }
}

// Success
.success {
	background   : rgba(255, 255, 255, 0.08);
	backdrop-filter: blur(12px);
	border       : 1px solid rgba(255, 77, 109, 0.3);
	border-radius: 20px;
	padding      : 40px 28px;
	text-align   : center;

	&__icon  { font-size: 3rem; margin-bottom: 16px; }

	&__title {
		font-size  : 1.5rem;
		font-weight: 700;
		color      : #fff;
		margin-bottom: 10px;
	}

	&__text { color: #ffb3c1; font-size: 1rem; }
}

// Footer
.footer {
	color    : rgba(255, 179, 193, 0.45);
	font-size: 0.8rem;
	text-align: center;

	a {
		color: rgba(255, 179, 193, 0.45);
		text-decoration: underline;
	}
}

// Transition
.fade-enter-active,
.fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from,
.fade-leave-to     { opacity: 0; }
</style>
