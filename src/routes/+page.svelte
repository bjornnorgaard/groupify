<script lang="ts">
    import Card from "./Card.svelte";
    import { testData } from "$lib";

    interface Settings {
        maxPerOption: number;
        minPerOption: number;
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

    let raw = $state("");
    let options = $derived(extractOptions(raw));
    let responses = $derived(parseAllResponses(raw));
    let settings = $state<Settings>({ maxPerOption: 8, minPerOption: 4 });
    let schedules = $derived(scheduleParticipants(settings, options, responses));
    let unscheduled = $state([]);

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
        let addedNames = new Set<string>();

        for (let r of responses) {
            for (let p of r.selections.sort((a, b) => a.priority - b.priority)) {
                if (!addedNames.has(r.name)) {
                    let option = results.find(o => o.optionKey === p.optionKey);
                    if (option && option.participantNames.length < settings.maxPerOption) {
                        option.participantNames.push(r.name);
                        addedNames.add(r.name);
                        break;
                    }
                }
            }
        }

        for (let result of results) {
            if (result.participantNames.length < settings.minPerOption) {
                for (let otherResult of results) {
                    if (otherResult.participantNames.length > settings.minPerOption) {
                        while (result.participantNames.length < settings.minPerOption && otherResult.participantNames.length > settings.minPerOption) {
                            let name = otherResult.participantNames.pop();
                            if (name) {
                                result.participantNames.push(name);
                                addedNames.delete(name);
                            }
                        }
                    }
                    if (result.participantNames.length >= settings.minPerOption) {
                        break;
                    }
                }
            }
        }

        return results;
    }

</script>

<div class="space-y-4">
    <label class="label">
        <span>Copy/pasta alle rækker og kolonner <button onclick={() => raw = testData} class="text-primary-500">her</button> (kun celler med data)</span>
        <textarea bind:value={raw} class="textarea" rows="4"></textarea>
    </label>

    <label class="label">
        <span>Max antal deltagere for en given mulighed</span>
        <input type="number" bind:value={settings.maxPerOption} class="input"/>
    </label>

    <label class="label">
        <span>min deltagere for en mulighed</span>
        <input type="number" bind:value={settings.minPerOption} class="input"/>
    </label>

    <Card title="Resultatet">
        {#each schedules as s}
            {@const o = options.find(o => o.key === s.optionKey)}
            {#if o}
                <div class="pb-4">
                    <h3 class="h3 text-primary-500">{o.label}</h3>
                    <div class="flex flex-wrap gap-4">
                        {s.participantNames.join(", ")}.
                    </div>
                </div>
            {/if}
        {/each}
    </Card>

    {#if unscheduled.length !== 0}
        <pre>{JSON.stringify({missing: unscheduled}, null, 2)}</pre>
    {/if}

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
