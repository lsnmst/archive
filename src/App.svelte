<script>
  import { onMount } from "svelte";
  import ArchiveCard from "./components/ArchiveCard.svelte";
  import AuthorPopup from "./components/AuthorPopup.svelte";
  import CollectionPopup from "./components/CollectionPopup.svelte";
  import data from "./data.js";
  import bios from "./bios";
  import collection from "./collection.js";

  let sortKey = "year";
  let ascending = false;
  let search = "";
  let selectedType = "";
  let selectedTerms = new Set();
  let selectedLanguages = "";
  let selectedLoc = "";
  let selectedAuthor = null;
  let showPopup = false;
  let selectedCollection = null;
  let showCollectionPopup = false;

  let mounted = false;

  onMount(() => {
    const params = new URLSearchParams(window.location.search);
    search = params.get("search") || "";
    selectedType = params.get("type") || "";
    selectedLanguages = params.get("lang") || "";
    selectedLoc = params.get("loc") || "";
    const terms = params.get("terms");
    selectedTerms = terms ? new Set(terms.split(",")) : new Set();

    // Force initial sort by year descending
    sortKey = "year";
    ascending = false;
    mounted = true;
  });

  export let author = "";
  export let coll = "";
  export let items = [];

  function handleAuthorClick(author) {
    selectedAuthor = {
      name: author,
      bio: bios[author] || "Biography not available.",
      items: data.filter(
        (item) => Array.isArray(item.authors) && item.authors.includes(author),
      ),
    };
    showPopup = true;
  }

  function handleCollectionClick(collectionName) {
    selectedCollection = {
      name: collectionName,
      coll: collection[collectionName] || "Description not available.",
      items: data.filter((item) => item.collection === collectionName),
    };
    showCollectionPopup = true;
  }

  function closePopup() {
    showPopup = false;
    selectedAuthor = null;
    showCollectionPopup = false;
    selectedCollection = null;
  }

  const allTypes = [...new Set(data.map((d) => d.type))].sort((a, b) =>
    a.localeCompare(b),
  );
  const allTerms = [...new Set(data.flatMap((d) => d.terms))];
  const visibleTerms = [
    "Traditional ecological knowledge",
    "Women rights",
    "Cartography studies",
  ];
  const allLanguages = [...new Set(data.flatMap((d) => d.languages))].sort(
    (a, b) => a.localeCompare(b),
  );
  const allLoc = [...new Set(data.flatMap((d) => d.location))].sort((a, b) =>
    a.localeCompare(b),
  );

  function toggle(set, val) {
    const next = new Set(set);
    next.has(val) ? next.delete(val) : next.add(val);
    return next;
  }

  $: filteredData = data.filter((item) => {
    const searchText = [
      item.title,
      item.authors?.join(" "),
      item.terms?.join(" "),
      item.languages?.join(" "),
      item.locations?.join(" "),
      item.type,
      item.annotation,
    ]
      .join(" ")
      .toLowerCase();

    const matchSearch = !search || searchText.includes(search.toLowerCase());
    const matchType = !selectedType || item.type === selectedType;
    const matchTerms =
      !selectedTerms.size || item.terms.some((t) => selectedTerms.has(t));
    const matchLang =
      !selectedLanguages ||
      (Array.isArray(item.languages) &&
        item.languages.includes(selectedLanguages));
    const matchLoc =
      !selectedLoc ||
      (Array.isArray(item.location) && item.location.includes(selectedLoc));

    return matchSearch && matchType && matchTerms && matchLang && matchLoc;
  });

  $: sortedData = sortKey
    ? [...filteredData].sort((a, b) => {
        let valA = a[sortKey];
        let valB = b[sortKey];

        // Sort numbers numerically
        if (!isNaN(valA) && !isNaN(valB)) {
          return ascending ? valA - valB : valB - valA;
        }

        // Fallback to string sort
        valA = getSortValue(valA);
        valB = getSortValue(valB);
        return ascending ? valA.localeCompare(valB) : valB.localeCompare(valA);
      })
    : [...filteredData];

  function getSortValue(value) {
    if (Array.isArray(value)) return value.join(", ").toLowerCase();
    return (value ?? "").toString().toLowerCase();
  }

  function sortBy(key) {
    if (sortKey === key) {
      ascending = !ascending;
    } else {
      sortKey = key;
      ascending = true;
    }
  }

  $: if (mounted) updateURL();

  function updateURL() {
    const params = new URLSearchParams();
    if (search) params.set("search", search);
    if (selectedType) params.set("type", selectedType);
    if (selectedLanguages) params.set("lang", selectedLanguages);
    if (selectedLoc) params.set("loc", selectedLoc);
    if (selectedTerms.size) params.set("terms", [...selectedTerms].join(","));

    const query = params.toString();
    const newURL = query
      ? `${window.location.pathname}?${query}`
      : window.location.pathname;
    history.replaceState(null, "", newURL);
  }
