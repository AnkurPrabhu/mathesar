<script lang="ts">
  import { router } from 'tinro';
  import { _ } from 'svelte-i18n';
  import { preloadCommonData } from '@mathesar/utils/preloadData';
  import {
    DropdownMenu,
    Icon,
    iconLoading,
    LinkMenuItem,
    MenuDivider,
    MenuHeading,
    ButtonMenuItem,
  } from '@mathesar-component-library';
  import {
    iconAddNew,
    iconConnection,
    iconDatabase,
    iconExploration,
    iconLogout,
    iconSettingsMajor,
    iconShortcuts,
    iconUser,
  } from '@mathesar/icons';
  import {
    ADMIN_URL,
    CONNECTIONS_URL,
    getDatabasePageUrl,
    getDataExplorerPageUrl,
    getImportPageUrl,
    getTablePageUrl,
    LOGOUT_URL,
    USER_PROFILE_URL,
  } from '@mathesar/routes/urls';
  import { currentDatabase } from '@mathesar/stores/databases';
  import { getReleaseDataStoreFromContext } from '@mathesar/stores/releases';
  import { currentSchema } from '@mathesar/stores/schemas';
  import { createTable } from '@mathesar/stores/tables';
  import { getUserProfileStoreFromContext } from '@mathesar/stores/userProfile';
  import Breadcrumb from './breadcrumb/Breadcrumb.svelte';

  const commonData = preloadCommonData();
  const userProfile = getUserProfileStoreFromContext();
  const releaseDataStore = getReleaseDataStoreFromContext();

  $: database = $currentDatabase;
  $: schema = $currentSchema;
  $: canExecuteDDL = $userProfile?.hasPermission(
    { database, schema },
    'canExecuteDDL',
  );
  $: upgradable = $releaseDataStore?.value?.upgradeStatus === 'upgradable';
  $: isNormalRoutingContext = commonData?.routing_context === 'normal';

  let isCreatingNewEmptyTable = false;

  async function handleCreateEmptyTable() {
    if (!schema || !database) {
      return;
    }
    isCreatingNewEmptyTable = true;
    const tableInfo = await createTable(database, schema, {});
    isCreatingNewEmptyTable = false;
    router.goto(getTablePageUrl(database.id, schema.id, tableInfo.id), false);
  }
</script>

<header class="app-header">
  <div class="left">
    <Breadcrumb />
  </div>

  {#if isNormalRoutingContext}
    <div class="right">
      {#if schema && database}
        <DropdownMenu
          triggerAppearance="ghost"
          size="small"
          closeOnInnerClick={true}
          icon={isCreatingNewEmptyTable ? iconLoading : undefined}
        >
          <span slot="trigger" class="shortcuts">
            <span class="icon"><Icon {...iconShortcuts} /></span>
            <span class="text">{$_('shortcuts')}</span>
          </span>
          {#if canExecuteDDL}
            <ButtonMenuItem icon={iconAddNew} on:click={handleCreateEmptyTable}>
              {$_('new_table_from_scratch')}
            </ButtonMenuItem>
            <LinkMenuItem
              icon={iconAddNew}
              href={getImportPageUrl(database.id, schema.id)}
            >
              {$_('new_table_from_data_import')}
            </LinkMenuItem>
          {/if}
          <LinkMenuItem
            icon={iconExploration}
            href={getDataExplorerPageUrl(database.id, schema.id)}
          >
            {$_('open_data_explorer')}
          </LinkMenuItem>
        </DropdownMenu>
      {/if}
      {#if $userProfile}
        <DropdownMenu
          triggerAppearance="ghost"
          size="small"
          closeOnInnerClick={true}
          menuStyle="--spacing-x: 0.3em;"
        >
          <div class="user-switcher" slot="trigger">
            <Icon {...iconSettingsMajor} hasNotificationDot={upgradable} />
          </div>
          {#if database}
            <MenuHeading>{$_('database')}</MenuHeading>
            <LinkMenuItem
              icon={iconDatabase}
              href={getDatabasePageUrl(database.id)}
            >
              {database.nickname}
            </LinkMenuItem>
            <MenuDivider />
          {/if}
          <MenuHeading>{$_('signed_in_as')}</MenuHeading>
          <LinkMenuItem icon={iconUser} href={USER_PROFILE_URL}>
            {$userProfile.getDisplayName()}
          </LinkMenuItem>
          <MenuDivider />
          <LinkMenuItem icon={iconConnection} href={CONNECTIONS_URL}>
            {$_('database_connections')}
          </LinkMenuItem>
          {#if $userProfile.isSuperUser}
            <LinkMenuItem
              icon={iconSettingsMajor}
              href={ADMIN_URL}
              hasNotificationDot={upgradable}
            >
              {$_('administration')}
            </LinkMenuItem>
          {/if}
          <LinkMenuItem icon={iconLogout} href={LOGOUT_URL} tinro-ignore>
            {$_('log_out')}
          </LinkMenuItem>
        </DropdownMenu>
      {/if}
    </div>
  {/if}
</header>

<style lang="scss">
  .app-header {
    display: flex;
    justify-content: space-between;
    padding: 0.25rem 1rem;
    height: var(--header-height, 60px);
    background-color: var(--slate-800);
    overflow: hidden;
    color: var(--white);
  }

  .left {
    display: flex;
    align-items: center;
    overflow: hidden;
  }

  .right {
    display: flex;
    align-items: center;
    font-size: var(--text-size-large);
  }

  .shortcuts .text {
    display: none;
  }
  .shortcuts .icon,
  .user-switcher {
    background-color: var(--slate-200);
    color: var(--slate-800);
    border-radius: var(--border-radius-m);
    padding: 0.5rem;
    display: flex;
    align-items: center;
  }

  @media (min-width: 45rem) {
    .shortcuts .text {
      display: unset;
    }
    .shortcuts .icon {
      display: none;
    }
  }
</style>
