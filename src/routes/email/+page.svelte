<script lang="ts">
	import { onMount } from 'svelte';

	let inputEmail = $state('');
	let correctedEmail = $state('');
	let showResult = $state(false);

	function handleCorrection() {
		// 임시 교정 로직 (나중에 API로 대체)
		if (inputEmail.trim()) {
			// 더미 교정 예시
			correctedEmail = `안녕하세요 교수님,

저는 [학과] [학년] [이름]입니다.

${inputEmail}

바쁘신 중에도 시간 내주셔서 감사합니다.
답변 주시면 감사하겠습니다.

감사합니다.
[이름] 드림`;
			
			showResult = true;
		}
	}

	function copyToClipboard() {
		navigator.clipboard.writeText(correctedEmail);
		// 복사 완료 피드백은 추후 추가 가능
	}
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-3xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">이메일 교정 도우미</h1>
			<p class="mt-2 text-gray-600">교수님께 보낼 이메일을 더 정중하게 다듬어 드립니다</p>
		</div>

		<div class="bg-white rounded-lg shadow-md p-6 space-y-6">
			<!-- 입력 영역 -->
			<div>
				<label for="email-input" class="block text-sm font-medium text-gray-700 mb-2">
					이메일 내용을 입력하세요
				</label>
				<textarea
					id="email-input"
					bind:value={inputEmail}
					placeholder="교수님에게 뭐라고 보낼까?"
					rows="8"
					class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent resize-none transition duration-200"
				></textarea>
			</div>

			<!-- 교정하기 버튼 -->
			<div class="text-center">
				<button
					onclick={handleCorrection}
					disabled={!inputEmail.trim()}
					class="px-8 py-3 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:bg-gray-300 disabled:cursor-not-allowed transition duration-200"
				>
					교정하기
				</button>
			</div>

			<!-- 결과 영역 -->
			{#if showResult}
				<div class="border-t pt-6 space-y-4 animate-fade-in">
					<div class="flex justify-between items-center">
						<h2 class="text-lg font-semibold text-gray-900">교정된 이메일</h2>
						<button
							onclick={copyToClipboard}
							class="px-4 py-2 text-sm bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition duration-200"
						>
							📋 복사하기
						</button>
					</div>
					
					<div class="bg-blue-50 rounded-lg p-4">
						<pre class="whitespace-pre-wrap text-sm text-gray-800 leading-relaxed font-sans">{correctedEmail}</pre>
					</div>
				</div>
			{/if}
		</div>

		<!-- 사용 팁 -->
		<div class="mt-6 bg-blue-50 rounded-lg p-4">
			<h3 class="font-semibold text-blue-900 mb-2">💡 사용 팁</h3>
			<ul class="space-y-1 text-sm text-blue-800">
				<li>• 간단명료하게 용건을 작성해주세요</li>
				<li>• 구체적인 질문이나 요청사항을 포함하면 좋습니다</li>
				<li>• 교정된 내용에서 [대괄호] 부분은 꼭 수정해주세요</li>
			</ul>
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
</style>