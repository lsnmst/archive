<script>
    import { createEventDispatcher } from "svelte";
    export let item;
    const dispatch = createEventDispatcher();
    $: isFeatured = item.featured?.toLowerCase() === "yes";
    $: hasCollection = !!item.collection && item.collection.trim() !== "";
    $: hasAnnotation = !!item.annotation && item.annotation.trim() !== "";
</script>

<tr>
    <td class="coll" class:collection={hasCollection} data-label="">
        {#if hasCollection}
            <span on:click={() => dispatch("collectionClick", item.collection)}>
                {item.collection}
            </span>
        {/if}
    </td>

    <td class="type" data-label="">
        <span
            class="clickable"
            on:click={() => dispatch("filterType", item.type)}
        >
            {item.type}
        </span>
    </td>

    <td class="terms" data-label="">
        {#each item.terms as term, i}
            <span
                class="clickable"
                on:click={() => dispatch("filterTerm", term)}
            >
                {term}
            </span>{#if i < item.terms.length - 1}{/if}
        {/each}
    </td>

    <td class="title" class:highlight={isFeatured} data-label="">
        <a href={item.url} target="_blank" rel="noopener noreferrer">
            {item.title}
        </a><span class="code"> {item.countrycode}</span>
        {#if isFeatured}
            <div class="featured-box">
                {#if item.featureImage}
                    <img
                        src={item.featureImage}
                        alt="Featured"
                        class="featured-img"
                    />
                {/if}
                {#if item.featureDescription}
                    <span class="featured-desc"
                        >{@html item.featureDescription}</span
                    >
                {/if}
            </div>
        {/if}
    </td>

    <td class:highlight={isFeatured} data-label="with" data-authors-cell>
        {#each item.authors as author, i}
            <span
                class="clickable"
                on:click={() => dispatch("authorClick", author)}
            >
                {author}
            </span>{i < item.authors.length - 1 ? "" : ""}
        {/each}
    </td>

    <td class:highlight={isFeatured} data-label="">
        {item.year}
    </td>
    {#if hasAnnotation}
        <td class="annotation" data-label="">
            {item.annotation}
        </td>
    {/if}
</tr>

<style>
    td {
        padding: 0.5rem 0.75rem;
        border-bottom: 1px solid #e0e0e0;
        vertical-align: top;
        font-size: 1rem;
        line-height: 1.1rem;
        color: #555;
    }
    .highlight {
        background: rgba(242, 228, 255, 0);
    }
    td.collection:not(:empty) {
        background: rgba(214, 242, 255, 1) !important;
        cursor: pointer;
    }
    td.collection span:not(:empty) {
        background: rgba(214, 242, 255, 1);
    }
    td.annotation:not(:empty) {
        background: rgba(208, 255, 228, 1);
    }
    .collection,
    .annotation {
        width: 10%;
        font-size: 0.9rem;
        line-height: 1rem;
        color: #555;
    }
    td.coll,
    td.annotation,
    .coll,
    .annotation {
        border: none !important;
    }
    .authors,
    .type,
    .terms,
    .year {
        width: 5vw;
        max-width: 10vw;
        color: #444;
    }
    .title {
        font-style: italic;
        font-size: 1.5rem;
        line-height: 1.4rem;
        width: 30vw;
    }
    .featured-box {
        margin-top: 0.5rem;
        display: flex;
        max-width: 100%;
        overflow: hidden;
        align-items: center;
        gap: 0.5rem;
        padding: 0.25rem 0.5rem;
        border-radius: 4px;
        border: 1px solid #e0e0e0;
    }
    .featured-img {
        height: auto;
        width: 200px;
        object-fit: cover;
    }
    .featured-desc {
        font-size: 1rem !important;
        line-height: 1rem;
        text-align: left;
        color: #444;
        font-style: italic;
    }
    .clickable {
        display: inline-block;
        margin-right: 0.25em;
        cursor: pointer;
        color: #444;
        text-decoration: underline;
    }
    .clickable:hover {
        color: #a255e9;
    }
    .code {
        font-size: 0.7rem;
        margin-left: 10px;
        padding: 5px;
        color: #a255e9;
        background-color: #f2e4ff;
        max-width: 20px;
        max-height: 20px;
    }

    @media (max-width: 768px) {
        table,
        thead,
        tbody,
        th,
        td,
        tr {
            display: block;
            max-width: 100%;
            background-color: #e0e0e000 !important;
        }

        thead {
            display: none;
        }

        tr {
            margin-bottom: 1.5rem;
            border: 1px solid #ccc;
            padding: 1em;
            border-radius: 6px;
            background: #fff;
            margin: auto;
            text-align: right;
        }

        td {
            display: flex;
            justify-content: space-between;
            padding: 0.5rem 0;
            border: none;
            border-bottom: 1px solid #eee;
        }

        td[data-authors-cell]:empty {
            display: none;
        }

        td::before {
            content: attr(data-label);
            font-weight: bold;
            flex-basis: 30vw;
            text-align: left;
            color: #444;
        }

        td:last-child {
            border-bottom: none;
        }
        .clickable {
            text-decoration: none;
        }
        .featured-box {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 0.5rem;
            border: none;
            gap: 0.5rem;
        }
        .featured-img {
            max-width: 90%;
            height: auto;
            display: block;
            margin: 0 auto;
        }
        .featured-desc {
            display: none;
        }
        .highlight {
            background: none;
        }
        .terms {
            display: none;
        }
        .annotation {
            width: 88vw;
            padding: 10px;
            background: rgba(208, 255, 228, 1) !important;
        }

        .type {
            width: unset;
            max-width: unset;
        }
        .title {
            font-size: 1.7em;
            width: 88vw;
            display: flow-root;
        }
        .collection {
            width: 88vw;
            padding: 10px;
        }
        .clickable {
            margin-right: 1em;
        }
    }
</style>
