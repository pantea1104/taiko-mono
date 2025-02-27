<script lang="ts">
  import { onDestroy, onMount } from 'svelte';
  import { t } from 'svelte-i18n';
  import type { Address } from 'viem';

  import { Icon } from '$components/Icon';
  import Erc20 from '$components/Icon/ERC20.svelte';
  import { OnAccount } from '$components/OnAccount';
  import { tokenService } from '$libs/storage/services';
  import type { Token } from '$libs/token';
  import { classNames } from '$libs/util/classNames';
  import { noop } from '$libs/util/noop';
  import { account } from '$stores/account';

  import AddCustomErc20 from './AddCustomERC20.svelte';
  import { symbolToIconMap } from './symbolToIconMap';

  export let id: string;
  export let menuOpen = false;
  export let tokens: Token[] = [];
  export let customTokens: Token[] = [];
  export let value: Maybe<Token> = null;
  export let selectToken: (token: Token) => void = noop;

  let addArc20ModalOpen = false;

  $: menuClasses = classNames(
    'menu absolute right-0 w-[265px] p-3 mt-2 rounded-[10px] bg-neutral-background z-10  box-shadow-small',
    menuOpen ? 'visible opacity-100' : 'invisible opacity-0',
  );

  const getTokenKeydownHandler = (token: Token) => {
    return (event: KeyboardEvent) => {
      if (event.key === 'Enter') {
        selectToken(token);
      }
    };
  };

  const showAddERC20 = () => (addArc20ModalOpen = true);

  const handleStorageChange = (newTokens: Token[]) => {
    customTokens = newTokens;
  };

  const onAccountChange = () => {
    if ($account?.address) {
      customTokens = tokenService.getTokens($account?.address as Address);
    }
  };

  onMount(() => {
    tokenService.subscribeToChanges(handleStorageChange);
  });

  onDestroy(() => {
    tokenService.unsubscribeFromChanges(handleStorageChange);
  });
</script>

<!-- Desktop (or larger) view -->
<ul role="listbox" {id} class={menuClasses}>
  {#each tokens as t (t.symbol)}
    <li
      role="option"
      tabindex="0"
      aria-selected={t === value}
      on:click={() => selectToken(t)}
      on:keydown={getTokenKeydownHandler(t)}>
      <div class="p-4">
        {#if symbolToIconMap[t.symbol]}
          <i role="img" aria-label={t.name}>
            <svelte:component this={symbolToIconMap[t.symbol]} size={28} />
          </i>
        {:else}
          <i role="img" aria-label={t.symbol}>
            <svelte:component this={Erc20} size={28} />
          </i>
        {/if}
        <span class="body-bold">{t.symbol}</span>
      </div>
    </li>
  {/each}
  {#each customTokens as ct, index (index)}
    <li
      role="option"
      tabindex="0"
      aria-selected={ct === value}
      on:click={() => selectToken(ct)}
      on:keydown={getTokenKeydownHandler(ct)}>
      <div class="p-4">
        <i role="img" aria-label={ct.name}>
          <Erc20 />
        </i>
        <span class="body-bold">{ct.symbol}</span>
      </div>
    </li>
  {/each}
  <div class="h-sep my-[8px]" />
  <li>
    <button on:click={showAddERC20} class="flex hover:bg-dark-5 justify-center items-center rounded-lg h-[64px]">
      <Icon type="plus-circle" fillClass="fill-primary-icon" size={32} vWidth={28} vHeight={28} />
      <span
        class="
            body-bold
            bg-transparent
            flex-1
            px-0">
        {$t('token_dropdown.add_custom')}
      </span>
    </button>
  </li>
</ul>

<AddCustomErc20 bind:modalOpen={addArc20ModalOpen} on:tokenRemoved />

<OnAccount change={onAccountChange} />
