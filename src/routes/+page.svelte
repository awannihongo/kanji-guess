<script lang="ts">
	import { Button } from '@/components/ui/button';
	import ThemeSwitcher from '../components/ThemeSwitcher.svelte';
	import JapaneseHighlight from '../components/JapaneseHighlight.svelte';
	import { fade } from 'svelte/transition';

	type Data = {
		kanji: string;
		jp: string;
		romaji: string;
		indonesia: string;

		exampleJp: string;
		exampleRomaji: string;
		exampleIndonesia: string;
	};

	let data = $state<Data[]>([]);
	let answers = $state<(boolean | null)[]>([]);
	let currentIndex = $state(0);
	let currentData = $derived.by<Data | null>(() => {
		if (data.length === 0 || currentIndex > data.length - 1) {
			return null;
		}
		return data[currentIndex];
	});
	let currentAnswer = $derived.by(() => {
		const score = answers.reduce((previousValue, currentValue) => {
			if (currentValue) {
				previousValue += 1;
			}
			return previousValue;
		}, 0);
		const percent = Math.floor((score / answers.length) * 100 || 0);

		return { score, percent };
	});
	let timeStart = $state(performance.now());
	let timeEnd = $state(performance.now());
	let isFlip = $state(false);
	let isFinish = $state(false);

	function shuffleArray<T>(array: T[]) {
		let arr = array.slice();
		for (let i = arr.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[arr[i], arr[j]] = [arr[j], arr[i]];
		}
		return arr;
	}

	function formatTime(ms: number) {
		const totalSeconds = Math.floor(ms / 1000);
		const minutes = Math.floor(totalSeconds / 60);
		const seconds = totalSeconds % 60;

		const formattedMinutes = String(minutes).padStart(2, '0');
		const formattedSeconds = String(seconds).padStart(2, '0');

		return `${formattedMinutes}:${formattedSeconds}`;
	}

	function onChangeUploadCsv(currentTarget: EventTarget & HTMLInputElement) {
		const file = currentTarget.files?.[0];
		if (!file) return;

		const reader = new FileReader();
		reader.onload = ({ target }) => {
			const result = target?.result;
			if (!result) return;
			const raw = result as string;
			const items = raw.split(/\r?\n/);

			if (items.length < 2) {
				return;
			}

			const cleanData = items.reduce<Data[]>((previousValue, currentValue, i) => {
				if (i === 0) {
					return previousValue;
				}

				const [kanji, jp, romaji, indonesia, exampleJp, exampleRomaji, exampleIndonesia] =
					currentValue.split(',');
				previousValue.push({
					kanji,
					jp,
					romaji,
					indonesia,
					exampleJp,
					exampleRomaji,
					exampleIndonesia
				});

				return previousValue;
			}, []);
			currentIndex = 0;
			data = shuffleArray(cleanData);
			answers = new Array(cleanData.length).fill(null);
			timeStart = performance.now();
			isFinish = false;

			currentTarget.value = '';
		};
		reader.readAsText(file);
	}

	function answer(value: boolean) {
		answers[currentIndex] = value;
		if (currentIndex === (data?.length ?? 1) - 1) {
			timeEnd = performance.now();
			isFinish = true;
		} else {
			currentIndex += 1;
		}
		isFlip = false;
	}
</script>

