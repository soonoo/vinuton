<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	let isRecording = $state(false);
	let isPaused = $state(false);
	let recordingTime = $state(0);
	let showSummary = $state(false);
	let meetingSummary = $state('');
	let timer: number;
	let mediaRecorder: MediaRecorder | null = null;
	let audioChunks: Blob[] = [];

	// 시각화를 위한 변수들
	let audioContext: AudioContext | null = null;
	let analyser: AnalyserNode | null = null;
	let animationId: number;
	let visualizerBars = $state<number[]>(new Array(20).fill(0));

	// 시간 포맷팅 함수
	function formatTime(seconds: number): string {
		const mins = Math.floor(seconds / 60);
		const secs = seconds % 60;
		return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
	}

	// 녹음 시작
	async function startRecording() {
		try {
			const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
			
			// MediaRecorder 설정
			mediaRecorder = new MediaRecorder(stream);
			audioChunks = [];
			
			mediaRecorder.ondataavailable = (event) => {
				audioChunks.push(event.data);
			};

			mediaRecorder.onstop = () => {
				// 녹음 종료 시 처리 (실제로는 서버로 전송)
				const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
				console.log('Recording complete', audioBlob);
			};

			// 오디오 시각화 설정
			audioContext = new AudioContext();
			analyser = audioContext.createAnalyser();
			const source = audioContext.createMediaStreamSource(stream);
			source.connect(analyser);
			analyser.fftSize = 64;
			
			visualize();
			
			mediaRecorder.start();
			isRecording = true;
			showSummary = false;
			
			// 타이머 시작
			timer = setInterval(() => {
				recordingTime++;
			}, 1000);
		} catch (error) {
			console.error('마이크 접근 권한이 필요합니다:', error);
			alert('마이크 접근 권한이 필요합니다.');
		}
	}

	// 오디오 시각화
	function visualize() {
		if (!analyser) return;
		
		const bufferLength = analyser.frequencyBinCount;
		const dataArray = new Uint8Array(bufferLength);
		
		function draw() {
			animationId = requestAnimationFrame(draw);
			analyser!.getByteFrequencyData(dataArray);
			
			// 시각화 바 업데이트
			const bars = [];
			const step = Math.floor(bufferLength / 20);
			for (let i = 0; i < 20; i++) {
				const value = dataArray[i * step];
				bars.push((value / 255) * 100);
			}
			visualizerBars = bars;
		}
		
		draw();
	}

	// 녹음 중지
	function stopRecording() {
		if (mediaRecorder && mediaRecorder.state !== 'inactive') {
			mediaRecorder.stop();
			mediaRecorder.stream.getTracks().forEach(track => track.stop());
		}
		
		if (animationId) {
			cancelAnimationFrame(animationId);
		}
		
		if (audioContext) {
			audioContext.close();
		}
		
		clearInterval(timer);
		isRecording = false;
		isPaused = false;
		visualizerBars = new Array(20).fill(0);
		
		// 더미 요약본 생성 (실제로는 API 호출)
		generateSummary();
	}

	// 일시정지/재개
	function togglePause() {
		if (!mediaRecorder) return;
		
		if (isPaused) {
			mediaRecorder.resume();
			timer = setInterval(() => {
				recordingTime++;
			}, 1000);
		} else {
			mediaRecorder.pause();
			clearInterval(timer);
		}
		isPaused = !isPaused;
	}

	// 더미 요약본 생성
	function generateSummary() {
		// 실제로는 API 호출하여 음성을 텍스트로 변환 후 요약
		meetingSummary = `📝 회의 요약

📅 일시: ${new Date().toLocaleString('ko-KR')}
⏱️ 회의 시간: ${formatTime(recordingTime)}

🎯 주요 안건:
1. 프로젝트 진행 상황 점검
   - 프론트엔드 개발 80% 완료
   - 백엔드 API 구현 진행 중
   - 디자인 시안 최종 확정

2. 다음 주 일정 논의
   - 월요일: 코드 리뷰 세션
   - 수요일: 클라이언트 미팅
   - 금요일: 스프린트 회고

3. 이슈 사항
   - 성능 최적화 필요
   - 보안 취약점 패치 예정
   - 문서화 작업 보완 필요

✅ 액션 아이템:
• [김개발] 로그인 기능 버그 수정 (~ 11/22)
• [이디자인] 모바일 반응형 디자인 수정 (~ 11/23)
• [박기획] 사용자 피드백 정리 및 공유 (~ 11/21)
• [전체] 다음 스프린트 계획 수립 (~ 11/24)

💡 주요 결정 사항:
- 출시일을 12월 1일로 확정
- 베타 테스트는 11월 25일부터 시작
- 추가 기능은 2차 업데이트로 연기

📌 다음 회의: 2024년 11월 24일 오후 3시`;
		
		recordingTime = 0;
		showSummary = true;
	}

	// 요약본 복사
	function copySummary() {
		navigator.clipboard.writeText(meetingSummary);
	}

	// 새 녹음 시작
	function startNewRecording() {
		showSummary = false;
		recordingTime = 0;
		meetingSummary = '';
		startRecording();
	}

	onDestroy(() => {
		if (isRecording) {
			stopRecording();
		}
	});
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-4xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">회의록 자동 생성기</h1>
			<p class="mt-2 text-gray-600">회의 내용을 녹음하면 자동으로 요약본을 만들어 드립니다</p>
		</div>

		{#if !showSummary}
			<!-- 녹음 인터페이스 -->
			<div class="bg-white rounded-lg shadow-md p-8">
				<!-- 시각화 영역 -->
				<div class="mb-8 flex justify-center items-center h-32">
					{#if isRecording}
						<div class="flex items-end space-x-1 h-full">
							{#each visualizerBars as bar}
								<div 
									class="w-3 bg-red-500 rounded-t transition-all duration-100"
									style="height: {bar}%; min-height: 4px;"
								></div>
							{/each}
						</div>
					{:else}
						<div class="text-6xl text-gray-300">🎙️</div>
					{/if}
				</div>

				<!-- 타이머 -->
				<div class="text-center mb-8">
					<div class="text-4xl font-mono font-bold text-gray-900">
						{formatTime(recordingTime)}
					</div>
					{#if isRecording}
						<p class="text-sm text-red-600 mt-2">
							{isPaused ? '일시정지됨' : '녹음 중...'}
						</p>
					{/if}
				</div>

				<!-- 컨트롤 버튼 -->
				<div class="flex justify-center space-x-4">
					{#if !isRecording}
						<button
							onclick={startRecording}
							class="px-8 py-4 bg-red-600 text-white font-medium rounded-full hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-200 flex items-center space-x-2"
						>
							<span class="text-xl">⏺️</span>
							<span>녹음 시작</span>
						</button>
					{:else}
						<button
							onclick={togglePause}
							class="p-4 bg-gray-600 text-white rounded-full hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition duration-200"
							title={isPaused ? '재개' : '일시정지'}
						>
							<span class="text-xl">{isPaused ? '▶️' : '⏸️'}</span>
						</button>
						<button
							onclick={stopRecording}
							class="px-8 py-4 bg-gray-900 text-white font-medium rounded-full hover:bg-black focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition duration-200 flex items-center space-x-2"
						>
							<span class="text-xl">⏹️</span>
							<span>녹음 종료</span>
						</button>
					{/if}
				</div>

				<!-- 안내 메시지 -->
				<div class="mt-8 bg-blue-50 rounded-lg p-4">
					<h3 class="font-semibold text-blue-900 mb-2">💡 사용 팁</h3>
					<ul class="space-y-1 text-sm text-blue-800">
						<li>• 조용한 환경에서 녹음하면 더 정확한 요약을 받을 수 있습니다</li>
						<li>• 발언자가 바뀔 때 이름을 말하면 구분이 쉬워집니다</li>
						<li>• 중요한 결정사항은 명확하게 언급해주세요</li>
						<li>• 녹음은 최대 60분까지 가능합니다</li>
					</ul>
				</div>
			</div>
		{:else}
			<!-- 요약 결과 -->
			<div class="bg-white rounded-lg shadow-md p-8 animate-fade-in">
				<div class="flex justify-between items-center mb-6">
					<h2 class="text-2xl font-bold text-gray-900">회의록 요약</h2>
					<div class="space-x-2">
						<button
							onclick={copySummary}
							class="px-4 py-2 text-sm bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 transition duration-200"
						>
							📋 복사하기
						</button>
						<button
							onclick={startNewRecording}
							class="px-4 py-2 text-sm bg-red-600 text-white rounded-lg hover:bg-red-700 transition duration-200"
						>
							🎙️ 새 녹음
						</button>
					</div>
				</div>

				<div class="bg-gray-50 rounded-lg p-6">
					<pre class="whitespace-pre-wrap text-sm text-gray-800 leading-relaxed font-sans">{meetingSummary}</pre>
				</div>

				<!-- 추가 기능 안내 -->
				<div class="mt-6 grid md:grid-cols-2 gap-4">
					<div class="bg-green-50 rounded-lg p-4">
						<h4 class="font-semibold text-green-900 mb-2">✅ 포함된 내용</h4>
						<ul class="space-y-1 text-sm text-green-800">
							<li>• 주요 안건 및 논의 사항</li>
							<li>• 액션 아이템과 담당자</li>
							<li>• 결정 사항 및 다음 일정</li>
						</ul>
					</div>
					<div class="bg-amber-50 rounded-lg p-4">
						<h4 class="font-semibold text-amber-900 mb-2">📌 활용 방법</h4>
						<ul class="space-y-1 text-sm text-amber-800">
							<li>• 팀원들과 공유하여 정리</li>
							<li>• 프로젝트 문서에 보관</li>
							<li>• 액션 아이템 트래킹</li>
						</ul>
					</div>
				</div>
			</div>
		{/if}
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