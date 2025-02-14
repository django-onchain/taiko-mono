<script lang="ts">
  import { wrap } from "svelte-spa-router/wrap";
  import QueryProvider from "./components/providers/QueryProvider.svelte";
  import Router from "svelte-spa-router";
  import { SvelteToast, toast } from "@zerodevx/svelte-toast";
  import type { SvelteToastOptions } from "@zerodevx/svelte-toast";

  import Home from "./pages/home/Home.svelte";
  import { setupI18n } from "./i18n";
  import { BridgeType } from "./domain/bridge";
  import ETHBridge from "./eth/bridge";
  import { bridges, chainIdToBridgeAddress } from "./store/bridge";
  import ERC20Bridge from "./erc20/bridge";
  import { pendingTransactions } from "./store/transactions";
  import Navbar from "./components/Navbar.svelte";
  import { signer } from "./store/signer";
  import type { Transactioner } from "./domain/transactions";
  import { RelayerService } from "./relayer/service";

  setupI18n({ withLocale: "en" });
  import { CHAIN_MAINNET, CHAIN_TKO } from "./domain/chain";
  import SwitchEthereumChainModal from "./components/modals/SwitchEthereumChainModal.svelte";
  import { ProofService } from "./proof/service";
  import { ethers } from "ethers";
  import type { Prover } from "./domain/proof";
  import { successToast } from "./utils/toast";

  const providerMap: Map<number, ethers.providers.JsonRpcProvider> = new Map<
    number,
    ethers.providers.JsonRpcProvider
  >();
  providerMap.set(
    CHAIN_MAINNET.id,
    new ethers.providers.JsonRpcProvider(import.meta.env.VITE_L1_RPC_URL)
  );
  providerMap.set(
    CHAIN_TKO.id,
    new ethers.providers.JsonRpcProvider(import.meta.env.VITE_L2_RPC_URL)
  );

  const prover: Prover = new ProofService(providerMap);

  const ethBridge = new ETHBridge(prover);
  const erc20Bridge = new ERC20Bridge(prover);

  bridges.update((store) => {
    store.set(BridgeType.ETH, ethBridge);
    store.set(BridgeType.ERC20, erc20Bridge);
    return store;
  });

  chainIdToBridgeAddress.update((store) => {
    store.set(CHAIN_TKO.id, import.meta.env.VITE_TAIKO_BRIDGE_ADDRESS);
    store.set(CHAIN_MAINNET.id, import.meta.env.VITE_MAINNET_BRIDGE_ADDRESS);
    return store;
  });

  const relayerURL = import.meta.env.VITE_RELAYER_URL;

  const transactioner: Transactioner = new RelayerService(relayerURL);

  pendingTransactions.subscribe((store) => {
    store.forEach(async (tx) => {
      await $signer.provider.waitForTransaction(tx.hash, 3);
      successToast("Transaction completed!");
      const s = store;
      s.pop();
      pendingTransactions.set(s);
    });
  });

  const toastOptions: SvelteToastOptions = {
    dismissable: false,
    duration: 4000,
    pausable: false,
  };

  const routes = {
    "/": wrap({
      component: Home,
      props: {},
      userData: {},
    }),
  };
</script>

<QueryProvider>
  <main>
    <Navbar {transactioner} />
    <Router {routes} />
  </main>
  <SvelteToast options={toastOptions} />

  <SwitchEthereumChainModal />
</QueryProvider>

<style global lang="postcss">
  @tailwind base;
  @tailwind components;
  @tailwind utilities;

  main {
    margin: 0;
    font-family: "Inter", sans-serif;
  }
</style>
