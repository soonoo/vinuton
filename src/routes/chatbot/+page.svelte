<script lang="ts">
	import { onMount, tick } from 'svelte';

	interface Message {
		id: number;
		text: string;
		sender: 'user' | 'bot';
		timestamp: Date;
	}

	let messages = $state<Message[]>([
		{
			id: 1,
			text: '안녕하세요! 대학생활에 필요한 것이 있으면 무엇이든 물어보세요 😊',
			sender: 'bot',
			timestamp: new Date()
		}
	]);

	let inputMessage = $state('');
	let isTyping = $state(false);
	let messageContainer: HTMLDivElement;
	let messageIdCounter = 2;
	let isComposing = false;

	// 메시지 전송
	async function sendMessage() {
		if (!inputMessage.trim()) return;

		// 사용자 메시지 추가
		const userMessage: Message = {
			id: messageIdCounter++,
			text: inputMessage,
			sender: 'user',
			timestamp: new Date()
		};
		messages = [...messages, userMessage];
		
		// 입력창 초기화
		const messageText = inputMessage;
		inputMessage = '';

		// 봇이 타이핑 중 표시
		await tick();
		await scrollToBottom();
		
		isTyping = true;

		// 더미 응답 생성 (실제로는 API 호출)
		setTimeout(async () => {
			const botResponse = getBotResponse(messageText);
			const botMessage: Message = {
				id: messageIdCounter++,
				text: botResponse,
				sender: 'bot',
				timestamp: new Date()
			};
			messages = [...messages, botMessage];
			isTyping = false;
			
			await tick();
			await scrollToBottom();
		}, 1000 + Math.random() * 1000);
	}

	// 더미 봇 응답 생성
	function getBotResponse(userMessage: string): string {
		const lowerMessage = userMessage.toLowerCase();

		// 키워드 기반 응답
		if (lowerMessage.includes('이메일')) {
			return '이메일 작성에 도움이 필요하신가요? 저희 이메일 교정 도우미를 이용해보세요! 교수님께 보내는 이메일을 더 정중하게 다듬어 드립니다. 📧';
		} else if (lowerMessage.includes('자소서') || lowerMessage.includes('자기소개서') || lowerMessage.includes('이력서')) {
			return '자기소개서 작성이 막막하시죠? 자기소개서/이력서 첨삭 서비스를 이용해보세요! AI가 분석하고 개선점을 제안해드립니다. 📝';
		} else if (lowerMessage.includes('면접')) {
			return '면접 준비 중이시군요! 면접 예상 질문 생성기로 맞춤형 질문을 받아보세요. 전공과 수강 과목 기반으로 예상 질문을 만들어드립니다. 💼';
		} else if (lowerMessage.includes('회의') || lowerMessage.includes('회의록')) {
			return '회의록 작성이 번거로우셨나요? 회의록 자동 생성기로 녹음만 하면 요약본을 만들어드립니다. 액션 아이템까지 정리해드려요! 🎙️';
		} else if (lowerMessage.includes('pdf') || lowerMessage.includes('논문') || lowerMessage.includes('보고서')) {
			return 'PDF 문서를 빠르게 파악하고 싶으시다면 PDF 문서 분석기를 사용해보세요. 페이지별로 요약하고 핵심 키워드를 추출해드립니다. 📑';
		} else if (lowerMessage.includes('시간표')) {
			return '시간표 짜기가 어려우시죠? 곧 시간표 자동 생성 기능도 추가될 예정이에요! 기대해주세요. 📅';
		} else if (lowerMessage.includes('공부') || lowerMessage.includes('학습')) {
			return '효율적인 공부를 위해 뽀모도로 타이머나 학습 플래너 기능을 준비 중입니다. 어떤 기능이 더 필요하신가요? 📚';
		} else if (lowerMessage.includes('안녕')) {
			return '안녕하세요! 오늘도 좋은 하루 되세요. 어떤 도움이 필요하신가요? 😊';
		} else if (lowerMessage.includes('감사') || lowerMessage.includes('고마')) {
			return '도움이 되어 기쁘네요! 언제든지 필요하시면 말씀해주세요. 화이팅! 💪';
		} else {
			const responses = [
				'흥미로운 질문이네요! 구체적으로 어떤 부분이 궁금하신가요?',
				'그것에 대해 더 자세히 알려주시면 더 정확한 답변을 드릴 수 있을 것 같아요.',
				'좋은 질문입니다! 제가 제공하는 서비스 중 어떤 것이 도움이 될 수 있을지 함께 찾아봐요.',
				'대학생활에서 가장 어려운 부분은 무엇인가요? 제가 도와드릴 수 있는 부분이 있을 거예요!',
				'저는 이메일 작성, 자소서 첨삭, 면접 준비, 회의록 작성, PDF 분석 등을 도와드릴 수 있어요. 어떤 것이 필요하신가요?'
			];
			return responses[Math.floor(Math.random() * responses.length)];
		}
	}

	// 자동 스크롤
	async function scrollToBottom() {
		if (messageContainer) {
			messageContainer.scrollTop = messageContainer.scrollHeight;
		}
	}

	// 시간 포맷팅
	function formatTime(date: Date): string {
		return date.toLocaleTimeString('ko-KR', { 
			hour: '2-digit', 
			minute: '2-digit' 
		});
	}

	// Enter 키로 전송
	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Enter' && !event.shiftKey) {
			event.preventDefault();
			if (!isComposing) {
				sendMessage();
			}
		}
	}

	onMount(() => {
		scrollToBottom();
	});
