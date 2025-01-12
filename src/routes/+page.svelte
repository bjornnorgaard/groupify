<script lang="ts">

    import { onMount } from "svelte";

    let raw = $state("Tidsstempel\tDansk / English\tFulde Navn\tLejligheds-/værelsesnummer\tTelefonnummer\tVegetar?\tPrioriter de dage, hvor du ønsker at deltage. [Mandag d. 10 februar]\tPrioriter de dage, hvor du ønsker at deltage. [Tirsdag d. 11. marts]\tPrioriter de dage, hvor du ønsker at deltage. [Onsdag d. 9. april]\tPrioriter de dage, hvor du ønsker at deltage. [Torsdag d. 8. maj]\n" +
        "\t\t\t\t\t\t\t\t\t\n" +
        "07/01/2025 21.02.56\tDansk\tEmma\t\t\tJa\t2. Prioritet\t\t1. Prioritet\t3. Prioritet\n" +
        "07/01/2025 21.04.07\tDansk\tAnja\t\t\tNej\t1. Prioritet\t2. Prioritet\t4. Prioritet\t4. Prioritet\n" +
        "07/01/2025 21.10.29\tDansk\tMaria\t\t\tNej\t1. Prioritet\t3. Prioritet\t\t2. Prioritet\n" +
        "07/01/2025 21.18.15\tDansk\tZacharias\t\t\tNej\t3. Prioritet\t1. Prioritet\t2. Prioritet\t4. Prioritet\n" +
        "08/01/2025 08.59.27\tDansk\tMarie\t\t\tJa\t2. Prioritet\t1. Prioritet\t3. Prioritet\t4. Prioritet\n" +
        "08/01/2025 10.01.30\tDansk\tMartin\t\t\tNej\t4. Prioritet\t1. Prioritet\t3. Prioritet\t3. Prioritet\n" +
        "08/01/2025 17.59.49\tDansk\tMorten\t\t\tNej\t\t2. Prioritet\t1. Prioritet\t1. Prioritet\n" +
        "09/01/2025 18.35.25\tDansk\tPeter\t\t\tJa\t\t1. Prioritet\t3. Prioritet\t2. Prioritet\n" +
        "11/01/2025 16.18.43\tDansk\tErik\t\t\tNej\t2. Prioritet\t1. Prioritet\t4. Prioritet\t1. Prioritet");

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

    let settings = $state<Settings>({ maxPerOption: 8, minPerOption: 5 });
    let responses = $state<Response[]>([]);
    let options = $state<Option[]>([]);

    onMount(() => {
        options = extractOptions();
        responses = parseAllResponses(raw);
    });

    function parseAllResponses(raw: string): Response[] {
        return raw.split("\n").slice(1).map(parseResponse).filter(r => r.name !== "");
    }

    function parseResponse(raw: string): Response {
        let [ timestamp, language, name, apartment, phone, vegetarian, ...selections ] = raw.split("\t");
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

    function extractOptions(): Option[] {
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
</script>

<div class="space-y-4">
    <label class="label">
        <span>Raw Input</span>
        <textarea bind:value={raw} class="textarea" rows="4"></textarea>
    </label>

    <div class="flex justify-between gap-4">
        <label class="w-full label">
            <span>Lower Limit</span>
            <input type="number" bind:value={settings.minPerOption} class="input"/>
        </label>

        <label class="label w-full">
            <span>Upper Limit</span>
            <input type="number" bind:value={settings.maxPerOption} class="input"/>
        </label>
    </div>

    <div class="flex flex-wrap gap-4">
    <span>Detected Options</span>
        {#each options as o}
            <li class="chip variant-filled-primary">{o.label}</li>
        {/each}
    </div>

    <div class="table-container">
        <table class="table table-hover">
            <thead>
            <tr>
                <!--            <th>Timestamp</th>-->
                <!--            <th>Language</th>-->
                <th>Name</th>
                <!--            <th>Apartment</th>-->
                <!--            <th>Phone</th>-->
                <!--            <th>Vegetarian</th>-->
                {#each options as option}
                    <th>{option.label}</th>
                {/each}
            </tr>
            </thead>
            <tbody>
            {#each responses as response}
                <tr>
                    <!--                <td>{response.timestamp}</td>-->
                    <!--                <td>{response.language}</td>-->
                    <td>{response.name}</td>
                    <!--                <td>{response.apartment}</td>-->
                    <!--                <td>{response.phone}</td>-->
                    <!--                <td>{response.vegetarian ? "Yes" : "No"}</td>-->
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
