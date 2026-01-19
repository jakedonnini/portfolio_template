<script lang="ts">
    import { type CollectionEntry } from 'astro:content';
    import { filters } from '../../stores/filters.store';
    import Filters from './Filters.svelte';
    import { cn } from '../../styles/cn';

    interface GitHubProject {
        slug: string;
        title: string;
        tags: string[];
        icon: string; // URL to the thumbnail
        repo: string; // GitHub repo link
    }

    interface Props {
        allPosts: GitHubProject[];
    }

    const { allPosts }: Props = $props();
    let sortedPosts: GitHubProject[] = $state(allPosts);


    function getIntersectionLength(a: any[], b: any[]): number {
        const intersection = a.filter(value => b.includes(value));
        return intersection.length;
    }

    function sortPosts(filters: string[]) {
        if (!filters || filters.length === 0) {
            // Sort by last GitHub push date
            sortedPosts = [...allPosts].sort((a, b) => b.date.valueOf() - a.date.valueOf());
            return;
        }

        // Sort by tag intersection count
        sortedPosts = [...allPosts].sort(
            (a, b) => getIntersectionLength(b.tags, filters) - getIntersectionLength(a.tags, filters)
        );
    }

    filters.subscribe((value) => {
        sortPosts([...value]);
    })

    function compareTags(a: string, b: string): number {
        const includesA: boolean = $filters.includes(a);
        const includesB: boolean = $filters.includes(b);
        return includesA && includesB ? 0 : includesA ? -1 : 1; 
    }

    function getSortedTags(post: GitHubProject): string[] {
        const sortedTags: string[] = [...post.tags]; // use post.tags instead of post.data.tags
        sortedTags.sort((a, b) => compareTags(a, b));
        return sortedTags;
    }

</script>

{#snippet postCard(post: GitHubProject)}
    <a
        href={`https://github.com/${post.repo}`}
        target="_blank"
        rel="noopener noreferrer"
        class="flex flex-col justify-between gap-2 min-h-32 h-full p-2 border rounded-interactive border-edge bg-linear-to-b from-secondary to-secondary/60 pointer-events-auto hover:border-accent duration-200"
    >
        <div class="flex flex-col gap-2">
            <img src={post.icon} alt={post.title} class="aspect-square w-full rounded-interactive object-cover"/>
            <h2 class="text-lg font-semibold">{post.title}</h2>
        </div>
        <div class="flex flex-row gap-2 overflow-hidden text-accent">
            {#each getSortedTags(post) as tag}
                <span class={($filters.length > 0 && !$filters.includes(tag)) ? "opacity-60" : "font-black"}>{tag}</span>
            {/each}
        </div>
    </a>
{/snippet}

<Filters />

<div class="group grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 p-4 rounded-interactive pointer-events-none hover:border-edge duration-200">					
    {#each sortedPosts as post}
        {@render postCard(post)}
    {/each}
</div>