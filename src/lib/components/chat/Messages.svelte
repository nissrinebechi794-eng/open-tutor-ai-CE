<script lang="ts">
	import { v4 as uuidv4 } from 'uuid';
	import {
		chats,
		config,
		settings,
		user as _user,
		currentChatPage,
		temporaryChatEnabled
	} from '$lib/stores';

	import { tick, getContext, createEventDispatcher } from 'svelte';
	import { toast } from 'svelte-sonner';

	import { getChatList, updateChatById } from '$lib/apis/chats';
	import { copyToClipboard } from '$lib/utils';

	import Message from './Messages/Message.svelte';
	import Loader from '../common/Loader.svelte';
	import Spinner from '../common/Spinner.svelte';
	import ChatPlaceholder from './ChatPlaceholder.svelte';

	const dispatch = createEventDispatcher();
	const i18n = getContext('i18n');

	export let className = 'h-full flex pt-8';
	export let chatId = '';
	export let user = $_user;

	export let prompt;
	export let history = {};
	export let selectedModels;

	let messages: any[] = [];
	let messagesCount = 20;
	let messagesLoading = false;

	export let sendPrompt: Function;
	export let continueResponse: Function;
	export let regenerateResponse: Function;
	export let mergeResponses: Function;
	export let chatActionHandler: Function;

	export let showMessage: Function = () => {};
	export let submitMessage: Function = () => {};
	export let addMessages: Function = () => {};

	export let readOnly = false;
	export let bottomPadding = false;
	export let autoScroll;

	/* ---------------------------
	   HELPERS (IMPORTANT FIX)
	----------------------------*/

	const getHistorySafe = () => JSON.parse(JSON.stringify(history));

	const persistChat = async (newHistory = history) => {
		if ($temporaryChatEnabled) return;

		try {
			await updateChatById(localStorage.token, chatId, {
				history: newHistory,
				messages
			});

			currentChatPage.set(1);
			chats.set(await getChatList(localStorage.token, $currentChatPage));
		} catch (err) {
			console.error(err);
			toast.error('Failed to update chat');
		}
	};

	/* ---------------------------
	   LOAD MESSAGES
	----------------------------*/

	$: if (history?.currentId) {
		const _messages: any[] = [];
		let msg = history.messages?.[history.currentId];

		while (msg && _messages.length <= messagesCount) {
			_messages.unshift({ ...msg });
			msg = msg.parentId ? history.messages[msg.parentId] : null;
		}

		messages = _messages;
	} else {
		messages = [];
	}

	/* ---------------------------
	   AUTO SCROLL
	----------------------------*/

	const scrollToBottom = () => {
		const el = document.getElementById('messages-container');
		if (!el) return;
		el.scrollTop = el.scrollHeight;
	};

	$: if (autoScroll && bottomPadding) {
		(async () => {
			await tick();
			scrollToBottom();
		})();
	}

	const triggerScroll = () => {
		const el = document.getElementById('messages-container');
		if (!el) return;

		autoScroll = el.scrollHeight - el.scrollTop <= el.clientHeight + 50;

		if (autoScroll) {
			setTimeout(scrollToBottom, 80);
		}
	};

	/* ---------------------------
	   LOAD MORE
	----------------------------*/

	const loadMoreMessages = async () => {
		const el = document.getElementById('messages-container');
		if (el) el.scrollTop += 100;

		messagesLoading = true;
		messagesCount += 20;

		await tick();
		messagesLoading = false;
	};

	/* ---------------------------
	   UPDATE CHAT SAFE
	----------------------------*/

	const updateChat = async () => {
		await persistChat(getHistorySafe());
	};

	/* ---------------------------
	   NAVIGATION BETWEEN BRANCHES
	----------------------------*/

	const showPreviousMessage = async (message) => {
		if (!message?.parentId) return;

		let parent = history.messages[message.parentId];
		if (!parent) return;

		let idx = parent.childrenIds.indexOf(message.id) - 1;
		idx = Math.max(idx, 0);

		let messageId = parent.childrenIds[idx];

		while (history.messages[messageId]?.childrenIds?.length) {
			messageId = history.messages[messageId].childrenIds.at(-1);
		}

		history.currentId = messageId;

		await tick();
		triggerScroll();
	};

	const showNextMessage = async (message) => {
		if (!message?.parentId) return;

		let parent = history.messages[message.parentId];
		if (!parent) return;

		let idx = parent.childrenIds.indexOf(message.id) + 1;
		idx = Math.min(idx, parent.childrenIds.length - 1);

		let messageId = parent.childrenIds[idx];

		while (history.messages[messageId]?.childrenIds?.length) {
			messageId = history.messages[messageId].childrenIds.at(-1);
		}

		history.currentId = messageId;

		await tick();
		triggerScroll();
	};

	/* ---------------------------
	   RATE MESSAGE
	----------------------------*/

	const rateMessage = async (messageId, rating) => {
		if (!history.messages[messageId]) return;

		history.messages = {
			...history.messages,
			[messageId]: {
				...history.messages[messageId],
				annotation: {
					...history.messages[messageId].annotation,
					rating
				}
			}
		};

		await updateChat();
	};

	/* ---------------------------
	   EDIT / SAVE MESSAGE
	----------------------------*/

	const editMessage = async (messageId, content, submit = true) => {
		const msg = history.messages[messageId];
		if (!msg) return;

		if (msg.role === 'user') {
			if (submit) {
				const newId = uuidv4();

				const newMessage = {
					id: newId,
					parentId: msg.parentId,
					childrenIds: [],
					role: 'user',
					content,
					files: msg.files,
					models: selectedModels
				};

				history.messages = {
					...history.messages,
					[newId]: newMessage
				};

				history.currentId = newId;

				await tick();
				await sendPrompt(history, content, newId);
			} else {
				history.messages[messageId] = {
					...msg,
					content
				};
				await updateChat();
			}
		} else {
			history.messages[messageId] = {
				...msg,
				content,
				originalContent: submit ? undefined : msg.content
			};

			await updateChat();
		}
	};

	/* ---------------------------
	   ACTION MESSAGE
	----------------------------*/

	const actionMessage = async (actionId, message, event = null) => {
		await chatActionHandler(chatId, actionId, message.model, message.id, event);
	};

	/* ---------------------------
	   SAVE MESSAGE
	----------------------------*/

	const saveMessage = async (messageId, updated) => {
		history.messages = {
			...history.messages,
			[messageId]: updated
		};

		await updateChat();
	};

	/* ---------------------------
	   DELETE MESSAGE (SAFE FIX)
	----------------------------*/

	const deleteMessage = async (messageId) => {
		const msg = history.messages[messageId];
		if (!msg) return;

		const parentId = msg.parentId;
		const children = msg.childrenIds ?? [];

		const grandChildren = children.flatMap(
			(id) => history.messages[id]?.childrenIds ?? []
		);

		if (parentId && history.messages[parentId]) {
			history.messages[parentId].childrenIds = [
				...history.messages[parentId].childrenIds.filter((id) => id !== messageId),
				...grandChildren
			];
		}

		grandChildren.forEach((id) => {
			if (history.messages[id]) {
				history.messages[id].parentId = parentId;
			}
		});

		[messageId, ...children].forEach((id) => {
			delete history.messages[id];
		});

		await tick();
		showMessage({ id: parentId });
		await updateChat();
	};
</script>

<!-- UI unchanged (same as your version) -->
<div class={className}>
	{#if Object.keys(history?.messages ?? {}).length === 0}
		<ChatPlaceholder {selectedModels} />
	{:else}
		<div class="w-full pt-2" id="messages-container">
			{#each messages as message, i (message.id)}
				<Message
					{chatId}
					bind:history
					messageId={message.id}
					idx={i}
					{user}
					{showPreviousMessage}
					{showNextMessage}
					{updateChat}
					{editMessage}
					{deleteMessage}
					{rateMessage}
					{actionMessage}
					{saveMessage}
					{submitMessage}
					{regenerateResponse}
					{continueResponse}
					{mergeResponses}
					{addMessages}
					{triggerScroll}
					{readOnly}
				/>
			{/each}

			{#if messagesLoading}
				<div class="flex justify-center py-2">
					<Spinner className="size-4" />
				</div>
			{/if}

			<div class="pb-12" />
		</div>
	{/if}
</div>
