<script lang="ts">
	import { onMount } from 'svelte';

	let fileInput: HTMLInputElement;
	let selectedFile = $state<File | null>(null);
	let isAnalyzing = $state(false);
	let showAnalysis = $state(false);
	let dragActive = $state(false);
	
	// PDF 분석 결과 타입
	interface PageSummary {
		pageNumber: number;
		summary: string;
		keywords: string[];
		type: 'text' | 'image' | 'mixed';
	}
	
	let pdfInfo = $state({
		fileName: '',
		totalPages: 0,
		fileSize: '',
		analysisTime: ''
	});
	
	let pageSummaries = $state<PageSummary[]>([]);
	let selectedPage = $state(1);
	
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
			if (file.type === 'application/pdf') {
				selectedFile = file;
			} else {
				alert('PDF 파일만 업로드 가능합니다.');
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
			if (file.type === 'application/pdf') {
				selectedFile = file;
			} else {
				alert('PDF 파일만 업로드 가능합니다.');
			}
		}
	}
	
	// PDF 분석 시작
	async function analyzePDF() {
		if (!selectedFile) return;
		
		isAnalyzing = true;
		
		// 시뮬레이션을 위한 지연
		await new Promise(resolve => setTimeout(resolve, 2000));
		
		// 더미 데이터 생성
		pdfInfo = {
			fileName: selectedFile.name,
			totalPages: 15,
			fileSize: formatFileSize(selectedFile.size),
			analysisTime: new Date().toLocaleString('ko-KR')
		};
		
		// 더미 페이지별 요약
		pageSummaries = Array.from({ length: 15 }, (_, i) => ({
			pageNumber: i + 1,
			summary: getDummySummary(i + 1),
			keywords: getDummyKeywords(i + 1),
			type: i % 3 === 0 ? 'image' : i % 3 === 1 ? 'mixed' : 'text'
		}));
		
		isAnalyzing = false;
		showAnalysis = true;
	}
	
	// 더미 요약 데이터 생성
	function getDummySummary(pageNumber: number): string {
		const summaries = [
			'이 페이지는 인공지능의 기본 개념과 역사를 설명합니다. 튜링 테스트부터 현대의 딥러닝까지 AI의 발전 과정을 다루고 있으며, 주요 마일스톤과 핵심 연구자들의 기여를 소개합니다.',
			'머신러닝의 세 가지 주요 패러다임인 지도학습, 비지도학습, 강화학습에 대해 설명합니다. 각 방법론의 특징과 적용 사례, 장단점을 비교 분석하고 있습니다.',
			'신경망의 구조와 작동 원리를 상세히 설명합니다. 퍼셉트론부터 다층 신경망까지의 발전 과정과 역전파 알고리즘의 수학적 원리를 다룹니다.',
			'컴퓨터 비전 분야의 최신 동향과 응용 사례를 소개합니다. CNN의 구조와 이미지 인식, 객체 탐지, 세그멘테이션 등의 주요 task들을 설명합니다.',
			'자연어 처리의 기초 개념과 최신 트렌드를 다룹니다. 토큰화, 임베딩, 트랜스포머 아키텍처 등 NLP의 핵심 기술들을 설명합니다.',
			'강화학습의 이론적 배경과 실제 응용 사례를 소개합니다. Q-learning, Policy Gradient, Actor-Critic 등의 주요 알고리즘을 설명합니다.',
			'AI 윤리와 사회적 영향에 대해 논의합니다. 편향성, 프라이버시, 일자리 대체 등 AI가 가져올 수 있는 문제점과 해결 방안을 제시합니다.',
			'의료 분야에서의 AI 활용 사례를 소개합니다. 진단 보조, 신약 개발, 맞춤형 치료 등 헬스케어 AI의 현재와 미래를 전망합니다.',
			'자율주행 기술의 현황과 과제를 분석합니다. 센서 기술, 인지 알고리즘, 의사결정 시스템 등 자율주행의 핵심 요소들을 설명합니다.',
			'금융 분야의 AI 응용을 다룹니다. 알고리즘 트레이딩, 신용 평가, 사기 탐지 등 핀테크 분야의 AI 활용 사례를 소개합니다.',
			'교육 분야에서의 AI 활용 방안을 제시합니다. 개인화 학습, 자동 채점, 학습 분석 등 에듀테크의 혁신적인 사례들을 설명합니다.',
			'AI 모델의 설명가능성과 해석가능성의 중요성을 강조합니다. XAI 기술과 블랙박스 문제 해결을 위한 다양한 접근법을 소개합니다.',
			'엣지 AI와 임베디드 시스템에서의 AI 구현을 다룹니다. 경량화 기법, 하드웨어 가속, 실시간 추론 등의 기술적 과제를 설명합니다.',
			'AI의 미래 전망과 AGI(Artificial General Intelligence)로의 발전 가능성을 논의합니다. 현재의 한계와 돌파구, 장기적 비전을 제시합니다.',
			'참고문헌과 추가 학습 자료를 제공합니다. AI 분야의 주요 논문, 교재, 온라인 코스, 컨퍼런스 등 유용한 리소스들을 정리했습니다.'
		];
		
		return summaries[(pageNumber - 1) % summaries.length];
	}
	
	// 더미 키워드 데이터 생성
	function getDummyKeywords(pageNumber: number): string[] {
		const keywordSets = [
			['인공지능', '튜링테스트', 'AI역사', '딥러닝'],
			['머신러닝', '지도학습', '비지도학습', '강화학습'],
			['신경망', '퍼셉트론', '역전파', '활성화함수'],
			['컴퓨터비전', 'CNN', '이미지인식', '객체탐지'],
			['자연어처리', 'NLP', '트랜스포머', '임베딩'],
			['강화학습', 'Q-learning', 'Policy', 'Reward'],
			['AI윤리', '편향성', '프라이버시', '공정성'],
			['의료AI', '진단', '신약개발', '정밀의료'],
			['자율주행', '센서', 'LIDAR', '컴퓨터비전'],
			['핀테크', '알고리즘트레이딩', '신용평가', '사기탐지'],
			['에듀테크', '개인화학습', '학습분석', 'AI튜터'],
			['XAI', '설명가능성', '해석가능성', '투명성'],
			['엣지AI', '경량화', '임베디드', '실시간추론'],
			['AGI', '범용인공지능', '특이점', '미래전망'],
			['참고문헌', '논문', '학습자료', '컨퍼런스']
		];
		
		return keywordSets[(pageNumber - 1) % keywordSets.length];
	}
	
	// 전체 요약 복사
	function copyAllSummaries() {
		const allSummaries = pageSummaries.map(page => 
			`[${page.pageNumber}페이지]\n${page.summary}\n키워드: ${page.keywords.join(', ')}`
		).join('\n\n');
		
		navigator.clipboard.writeText(allSummaries);
	}
	
	// 페이지별 요약 복사
	function copyPageSummary(page: PageSummary) {
		const text = `[${page.pageNumber}페이지]\n${page.summary}\n키워드: ${page.keywords.join(', ')}`;
		navigator.clipboard.writeText(text);
	}
	
	// 새 파일 분석
	function analyzeNewFile() {
		selectedFile = null;
		showAnalysis = false;
		pageSummaries = [];
		if (fileInput) {
			fileInput.value = '';
		}
	}
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-6xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">PDF 문서 분석기</h1>
			<p class="mt-2 text-gray-600">PDF 파일을 업로드하면 페이지별로 요약해 드립니다</p>
		</div>

		{#if !showAnalysis}
			<!-- 파일 업로드 영역 -->
			<div class="bg-white rounded-lg shadow-md p-8">
				<div
					class="border-2 border-dashed {dragActive ? 'border-green-500 bg-green-50' : 'border-gray-300'} rounded-lg p-12 text-center transition-colors"
					ondragover={handleDragOver}
					ondragleave={handleDragLeave}
					ondrop={handleDrop}
				>
					<input
						bind:this={fileInput}
						type="file"
						accept=".pdf"
						onchange={handleFileSelect}
						class="hidden"
						id="pdf-upload"
					/>
					
					<div class="mb-4">
						<span class="text-6xl">📄</span>
					</div>
					
					<label for="pdf-upload" class="cursor-pointer">
						<p class="text-lg font-medium text-gray-900 mb-2">
							PDF 파일을 드래그하거나 클릭하여 업로드
						</p>
						<p class="text-sm text-gray-500">
							최대 50MB까지 지원됩니다
						</p>
					</label>
				</div>

				{#if selectedFile}
					<div class="mt-6 bg-gray-50 rounded-lg p-4">
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

					<div class="mt-6 text-center">
						<button
							onclick={analyzePDF}
							disabled={isAnalyzing}
							class="px-8 py-3 bg-green-600 text-white font-medium rounded-lg hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 disabled:bg-gray-300 disabled:cursor-not-allowed transition duration-200"
						>
							{#if isAnalyzing}
								<span class="inline-flex items-center">
									<svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
										<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
										<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
									</svg>
									분석 중...
								</span>
							{:else}
								PDF 분석 시작
							{/if}
						</button>
					</div>
				{/if}
			</div>
		{:else}
			<!-- 분석 결과 -->
			<div class="bg-white rounded-lg shadow-md p-6 mb-6">
				<div class="flex justify-between items-start mb-4">
					<div>
						<h2 class="text-xl font-bold text-gray-900">{pdfInfo.fileName}</h2>
						<div class="flex space-x-4 mt-2 text-sm text-gray-600">
							<span>📄 {pdfInfo.totalPages}페이지</span>
							<span>💾 {pdfInfo.fileSize}</span>
							<span>🕐 {pdfInfo.analysisTime}</span>
						</div>
					</div>
					<div class="flex space-x-2">
						<button
							onclick={copyAllSummaries}
							class="px-4 py-2 text-sm bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition duration-200"
						>
							📋 전체 복사
						</button>
						<button
							onclick={analyzeNewFile}
							class="px-4 py-2 text-sm bg-green-600 text-white rounded-lg hover:bg-green-700 transition duration-200"
						>
							📄 새 파일
						</button>
					</div>
				</div>

				<!-- 페이지 네비게이션 -->
				<div class="border-t pt-4">
					<div class="flex items-center justify-between mb-4">
						<h3 class="text-lg font-semibold text-gray-900">페이지별 요약</h3>
						<div class="flex items-center space-x-2">
							<button
								onclick={() => selectedPage = Math.max(1, selectedPage - 1)}
								disabled={selectedPage === 1}
								class="p-2 rounded hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
							>
								←
							</button>
							<span class="px-3 py-1 bg-gray-100 rounded text-sm font-medium">
								{selectedPage} / {pdfInfo.totalPages}
							</span>
							<button
								onclick={() => selectedPage = Math.min(pdfInfo.totalPages, selectedPage + 1)}
								disabled={selectedPage === pdfInfo.totalPages}
								class="p-2 rounded hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
							>
								→
							</button>
						</div>
					</div>

					<!-- 현재 페이지 요약 -->
					{#if pageSummaries[selectedPage - 1]}
						{@const currentPage = pageSummaries[selectedPage - 1]}
						<div class="bg-green-50 rounded-lg p-4 mb-4">
							<div class="flex justify-between items-start mb-3">
								<div class="flex items-center space-x-2">
									<span class="text-lg font-semibold text-green-900">
										{currentPage.pageNumber}페이지
									</span>
									<span class="px-2 py-1 bg-green-200 text-green-800 text-xs rounded-full">
										{currentPage.type === 'text' ? '텍스트' : currentPage.type === 'image' ? '이미지' : '혼합'}
									</span>
								</div>
								<button
									onclick={() => copyPageSummary(currentPage)}
									class="text-green-600 hover:text-green-800"
									title="이 페이지 복사"
								>
									📋
								</button>
							</div>
							<p class="text-gray-800 mb-3 leading-relaxed">{currentPage.summary}</p>
							<div class="flex flex-wrap gap-2">
								{#each currentPage.keywords as keyword}
									<span class="px-2 py-1 bg-green-100 text-green-700 text-xs rounded">
										{keyword}
									</span>
								{/each}
							</div>
						</div>
					{/if}

					<!-- 페이지 목록 -->
					<div class="grid grid-cols-5 gap-2 mt-4">
						{#each pageSummaries as page}
							<button
								onclick={() => selectedPage = page.pageNumber}
								class="p-2 text-sm rounded {selectedPage === page.pageNumber ? 'bg-green-600 text-white' : 'bg-gray-100 hover:bg-gray-200'} transition-colors"
							>
								{page.pageNumber}
							</button>
						{/each}
					</div>
				</div>
			</div>

			<!-- 사용 팁 -->
			<div class="grid md:grid-cols-2 gap-4">
				<div class="bg-blue-50 rounded-lg p-4">
					<h4 class="font-semibold text-blue-900 mb-2">💡 활용 팁</h4>
					<ul class="space-y-1 text-sm text-blue-800">
						<li>• 논문이나 보고서의 핵심 내용을 빠르게 파악하세요</li>
						<li>• 키워드를 활용해 관련 자료를 검색하세요</li>
						<li>• 요약본을 기반으로 발표 자료를 준비하세요</li>
					</ul>
				</div>
				<div class="bg-amber-50 rounded-lg p-4">
					<h4 class="font-semibold text-amber-900 mb-2">⚠️ 주의사항</h4>
					<ul class="space-y-1 text-sm text-amber-800">
						<li>• 개인정보가 포함된 문서는 주의하세요</li>
						<li>• 요약은 참고용으로만 활용하세요</li>
						<li>• 중요한 내용은 원문을 확인하세요</li>
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