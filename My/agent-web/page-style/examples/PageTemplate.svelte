<script lang="ts">
    import Icon from "$lib/shared/ui/Icon.svelte";
    import Spinner from "$lib/shared/ui/Spinner.svelte";
    import { fade } from "svelte/transition";

    // État de démonstration (Svelte 5 Runes)
    let searchQuery = $state("");
    let isFocused = $state(false);
    let activeFilter = $state<"all" | "active" | "archived">("all");
    let isLoading = $state(false);

    let totalCount = 42;
    let activeCount = 38;
</script>

<div class="page-container">
    <!-- Header Section -->
    <div class="header-section">
        <div class="title-bar">
            <div>
                <h1>Titre de la Page</h1>
                <p class="subtitle">Description élégante de la fonctionnalité ou de la sous-section</p>
            </div>
            <button class="btn-primary-premium">
                <div class="icon-wrapper">
                    <Icon name="plus" size={18} />
                </div>
                <span class="label">Nouvel Élément</span>
            </button>
        </div>

        <!-- Bento KPI Stats Grid -->
        <div class="bento-stats-grid">
            <div 
                class="bento-stat-card" 
                class:active={activeFilter === "all"}
                onclick={() => activeFilter = "all"}
                role="button"
                tabindex="0"
            >
                <div class="stat-icon-bg pink">
                    <Icon name="users" size={20} />
                </div>
                <div class="stat-info">
                    <span class="stat-value">{totalCount}</span>
                    <span class="stat-label">Total Éléments</span>
                </div>
            </div>

            <div 
                class="bento-stat-card" 
                class:active={activeFilter === "active"}
                onclick={() => activeFilter = "active"}
                role="button"
                tabindex="0"
            >
                <div class="stat-icon-bg amber">
                    <Icon name="check-circle" size={20} />
                </div>
                <div class="stat-info">
                    <span class="stat-value">{activeCount}</span>
                    <span class="stat-label">Éléments Actifs</span>
                </div>
            </div>
        </div>

        <!-- Search Bar Glassmorphism -->
        <div class="search-bar" class:is-focused={isFocused}>
            <div class="search-icon">
                <Icon name="search" size={20} />
            </div>
            <input
                type="text"
                bind:value={searchQuery}
                onfocus={() => isFocused = true}
                onblur={() => isFocused = false}
                placeholder="Rechercher..."
                class="search-input"
            />
            {#if searchQuery}
                <button class="clear-btn" onclick={() => searchQuery = ""}>
                    <Icon name="x" size={16} />
                </button>
            {/if}
        </div>
    </div>

    <!-- Content / Table Grid -->
    <div class="content-grid">
        {#if isLoading}
            <div class="loading-overlay" in:fade>
                <Spinner size={36} color="var(--color-primary)" />
                <p>Chargement des données...</p>
            </div>
        {/if}

        <div class="list-header">
            <div>Nom</div>
            <div>Statut</div>
            <div>Actions</div>
        </div>

        <div class="list-body">
            <div class="table-row">
                <div class="col-name">
                    <span class="name">Exemple d'Élément</span>
                </div>
                <div class="col-status">Actif</div>
                <div class="col-actions">
                    <button class="btn-sync-premium">
                        <div class="icon-wrapper">
                            <Icon name="settings" size={16} />
                        </div>
                        <span class="label">Gérer</span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>

<style lang="scss">
    @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,500;0,700;1,500;1,700&display=swap');

    .page-container {
        padding: 30px 40px;
        max-width: 1400px;
        margin: 0 auto;
        height: 100%;
        width: 100%;
        overflow-y: auto;
        overflow-x: hidden;
        display: flex;
        flex-direction: column;
        gap: 20px;
        box-sizing: border-box;
        background: linear-gradient(135deg, #eee7dd 0%, #f4eee6 50%, #eae3d8 100%);
        border: 1px solid #dfd5c6;
        border-radius: var(--radius-lg, 24px);

        @media (max-width: 1024px) {
            padding: 15px 20px;
            gap: 12px;
        }
    }

    .header-section {
        display: flex;
        flex-direction: column;
        gap: 16px;

        .title-bar {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 16px;
            flex-wrap: wrap;

            h1 {
                font-family: var(--font-heading, "Playfair Display", serif);
                font-size: 2.3rem;
                font-weight: 700;
                color: #3d121c;
                margin: 0 0 4px 0;
                line-height: 1.15;
                font-style: italic;
            }

            .subtitle {
                margin: 0;
                font-size: var(--font-size-md, 0.95rem);
                color: #574c4c;
                font-weight: 500;
            }
        }
    }

    .bento-stats-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 16px;

        .bento-stat-card {
            background: #ffffff;
            border: 1.5px solid #e5d9cd;
            border-radius: 16px;
            padding: 16px 20px;
            display: flex;
            align-items: center;
            gap: 16px;
            cursor: pointer;
            transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
            box-shadow: 0 4px 16px rgba(92, 31, 43, 0.08);

            &:hover {
                transform: translateY(-2px);
                border-color: #803d4a;
                box-shadow: 0 10px 24px rgba(128, 61, 74, 0.18);
            }

            &.active {
                background: linear-gradient(135deg, #fff0f2 0%, #ffffff 100%);
                border: 2px solid #803d4a;
                box-shadow: 0 10px 26px rgba(128, 61, 74, 0.22);
            }

            .stat-icon-bg {
                width: 46px;
                height: 46px;
                border-radius: 14px;
                display: flex;
                align-items: center;
                justify-content: center;
                flex-shrink: 0;

                &.pink { background: #ffe3e6; color: #803d4a; }
                &.amber { background: #feebd9; color: #a05215; }
            }

            .stat-info {
                display: flex;
                flex-direction: column;

                .stat-value {
                    font-family: var(--font-number, sans-serif);
                    font-size: 1.5rem;
                    font-weight: 800;
                    color: #2b0b13;
                }

                .stat-label {
                    font-size: 0.8rem;
                    color: #4a3e3e;
                    font-weight: 700;
                    text-transform: uppercase;
                    letter-spacing: 0.04em;
                }
            }
        }
    }

    .search-bar {
        display: flex;
        align-items: center;
        background: #ffffff;
        border: 1.5px solid rgba(128, 61, 74, 0.28);
        border-radius: var(--radius-lg, 20px);
        padding: 0 12px 0 20px;
        height: 54px;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        box-shadow: 0 8px 24px -4px rgba(92, 31, 43, 0.12);

        &.is-focused {
            background: #ffffff;
            border: 2px solid #803d4a;
            box-shadow: 0 0 0 4px rgba(128, 61, 74, 0.18);
            transform: translateY(-1px);
        }

        .search-input {
            border: none;
            background: transparent;
            width: 100%;
            font-size: 1rem;
            font-weight: 600;
            color: #1a1a1a;
            outline: none;
        }

        .clear-btn {
            background: rgba(128, 61, 74, 0.12);
            border: none;
            color: #803d4a;
            cursor: pointer;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
        }
    }

    .content-grid {
        background: #ffffff;
        border-radius: var(--radius-lg, 20px);
        border: 1.5px solid #d4c4b2;
        box-shadow: 0 14px 34px -6px rgba(92, 31, 43, 0.16);
        overflow: hidden;

        .list-header {
            display: grid;
            grid-template-columns: 2fr 1fr 1fr;
            padding: 12px 20px;
            background: linear-gradient(90deg, #fcebed 0%, #faf0e6 100%);
            border-bottom: 1.5px solid #e8decb;
            font-weight: 700;
            color: #3d121c;
            font-size: 0.78rem;
            text-transform: uppercase;
            letter-spacing: 0.06em;
        }

        .table-row {
            display: grid;
            grid-template-columns: 2fr 1fr 1fr;
            padding: 12px 20px;
            align-items: center;
            border-bottom: 1px solid #eee5db;
            background: #ffffff;
            position: relative;
            transition: all 0.2s ease;

            &:nth-child(even) { background: #fdfbf8; }

            &::before {
                content: '';
                position: absolute;
                left: 0;
                top: 0;
                bottom: 0;
                width: 3px;
                background: #803d4a;
                opacity: 0;
                transition: opacity 0.2s ease;
            }

            &:hover {
                background: #fbf5ee;
                &::before { opacity: 1; }
            }
        }
    }

    .btn-primary-premium {
        background: linear-gradient(135deg, #803d4a 0%, #5c1f2b 100%);
        color: #ffffff;
        border: none;
        border-radius: 14px;
        padding: 0 20px 0 12px;
        height: 44px;
        font-weight: 700;
        font-size: 0.92rem;
        cursor: pointer;
        display: flex;
        align-items: center;
        gap: 10px;
        box-shadow: 0 6px 20px rgba(92, 31, 43, 0.35);

        .icon-wrapper {
            width: 28px;
            height: 28px;
            background: rgba(255, 255, 255, 0.22);
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: transform 0.25s ease;
        }

        &:hover .icon-wrapper {
            transform: scale(1.12) rotate(90deg);
        }
    }

    .btn-sync-premium {
        background: #ffffff;
        color: #2b1f1f;
        border: 1.5px solid #d4c4b2;
        border-radius: 12px;
        padding: 0 16px 0 8px;
        height: 42px;
        font-weight: 700;
        font-size: 0.88rem;
        cursor: pointer;
        display: inline-flex;
        align-items: center;
        gap: 10px;

        .icon-wrapper {
            width: 28px;
            height: 28px;
            background: #ffe0e4;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #803d4a;
        }
    }
</style>
