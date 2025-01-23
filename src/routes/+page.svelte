<script lang="ts">
    import Card from "./Card.svelte";
    import { onMount } from "svelte";
    import { browser } from "$app/environment";
    import { testData } from "$lib";

    interface Settings {
        maxPerOption: number;
    }

    interface Response {
        timestamp: string;
        language: string;
        name: string;
        apartment: string;
        phone: string;
        vegetarian: boolean;
        selections: Selection[];
    }

    interface Selection {
        optionKey: number;
        priority: number;
    }

    interface Option {
        key: number;
        label: string;
    }

    interface Schedule {
        optionKey: number;
        participantNames: string[];
    }

    let settings = $state<Settings>({ maxPerOption: 5 });
    const key = "groupify";

    let raw = $state("");
    let options = $derived(extractOptions(raw));
    let responses = $derived(parseAllResponses(raw));
    let schedules = $derived(scheduleParticipants(settings, options, responses));
    let evaluationCounts = $derived(participationCounts(schedules));

    $effect(() => {
        if (!browser) {
            return;
        }

        if (!raw?.length) {
            return;
        }

        localStorage.setItem(key, raw);
    });

    onMount(() => {
        if (!browser) {
            return;
        }

        const stored = localStorage.getItem(key);
        if (!stored) {
            return;
        }

        raw = stored;
    });

    function participationCounts(schedules: Schedule[]): { [name: string]: number } {
        const counts: { [name: string]: number } = {};
        for (let s of schedules) {
            for (const n of s.participantNames) {
                counts[n] = (counts[n] || 0) + 1;
            }
        }
        return counts;
    }

    function parseAllResponses(raw: string): Response[] {
        return raw.split("\n").slice(1).map(parseResponse).filter(r => r.name !== "");
    }

    function parseResponse(line: string): Response {
        let [ timestamp, language, name, apartment, phone, vegetarian, ...selections ] = line.split("\t");
        return {
            timestamp,
            language,
            name,
            apartment,
            phone,
            vegetarian: vegetarian === "Ja",
            selections: extractSelections(selections),
        };
    }

    function extractPriority(option: string): number {
        let match = option.match(/(\d+)\. Prioritet/);
        return match ? parseInt(match[1]) : 0;
    }

    function extractOptions(raw: string): Option[] {
        if (!raw?.length) {
            return [];
        }

        let matches = raw.match(/\[(.*?)\]/g);
        if (!matches?.length) {
            return [];
        }

        const result: Option[] = [];
        for (let i = 0; i < matches.length; i++) {
            let m = matches[i];
            result.push({ key: i, label: m.slice(1, -1) });
        }

        return result;
    }

    function extractSelections(selections: string[]): Selection[] {
        let result: Selection[] = [];
        for (let i = 0; i < selections.length; i++) {
            let priority = extractPriority(selections[i]);
            if (priority) {
                result.push({ optionKey: i, priority });
            }
        }
        return result;
    }

    function scheduleParticipants(settings: Settings, options: Option[], responses: Response[]): Schedule[] {
        let results = options.map(o => ({ optionKey: o.key, participantNames: [] }));
        for (let i = 0; i < options.length; i++) {
            let option = options[i];
            let schedule = results[i];
            let participants = responses.filter(r => r.selections.some(s => s.optionKey === option.key));
            let sorted = participants.sort((a, b) => a.selections.find(s => s.optionKey === option.key).priority - b.selections.find(s => s.optionKey === option.key).priority);
            for (let j = 0; j < sorted.length; j++) {
                let participant = sorted[j];
                if (schedule.participantNames.length < settings.maxPerOption) {
                    schedule.participantNames.push(participant.name);
                }
            }
        }
        return results;
    }
</script>

<div class="space-y-4">
    <label class="label">
        <span>Copy/pasta alle rækker og kolonner <button onclick={() => raw = testData}>her</button> (kun celler med data)</span>
        <textarea bind:value={raw} class="textarea" rows="4"></textarea>
    </label>

    <label class="label">
        <span>Max antal deltagere for en given mulighed</span>
        <input type="number" bind:value={settings.maxPerOption} class="input"/>
    </label>

    <Card title="Resultatet">
        {#each schedules as s}
            {@const o = options.find(o => o.key === s.optionKey)}
            {#if o}
                <div class="pb-4">
                    <h3 class="h3 text-primary-500">{o.label}</h3>
                    <div class="flex gap-4 flex-wrap">
                        {s.participantNames.join(", ")}.
                    </div>
                </div>
            {/if}
        {/each}
    </Card>

    <Card title="Antal middage per person">
        <pre class="pre">{JSON.stringify(evaluationCounts, null, 2)}</pre>
    </Card>

    <div class="table-container">
        <table class="table table-hover">
            <thead>
            <tr>
                <th>Navn</th>
                {#each options as option}
                    <th>{option.label}</th>
                {/each}
            </tr>
            </thead>
            <tbody>
            {#each responses as response}
                <tr>
                    <td>{response.name}</td>
                    {#each options as option}
                        <td>
                            {#each response.selections as selection}
                                {#if selection.optionKey === option.key}
                                    {selection.priority}
                                {/if}
                            {/each}
                        </td>
                    {/each}
                </tr>
            {/each}
            </tbody>
        </table>
    </div>
</div>
