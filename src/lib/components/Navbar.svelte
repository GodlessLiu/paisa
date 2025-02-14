<script lang="ts">
  import { page } from "$app/stores";
  import Actions from "$lib/components/Actions.svelte";
  import { month, year, dateMax, dateMin, dateRangeOption } from "../../store";
  import {
    cashflowType,
    cashflowExpenseDepth,
    cashflowExpenseDepthAllowed,
    cashflowIncomeDepth,
    cashflowIncomeDepthAllowed
  } from "../../persisted_store";
  import _ from "lodash";
  import { financialYear, forEachFinancialYear, helpUrl } from "$lib/utils";
  import { onMount } from "svelte";
  import { get } from "svelte/store";
  import dayjs from "dayjs";
  import DateRange from "./DateRange.svelte";
  import ThemeSwitcher from "./ThemeSwitcher.svelte";
  import BoxedTabs from "./BoxedTabs.svelte";
  import MonthPicker from "./MonthPicker.svelte";
  import Logo from "./Logo.svelte";
  import InputRange from "./InputRange.svelte";
  export let isBurger: boolean = null;

  onMount(async () => {
    if (get(year) == "") {
      year.set(financialYear(dayjs()));
    }
  });

  interface Link {
    label: string;
    href: string;
    tag?: string;
    help?: string;
    hide?: boolean;
    dateRangeSelector?: boolean;
    monthPicker?: boolean;
    cashflowTypePicker?: boolean;
    financialYearPicker?: boolean;
    maxDepthSelector?: boolean;
    children?: Link[];
  }
  const links: Link[] = [
    { label: "Dashboard", href: "/", hide: true },
    {
      label: "Cash Flow",
      href: "/cash_flow",
      children: [
        { label: "Monthly", href: "/monthly", dateRangeSelector: true },
        {
          label: "Yearly",
          href: "/yearly",
          cashflowTypePicker: true,
          financialYearPicker: true,
          maxDepthSelector: true
        },
        { label: "Recurring", href: "/recurring", help: "recurring" }
      ]
    },
    {
      label: "Expenses",
      href: "/expense",
      children: [
        { label: "Monthly", href: "/monthly", monthPicker: true, dateRangeSelector: true },
        { label: "Yearly", href: "/yearly", financialYearPicker: true },
        { label: "Budget", href: "/budget", help: "budget", monthPicker: true }
      ]
    },
    {
      label: "Assets",
      href: "/assets",
      children: [
        { label: "Balance", href: "/balance" },
        { label: "Networth", href: "/networth", dateRangeSelector: true },
        { label: "Investment", href: "/investment" },
        { label: "Gain", href: "/gain" },
        { label: "Allocation", href: "/allocation", help: "allocation-targets" },
        { label: "Analysis", href: "/analysis", tag: "alpha", help: "analysis" }
      ]
    },
    {
      label: "Liabilities",
      href: "/liabilities",
      children: [
        { label: "Balance", href: "/balance" },
        { label: "Repayment", href: "/repayment" },
        { label: "Interest", href: "/interest" }
      ]
    },
    { label: "Income", href: "/income" },
    {
      label: "Ledger",
      href: "/ledger",
      children: [
        { label: "Import", href: "/import", help: "import" },
        { label: "Editor", href: "/editor" },
        { label: "Transactions", href: "/transaction", help: "bulk-edit" },
        { label: "Postings", href: "/posting" },
        { label: "Price", href: "/price" }
      ]
    },
    {
      label: "More",
      href: "/more",
      children: [
        { label: "Configuration", href: "/config", tag: "alpha" },
        { label: "Retirement", href: "/retirement/progress", help: "retirement" },
        { label: "Doctor", href: "/doctor" },
        { label: "Logs", href: "/logs" }
      ]
    }
  ];

  const tax = {
    label: "Tax",
    href: "/tax",
    help: "tax",
    children: [
      { label: "Harvest", href: "/harvest", help: "tax-harvesting" },
      { label: "Capital Gains", href: "/capital_gains", help: "capital-gains" },
      {
        label: "Schedule AL",
        href: "/schedule_al",
        help: "schedule-al",
        financialYearPicker: true
      }
    ]
  };

  if (USER_CONFIG.default_currency == "INR") {
    _.last(links).children.push(tax);
  }

  let selectedLink: Link = null;
  let selectedSubLink: Link = null;
  let selectedSubSubLink: Link = null;

  $: if ($page.url.pathname) {
    selectedSubLink = null;
    selectedSubSubLink = null;
    selectedLink = _.find(links, (l) => $page.url.pathname == l.href);
    if (!selectedLink) {
      selectedLink = _.find(
        links,
        (l) => !_.isEmpty(l.children) && $page.url.pathname.startsWith(l.href)
      );

      selectedSubLink = _.find(
        selectedLink.children,
        (l) => $page.url.pathname == selectedLink.href + l.href
      );

      if (!selectedSubLink) {
        selectedSubLink = _.find(selectedLink.children, (l) =>
          $page.url.pathname.startsWith(selectedLink.href + l.href)
        );

        if (!_.isEmpty(selectedSubLink.children)) {
          selectedSubSubLink = _.find(selectedSubLink.children, (l) =>
            $page.url.pathname.startsWith(selectedLink.href + selectedSubLink.href + l.href)
          );
        }
      }
    }
  }
