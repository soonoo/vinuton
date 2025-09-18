<script lang="ts">
	import { onMount } from 'svelte';

	let inputResume = $state('');
	let improvedResume = $state('');
	let feedback = $state('');
	let showResult = $state(false);

	function handleImprovement() {
		// 임시 개선 로직 (나중에 API로 대체)
		if (inputResume.trim()) {
			// 더미 개선된 자기소개서
			improvedResume = `[개선된 자기소개서]

${inputResume}

[추가 개선 사항]
• 구체적인 성과와 수치를 포함하여 신뢰성을 높였습니다
• 기업이 원하는 인재상과 연결된 경험을 강조했습니다
• 문장을 간결하고 명확하게 다듬었습니다
• 적극적이고 긍정적인 어조로 수정했습니다`;
			
			// 더미 피드백
			feedback = `📝 자기소개서 피드백

✅ 잘한 점:
• 전반적인 구성이 논리적입니다
• 자신의 경험을 구체적으로 서술했습니다
• 진정성 있는 내용이 담겨있습니다

💡 개선할 점:
• 첫 문장을 더 임팩트 있게 시작해보세요
• 구체적인 수치나 성과를 추가하면 좋겠습니다
• 회사에 기여할 수 있는 부분을 더 강조해주세요
• 마무리 부분에서 포부를 더 명확히 표현해보세요

🎯 추천 키워드:
• 문제 해결 능력
• 협업 경험
• 성장 가능성
• 도전 정신`;
			
			showResult = true;
		}
	}

	function copyToClipboard() {
		navigator.clipboard.writeText(improvedResume);
		// 복사 완료 피드백은 추후 추가 가능
	}
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-4xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">자기소개서/이력서 첨삭 도우미</h1>
			<p class="mt-2 text-gray-600">AI가 여러분의 자기소개서를 분석하고 개선점을 제안합니다</p>
		</div>

		<div class="bg-white rounded-lg shadow-md p-6 space-y-6">
			<!-- 입력 영역 -->
			<div>
				<label for="resume-input" class="block text-sm font-medium text-gray-700 mb-2">
					자기소개서/이력서 내용을 입력하세요
				</label>
				<textarea
					id="resume-input"
					bind:value={inputResume}
					placeholder="자기소개서나 이력서 내용을 입력해주세요..."
					rows="12"
					class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent resize-none transition duration-200"
				></textarea>
			</div>

			<!-- 첨삭하기 버튼 -->
			<div class="text-center">
				<button
					onclick={handleImprovement}
					disabled={!inputResume.trim()}
					class="px-8 py-3 bg-indigo-600 text-white font-medium rounded-lg hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:bg-gray-300 disabled:cursor-not-allowed transition duration-200"
				>
					첨삭받기
				</button>
			</div>

			<!-- 결과 영역 -->
			{#if showResult}
				<div class="border-t pt-6 space-y-6 animate-fade-in">
					<!-- 피드백 섹션 -->
					<div class="bg-indigo-50 rounded-lg p-6">
						<pre class="whitespace-pre-wrap text-sm text-gray-800 leading-relaxed font-sans">{feedback}</pre>
					</div>

					<!-- 개선된 자기소개서 섹션 -->
					<div class="space-y-4">
						<div class="flex justify-between items-center">
							<h2 class="text-lg font-semibold text-gray-900">개선된 자기소개서</h2>
							<button
								onclick={copyToClipboard}
								class="px-4 py-2 text-sm bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition duration-200"
							>
								📋 복사하기
							</button>
						</div>
						
						<div class="bg-green-50 rounded-lg p-4">
							<pre class="whitespace-pre-wrap text-sm text-gray-800 leading-relaxed font-sans">{improvedResume}</pre>
						</div>
					</div>
				</div>
			{/if}
		</div>

		<!-- 사용 팁 -->
		<div class="mt-6 grid md:grid-cols-2 gap-4">
			<div class="bg-indigo-50 rounded-lg p-4">
				<h3 class="font-semibold text-indigo-900 mb-2">💡 작성 팁</h3>
				<ul class="space-y-1 text-sm text-indigo-800">
					<li>• 구체적인 경험과 성과를 포함하세요</li>
					<li>• STAR 기법(상황-과제-행동-결과)을 활용하세요</li>
					<li>• 지원하는 직무와 연관된 역량을 강조하세요</li>
					<li>• 긍정적이고 적극적인 표현을 사용하세요</li>
				</ul>
			</div>

			<div class="bg-green-50 rounded-lg p-4">
				<h3 class="font-semibold text-green-900 mb-2">📌 주의사항</h3>
				<ul class="space-y-1 text-sm text-green-800">
					<li>• 개인정보는 입력하지 마세요</li>
					<li>• AI 피드백은 참고용으로만 활용하세요</li>
					<li>• 최종 검토는 반드시 본인이 해주세요</li>
					<li>• 회사별 특성에 맞게 수정하세요</li>
				</ul>
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
</style>