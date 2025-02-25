<script lang="ts">
  import { params, replace } from 'svelte-spa-router'

  import { apiKeyStorage, chatsStorage, clearChats, deleteChat, getLog } from './Storage.svelte'
  import { exportAsMarkdown } from './Export.svelte'

  $: sortedChats = $chatsStorage.sort((a, b) => b.id - a.id)

  $: activeChatId = $params && $params.chatId ? parseInt($params.chatId) : undefined

  function delChat (chatId) {
    if (activeChatId === chatId) {
    // Find the max chatId other than the current one
      const newChatId = sortedChats.reduce((maxId, chat) => {
        if (chat.id === chatId) return maxId
        return Math.max(maxId, chat.id)
      }, 0)

      if (!newChatId) {
        // No other chats, clear all and go to home
        replace('/').then(() => { deleteChat(chatId) })
      } else {
        // Delete the current chat and go to the max chatId
        replace(`/chat/${newChatId}`).then(() => { deleteChat(chatId) })
      }
    } else {
      deleteChat(chatId)
    }
  }
</script>

<aside class="menu">
  <p class="menu-label">Chats</p>
  <ul class="menu-list">
    {#if sortedChats.length === 0}
      <li><a href={'#'} class="is-disabled">No chats yet...</a></li>
    {:else}
      <li>
        <ul>
          {#each sortedChats as chat}
            <li>
              <a style="position: relative" href={`#/chat/${chat.id}`} class:is-disabled={!$apiKeyStorage} class:is-active={activeChatId === chat.id}>
                <a class="is-pulled-right is-hidden px-1 py-0 greyscale has-text-weight-bold delete-button" href={'$'} on:click|preventDefault={() => delChat(chat.id)}>🗑️</a>
                {chat.name || `Chat ${chat.id}`}
              </a>
            </li>
          {/each}
        </ul>
      </li>
    {/if}
  </ul>
  <p class="menu-label">Actions</p>
  <ul class="menu-list">
    <li>
      <a href={'#/'} class="panel-block" class:is-disabled={!$apiKeyStorage} class:is-active={!activeChatId}
        ><span class="greyscale mr-2">🔑</span> API key</a
      >
    </li>
    <li>
      <a href={'#/chat/new'} class="panel-block" class:is-disabled={!$apiKeyStorage}
        ><span class="greyscale mr-2">➕</span> New chat</a
      >
    </li>
    <li>
      <a class="panel-block"
        href="{'#/'}"
        class:is-disabled={!$apiKeyStorage}
        on:click|preventDefault={() => {
          const confirmDelete = window.confirm('Are you sure you want to delete all your chats?')
          if (confirmDelete) {
            replace('#/').then(() => clearChats())
          }
        }}><span class="greyscale mr-2">🗑️</span> Clear chats</a
      >
    </li>
    {#if activeChatId}
    <li>
      <a
        href={'#/'}
        class="panel-block"
        class:is-disabled={!$apiKeyStorage}
        on:click|preventDefault={() => {
          if (activeChatId) {
            exportAsMarkdown(activeChatId)
          }
        }}><span class="greyscale mr-2">📥</span> Export chat</a
      >
    </li>
    {/if}
    <li>
      <a
        href={'#/'}
        class="panel-block"
        on:click|preventDefault={() => {
            const logs = getLog()
            const blob = new Blob([logs], { type: 'text/plain' })
            const url = URL.createObjectURL(blob)
            const link = document.createElement('a')
            link.href = url
            link.download = 'logs.txt'
            document.body.appendChild(link)
            link.click()
            document.body.removeChild(link)
            URL.revokeObjectURL(url)
        }}><span class="greyscale mr-2">💬</span> Download log</a
      >
    </li>
  </ul>
</aside>
