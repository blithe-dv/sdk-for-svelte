<script>
  import { createEventDispatcher } from "svelte";
  import Appwrite from "../appwrite";

  const dispatch = createEventDispatcher();
  export let id;
  export let collection;
  export let document;

  const fetchDocument = () => Appwrite.sdk.database.getDocument(collection, id);

  if (id && collection && !document) {
    document = fetchDocument();
  } else {
    collection = document.$collection;
    id = document.$id;
  }

  export const actions = {
    reload: () => (document = fetchDocument()),
    update: async data => {
      await Appwrite.sdk.database.updateDocument(
        document.$collection,
        document.$id,
        data,
        document.$permissions.read,
        document.$permissions.write
      );
      dispatch("change");
    },
    remove: async () => {
      await Appwrite.sdk.database.deleteDocument(
        document.$collection,
        document.$id
      );
      dispatch("change");
    },
  };
</script>

{#if id && collection}
  {#await document}
    <slot name="loading" />
  {:then current}
    <slot document={current} {actions} />
  {:catch error}
    <slot name="error" {error} />
  {/await}
{:else}
  <slot {actions} />
{/if}