<div class="mx-auto flex min-h-dvh max-w-3xl flex-col gap-2 p-2">
	<div class="flex flex-row items-center justify-between">
		<div class="flex flex-row items-center justify-between gap-4">
			<ThemeSwitcher />
			<div class="font-semibold tracking-tight uppercase opacity-30">Awan Nihongo - Kanji</div>
		</div>
		<div>
			<div class="tabular-nums">
				{#if currentData}
					{currentIndex + 1} / {data.length}
				{/if}
			</div>
		</div>
	</div>
	{#if isFinish}
		<div class="flex grow flex-col justify-center gap-4">
			<div class="flex flex-row gap-6">
				<div class="grow text-right">
					<div class="text-2xl font-semibold tracking-tight uppercase">Skor</div>
					<div class="text-xl font-medium tabular-nums">
						{currentAnswer.score} / {data.length}
					</div>
				</div>
				<div class="grow">
					<div class="text-2xl font-semibold tracking-tight uppercase">Waktu</div>
					<div class="text-xl font-medium tabular-nums">
						{formatTime(timeEnd - timeStart)}
					</div>
				</div>
			</div>
			<div class="text-center text-4xl font-semibold tracking-tight tabular-nums">
				{currentAnswer.percent}%
			</div>
			<div class="flex flex-col gap-2 text-center">
				<div>
					<Button
						onclick={() => {
							isFinish = false;
							currentIndex = 0;
							timeStart = performance.now();
							data = shuffleArray(data);
						}}
					>
						Main Ulang
					</Button>
				</div>
				<div>
					<Button
						onclick={(e) => {
							const inputEl = e.currentTarget.nextElementSibling as HTMLInputElement;
							inputEl.click();
						}}
					>
						Buka File CSV
					</Button>
					<input
						type="file"
						class="hidden"
						accept="text/csv"
						onchange={({ currentTarget }) => {
							onChangeUploadCsv(currentTarget);
						}}
					/>
				</div>
			</div>
		</div>
	{:else if currentData}
		<div class="flex grow flex-col">
			<Button
				variant="outline"
				class="relative grow hover:bg-transparent"
				size="lg"
				onclick={() => {
					isFlip = !isFlip;
				}}
			>
				{#key currentIndex}
					<div
						class="front absolute inset-0 flex flex-col justify-center transition-transform duration-300 backface-hidden"
						class:active={isFlip}
						transition:fade
					>
						<div class="font-jp text-6xl md:text-8xl">{currentData.kanji}</div>
					</div>
					<div
						class="back absolute inset-0 flex flex-col justify-center transition-transform duration-300 backface-hidden"
						class:active={isFlip}
						transition:fade
					>
						<div class="flex flex-col gap-22">
							<div class="flex flex-col gap-4">
								<div class="px-4 text-center font-jp text-5xl text-red-500">
									{currentData.kanji}
								</div>
								<div class="px-4 text-center font-jp text-5xl">{currentData.jp}</div>
								<div class="px-4 text-center text-4xl opacity-20">{currentData.romaji}</div>
								<div class="px-4 text-center text-4xl">{currentData.indonesia}</div>
							</div>
							<div class="flex flex-col gap-2">
								<div class="px-4 text-center font-jp text-2xl text-wrap">
									<JapaneseHighlight string={currentData.exampleJp} target={currentData.kanji} />
								</div>
								<div class="px-4 text-center text-2xl text-wrap opacity-20">
									{currentData.exampleRomaji}
								</div>
								<div class="px-4 text-center text-2xl text-wrap">
									{currentData.exampleIndonesia}
								</div>
							</div>
						</div>
					</div>
				{/key}
			</Button>
		</div>
		<div class="flex flex-row gap-2">
			<Button
				variant="outline"
				class="grow text-xl"
				size="lg"
				onclick={() => {
					answer(false);
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="32"
					height="32"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="3"
					stroke-linecap="round"
					stroke-linejoin="round"
					class="lucide lucide-x-icon lucide-x"
				>
					<path d="M18 6 6 18" /><path d="m6 6 12 12" />
				</svg>
			</Button>
			<Button
				variant="outline"
				class="grow text-xl"
				size="lg"
				onclick={() => {
					answer(true);
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="32"
					height="32"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="3"
					stroke-linecap="round"
					stroke-linejoin="round"
					class="lucide lucide-check-icon lucide-check"
				>
					<path d="M20 6 9 17l-5-5" />
				</svg>
			</Button>
		</div>
	{:else}
		<div class="flex grow flex-col items-center justify-center gap-4">
			<!-- div class="uppercase">Drag and drop CSV file</div -->
			<Button
				class="font-main"
				onclick={(e) => {
					const inputEl = e.currentTarget.nextElementSibling as HTMLInputElement;
					inputEl.click();
				}}
			>
				Buka File CSV
			</Button>
			<input
				type="file"
				class="hidden"
				accept="text/csv"
				onchange={({ currentTarget }) => {
					onChangeUploadCsv(currentTarget);
				}}
			/>
		</div>
	{/if}
</div>

<style>
	.front {
		transform: rotateY(0deg);

		&.active {
			transform: rotateY(180deg);
		}
	}

	.back {
		transform: rotateY(180deg);

		&.active {
			transform: rotateY(0deg);
		}
	}
</style>