</script>

<div class="header-section">
  <div class="header-title">
    <h1>Alessandro</h1>
  </div>
  <div class="header-description">
    <p>
      Amplify voices from marginalised communities, building solidariety through
      collective actions.
    </p>
  </div>
</div>
<div class="archive-section">
  <p>
    <b>Why an archive instead of a promo? </b><br />If you were expecting a
    promo, you've hit an archive. The archive is a personal, intimate,
    authentic, sentimental, non-romanticised, political, urgent, grave
    alternative to a promo, designed to learn about myself and inspire others to
    build together. Aimed at organise ideas, imagine new paths, celebrate fellow
    comrades and build bridges. It opposes the short and simple argument,
    requiring time and an exploratory strategy, along with a generous share of
    curiosity. Fair winds.
  </p>
  <p style="font-size: 0.9rem;">
    <b>Get in touch</b><br />hello@alessandromusetta.com <br /><a
      href="/archive/HgauJwTzvNjsxVS2P3oJX.asc">PGP KEY</a
    ><br />FINGERPRINT: D43A CD45 175E 79EA F11F D448 C90C 1302 EDCB 1889
  </p>
</div>

<div class="filters">
  <input type="text" placeholder="Search…" bind:value={search} />

  <select bind:value={selectedType}>
    <option value="">All Types</option>
    {#each allTypes as type}
      <option value={type}>{type}</option>
    {/each}
  </select>

  <div class="multi-select">
    <div class="filter-title">Featured terms:</div>
    {#each visibleTerms as term}
      <span
        class:selected={selectedTerms.has(term)}
        on:click={() => (selectedTerms = toggle(selectedTerms, term))}
      >
        {term}
      </span>
    {/each}
  </div>

  <select bind:value={selectedLanguages}>
    <option value="">All Languages</option>
    {#each allLanguages as lang}
      <option value={lang}>{lang}</option>
    {/each}
  </select>

  <select bind:value={selectedLoc}>
    <option value="">All Locations</option>
    {#each allLoc as loc}
      <option value={loc}>{loc}</option>
    {/each}
  </select>
</div>

<main class="archive">
  <table>
    <thead>
      <tr>
        <th class="non-sortable">Collection</th>
        <th class="sortable" on:click={() => sortBy("type")}>
          Type <span>{sortKey === "type" ? (ascending ? "▲" : "▼") : "▲▼"}</span
          >
        </th>
        <th class="non-sortable">Terms</th>
        <th class="sortable" on:click={() => sortBy("title")}>
          Title <span
            >{sortKey === "title" ? (ascending ? "▲" : "▼") : "▲▼"}</span
          >
        </th>
        <th class="sortable" on:click={() => sortBy("authors")}>
          Co-actors <span
            >{sortKey === "authors" ? (ascending ? "▲" : "▼") : "▲▼"}</span
          >
        </th>
        <th class="sortable" on:click={() => sortBy("year")}>
          Year<span>{sortKey === "year" ? (ascending ? "▲" : "▼") : "▲▼"}</span>
        </th>
        <th class="non-sortable">Annotation</th>
      </tr>
    </thead>
    <tbody>
      {#if sortedData.length > 0}
        {#each sortedData as item}
          <ArchiveCard
            {item}
            on:filterType={(e) => (selectedType = e.detail)}
            on:filterTerm={(e) =>
              (selectedTerms = toggle(selectedTerms, e.detail))}
            on:authorClick={(e) => handleAuthorClick(e.detail)}
            on:collectionClick={(e) => handleCollectionClick(e.detail)}
          />
        {/each}
      {:else}
        <tr>
          <td colspan="7">
            <div class="no-results">No results found.</div>
          </td>
        </tr>
      {/if}
    </tbody>
  </table>
</main>

{#if showPopup && selectedAuthor}
  <AuthorPopup
    author={selectedAuthor.name}
    bio={selectedAuthor.bio}
    items={selectedAuthor.items}
    onClose={closePopup}
  />
{/if}

{#if showCollectionPopup && selectedCollection}
  <CollectionPopup
    collection={selectedCollection.name}
    coll={selectedCollection.coll}
    items={selectedCollection.items}
    onClose={closePopup}
  />
{/if}

<div class="archive-section">
  <p>
    Some of the projects featured were built in solidarity and jointly with
    Indigenous, traditional and local communities between Latin America and
    Africa, with the purpose of raising awareness of their struggle for the
    cultural and biological survival. Any information and data rights on
    culture, cosmologic view, customary law, arts and crafts, tangible and
    intangible heritage, biodiversity, folklore and commons, remain with these
    Peoples.<br /><br />ALESSANDRO is committed to the development of new modes
    of collaboration, engagement, and partnership with Indigenous peoples for
    the care and stewardship of past and future heritage collections.
    <a
      target="_blank"
      href="https://localcontexts.org/notice/open-to-collaborate/"
      >Read more here</a
    >.
  </p>
  <p>
    Archive design is inspired by <a
      target="_blank"
      href="https://rectangle.design/">Rectangle</a
    >
    studio. Code is published on
    <a target="_blank" href="https://github.com/lsnmst/archive">GitHub</a>. The
    answer to the question <i>Why an archive instead of a promo?</i> is inspired
    by
    <a target="_blank" href="https://silviolorusso.com/">Silvio Lorusso</a>'s
    words. I thank all the people I have met along the path and those I will
    meet. Archive update 11th August 2025.
  </p>
</div>

<style>
  .header-section {
    display: flex;
    flex-direction: row; /* Ensure horizontal layout */
    justify-content: space-between;
    align-items: flex-start;
    padding: 1rem 0.75rem;
    flex-wrap: nowrap; /* Prevent wrapping on wider screens */
    margin-bottom: 1rem;
    border-bottom: 1px #ccc solid;
  }

  .header-title,
  .header-description {
    flex: 1;
    max-width: 30%;
    padding: 0 10px 0 10px;
  }

  .header-title h1 {
    margin: 0;
    font-size: 2.2rem;
    font-style: italic;
    letter-spacing: 1px;
    -webkit-text-stroke-width: 1px;
    -webkit-text-stroke-color: #444;
    -webkit-text-fill-color: #f0f0f000;
  }

  .header-description p {
    margin: 0;
    font-size: 1.1rem;
    color: #444;
    line-height: 1.2rem;
  }

  .archive-section {
    display: flex;
    flex-direction: row; /* Ensure horizontal layout */
    justify-content: space-around;
    align-items: flex-start;
    padding: 1rem 5rem;
    flex-wrap: nowrap; /* Prevent wrapping on wider screens */
    margin-top: 1rem;
    margin-bottom: 2rem;
  }

  .archive-section p {
    font-size: 1rem;
    line-height: 1rem;
    color: #444;
    max-width: 40vh;
  }

  .archive-section a {
    font-weight: 300;
    text-decoration: underline;
  }

  .archive {
    padding: 0 1em;
    max-width: 100%;
    overflow-x: auto;
    box-sizing: border-box;
    font-family: "Libertinus Sans", sans-serif;
    font-style: normal;
    background-color: #f9faf6;
    text-align: center;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.95rem;
  }

  th {
    user-select: none;
    padding: 0.75rem 1rem;
    border-bottom: 2px solid #000;
    font-weight: normal;
    font-size: 0.85rem;
    color: #222;
    white-space: nowrap;
  }

  th.sortable {
    cursor: pointer;
  }

  th.sortable:hover {
    background-color: #f2e4ff;
  }

  th.non-sortable {
    pointer-events: none;
    cursor: default;
    background-color: transparent;
  }

  th span {
    font-size: 0.7em;
    margin-left: 0.25em;
    -webkit-text-fill-color: #ffffff;
    -webkit-text-stroke-width: 1px;
    -webkit-text-stroke-color: rgb(170, 169, 169);
  }

  th:hover {
    background-color: #f0f0f0;
  }

  .filters {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-bottom: 1rem;
    color: #222;
    align-items: center; /* vertical centering */
    justify-content: center; /* horizontal centering */
    text-align: center;
  }

  .filters input,
  .filters select {
    padding: 0.5rem;
    font-size: 0.9rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .multi-select {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    align-items: center;
  }

  .multi-select .filter-title {
    font-weight: bold;
    margin-right: 0.5rem;
    color: #333;
  }

  .multi-select span {
    padding: 0.4em 0.75em;
    border-radius: 10px;
    background-color: #f0f0f0;
    cursor: pointer;
    transition: all 0.2s ease;
    font-size: 0.85rem;
    user-select: none;
  }

  .multi-select span.selected {
    background-color: #264653;
    color: white;
    border-color: #264653;
  }

  .multi-select span:hover {
    background-color: #f2e4ff;
    color: #a255e9;
  }

  .no-results {
    text-align: center;
    padding: 2rem;
    color: #666;
    font-style: italic;
  }

  @media (max-width: 768px) {
    .archive {
      width: 98vh;
      padding: 0 !important;
    }
    table {
      min-width: 100vw !important;
    }
    tbody {
      max-width: 100vw !important;
    }
    thead {
      display: none;
    }
    .multi-select {
      display: none;
    }
    .header-section {
      margin-top: 0;
      flex-direction: column;
    }
    .header-title,
    .header-description {
      max-width: 100%;
      display: inherit;
    }
    .archive-section {
      margin-top: 0;
      flex-direction: column;
      padding: 1rem 2rem;
    }
  }
</style>
