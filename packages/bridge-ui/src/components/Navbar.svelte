<script lang="ts">
  import Connect from "./buttons/Connect.svelte";
  import Logo from "./icons/Logo.svelte";
  import { signer } from "../store/signer";
  import AddressDropdown from "./AddressDropdown.svelte";
  import type {
    BridgeTransaction,
    Transactioner,
  } from "../domain/transactions";
  import type { Signer } from "ethers";
  import { fromChain } from "../store/chain";
  import ChainDropdown from "./ChainDropdown.svelte";
  import type { Chain } from "../domain/chain";

  export let transactioner: Transactioner;
  let transactions: BridgeTransaction[];
  $: getTransactions($signer, $fromChain);

  async function getTransactions(signer: Signer, chain: Chain) {
    if (!signer || !chain) return;
    transactions = await transactioner.GetAllByAddress(
      await signer.getAddress(),
      chain.id
    );
  }
</script>

<nav class="navbar mb-4 md:h-[125px] pt-4 md:pt-0 md:px-4 w-full">
  <div class="navbar-end justify-start">
    <Logo />
  </div>
  <div class="navbar-end">
    {#if $signer}
      <ChainDropdown />
      <AddressDropdown {transactions} />
    {:else}
      <Connect />
    {/if}
  </div>
</nav>
