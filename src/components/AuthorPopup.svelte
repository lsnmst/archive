<script>
    export let author = "";
    export let bio = "";
    export let items = [];
    export let onClose;
</script>

<div class="popup-backdrop" on:click={onClose}>
    <div class="popup-content" on:click|stopPropagation>
        <button class="close-btn" on:click={onClose}>×</button>
        <h2>{author}</h2>
        <p class="bio">{bio}</p>
        <table class="author-table">
            <thead>
                <tr>
                    <th>Type</th>
                    <th class="termspopup">Terms</th>
                    <th>Title</th>
                    <th>Year</th>
                </tr>
            </thead>
            <tbody>
                {#each items as item}
                    <tr>
                        <td>{item.type}</td>
                        <td class="termspopup">{item.terms.join(", ")}</td>
                        <td>
                            <a
                                href={item.url}
                                target="_blank"
                                rel="noopener noreferrer"
                            >
                                {item.title}
                            </a>
                        </td>
                        <td>{item.year}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>

<style>
    .popup-backdrop {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: transparent;
        display: flex;
        justify-content: flex-start; /* Align to left */
        align-items: flex-start;
        z-index: 1000;
        padding: 5vh 1em 1em 5vh; /* 1em margin all around */
    }

    .popup-content {
        pointer-events: auto;
        color: #444;
        background-color: rgba(242, 228, 255, 1);
        padding: 1.5rem;
        border-radius: 6px;
        max-width: 50%;
        width: 100%;
        position: relative;
        overflow-y: auto;
        max-height: 80vh;
        border: 1px solid #ccc;
        pointer-events: auto;
    }

    .close-btn {
        position: absolute;
        top: 0.5rem;
        right: 0.75rem;
        font-size: 1.5rem;
        background: none;
        border: none;
        cursor: pointer;
        color: #444;
    }

    .bio {
        font-style: italic;
        margin-bottom: 1rem;
        color: #555;
    }

    .author-table {
        width: 100%;
        border-collapse: collapse;
        font-size: 0.95rem;
        margin-top: 1rem;
    }

    .author-table th,
    .author-table td {
        padding: 0.75rem 1rem;
        border-bottom: 1px solid #e0e0e0;
        text-align: left;
        vertical-align: top;
        color: #444;
    }

    .author-table th {
        border-bottom: 2px solid #000;
        font-size: 0.85rem;
        font-weight: normal;
        white-space: nowrap;
    }

    .author-table a {
        color: #264653;
        text-decoration: none;
    }

    .author-table a:hover {
        text-decoration: underline;
    }

    @media (max-width: 768px) {
        .popup-backdrop {
            padding: 0;
        }
        .popup-content {
            max-width: 90vh;
        }
        .termspopup {
            display: none;
        }
    }
</style>