</script>

<nav class="navbar px-3 is-transparent" aria-label="main navigation">
  <div class="navbar-brand">
    <a
      href="/"
      class:is-active={$page.url.pathname == "/"}
      class="navbar-item is-size-4 has-text-weight-medium"><Logo size={20} /> Paisa</a
    >
    <a
      role="button"
      tabindex="-1"
      class="navbar-burger"
      class:is-active={isBurger === true}
      on:click|preventDefault={(_e) => (isBurger = !isBurger)}
      aria-label="menu"
      aria-expanded="false"
      data-target="navbarBasicExample"
    >
      <span aria-hidden="true" />
      <span aria-hidden="true" />
      <span aria-hidden="true" />
    </a>
  </div>

  <div class="navbar-menu" class:is-active={isBurger === true}>
    <div class="navbar-start">
      {#each links as link}
        {#if _.isEmpty(link.children)}
          {#if !link.hide}
            <a
              class="navbar-item"
              href={link.href}
              class:is-active={$page.url.pathname == link.href}>{link.label}</a
            >
          {/if}
        {:else}
          <div class="navbar-item has-dropdown is-hoverable">
            <a class="navbar-link" class:is-active={$page.url.pathname.startsWith(link.href)}
              >{link.label}</a
            >
            <div class="navbar-dropdown is-boxed">
              {#each link.children as sublink}
                {@const href = link.href + sublink.href}
                {#if _.isEmpty(sublink.children)}
                  <a
                    class="navbar-item"
                    {href}
                    class:is-active={$page.url.pathname.startsWith(href)}>{sublink.label}</a
                  >
                {:else}
                  <div class="nested has-dropdown navbar-item">
                    <a
                      class="navbar-link is-arrowless is-flex is-justify-content-space-between is-active"
                      class:is-active={$page.url.pathname.startsWith(href)}
                    >
                      <span>{sublink.label}</span>
                      <span class="icon is-small">
                        <i class="fas fa-angle-right" aria-hidden="true"></i>
                      </span>
                    </a>

                    <div class="dropdown-menu">
                      <div class="dropdown-content">
                        {#each sublink.children as subsublink}
                          <a
                            href={href + subsublink.href}
                            class="navbar-item"
                            class:is-active={$page.url.pathname == href + subsublink.href}
                            >{subsublink.label}</a
                          >
                        {/each}
                      </div>
                    </div>
                  </div>
                {/if}
              {/each}
            </div>
          </div>
        {/if}
      {/each}
    </div>
    <div class="navbar-end">
      <div class="navbar-item">
        <div class="field is-grouped">
          <p class="control">
            <ThemeSwitcher />
          </p>
          <p class="control">
            <Actions />
          </p>
        </div>
      </div>
    </div>
  </div>
</nav>

<div class="mt-2 px-3 is-flex is-justify-content-space-between">
  {#if selectedLink}
    <nav
      style="margin-left: 12px;"
      class="breadcrumb has-chevron-separator mb-0 is-small"
      aria-label="breadcrumbs"
    >
      <ul>
        <li>
          <a class="is-inactive">{selectedLink.label}</a>
          {#if selectedLink.help}
            <a style="margin-left: -10px;" class="p-0" href={helpUrl(selectedLink.help)}
              ><span class="icon is-small">
                <i class="fas fa-question fa-border" />
              </span></a
            >
          {/if}

          {#if selectedLink.tag}
            <span style="font-size: 0.6rem" class="tag is-rounded is-warning"
              >{selectedLink.tag}</span
            >
          {/if}
        </li>
        {#if selectedSubLink}
          <li>
            <a class="is-inactive">{selectedSubLink.label}</a>

            {#if selectedSubLink.help}
              <a style="margin-left: -10px;" class="p-0" href={helpUrl(selectedSubLink.help)}
                ><span class="icon is-small">
                  <i class="fas fa-question fa-border" />
                </span></a
              >
            {/if}

            {#if selectedSubLink.tag}
              <span style="font-size: 0.6rem" class="tag is-rounded is-warning mr-2"
                >{selectedSubLink.tag}</span
              >
            {/if}
          </li>
        {/if}

        {#if selectedSubLink}
          {#if selectedSubSubLink}
            <li>
              <a class="is-inactive">{selectedSubSubLink.label}</a>
            </li>
          {:else if selectedLink.href + selectedSubLink.href != $page.url.pathname}
            <li>
              <a class="is-inactive">{decodeURIComponent(_.last($page.url.pathname.split("/")))}</a>
            </li>
          {/if}
        {/if}
      </ul>
    </nav>
  {/if}

  <div class="mr-3 is-flex" style="gap: 12px">
    {#if selectedSubLink?.cashflowTypePicker}
      <BoxedTabs
        options={[
          { label: "Flat", value: "flat" },
          { label: "Hierarchy", value: "hierarchy" }
        ]}
        bind:value={$cashflowType}
      />
    {/if}

    {#if selectedSubLink?.maxDepthSelector && ($cashflowExpenseDepthAllowed.max > 1 || $cashflowIncomeDepthAllowed.max > 1) && $cashflowType == "hierarchy"}
      <div class="dropdown is-right is-hoverable">
        <div class="dropdown-trigger">
          <button class="button is-small" aria-haspopup="true">
            <span class="icon is-small">
              <i class="fas fa-sliders" />
            </span>
          </button>
        </div>
        <div class="dropdown-menu" id="dropdown-menu4" role="menu">
          <div class="dropdown-content px-2 py-2">
            <InputRange
              label="Expenses"
              bind:value={$cashflowExpenseDepth}
              allowed={$cashflowExpenseDepthAllowed}
            />
            <InputRange
              label="Income"
              bind:value={$cashflowIncomeDepth}
              allowed={$cashflowIncomeDepthAllowed}
            />
          </div>
        </div>
      </div>
    {/if}

    {#if selectedSubLink?.dateRangeSelector || selectedLink?.dateRangeSelector}
      <div>
        <DateRange bind:value={$dateRangeOption} dateMin={$dateMin} dateMax={$dateMax} />
      </div>
    {/if}

    {#if selectedSubLink?.monthPicker || selectedLink?.monthPicker}
      <MonthPicker bind:value={$month} max={$dateMax} min={$dateMin} />
    {/if}

    {#if selectedSubLink?.financialYearPicker || selectedLink?.financialYearPicker}
      <div class="has-text-centered">
        <div class="select is-small">
          <select bind:value={$year}>
            {#each forEachFinancialYear($dateMin, $dateMax).reverse() as fy}
              <option>{financialYear(fy)}</option>
            {/each}
          </select>
        </div>
      </div>
    {/if}
  </div>
</div>

<style lang="scss">
  li a span.icon {
    margin-top: -5px;
  }
</style>