</script>

<div class="h-screen bg-gray-50 flex flex-col">
	<!-- 헤더 -->
	<div class="bg-white shadow-sm border-b fixed top-0 left-0 right-0 z-10">
		<div class="max-w-4xl mx-auto px-4 py-4">
			<div class="flex items-center space-x-3">
				<div class="w-10 h-10 bg-teal-500 rounded-full flex items-center justify-center">
					<span class="text-xl">🤖</span>
				</div>
				<div>
					<h1 class="text-lg font-semibold text-gray-900">대학생 도우미 챗봇</h1>
					<p class="text-xs text-gray-500">항상 온라인</p>
				</div>
			</div>
		</div>
	</div>

	<!-- 채팅 영역 -->
	<div class="flex-1 overflow-hidden pt-[72px] pb-[140px]">
		<div 
			bind:this={messageContainer}
			class="h-full overflow-y-auto px-4 py-6 max-w-4xl mx-auto"
		>
			<div class="space-y-4">
				{#each messages as message (message.id)}
					<div class="flex {message.sender === 'user' ? 'justify-end' : 'justify-start'} animate-fade-in">
						<div class="flex max-w-xs lg:max-w-md {message.sender === 'user' ? 'flex-row-reverse' : 'flex-row'} items-end space-x-2">
							<!-- 아바타 -->
							<div class="flex-shrink-0">
								{#if message.sender === 'bot'}
									<div class="w-8 h-8 bg-teal-500 rounded-full flex items-center justify-center">
										<span class="text-sm">🤖</span>
									</div>
								{:else}
									<div class="w-8 h-8 bg-gray-400 rounded-full flex items-center justify-center">
										<span class="text-sm">👤</span>
									</div>
								{/if}
							</div>

							<!-- 메시지 버블 -->
							<div>
								<div class="px-4 py-2 rounded-2xl {message.sender === 'user' 
									? 'bg-teal-600 text-white rounded-br-none' 
									: 'bg-white text-gray-800 rounded-bl-none shadow-md'}">
									<p class="text-sm leading-relaxed">{message.text}</p>
								</div>
								<p class="text-xs text-gray-500 mt-1 {message.sender === 'user' ? 'text-right mr-2' : 'ml-2'}">
									{formatTime(message.timestamp)}
								</p>
							</div>
						</div>
					</div>
				{/each}

				<!-- 타이핑 인디케이터 -->
				{#if isTyping}
					<div class="flex justify-start animate-fade-in">
						<div class="flex items-end space-x-2">
							<div class="w-8 h-8 bg-teal-500 rounded-full flex items-center justify-center">
								<span class="text-sm">🤖</span>
							</div>
							<div class="bg-white rounded-2xl rounded-bl-none shadow-md px-4 py-3">
								<div class="flex space-x-1">
									<div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0ms"></div>
									<div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 150ms"></div>
									<div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 300ms"></div>
								</div>
							</div>
						</div>
					</div>
				{/if}
			</div>
		</div>
	</div>

	<!-- 입력 영역 -->
	<div class="bg-white border-t fixed bottom-0 left-0 right-0 z-10">
		<div class="max-w-4xl mx-auto p-4">
			<div class="flex space-x-2">
				<input
					type="text"
					bind:value={inputMessage}
					onkeydown={handleKeydown}
					oncompositionstart={() => isComposing = true}
					oncompositionend={() => isComposing = false}
					placeholder="메시지를 입력하세요..."
					class="flex-1 px-4 py-3 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-teal-500 focus:border-transparent"
				/>
				<button
					type="button"
					onclick={sendMessage}
					disabled={!inputMessage.trim()}
					class="px-6 py-3 bg-teal-600 text-white rounded-full hover:bg-teal-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-teal-500 disabled:bg-gray-300 disabled:cursor-not-allowed transition duration-200"
				>
					<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
					</svg>
				</button>
			</div>

			<!-- 빠른 액션 버튼들 -->
			<div class="mt-2 flex flex-wrap gap-2">
				<button
					onclick={() => inputMessage = '이메일 작성을 도와주세요'}
					class="px-3 py-1 bg-gray-100 text-gray-700 text-sm rounded-full hover:bg-gray-200 transition"
				>
					📧 이메일 도움
				</button>
				<button
					onclick={() => inputMessage = '자소서를 첨삭받고 싶어요'}
					class="px-3 py-1 bg-gray-100 text-gray-700 text-sm rounded-full hover:bg-gray-200 transition"
				>
					📝 자소서 첨삭
				</button>
				<button
					onclick={() => inputMessage = '면접 준비를 하고 있어요'}
					class="px-3 py-1 bg-gray-100 text-gray-700 text-sm rounded-full hover:bg-gray-200 transition"
				>
					💼 면접 준비
				</button>
				<button
					onclick={() => inputMessage = 'PDF 문서를 요약하고 싶어요'}
					class="px-3 py-1 bg-gray-100 text-gray-700 text-sm rounded-full hover:bg-gray-200 transition"
				>
					📑 PDF 요약
				</button>
			</div>
		</div>
	</div>
</div>

<style>
	@keyframes fade-in {
		from {
			opacity: 0;
			transform: translateY(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.animate-fade-in {
		animation: fade-in 0.3s ease-out;
	}

	@keyframes bounce {
		0%, 60%, 100% {
			transform: translateY(0);
		}
		30% {
			transform: translateY(-10px);
		}
	}

	.animate-bounce {
		animation: bounce 1.4s infinite ease-in-out;
	}
</style>
