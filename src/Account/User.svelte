<script>
  import { createEventDispatcher } from "svelte";
  import { active, currentUser } from "../stores";
  import Appwrite from "../appwrite";

  const dispatch = createEventDispatcher();

  const fetchUser = async () => {
    try {
      const response = await Appwrite.sdk.account.get();
      $currentUser = response;
      dispatch("success", $currentUser);
      return response;
    } catch (error) {
      dispatch("failure", error);
      throw error;
    }
  };

  const actions = {
    reload: () => (request = fetchUser()),
    logout: async () => {
      try {
        const response = await Appwrite.sdk.account.deleteSession("current");
        actions.reload();
        dispatch("successLogout", response);
      } catch (error) {
        dispatch("failureLogout", error);
        throw error;
      }
    },
    logoutFrom: async session => {
      try {
        const response = await Appwrite.sdk.account.deleteSession(session);
        actions.reload();
        dispatch("successLogoutFrom", response);
      } catch (error) {
        dispatch("failureLogoutFrom", error);
        throw error;
      }
    },
    logoutAll: async () => {
      try {
        const response = await Appwrite.sdk.account.deleteSessions();
        actions.reload();
        dispatch("successLogoutAll", response);
      } catch (error) {
        dispatch("failureLogoutAll", error);
        throw error;
      }
    },
  };

  let request = fetchUser();
</script>

{#if $active}
  {#await request}
    <slot name="loading" />
  {:then user}
    <slot {user} {actions} />
  {:catch error}
    <slot name="error" {error} />
  {/await}
{/if}
