<script lang="ts">
    import words from '$lib/words.json';
    import type { Character, Color, Letter } from '$lib/types.ts';
    import { getAllIndexes } from '$lib';

    let filteredWords = $state(words);
    const colors: Color[] = ['gray', 'yellow', 'green'];

    const word: Character[] = $state((() => {
        const tmp = [];
        for (let i = 0; i < 5; i++) {
            tmp.push({ letter: null, color: 'gray' as Color });
        }
        return tmp;
    })());
    const characterInputs: HTMLInputElement[] = $state([]);

    const previousWords: Character[][] = $state([]);
    const wordFilters: {letter: Letter, color: Color, index: number}[] = $state([]);

    function resetCurrentWord() {
        for (const character of word) {
            character.letter = null;
            character.color = 'gray';
        }
        for (const input of characterInputs) {
            input.value = '';
        }
    }

    function addWord() {
        const enteredWord = word.map(character => character.letter).join('');
        if (!words.includes(enteredWord)) {
            alert('Invalid word.');
            return;
        }

        const newWord: Character[] = [];
        for (let i = 0; i < word.length; i++) {
            const character = word[i];
            newWord.push({ ...character });
            wordFilters.push({ letter: character.letter!, color: character.color!, index: word.indexOf(character) });
        }
        previousWords.push(newWord);

        resetCurrentWord();
        applyFilter();
    }

    function handleInput(index: number) {
        try {
            let enteredLetter = characterInputs[index].value;
            if (enteredLetter === null) return;
            if (enteredLetter === '') {
                word[index].letter = null;
                characterInputs[index].value = '';
                return;
            }

            enteredLetter = enteredLetter.trim().toUpperCase();
            if (enteredLetter < 'A' || enteredLetter > 'Z') {
                characterInputs[index].value = word[index].letter ?? '';
                return;
            }

            word[index].letter = enteredLetter as Letter;
            characterInputs[index].value = word[index].letter;
            if (index < word.length - 1) {
                characterInputs[index + 1].focus();
            }
        } catch {
            word[index].letter = null;
            characterInputs[index].value = '';
        }
    }

    function applyFilter() {
        filteredWords = filteredWords.filter(word => {
            // check there aren't any letters that shouldn't be there
            for (const letter of word) {
                const filters = wordFilters.filter(filter => filter.letter === letter);
                for (const filter of filters) {
                    if (filter.color === 'gray') {
                        return false;
                    }

                    if (filter.color === 'yellow' && filter.index === word.indexOf(letter)) {
                        return false;
                    }
                }
            }

            // check all letters that should be there are there
            for (const filter of wordFilters.filter(filter => filter.color !== 'gray')) {
                if (!word.includes(filter.letter)) {
                    return false;
                }

                if (filter.color === 'green' && !getAllIndexes(word, filter.letter).includes(filter.index)) {
                    return false;
                }
            }
            return true;
        });
    }
</script>

<div class="main">
    <h1>Enter a word and the response you got.</h1>
    <form onsubmit={addWord}>
        <div class="word">
            {#each word as letter, index}
                <div>
                    <input type="text" oninput={() => handleInput(index)} bind:this={characterInputs[index]} maxlength="1" autocomplete="off" required />
                    {#each colors as color}
                        <label class={`color-input ${color}`} for={`color-${index}-${color}`}>
                            <input type="radio" id={`color-${index}-${color}`} name={`color-${index}`} class={`color-input ${color}`} value={color} bind:group={letter.color} />
                        </label>
                    {/each}
                </div>
            {/each}
        </div>
        <button type="submit">Add Word</button>
    </form>
    <div class="previous-words">
        <h1>Previous Words</h1>
        <div class="words">
            {#each previousWords as word, index}
                <div class="previous-word">
                    {#each word as letter}
                        <span class="{letter.color}">{letter.letter}</span>
                    {/each}
                </div>
            {/each}
        </div>
    </div>
    <div class="possible-words">
        <h1>Possible Words</h1>
        <div class="words">
            {#each filteredWords as word}
                <span>{word}</span>
            {/each}
        </div>
    </div>
</div>

<style>
    :global(body) {
        background-color: #121212;
    }
    
    .main {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        align-items: center;
        margin-top: 2rem;
        font-family: monospace;
        color: white;
    }

    h1 {
        text-align: center;
    }

    form {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        align-items: center;
    }

    .word {
        display: flex;
        flex-direction: row;
        gap: 0.5rem;
        justify-content: center;
    }
    
    .word div {
        display: flex;
        flex-direction: column;
        gap: 0.4rem;
    }

    div input {
        width: 1rem;
        text-align: center;
    }
    
    .color-input {
        width: 100%;
        aspect-ratio: 1 / 1;
        margin: 0;
        border-radius: 2px;
    }

    .color-input:has(:checked) {
        outline: 0.1rem solid white;
    }

    input[type="radio"] {
        display: none;
    }

    .words {
        display: flex;
        gap: 0.5rem;
        justify-content: center;
        flex-direction: column;
        align-items: center;
    }

    .previous-word {
        display: flex;
        gap: 1rem;
        align-items: center;
    }

    .previous-word span {
        font-size: 1rem;
        width: 1rem;
        height: 1rem;
        text-align: center;
        outline: 0.1rem solid white;
    }

    .gray {
        background-color: #3a3a3c;
    }

    .yellow {
        background-color: #b59f3b;
    }

    .green {
        background-color: #538d4e;
    }
</style>
