<script lang="ts">
	interface Props {
		string: string;
		target: string;
	}

	let { string: str, target }: Props = $props();
	let words = $derived.by(() => {
		const result = [];
		let startIndex = 0;

		while (startIndex < str.length) {
			const targetIndex = str.indexOf(target, startIndex);

			if (targetIndex === -1) {
				result.push(str.slice(startIndex));
				break;
			}

			result.push(str.slice(startIndex, targetIndex));
			result.push(target);
			startIndex = targetIndex + target.length;
		}

		return result;
	});
</script>

{#each words as word}
	{#if word === target}
		<span class="text-red-500">{word}</span>
	{:else}
		{word}
	{/if}
{/each}
