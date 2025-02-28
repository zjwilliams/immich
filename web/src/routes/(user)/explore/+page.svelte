<script lang="ts">
  import ImageThumbnail from '$lib/components/assets/thumbnail/image-thumbnail.svelte';
  import UserPageLayout from '$lib/components/layouts/user-page-layout.svelte';
  import { AppRoute } from '$lib/constants';
  import { getAssetThumbnailUrl, getPeopleThumbnailUrl } from '$lib/utils';
  import { AssetMediaSize, type SearchExploreResponseDto } from '@immich/sdk';
  import type { PageData } from './$types';
  import { getMetadataSearchQuery } from '$lib/utils/metadata-search';
  import { t } from 'svelte-i18n';
  import EmptyPlaceholder from '$lib/components/shared-components/empty-placeholder.svelte';
  import { onMount } from 'svelte';
  import { websocketEvents } from '$lib/stores/websocket';
  import SingleGridRow from '$lib/components/shared-components/single-grid-row.svelte';

  export let data: PageData;

  enum Field {
    CITY = 'exifInfo.city',
  }

  const getFieldItems = (items: SearchExploreResponseDto[], field: Field) => {
    const targetField = items.find((item) => item.fieldName === field);
    return targetField?.items || [];
  };

  $: places = getFieldItems(data.items, Field.CITY);
  $: people = data.response.people;
  $: hasPeople = data.response.total > 0;

  onMount(() => {
    return websocketEvents.on('on_person_thumbnail', (personId: string) => {
      people.map((person) => {
        if (person.id === personId) {
          person.updatedAt = Date.now().toString();
        }
      });
      // trigger reactivity
      people = people;
    });
  });
</script>

<UserPageLayout title={data.meta.title}>
  {#if hasPeople}
    <div class="mb-6 mt-2">
      <div class="flex justify-between">
        <p class="mb-4 font-medium dark:text-immich-dark-fg">{$t('people')}</p>
        <a
          href={AppRoute.PEOPLE}
          class="pr-4 text-sm font-medium hover:text-immich-primary dark:text-immich-dark-fg dark:hover:text-immich-dark-primary"
          draggable="false">{$t('view_all')}</a
        >
      </div>
      <SingleGridRow class="grid md:grid-auto-fill-28 grid-auto-fill-20 gap-x-4" let:itemCount>
        {#each people.slice(0, itemCount) as person (person.id)}
          <a href="{AppRoute.PEOPLE}/{person.id}" class="text-center">
            <ImageThumbnail circle shadow url={getPeopleThumbnailUrl(person)} altText={person.name} widthStyle="100%" />
            <p class="mt-2 text-ellipsis text-sm font-medium dark:text-white">{person.name}</p>
          </a>
        {/each}
      </SingleGridRow>
    </div>
  {/if}

  {#if places.length > 0}
    <div class="mb-6 mt-2">
      <div class="flex justify-between">
        <p class="mb-4 font-medium dark:text-immich-dark-fg">{$t('places')}</p>
        <a
          href={AppRoute.PLACES}
          class="pr-4 text-sm font-medium hover:text-immich-primary dark:text-immich-dark-fg dark:hover:text-immich-dark-primary"
          draggable="false">{$t('view_all')}</a
        >
      </div>
      <SingleGridRow class="grid md:grid-auto-fill-36 grid-auto-fill-28 gap-x-4" let:itemCount>
        {#each places.slice(0, itemCount) as item (item.data.id)}
          <a class="relative" href="{AppRoute.SEARCH}?{getMetadataSearchQuery({ city: item.value })}" draggable="false">
            <div class="flex justify-center overflow-hidden rounded-xl brightness-75 filter">
              <img
                src={getAssetThumbnailUrl({ id: item.data.id, size: AssetMediaSize.Thumbnail })}
                alt={item.value}
                class="object-cover aspect-square w-full"
              />
            </div>
            <span
              class="w-100 absolute bottom-2 w-full text-ellipsis px-1 text-center text-sm font-medium capitalize text-white backdrop-blur-[1px] hover:cursor-pointer"
            >
              {item.value}
            </span>
          </a>
        {/each}
      </SingleGridRow>
    </div>
  {/if}

  {#if !hasPeople && places.length === 0}
    <EmptyPlaceholder text={$t('no_explore_results_message')} />
  {/if}
</UserPageLayout>
