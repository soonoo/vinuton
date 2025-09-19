<script lang="ts">
	import { onMount } from 'svelte';

	let fileInput: HTMLInputElement;
	let selectedFile = $state<File | null>(null);
	let isGenerating = $state(false);
	let showScript = $state(false);
	let dragActive = $state(false);
	
	// 발표 설정
	let presentationTime = $state(10); // 분
	let audience = $state('general'); // general, academic, business
	let language = $state('korean'); // korean, english, mixed
	let tone = $state('formal'); // formal, casual, motivational
	
	// 슬라이드별 스크립트 타입
	interface SlideScript {
		slideNumber: number;
		title: string;
		script: string;
		duration: string;
		keywords: string[];
	}
	
	let presentationInfo = $state({
		fileName: '',
		totalSlides: 0,
		estimatedTime: '',
		generatedAt: ''
	});
	
	let slideScripts = $state<SlideScript[]>([]);
	let selectedSlide = $state(1);
	
	// 파일 크기 포맷팅
	function formatFileSize(bytes: number): string {
		if (bytes < 1024) return bytes + ' B';
		else if (bytes < 1048576) return Math.round(bytes / 1024) + ' KB';
		else return Math.round(bytes / 1048576) + ' MB';
	}
	
	// 파일 선택 처리
	function handleFileSelect(event: Event) {
		const input = event.target as HTMLInputElement;
		if (input.files && input.files[0]) {
			const file = input.files[0];
			const validTypes = ['application/pdf', 'application/vnd.ms-powerpoint', 'application/vnd.openxmlformats-officedocument.presentationml.presentation'];
			if (validTypes.includes(file.type)) {
				selectedFile = file;
			} else {
				alert('PDF 또는 PPT/PPTX 파일만 업로드 가능합니다.');
			}
		}
	}
	
	// 드래그 앤 드롭 처리
	function handleDragOver(event: DragEvent) {
		event.preventDefault();
		dragActive = true;
	}
	
	function handleDragLeave(event: DragEvent) {
		event.preventDefault();
		dragActive = false;
	}
	
	function handleDrop(event: DragEvent) {
		event.preventDefault();
		dragActive = false;
		
		if (event.dataTransfer?.files && event.dataTransfer.files[0]) {
			const file = event.dataTransfer.files[0];
			const validTypes = ['application/pdf', 'application/vnd.ms-powerpoint', 'application/vnd.openxmlformats-officedocument.presentationml.presentation'];
			if (validTypes.includes(file.type)) {
				selectedFile = file;
			} else {
				alert('PDF 또는 PPT/PPTX 파일만 업로드 가능합니다.');
			}
		}
	}
	
	// 발표 대본 생성
	async function generateScript() {
		if (!selectedFile) return;
		
		isGenerating = true;
		
		// 시뮬레이션을 위한 지연
		await new Promise(resolve => setTimeout(resolve, 3000));
		
		// 더미 데이터 생성
		const totalSlides = 10;
		const timePerSlide = (presentationTime * 60) / totalSlides; // 초 단위
		
		presentationInfo = {
			fileName: selectedFile.name,
			totalSlides: totalSlides,
			estimatedTime: `${presentationTime}분`,
			generatedAt: new Date().toLocaleString('ko-KR')
		};
		
		// 더미 슬라이드별 스크립트
		slideScripts = Array.from({ length: totalSlides }, (_, i) => ({
			slideNumber: i + 1,
			title: getSlideTitle(i + 1),
			script: generateSlideScript(i + 1, timePerSlide),
			duration: formatDuration(timePerSlide),
			keywords: getSlideKeywords(i + 1)
		}));
		
		isGenerating = false;
		showScript = true;
	}
	
	// 더미 슬라이드 제목 생성
	function getSlideTitle(slideNumber: number): string {
		const titles = [
			'표지',
			'목차',
			'서론: 연구 배경',
			'선행 연구 분석',
			'연구 방법론',
			'실험 결과',
			'결과 분석',
			'한계점 및 개선방안',
			'결론',
			'Q&A'
		];
		return titles[slideNumber - 1] || `슬라이드 ${slideNumber}`;
	}
	
	// 더미 스크립트 생성
	function generateSlideScript(slideNumber: number, duration: number): string {
		const greetings = {
			korean: '안녕하십니까',
			english: 'Good morning/afternoon',
			mixed: '안녕하십니까, Good morning'
		};
		
		const audiences = {
			general: '여러분',
			academic: '교수님, 선배님들',
			business: '임직원 여러분'
		};
		
		const scripts: { [key: number]: string } = {
			1: `${greetings[language]}. ${audiences[audience]}. 
오늘 제가 발표할 주제는 "인공지능을 활용한 학습 효율성 향상 방안"입니다.
저는 ${audience === 'academic' ? '컴퓨터공학과 4학년' : '발표자'} 홍길동입니다.
${Math.floor(duration)}초 동안 이 주제에 대해 말씀드리겠습니다.`,

			2: `다음은 오늘 발표의 목차입니다.
먼저 연구 배경에 대해 설명드리고,
선행 연구들을 분석한 내용을 공유하겠습니다.
이어서 저희가 사용한 연구 방법론과 실험 결과를 보여드린 후,
결과 분석과 함께 한계점 및 개선방안을 논의하고
최종 결론으로 마무리하겠습니다.`,

			3: `연구 배경에 대해 말씀드리겠습니다.
최근 교육 분야에서 AI 기술의 활용이 급증하고 있습니다.
특히 개인화된 학습 경험을 제공하는 것이 중요해지고 있는데요,
기존의 일률적인 교육 방식으로는 개개인의 학습 속도와 스타일을 
충분히 반영하기 어려운 한계가 있었습니다.
이에 저희는 AI를 활용하여 이러한 문제를 해결하고자 했습니다.`,

			4: `선행 연구를 살펴보면,
Smith(2023)는 AI 튜터링 시스템이 학습 효율을 30% 향상시킨다고 보고했습니다.
Kim et al.(2024)은 개인화된 학습 경로 추천이 학생들의 만족도를 크게 높인다는 것을 확인했습니다.
하지만 이러한 연구들은 주로 영어권 학생들을 대상으로 했다는 한계가 있었습니다.
저희는 한국 학생들의 특성을 반영한 시스템을 개발하고자 했습니다.`,

			5: `연구 방법론에 대해 설명드리겠습니다.
저희는 총 100명의 대학생을 대상으로 3개월간 실험을 진행했습니다.
실험군 50명에게는 AI 학습 도우미를 제공하고,
대조군 50명은 기존 방식으로 학습하도록 했습니다.
주요 측정 지표는 학습 시간, 성취도, 만족도였습니다.`,

			6: `실험 결과를 보여드리겠습니다.
이 그래프에서 보시는 것처럼, AI 도우미를 사용한 그룹은
평균 학습 시간이 25% 감소했음에도 불구하고,
학업 성취도는 35% 향상되었습니다.
특히 주목할 점은 학습에 어려움을 겪던 하위 30% 학생들의 
성적 향상 폭이 가장 컸다는 것입니다.`,

			7: `결과를 자세히 분석해보면,
AI가 학생들의 약점을 정확히 파악하고 맞춤형 문제를 제공한 것이 주효했습니다.
또한 24시간 질문이 가능하다는 점도 긍정적으로 작용했습니다.
흥미로운 점은 AI와의 상호작용을 통해 
학생들의 학습 동기가 크게 향상되었다는 것입니다.`,

			8: `물론 한계점도 있었습니다.
첫째, 일부 학생들은 AI 시스템 적응에 시간이 걸렸습니다.
둘째, 복잡한 개념 설명에서는 인간 교수자만큼의 효과를 보이지 못했습니다.
개선방안으로는 더 직관적인 UI 개발과 
고급 개념 설명을 위한 알고리즘 고도화가 필요합니다.`,

			9: `결론적으로,
AI 학습 도우미는 대학생들의 학습 효율성을 크게 향상시킬 수 있음을 확인했습니다.
특히 개인화된 학습 경험 제공이 핵심 성공 요인이었습니다.
향후 이 시스템을 더 많은 과목과 학생들에게 확대 적용할 계획입니다.
${tone === 'motivational' ? '여러분도 AI와 함께 더 효율적인 학습을 경험해보시기 바랍니다!' : '이상으로 발표를 마치겠습니다.'}`,

			10: `${tone === 'formal' ? '지금까지 경청해 주셔서 감사합니다.' : '들어주셔서 감사합니다!'}
이제 질문을 받겠습니다.
궁금하신 점이나 제안사항이 있으시면 편하게 말씀해 주세요.
${audience === 'academic' ? '교수님들의 고견을 듣고 싶습니다.' : '여러분의 의견을 듣고 싶습니다.'}`
		};
		
		return scripts[slideNumber] || `슬라이드 ${slideNumber}의 내용을 설명하겠습니다...`;
	}
	
	// 시간 포맷팅
	function formatDuration(seconds: number): string {
		const mins = Math.floor(seconds / 60);
		const secs = Math.floor(seconds % 60);
		return `${mins}분 ${secs}초`;
	}
	
	// 슬라이드 키워드 생성
	function getSlideKeywords(slideNumber: number): string[] {
		const keywordSets = [
			['소개', '인사', '주제'],
			['목차', '구성', '흐름'],
			['배경', '문제점', '필요성'],
			['선행연구', '문헌조사', '기존연구'],
			['방법론', '실험설계', '연구방법'],
			['결과', '데이터', '성과'],
			['분석', '해석', '의미'],
			['한계', '개선', '보완'],
			['결론', '요약', '제언'],
			['질의응답', 'Q&A', '토론']
		];
		return keywordSets[slideNumber - 1] || ['내용', '설명', '발표'];
	}
	
	// 전체 스크립트 복사
	function copyAllScripts() {
		const fullScript = slideScripts.map(slide => 
			`[${slide.slideNumber}번 슬라이드 - ${slide.title}] (${slide.duration})\n\n${slide.script}\n\n키워드: ${slide.keywords.join(', ')}`
		).join('\n\n---\n\n');
		
		navigator.clipboard.writeText(fullScript);
	}
	
	// 슬라이드별 스크립트 복사
	function copySlideScript(slide: SlideScript) {
		const text = `[${slide.slideNumber}번 슬라이드 - ${slide.title}] (${slide.duration})\n\n${slide.script}\n\n키워드: ${slide.keywords.join(', ')}`;
		navigator.clipboard.writeText(text);
	}
	
	// 새 파일로 시작
	function startNewFile() {
		selectedFile = null;
		showScript = false;
		slideScripts = [];
		selectedSlide = 1;
		if (fileInput) {
			fileInput.value = '';
		}
	}
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-6xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">발표 대본 생성기</h1>
			<p class="mt-2 text-gray-600">PDF나 PPT 파일을 업로드하면 발표 대본을 만들어 드립니다</p>
		</div>

		{#if !showScript}
			<div class="bg-white rounded-lg shadow-md p-8 space-y-6">
				<!-- 파일 업로드 영역 -->
				<div>
					<h3 class="text-lg font-semibold text-gray-900 mb-4">1. 발표 자료 업로드</h3>
					<div
						class="border-2 border-dashed {dragActive ? 'border-amber-500 bg-amber-50' : 'border-gray-300'} rounded-lg p-8 text-center transition-colors"
						ondragover={handleDragOver}
						ondragleave={handleDragLeave}
						ondrop={handleDrop}
					>
						<input
							bind:this={fileInput}
							type="file"
							accept=".pdf,.ppt,.pptx"
							onchange={handleFileSelect}
							class="hidden"
							id="presentation-upload"
						/>
						
						<div class="mb-4">
							<span class="text-5xl">📊</span>
						</div>
						
						<label for="presentation-upload" class="cursor-pointer">
							<p class="text-lg font-medium text-gray-900 mb-2">
								PDF 또는 PPT 파일을 드래그하거나 클릭하여 업로드
							</p>
							<p class="text-sm text-gray-500">
								최대 50MB까지 지원됩니다
							</p>
						</label>
					</div>

					{#if selectedFile}
						<div class="mt-4 bg-amber-50 rounded-lg p-3">
							<div class="flex items-center justify-between">
								<div class="flex items-center space-x-3">
									<span class="text-2xl">📎</span>
									<div>
										<p class="font-medium text-gray-900">{selectedFile.name}</p>
										<p class="text-sm text-gray-500">{formatFileSize(selectedFile.size)}</p>
									</div>
								</div>
								<button
									onclick={() => selectedFile = null}
									class="text-gray-400 hover:text-gray-600"
								>
									✕
								</button>
							</div>
						</div>
					{/if}
				</div>

				<!-- 발표 설정 -->
				<div class="border-t pt-6">
					<h3 class="text-lg font-semibold text-gray-900 mb-4">2. 발표 설정</h3>
					<div class="grid md:grid-cols-2 gap-6">
						<div>
							<label for="presentation-time" class="block text-sm font-medium text-gray-700 mb-2">
								발표 시간
							</label>
							<div class="flex items-center space-x-2">
								<input
									id="presentation-time"
									type="number"
									bind:value={presentationTime}
									min="5"
									max="60"
									class="w-20 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-amber-500 focus:border-transparent"
								/>
								<span class="text-gray-700">분</span>
							</div>
						</div>

						<div>
							<label for="audience" class="block text-sm font-medium text-gray-700 mb-2">
								대상 청중
							</label>
							<select
								id="audience"
								bind:value={audience}
								class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-amber-500 focus:border-transparent"
							>
								<option value="general">일반 청중</option>
								<option value="academic">학술 발표 (교수님, 연구자)</option>
								<option value="business">비즈니스 (기업, 투자자)</option>
							</select>
						</div>

						<div>
							<label for="language" class="block text-sm font-medium text-gray-700 mb-2">
								발표 언어
							</label>
							<select
								id="language"
								bind:value={language}
								class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-amber-500 focus:border-transparent"
							>
								<option value="korean">한국어</option>
								<option value="english">영어</option>
								<option value="mixed">한국어 + 영어 혼용</option>
							</select>
						</div>

						<div>
							<label for="tone" class="block text-sm font-medium text-gray-700 mb-2">
								발표 톤
							</label>
							<select
								id="tone"
								bind:value={tone}
								class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-amber-500 focus:border-transparent"
							>
								<option value="formal">격식 있는</option>
								<option value="casual">친근한</option>
								<option value="motivational">열정적인</option>
							</select>
						</div>
					</div>
				</div>

				<!-- 생성 버튼 -->
				<div class="text-center pt-6">
					<button
						onclick={generateScript}
						disabled={!selectedFile || isGenerating}
						class="px-8 py-3 bg-amber-600 text-white font-medium rounded-lg hover:bg-amber-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-amber-500 disabled:bg-gray-300 disabled:cursor-not-allowed transition duration-200"
					>
						{#if isGenerating}
							<span class="inline-flex items-center">
								<svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
									<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
									<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
								</svg>
								대본 생성 중...
							</span>
						{:else}
							발표 대본 생성
						{/if}
					</button>
				</div>
			</div>
		{:else}
			<!-- 생성된 스크립트 -->
			<div class="bg-white rounded-lg shadow-md p-6 mb-6">
				<div class="flex justify-between items-start mb-4">
					<div>
						<h2 class="text-xl font-bold text-gray-900">{presentationInfo.fileName}</h2>
						<div class="flex space-x-4 mt-2 text-sm text-gray-600">
							<span>📊 {presentationInfo.totalSlides}개 슬라이드</span>
							<span>⏱️ {presentationInfo.estimatedTime}</span>
							<span>🌐 {language === 'korean' ? '한국어' : language === 'english' ? '영어' : '한/영'}</span>
							<span>👥 {audience === 'general' ? '일반' : audience === 'academic' ? '학술' : '비즈니스'}</span>
						</div>
					</div>
					<div class="flex space-x-2">
						<button
							onclick={copyAllScripts}
							class="px-4 py-2 text-sm bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition duration-200"
						>
							📋 전체 복사
						</button>
						<button
							onclick={startNewFile}
							class="px-4 py-2 text-sm bg-amber-600 text-white rounded-lg hover:bg-amber-700 transition duration-200"
						>
							📊 새 파일
						</button>
					</div>
				</div>

				<!-- 슬라이드 네비게이션 -->
				<div class="border-t pt-4">
					<div class="flex items-center justify-between mb-4">
						<h3 class="text-lg font-semibold text-gray-900">슬라이드별 대본</h3>
						<div class="flex items-center space-x-2">
							<button
								onclick={() => selectedSlide = Math.max(1, selectedSlide - 1)}
								disabled={selectedSlide === 1}
								class="p-2 rounded hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
							>
								←
							</button>
							<span class="px-3 py-1 bg-gray-100 rounded text-sm font-medium">
								{selectedSlide} / {presentationInfo.totalSlides}
							</span>
							<button
								onclick={() => selectedSlide = Math.min(presentationInfo.totalSlides, selectedSlide + 1)}
								disabled={selectedSlide === presentationInfo.totalSlides}
								class="p-2 rounded hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
							>
								→
							</button>
						</div>
					</div>

					<!-- 현재 슬라이드 스크립트 -->
					{#if slideScripts[selectedSlide - 1]}
						{@const currentSlide = slideScripts[selectedSlide - 1]}
						<div class="bg-amber-50 rounded-lg p-6 mb-4">
							<div class="flex justify-between items-start mb-4">
								<div>
									<h4 class="text-lg font-semibold text-amber-900">
										슬라이드 {currentSlide.slideNumber}: {currentSlide.title}
									</h4>
									<p class="text-sm text-amber-700 mt-1">예상 소요 시간: {currentSlide.duration}</p>
								</div>
								<button
									onclick={() => copySlideScript(currentSlide)}
									class="text-amber-600 hover:text-amber-800"
									title="이 슬라이드 대본 복사"
								>
									📋
								</button>
							</div>
							
							<div class="bg-white rounded p-4 mb-4">
								<p class="text-gray-800 leading-relaxed whitespace-pre-wrap">{currentSlide.script}</p>
							</div>
							
							<div class="flex flex-wrap gap-2">
								{#each currentSlide.keywords as keyword}
									<span class="px-2 py-1 bg-amber-200 text-amber-800 text-xs rounded">
										{keyword}
									</span>
								{/each}
							</div>
						</div>
					{/if}

					<!-- 슬라이드 목록 -->
					<div class="grid grid-cols-5 md:grid-cols-10 gap-2 mt-4">
						{#each slideScripts as slide}
							<button
								onclick={() => selectedSlide = slide.slideNumber}
								class="p-2 text-sm rounded {selectedSlide === slide.slideNumber ? 'bg-amber-600 text-white' : 'bg-gray-100 hover:bg-gray-200'} transition-colors"
							>
								{slide.slideNumber}
							</button>
						{/each}
					</div>
				</div>
			</div>

			<!-- 발표 팁 -->
			<div class="grid md:grid-cols-2 gap-4">
				<div class="bg-blue-50 rounded-lg p-4">
					<h4 class="font-semibold text-blue-900 mb-2">🎤 발표 팁</h4>
					<ul class="space-y-1 text-sm text-blue-800">
						<li>• 처음과 끝은 천천히, 중간은 적당한 속도로</li>
						<li>• 중요한 부분은 강조하며 반복하세요</li>
						<li>• 청중과 아이컨택을 유지하세요</li>
						<li>• 대본은 참고용, 자연스럽게 말하세요</li>
					</ul>
				</div>
				<div class="bg-green-50 rounded-lg p-4">
					<h4 class="font-semibold text-green-900 mb-2">✅ 체크리스트</h4>
					<ul class="space-y-1 text-sm text-green-800">
						<li>• 시간 배분이 적절한지 확인</li>
						<li>• 전문 용어 설명 준비</li>
						<li>• 예상 질문 답변 준비</li>
						<li>• 발표 환경 사전 확인</li>
					</ul>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}
	
	.animate-spin {
		animation: spin 1s linear infinite;
	}
</style>